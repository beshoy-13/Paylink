# User Stories US-15 to US-28 (Member D)

Stack context: PayLink, .NET 10, modular monolith. Money is `decimal`, balances are computed from ledger entries and holds.
Payment states used: Initiated, FundsHeld, Rejected, SentToBiller, Completed, Failed, Unknown, Refunded.

---

## US-15 -> FR-15 | Priority: Must

**Story:** As a customer, I want to pay bills at ElectroCo, AquaCo and TopUpNet, so that I can pay all my utilities from one wallet even though each biller uses a different format.

**Acceptance Criteria:**

- **Happy path (ElectroCo):** Given a wallet with 1,000.00 EGP available and an ElectroCo bill of 350.00 EGP / When the customer pays / Then the payment reaches Completed, and 352.50 EGP (350.00 + 2.50 fee) is charged.
- **Happy path (AquaCo):** Given an AquaCo bill of 100.00 EGP / When the customer pays / Then PayLink.BillerIntegration sends the SOAP-style XML request and the payment reaches Completed.
- **Happy path (TopUpNet):** Given a TopUpNet top-up of 100.00 EGP / When the biller replies "accepted" / Then the payment is in SentToBiller, and when the callback "success" arrives it becomes Completed.
- **Failure path (biller outage):** Given the ElectroCo mock is stopped / When a customer pays an AquaCo bill of 100.00 EGP / Then the AquaCo payment reaches Completed.
- **Failure path (disabled biller):** Given a biller marked disabled in PayLink.Catalog / When a customer tries to pay it / Then no payment is created and no hold is made.
- **Failure path (AquaCo overpayment):** Given an AquaCo bill of 100.00 EGP / When the payment amount is 100.01 EGP / Then AquaCo refuses, the payment becomes Failed and the hold is released.

---

## US-16 -> FR-16 | Priority: Must

**Story:** As a customer, I want fees calculated by clear rules, including promotions, so that I see exactly what I pay before confirming.

**Acceptance Criteria:**

- **Happy path (fixed):** Given an electricity or water bill of 350.00 EGP / When the fee is calculated / Then fee = 2.50 EGP and total = 352.50 EGP.
- **Happy path (percentage):** Given a mobile top-up of 100.00 EGP / When the fee is calculated / Then fee = 1.50 EGP (1.5%).
- **Minimum fee:** Given a mobile top-up of 50.00 EGP / When the fee is calculated / Then fee = 1.00 EGP (1.5% = 0.75, minimum 1 EGP applies).
- **Tiered:** Given an internet payment of 200.00 EGP / Then fee = 2.00 EGP; given 200.01 EGP / Then fee = 4.00 EGP.
- **Promotion:** Given a promotion on one chosen biller with start and end dates / When a payment to that biller is made inside the dates / Then fee = 0.00 EGP; on the day after the end date the normal fee applies.
- **Failure path (invalid configuration):** Given a fee rule with a negative value or an end date before its start date / When an admin saves it / Then it is rejected and the old rule stays active.
- **Rounding:** Given any fee calculation / Then the result is `decimal`, rounded to 2 decimal places (rounding mode: see OPEN QUESTIONS).

---

## US-17 -> FR-17 | Priority: Should

**Story:** As an agent, I want to pay a bill on behalf of a customer and earn commission, so that I am paid for serving customers at my terminal.

**Acceptance Criteria:**

- **Happy path:** Given an agent-initiated ElectroCo payment of 350.00 EGP with fee 2.50 EGP / When the payment becomes Completed / Then the agent's wallet is credited 1.00 EGP (40% of 2.50) and a commission record exists.
- **No commission before completion:** Given the payment is in FundsHeld, SentToBiller or Unknown / Then the agent's wallet balance has not changed.
- **Failure path (payment fails):** Given an agent payment that becomes Failed / Then no commission is credited and the hold is released.
- **Failure path (unauthorized):** Given a user who is not an agent / When they submit a payment as an agent / Then the request is refused and no payment is created.
- **Zero fee:** Given a promotion fee of 0.00 EGP / When the payment completes / Then the commission is 0.00 EGP (see OPEN QUESTIONS).

