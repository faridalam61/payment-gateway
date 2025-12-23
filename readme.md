# Payment Gateway System Documentation

## Overview

A comprehensive payment gateway system that enables merchants to accept online payments through multiple mobile wallets (bKash, Nagad, Rocket, Upay) via an agent network.

### System Architecture

```
Customer → Merchant Website → Payment Gateway API → Agent (Mobile Wallet) → Customer
```

### Core Concepts

**Agent Balance System:**
- Agents deposit money and receive a limit with a commission bonus
- Example: $500 deposit + 2% bonus = $510 available limit
- **DEPOSIT (Cash In)**: When customers pay agents, the agent's balance decreases
- **WITHDRAWAL (Cash Out)**: When agents pay customers, the agent's balance increases

---

## Table of Contents

1. [User Roles & Permissions](#user-roles--permissions)
2. [Admin Guide](#admin-guide)
3. [Agent Guide](#agent-guide)
4. [Merchant Guide](#merchant-guide)
5. [Transaction Flows](#transaction-flows)
6. [Security Features](#security-features)
7. [Technical Reference](#technical-reference)

---

## User Roles & Permissions

### Super Admin
**Full system control with all permissions**

**Can do:**
- Manage all staff, admins, and editors
- Create and manage merchants and agents
- Configure global and individual charges/commissions
- Approve topup and withdrawal requests
- Assign and revoke permissions to staff
- Access all reports, analytics, and audit logs
- Approve device login requests
- Manage system settings

### Admin
**System management without admin creation rights**

**Can do:**
- Manage merchants and agents
- Configure charges and commissions
- Approve topup and withdrawal requests
- View all reports and settings
- Approve device login requests

**Cannot do:**
- Create other admin accounts

### Staff
**Limited operations with configurable permissions**

**Can do (if granted by Super Admin):**
- Create merchant accounts
- Create agent accounts
- Approve topup requests
- View transactions and basic reports

**Cannot do:**
- Modify charges or commissions
- Approve withdrawal requests
- Delete records

### Editor
**View-only access with basic editing capabilities**

**Can do:**
- View all system data (read-only mode)
- Edit basic merchant and agent information
- View reports and analytics

**Cannot do:**
- Approve any requests
- Configure charges
- Delete any records

### Agent
**Payment processing handler**

**Can do:**
- Receive and process deposit payments from customers
- Process withdrawal payments to customers
- Request topups for their account
- Manage multiple payment gateway accounts
- View earnings, commissions, and transaction history
- Manually approve or reject payments when needed

### Merchant
**API integration for payment processing**

**Can do:**
- Create deposit requests via API (customer payments)
- Create withdrawal requests via API (customer payouts)
- Query transaction status
- View account balance and transaction history
- Request withdrawals of funds

---

## Admin Guide

### Initial Setup

#### Step 1: System Installation
After installing the system, run database migrations and seed the initial Super Admin account.

#### Step 2: Configure Global Settings

**Merchant Charges Configuration:**
- **Deposit Charge:** Set default percentage (e.g., 1.5%) or fixed amount
- **Withdrawal Charge:** Set default percentage (e.g., 1.5%) or fixed amount
- **Min/Max Charge:** Set minimum and maximum charge limits

**Agent Commissions Configuration:**
- **Deposit Commission:** Set percentage agents earn when receiving payments (e.g., 0.5%)
- **Topup Bonus:** Set percentage bonus agents receive on deposits (e.g., 2%)
- **Withdrawal Commission:** Set percentage agents earn when paying customers (e.g., 0.5%)

**Transaction Limits:**
- **Min Transaction:** Set minimum transaction amount (e.g., $10)
- **Max Transaction:** Set maximum transaction amount (e.g., $50,000)
- **Transaction Timeout:** Set how long payment pages remain active (e.g., 15 minutes)
- **Auto-Approval Timeout:** Set when system auto-approves if agent doesn't respond (e.g., 5 minutes)

---

### Creating Staff Accounts

Navigate to: **Admin Panel → Staff Management → Create Staff**

**Required Information:**
- Full name and email address
- Role selection (Admin, Staff, or Editor)
- Permission assignment via checkboxes
- 2FA requirement (enabled/disabled)

**Available Permissions for Staff:**
- Manage merchants
- Manage agents
- Manage charges
- Approve topup requests
- Approve withdrawal requests
- View reports and analytics

**After Creation:**
Staff receives an email with login credentials and must set up 2FA on first login.

---

### Creating Merchant Accounts

Navigate to: **Admin Panel → Merchants → Create Merchant**

#### Basic Information
- Business name, email, phone number
- Password (can be auto-generated)

#### API Credentials (Auto-Generated)
- **Merchant Code:** Unique identifier (e.g., MER-00001)
- **API Key:** UUID for API authentication
- **Secret Key:** Used for generating security tokens

#### Security Settings
- **Whitelisted IP Addresses:** Only these IPs can call the merchant's API
- **Callback URL:** Where to send payment notifications
- **Return URL:** Where to redirect customers after payment

#### Charge Settings (Optional - Overrides Global)
You can set custom charges for specific merchants that differ from global settings:
- Custom deposit charge percentage
- Custom withdrawal charge percentage

This is useful for VIP merchants who may get lower rates.

#### Transaction Limits
- Daily deposit limit
- Daily withdrawal limit
- Minimum withdrawal amount

#### Payment Gateway Settings
- Select which gateways merchant can use (bKash, Nagad, Rocket, Upay)
- Enable/disable auto-approval for withdrawals
- Set account status (Active/Inactive)

**After Creation:**
Merchant receives an email containing:
- Login credentials for the merchant portal
- API Key and Secret Key
- Integration documentation
- Test API endpoint URLs

---

### Creating Agent Accounts

Navigate to: **Admin Panel → Agents → Create Agent**

#### Basic Information
- Full name, email, phone number
- Agent Code (auto-generated, e.g., AGT-00001)
- Password (can be auto-generated)

#### Commission Settings (Optional - Overrides Global)
Set custom commission rates for specific agents:
- Custom deposit commission percentage
- Custom topup bonus percentage
- Custom withdrawal commission percentage

This allows you to offer different rates to high-performing agents.

#### Operational Limits
- **Daily Transaction Limit:** Maximum transaction volume per day
- **Max Concurrent Transactions:** How many transactions can be pending at once (typically 1)

#### Security Settings
- Allowed IP addresses (optional restriction)

#### Initial Status
- Status starts as "PENDING"
- Changes to "ACTIVE" after first approved topup

**After Creation:**
Agent receives an email containing:
- Login credentials
- Mobile app download link
- Instructions for adding payment gateways
- Topup instructions

---

### Approving Agent Topup Requests

Navigate to: **Admin Panel → Topup Requests → Pending**

#### Review Process

**Request Details Include:**
- Agent information (name, code)
- Requested topup amount
- Bonus percentage and amount (calculated automatically)
- Total limit to be added (deposit + bonus)
- Payment method used (bank transfer, etc.)
- Payment proof (screenshot or receipt)
- Agent's remarks or notes
- Request timestamp

#### Approval Calculation Example
If an agent requests $500 topup with 2% bonus:
- Deposit amount: $500
- Bonus (2%): $10
- Total limit added: $510

#### When You Approve:
1. Agent's balance increases by $510
2. Agent's available limit increases by $510
3. Record created in agent balance history
4. Agent receives notification with new balance
5. If agent status was "PENDING", it changes to "ACTIVE"

#### If You Reject:
1. Agent receives notification with rejection reason
2. No changes to agent balance
3. Agent can submit a new request

---

### Managing Charges & Commissions

#### Global Charges
Navigate to: **Admin Panel → Settings → Charges → Global**

**Merchant Charges:**
Configure how merchants are charged:
- Choose charge type: Percentage, Fixed Amount, or Both
- Set percentage value (e.g., 1.5%)
- Set fixed amount if applicable
- Set minimum and maximum charge limits

These charges apply to all deposits and withdrawals unless overridden.

**Agent Commissions:**
Configure how agents earn:
- Deposit commission: What agents earn when receiving customer payments
- Topup bonus: Extra percentage added when agents deposit funds
- Withdrawal commission: What agents earn when paying customers

#### Individual Merchant Charges
Navigate to: **Admin Panel → Merchants → Edit Merchant → Charges Tab**

Enable the "Override Global Charges" option to set custom rates for specific merchants.

**Example Use Case:**
A high-volume merchant may receive preferential rates:
- Global rate: 1.5%
- VIP merchant rate: 1.2%

#### Individual Agent Commissions
Navigate to: **Admin Panel → Agents → Edit Agent → Commission Tab**

Enable the "Override Global Commissions" option to set custom commission rates.

**Example Use Case:**
A top-performing agent may receive higher commissions:
- Global commission: 0.5%
- Top agent commission: 0.6%

---

### Approving Merchant Withdrawal Requests

Navigate to: **Admin Panel → Withdrawal Requests → Pending**

#### Review Process

**Request Details Include:**
- Merchant information and current balance
- Withdrawal amount requested
- Charge amount (calculated automatically)
- Net amount to transfer (withdrawal - charge)
- Bank details (bank name, account number, account name)
- Merchant's remarks
- Request timestamp

#### Approval Process:

**Step 1:** Review the request and verify merchant has sufficient balance

**Step 2:** Click "Approve" - status changes to "PROCESSING"

**Step 3:** Manually transfer the net amount to merchant's bank account

**Step 4:** Upload payment proof (bank transfer receipt, screenshot)

**Step 5:** Click "Mark as Completed"

**System Actions:**
1. Deducts full amount from merchant balance
2. Records charge separately
3. Updates withdrawal status to "COMPLETED"
4. Sends email to merchant with payment proof
5. Records entry in merchant balance history

#### If You Reject:
1. Merchant receives notification with reason
2. No changes to merchant balance
3. Merchant can submit a new request

---

### Approving Agent Device Login

Navigate to: **Admin Panel → Device Approvals → Pending**

#### Review Process

**Login Request Details:**
- Agent information
- Device ID and device name
- Operating system and app version
- IP address
- Request timestamp

#### Security Feature
Agents can only have ONE active device at a time for security.

#### When You Approve:
1. All other devices for this agent are automatically deactivated
2. This device is marked as approved and active
3. Agent can now use the mobile app
4. Agent receives push notification confirming approval

#### If You Reject:
Agent cannot use the app and must request approval again or contact support.

---

### Admin Dashboard

Navigate to: **Admin Panel → Dashboard**

#### Today's Statistics
View real-time metrics:
- Total transactions processed today
- Successful, pending, and failed transaction counts
- Success rate percentage
- Total revenue (charges collected)
- Total transaction volume

#### Pending Actions
See items requiring attention:
- Number of agent topup requests awaiting approval
- Number of merchant withdrawal requests awaiting approval
- Number of device login approvals needed
- Number of transactions flagged for manual review

#### System Overview
View high-level metrics:
- Number of active agents and their total available limit
- Number of active merchants and their total balance
- System health indicators

---

## Agent Guide

### Getting Started

#### Step 1: Receive Credentials
Admin creates your account and you receive an email with:
- Login credentials
- Mobile app download link
- Setup instructions

#### Step 2: Download & Install App
Download the agent mobile app from the provided link and install it on your device.

#### Step 3: Login & Request Device Approval
Enter your credentials. Your device login request is automatically sent to admin for approval.

#### Step 4: Wait for Approval
You'll receive a notification when admin approves your device. This typically happens within a few hours.

#### Step 5: Set Up 2FA
After device approval, set up two-factor authentication using Google Authenticator for added security.

---

### Requesting a Topup

Navigate to: **Agent App → Topup → Request New Topup**

#### Process:

**Step 1:** Enter the amount you want to deposit

**Step 2:** View the calculation
- Your deposit amount
- Bonus percentage (e.g., 2%)
- Bonus amount (calculated)
- Total limit you'll receive

**Step 3:** Make the payment
Follow the displayed bank transfer instructions to send money to the company account.

**Step 4:** Upload proof
Take a screenshot of your bank transfer confirmation and upload it.

**Step 5:** Add remarks (optional)
Provide any additional information (e.g., "Transferred from my BRAC account ending 4567")

**Step 6:** Submit request
Your request is sent to admin for approval.

#### After Approval:
- You receive a notification
- Your balance is updated with deposit + bonus
- Your status changes to "ACTIVE" if this was your first topup
- You can now start receiving payments

**Example:**
If you deposit $500 with a 2% bonus:
- Your balance increases by $510 ($500 + $10 bonus)
- You can now handle transactions up to $510

---

### Adding Payment Gateways

Navigate to: **Agent App → Payment Gateways → Add Gateway**

#### Setup Process:

**Step 1:** Select gateway type
Choose from bKash, Nagad, Rocket, or Upay.

**Step 2:** Enter account details
- Your mobile wallet account number
- Account holder name
- Optional daily limit for this gateway

**Step 3:** Set priority
If you have multiple gateways, set priority (1 = highest priority).

**Step 4:** Set status
Mark as Active or Inactive.

#### Multiple Gateways:
You can add multiple gateways:
- bKash - 01711222333 (Active, Priority 1)
- Nagad - 01811222333 (Active, Priority 2)
- Rocket - 01911222333 (Inactive)

The system will show customers the gateway based on their selected payment method and your priority settings.

---

### Receiving Deposit Payments (Cash In)

#### How It Works

**The Complete Flow:**

1. A merchant creates a deposit request in their system
2. The payment gateway selects you (the agent) randomly
3. The customer is redirected to a payment page
4. The payment page displays your mobile wallet number
5. The customer sends money to your number using their wallet
6. The customer receives a TrxID from their wallet
7. Two things can happen in any order:
   - Your app automatically reads the SMS from your wallet
   - The customer submits the TrxID on the payment page
8. The system matches both pieces of information
9. Your balance is deducted
10. The merchant receives payment confirmation

#### Scenario A: Your SMS Arrives First

**What Happens:**
1. Customer pays your wallet number
2. Your mobile wallet (bKash/Nagad/etc.) sends you an SMS confirmation
3. Your app automatically reads and parses the SMS
4. Your app immediately sends the details to the backend
5. The system stores this information and waits for the customer to submit their TrxID
6. When the customer submits their TrxID, the system instantly matches it with your SMS
7. Transaction is completed immediately

**Example SMS:**
"You have received 100.00 BDT from 01811555666. TrxID: ABC123XYZ. Balance: 1,500 BDT."

The app extracts:
- Amount: 100 BDT
- Sender: 01811555666
- TrxID: ABC123XYZ

#### Scenario B: Customer Submits TrxID First

**What Happens:**
1. Customer pays your wallet and immediately submits their TrxID on the payment page
2. The system stores the customer's TrxID and waits for your SMS
3. Your mobile wallet sends the confirmation SMS (typically 5-30 seconds later)
4. Your app reads the SMS and sends it to the backend
5. The system instantly matches the SMS with the customer's submission
6. Transaction is completed immediately

#### Transaction Completion

**When both pieces match:**
1. Transaction status changes to "SUCCESS"
2. Your balance is reduced by the payment amount
3. You receive commission for processing the payment
4. Merchant's balance is increased (minus their charge)
5. Merchant receives instant callback notification
6. You receive push notification confirming the payment

**Balance Calculation Example:**
- Customer paid: $100
- Your balance before: $510
- Amount deducted: $100
- Your commission (0.5%): $0.50
- Your new balance: $410.50
- Merchant receives: $98.50 (after 1.5% charge)
- System earns: $1.00 ($1.50 charge - $0.50 commission)

---

### Auto-Approval System (5-Minute Safety Net)

**Problem It Solves:**
If your app crashes, loses internet, or fails to read the SMS, customers shouldn't wait forever.

**How It Works:**
- Customer pays and submits their TrxID
- System waits for your SMS confirmation
- If 5 minutes pass without your SMS:
  - System automatically approves the transaction
  - Merchant receives payment confirmation
  - Admin receives an alert to review the transaction
  - Transaction is flagged for manual review

**Why This Is Safe:**
- Customer has already paid (they have TrxID proof)
- Merchant can fulfill the order
- Admin can verify the transaction later
- Prevents customer complaints and delays

**Your Responsibility:**
Keep your app running and ensure it has permission to read SMS automatically.

---

### Manual Approval

Navigate to: **Agent App → Pending Transactions**

If automatic SMS reading fails, you can manually verify payments.

#### Process:

**Step 1:** View pending transaction details
- Transaction ID
- Amount customer should pay
- Your wallet number they paid to
- TrxID customer submitted
- Customer's phone number
- Time elapsed

**Step 2:** Check your mobile wallet
Open your bKash/Nagad app and verify the transaction manually.

**Step 3:** Confirm receipt
- Enter the TrxID from your SMS
- Optionally upload a screenshot
- Confirm you received the payment

**Step 4:** Submit approval
The transaction completes using the same process as automatic approval.

---

### Processing Withdrawal Payments (Cash Out)

#### How It Works

**The Complete Flow:**

1. A merchant creates a withdrawal request (customer wants money)
2. The system selects you (the agent) to handle the payout
3. You receive a notification in your app
4. You open the withdrawal request and see customer details
5. You manually pay the customer using your mobile wallet
6. You get a TrxID from your wallet
7. You enter the TrxID in the app
8. Your balance increases (you get the money back plus commission)
9. Merchant receives confirmation that customer was paid

#### Processing Steps:

**Step 1:** Receive notification
"New Withdrawal Request: Pay $50 to 01822444555"

**Step 2:** Open transaction in app
View details:
- Amount to pay: $50
- Recipient number: 01822444555
- Recipient name: Aminul Islam
- Gateway to use: bKash (your account 01711222333)

**Step 3:** Pay the customer
- Open your bKash app
- Send $50 to 01822444555
- Note the TrxID from bKash

**Step 4:** Confirm in agent app
- Enter the TrxID from your bKash confirmation
- Optionally upload screenshot
- Click "Confirm Payment"

**Step 5:** Transaction completion
- Your balance increases by $50 (money returned)
- You earn commission (e.g., 0.5% = $0.25)
- Merchant receives callback confirming customer was paid
- You receive confirmation notification

**Balance Calculation Example:**
- You paid customer: $50
- Your balance before: $410.50
- Money returned: $50
- Commission earned: $0.25
- Your new balance: $460.75

---

### Agent Dashboard

Navigate to: **Agent App → Home**

#### Account Overview
- Current available balance
- Balance on hold (pending transactions)
- Account status (Active/Inactive)
- App version

#### Today's Summary
- Total number of transactions
- Deposits received (count and amount)
- Withdrawals paid (count and amount)
- Total commission earned today

#### Pending Actions
Quick access to:
- Deposits awaiting your confirmation
- Withdrawal requests assigned to you

#### Quick Actions
- Request topup
- Manage payment gateways
- View transaction history
- View earnings reports

---

## Merchant Guide

### Getting Started

#### Account Creation
Admin creates your merchant account and you receive an email containing:
- Merchant Code (e.g., MER-00001)
- Dashboard login credentials
- API Key (UUID format)
- Secret Key (for security token generation)
- API documentation URL
- Staging and production API endpoint URLs

#### First Steps:
1. Login to merchant portal
2. Change your password
3. Review your account settings
4. Set up your callback URL
5. Test the integration using staging environment
6. Go live with production

---

### Integration Overview

#### API Authentication
Every API request requires three security components:
1. **API Key:** Included in request headers for authentication
2. **Merchant Code:** Your unique identifier
3. **Security Token:** MD5 hash generated from request parameters + secret key

#### Security Token Generation
Concatenate specific fields in exact order, append your secret key, and generate MD5 hash.

For deposits: `merchantCode + amount + reference + transactionTime + callbackUrl + returnUrl + secretKey`

For withdrawals: `merchantCode + amount + reference + transactionTime + paymentMethod + recipientAccount + callbackUrl + secretKey`

---

### Creating Deposits (Customer Payments)

#### When to Use:
When a customer on your website wants to make a payment.

#### Required Information:
- **Merchant Code:** Your unique identifier
- **Amount:** Payment amount in your currency
- **Reference:** Your unique order/invoice ID
- **Customer Details:** Name, phone, IP address
- **Transaction Time:** Current timestamp
- **Payment Method:** Customer's chosen gateway (bKash/Nagad/Rocket/Upay)
- **Callback URL:** Where to send payment confirmation
- **Return URL:** Where to redirect customer after payment
- **Security Token:** MD5 hash for verification

#### Process:

**Step 1:** Customer initiates payment on your website

**Step 2:** Your backend calls payment gateway API with deposit request

**Step 3:** Payment gateway validates request and responds with:
- Success status
- System transaction ID
- Redirect URL for customer

**Step 4:** Redirect customer to payment page

**Step 5:** Customer sees agent's wallet number and payment instructions

**Step 6:** Customer pays agent and submits TrxID

**Step 7:** System processes payment (see Agent Guide for details)

**Step 8:** Your callback URL receives payment confirmation

**Step 9:** You update order status and complete customer's purchase

---

### Payment Page (Customer Experience)

After redirecting to the payment gateway, customers see:

#### Page Contents:
- Merchant name and order reference
- Payment amount
- Selected payment method
- Agent's mobile wallet number to send money to
- Clear instructions on how to complete payment
- Input field for TrxID
- Optional input for sender number
- Timer showing remaining time (typically 15 minutes)
- SSL security indicator

#### Customer Steps:
1. Open their mobile wallet app
2. Send money to the displayed number
3. Get TrxID from wallet confirmation
4. Return to payment page
5. Enter TrxID
6. Submit and wait for confirmation
7. Automatically redirected to your return URL

---

### Receiving Callbacks

#### What Are Callbacks:
After successful payment, the gateway sends a POST request to your callback URL with transaction details.

#### Callback Data Includes:
- Status code (00000 for success)
- Your order reference
- System transaction ID
- Currency code
- Amount paid
- Fee charged
- Payment method used
- Customer details
- Transaction timestamp
- Security token for verification

#### Your Responsibilities:

**Step 1:** Verify security token
Generate expected token and compare with received token to prevent fraud.

**Step 2:** Check for duplicate processing
Verify you haven't already processed this reference to avoid double-crediting.

**Step 3:** Process based on status
- **Success (00000):** Update order to paid, send confirmation email, update inventory
- **Failed:** Update order to failed, notify customer

**Step 4:** Respond with success code
Send back status "00000" to acknowledge receipt.

**Important:** If you don't respond with success code, the gateway will retry the callback multiple times.

---

### Creating Withdrawals (Customer Payouts)

#### When to Use:
When customers want to withdraw funds from their account on your platform.

#### Required Information:
- **Merchant Code:** Your unique identifier
- **Amount:** Withdrawal amount
- **Reference:** Your unique withdrawal request ID
- **Transaction Time:** Current timestamp
- **Payment Method:** Customer's chosen gateway
- **Recipient Account:** Customer's mobile wallet number
- **Recipient Name:** Customer's name
- **Callback URL:** Where to send completion confirmation
- **Remarks:** Optional notes
- **Security Token:** MD5 hash for verification

#### Process:

**Step 1:** Customer requests withdrawal on your platform

**Step 2:** Your backend calls payment gateway API with withdrawal request

**Step 3:** Payment gateway validates and responds with:
- Success status
- System transaction ID

**Step 4:** System selects agent and sends notification

**Step 5:** Agent pays customer manually (see Agent Guide)

**Step 6:** Agent confirms payment with TrxID

**Step 7:** Your callback URL receives completion confirmation

**Step 8:** You update customer's balance and send confirmation

#### Important Notes:
- Merchant balance is deducted immediately when request is created
- Agent processes payment within minutes
- You receive callback when customer has been paid
- Transaction cannot be cancelled after creation

---

### Checking Transaction Status

#### When to Use:
To query the current status of a specific transaction using your reference ID.

#### Required Information:
- Merchant Code
- Your order reference
- Security token

#### Response Includes:
- Transaction status
- System transaction ID
- Amount and fees
- Payment method
- Customer details
- Transaction timestamp
- All relevant transaction data

#### Use Cases:
- Customer asking about payment status
- Order fulfillment verification
- Reconciliation processes
- Debugging payment issues

---

### Checking Account Balance

#### When to Use:
To query your current merchant balance programmatically.

#### Required Information:
- Merchant Code
- Security token

#### Response Includes:
- Current available balance

#### Use Cases:
- Display balance in merchant dashboard
- Automated withdrawal triggers
- Balance monitoring alerts
- Financial reporting

---

### Merchant Dashboard

Login to: **Merchant Portal**

#### Balance Overview
- Current available balance
- Balance on hold (processing transactions)
- Available for withdrawal

#### Quick Actions
- Request withdrawal
- View transaction history
- Download reports
- Manage settings

#### Today's Summary
- Number of deposits processed
- Number of withdrawals processed
- Total charges paid
- Net revenue

#### Recent Transactions
View latest transactions with:
- Transaction ID
- Amount
- Status
- Timestamp
- Quick action buttons

---

### Requesting Withdrawals (Merchant Settlement)

Navigate to: **Merchant Portal → Request Withdrawal**

#### Process:

**Step 1:** Enter withdrawal amount

**Step 2:** Review calculation
- Withdrawal amount
- Service charge (e.g., 1.5%)
- Net amount you'll receive

**Step 3:** Verify bank details
System shows your saved bank details on file.

**Step 4:** Add remarks (optional)

**Step 5:** Submit request

#### After Submission:
- Balance is immediately put on hold
- Request goes to admin for approval
- You receive notification when approved
- Admin transfers money to your bank
- You receive email with payment proof
- Status updates to completed

**Typical Timeline:**
- Approval: Within 24 hours
- Bank transfer: 1-2 business days

---

## Transaction Flows

### Deposit Flow (Customer Paying Merchant)

```
Step 1: Customer initiates payment on merchant website
Step 2: Merchant backend calls gateway API with deposit request
Step 3: Gateway validates merchant credentials and security token
Step 4: Gateway selects available agent based on criteria
Step 5: Gateway creates transaction record (status: PENDING)
Step 6: Gateway temporarily holds amount in agent's limit
Step 7: Gateway returns redirect URL to merchant
Step 8: Merchant redirects customer to payment page
Step 9: Payment page displays agent's wallet number
Step 10: Customer pays agent using their mobile wallet
Step 11: Customer receives TrxID from their wallet
Step 12: Customer enters TrxID on payment page
Step 13: Gateway stores customer's TrxID (status: PROCESSING)
Step 14: Agent's app reads SMS from wallet and sends callback
Step 15: Gateway matches customer TrxID with agent SMS
Step 16: Gateway validates both TrxIDs match
Step 17: Gateway updates transaction (status: SUCCESS)
Step 18: Gateway deducts amount from agent balance
Step 19: Gateway adds commission to agent balance
Step 20: Gateway adds net amount to merchant balance
Step 21: Gateway records merchant charge
Step 22: Gateway sends callback to merchant
Step 23: Merchant receives callback and updates order
Step 24: Merchant sends confirmation to customer
Step 25: Gateway notifies agent via push notification
```

**Timeline:** Typically completes in 5-30 seconds

---

### Withdrawal Flow (Merchant Paying Customer)

```
Step 1: Customer requests withdrawal on merchant platform
Step 2: Merchant backend calls gateway API with withdrawal request
Step 3: Gateway validates merchant credentials and balance
Step 4: Gateway deducts amount from merchant balance immediately
Step 5: Gateway selects available agent
Step 6: Gateway creates transaction record (status: PENDING)
Step 7: Gateway notifies agent via push notification
Step 8: Agent opens withdrawal request in app
Step 9: Agent reviews customer details
Step 10: Agent opens their mobile wallet app
Step 11: Agent pays customer manually
Step 12: Agent receives TrxID from wallet
Step 13: Agent enters TrxID in the app
Step 14: Agent optionally uploads screenshot proof
Step 15: Agent confirms payment
Step 16: Gateway updates transaction (status: SUCCESS)
Step 17: Gateway adds withdrawal amount to agent balance
Step 18: Gateway adds commission to agent balance
Step 19: Gateway records transaction in agent history
Step 20: Gateway sends callback to merchant
Step 21: Merchant receives callback and updates customer account
Step 22: Merchant notifies customer of successful payout
Step 23: Gateway notifies agent of completion
```

**Timeline:** Typically completes in 2-10 minutes (depends on agent availability)

---

## Security Features

### 1. MD5 Security Tokens

**Purpose:** Prevent unauthorized API requests and ensure data integrity.

**How It Works:**
- Every API request must include a security token
- Token is generated by concatenating specific fields in exact order
- Secret key is appended to the end
- MD5 hash is generated from the complete string
- Backend regenerates token and compares with received token
- Request is rejected if tokens don't match

**Protection Against:**
- Unauthorized API access
- Parameter tampering
- Replay attacks (combined with timestamp validation)

---

### 2. IP Whitelisting

**Purpose:** Restrict API access to approved servers only.

**How It Works:**
- Merchants configure allowed IP addresses during account setup
- Every API request's source IP is checked
- Requests from non-whitelisted IPs are rejected immediately
- Can add multiple IPs for redundancy

**Protection Against:**
- Unauthorized access even if API key is compromised
- API key theft
- Malicious requests from unknown sources

**Best Practice:**
Use static IP addresses for production servers.

---

### 3. Single Device Login

**Purpose:** Prevent account sharing and unauthorized access.

**How It Works:**
- Agents and merchants can only be logged in on one device at a time
- New device login requires admin approval
- When new device is approved, all other devices are logged out
- Device information is stored (ID, name, OS, app version, IP)
- Admin can revoke device access at any time

**Protection Against:**
- Account sharing
- Stolen credentials being used from multiple locations
- Unauthorized device access

---

### 4. Two-Factor Authentication (2FA)

**Purpose:** Add extra layer of security beyond passwords.

**How It Works:**
- Users install Google Authenticator app
- System generates unique secret key
- User scans QR code to link account
- System displays 6-digit code that changes every 30 seconds
- User must enter correct code to login
- Backup codes provided for emergency access

**Protection Against:**
- Password theft
- Brute force attacks
- Unauthorized login attempts

**Required For:**
- Super Admin (mandatory)
- Admin and Staff (mandatory)
- Agents (recommended)
- Merchants (optional)

---

### 5. Smart Agent Selection Algorithm

**Purpose:** Distribute transactions fairly and efficiently.

**Selection Criteria:**
- Agent must be ACTIVE status
- App must be online and running
- Last heartbeat within 2 minutes
- Sufficient balance for transaction amount
- Has active gateway for selected payment method
- No pending transactions (to show wallet number)
