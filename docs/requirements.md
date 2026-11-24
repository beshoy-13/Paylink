# PayLink: problem summary, stakeholders and user stories

## Problem summary

PayLink is a company which helps people pay their bills through its mobile app or its agents to avoid people waiting in queues in utility offices to pay their bills. It earns money by charging a service fee on each payment and pays agents a commission for their services. It holds customers' money in wallets (accounts with available and held balance).

**Core issue:** PayLink keeps customers' money in wallets which makes it responsible for any mistakes that could occur due to slow network or payments arriving at the same time which results in customers getting charged twice, or the customers paying but the billers never receiving their payment.

**Consequence / risk:** due to the mistakes that may occur, people may lose their trust in the company resulting in money losses for the company due to having to refund double payments or payments that never reached the billers.

**Solution:** a new backend is needed to make sure the system handles money correctly, handles double requests by charging only once, the system is easy to extend with new billers, can handle heavy load. It does this through:

- **Protecting money:** ledger entries where payments are recorded, holds and refunds.
- **Payment processing:** keeping a clear payment life cycle to avoid any chaos.
- **Biller integration:** retry and callback handling for billers to ensure their integration with the system.
- **Fees and commissions:** calculate fees and agent commission.
- **Notifications:** send notifications via SMS or push messages with retry to ensure the customer knows whether the payment was a success or failure.
- **Fraud checks:** verification levels in accounts, wallets, fraud checks to protect customers' wallets and accounts.

**Scope:** this system will be built with modules like accounts and wallets, biller catalog, payment processing, fees and commissions and notifications. The billers will be mockups instead of real companies. The system will have a simple web page or command-line client.

## Stakeholders

| Stakeholder | Role | Main needs |
|---|---|---|
| Customer | Pays bills from their wallet through app or agent | Pay bills, view wallet balance, transaction history, see list of billers, could request refunds |
| Agent | Shop owner with a PayLink terminal | Pay on behalf of customers. Earn a commission of 40% of the service fee when payment is complete |
| Biller | Utility or service companies that provide billing services and receive payment (ex: ElectroCo, AquaCo) | Receive payments correctly and only once |
| Admin | Manages the system's configuration | Have access to add, edit, disable services, edit fees and limits without redeploying, view daily totals |
| Support employee | Handle and fix the customer's problems | Search payment by reference, view its history, block and unblock an account |
| SMS / push provider | Delivers notifications when a payment is done | Provide SMS message when payment is successfully complete or when it has failed |
| Managers | Manage the business and request changes | Reliable payments even on busy days, request changes from developers |
| Engineering team | Build and maintain the system | Separate modules that are easy to change and test |

## Story index

| ID | FR | Priority | Title |
|---|---|---|---|
| US-01 | FR-1 | Must | Register an account |
| US-02 | FR-2 | Should | Upgrade to verified |
| US-03 | FR-3 | Must | View wallet balance and transaction history |
| US-04 | FR-4 | Must | Top up by simulated card |
| US-05 | FR-4 | Must | Top up with cash through an agent |
| US-06 | FR-5 | Must | Record every balance change in the ledger |
| US-07 | FR-6 | Must | Add and edit billers and services |
| US-08 | FR-6 | Must | Disable billers and services |
| US-09 | FR-7 | Must | View billers and their required fields |
| US-10 | FR-8 | Must | Run a bill inquiry |
| US-11 | FR-9 | Must | Pay a bill from the wallet |
| US-12 | FR-9 | Must | Enforce payment limits |
| US-13 | FR-10 | Must | Return the original result for a repeated idempotency key |
| US-14 | FR-11 | Must | Enforce payment life-cycle transitions |
| US-15 | FR-12 | Must | Hold and release funds |
| US-16 | FR-13 | Must | Mark timed-out payments as Unknown |
| US-17 | FR-13 | Must | Re-query Unknown payments |
| US-18 | FR-14 | Must | Handle TopUpNet callbacks |

## US-01 | FR-1 | Priority: Must
**Story:** As a customer, I want to register, so that I can use a wallet.

**Acceptance criteria**
- **AC1 (happy path):**
  - **Given** a person is not registered
  - **When** they submit their registration info
  - **Then** an account is created with verification level Basic and a verification email is sent