---

## US-18 -> FR-18 | Priority: Must

**Story:** As a customer, I want payments above my limits to be rejected with a clear reason, so that I understand why a payment did not go through.

**Acceptance Criteria:**

- **Single-payment limit (high):** Given a payment of 10,000.01 EGP / When submitted / Then the state is Rejected with reason "above single-payment maximum (10,000 EGP)".
- **Single-payment limit (low):** Given a payment of 4.99 EGP / When submitted / Then Rejected with reason "below minimum (5 EGP)".
- **Boundary accepted:** Given payments of exactly 5.00 EGP and exactly 10,000.00 EGP from a verified account with enough funds and daily room / Then neither is rejected for amount.
- **Daily limit (basic):** Given a basic account that has already paid 4,800.00 EGP today / When it submits 300.00 EGP / Then Rejected with reason "daily limit of 5,000 EGP exceeded".
- **Daily limit (verified):** Given a verified account with 29,800.00 EGP paid today / When it submits 300.00 EGP / Then Rejected with reason "daily limit of 30,000 EGP exceeded".
- **No side effects:** Given any Rejected payment / Then no hold is created and wallet balances are unchanged.

---

## US-19 -> FR-19 | Priority: Should

**Story:** As PayLink, I want an account blocked when it makes too many payments too quickly, so that fraud and runaway clients are stopped.

**Acceptance Criteria:**

- **Happy path:** Given 5 payments from one account within 60 seconds / When the 6th payment is submitted within the same 60 seconds / Then it is Rejected and the account is blocked for 10 minutes.
- **Boundary:** Given exactly 5 payments in 60 seconds / Then none are blocked by the velocity rule.
- **During block:** Given an account blocked at 10:00:00 / When it submits a payment at 10:09:59 / Then it is Rejected with reason "account blocked".
- **After block:** Given the same account / When it submits a payment at 10:10:01 / Then the velocity block no longer applies.
- **Failure path (window):** Given 5 payments, then the 6th arrives 61 seconds after the first / Then the account is not blocked.
- Rule values (5, 60 seconds, 10 minutes) are read from configuration.

---

## US-20 -> FR-20 | Priority: Must

**Story:** As a support employee, I want to search a payment by its reference and see its full history, so that I can resolve customer complaints.

**Acceptance Criteria:**

- **Happy path:** Given a payment with reference R1 that went Initiated -> FundsHeld -> SentToBiller -> Unknown -> Completed / When support searches R1 / Then the result shows every state change in order with timestamps, the amounts, hold and ledger entries, and the biller reference.
- **Failure path (not found):** Given a reference that does not exist / When support searches it / Then the result is "not found" and no data from other payments is shown.
- **Failure path (unauthorized):** Given a customer (not support) / When they call the support search / Then access is refused.
- **Audit:** Given any support search / Then an audit entry records who searched and when.

---

## US-21 -> FR-21 | Priority: Should

**Story:** As a support employee, I want to block and unblock an account, so that I can stop suspicious activity or restore a customer's access.

**Acceptance Criteria:**

- **Block:** Given an active account / When support blocks it with a reason / Then a new payment from that account is Rejected with reason "account blocked".
- **Unblock:** Given a blocked account / When support unblocks it / Then the next valid payment is accepted.
- **Failure path (no reason):** Given a block request with an empty reason / Then it is refused and the account state is unchanged.
- **Failure path (unauthorized):** Given a customer calls block or unblock / Then access is refused.
- **Audit:** Given each block or unblock / Then an audit entry records who, when and the reason.

---

## US-22 -> FR-22 | Priority: Must

**Story:** As a customer, I want an SMS and a push message when my payment completes or fails, so that I know the result without opening the app.

**Acceptance Criteria:**

