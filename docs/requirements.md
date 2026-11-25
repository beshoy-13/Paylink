# PayLink: problem summary, stakeholders and user stories

## Problem summary

PayLink is a company which helps people pay their bills through its mobile app or its agents to avoid people waiting in queues in utility offices to pay their bills. It earns money by charging a service fee on each payment and pays agents a commission for their services. It holds customers' money in wallets (accounts with available and held balance).

**Core issue:** PayLink keeps customers' money in wallets which makes it responsible for any mistakes that could occur due to slow network or payments arriving at the same time which results in customers getting charged twice, or the customers paying but the billers never receiving their payment.

**Consequence / risk:** due to the mistakes that may occur, people may lose their trust in the company resulting in money losses for the company due to having to refund double payments or payments that never reached the billers. Regulators may also investigate or fine the company, and the finance team has to find and correct the wrong balances.

**Main fears:** a customer being charged twice, a wallet balance going below zero, and a payment stuck in an unknown state while the customer's money stays held.

**Solution:** a new backend is needed to make sure the system handles money correctly, handles double requests by charging only once, the system is easy to extend with new billers, can handle heavy load. It does this through:

- **Protecting money:** ledger entries where payments are recorded, holds and refunds.
- **Payment processing:** keeping a clear payment life cycle to avoid any chaos.
- **Biller integration:** retry and callback handling for billers to ensure their integration with the system.
- **Fees and commissions:** calculate fees and agent commission.
- **Notifications:** send notifications via SMS or push messages with retry to ensure the customer knows whether the payment was a success or failure.
- **Fraud checks:** verification levels in accounts, wallets, fraud checks to protect customers' wallets and accounts.

**Scope:** this system will be built with modules like accounts and wallets, biller catalog, payment processing, fees and commissions and notifications. The billers will be mockups instead of real companies. The system will have a simple web page or command-line client.

**Out of scope:** real billers, real card payments, real SMS and push delivery, and a polished user interface. The NetCo batch biller is optional.

## Stakeholders

| Stakeholder         | Role                                                                                                   | Main needs                                                                                              |
| ------------------- | ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------- |
| Customer            | Pays bills from their wallet through app or agent                                                      | Pay bills, view wallet balance, transaction history, see list of billers, could request refunds         |
| Agent               | Shop owner with a PayLink terminal                                                                     | Pay on behalf of customers. Earn a commission of 40% of the service fee when payment is complete        |
| Biller              | Utility or service companies that provide billing services and receive payment (ex: ElectroCo, AquaCo) | Receive payments correctly and only once                                                                |
| Admin               | Manages the system's configuration                                                                     | Have access to add, edit, disable services, edit fees and limits without redeploying, view daily totals |
| Support employee    | Handle and fix the customer's problems                                                                 | Search payment by reference, view its history, block and unblock an account                             |
| SMS / push provider | Delivers notifications when a payment is done                                                          | Provide SMS message when payment is successfully complete or when it has failed                         |
| Managers            | Manage the business and request changes                                                                | Reliable payments even on busy days, request changes from developers                                    |
| Engineering team    | Build and maintain the system                                                                          | Separate modules that are easy to change and test                                                       |

## Story index

| ID       | FR          | Priority       | Title                                                                |
| -------- | ----------- | -------------- | -------------------------------------------------------------------- |
| US-01    | FR-1        | Must           | Register an account                                                  |
| US-02    | FR-2        | Should         | Upgrade to verified                                                  |
| US-03    | FR-3        | Must           | View wallet balance and transaction history                          |
| US-04    | FR-4        | Must           | Top up by simulated card                                             |
| US-05    | FR-4        | Must           | Top up with cash through an agent                                    |
| US-06    | FR-5        | Must           | Record every balance change in the ledger                            |
| US-07    | FR-6        | Must           | Add and edit billers and services                                    |
| US-08    | FR-6        | Must           | Disable billers and services                                         |
| US-09    | FR-7        | Must           | View billers and their required fields                               |
| US-10    | FR-8        | Must           | Run a bill inquiry                                                   |
| US-11    | FR-9        | Must           | Pay a bill from the wallet                                           |
| US-12    | FR-9        | Must           | Enforce payment limits                                               |
| US-13    | FR-10       | Must           | Return the original result for a repeated idempotency key            |
| US-14    | FR-11       | Must           | Enforce payment life-cycle transitions                               |
| US-15    | FR-12       | Must           | Hold and release funds                                               |
| US-16    | FR-13       | Must           | Mark timed-out payments as Unknown                                   |
| US-17    | FR-13       | Must           | Re-query Unknown payments                                            |
| US-18    | FR-14       | Must           | Handle TopUpNet callbacks                                            |
| US-19    | FR-9, FR-12 | Must           | Process concurrent payments without a negative balance (hard case 4) |
| US-20    | FR-9, FR-12 | Must           | Complete a payment: final charge, fee income and commission          |
| US-21    | FR-15       | Must           | Pay at ElectroCo, AquaCo and TopUpNet                                |
| US-22    | FR-16       | Must           | Calculate fees and promotions                                        |
| US-23    | FR-17       | Should         | Agent pays and earns commission                                      |
| US-24    | FR-18       | Must           | Reject payments above the daily limit                                |
| US-25    | FR-19       | Should         | Block an account for velocity                                        |
| US-26    | FR-20       | Must           | Support searches a payment                                           |
| US-27    | FR-21       | Should         | Block and unblock an account                                         |
| US-28    | FR-22       | Must           | SMS and push on completion or failure                                |
| US-29    | FR-23       | Should         | Retry notifications                                                  |
| US-30    | FR-24       | Should         | Edit fees and limits without redeploying                             |
| US-31    | FR-25       | Must           | Audit log                                                            |
| US-32    | FR-26       | Should         | Daily reconciliation report                                          |
| US-33    | FR-27       | Could          | Refund request for a completed payment                               |
| US-34    | FR-28       | Could          | Admin daily totals                                                   |
| ## US-01 | FR-1        | Priority: Must |

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
- **AC2 (failure path, already verified):**
  - **Given** a customer who is already Verified
  - **When** the admin tries to upgrade them again
  - **Then** the request is refused with the reason "already verified" and the daily limit stays 30,000 EGP