- **AC2 (failure path, email not verified):**
  - **Given** an account whose email is not yet verified
  - **When** the customer tries to enter
  - **Then** access is denied until the email is verified
- **AC3 (failure path, duplicate email):**
  - **Given** a customer already has an account
  - **When** they register with the same email
  - **Then** registration fails and no second account is created
- **AC4 (business rule, default daily limit):**
  - **Given** a new Basic account
  - **When** its daily limit is checked
  - **Then** it is 5,000 EGP

## US-02 | FR-2 | Priority: Should
**Story:** As an admin, I want to upgrade a customer to Verified after a manual review, so that their daily limit is raised.

**Acceptance criteria**
- **AC1 (happy path, daily limit raised):**
  - **Given** a Basic customer
  - **When** the admin completes the upgrade
  - **Then** the level is Verified and the daily limit is 30,000 EGP
- **AC2 (failure path, maximum unchanged):**
  - **Given** a Verified customer
  - **When** they submit a payment with a total of 10,000.01 EGP
  - **Then** it is refused, because the single-payment maximum stays 10,000 EGP
- **AC3 (failure path, minimum unchanged):**
  - **Given** a Verified customer
  - **When** they submit a payment with a total of 4.99 EGP
  - **Then** it is refused, because the single-payment minimum stays 5 EGP
- **AC4 (business rule, not upgraded):**
  - **Given** a Basic customer who has not been upgraded
  - **When** their daily limit is checked
  - **Then** it is 5,000 EGP

## US-03 | FR-3 | Priority: Must
**Story:** As a customer, I want to view my wallet's available and held balance and my transaction history, so that I know what I can spend and what happened to my money.

**Acceptance criteria**
- **AC1 (happy path, balances shown):**
  - **Given** a wallet with 100.00 EGP available and 20.00 EGP held
  - **When** the customer opens the wallet
  - **Then** available shows 100.00 and held shows 20.00, both in EGP with 2 decimals
- **AC2 (happy path, hold reflected):**
  - **Given** a hold of 50.00 EGP is created
  - **When** the customer reopens the wallet
  - **Then** available is reduced by 50.00 and held is increased by 50.00
- **AC3 (happy path, history paging):**
  - **Given** a customer has 11 transactions
  - **When** they open history
  - **Then** page 1 shows 10 transactions and page 2 shows 1
- **AC4 (edge case, empty history):**
  - **Given** a customer has 0 transactions
  - **When** they open history
  - **Then** an empty list is shown and no error occurs

## US-04 | FR-4 | Priority: Must
**Story:** As a customer, I want to top up my wallet with a simulated card, so that I have money to pay bills.

**Acceptance criteria**
- **AC1 (happy path):**
  - **Given** a registered customer
  - **When** a simulated card top-up of 100.00 EGP succeeds
  - **Then** available increases by 100.00 and 1 ledger entry is created
- **AC2 (failure path, card declined):**
  - **Given** a simulated card top-up is declined
  - **When** the result is returned
  - **Then** the balance is unchanged and 0 ledger entries are created

## US-05 | FR-4 | Priority: Must
**Story:** As an agent, I want to top up a customer's wallet with cash, so that the customer can pay bills without a card.

**Acceptance criteria**
- **AC1 (happy path):**
  - **Given** an agent receives 200.00 EGP cash from a registered customer
  - **When** the agent confirms
  - **Then** the customer's available balance increases by 200.00 and 1 ledger entry is created
- **AC2 (failure path, account not found):**
  - **Given** the agent enters an account that does not exist
  - **When** they confirm
  - **Then** the top-up fails and 0 ledger entries are created

## US-06 | FR-5 | Priority: Must
**Story:** As a system, I want to create a ledger entry for every balance change and compute balances from the ledger, so that every money movement is recorded and a balance is never overwritten.

**Acceptance criteria**
- **AC1 (happy path, entry per change):**
  - **Given** a top-up, hold, release or refund occurs
  - **When** the balance changes
  - **Then** exactly 1 ledger entry is created with the amount and the wallet
- **AC2 (happy path, balance from ledger):**
  - **Given** a wallet with ledger entries of +100.00, +50.00 and −30.00
  - **When** the balance is calculated
  - **Then** it equals 120.00
- **AC3 (failure path, direct overwrite):**
  - **Given** a request to set a balance directly to a value
  - **When** it is submitted
  - **Then** it is rejected and the ledger is unchanged