- **Completion:** Given a payment that becomes Completed / Then one SMS and one push notification are created, containing the amount and the payment reference.
- **Failure:** Given a payment that becomes Failed (e.g. AquaCo error code) / Then an SMS and a push notification are created, containing a clear reason.
- **Failure path (SMS provider down):** Given the SMS provider is unavailable when a payment completes / Then the payment still ends Completed and the push notification is still sent.
- **One per outcome:** Given a duplicate TopUpNet "success" callback / Then no second notification is sent.
- **Unknown:** Given a payment in Unknown / Then the customer is told it is being processed (no completion message yet).

---

## US-23 -> FR-23 | Priority: Should

**Story:** As PayLink, I want failed notifications retried and finally marked undelivered, so that messages are not lost silently.

**Acceptance Criteria:**

- **Retry then success:** Given an SMS that fails on the first attempt / When retry 2 succeeds / Then status = Delivered and no further attempts are made.
- **Retry exhaustion:** Given an SMS that fails on the first attempt and on all 3 retries (4 attempts in total) / Then status = Undelivered.
- **Payment unaffected:** Given a notification ends Undelivered / Then the related payment state is unchanged.
- **Record:** Given each attempt / Then the attempt number, time and error are stored.

---

## US-24 -> FR-24 | Priority: Should

**Story:** As an admin, I want to edit fee and limit values without redeploying, so that business changes take effect quickly.

**Acceptance Criteria:**

- **Happy path:** Given the daily limit for basic accounts is 5,000 EGP / When an admin changes it to 4,000 EGP / Then the next payment is checked against 4,000 EGP without restarting the application.
- **Fee change:** Given the electricity fee is 2.50 EGP / When an admin sets it to 3.00 EGP / Then a new inquiry shows fee 3.00 EGP.
- **In-flight payments:** Given a payment already in FundsHeld / Then it keeps the fee it was quoted.
- **Failure path (invalid):** Given an admin enters a negative or non-numeric value / Then it is rejected and the old value stays.
- **Failure path (unauthorized):** Given a non-admin / When they try to edit values / Then access is refused.
- **Audit:** Given each change / Then an audit entry stores who, when, old value and new value.

---

## US-25 -> FR-25 | Priority: Must

**Story:** As an auditor or admin, I want every money movement and admin action written to an audit log, so that I can always see who did what, when and why.

**Acceptance Criteria:**

- **Money movement:** Given a payment that creates a hold and then a final charge / Then each ledger entry has a matching audit record with who, when and why.
- **Completeness:** Given a test run of payments, refunds and top-ups / Then number of ledger entries = number of money-movement audit records (100%).
- **Admin action:** Given an admin disables a biller / Then an audit record is written.
- **Failure path (audit write fails):** Given the audit record cannot be written / Then the money movement is not committed (same transaction) and no ledger entry exists without an audit record.
- **Failure path (tampering):** Given any user or API / When they try to edit or delete an audit record / Then it is refused.

---

## US-26 -> FR-26 | Priority: Should

**Story:** As a finance or support employee, I want a daily reconciliation report from the settlement file, so that differences between PayLink and the biller are found.

**Acceptance Criteria:**

- **Match:** Given the settlement file lists a payment PayLink has as Completed with the same amount / Then it is marked matched.
- **Hard case 7:** Given the settlement file lists a payment PayLink has as Failed / Then the report flags it as a difference for support review, and the state is not changed automatically.
- **Missing at PayLink:** Given the settlement file lists a reference PayLink does not know / Then it is flagged.
- **Unknown resolution:** Given a payment in Unknown that is absent from the settlement file / Then it is flagged (it may then move to Failed per section 2.5).
- **Failure path (bad file):** Given a settlement file that cannot be parsed / Then no report is produced from it and the error is logged.
- **Failure path (repeat):** Given the same file is uploaded twice / Then no duplicate report entries are created.

---

## US-27 -> FR-27 | Priority: Could

**Story:** As a customer, I want to request a refund of a completed payment, and support approves it, so that I can get my money back for a legitimate problem.

**Acceptance Criteria:**