- **AC3 (failure path, customer not found):**
  - **Given** a customer account that does not exist
  - **When** the admin tries to upgrade it
  - **Then** the request fails and no account is changed
- **AC4 (failure path, not an admin):**
  - **Given** a user who is not an admin
  - **When** they try to upgrade a customer
  - **Then** the request is refused and the level stays Basic
- **AC5 (business rule, not upgraded):**
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
- **AC5 (failure path, not signed in):**
  - **Given** a person who is not signed in
  - **When** they request a wallet or its history
  - **Then** access is denied and no balance or transaction is shown
- **AC6 (failure path, another customer's wallet):**
  - **Given** a signed-in customer
  - **When** they request the wallet of a different customer
  - **Then** access is denied and no balance or transaction is shown

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
- **AC3 (failure path, invalid amount):**
  - **Given** a registered customer
  - **When** they submit a top-up of 0.00 EGP or a negative amount
  - **Then** the top-up is refused with a clear reason and 0 ledger entries are created

## US-05 | FR-4 | Priority: Must

**Story:** As an agent, I want to top up a customer's wallet with cash, so that the customer can pay bills without a card.

**Acceptance criteria**

- **AC1 (happy path):**
  - **Given** an agent with at least 200.00 EGP available receives 200.00 EGP cash from a registered customer
  - **When** the agent confirms
  - **Then** the agent's available balance decreases by 200.00, the customer's available balance increases by 200.00, and 2 ledger entries are created
- **AC2 (failure path, account not found):**
  - **Given** the agent enters an account that does not exist
  - **When** they confirm
  - **Then** the top-up fails and 0 ledger entries are created
- **AC3 (failure path, agent balance too low):**
  - **Given** an agent with 100.00 EGP available
  - **When** they confirm a cash top-up of 200.00 EGP
  - **Then** the top-up is refused, both balances are unchanged and 0 ledger entries are created

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

_Fee and limit values are edited under FR-24 (fee and limit configuration), not in this story._

**Acceptance criteria**

- **AC1 (happy path, add):**
  - **Given** the admin enters a biller name and service details
  - **When** they save
  - **Then** the biller and service exist and appear in the customer list
- **AC2 (failure path, missing name):**
  - **Given** the biller name is empty
  - **When** the admin saves
  - **Then** saving fails and 0 billers are created
- **AC3 (happy path, edit required fields):**
  - **Given** an existing biller that requires 1 input field (meter number)
  - **When** the admin adds a second required field and saves
  - **Then** the customer list shows the biller with exactly those 2 fields
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
- **AC3 (failure path, unknown or disabled biller):**
  - **Given** a biller that does not exist or is Disabled
  - **When** the customer requests its required fields directly
  - **Then** an error with a clear reason is returned and no fields are shown

## US-10 | FR-8 | Priority: Must

**Story:** As a customer, I want to run a bill inquiry and see the amount, fee and total before paying, so that I know the cost.

**Acceptance criteria**

- **AC1 (happy path):**
  - **Given** a valid ElectroCo meter number and the biller returns an amount of 350.00 EGP with the fixed electricity fee of 2.50 EGP
  - **When** the customer runs the inquiry
  - **Then** the amount 350.00, fee 2.50 and total 352.50 are shown
- **AC2 (failure path, missing field):**
  - **Given** a required field is missing
  - **When** the customer runs the inquiry
  - **Then** no amounts are shown and the missing field is identified
- **AC3 (failure path, bill not found):**
  - **Given** a meter number for which the biller has no bill
  - **When** the customer runs the inquiry
  - **Then** no amounts are shown and a "bill not found" reason is displayed
- **AC4 (failure path, biller unreachable):**
  - **Given** the ElectroCo mock does not answer within 10 seconds
  - **When** the customer runs the inquiry
  - **Then** no amounts are shown, a "try again later" reason is displayed and no payment is created
- **AC5 (edge case, no side effects):**
  - **Given** an inquiry is run
  - **When** the result is displayed
  - **Then** 0 payments, 0 holds and 0 ledger entries are created

## US-11 | FR-9 | Priority: Must

**Story:** As a customer, I want to pay a bill from my wallet, so that I can settle it without cash.

**Acceptance criteria**

- **AC1 (happy path):**
  - **Given** available 1,000.00 EGP and an inquiry with bill 350.00 EGP, fee 2.50 EGP and total 352.50 EGP
  - **When** the customer confirms
  - **Then** a payment is created in status Initiated
- **AC2 (failure path, insufficient funds):**
  - **Given** available 50.00 EGP and an inquiry total of 352.50 EGP
  - **When** the customer confirms
  - **Then** the payment becomes Rejected with the reason "insufficient funds" and the balance is unchanged
- **AC3 (failure path, blocked account):**
  - **Given** an account that is blocked
  - **When** the customer confirms a payment
  - **Then** the payment becomes Rejected with the reason "account blocked" and 0 holds are created
- **AC4 (failure path, disabled biller):**
  - **Given** a biller that has just been disabled
  - **When** the customer confirms a payment to it
  - **Then** no payment is created and no hold is made

## US-12 | FR-9 | Priority: Must

**Story:** As a system, I want to check each payment against the single-payment and daily limits, so that no payment breaks a limit.

_In this story "log" means the audit log (FR-25). Limits apply to the bill amount, not to the fee._

**Acceptance criteria**

_Single-payment limits are validation: the request is refused before a payment is created._

- **AC1 (failure path, below minimum):**
  - **Given** a bill amount of 4.99 EGP
  - **When** it is submitted
  - **Then** it is refused with the reason "below the 5 EGP minimum" before creation: 0 payments and 0 log entries are created
- **AC2 (failure path, above maximum):**
  - **Given** a bill amount of 10,000.01 EGP
  - **When** it is submitted
  - **Then** it is refused with the reason "above the 10,000 EGP maximum" before creation: 0 payments and 0 log entries are created
- **AC3 (boundary, minimum and maximum accepted):**
  - **Given** bill amounts of exactly 5.00 EGP and exactly 10,000.00 EGP within the daily limit
  - **When** submitted
  - **Then** both are accepted

_Daily limit, velocity and blocked account are fraud checks: a payment is created and moves to Rejected with a clear reason._

- **AC4 (failure path, daily limit exceeded):**
  - **Given** a Basic customer with 4,900.00 EGP of bill amounts already counted today (payments in FundsHeld, SentToBiller, Unknown or Completed)
  - **When** they submit a bill amount of 200.00 EGP
  - **Then** the payment becomes Rejected with the reason "daily limit exceeded", and the balance is unchanged
- **AC5 (boundary, daily limit reached exactly):**
  - **Given** a Basic customer with 4,800.00 EGP already counted today
  - **When** they submit a bill amount of 200.00 EGP
  - **Then** it is accepted (5,000.00 EGP in total for the day)
- **AC6 (edge case, daily reset at midnight Cairo time):**
  - **Given** a customer who made payments at 23:59 and 00:01 Cairo time
  - **When** the daily total is counted
  - **Then** each payment counts toward its own calendar day
- **AC7 (failure path, velocity, rejected attempts count):**
  - **Given** a customer made 5 payment attempts in the last 60 seconds, 2 of them Rejected
  - **When** they submit a 6th attempt
  - **Then** it becomes Rejected with the reason "too many payments" and the account is blocked for 10 minutes
- **AC8 (failure path, blocked by velocity):**
  - **Given** an account blocked for 10 minutes by the velocity rule
  - **When** the customer submits a payment during that time
  - **Then** it becomes Rejected with the reason "account blocked"

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
- **AC7 (happy path, Unknown to Completed):**
  - **Given** a payment in Unknown
  - **When** the biller later confirms (callback, retry or settlement file)
  - **Then** the status becomes Completed
- **AC8 (failure path, Unknown to Failed):**
  - **Given** a payment in Unknown
  - **When** the biller later refuses, or reconciliation finds no payment
  - **Then** the status becomes Failed
- **AC9 (happy path, Failed to Refunded):**
  - **Given** a payment in Failed
  - **When** the hold is released to the wallet
  - **Then** the status becomes Refunded

## US-15 | FR-12 | Priority: Must

**Story:** As a system, I want to hold funds before calling the biller and release them if the payment fails, so that the customer's money is reserved and returned on failure.

**Acceptance criteria**

- **AC1 (happy path, hold):**
  - **Given** a payment with a total of 352.50 EGP reaches FundsHeld
  - **When** the hold is created
  - **Then** available is reduced by 352.50, held is increased by 352.50, 1 ledger entry is created, and this happens before the biller is called
- **AC2 (failure path, hold not created):**
  - **Given** the hold could not be created
  - **When** the payment is processed
  - **Then** the biller is called 0 times
- **AC3 (happy path, release and refund):**
  - **Given** a payment in Failed with a hold of 352.50 EGP
  - **When** the failure is recorded
  - **Then** immediately held is reduced by 352.50, available is increased by 352.50, 1 ledger entry is created and the status becomes Refunded
- **AC4 (edge case, Unknown keeps hold):**
  - **Given** a payment in Unknown with a hold of 352.50 EGP
  - **When** time passes
  - **Then** the hold stays unchanged until the status is resolved

## US-16 | FR-13 | Priority: Must

**Story:** As a system, I want a payment to become Unknown when the biller does not answer in time, so that the result is not guessed.

**Acceptance criteria**

- **AC1 (failure path, timeout on a live biller):**
  - **Given** a payment in SentToBiller to a live biller
  - **When** no answer arrives within 10 seconds
  - **Then** the status becomes Unknown
- **AC2 (happy path, answer in time):**
  - **Given** the biller answers within 10 seconds
  - **When** the answer arrives
  - **Then** the status becomes Completed or Failed and not Unknown
- **AC3 (failure path, ElectroCo takes 12 seconds, hard case 2):**
  - **Given** ElectroCo takes 12 seconds to answer a payment with a hold of 352.50 EGP
  - **When** the 10-second timeout passes
  - **Then** the status becomes Unknown, the customer is told the payment is being processed, and the hold of 352.50 EGP stays held
- **AC4 (failure path, TopUpNet callback never arrives):**
  - **Given** a TopUpNet payment in SentToBiller after the reply "accepted"
  - **When** no callback arrives within 30 minutes (configurable)
  - **Then** the status becomes Unknown and the hold stays held

## US-17 | FR-13 | Priority: Must

**Story:** As a system, I want to re-query Unknown payments at 1, 5 and 15 minutes, so that they reach a final result.

_The times are counted from the moment the payment became Unknown (cumulative)._

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

- **AC1 (happy path, first "success" callback):**
  - **Given** a callback "success" with transaction reference R arrives for the first time
  - **When** it is received
  - **Then** it is processed once, the payment becomes Completed and exactly 1 result is recorded for R
- **AC2 (edge case, duplicate, hard case 6):**
  - **Given** a callback for R was already processed
  - **When** the same callback arrives again
  - **Then** it is ignored and logged (1 log entry), and the balance, ledger entries and statuses are unchanged
- **AC3 (edge case, simultaneous duplicates):**
  - **Given** 2 identical callbacks for R arrive at the same time
  - **When** both are received
  - **Then** exactly 1 is processed and 1 is ignored and logged
- **AC4 (failure path, "failed" callback):**
  - **Given** a TopUpNet payment in SentToBiller with a hold of 100.00 EGP plus its fee
  - **When** a callback "failed" arrives for its reference
  - **Then** the status becomes Failed and then Refunded immediately, and the hold is released to the wallet

## US-19 | FR-9, FR-12 | Priority: Must

**Story:** As a system, I want to process payments that arrive at the same moment from one wallet safely, so that the balance never goes negative and the customer is never charged more than they have.

_Covers hard case 4._

**Acceptance criteria**

- **AC1 (failure path, balance covers only one):**
  - **Given** a wallet with 400.00 EGP available and 2 payments with a total of 352.50 EGP each arriving at the same moment
  - **When** both are processed
  - **Then** exactly 1 payment reaches FundsHeld and the other is Rejected with the reason "insufficient funds"
- **AC2 (edge case, balance never negative):**
  - **Given** the same 2 payments
  - **When** both have been processed
  - **Then** available is 47.50 EGP, held is 352.50 EGP, available is never below 0.00 at any moment, and the balance equals the sum of the ledger entries
- **AC3 (happy path, balance covers both):**
  - **Given** a wallet with 1,000.00 EGP available and 2 payments with a total of 352.50 EGP each arriving at the same moment
  - **When** both are processed
  - **Then** both reach FundsHeld, available is 295.00 EGP and held is 705.00 EGP

## US-20 | FR-9, FR-12 | Priority: Must

**Story:** As a system, I want to turn the hold into a final charge when the biller confirms, credit PayLink's fee income and record the agent's commission, so that every completed payment is settled correctly.

_The related fee and commission rules are in section 2.6 of the brief (FR-16 and FR-17). The commission is paid out of PayLink's fee income._

**Acceptance criteria**

- **AC1 (happy path, customer payment):**
  - **Given** a payment in SentToBiller with a hold of 352.50 EGP (bill 350.00 plus fee 2.50) and no agent involved
  - **When** the biller confirms
  - **Then** the status becomes Completed, held is reduced by 352.50, available is unchanged, PayLink fee income is credited 2.50 EGP and no commission is recorded
- **AC2 (happy path, agent payment):**
  - **Given** the same payment made at an agent's terminal
  - **When** the biller confirms
  - **Then** the status becomes Completed, the agent's wallet is credited a commission of 1.00 EGP (40% of the 2.50 fee) and PayLink keeps 1.50 EGP of the fee as income
- **AC3 (failure path, biller refuses):**
  - **Given** a payment in SentToBiller with a hold of 352.50 EGP
  - **When** the biller refuses
  - **Then** the status becomes Failed, no fee income is credited, no commission is recorded and the hold is released
- **AC4 (edge case, duplicate confirmation):**
  - **Given** a payment already Completed
  - **When** the biller confirms again
  - **Then** the confirmation is ignored and 0 additional ledger entries, fee income or commissions are created

## US-21 -> FR-15 (Must)

**Story:** As a customer, I want to pay bills at ElectroCo, AquaCo and TopUpNet, so that I can pay all my utilities from one wallet.

- **ElectroCo:** Given 1,000.00 EGP available and a 350.00 EGP bill / When I pay / Then the payment is Completed and 352.50 EGP (350.00 + 2.50) is charged.
- **AquaCo:** Given a 100.00 EGP bill / When I pay / Then PayLink.BillerIntegration sends the SOAP-style XML request and the payment is Completed.
- **TopUpNet:** Given a 100.00 EGP top-up / When the biller replies "accepted" / Then the payment is SentToBiller; when the callback "success" arrives it becomes Completed.
- **Failure (outage):** Given the ElectroCo mock is stopped / When I pay a 100.00 EGP AquaCo bill / Then it is Completed.
- **Failure (disabled biller):** Given a biller disabled in PayLink.Catalog / When I try to pay / Then no payment and no hold are created.
- **Failure (overpayment):** Given a 100.00 EGP AquaCo bill / When the amount is 100.01 EGP / Then AquaCo refuses, the payment becomes Failed and the hold is released.
- **Hard case 8:** Given a new biller is added / Then only a new BillerIntegration adapter and configuration are added; 0 files change in other modules (git diff).

## US-22 -> FR-16 (Must)

**Story:** As a customer, I want fees calculated by clear rules, including promotions, so that I see exactly what I pay before confirming.

- **Fixed:** 350.00 EGP electricity or water bill -> fee 2.50, total 352.50.
- **Percentage:** 100.00 EGP top-up -> fee 1.50 (1.5%).
- **Minimum:** 50.00 EGP top-up -> fee 1.00 (1.5% = 0.75, minimum 1 applies).
- **Tiered:** 200.00 EGP internet -> fee 2.00; 200.01 EGP -> fee 4.00.
- **Promotion:** Given a promotion on one biller with start and end dates / When a payment is made inside the dates / Then fee = 0.00; the day after the end date the normal fee applies.
- **Rounding:** 33.33 EGP top-up -> fee 1.00 (1.5% = 0.49995, minimum applies); 100.10 EGP -> fee 1.50 (1.5015 rounded half up). Rounding is half up after the minimum rule (see decision 24).
- **Failure (invalid rule):** Given a fee rule with a negative value or an end date before its start date / When an admin saves it / Then it is rejected and the old rule stays active.

## US-23 -> FR-17 (Should)

**Story:** As an agent, I want to pay a bill on behalf of a customer and earn commission, so that I am paid for serving customers at my terminal.

- **Happy path:** Given an agent ElectroCo payment of 350.00 EGP with fee 2.50 / When it becomes Completed / Then the agent's wallet is credited 1.00 EGP (40% of 2.50), PayLink keeps 1.50 EGP, and a commission record exists. (Completion ledger entries are covered by US-20; not repeated here.)
- **Not before completion:** In FundsHeld, SentToBiller or Unknown the agent's balance is unchanged.
- **Failure (payment fails):** Given an agent payment that becomes Failed / Then no commission is credited and the hold is released.
- **Failure (unauthorized):** Given a user who is not an agent submits an agent payment / Then it is refused and no payment is created.
- **Zero fee:** Given a promotion makes the fee 0.00 / When an agent payment completes / Then commission = 0.00 and no commission record is created.

## US-24 -> FR-18 (Must)

**Story:** As a customer, I want payments above my daily limit rejected with a clear reason, so that I understand why a payment did not go through.
Min/max (5 / 10,000 EGP) is tested in US-12 and not repeated here. Refused amounts create no payment (error with reason, no Rejected state).

- **Daily limit boundary:** Given a basic account with 4,700.00 EGP counted today / When it submits a bill amount of 300.00 / Then it is accepted (total exactly 5,000.00); given 300.01 / Then Rejected with reason "daily limit of 5,000 EGP exceeded".
- **Basic:** Given 4,800.00 counted / When it submits 300.00 / Then Rejected with reason "daily limit of 5,000 EGP exceeded".
- **Verified:** Given 29,800.00 counted / When it submits 300.00 / Then Rejected with reason "daily limit of 30,000 EGP exceeded".
- **What counts:** Completed, FundsHeld, SentToBiller and Unknown payments, by bill amount (no fee), on a Cairo calendar day.
- **No side effects:** A Rejected payment creates no hold and leaves balances unchanged.

## US-25 -> FR-19 (Should)

**Story:** As PayLink, I want an account blocked when it makes too many payments too quickly, so that fraud and runaway clients are stopped.

- **Happy path:** Given 5 payments within 60 seconds / When the 6th is submitted in the same 60 seconds / Then it is Rejected and the account is blocked for 10 minutes.
- **Boundary:** Exactly 5 payments in 60 seconds -> none blocked.
- **During block:** Blocked at 10:00:00, payment at 10:09:59 -> Rejected with reason "account blocked".
- **After block:** Payment at 10:10:01 -> the velocity block no longer applies.
- **Window:** The 6th payment 61 seconds after the first -> not blocked.
- Rejected attempts count toward the rule. Values (5, 60 s, 10 min) are read from configuration.

## US-26 -> FR-20 (Must)

**Story:** As a support employee, I want to search a payment by reference and see its full history, so that I can resolve customer complaints.

- **Happy path:** Given R1 went Initiated -> FundsHeld -> SentToBiller -> Unknown -> Completed / When support searches R1 / Then every state change shows in order with timestamps, amounts, hold and ledger entries, and the biller reference.
- **Failure (not found):** Unknown reference -> "not found", no data from other payments.
- **Failure (unauthorized):** A customer calls the search -> refused (403).
- **Audit:** Each search writes an audit entry (who, when).

## US-27 -> FR-21 (Should)

**Story:** As a support employee, I want to block and unblock an account, so that I can stop suspicious activity or restore access.

- **Block:** Given an active account / When support blocks it with a reason / Then a new payment is Rejected with reason "account blocked".
- **Unblock:** Given a blocked account / When support unblocks it / Then the next valid payment is accepted.
- **Failure (no reason):** Empty reason -> refused, state unchanged.
- **Failure (unauthorized):** Customer calls block or unblock -> refused.
- **Audit:** Who, when and reason are recorded.

## US-28 -> FR-22 (Must)

**Story:** As a customer, I want an SMS and a push message when my payment completes or fails, so that I know the result without opening the app.

- **Completion:** Payment Completed -> one SMS and one push with amount and reference.
- **Failure:** Payment Failed (e.g. AquaCo error code) -> SMS and push with a clear reason.
- **Failure (SMS provider down):** Payment still ends Completed and the push is still sent.
- **One per outcome:** A duplicate TopUpNet "success" callback sends no second notification.
- **Unknown:** Customer is told it is being processed; no completion message yet.

## US-29 -> FR-23 (Should)

**Story:** As PayLink, I want failed notifications retried and finally marked undelivered, so that messages are not lost silently.

- **Retry then success:** First attempt fails, retry 2 succeeds -> Delivered, no more attempts.
- **Exhaustion:** First attempt and all 3 retries fail (4 attempts) -> Undelivered.
- **Payment unaffected:** Undelivered leaves the payment state unchanged.
- **Record:** Each attempt stores number, time and error.

## US-30 -> FR-24 (Should)

**Story:** As an admin, I want to edit fee and limit values without redeploying, so that business changes take effect quickly.

- **Limit change:** Basic daily limit 5,000 -> 4,000: the next payment is checked against 4,000 without restart.
- **Fee change:** Electricity fee 2.50 -> 3.00: a new inquiry shows 3.00.
- **In-flight:** A payment already in FundsHeld keeps its quoted fee.
- **Failure (invalid):** Negative or non-numeric value -> rejected, old value stays.
- **Failure (unauthorized):** Non-admin -> refused.
- **Audit:** Who, when, old value, new value.

## US-31 -> FR-25 (Must)

**Story:** As an auditor or admin, I want every money movement and admin action written to an audit log, so that I can see who did what, when and why.

- **Money movement:** Each ledger entry of a hold and its final charge has an audit record (who, when, why).
- **Completeness:** Over a test run of payments, refunds and top-ups, ledger entries = money-movement audit records (100%).
- **Admin action:** Disabling a biller writes an audit record.
- **Failure (audit write fails):** The money movement is not committed (same transaction); no ledger entry exists without an audit record.
- **Failure (tampering):** Editing or deleting an audit record is refused.

## US-32 -> FR-26 (Should)

**Story:** As a finance or support employee, I want a daily reconciliation report from the settlement file, so that differences are found and reviewed.

- **Match:** R1 in file at 350.00 and Completed at 350.00 -> matched.
- **Hard case 7:** R2 in file but Failed in PayLink -> flagged for support review; state not changed.
- **Unknown, in file:** R3 Unknown and listed -> Unknown -> Completed, hold becomes the final charge, customer notified.
- **Unknown, absent:** R4 Unknown and not listed -> Unknown -> Failed, then hold released (Failed -> Refunded); change recorded in the report.
- **Completed, absent:** R5 Completed and not listed -> flagged; state not changed.
- **Amount mismatch:** R6 Completed at 350.00, file says 305.00 -> flagged with both amounts.
- **Unknown reference:** File lists a reference PayLink lacks -> flagged "unknown to PayLink".
- **Failure (bad file):** File cannot be parsed -> no report, no state change, error logged.
- **Failure (repeat):** Same file processed twice -> no duplicate report entries, no state change.

## US-33 -> FR-27 (Could)

**Story:** As a customer, I want to request a refund of a completed payment, and support approves it, so that I can get my money back for a legitimate problem.

- **Request:** Completed payment of 350.00 / customer requests with a reason -> request is Pending, no money moves.
- **Approval:** Support approves -> new ledger entries return the approved amount, each with an audit record; the payment stays Completed (decision 2 in A's table).
- **Rejection:** Support rejects with a reason -> no money moves, customer notified.
- **Failure (not completed):** Payment Failed, Unknown or Rejected -> request refused.
- **Failure (duplicate):** A refund already approved -> new request refused.
- **Failure (unauthorized):** Customer approves own request -> refused.

## US-34 -> FR-28 (Could)

**Story:** As an admin, I want to view daily totals, so that I can monitor the business and biller health.

- **Happy path:** 3 Completed payments with fees 2.50, 2.50, 1.50 (one by an agent) -> payments = 3, fees earned = 6.50, commissions paid = 1.00.
- **Failures per biller:** 2 Failed AquaCo and 1 Failed ElectroCo -> AquaCo = 2, ElectroCo = 1.
- **Empty day:** All totals 0.00, no error.
- **Failure (unauthorized):** Non-admin -> refused.
- **Consistency:** Totals equal the sums from ledger entries for that day.

## Requirement to story mapping

| FR    | Requirement                                                                        | User stories                | Priority |
| ----- | ---------------------------------------------------------------------------------- | --------------------------- | -------- |
| FR-1  | A customer can register and has a basic verification level                         | US-01                       | Must     |
| FR-2  | A customer can be upgraded to verified, which raises limits                        | US-02                       | Should   |
| FR-3  | A customer can view wallet balance (available and held) and transaction history    | US-03                       | Must     |
| FR-4  | A customer can top up the wallet (simulated card or agent cash)                    | US-04, US-05                | Must     |
| FR-5  | Every balance change creates a ledger entry; balances are computed from the ledger | US-06                       | Must     |
| FR-6  | An admin can add, edit and disable billers and services                            | US-07, US-08                | Must     |
| FR-7  | A customer can see the list of billers and the input fields each requires          | US-09                       | Must     |
| FR-8  | A customer can run a bill inquiry and see the amount, fee and total before paying  | US-10                       | Must     |
| FR-9  | A customer can pay a bill from the wallet                                          | US-11, US-12, US-19, US-20  | Must     |
| FR-10 | A repeated request with the same idempotency key returns the original result       | US-13                       | Must     |
| FR-11 | A payment follows the life cycle and no other transitions                          | US-14                       | Must     |
| FR-12 | Funds are held before the biller is called and released if the payment fails       | US-15, US-19, US-20         | Must     |
| FR-13 | A payment that times out becomes Unknown and is re-queried by the retry rules      | US-16, US-17                | Must     |
| FR-14 | The system accepts asynchronous callbacks from TopUpNet and ignores duplicates     | US-18                       | Must     |
| FR-15 | At least 3 billers with different formats                                          | US-21                       | Must     |
| FR-16 | Fees follow section 2.6, including promotion                                       | US-22                       | Must     |
| FR-17 | Agent pays on behalf of a customer and earns commission                            | US-23                       | Should   |
| FR-18 | Payments above single or daily limit rejected with a reason                        | US-12, US-24                | Must     |
| FR-19 | More than 5 payments in 60 s blocks the account for 10 min                         | US-25 (also US-12 AC7, AC8) | Should   |
| FR-20 | Support searches a payment by reference                                            | US-26                       | Must     |
| FR-21 | Support blocks and unblocks an account                                             | US-27                       | Should   |
| FR-22 | SMS and push on completion and failure                                             | US-28                       | Must     |
| FR-23 | Failed notifications retried 3 times                                               | US-29                       | Should   |
| FR-24 | Admin edits fee and limit values                                                   | US-30                       | Should   |
| FR-25 | Audit log for money movements and admin actions                                    | US-31                       | Must     |
| FR-26 | Daily reconciliation report                                                        | US-32                       | Should   |
| FR-27 | Refund of a completed payment, support approves                                    | US-33                       | Could    |
| FR-28 | Admin daily totals                                                                 | US-34                       | Could    |

## Ambiguities

The questions, why they were unclear, and the answers given.

## Ambiguities

The questions, why they were unclear, and the answers given.

| #   | Question                                                                                                                                                                                                          | Related                   | Why unclear                                                                                                   | Answer given                                                                                                                                                                                                                                                                       |
| --- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------- | ------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Who performs the upgrade to Verified, and what is the verification process (documents, manual review)?                                                                                                            | FR-2                      | The brief says a customer "can be upgraded" but not by whom or how.                                           | Admin verifies, manual review.                                                                                                                                                                                                                                                     |
| 2   | What does "verification before they can enter" mean at registration (OTP, email link)?                                                                                                                            | FR-1                      | The brief only says the customer gets a basic verification level, which is about limits, not identity checks. | Verification is done by email.                                                                                                                                                                                                                                                     |
| 3   | Do the single-payment limits (5 and 10,000 EGP) change for Verified customers? The requirement says Verified "raises limits" but only gives daily numbers.                                                        | FR-2                      | "Limits" is plural but only the daily numbers are given.                                                      | Only the daily limit is changed.                                                                                                                                                                                                                                                   |
| 4a  | Is a payment below the minimum or above the maximum single-payment amount moved to Rejected, or refused before a payment is created?                                                                              | FR-9, FR-11               | The life cycle allows Rejected only for fraud checks or insufficient funds, and min/max is not a fraud check. | Min/max violations are validation: refused with a clear reason before the payment is created, and not written in the audit log.                                                                                                                                                    |
| 4b  | What happens when a payment breaks the daily limit, the velocity rule or the account is blocked?                                                                                                                  | FR-9, FR-11, FR-18, FR-19 | The brief lists these as fraud checks, and FR-18 asks for a clear reason.                                     | A payment is created and becomes Rejected with a clear reason. Rejected attempts count toward the velocity rule.                                                                                                                                                                   |
| 5   | Does the daily limit count only Completed payments, or also FundsHeld, SentToBiller and Unknown ones? Is a day a Cairo calendar day or a rolling 24 hours?                                                        | FR-2, FR-9                | The brief says "per day" without defining which payments count or when a day starts.                          | It counts all of them (FundsHeld, SentToBiller, Unknown and Completed). The day is a Cairo calendar day, resetting at midnight Cairo time.                                                                                                                                         |
| 6   | How is the fee calculated, and is total always amount plus fee?                                                                                                                                                   | FR-8, FR-16               | Section 2.6 gives fee rules per biller type, so it is unclear whether an admin sets fees per biller.          | Fees follow section 2.6 (fixed 2.50 EGP for electricity and water, 1.5% with a minimum of 1 EGP for mobile top-up, tiered for internet, and the promotion period), kept in configuration. Total = amount + fee. Editing fee values belongs to FR-24, not to biller editing.        |
| 7   | Do the limit checks (single payment 5 / 10,000 EGP and the daily limit) apply to the bill amount or to bill + fee?                                                                                                | FR-9                      | The brief says "payment" without saying whether the fee is included.                                          | The bill amount only (not the fee), for both the single-payment limits and the daily limit. The wallet must still cover the total (bill + fee).                                                                                                                                    |
| 8   | Are there top-up limits, minimums or fees, and can a customer have several pending top-ups?                                                                                                                       | FR-4                      | The brief's limits and fees are all defined for bill payments, not for wallet top-ups.                        | Limits, minimums and fees are not specified. There is no pending state in top-ups.                                                                                                                                                                                                 |
| 9   | How do agents authenticate, and is there an agent commission or float? If the fee is 0, is the agent commission 0 (40% of 0)?                                                                                     | FR-4, FR-17               | The brief gives the 40% rule but not how agents sign in or where the commission money comes from.             | Agents are authenticated users. The agent commission is 40% of the service fee when the payment is complete, paid out of PayLink's fee income (PayLink keeps 60%). If the fee is 0, the commission is 0.00 EGP.                                                                    |
| 10  | What is TopUpNet exactly, and does it handle only card top-ups? What identifies a duplicate callback (transaction id)?                                                                                            | FR-14                     | The word "top-up" is used both for wallet top-ups and for mobile top-ups.                                     | TopUpNet is the mobile top-up service.                                                                                                                                                                                                                                             |
| 11  | What does disabling a biller do to payments already in progress?                                                                                                                                                  | FR-6                      | The brief does not say what "disable" means for payments already sent.                                        | Only new payments are blocked.                                                                                                                                                                                                                                                     |
| 12  | Which fields can an admin edit, and is every edit audited?                                                                                                                                                        | FR-6, FR-24, FR-25        | The brief says "add, edit and disable" without listing the fields.                                            | The admin edits biller and service details (name, required input fields, minimum and maximum amounts, active or disabled). Every admin action is written to the audit log. Fee and limit values are edited under FR-24.                                                            |
| 13  | How many transactions does history show, and is there paging, filtering or sorting?                                                                                                                               | FR-3                      | The brief only says "transaction history".                                                                    | 10 transactions per page.                                                                                                                                                                                                                                                          |
| 14  | Does "10 seconds" apply to non-live billers? What is the timeout for them?                                                                                                                                        | FR-13                     | The brief says "10 seconds for live billers" and never defines "live" or gives other timeouts.                | The 10 seconds applies to live billers (ElectroCo, AquaCo, and TopUpNet's first reply). For TopUpNet the payment stays in SentToBiller after "accepted" and becomes Unknown if no callback arrives within 30 minutes (configurable). NetCo is optional and has no timeout defined. |
| 15  | Who triggers the Unknown to Completed/Failed update after the 3 re-queries are exhausted, and what is the manual process?                                                                                         | FR-13                     | The brief says the payment "waits for reconciliation" but not who acts.                                       | There is no manual process. The job itself is the trigger.                                                                                                                                                                                                                         |
| 16  | What happens to a hold on an Unknown payment that stays Unknown after the third re-query?                                                                                                                         | FR-12, FR-13              | The brief says Unknown is never refunded automatically but does not say what happens to the money.            | Money stays held until resolved.                                                                                                                                                                                                                                                   |
| 17  | Are notifications (SMS, push) required for payment results?                                                                                                                                                       | FR-9, FR-22               | FR-22 covers notifications but the payment requirement does not mention them.                                 | Yes, they are required.                                                                                                                                                                                                                                                            |
| 18  | Which currency rounding rule applies for exact decimals (number of places)?                                                                                                                                       | Business rules            | The brief says "exact decimal types" without the number of places.                                            | 2 decimal places.                                                                                                                                                                                                                                                                  |
| 19  | When the biller refuses, what moves the payment from Failed to Refunded, and is it immediate? Completed is final, but customers could request refunds (see stakeholders): can a refund be reached from Completed? | FR-12, FR-27              | Section 2.5 says "hold released" without a trigger or timing, and Completed is final.                         | The system does it immediately. A refund is reached from the Failed state.                                                                                                                                                                                                         |
| 20  | Are the re-query times (1, 5 and 15 minutes) cumulative or gaps between attempts?                                                                                                                                 | FR-13                     | "After 1, 5 and 15 minutes" can be read both ways.                                                            | Cumulative: counted from the moment the payment became Unknown.                                                                                                                                                                                                                    |
| 21  | When an agent tops up a customer with cash, is the agent's own wallet debited?                                                                                                                                    | FR-4                      | The brief says "agent cash" without saying where the money comes from.                                        | Yes, the agent's wallet is debited by the same amount that the customer's wallet is credited.                                                                                                                                                                                      |
| 22  | What happens to a bill inquiry when the biller does not answer or has no bill?                                                                                                                                    | FR-8                      | The brief describes the timeout only for payments.                                                            | The inquiry fails with a clear reason, no amounts are shown and no payment is created.                                                                                                                                                                                             |
| 23  | Notification retry interval                                                                                                                                                                                       | FR-23                     | The brief gives 3 retries but no interval.                                                                    | Configurable, default 1 minute; tests use a fake clock.                                                                                                                                                                                                                            |
| 24  | Rounding of percentage fees                                                                                                                                                                                       | FR-16                     | The brief says exact decimals but no rounding mode.                                                           | Round half up to 2 decimals (`MidpointRounding.AwayFromZero`), minimum fee applied after rounding.                                                                                                                                                                                 |
| 25  | Partial payment at AquaCo                                                                                                                                                                                         | FR-15                     | The brief only says AquaCo rejects amounts above the bill.                                                    | Not supported; the amount must equal the bill.                                                                                                                                                                                                                                     |
| 26  | Settlement file format                                                                                                                                                                                            | FR-26                     | The brief gives no format.                                                                                    | CSV: reference, amount, status, date; match key = payment reference.                                                                                                                                                                                                               |
| 27  | Who authenticates admin, support and agent roles                                                                                                                                                                  | FR-17, FR-20, FR-21       | The brief does not say.                                                                                       | Role claim on the token; unauthorized requests are refused (403).                                                                                                                                                                                                                  |
| 28  | The "why" in audit records for automatic actions                                                                                                                                                                  | FR-25                     | There is no human actor for retries or refunds.                                                               | Reason code plus actor "system" (e.g. "biller timeout retry", "failed payment refund").                                                                                                                                                                                            |
| 29  | Fee on in-flight payments when an admin changes a fee                                                                                                                                                             | FR-24                     | The brief does not say.                                                                                       | The fee quoted at inquiry is kept.                                                                                                                                                                                                                                                 |
| 30  | The biller changes the bill between inquiry and payment                                                                                                                                                           | FR-8, FR-9                | The brief does not cover it.                                                                                  | The payment uses the amount quoted at inquiry. If the biller refuses because the bill changed, the payment becomes Failed and the hold is released.                                                                                                                                |
| 31  | Do refused min/max requests count toward velocity?                                                                                                                                                                | FR-19                     | No payment is created for them.                                                                               | No: only created payments count, including Rejected ones (see 4b).                                                                                                                                                                                                                 |

## Non-functional requirements

| ID    | Quality         | Metric                                                                          | Target                                                     | How it is measured                                                        |
| ----- | --------------- | ------------------------------------------------------------------------------- | ---------------------------------------------------------- | ------------------------------------------------------------------------- |
| NFR-1 | Correctness     | Double charges and negative balances                                            | 0 and 0                                                    | Duplicate-request and concurrent-payment tests (hard cases 1, 4 and 6)    |
| NFR-2 | Performance     | Sustained payment throughput against the mock billers                           | 100 payments per second for 60 seconds                     | Load test with k6 or NBomber in Phase 2                                   |
| NFR-3 | Resilience      | Payments to other billers during a biller outage                                | 100% still succeed                                         | Stop the ElectroCo mock, then pay an AquaCo bill                          |
| NFR-4 | Modifiability   | Files changed outside Biller Integration and configuration when adding a biller | 0                                                          | Count changed files in the git diff (hard case 8)                         |
| NFR-5 | Auditability    | Money movements with an audit record (who, when, why)                           | 100%                                                       | Test compares the number of ledger entries with audit entries             |
| NFR-6 | Testability     | Line coverage on domain logic                                                   | At least 70%                                               | Coverlet report produced by the CI pipeline                               |
| NFR-7 | Reproducibility | A fresh clone starts the application and the database with one command          | docker compose up --build succeeds and /health returns 200 | Clean-clone test on a second machine, plus the green CI run on every push |