- **AC4 (edge case, exact decimals):**
  - **Given** ledger entries of 0.10 and 0.20
  - **When** the balance is calculated
  - **Then** it equals exactly 0.30 (exact decimals, 2 decimal places)
- **AC5 (failure path, rollback):**
  - **Given** a balance change fails and is rolled back
  - **When** the ledger is checked
  - **Then** 0 entries exist for it

## US-07 | FR-6 | Priority: Must
**Story:** As an admin, I want to add and edit billers and services, so that customers can pay them and their details stay correct.

**Acceptance criteria**
- **AC1 (happy path, add):**
  - **Given** the admin enters a biller name and service details
  - **When** they save
  - **Then** the biller and service exist and appear in the customer list
- **AC2 (failure path, missing name):**
  - **Given** the biller name is empty
  - **When** the admin saves
  - **Then** saving fails and 0 billers are created
- **AC3 (happy path, fee edit):**
  - **Given** an existing biller with a fee of 2.00 EGP
  - **When** the admin changes the fee to 3.00 EGP and saves
  - **Then** the next inquiry for that biller shows a fee of 3.00 EGP
- **AC4 (failure path, invalid edit):**
  - **Given** the admin submits an edit that fails validation
  - **When** they save
  - **Then** the change is rejected and the old values remain

## US-08 | FR-6 | Priority: Must
**Story:** As an admin, I want to disable billers and services, so that customers can no longer start payments to them.

**Acceptance criteria**
- **AC1 (happy path):**
  - **Given** an active biller
  - **When** the admin disables it
  - **Then** its status is Disabled and it is not shown in the customer biller list
- **AC2 (failure path, disabled biller):**
  - **Given** a Disabled biller
  - **When** a customer tries to start a payment to it
  - **Then** no payment is created
- **AC3 (edge case, payment in progress):**
  - **Given** a payment to a biller is in SentToBiller
  - **When** the admin disables that biller
  - **Then** the payment continues to Completed, Failed or Unknown

## US-09 | FR-7 | Priority: Must
**Story:** As a customer, I want to see the list of billers and the input fields each requires, so that I know what to enter.

**Acceptance criteria**
- **AC1 (happy path):**
  - **Given** an active biller that requires 2 input fields
  - **When** the customer opens the list
  - **Then** the biller is shown with exactly those 2 fields
- **AC2 (edge case, disabled biller hidden):**
  - **Given** a Disabled biller
  - **When** the customer opens the list
  - **Then** it is not shown

## US-10 | FR-8 | Priority: Must
**Story:** As a customer, I want to run a bill inquiry and see the amount, fee and total before paying, so that I know the cost.

**Acceptance criteria**
- **AC1 (happy path):**
  - **Given** all required fields are entered and the biller returns an amount of 100.00 EGP with a fee of 2.00 EGP
  - **When** the customer runs the inquiry
  - **Then** the amount 100.00, fee 2.00 and total 102.00 are shown
- **AC2 (failure path, missing field):**
  - **Given** a required field is missing
  - **When** the customer runs the inquiry
  - **Then** no amounts are shown and the missing field is identified
- **AC3 (edge case, no side effects):**
  - **Given** an inquiry is run
  - **When** the result is displayed
  - **Then** 0 payments, 0 holds and 0 ledger entries are created

## US-11 | FR-9 | Priority: Must
**Story:** As a customer, I want to pay a bill from my wallet, so that I can settle it without cash.

**Acceptance criteria**
- **AC1 (happy path):**
  - **Given** available 500.00 EGP and an inquiry total of 102.00 EGP
  - **When** the customer confirms
  - **Then** a payment is created in status Initiated
- **AC2 (failure path, insufficient funds):**
  - **Given** available 50.00 EGP and an inquiry total of 102.00 EGP
  - **When** the customer confirms
  - **Then** the payment becomes Rejected and the balance is unchanged

## US-12 | FR-9 | Priority: Must
**Story:** As a system, I want to check each payment against the single-payment and daily limits before it is created, so that no payment breaks a limit.

**Acceptance criteria**
- **AC1 (failure path, below minimum):**
  - **Given** a total of 4.99 EGP
  - **When** submitted
  - **Then** it is refused before creation: 0 payments and 0 log entries are created
