# Payment Gateway System Documentation

## Overview

A comprehensive payment gateway system enabling merchants to accept online payments through multiple mobile wallets (bKash, Nagad, Rocket, Upay) via agent networks.

### System Architecture

```
Customer → Merchant Website → Payment Gateway API → Agent (Mobile Wallet) → Customer
```

### Core Concepts

**Agent Balance System:**
- Agent deposits money and receives a limit with commission bonus
- Example: $500 deposit + 2% bonus = $510 available limit
- **DEPOSIT (Cash In)**: Customer pays agent → Agent balance decreases
- **WITHDRAWAL (Cash Out)**: Agent pays customer → Agent balance increases

---

## Table of Contents

1. [User Roles](#user-roles)
2. [Admin Guide](#admin-guide)
3. [Agent Guide](#agent-guide)
4. [Merchant Guide](#merchant-guide)
5. [Transaction Flows](#transaction-flows)
6. [Security Features](#security-features)
7. [Technical Reference](#technical-reference)

---

## User Roles

### Super Admin
**Full system control with all permissions**

- ✅ Manage all staff, admins, editors
- ✅ Create/manage merchants and agents
- ✅ Configure global and individual charges
- ✅ Approve topups and withdrawals
- ✅ Assign permissions to staff
- ✅ Access all reports and audit logs

### Admin
**System management without admin creation**

- ✅ Manage merchants and agents
- ✅ Configure charges
- ✅ Approve topups and withdrawals
- ✅ View reports and settings
- ❌ Cannot create other admins

### Staff
**Limited operations with configurable permissions**

- ✅ Create merchants and agents (if granted)
- ✅ Approve topups (if granted)
- ✅ View transactions and basic reports
- ❌ Cannot modify charges
- ❌ Cannot approve withdrawals

### Editor
**View-only with basic edit capabilities**

- ✅ View all data (read-only)
- ✅ Edit merchant/agent basic information
- ✅ View reports
- ❌ No approvals or deletions

### Agent
**Payment processing handler**

- ✅ Receive and process deposit payments
- ✅ Process withdrawal payments
- ✅ Request topups
- ✅ Manage payment gateways
- ✅ View earnings and transaction history

### Merchant
**API integration for payment processing**

- ✅ Create deposit/withdrawal requests via API
- ✅ Query transaction status
- ✅ View balance and history
- ✅ Request withdrawals

---

## Admin Guide

### Initial Setup

#### Step 1: Configure Global Settings

```typescript
interface GlobalSettings {
  merchantCharges: {
    depositPercent: number;      // Default: 1.5%
    withdrawalPercent: number;   // Default: 1.5%
    minCharge: number;           // Default: $5
    maxCharge: number;           // Default: $500
  };
  agentCommissions: {
    depositPercent: number;      // Default: 0.5%
    topupBonus: number;          // Default: 2%
    withdrawalPercent: number;   // Default: 0.5%
  };
  transactionLimits: {
    minAmount: number;           // Default: $10
    maxAmount: number;           // Default: $50,000
    timeout: number;             // Default: 15 minutes
    autoApprovalTimeout: number; // Default: 5 minutes
  };
}
```

#### Step 2: Create Staff Accounts

**Form Fields:**
- Name, Email, Role (Admin/Staff/Editor)
- Permissions (checkboxes based on role)
- 2FA requirement

**Result:** Staff receives email with credentials and must set up 2FA on first login.

---

### Creating Merchant Accounts

```typescript
interface CreateMerchantRequest {
  // Basic Information
  businessName: string;
  email: string;
  phone: string;
  password?: string; // Auto-generated if not provided
  
  // API Credentials (Auto-generated)
  merchantCode: string;  // e.g., "MER-00001"
  apiKey: string;        // UUID
  secretKey: string;     // For MD5 token generation
  
  // Security
  whitelistedIps: string[];
  callbackUrl: string;
  returnUrl: string;
  
  // Optional: Custom Charges (overrides global)
  customCharges?: {
    depositPercent: number;
    withdrawalPercent: number;
  };
  
  // Limits
  dailyDepositLimit: number;
  dailyWithdrawalLimit: number;
  minWithdrawalAmount: number;
  
  // Settings
  allowedGateways: ('BKASH' | 'NAGAD' | 'ROCKET' | 'UPAY')[];
  autoApproveWithdrawals: boolean;
  status: 'ACTIVE' | 'INACTIVE';
}
```

**Merchant receives:**
- Login credentials
- API Key & Secret Key
- Integration documentation
- Test API endpoints

---

### Creating Agent Accounts

```typescript
interface CreateAgentRequest {
  // Basic Information
  name: string;
  email: string;
  phone: string;
  agentCode: string;  // Auto-generated: "AGT-00001"
  password?: string;   // Auto-generated if not provided
  
  // Optional: Custom Commissions (overrides global)
  customCommissions?: {
    depositPercent: number;
    topupBonus: number;
    withdrawalPercent: number;
  };
  
  // Limits
  dailyTransactionLimit: number;
  maxConcurrentTransactions: number; // Default: 1
  
  // Security
  allowedIps?: string[];
  
  status: 'PENDING' | 'ACTIVE'; // Changes to ACTIVE after first topup
}
```

**Agent receives:**
- Login credentials
- Mobile app download link
- Instructions to add payment gateways
- Topup instructions

---

### Approving Agent Topup

**Topup Request Details:**

```typescript
interface TopupRequest {
  agent: {
    id: string;
    agentCode: string;
    name: string;
  };
  amount: number;
  bonusPercent: number;
  bonusAmount: number;
  totalLimit: number;
  paymentMethod: string;
  paymentProof: string; // Screenshot URL
  remarks?: string;
  createdAt: Date;
}
```

**Approval Process:**

```typescript
async function approveTopup(topupId: string): Promise<void> {
  const topup = await getTopupRequest(topupId);
  const agent = await getAgent(topup.agentId);
  
  // Calculate amounts
  const depositAmount = topup.amount;
  const bonusPercent = agent.customTopupBonus || globalSettings.agentTopupBonus;
  const bonusAmount = depositAmount * (bonusPercent / 100);
  const totalLimit = depositAmount + bonusAmount;
  
  // Update agent balance
  agent.balance += totalLimit;
  agent.limitBalance += totalLimit;
  
  if (agent.status === 'PENDING') {
    agent.status = 'ACTIVE';
  }
  
  await agent.save();
  
  // Record in balance history
  await AgentBalanceHistory.create({
    agentId: agent.id,
    type: 'TOPUP',
    amount: totalLimit,
    description: `Topup approved - $${depositAmount} + $${bonusAmount} bonus (${bonusPercent}%)`,
  });
  
  // Notify agent
  await sendNotification(agent, {
    title: 'Topup Approved',
    body: `Your topup of $${depositAmount} has been approved. Bonus: $${bonusAmount}. New limit: $${totalLimit}`,
  });
  
  // Update topup status
  topup.status = 'APPROVED';
  await topup.save();
}
```

---

### Managing Charges

#### Global Charges Configuration

```typescript
interface ChargesConfig {
  merchantCharges: {
    deposit: {
      type: 'PERCENTAGE' | 'FIXED' | 'BOTH';
      percentage?: number;
      fixedAmount?: number;
      minCharge?: number;
      maxCharge?: number;
    };
    withdrawal: {
      type: 'PERCENTAGE' | 'FIXED' | 'BOTH';
      percentage?: number;
      fixedAmount?: number;
    };
  };
  agentCommissions: {
    depositPercent: number;
    topupBonus: number;
    withdrawalPercent: number;
  };
}
```

#### Individual Merchant Charges

```typescript
interface MerchantCharges {
  merchantId: string;
  overrideGlobal: boolean;
  customDepositPercent?: number;
  customWithdrawalPercent?: number;
}

// Example: VIP merchant with lower rates
const vipMerchant: MerchantCharges = {
  merchantId: 'mer-uuid',
  overrideGlobal: true,
  customDepositPercent: 1.2, // Instead of global 1.5%
  customWithdrawalPercent: 1.2,
};
```

---

### Approving Merchant Withdrawals

```typescript
interface WithdrawalRequest {
  merchant: {
    id: string;
    merchantCode: string;
    businessName: string;
    currentBalance: number;
  };
  amount: number;
  charge: number;
  netAmount: number;
  bankDetails: {
    bankName: string;
    accountNumber: string;
    accountName: string;
  };
  remarks?: string;
  status: 'PENDING' | 'PROCESSING' | 'COMPLETED' | 'REJECTED';
  createdAt: Date;
}

async function approveWithdrawal(withdrawalId: string): Promise<void> {
  const withdrawal = await getWithdrawalRequest(withdrawalId);
  const merchant = await getMerchant(withdrawal.merchantId);
  
  // Step 1: Update status to PROCESSING
  withdrawal.status = 'PROCESSING';
  await withdrawal.save();
  
  // Step 2: Admin manually transfers money to merchant's bank
  // Step 3: Admin uploads payment proof
  
  // Step 4: Mark as completed
  withdrawal.status = 'COMPLETED';
  withdrawal.completedAt = new Date();
  
  // Update merchant balance
  merchant.balance -= withdrawal.amount;
  await merchant.save();
  
  // Record in balance history
  await MerchantBalanceHistory.create({
    merchantId: merchant.id,
    type: 'WITHDRAWAL',
    amount: -withdrawal.amount,
    description: `Withdrawal completed - ${withdrawal.netAmount} transferred`,
  });
  
  await MerchantBalanceHistory.create({
    merchantId: merchant.id,
    type: 'CHARGE',
    amount: -withdrawal.charge,
    description: `Withdrawal charge - ${withdrawal.charge}`,
  });
  
  // Send email with payment proof
  await sendEmail(merchant.email, {
    subject: 'Withdrawal Completed',
    body: 'Your withdrawal has been processed.',
    attachments: [withdrawal.paymentProof],
  });
  
  await withdrawal.save();
}
```

---

### Approving Agent Device Login

```typescript
interface DeviceApprovalRequest {
  agent: {
    id: string;
    agentCode: string;
    name: string;
  };
  device: {
    deviceId: string;
    deviceName: string;
    deviceOs: string;
    appVersion: string;
  };
  ipAddress: string;
  requestedAt: Date;
}

async function approveDevice(requestId: string): Promise<void> {
  const request = await getDeviceApprovalRequest(requestId);
  
  // Deactivate all other devices for this agent
  await DeviceSession.updateMany(
    {
      agentId: request.agentId,
      id: { $ne: request.id },
    },
    { isActive: false }
  );
  
  // Approve this device
  request.status = 'APPROVED';
  request.isActive = true;
  request.approvedAt = new Date();
  await request.save();
  
  // Notify agent
  await sendPushNotification(request.agentId, {
    title: 'Device Approved',
    body: 'Your device has been approved. You can now use the app.',
  });
}
```

**Security Feature:** Agent can only have 1 active device at a time.

---

### Dashboard Overview

```typescript
interface AdminDashboard {
  todayStats: {
    totalTransactions: number;
    successful: number;
    pending: number;
    failed: number;
    successRate: number;
    revenue: number;        // Total charges collected
    totalVolume: number;    // Total transaction amount
  };
  pendingActions: {
    agentTopupRequests: number;
    merchantWithdrawalRequests: number;
    deviceLoginApprovals: number;
    transactionsAwaitingReview: number;
  };
  overview: {
    activeAgents: number;
    totalAgentLimit: number;
    activeMerchants: number;
    totalMerchantBalance: number;
  };
}
```

---

## Agent Guide

### Registration & Setup

**Step 1:** Receive email with credentials and app download link  
**Step 2:** Download mobile app and enter credentials  
**Step 3:** Wait for admin to approve device login  
**Step 4:** Complete profile and set up 2FA  

---

### Requesting Topup

```typescript
interface TopupRequestForm {
  amount: number;
  paymentProof: File; // Bank transfer screenshot
  remarks?: string;
}

// Example
const topupRequest: TopupRequestForm = {
  amount: 500,
  paymentProof: uploadedFile,
  remarks: 'Transferred from my BRAC account ending 4567',
};

// Calculation
const bonusPercent = 2; // From global or custom setting
const bonusAmount = 500 * 0.02; // 10
const totalLimit = 500 + 10; // 510
```

**After approval:**
- Agent balance: $0 → $510
- Agent can start receiving payments

---

### Adding Payment Gateways

```typescript
interface AgentGateway {
  gatewayType: 'BKASH' | 'NAGAD' | 'ROCKET' | 'UPAY';
  accountNumber: string;
  accountName: string;
  dailyLimit?: number;
  priority: number; // 1 = highest
  isActive: boolean;
}

// Example: Agent can add multiple gateways
const gateways: AgentGateway[] = [
  {
    gatewayType: 'BKASH',
    accountNumber: '01711222333',
    accountName: 'Agent Karim',
    priority: 1,
    isActive: true,
  },
  {
    gatewayType: 'NAGAD',
    accountNumber: '01811222333',
    accountName: 'Agent Karim',
    priority: 2,
    isActive: true,
  },
];
```

---

### Receiving Deposit Payments (Cash In)

#### Flow Overview

```
1. Merchant creates deposit → System selects agent
2. Customer redirected to payment page → Sees agent's wallet number
3. Customer pays agent's wallet → Gets TrxID
4. Two scenarios:
   a) Agent app SMS arrives first
   b) Customer submits TrxID first
5. System matches both → Transaction completed
```

#### Scenario A: Agent SMS Arrives First

**Step 1:** Customer pays agent's wallet  
**Step 2:** Agent receives SMS from mobile wallet

```
SMS from bKash:
"You have received 100.00 BDT from 01811555666.
TrxID: ABC123XYZ. Balance: 1,500 BDT."
```

**Step 3:** Agent app automatically parses SMS and sends callback

```typescript
interface SMSCallbackPayload {
  agentId: string;
  gatewayType: 'BKASH' | 'NAGAD' | 'ROCKET' | 'UPAY';
  amount: number;
  senderNumber: string;
  trxId: string;
  timestamp: Date;
}

// App automatically parses SMS
const parsed: SMSCallbackPayload = {
  agentId: 'agent-uuid',
  gatewayType: 'BKASH',
  amount: 100,
  senderNumber: '01811555666',
  trxId: 'ABC123XYZ',
  timestamp: new Date(),
};

// Send to backend immediately
await fetch('/api/agent/sms-callback', {
  method: 'POST',
  body: JSON.stringify(parsed),
});
```

**Step 4:** Backend stores SMS data and waits for customer submission

```typescript
async function handleSMSCallback(payload: SMSCallbackPayload): Promise<void> {
  // Look for matching PENDING transaction
  const transaction = await Transaction.findOne({
    agentId: payload.agentId,
    amount: payload.amount,
    gatewayType: payload.gatewayType,
    status: 'PENDING',
  });
  
  if (transaction) {
    // Store SMS data
    transaction.agentReceivedTrxId = payload.trxId;
    transaction.agentReceivedFrom = payload.senderNumber;
    transaction.agentReceivedAt = new Date();
    transaction.status = 'PROCESSING'; // Partial match
    await transaction.save();
    
    // Wait for customer to submit TrxID
  }
}
```

**Step 5:** Customer submits TrxID → Backend validates immediately

```typescript
async function handleCustomerSubmission(
  transactionId: string,
  trxId: string
): Promise<void> {
  const transaction = await Transaction.findById(transactionId);
  
  // Check if SMS already received
  if (transaction.agentReceivedTrxId === trxId) {
    // ✓ MATCH! SMS already received, TrxIDs match
    await completeTransaction(transaction);
  }
}
```

#### Scenario B: Customer Submits TrxID First

**Step 1:** Customer pays and immediately submits TrxID  
**Step 2:** Backend stores customer submission

```typescript
async function handleCustomerSubmission(
  transactionId: string,
  trxId: string
): Promise<void> {
  const transaction = await Transaction.findById(transactionId);
  
  transaction.customerSubmittedTrxId = trxId;
  transaction.customerSubmittedAt = new Date();
  transaction.status = 'PROCESSING'; // Partial match
  await transaction.save();
  
  // Wait for agent SMS callback
}
```

**Step 3:** Agent app receives SMS and sends callback (5-30 seconds later)  
**Step 4:** Backend validates immediately

```typescript
async function handleSMSCallback(payload: SMSCallbackPayload): Promise<void> {
  // Check if customer already submitted
  const transaction = await Transaction.findOne({
    agentId: payload.agentId,
    customerSubmittedTrxId: payload.trxId,
    status: 'PROCESSING',
  });
  
  if (transaction && transaction.customerSubmittedTrxId === payload.trxId) {
    // ✓ MATCH! Customer already submitted, TrxIDs match
    await completeTransaction(transaction);
  }
}
```

#### Transaction Completion (Both Scenarios)

```typescript
async function completeTransaction(transaction: Transaction): Promise<void> {
  const agent = await getAgent(transaction.agentId);
  const merchant = await getMerchant(transaction.merchantId);
  
  // 1. Update transaction status
  transaction.status = 'SUCCESS';
  transaction.matchedAt = new Date();
  transaction.completedAt = new Date();
  
  // 2. Calculate amounts
  const depositAmount = transaction.amount;
  
  const merchantChargePercent =
    merchant.customDepositChargePercent ||
    globalSettings.merchantDepositPercent; // 1.5%
  const merchantCharge = depositAmount * (merchantChargePercent / 100); // 1.50
  const merchantNetAmount = depositAmount - merchantCharge; // 98.50
  
  const agentCommissionPercent =
    agent.customDepositCommissionPercent ||
    globalSettings.agentDepositCommissionPercent; // 0.5%
  const agentCommission = depositAmount * (agentCommissionPercent / 100); // 0.50
  
  transaction.merchantCharge = merchantCharge;
  transaction.agentCommission = agentCommission;
  transaction.netAmount = merchantNetAmount;
  
  // 3. Update Agent Balance (DEDUCT)
  const agentBalanceBefore = agent.balance; // 510
  agent.balance = agent.balance - depositAmount; // 410
  agent.holdBalance = agent.holdBalance - depositAmount; // Release hold
  agent.successfulTransactions += 1;
  
  // Add commission to agent
  agent.balance = agent.balance + agentCommission; // 410.50
  await agent.save();
  
  // Record in balance history
  await AgentBalanceHistory.createMany([
    {
      agentId: agent.id,
      type: 'DEPOSIT_RECEIVED',
      amount: -depositAmount,
      balanceBefore: agentBalanceBefore,
      balanceAfter: agentBalanceBefore - depositAmount,
      description: `Deposit received - TXN-${transaction.systemTransactionId}`,
    },
    {
      agentId: agent.id,
      type: 'COMMISSION',
      amount: agentCommission,
      balanceBefore: agentBalanceBefore - depositAmount,
      balanceAfter: agent.balance,
      description: `Commission earned - ${agentCommissionPercent}%`,
    },
  ]);
  
  // 4. Update Merchant Balance (ADD)
  const merchantBalanceBefore = merchant.balance;
  merchant.balance = merchant.balance + merchantNetAmount;
  await merchant.save();
  
  await MerchantBalanceHistory.createMany([
    {
      merchantId: merchant.id,
      type: 'DEPOSIT',
      amount: merchantNetAmount,
      balanceBefore: merchantBalanceBefore,
      balanceAfter: merchant.balance,
      description: `Deposit received - TXN-${transaction.systemTransactionId}`,
    },
    {
      merchantId: merchant.id,
      type: 'CHARGE',
      amount: -merchantCharge,
      description: `Service charge - ${merchantChargePercent}%`,
    },
  ]);
  
  // 5. Send callback to merchant immediately
  await sendMerchantCallback(transaction);
  
  // 6. Notify agent via app
  await sendPushNotification(agent.id, {
    title: 'Payment Received ✓',
    body: `$${depositAmount} received. Balance: $${agent.balance}`,
    data: { transactionId: transaction.id },
  });
  
  await transaction.save();
}
```

**Summary:**
- Customer paid: $100
- Agent balance: $510 → $410.50 (deducted $100, earned $0.50)
- Merchant balance: $0 → $98.50 (received net after $1.50 charge)
- System revenue: $1.50 - $0.50 = $1.00

---

### Auto-Approval System (5-Minute Rule)

If agent app fails to send callback within 5 minutes, system auto-approves.

```typescript
// Background job runs every 30 seconds
async function checkAutoApproval(): Promise<void> {
  const fiveMinutesAgo = new Date(Date.now() - 5 * 60 * 1000);
  
  // Find transactions waiting for agent confirmation
  const stuckTransactions = await Transaction.findMany({
    where: {
      status: 'PROCESSING',
      customerSubmittedAt: { lte: fiveMinutesAgo },
      agentReceivedTrxId: null,
    },
  });
  
  for (const transaction of stuckTransactions) {
    // Auto-approve the transaction
    transaction.status = 'SUCCESS';
    transaction.isAutoApproved = true;
    transaction.autoApprovedAt = new Date();
    
    // Process as normal
    await completeTransaction(transaction);
    
    // Log for review
    await AuditLog.create({
      actorType: 'SYSTEM',
      action: 'AUTO_APPROVE',
      resource: 'TRANSACTION',
      resourceId: transaction.id,
      metadata: {
        reason: 'No agent SMS callback received within 5 minutes',
        customerSubmittedAt: transaction.customerSubmittedAt,
      },
    });
    
    // Alert admin
    await SecurityAlert.create({
      alertType: 'AUTO_APPROVED_TRANSACTION',
      severity: 'MEDIUM',
      title: 'Transaction Auto-Approved',
      description: `Transaction ${transaction.systemTransactionId} was auto-approved after 5 minutes.`,
    });
  }
}
```

**Why this works:**
- Prevents customers from waiting indefinitely
- Merchant receives callback and completes order
- Admin is alerted to review the transaction

---

### Manual Approval

Agent can manually verify and approve transactions.

```typescript
interface ManualApprovalPayload {
  transactionId: string;
  agentReceivedTrxId: string;
  proof?: string; // Screenshot URL
}

async function manualApprove(payload: ManualApprovalPayload): Promise<void> {
  const transaction = await Transaction.findById(payload.transactionId);
  
  transaction.agentReceivedTrxId = payload.agentReceivedTrxId;
  transaction.manualApprovalProof = payload.proof;
  
  // Same flow as automatic approval
  await completeTransaction(transaction);
}
```

---

### Handling Withdrawal Payments (Cash Out)

#### Flow Overview

```
1. Merchant creates withdrawal → System selects agent
2. Agent receives notification
3. Agent pays customer manually via wallet
4. Agent submits TrxID
5. Agent balance increases (gets paid back + commission)
6. Merchant receives callback
```

#### Processing Withdrawal

**Step 1:** Agent receives notification

```typescript
interface WithdrawalNotification {
  transactionId: string;
  amount: number;
  gatewayType: 'BKASH' | 'NAGAD' | 'ROCKET' | 'UPAY';
  recipientNumber: string;
  recipientName: string;
}
```

**Step 2:** Agent pays customer via mobile wallet  
**Step 3:** Agent submits TrxID

```typescript
interface ConfirmWithdrawalPayload {
  transactionId: string;
  agentPaidTrxId: string;
  proof?: string; // Screenshot URL
}

async function confirmWithdrawal(payload: ConfirmWithdrawalPayload): Promise<void> {
  const transaction = await Transaction.findById(payload.transactionId);
  const agent = await getAgent(transaction.agentId);
  
  // 1. Update transaction
  transaction.status = 'SUCCESS';
  transaction.agentReceivedTrxId = payload.agentPaidTrxId;
  transaction.manualApprovalProof = payload.proof;
  transaction.completedAt = new Date();
  
  // 2. Calculate amounts
  const withdrawalAmount = transaction.amount;
  
  const agentCommissionPercent =
    agent.customWithdrawalCommissionPercent ||
    globalSettings.agentWithdrawalCommissionPercent; // 0.5%
  const agentCommission = withdrawalAmount * (agentCommissionPercent / 100); // 0.25
  
  transaction.agentCommission = agentCommission;
  
  // 3. Update Agent Balance (ADD - agent gets money back + commission)
  const agentBalanceBefore = agent.balance;
  
  // Agent gets withdrawal amount back (they paid customer)
  agent.balance = agent.balance + withdrawalAmount + agentCommission;
  await agent.save();
  
  // Record in balance history
  await AgentBalanceHistory.createMany([
    {
      agentId: agent.id,
      type: 'WITHDRAWAL_PAID',
      amount: withdrawalAmount,
      balanceBefore: agentBalanceBefore,
      balanceAfter: agentBalanceBefore + withdrawalAmount,
      description: `Withdrawal paid to customer - TXN-${transaction.systemTransactionId}`,
    },
    {
      agentId: agent.id,
      type: 'COMMISSION',
      amount: agentCommission,
      balanceBefore: agentBalanceBefore + withdrawalAmount,
      balanceAfter: agent.balance,
      description: `Withdrawal commission - ${agentCommissionPercent}%`,
    },
  ]);
  
  // 4. Send callback to merchant
  await sendMerchantCallback(transaction);
  
  // 5. Notify agent
  await sendPushNotification(agent.id, {
    title: 'Withdrawal Completed ✓',
    body: `Paid $${withdrawalAmount}. Earned $${agentCommission}. Balance: $${agent.balance}`,
  });
  
  await transaction.save();
}
```

**Summary:**
- Agent paid customer: $50
- Agent commission: $0.25
- Agent balance: $410.50 → $460.75 (got back $50 + earned $0.25)

---

### Agent Dashboard

```typescript
interface AgentDashboard {
  agentInfo: {
    name: string;
    balance: number;
    availableLimit: number;
    holdBalance: number;
    status: 'ACTIVE' | 'INACTIVE';
    appVersion: string;
  };
  todaySummary: {
    totalTransactions: number;
    depositsReceived: { count: number; amount: number };
    withdrawalsPaid: { count: number; amount: number };
    commissionEarned: number;
  };
  pendingActions: {
    depositsAwaitingConfirmation: number;
    withdrawalRequestsAssigned: number;
  };
}
```

---

## Merchant Guide

### Integration via API

#### Creating a Deposit (Customer Payment)

```typescript
interface DepositRequest {
  merchantCode: string;
  amount: number;
  reference: string; // Unique order ID
  customerName: string;
  customerPhone: string;
  customerIp: string;
  transactionTime: string;
  paymentMethod: 'BKASH' | 'NAGAD' | 'ROCKET' | 'UPAY';
  callbackUrl: string;
  returnUrl: string;
  securityToken: string; // MD5 hash
}

// Generate security token
function generateSecurityToken(
  merchantCode: string,
  amount: number,
  reference: string,
  transactionTime: string,
  callbackUrl: string,
  returnUrl: string,
  secretKey: string
): string {
  const tokenString = `${merchantCode}${amount}${reference}${transactionTime}${callbackUrl}${returnUrl}${secretKey}`;
  return md5(tokenString);
}

// Example usage
const depositData: DepositRequest = {
  merchantCode: 'MER-00001',
  amount: 1000,
  reference: `ORDER_${Date.now()}`,
  customerName: 'Customer Name',
  customer
