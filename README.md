# Expense Tracker

> Expense Sharing Application — Backend Design

**Author:** Akhil Tuluri

## Overview

This document describes the backend design for an expense-sharing application similar to Splitwise. The system allows users to create groups, add shared expenses, split costs using multiple strategies, track balances, and settle dues with simplified balance calculations.

## Core Features

- **User Management**: Manage user profiles.
- **Group Creation**: Create and manage expense groups.
- **Expense Management**: Add expenses with equal, exact, and percentage splits.
- **Balance Tracking**: Real-time tracking of who owes whom.
- **Balance Simplification**: Algorithm to minimize the number of transactions.
- **Settlement**: Mark dues as settled.
- **Notifications**: Alerts and reminders for dues.
- **Expense Limits**: Manage spending limits.

## High-Level Architecture

`Client (Web/Mobile)` → `REST APIs (Node.js/Express)` → `Service Layer` → `MongoDB`

## Data Models

### 1. User Model

| Field | Type   | Description                    |
| :---- | :----- | :----------------------------- |
| id    | String | Unique identifier for the user |
| name  | String | User's full name               |
| email | String | Unique email address           |

### 2. Group Model

| Field   | Type   | Description                        |
| :------ | :----- | :--------------------------------- |
| id      | String | Unique group identifier            |
| name    | String | Group name (e.g., Trip, Roommates) |
| members | Array  | List of user IDs in the group      |

### 3. Expense Model

| Field        | Type   | Description                                    |
| :----------- | :----- | :--------------------------------------------- |
| groupId      | String | Group to which the expense belongs             |
| paidBy       | String | User ID of the payer                           |
| amount       | Number | Total expense amount                           |
| splitType    | Enum   | `EQUAL`, `EXACT`, or `PERCENTAGE`              |
| participants | Array  | Users involved in the split with share details |

### 4. Balance Model

| Field    | Type   | Description                   |
| :------- | :----- | :---------------------------- |
| fromUser | String | User who owes money           |
| toUser   | String | User who should receive money |
| amount   | Number | Amount owed                   |

## API Endpoints

### Create a Group

**POST** `/group`
Creates a new group where users can share expenses.

**Request:**
```json
{
  "name": "Goa Trip",
  "members": ["U1", "U2", "U3"]
}
```

**Response:**
```json
{
  "groupId": "G1",
  "message": "Group created successfully"
}
```

### Add an Expense

**POST** `/expense`
Adds a shared expense to a group and updates balances based on the split type.

**Request:**
```json
{
  "groupId": "G1",
  "paidBy": "U1",
  "amount": 3000,
  "splitType": "EQUAL",
  "participants": ["U1", "U2", "U3"]
}
```

**Response:**
```json
{
  "expenseId": "E1",
  "message": "Expense added successfully"
}
```

### Get User Balance

**GET** `/balances/:userId`
Returns all balances for a user, showing who they owe and who owes them.

**Response:**
```json
{
  "userId": "U1",
  "balances": [
    {
      "fromUser": "U2",
      "amount": 1000
    },
    {
      "fromUser": "U3",
      "amount": 1000
    }
  ]
}
```

### Settle Dues

**POST** `/settle`
Settles full or partial dues between two users.

**Request:**
```json
{
  "fromUser": "U2",
  "toUser": "U1",
  "amount": 1000
}
```

**Response:**
```json
{
  "message": "Settlement completed successfully"
}
```

## Expense Split Logic (Design Approach)

The system calculates each user's share based on the selected split type. The goal is to ensure **correctness, validation, and simplicity** before updating balances.

To ensure data consistency, expense creation and corresponding balance updates should be handled atomically.

### Equal Split
- Divide the total amount by the number of participants.
- Assign the same share to each user.

**Use Case:** Splitting a dinner bill.
**Validation:** Ensure `participantsCount > 0`.

```javascript
const share = totalAmount / participantsCount;
participants.forEach(user => {
  user.share = share;
});
```

### Exact Split
- Each user provides the exact amount they owe.
- Validate that all shares sum to the total amount.

**Use Case:** One person bought groceries ($50), another paid for transport ($30).

```javascript
const totalShares = userShares.reduce((sum, share) => sum + share, 0);
if (totalShares !== totalAmount) {
  throw new Error('Shares must sum to total amount');
}
```

### Percentage Split
- Each user is assigned a percentage of the total amount.
- Validate that all percentages sum to 100.

**Use Case:** Proportional cost sharing (e.g., 60-40 split).

```javascript
const totalPercentage = percentages.reduce((sum, p) => sum + p, 0);
if (Math.abs(totalPercentage - 100) > 0.01) {
  throw new Error('Percentages must sum to 100');
}
const shares = percentages.map(p => (totalAmount * p) / 100);
```

## Balance Simplification

The system simplifies balances to minimize transactions.

### Logic
1. Calculate each user's **net balance**:
   - **Positive**: Money to receive.
   - **Negative**: Money to pay.
2. Match debtors with creditors and transfer the minimum amount possible.

### Example
**Before:**
- A owes B: $100
- B owes C: $100

**After Simplification:**
- A owes C: $100

### Principle
Achieve the same outcome with fewer transactions.

---

## Conclusion

This design prioritizes simplicity, correctness, and clarity. The system separates API handling, business logic, and data storage while supporting flexible expense splits and simplified balance tracking. By minimizing transactions and centralizing logic, the backend is easy to maintain, scalable, and ready for real-world use.



## for detailed docs

[Docs](https://discreet-pony-99e.notion.site/Expense-Sharing-Application-Backend-Design-2d2b8566604e80c59707d98f7e71e473?source=copy_link)