- **AC2 (failure path, above maximum):**
  - **Given** a total of 10,000.01 EGP
  - **When** submitted
  - **Then** it is refused before creation: 0 payments and 0 log entries are created
- **AC3 (failure path, daily limit exceeded):**
  - **Given** a Basic customer with 4,900.00 EGP already counted today (FundsHeld, SentToBiller, Unknown or Completed payments)
  - **When** they submit a total of 200.00 EGP
  - **Then** it is refused before creation
- **AC4 (boundary, daily limit reached exactly):**
  - **Given** a Basic customer with 4,800.00 EGP already counted today
  - **When** they submit a total of 200.00 EGP
  - **Then** it is accepted (total 5,000.00 EGP)
- **AC5 (boundary, minimum and maximum accepted):**
  - **Given** totals of exactly 5.00 EGP and exactly 10,000.00 EGP within the daily limit
  - **When** submitted
  - **Then** both are accepted
- **AC6 (edge case, daily reset at midnight Cairo time):**
  - **Given** a customer who made payments at 23:59 and 00:01 Cairo time
  - **When** the daily total is counted
  - **Then** each payment counts toward its own calendar day

## US-13 | FR-10 | Priority: Must
**Story:** As a customer, I want a repeated request with the same idempotency key to return the original result, so that I am never charged twice.

**Acceptance criteria**
- **AC1 (happy path, repeat returns original):**
  - **Given** a payment was created with key K
  - **When** the same request is sent again with key K
  - **Then** the original result is returned and 0 additional payments, holds or ledger entries are created
- **AC2 (happy path, different key):**
  - **Given** a payment was created with key K
  - **When** a request with a different key is sent
  - **Then** a new payment is created
- **AC3 (failure path, same key different data):**
  - **Given** key K was used
  - **When** it is sent again with different request data
  - **Then** an error is returned and nothing is created
- **AC4 (edge case, request still processing):**
  - **Given** the first request with key K is still processing
  - **When** the same key is sent again
  - **Then** the in-progress status is returned and nothing is created

## US-14 | FR-11 | Priority: Must
**Story:** As a system, I want every payment to follow only the allowed life-cycle transitions, so that payments never reach an invalid state.

**Acceptance criteria**
- **AC1 (happy path, Initiated to FundsHeld):**
  - **Given** a payment in Initiated
  - **When** fraud checks pass and funds are enough
  - **Then** the status becomes FundsHeld
- **AC2 (failure path, Initiated to Rejected):**
  - **Given** a payment in Initiated
  - **When** a fraud check fails or funds are insufficient
  - **Then** the status becomes Rejected
- **AC3 (happy path, biller outcome):**
  - **Given** a payment in SentToBiller
  - **When** the biller confirms, refuses, or gives no answer within the timeout
  - **Then** the status becomes Completed, Failed or Unknown respectively
- **AC4 (failure path, final states):**
  - **Given** a payment in Completed, Rejected or Refunded
  - **When** any status change is requested
  - **Then** it is refused and the status is unchanged
- **AC5 (failure path, skipped transition):**
  - **Given** a payment in Initiated
  - **When** a change to Completed is requested
  - **Then** it is refused and the status stays Initiated
- **AC6 (failure path, Unknown not refunded):**
  - **Given** a payment in Unknown
  - **When** a change to Refunded is requested
  - **Then** it is refused and the status stays Unknown

## US-15 | FR-12 | Priority: Must
**Story:** As a system, I want to hold funds before calling the biller and release them if the payment fails, so that the customer's money is reserved and returned on failure.

**Acceptance criteria**
- **AC1 (happy path, hold):**
  - **Given** a payment with a total of 102.00 EGP reaches FundsHeld
  - **When** the hold is created
  - **Then** available is reduced by 102.00, held is increased by 102.00, 1 ledger entry is created, and this happens before the biller is called
- **AC2 (failure path, hold not created):**
  - **Given** the hold could not be created
  - **When** the payment is processed
  - **Then** the biller is called 0 times
- **AC3 (happy path, release and refund):**
  - **Given** a payment in Failed with a hold of 102.00 EGP
  - **When** the failure is recorded
  - **Then** immediately held is reduced by 102.00, available is increased by 102.00, 1 ledger entry is created and the status becomes Refunded
