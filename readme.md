# Payment Gateway System - Complete Workflow Documentation

## Table of Contents

1. [System Overview](about:blank#system-overview)
2. [User Roles & Permissions](about:blank#user-roles--permissions)
3. [Admin Workflow](about:blank#admin-workflow)
4. [Agent Workflow](about:blank#agent-workflow)
5. [Merchant Workflow](about:blank#merchant-workflow)
6. [Transaction Flows](about:blank#transaction-flows)
7. [Security Features](about:blank#security-features)
8. [Technical Implementation](about:blank#technical-implementation)

---

## System Overview

### Architecture

```
Customer (Website User)
↓
Merchant Website 
↓
Payment Gateway API
↓
Agent (Manual Payment Handler)
↓
E-wallet (bKash, Nagad, Rocket, Upay)
```

### Key Concepts

**Agent Balance System:**
- Agent deposits money → Gets limit with commission bonus
- Example: Agent deposits $500 with 2% commission → Gets $510 limit
- When agent receives customer payment (CASH IN): `balance - amount`
- When agent pays customer (CASH OUT): `balance + amount + commission`

**Transaction Types:**
1. **DEPOSIT (Cash In)**: Customer paying merchant
- Customer pays agent’s wallet
- Agent balance decreases
- Merchant balance increases (minus charges)

1. **WITHDRAWAL (Cash Out)**: Merchant paying customer
    - Agent pays customer from their wallet
    - Agent balance increases (plus commission)
    - Merchant balance decreases

---

## User Roles & Permissions

### 1. Super Admin

**Full System Control**

Permissions:
- ✅ Manage all admins, staff, editors
- ✅ Manage merchants and agents
- ✅ Configure global and individual charges
- ✅ Approve/Reject topups and withdrawals
- ✅ Manage system settings
- ✅ View all reports and analytics
- ✅ Assign/Unassign permissions to staff
- ✅ Approve device logins
- ✅ Access audit logs

### 2. Admin

**System Management (No Admin Creation)**

Permissions:
- ✅ Manage merchants and agents
- ✅ Configure charges
- ✅ Approve/Reject topups and withdrawals
- ✅ View reports
- ✅ Manage settings
- ✅ Approve device logins
- ❌ Cannot create other admins

### 3. Staff

**Limited Operations**

Configurable Permissions (assigned by Super Admin):
- ✅ Create merchants
- ✅ Create agents
- ✅ Approve topups (if granted)
- ✅ View transactions
- ✅ Basic reports
- ❌ Cannot modify charges
- ❌ Cannot approve withdrawals

### 4. Editor

**View-Only + Basic Edits**

Permissions:
- ✅ View all data (read-only)
- ✅ Edit merchant/agent basic info
- ✅ View reports
- ❌ No approvals
- ❌ No charge configuration
- ❌ No deletions

### 5. Agent

**Payment Handler**

Permissions:
- ✅ View assigned transactions
- ✅ Approve/Reject payments manually
- ✅ Request topups
- ✅ Manage payment gateways (bKash, Nagad, etc.)
- ✅ View transaction history
- ✅ View earnings and commissions

### 6. Merchant

**API Integration**

Permissions:
- ✅ Create deposit/withdrawal requests via API
- ✅ Query transaction status
- ✅ View balance
- ✅ Request withdrawals
- ✅ View transaction history

---

## Admin Workflow

### 1. Initial Setup

**Step 1: Super Admin Creates System**
```
1. Install system
2. Run database migrations
3. Seed initial super admin
4. Configure global charge settings
```

**Step 2: Configure Global Settings**
```sql
– Set global merchant charges
Deposit Charge: 1.5% (default)
Withdrawal Charge: 1.5% (default)

– Set global agent commissions
Deposit Commission: 0.5% (agent earns when receiving payments)
Topup Bonus: 2% (agent gets when depositing to their account)
Withdrawal Commission: 0.5% (agent earns when paying out)

– Set limits
Min Transaction: $10
Max Transaction: $50,000
Transaction Timeout: 15 minutes
Auto-Approval Timeout: 5 minutes
```

### 2. Creating Staff Accounts

**Admin Panel → Staff Management → Create Staff**

```
Form Fields:
- Name: “John Doe”
- Email: “john@example.com”
- Role: [Admin | Staff | Editor]
- Permissions: [Checkboxes based on role]
☑ Manage Merchants
☑ Manage Agents
☐ Manage Charges
☑ Approve Topups
☐ Approve Withdrawals
☑ View Reports

- 2FA: Enable/Disable
```

**Result:**
- Staff receives email with credentials
- Must set up 2FA on first login
- Can only access features per assigned permissions

### 3. Creating Merchant Accounts

**Admin Panel → Merchants → Create Merchant**

```
Form Fields:
Basic Info:
- Business Name: “Example Store”
- Email: “merchant@example.com”
- Phone: “+8801711222333”
- Password: [Auto-generated or custom]

API Credentials:
- Merchant Code: MER-00001 [Auto-generated]
- API Key: [Auto-generated UUID]
- Secret Key: [Auto-generated]

Security:
- Whitelisted IPs: [“123.45.67.89”, “123.45.67.90”]
- Callback URL: “https://merchant.com/payment/callback”
- Return URL: “https://merchant.com/payment/success”

Charge Settings (Optional - Override Global):
□ Use Custom Charges
Deposit Charge: 1.2% (instead of global 1.5%)
Withdrawal Charge: 1.2%

Limits:
- Daily Deposit Limit: $100,000
- Daily Withdrawal Limit: $50,000
- Min Withdrawal Amount: $100

Settings:
- Allowed Gateways: [☑ bKash] [☑ Nagad] [☑ Rocket] [☑ Upay]
- Auto-Approve Withdrawals: No

Status: Active
```

**Result:**
- Merchant receives email with:
- Login credentials
- API Key & Secret Key
- Integration documentation
- Test API endpoints

### 4. Creating Agent Accounts

**Admin Panel → Agents → Create Agent**

```
Form Fields:
Basic Info:
- Name: “Agent Karim”
- Email: “agent1@example.com”
- Phone: “+8801811333444”
- Agent Code: AGT-00001 [Auto-generated]
- Password: [Auto-generated]

Commission Settings (Optional - Override Global):
□ Use Custom Commissions
Deposit Commission: 0.6% (instead of global 0.5%)
Topup Bonus: 2.5% (instead of global 2%)
Withdrawal Commission: 0.6%

Limits:
- Daily Transaction Limit: $10,000
- Max Concurrent Transactions: 1 (can’t have 2 pending at same time)

Security:
- Allowed IPs: [“103.45.67.89”] (optional)

Status: Pending (changes to Active after first topup)
```

**Result:**
- Agent receives email with:
- Login credentials
- Mobile app download link
- Instructions to add payment gateways
- Topup instructions

### 5. Approving Agent Topup

**Scenario: Agent Requests Topup**

**Admin Panel → Topup Requests → Pending**

```
Request Details:
- Agent: AGT-00001 (Agent Karim)
- Amount: $500
- Bonus Percentage: 2% (from global or custom)
- Bonus Amount: $10
- Total Limit to Add: $510
- Payment Method: Bank Transfer
- Payment Proof: [View Screenshot]
- Agent Remarks: “Transferred to account ending 1234”
- Date: 2025-01-15 10:30 AM

Actions:
[Approve] [Reject]
```

**If Admin Approves:**
```
1. Agent limit increases: $510
2. Agent balance increases: $510
3. Record in AgentBalanceHistory:
- Type: TOPUP
- Amount: +$510
- Description: “Topup approved - $500 + $10 bonus (2%)”

1. Agent receives notification:
“Your topup of $500 has been approved.
Bonus: $10 (2%).
New limit: $510”
2. Agent status changes to ACTIVE (if was PENDING)
```

**If Admin Rejects:**
```
1. Agent receives notification with reason
2. No balance change
```

### 6. Managing Charges

### Global Charges

**Admin Panel → Settings → Charges → Global**

```
Merchant Charges:
┌─────────────────────────────────────────┐
│ Deposit Charges │
│ Type: [Percentage ▼] │
│ Percentage: 1.5 % │
│ Fixed Amount: 0 BDT │
│ Min Charge: 5 BDT │
│ Max Charge: 500 BDT │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ Withdrawal Charges │
│ Type: [Percentage ▼] │
│ Percentage: 1.5 % │
│ Fixed Amount: 0 BDT │
└─────────────────────────────────────────┘

Agent Commissions:
┌─────────────────────────────────────────┐
│ Deposit Commission: 0.5 % │
│ Topup Bonus: 2.0 % │
│ Withdrawal Commission: 0.5 % │
└─────────────────────────────────────────┘

[Save Global Settings]
```

### Individual Merchant Charges

**Admin Panel → Merchants → Edit Merchant → Charges Tab**

```
Merchant: MER-00001 (Example Store)

□ Override Global Charges

If checked:
┌─────────────────────────────────────────┐
│ Custom Deposit Charges │
│ Type: [Percentage ▼] │
│ Percentage: 1.2 % (VIP merchant rate) │
└─────────────────────────────────────────┘

Note: If not checked, global charges apply (1.5%)
```

### Individual Agent Commissions

**Admin Panel → Agents → Edit Agent → Commission Tab**

```
Agent: AGT-00001 (Agent Karim)

□ Override Global Commissions

If checked:
┌─────────────────────────────────────────┐
│ Deposit Commission: 0.6 % │
│ Topup Bonus: 2.5 % │
│ Withdrawal Commission: 0.6 % │
└─────────────────────────────────────────┘

Note: If not checked, global commissions apply
```

### 7. Approving Merchant Withdrawals

**Scenario: Merchant Requests Withdrawal**

**Admin Panel → Withdrawal Requests → Pending**

```
Request Details:
- Merchant: MER-00001 (Example Store)
- Amount: $5,000
- Charge: $75 (1.5%)
- Net Amount: $4,925
- Bank: BRAC Bank
- Account Number: 1234567890
- Account Name: Example Store Ltd.
- Merchant Remarks: “Weekly settlement”
- Date: 2025-01-15 14:00 PM

Current Merchant Balance: $12,500

Actions:
[Approve] [Reject]
```

**If Admin Approves:**
```
Step 1: Update status to PROCESSING
Step 2: Admin manually transfers $4,925 to merchant’s bank
Step 3: Admin uploads payment proof
Step 4: Click “Mark as Completed”

Result:
1. Merchant balance: $12,500 - $5,000 = $7,500
2. Charge collected: $75
3. Email sent to merchant with payment proof
4. Status: COMPLETED
```

### 8. Approving Agent Device Login

**Scenario: Agent Logs into Mobile App**

**Admin Panel → Device Approvals → Pending**

```
Login Request:
- Agent: AGT-00001 (Agent Karim)
- Device ID: a1b2c3d4e5f6
- Device Name: “Karim’s Samsung A52”
- Device OS: Android 12
- App Version: 1.2.3
- IP Address: 103.45.67.89
- Date: 2025-01-15 09:00 AM

Actions:
[Approve] [Reject]
```

**If Admin Approves:**
```
1. Device status: APPROVED
2. Agent can now use the app
3. Only this device can be active at a time
4. If agent logs in from another device, this one is logged out
```

**Security Feature:**
- Agent can only have 1 active device
- Admin can revoke device access anytime
- All device activities are logged

### 9. Dashboard Overview

**Admin Dashboard**

```
┌─────────────────────────────────────────────────────────┐
│ Today’s Stats │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ Total Transactions: 1,245 │
│ Successful: 1,180 (94.8%) │
│ Pending: 45 │
│ Failed: 20 │
│ │
│ Revenue: $1,875 (charges collected) │
│ Total Volume: $125,000 │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│ Pending Actions │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ 🔔 8 Agent Topup Requests │
│ 🔔 3 Merchant Withdrawal Requests │
│ 🔔 2 Device Login Approvals │
│ 🔔 5 Transactions Awaiting Manual Review │
└─────────────────────────────────────────────────────────┘

┌──────────────────────┬──────────────────────────────────┐
│ Active Agents: 45 │ Active Merchants: 120 │
│ Total Limit: $2.5M │ Total Balance: $1.8M │
└──────────────────────┴──────────────────────────────────┘
```

---

## Agent Workflow

### 1. Agent Registration & Setup

**Step 1: Admin Creates Agent Account**
- Agent receives email with credentials and app download link

**Step 2: Agent Downloads Mobile App**
```
1. Download app from provided link
2. Enter credentials
3. Device login request sent to admin
4. Wait for admin approval
5. Receive approval notification
6. Complete profile setup
```

**Step 3: Agent Sets Up 2FA**
```
1. Open Google Authenticator
2. Scan QR code from app
3. Enter 6-digit code
4. 2FA enabled
```

### 2. Agent Topup Process

**Agent App → Topup → Request New Topup**

```
Step 1: Enter Topup Amount
┌─────────────────────────────────────────┐
│ Topup Amount: $500 │
│ Bonus (2%): +$10 │
│ Total Limit: $510 │
└─────────────────────────────────────────┘

Step 2: Make Payment
Instructions:
“Transfer $500 to the following account:
Bank: BRAC Bank
Account: 1234567890
Account Name: Your Company Ltd.”

Step 3: Upload Proof
[Upload Screenshot of Bank Transfer]

Step 4: Add Remarks (Optional)
“Transferred from my BRAC account ending 4567”

[Submit Topup Request]
```

**Result:**
```
Request submitted ✓
Status: Pending
You’ll be notified once admin approves your topup.
```

**After Admin Approves:**
```
🎉 Topup Approved!

Amount: $500
Bonus: $10 (2%)
New Balance: $510

You can now start receiving payments.
```

**Balance Calculation Logic:**
```javascript
// When topup is approved
const depositAmount = 500;
const bonusPercent = 2; // from global or custom setting
const bonusAmount = depositAmount * (bonusPercent / 100); // 500 * 0.02 = 10
const totalLimit = depositAmount + bonusAmount; // 510

agent.balance += totalLimit; // 0 + 510 = 510
agent.limitBalance += totalLimit;
```

### 3. Adding Payment Gateways

**Agent App → Payment Gateways → Add Gateway**

```
Step 1: Select Gateway Type
[bKash] [Nagad] [Rocket] [Upay]

Step 2: Enter Details
┌─────────────────────────────────────────┐
│ Gateway: bKash │
│ Account Number: 01711222333 │
│ Account Name: Agent Karim │
│ Daily Limit: $5,000 (optional) │
│ Priority: 1 (1=highest) │
│ Status: [Active ▼] │
└─────────────────────────────────────────┘

[Add Gateway]
```

**Agent Can Add Multiple Gateways:**
```
✓ bKash - 01711222333 (Active)
✓ Nagad - 01811222333 (Active)
✓ Rocket - 01911222333 (Inactive)
```

### 4. Receiving Deposit Payments (Cash In)

**Scenario: Customer Makes Payment**

### **Flow Overview:**

```
1. Merchant creates deposit on your system
2. Your system selects random agent
3. Customer redirected to payment page
4. Customer sees agent’s wallet number
5. Customer pays agent’s wallet
6. Two possible scenarios:
a) Agent app SMS arrives first
b) Customer submits TrxID first
7. System matches both
8. Agent balance deducted
9. Merchant receives callback
```

### **Scenario A: Agent SMS Arrives First**

**Step 1: Customer Pays Agent’s Wallet**
```
Customer opens bKash app
Customer sends $100 to 01711222333 (agent’s number)
bKash TrxID: ABC123XYZ
```

**Step 2: Agent Receives SMS**
```
SMS from bKash:
“You have received 100.00 BDT from 01811555666.
TrxID: ABC123XYZ.
Fee: 0.00 BDT.
Balance: 1,500 BDT.”
```

**Step 3: Agent App Parses SMS & Sends Callback**
```javascript
// App automatically parses SMS
const parsed = {
gatewayType: “BKASH”,
amount: 100,
senderNumber: “01811555666”,
trxId: “ABC123XYZ”
};

// Send to backend immediately
POST /api/agent/sms-callback
{
agentId: “agent-uuid”,
gatewayType: “BKASH”,
amount: 100,
senderNumber: “01811555666”,
trxId: “ABC123XYZ”,
timestamp: “2025-01-15T10:30:45Z”
}
```

**Step 4: Backend Stores SMS Data**
```javascript
// Backend receives callback
// Look for matching PENDING transaction
const transaction = await findTransaction({
agentId: agent.id,
amount: 100,
gatewayType: “BKASH”,
status: “PENDING”
});

if (transaction) {
// Found matching transaction, but customer hasn’t submitted TrxID yet
transaction.agentReceivedTrxId = “ABC123XYZ”;
transaction.agentReceivedFrom = “01811555666”;
transaction.agentReceivedAt = new Date();
transaction.status = “PROCESSING”; // Partial match
await transaction.save();

// Wait for customer to submit TrxID
}
```

**Step 5: Customer Submits TrxID**
```
Customer returns to payment page
Customer enters TrxID: ABC123XYZ
Customer clicks “Submit”
```

**Step 6: Backend Validates Immediately**
```javascript
// Backend receives customer submission
POST /api/payment/submit-trxid
{
transactionId: “txn-uuid”,
trxId: “ABC123XYZ”,
senderNumber: “01811555666” // optional
}

// Check if SMS already received
const transaction = await getTransaction(transactionId);

if (transaction.agentReceivedTrxId === “ABC123XYZ”) {
// ✓ MATCH! SMS already received, TrxIDs match

// Immediate processing
await completeTransaction(transaction);
}
```

### **Scenario B: Customer Submits TrxID First**

**Step 1: Customer Pays & Submits TrxID**
```
Customer pays agent: 01711222333
Customer immediately submits TrxID: XYZ789ABC
```

**Step 2: Backend Stores Customer Submission**
```javascript
// Backend receives customer submission
const transaction = await getTransaction(transactionId);

transaction.customerSubmittedTrxId = “XYZ789ABC”;
transaction.customerSenderNumber = “01811555666”;
transaction.customerSubmittedAt = new Date();
transaction.status = “PROCESSING”; // Partial match
await transaction.save();

// Wait for agent SMS callback
```

**Step 3: Agent App Receives SMS & Sends Callback**
```
(SMS arrives 5-30 seconds later)

Agent app parses SMS and sends callback with TrxID: XYZ789ABC
```

**Step 4: Backend Validates Immediately**
```javascript
// Backend receives SMS callback
POST /api/agent/sms-callback
{
trxId: “XYZ789ABC”,
amount: 100,
// …
}

// Check if customer already submitted
const transaction = await findTransaction({
agentId: agent.id,
customerSubmittedTrxId: “XYZ789ABC”,
status: “PROCESSING”
});

if (transaction && transaction.customerSubmittedTrxId === “XYZ789ABC”) {
// ✓ MATCH! Customer already submitted, TrxIDs match

// Immediate processing
await completeTransaction(transaction);
}
```

### **Transaction Completion (Both Scenarios)**

```javascript
async function completeTransaction(transaction) {
// 1. Update transaction status
transaction.status = “SUCCESS”;
transaction.matchedAt = new Date();
transaction.completedAt = new Date();

// 2. Calculate amounts
const depositAmount = 100;
const merchantChargePercent = merchant.customDepositChargePercent || globalSetting.merchantDepositPercent; // 1.5%
const merchantCharge = depositAmount * (merchantChargePercent / 100); // 100 * 0.015 = 1.50
const merchantNetAmount = depositAmount - merchantCharge; // 100 - 1.50 = 98.50

const agentCommissionPercent = agent.customDepositCommissionPercent || globalSetting.agentDepositCommissionPercent; // 0.5%
const agentCommission = depositAmount * (agentCommissionPercent / 100); // 100 * 0.005 = 0.50

transaction.merchantCharge = merchantCharge;
transaction.agentCommission = agentCommission;
transaction.netAmount = merchantNetAmount;

// 3. Update Agent Balance (DEDUCT)
const agentBalanceBefore = agent.balance; // 510
agent.balance = agent.balance - depositAmount; // 510 - 100 = 410
agent.holdBalance = agent.holdBalance - depositAmount; // Release hold
agent.successfulTransactions += 1;
await agent.save();

// Record in balance history
await AgentBalanceHistory.create({
agentId: agent.id,
type: “DEPOSIT_RECEIVED”,
amount: -depositAmount, // Negative because it’s deducted
balanceBefore: agentBalanceBefore,
balanceAfter: agent.balance,
description: `Deposit received - TXN-${transaction.systemTransactionId}`,
referenceType: “Transaction”,
referenceId: transaction.id
});

// Add commission to agent
agent.balance = agent.balance + agentCommission; // 410 + 0.50 = 410.50
await agent.save();

await AgentBalanceHistory.create({
agentId: agent.id,
type: “COMMISSION”,
amount: +agentCommission,
balanceBefore: 410,
balanceAfter: 410.50,
description: `Commission earned - ${agentCommissionPercent}%`,
referenceType: “Transaction”,
referenceId: transaction.id
});

// 4. Update Merchant Balance (ADD)
const merchantBalanceBefore = merchant.balance;
merchant.balance = merchant.balance + merchantNetAmount; // Add net amount
await merchant.save();

await MerchantBalanceHistory.create({
merchantId: merchant.id,
type: “DEPOSIT”,
amount: +merchantNetAmount,
balanceBefore: merchantBalanceBefore,
balanceAfter: merchant.balance,
description: `Deposit received - TXN-${transaction.systemTransactionId}`,
referenceType: “Transaction”,
referenceId: transaction.id
});

// Record charge
await MerchantBalanceHistory.create({
merchantId: merchant.id,
type: “CHARGE”,
amount: -merchantCharge,
balanceBefore: merchant.balance,
balanceAfter: merchant.balance,
description: `Service charge - ${merchantChargePercent}%`,
referenceType: “Transaction”,
referenceId: transaction.id
});

// 5. Send callback to merchant immediately
await sendMerchantCallback(transaction);

// 6. Notify agent via app
await sendPushNotification(agent, {
title: “Payment Received ✓”,
body: `$${depositAmount} received. Balance: $${agent.balance}`,
data: { transactionId: transaction.id }
});

return transaction;
}
```

**Summary:**
```
Customer paid: $100
Agent balance: $510 → $410.50 (deducted $100, earned $0.50 commission)
Merchant balance: $0 → $98.50 (received net amount after $1.50 charge)
Your revenue: $1.50 - $0.50 = $1.00
```

### 5. Auto-Approval System (5-Minute Rule)

**Scenario: Agent App Fails to Send Callback**

```javascript
// Background job runs every 30 seconds
async function checkAutoApproval() {
const fiveMinutesAgo = new Date(Date.now() - 5 * 60 * 1000);

// Find transactions waiting for agent confirmation
const stuckTransactions = await Transaction.findMany({
where: {
status: “PROCESSING”, // Customer submitted TrxID but no SMS callback
customerSubmittedAt: { lte: fiveMinutesAgo }, // More than 5 minutes ago
agentReceivedTrxId: null // No SMS received yet
}
});

for (const transaction of stuckTransactions) {
// Auto-approve the transaction
transaction.status = “SUCCESS”;
transaction.isAutoApproved = true;
transaction.autoApprovedAt = new Date();

```
// Process as normal
await completeTransaction(transaction);

// Log for review
await AuditLog.create({
  actorType: "SYSTEM",
  action: "AUTO_APPROVE",
  resource: "TRANSACTION",
  resourceId: transaction.id,
  metadata: {
    reason: "No agent SMS callback received within 5 minutes",
    customerSubmittedAt: transaction.customerSubmittedAt
  }
});

// Alert admin
await SecurityAlert.create({
  alertType: "AUTO_APPROVED_TRANSACTION",
  severity: "MEDIUM",
  subjectType: "TRANSACTION",
  subjectId: transaction.id,
  title: "Transaction Auto-Approved",
  description: `Transaction ${transaction.systemTransactionId} was auto-approved after 5 minutes without agent confirmation.`,
  metadata: { agentId: transaction.agentId }
});
```

}
}
```

**Why This Works:**
- If customer pays and submits TrxID, but agent app crashes or fails
- After 5 minutes, system assumes payment was successful
- Merchant receives callback and completes order
- Admin is alerted to review the transaction
- Prevents customer waiting indefinitely

### 6. Manual Approval (Agent Panel)

**Scenario: Agent Wants to Manually Verify**

**Agent App → Pending Transactions**

```
┌─────────────────────────────────────────┐
│ Transaction #TXN-00123 │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ Amount: $100 │
│ Gateway: bKash │
│ Your Number: 01711222333 │
│ Customer TrxID: ABC123XYZ │
│ Customer Number: 01811555666 │
│ Status: Processing │
│ Time: 2 minutes ago │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ [Approve] [Reject] [View Details] │
└─────────────────────────────────────────┘
```

**Agent Clicks “Approve”:**
```
Confirm Approval
─────────────────
Did you receive $100 from 01811555666?

TrxID from your SMS: [ ABC123XYZ ]

□ I confirm I received this payment

[Upload Screenshot (Optional)]

[Confirm Approval]
```

**Backend Processing:**
```javascript
POST /api/agent/manual-approve
{
transactionId: “txn-uuid”,
agentReceivedTrxId: “ABC123XYZ”,
proof: “screenshot-url.jpg”
}

// Same flow as automatic approval
await completeTransaction(transaction);
```

### 7. Handling Withdrawal Payments (Cash Out)

**Scenario: Merchant Requests Withdrawal for Customer**

### **Flow Overview:**

```
1. Merchant creates withdrawal request via API
2. Your system selects random agent
3. Agent receives notification
4. Agent pays customer manually
5. Agent submits TrxID
6. Agent balance increases (gets paid back + commission)
7. Merchant receives callback
```

**Step 1: Agent Receives Notification**
```
🔔 New Withdrawal Request

Transaction: TXN-00456
Amount: $50
Gateway: bKash
Pay to: 01822444555
Customer Name: Aminul Islam

[View Details]
```

**Step 2: Agent Opens Transaction**
```
┌─────────────────────────────────────────┐
│ Withdrawal Request │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ Pay Amount: $50 │
│ To Number: 01822444555 │
│ Customer: Aminul Islam │
│ From Gateway: bKash - 01711222333 │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ Instructions: │
│ 1. Send $50 to 01822444555 via bKash │
│ 2. Get TrxID from bKash │
│ 3. Enter TrxID below │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ TrxID: [ ] │
│ [Upload Screenshot (Optional)] │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ [Confirm Payment] [Cancel] │
└─────────────────────────────────────────┘
```

**Step 3: Agent Pays Customer**
```
Agent opens bKash app
Agent sends $50 to 01822444555
bKash confirms: TrxID XYZ789ABC
Agent returns to app
Agent enters: XYZ789ABC
Agent clicks “Confirm Payment”
```

**Step 4: Backend Processes Withdrawal**
```javascript
POST /api/agent/confirm-withdrawal
{
transactionId: “txn-uuid”,
agentPaidTrxId: “XYZ789ABC”,
proof: “screenshot-url.jpg”
}

async function processWithdrawal(transaction, agentTrxId, proof) {
// 1. Update transaction
transaction.status = “SUCCESS”;
transaction.agentReceivedTrxId = agentTrxId; // Agent PAID, so it’s “received” by customer
transaction.manualApprovalProof = proof;
transaction.completedAt = new Date();

// 2. Calculate amounts
const withdrawalAmount = 50;

// Agent commission for processing withdrawal
const agentCommissionPercent = agent.customWithdrawalCommissionPercent || globalSetting.agentWithdrawalCommissionPercent; // 0.5%
const agentCommission = withdrawalAmount * (agentCommissionPercent / 100); // 50 * 0.005 = 0.25

transaction.agentCommission = agentCommission;

// 3. Update Agent Balance (ADD - agent gets money back + commission)
const agentBalanceBefore = agent.balance; // 410.50

// Agent gets withdrawal amount back (they paid customer)
agent.balance = agent.balance + withdrawalAmount; // 410.50 + 50 = 460.50

// Plus commission
agent.balance = agent.balance + agentCommission; // 460.50 + 0.25 = 460.75

await agent.save();

// Record withdrawal payment
await AgentBalanceHistory.create({
agentId: agent.id,
type: “WITHDRAWAL_PAID”,
amount: +withdrawalAmount,
balanceBefore: agentBalanceBefore,
balanceAfter: agentBalanceBefore + withdrawalAmount,
description: `Withdrawal paid to customer - TXN-${transaction.systemTransactionId}`,
referenceType: “Transaction”,
referenceId: transaction.id
});

// Record commission
await AgentBalanceHistory.create({
agentId: agent.id,
type: “COMMISSION”,
amount: +agentCommission,
balanceBefore: agentBalanceBefore + withdrawalAmount,
balanceAfter: agent.balance,
description: `Withdrawal commission - ${agentCommissionPercent}%`,
referenceType: “Transaction”,
referenceId: transaction.id
});

// 4. Update Merchant Balance (already deducted when request created)
// No change needed here

// 5. Send callback to merchant
await sendMerchantCallback(transaction);

// 6. Notify agent
await sendPushNotification(agent, {
title: “Withdrawal Completed ✓”,
body: `Paid $${withdrawalAmount}. Earned $${agentCommission}. Balance: $${agent.balance}`
});

return transaction;
}
```

**Summary:**
```
Agent paid customer: $50
Agent commission: $0.25
Agent balance: $410.50 → $460.75 (got back $50 + earned $0.25)
Merchant balance: Already deducted when request was created
```

### 8. Agent Dashboard

**Agent App Home Screen**

```
┌─────────────────────────────────────────┐
│ Welcome, Agent Karim │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ Balance: $460.75 │
│ Available Limit: $460.75 │
│ On Hold: $0.00 │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ Status: 🟢 Active │
│ App Version: 1.2.3 │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ Today’s Summary │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ Transactions: 15 │
│ Deposits Received: 12 ($1,200) │
│ Withdrawals Paid: 3 ($150) │
│ Commission Earned: $6.75 │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ Pending Actions (2) │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ 🔔 1 Deposit awaiting confirmation │
│ 🔔 1 Withdrawal request assigned │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ [View Pending] │
└─────────────────────────────────────────┘

┌───────────────┬─────────────────────────┐
│ [Topup] │ [Gateways] │
├───────────────┼─────────────────────────┤
│ [History] │ [Reports] │
└───────────────┴─────────────────────────┘
```

---

## Merchant Workflow

### 1. Merchant Registration

**Merchant Receives Email from Admin:**
```
Subject: Your Payment Gateway Account

Dear Example Store,

Your payment gateway account has been created.

Login Credentials:
- Merchant Code: MER-00001
- Email: merchant@example.com
- Password: [temp-password]
- Dashboard: https://portal.yourgateway.com

API Credentials:
- API Key: a1b2c3d4-e5f6-g7h8-i9j0-k1l2m3n4o5p6
- Secret Key: secret_a1b2c3d4e5f6g7h8i9j0

Integration URLs:
- Staging API: https://staging-api.yourgateway.com
- Production API: https://api.yourgateway.com
- Documentation: https://docs.yourgateway.com

Please login and change your password immediately.
```

### 2. Merchant Integration (API)

### **Creating a Deposit (Customer Payment)**

**Merchant Backend Code (PHP Example):**

```php
 ‘MER-00001’,
‘amount’ => 1000, // BDT
‘reference’ => ‘ORDER_’ . time(), // Unique order ID
‘customerName’ => ‘Customer Name’,
‘customerPhone’ => ‘01711222333’,
‘customerIp’ => $_SERVER[‘REMOTE_ADDR’],
‘transactionTime’ => date(‘Y-m-d H:i:s’),
‘paymentMethod’ => ‘BKASH’, // or NAGAD, ROCKET, UPAY
‘callbackUrl’ => ‘https://merchant.com/payment/callback’,
‘returnUrl’ => ‘https://merchant.com/payment/success’
];

// Step 2: Generate security token (MD5 hash)
$secretKey = ‘secret_a1b2c3d4e5f6g7h8i9j0’;

$tokenString =
$data[‘merchantCode’] .
$data[‘amount’] .
$data[‘reference’] .
$data[‘transactionTime’] .
$data[‘callbackUrl’] .
$data[‘returnUrl’] .
$secretKey;

*data*[′*securityToken*′] = *md*5(tokenString);

// Step 3: Call payment gateway API
*ch* = *curlinit*(′*https* : //*api*.*yourgateway*.*com*/*v*1/*deposit*′); *curlsetopt*(ch, CURLOPT_POST, true);
curl_setopt(*ch*, *CURLOPTPOSTFIELDS*, *jsonencode*(data));
curl_setopt(*ch*, *CURLOPTRETURNTRANSFER*, *true*); *curlsetopt*(ch, CURLOPT_HTTPHEADER, [
‘Content-Type: application/json’,
‘X-API-Key: a1b2c3d4-e5f6-g7h8-i9j0-k1l2m3n4o5p6’]);

*response* = *curlexec*(ch);
curl_close($ch);

*result* = *jsondecode*(response, true);

// Step 4: Redirect customer to payment page
if ($result[‘status’] === ‘success’) {
header(‘Location:’ . $result[‘redirectUrl’]);
exit;
} else {
die(‘Payment initiation failed:’ . $result[‘description’]);
}
?>
```

**Your Gateway API Response:**
```json
{
“status”: “success”,
“description”: “Transaction created successfully”,
“systemTransactionId”: “TXN-00123”,
“redirectUrl”: “https://pay.yourgateway.com/p/abc123def456”
}
```

### **Payment Page (Your System)**

**Customer Lands on: `https://pay.yourgateway.com/p/abc123def456`**

```html
┌─────────────────────────────────────────┐
│ Your Gateway - Secure Payment │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ Merchant: Example Store │
│ Order: ORDER_1736957400 │
│ Amount: ৳1,000.00 │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ Payment Method: bKash │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ 📱 Send Money To: │
│ │
│ 01711-222333 │
│ │
│ Instructions: │
│ 1. Open your bKash app │
│ 2. Select “Send Money” │
│ 3. Enter: 01711222333 │
│ 4. Amount: ৳1,000.00 │
│ 5. Complete payment │
│ 6. Enter TrxID below │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ Your bKash TrxID: │
│ [ ABC123XYZ_________________ ] │
│ │
│ Sender Number (Optional): │
│ [ 01811555666_______________ ] │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ [Submit Payment] │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ ⏱ Time Remaining: 14:30 │
│ 🔒 Secure Payment │
└─────────────────────────────────────────┘
```

**Security Features on Payment Page:**
- ✅ Agent number only shown if agent has no pending transactions
- ✅ Agent number only shown if agent has sufficient balance
- ✅ Agent selected randomly (or weighted by success rate)
- ✅ Transaction expires in 15 minutes
- ✅ SSL encrypted connection

### **Receiving Callback (Merchant Backend)**

**Your Gateway Sends Callback to Merchant:**

```php
 ‘00000’]); // Already processed
exit;
}

// Step 3: Process based on status
if ($data['status'] === '00000') { // Success
    // Update order status
    updateOrder($data[‘merchantReference’], [
‘status’ => ‘paid’,
‘payment_gateway_txn_id’ => $data[‘systemTransactionId’],
‘paid_amount’ => $data[‘amount’],
‘paid_at’ => $data[‘transactionTime’]
]);

```
// Send confirmation email to customer
sendOrderConfirmationEmail($order);

// Update inventory
updateInventory($order);

// Respond with success
echo json_encode(['status' => '00000']);
```

} else { // Failed
// Update order status
updateOrder($data[‘merchantReference’], [
‘status’ => ‘failed’,
‘failure_reason’ => $data[‘description’]
]);

```
// Respond with success (callback received)
echo json_encode(['status' => '00000']);
```

}

function getOrder($reference) {
// Get order from database
}

function updateOrder($reference, $data) {
// Update order in database
}

function sendOrderConfirmationEmail($order) {
// Send email
}

function updateInventory($order) {
// Update stock
}
?>
```

**Important: Merchant MUST Respond**
```json
{
“status”: “00000”
}
```

If merchant doesn’t respond with `00000`, your gateway will retry callback.

### **Creating a Withdrawal (Customer Payout)**

**Merchant Backend Code:**

```php
 ‘MER-00001’,
‘amount’ => 50,
‘reference’ => ‘WITHDRAW_’ . time(),
‘transactionTime’ => date(‘Y-m-d H:i:s’),
‘paymentMethod’ => ‘BKASH’,
‘recipientName’ => ‘Aminul Islam’,
‘recipientAccount’ => ‘01822444555’, // Customer’s bKash number
‘callbackUrl’ => ‘https://merchant.com/payment/withdrawal-callback’,
‘remarks’ => ‘User withdrawal request’
];

// Generate security token
$secretKey = ‘secret_a1b2c3d4e5f6g7h8i9j0’;

$tokenString =
$data[‘merchantCode’] .
$data[‘amount’] .
$data[‘reference’] .
$data[‘transactionTime’] .
$data[‘paymentMethod’] .
$data[‘recipientAccount’] .
$data[‘callbackUrl’] .
$secretKey;

*data*[′*securityToken*′] = *md*5(tokenString);

// Call withdrawal API
*ch* = *curlinit*(′*https* : //*api*.*yourgateway*.*com*/*v*1/*withdraw*′); *curlsetopt*(ch, CURLOPT_POST, true);
curl_setopt(*ch*, *CURLOPTPOSTFIELDS*, *jsonencode*(data));
curl_setopt(*ch*, *CURLOPTRETURNTRANSFER*, *true*); *curlsetopt*(ch, CURLOPT_HTTPHEADER, [
‘Content-Type: application/json’,
‘X-API-Key: a1b2c3d4-e5f6-g7h8-i9j0-k1l2m3n4o5p6’]);

*response* = *curlexec*(ch);
curl_close($ch);

*result* = *jsondecode*(response, true);

// Check response
if ($result[‘status’] === ‘00000’) {
// Withdrawal request created
// Agent will process it
// You’ll receive callback when complete

```
echo "Withdrawal request submitted. Reference: " . $data['reference'];
```

} else {
echo “Withdrawal failed:” . $result[‘description’];
}
?>
```

**Your Gateway API Response:**
```json
{
“status”: “00000”,
“description”: “Withdrawal request created successfully”,
“systemTransactionId”: “TXN-00456”
}
```

### **Checking Transaction Status**

```php
 ‘MER-00001’,
‘reference’ => $reference,
‘securityToken’ => $securityToken
]);

*response* = *filegetcontents*(url);
*result* = *jsondecode*(response, true);

/
*Response:
{
“status”: “00000”,
“systemTransactionId”: “TXN-00123”,
“reference”: “ORDER_1736957400”,
“currencyCode”: “BDT”,
“amount”: 1000,
“fee”: 15,
“paymentMethod”: “BKASH”,
“senderUsername”: “Customer Name”,
“senderAccount”: “01711222333”,
“transactionTime”: “2025-01-15 10:30:45”
}*
/
?>
```

### **Checking Merchant Balance**

```php
 ‘MER-00001’,
‘securityToken’ => $securityToken
]);

*response* = *filegetcontents*(url);
*result* = *jsondecode*(response, true);

/
*Response:
{
“currentBalance”: 12500.50
}*
/
?>
```

### 3. Merchant Dashboard

**Merchant Portal: `https://portal.yourgateway.com`**

```
┌─────────────────────────────────────────┐
│ Example Store Dashboard │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ Balance: $12,500.50 │
│ On Hold: $250.00 │
│ Available: $12,250.50 │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ [Request Withdrawal] │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ Today’s Summary │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ Deposits: 45 ($4,500) │
│ Withdrawals: 12 ($600) │
│ Charges Paid: $61.50 │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ Recent Transactions │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ TXN-00123 $100 Success 10:30 AM │
│ TXN-00124 $50 Success 10:35 AM │
│ TXN-00125 $200 Pending 10:40 AM │
└─────────────────────────────────────────┘
```

**Merchant Requests Withdrawal:**

```
┌─────────────────────────────────────────┐
│ Request Withdrawal │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ Available Balance: $12,250.50 │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ Withdrawal Amount: │
│ [ 5000__________________ ] │
│ │
│ Charge (1.5%): $75.00 │
│ You’ll Receive: $4,925.00 │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ Bank Details: │
│ Bank: BRAC Bank │
│ Account: 1234567890 │
│ Account Name: Example Store Ltd. │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ Remarks (Optional): │
│ [ Weekly settlement________ ] │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│ [Request Withdrawal] │
└─────────────────────────────────────────┘
```

**After Request Submitted:**
```
✓ Withdrawal request submitted

Status: Pending Admin Approval
Amount: $5,000
You’ll receive: $4,925 (after $75 charge)

Your balance has been put on hold.
You’ll receive a notification once approved.
```

---

## Transaction Flows

### Deposit Flow (Complete)

```
┌─────────────┐
│ Merchant │ Creates deposit via API
│ Website │ POST /v1/deposit
└──────┬──────┘
│
v
┌─────────────┐
│ Gateway │ 1. Validate merchant, IP, security token
│ Backend │ 2. Select random agent (with rules)
└──────┬──────┘ 3. Create transaction (status: PENDING)
│ 4. Hold agent limit temporarily
│ 5. Return redirect URL
v
┌─────────────┐
│ Payment │ Show agent’s wallet number
│ Page │ Customer pays agent
└──────┬──────┘ Customer submits TrxID
│
v
┌─────────────┐
│ Gateway │ Store customer TrxID
│ Backend │ Status: PROCESSING
└──────┬──────┘ Wait for agent SMS or manual approval
│
│ ┌──────────────────┐
│ │ Agent App │ Receives SMS
│◄───────┤ SMS Reader │ Parses TrxID
│ └──────────────────┘ Sends callback
│
v
┌─────────────┐
│ Gateway │ 1. Match TrxIDs
│ Backend │ 2. Update transaction (SUCCESS)
└──────┬──────┘ 3. Deduct agent balance
│ 4. Add merchant balance
│ 5. Send merchant callback
│ 6. Notify agent
v
┌─────────────┐
│ Merchant │ Receives callback
│ Callback │ Updates order status
└─────────────┘ Completes customer order
```

### Withdrawal Flow (Complete)

```
┌─────────────┐
│ Merchant │ Creates withdrawal via API
│ Website │ POST /v1/withdraw
└──────┬──────┘
│
v
┌─────────────┐
│ Gateway │ 1. Validate merchant, check balance
│ Backend │ 2. Deduct merchant balance immediately
└──────┬──────┘ 3. Select random agent
│ 4. Create transaction (PENDING)
│ 5. Notify agent
v
┌─────────────┐
│ Agent App │ Agent sees withdrawal request
│ │ Agent pays customer manually
└──────┬──────┘ Agent submits TrxID & proof
│
v
┌─────────────┐
│ Gateway │ 1. Update transaction (SUCCESS)
│ Backend │ 2. Add amount to agent balance
└──────┬──────┘ 3. Add commission to agent balance
│ 4. Send merchant callback
│ 5. Notify agent
v
┌─────────────┐
│ Merchant │ Receives callback
│ Callback │ Updates customer’s wallet
└─────────────┘ Notifies customer
```

---

## Security Features

### 1. MD5 Security Tokens

**Every API Request Must Include Token:**

```javascript
// Deposit Request
const tokenString = [
merchantCode,
amount,
reference,
transactionTime,
callbackUrl,
returnUrl,
secretKey].join(’’);

const securityToken = md5(tokenString);
```

**Backend Validation:**

```javascript
function validateSecurityToken(data, secretKey) {
const expectedToken = md5(/* same fields */ + secretKey);

if (data.securityToken !== expectedToken) {
throw new Error(‘Invalid security token’);
}
}
```

### 2. IP Whitelisting

**Merchant Configuration:**
```javascript
// Only these IPs can call merchant’s API
merchant.whitelistedIps = [
“123.45.67.89”,
“123.45.67.90”];

// Middleware
async function validateIpAddress(req, res, next) {
const merchant = await getMerchant(req.body.merchantCode);
const clientIp = req.ip;

if (!merchant.whitelistedIps.includes(clientIp)) {
return res.status(403).json({
status: ‘10003’,
description: ‘IP address not whitelisted’
});
}

next();
}
```

### 3. Single Device Login

**Agent/Merchant Login:**

```javascript
async function handleLogin(userId, userType, deviceId) {
// Check if device already approved
const existingSession = await DeviceSession.findOne({
userId,
userType,
deviceId,
status: ‘APPROVED’,
isActive: true
});

if (existingSession) {
// Deactivate all other devices
await DeviceSession.updateMany(
{
userId,
userType,
id: { ne: existingSession.id }
},
{ isActive: false }
);

```
// Activate this device
existingSession.lastActivityAt = new Date();
await existingSession.save();

return { token: existingSession.token };
```

}

// New device - requires admin approval
const newSession = await DeviceSession.create({
userId,
userType,
deviceId,
status: ‘PENDING’,
isActive: false,
token: generateToken()
});

// Notify admin
await notifyAdminForDeviceApproval(userId, userType, deviceId);

throw new Error(‘Device approval pending’);
}
```

### 4. 2FA (Google Authenticator)

**Setup 2FA:**

```javascript
import speakeasy from ‘speakeasy’;
import QRCode from ‘qrcode’;

async function setup2FA(userId) {
const secret = speakeasy.generateSecret({
name: ‘Payment Gateway’,
issuer: ‘Your Company’
});

// Save secret
await User.update(userId, {
twoFactorSecret: secret.base32,
twoFactorEnabled: false // Not enabled until verified
});

// Generate QR code
const qrCode = await QRCode.toDataURL(secret.otpauth_url);

return {
secret: secret.base32,
qrCode: qrCode
};
}
```

**Verify 2FA:**

```javascript
async function verify2FA(userId, token) {
const user = await User.findById(userId);

const verified = speakeasy.totp.verify({
secret: user.twoFactorSecret,
encoding: ‘base32’,
token: token,
window: 2 // Allow 2 time steps tolerance
});

if (verified) {
user.twoFactorEnabled = true;
await user.save();
}

return verified;
}
```

**Login with 2FA:**

```javascript
async function login(email, password, token2FA) {
const user = await User.findOne({ email });

if (!user) throw new Error(‘Invalid credentials’);

const passwordValid = await bcrypt.compare(password, user.password);
if (!passwordValid) throw new Error(‘Invalid credentials’);

if (user.twoFactorEnabled) {
const verified = speakeasy.totp.verify({
secret: user.twoFactorSecret,
encoding: ‘base32’,
token: token2FA
});

```
if (!verified) throw new Error('Invalid 2FA token');
```

}

return generateJWT(user);
}
```

### 5. Agent Selection Algorithm

**Smart Agent Selection:**

```javascript
async function selectAgent(gatewayType, amount) {
// Filter criteria
const agents = await Agent.findMany({
where: {
status: ‘ACTIVE’,
isActive: true,
isAppOnline: true, // App must be running
lastAppHeartbeat: {
gte: new Date(Date.now() - 2 * 60 * 1000) // Active in last 2 minutes
},
balance: { gte: amount }, // Has sufficient balance
limitBalance: { gte: amount },
agentGateways: {
some: {
gatewayType: gatewayType,
isActive: true
}
}
},
include: {
transactions: {
where: {
status: { in: [‘PENDING’, ‘PROCESSING’] }
}
},
agentGateways: {
where: { gatewayType: gatewayType }
}
}
});

// Filter out agents with pending transactions (can’t show number)
const availableAgents = agents.filter(agent => {
return agent.transactions.length === 0; // No pending
});

if (availableAgents.length === 0) {
throw new Error(‘No available agents at the moment’);
}

// Weighted random selection based on:
// 1. Success rate (higher = more likely)
// 2. Available balance (more = more likely)
// 3. Recent activity (less recent = more likely to balance load)

const weights = availableAgents.map(agent => {
const successWeight = agent.successRate; // 0-100
const balanceWeight = Math.min(agent.balance / 10000, 100); // Cap at 100
const activityWeight = 100 - (agent.totalTransactions % 100); // Balance load

```
return (successWeight + balanceWeight + activityWeight) / 3;
```

});

const selectedAgent = weightedRandom(availableAgents, weights);

// Hold the amount temporarily
selectedAgent.holdBalance += amount;
await selectedAgent.save();

return selectedAgent;
}

function weightedRandom(items, weights) {
const totalWeight = weights.reduce((a, b) => a + b, 0);
let random = Math.random() * totalWeight;

for (let i = 0; i < items.length; i++) {
random -= weights[i];
if (random <= 0) return items[i];
}

return items[items.length - 1];
}
```

### 6. Rate Limiting

```javascript
import rateLimit from ‘express-rate-limit’;

// API rate limiting
const apiLimiter = rateLimit({
windowMs: 1 * 60 * 1000, // 1 minute
max: 60, // 60 requests per minute
keyGenerator: (req) => {
return req.body.merchantCode || req.ip;
},
handler: (req, res) => {
res.status(429).json({
status: ‘10007’,
description: ‘Too many requests. Please try again later.’
});
}
});

app.use(‘/v1/’, apiLimiter);
```

### 7. Fraud Detection

```javascript
// Background job: Detect suspicious patterns
async function detectFraud() {
// 1. Same TrxID used multiple times
const duplicateTrxIds = await Transaction.groupBy({
by: [‘agentReceivedTrxId’],
having: {
agentReceivedTrxId: { _count: { gt: 1 } }
}
});

for (const duplicate of duplicateTrxIds) {
await SecurityAlert.create({
alertType: ‘DUPLICATE_TRX_ID’,
severity: ‘HIGH’,
subjectType: ‘TRANSACTION’,
subjectId: duplicate.agentReceivedTrxId,
title: ‘Duplicate TrxID Detected’,
description: `TrxID ${duplicate.agentReceivedTrxId} used in multiple transactions`
});
}

// 2. Agent with high failure rate
const suspiciousAgents = await Agent.findMany({
where: {
successRate: { lt: 80 },
totalTransactions: { gte: 10 }
}
});

for (const agent of suspiciousAgents) {
await SecurityAlert.create({
alertType: ‘LOW_SUCCESS_RATE’,
severity: ‘MEDIUM’,
subjectType: ‘AGENT’,
subjectId: agent.id,
title: ‘Agent Low Success Rate’,
description: `Agent ${agent.agentCode} has success rate of ${agent.successRate}%`
});
}

// 3. Rapid transactions from same IP
const rapidTransactions = await Transaction.groupBy({
by: [‘customerIp’],
where: {
createdAt: { gte: new Date(Date.now() - 5 * 60 * 1000) } // Last 5 minutes
},
having: {
customerIp: { _count: { gt: 10 } }
}
});

for (const rapid of rapidTransactions) {
await SecurityAlert.create({
alertType: ‘RAPID_TRANSACTIONS’,
severity: ‘HIGH’,
subjectType: ‘IP_ADDRESS’,
subjectId: rapid.customerIp,
title: ‘Suspicious Activity Detected’,
description: `IP ${rapid.customerIp} made ${rapid._count} transactions in 5 minutes`
});
}
}

// Run every 5 minutes
setInterval(detectFraud, 5 * 60 * 1000);
```

---

## Technical Implementation

### Technology Stack

**Backend:**
- Node.js + NestJS (or Express.js)
- TypeScript
- Prisma ORM
- PostgreSQL
- Redis (caching + queues)
- Bull/BullMQ (background jobs)

**Frontend:**
- React + Next.js
- TypeScript
- TailwindCSS
- shadcn/ui components

**Mobile App (Agent):**
- React Native
- TypeScript
- SMS reader library
- Push notifications (FCM)

**Infrastructure:**
- Docker + Docker Compose
- Nginx (reverse proxy)
- Let’s Encrypt (SSL)

### Key Background Jobs

```javascript
// 1. Transaction Timeout Checker
async function checkExpiredTransactions() {
const expired = await Transaction.updateMany({
where: {
status: { in: [‘PENDING’, ‘PROCESSING’] },
expiresAt: { lte: new Date() }
},
data: {
status: ‘EXPIRED’
}
});

// Release held balances
for (const txn of expired) {
await releaseAgentHold(txn.agentId, txn.amount);
}
}

// 2. Auto-Approval Job
async function checkAutoApproval() {
const autoApproveTime = new Date(Date.now() - 5 * 60 * 1000);

const toApprove = await Transaction.findMany({
where: {
status: ‘PROCESSING’,
customerSubmittedAt: { lte: autoApproveTime },
agentReceivedTrxId: null
}
});

for (const txn of toApprove) {
await completeTransaction(txn);
txn.isAutoApproved = true;
await txn.save();
}
}

// 3. Callback Retry Job
async function retryFailedCallbacks() {
const maxAttempts = 5;

const failed = await Transaction.findMany({
where: {
callbackStatus: ‘FAILED’,
callbackAttempts: { lt: maxAttempts },
status: ‘SUCCESS’
}
});

for (const txn of failed) {
await sendMerchantCallback(txn);
}
}

// 4. Daily Report Generator
async function generateDailyReport() {
const today = startOfDay(new Date());

const stats = await Transaction.aggregate({
where: {
createdAt: { gte: today }
},
_count: true,
_sum: {
amount: true,
merchantCharge: true,
agentCommission: true
}
});

await DailyReport.create({
reportDate: today,
totalTransactions: stats._count,
totalDepositAmount: stats._sum.amount,
depositChargesCollected: stats._sum.merchantCharge,
totalAgentCommissions: stats._sum.agentCommission
});
}

// Schedule jobs
const queue = new Bull(‘payment-gateway’);

queue.add(‘check-expired’, {}, { repeat: { every: 30000 } }); // 30s
queue.add(‘auto-approve’, {}, { repeat: { every: 30000 } });
queue.add(‘retry-callbacks’, {}, { repeat: { every: 60000 } }); // 1min
queue.add(‘daily-report’, {}, { repeat: { cron: ’0 0 * * *’ } }); // Midnight
```

### API Endpoints Summary

```
POST /v1/deposit - Create deposit
POST /v1/withdraw - Create withdrawal
GET /v1/get-deposit-info - Get single deposit
GET /v1/get-multiple-deposits-info - Get multiple deposits
POST /v1/flag-deposit - Flag deposits as processed
GET /v1/get-withdrawal-info - Get withdrawal info
GET /v1/get-merchant-balance - Get merchant balance

POST /api/admin/login - Admin login
POST /api/admin/merchants - Create merchant
POST /api/admin/agents - Create agent
POST /api/admin/topups/:id/approve - Approve topup
POST /api/admin/devices/:id/approve - Approve device

POST /api/agent/login - Agent login
POST /api/agent/topup - Request topup
POST /api/agent/gateways - Add gateway
POST /api/agent/sms-callback - SMS received callback
POST /api/agent/manual-approve - Manual approve payment
POST /api/agent/confirm-withdrawal - Confirm withdrawal paid

POST /api/payment/submit-trxid - Customer submits TrxID
```

---

## Conclusion

This payment gateway system provides:

✅ **For Merchants:**
- Easy API integration
- Secure payment processing
- Real-time callbacks
- Balance management
- Automated settlements

✅ **For Agents:**
- Flexible topup system with bonuses
- Multiple payment gateway support
- Automated SMS processing
- Manual control when needed
- Commission earnings

✅ **For Admins:**
- Complete control over system
- Flexible charge configuration
- Role-based permissions
- Comprehensive reporting
- Fraud detection

✅ **Security:**
- MD5 security tokens
- IP whitelisting
- 2FA authentication
- Single device login
- Rate limiting
- Audit logging

✅ **Performance:**
- Fast transaction matching (both scenarios handled)
- Auto-approval fallback
- Weighted agent selection
- Background job processing
- Redis caching

The system is designed to be **secure, fast, and scalable** while providing a smooth experience for all user types.