- **Request:** Given a Completed payment of 350.00 EGP / When the customer requests a refund with a reason / Then a refund request with status Pending is created and no money moves.
- **Approval:** Given a Pending request / When support approves / Then new ledger entries return the approved amount to the wallet, each with an audit record (the Completed payment's state is not edited: see OPEN QUESTIONS).
- **Rejection:** Given a Pending request / When support rejects it with a reason / Then no money moves and the customer is notified.
- **Failure path (not completed):** Given a payment in Failed, Unknown or Rejected / When the customer requests a refund this way / Then the request is refused.
- **Failure path (duplicate):** Given a refund already approved for the payment / When another request is made / Then it is refused.
- **Failure path (unauthorized):** Given a customer tries to approve their own request / Then access is refused.

---

## US-28 -> FR-28 | Priority: Could

**Story:** As an admin, I want to view daily totals, so that I can monitor PayLink's business and biller health.

**Acceptance Criteria:**

- **Happy path:** Given on one day 3 Completed payments with fees of 2.50, 2.50 and 1.50 EGP, where one was made by an agent / When the admin opens the totals for that day / Then it shows payments = 3, fees earned = 6.50 EGP, commissions paid = 1.00 EGP.
- **Failures per biller:** Given 2 Failed AquaCo payments and 1 Failed ElectroCo payment / Then failures show AquaCo = 2 and ElectroCo = 1.
- **Empty day:** Given a day with no payments / Then all totals are 0.00 and no error occurs.
- **Failure path (unauthorized):** Given a non-admin / When they request totals / Then access is refused.
- **Consistency:** Given the totals / Then they equal the sums computed from ledger entries for that day.

---

## FR to story map

| FR    | Story | Priority |
| ----- | ----- | -------- |
| FR-15 | US-15 | Must     |
| FR-16 | US-16 | Must     |
| FR-17 | US-17 | Should   |
| FR-18 | US-18 | Must     |
| FR-19 | US-19 | Should   |
| FR-20 | US-20 | Must     |
| FR-21 | US-21 | Should   |
| FR-22 | US-22 | Must     |
| FR-23 | US-23 | Should   |
| FR-24 | US-24 | Should   |
| FR-25 | US-25 | Must     |
| FR-26 | US-26 | Should   |
| FR-27 | US-27 | Could    |
| FR-28 | US-28 | Could    |

---

## OPEN QUESTIONS (not decided: needs team decision, add to ambiguities list)

1. **Promotion and commission:** If the fee is 0, is the agent commission 0 (40% of 0)? Stories assume 0.00 EGP.
2. **Refund vs final state:** Completed is final, yet FR-27 allows refunds. Stories assume a separate refund request with new ledger entries, leaving the payment state Completed. The Refunded state is only reached from Failed. Confirm.
3. **Daily limit counting:** Does it count Completed only, or also FundsHeld, SentToBiller and Unknown? Does it include the fee? Is a day a Cairo calendar day or a rolling 24 hours?
4. **Single-payment limits (5 / 10,000 EGP):** Do they apply to the bill amount or to bill + fee? Stories assume the bill amount.
5. **Velocity rule:** Do Rejected attempts count toward the 5 in 60 seconds? Does the 6th payment count as the trigger and get rejected, or is only the next one blocked?
6. **Notification retry timing:** The brief gives 3 retries but no interval. Interval left unspecified.
7. **Rounding mode** for percentage fees (e.g. 1.5% of 33.33 EGP) is not specified.
8. **Overpayment at AquaCo:** The brief says AquaCo rejects payments above the bill amount. Can a customer pay a partial amount?
9. **Settlement-file format** (columns, matching key) is not given for reconciliation.
10. **Roles and authentication:** The brief does not say how admin, support and agent roles are authenticated; stories only assume "unauthorized access is refused".
11. **Audit log "why":** What counts as a reason for automatic system actions (e.g. retries)?
12. **Fee on in-flight payments** when an admin changes a fee: assumed unchanged (US-24); confirm.