- **AC4 (edge case, Unknown keeps hold):**
  - **Given** a payment in Unknown with a hold of 102.00 EGP
  - **When** time passes
  - **Then** the hold stays unchanged until the status is resolved

## US-16 | FR-13 | Priority: Must
**Story:** As a system, I want a payment to a live biller to become Unknown when the biller does not answer within 10 seconds, so that the result is not guessed.

**Acceptance criteria**
- **AC1 (failure path, timeout):**
  - **Given** a payment in SentToBiller to a live biller
  - **When** no answer arrives within 10 seconds
  - **Then** the status becomes Unknown
- **AC2 (happy path, answer in time):**
  - **Given** the biller answers within 10 seconds
  - **When** the answer arrives
  - **Then** the status becomes Completed or Failed and not Unknown

## US-17 | FR-13 | Priority: Must
**Story:** As a system, I want to re-query Unknown payments at 1, 5 and 15 minutes, so that they reach a final result.

**Acceptance criteria**
- **AC1 (happy path, 3 re-queries):**
  - **Given** a payment became Unknown
  - **When** 1, 5 and 15 minutes pass
  - **Then** the biller is re-queried at each time, 3 times in total
- **AC2 (happy path, confirmed):**
  - **Given** a re-query returns a confirmation
  - **When** received
  - **Then** the status becomes Completed
- **AC3 (failure path, refused or not found):**
  - **Given** a re-query returns a refusal, or reconciliation finds no payment
  - **When** received
  - **Then** the status becomes Failed and then Refunded immediately
- **AC4 (edge case, no answer after 3):**
  - **Given** all 3 re-queries get no answer
  - **When** the third ends
  - **Then** the status remains Unknown, 0 further automatic re-queries occur, 0 refunds are made and the hold stays

## US-18 | FR-14 | Priority: Must
**Story:** As a system, I want to accept asynchronous callbacks from TopUpNet and ignore duplicates, so that each result is processed exactly once.

**Acceptance criteria**
- **AC1 (happy path, first callback):**
  - **Given** a callback with transaction reference R arrives for the first time
  - **When** it is received
  - **Then** it is processed once and exactly 1 result is recorded for R
- **AC2 (edge case, duplicate):**
  - **Given** a callback for R was already processed
  - **When** the same callback arrives again
  - **Then** it is ignored and the balance, ledger entries and statuses are unchanged
- **AC3 (edge case, simultaneous duplicates):**
  - **Given** 2 identical callbacks for R arrive at the same time
  - **When** both are received
  - **Then** exactly 1 is processed and 1 is ignored

## Requirement to story mapping

| FR | Requirement | User stories | Priority |
|---|---|---|---|
| FR-1 | A customer can register and has a basic verification level | US-01 | Must |
| FR-2 | A customer can be upgraded to verified, which raises limits | US-02 | Should |
| FR-3 | A customer can view wallet balance (available and held) and transaction history | US-03 | Must |
| FR-4 | A customer can top up the wallet (simulated card or agent cash) | US-04, US-05 | Must |
| FR-5 | Every balance change creates a ledger entry; balances are computed from the ledger | US-06 | Must |
| FR-6 | An admin can add, edit and disable billers and services | US-07, US-08 | Must |
| FR-7 | A customer can see the list of billers and the input fields each requires | US-09 | Must |
| FR-8 | A customer can run a bill inquiry and see the amount, fee and total before paying | US-10 | Must |
| FR-9 | A customer can pay a bill from the wallet | US-11, US-12 | Must |
| FR-10 | A repeated request with the same idempotency key returns the original result | US-13 | Must |
| FR-11 | A payment follows the life cycle and no other transitions | US-14 | Must |
| FR-12 | Funds are held before the biller is called and released if the payment fails | US-15 | Must |
| FR-13 | A payment that times out becomes Unknown and is re-queried by the retry rules | US-16, US-17 | Must |
| FR-14 | The system accepts asynchronous callbacks from TopUpNet and ignores duplicates | US-18 | Must |

## Ambiguities

The questions and the answers given.

| # | Question | Related | Answer given |
|---|---|---|---|
| 1 | Who performs the upgrade to Verified, and what is the verification process (documents, manual review)? | FR-2 | Admin verifies, manual review. |
| 2 | What does "verification before they can enter" mean at registration (OTP, email link)? | FR-1 | Verification is done by email. |
| 3 | Do the single-payment limits (5 and 10,000 EGP) change for Verified customers? The requirement says Verified "raises limits" but only gives daily numbers. | FR-2 | Only the daily limit is changed. |
| 4 | Is a payment that breaks a limit moved to Rejected, or refused before a payment is created? The life cycle allows Rejected only for fraud or insufficient funds. | FR-9, FR-11 | It is refused before the payment is created and not written in the log. |
| 5 | Does the daily limit count only Completed payments, or also FundsHeld, SentToBiller and Unknown ones? Is a day a Cairo calendar day or a rolling 24 hours? | FR-2, FR-9 | It counts all of them (FundsHeld, SentToBiller, Unknown and Completed). The day is a Cairo calendar day, resetting at midnight Cairo time. |
| 6 | How is the fee calculated, and is total always amount plus fee? | FR-8 | The fee is decided by the admin per biller, total equals amount + fee. |
| 7 | Do the limit checks (single payment 5 / 10,000 EGP and the daily limit) apply to the bill amount or to bill + fee? | FR-9 | The total (bill + fee), for both the single-payment limits and the daily limit. |
| 8 | Are there top-up limits, minimums or fees, and can a customer have several pending top-ups? | FR-4 | Limits, minimums and fees are not specified. There is no pending state in top-ups. |
| 9 | How do agents authenticate, and is there an agent commission or float? If the fee is 0, is the agent commission 0 (40% of 0)? | FR-4 | Agents are authenticated users. The agent commission is 40% of the service fee when the payment is complete. If the fee is 0, the commission is 0.00 EGP. |
| 10 | What is TopUpNet exactly, and does it handle only card top-ups? What identifies a duplicate callback (transaction id)? | FR-14 | TopUpNet is the mobile top-up service. |
| 11 | What does disabling a biller do to payments already in progress? | FR-6 | Only new payments are blocked. |
| 12 | Which fields can an admin edit, and is every edit audited? | FR-6 | Admin can add, edit, disable billers, edit fee and limit values, view daily totals, payments, fees, commissions and failures. |
| 13 | How many transactions does history show, and is there paging, filtering or sorting? | FR-3 | 10 transactions per page. |
| 14 | Does "10 seconds" apply to non-live billers? What is the timeout for them? | FR-13 | No, it does not. |
| 15 | Who triggers the Unknown to Completed/Failed update after the 3 re-queries are exhausted, and what is the manual process? | FR-13 | There is no manual process. The job itself is the trigger. |
| 16 | What happens to a hold on an Unknown payment that stays Unknown after the third re-query? | FR-12, FR-13 | Money stays held until resolved. |
| 17 | Are notifications (SMS, push) required for payment results? | FR-9 | Yes, they are required. |
| 18 | Which currency rounding rule applies for exact decimals (number of places)? | Business rules | 2 decimal places. |
| 19 | When the biller refuses, what moves the payment from Failed to Refunded, and is it immediate? Completed is final, but customers could request refunds (see stakeholders): can a refund be reached from Completed? | FR-12 | The system does it immediately. A refund is reached from the Failed state. |




## Non-functional requirements

| ID | Quality | Metric | Target | How it is measured |
|---|---|---|---|---|
| NFR-1 | Correctness | Double charges and negative balances | 0 and 0 | Duplicate-request and concurrent-payment tests (hard cases 1, 4 and 6) |
| NFR-2 | Performance | Sustained payment throughput against the mock billers | 100 payments per second for 60 seconds | Load test with k6 or NBomber in Phase 2 |
| NFR-3 | Resilience | Payments to other billers during a biller outage | 100% still succeed | Stop the ElectroCo mock, then pay an AquaCo bill |
| NFR-4 | Modifiability | Files changed outside Biller Integration and configuration when adding a biller | 0 | Count changed files in the git diff (hard case 8) |
| NFR-5 | Auditability | Money movements with an audit record (who, when, why) | 100% | Test compares the number of ledger entries with audit entries |
| NFR-6 | Testability | Line coverage on domain logic | At least 70% | Coverlet report produced by the CI pipeline |
| NFR-7 | Reproducibility | A fresh clone starts the application and the database with one command | docker compose up --build succeeds and /health returns 200 | Clean-clone test on a second machine, plus the green CI run on every push |
