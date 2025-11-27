# **Test User Management Service – README**

A centralized backend system for managing test users, tokens, environment registration, server health, and analytics for QA automation.

---

# **📘 Table of Contents**

1. [Introduction](#introduction)
2. [System Scope](#system-scope)
3. [Technical Standards](#technical-standards)
4. [Data Model](#data-model)
5. [Features & Use Cases](#features--use-cases)
6. [API Specifications](#api-specifications)

   * [Test User](#test-user)
   * [Analytics](#analytics)
   * [Health Endpoints](#health-endpoints)
   * [Token Services](#token-services)
   * [Environment Actions](#environment-actions)
7. [Concurrency Rules](#concurrency-rules)
8. [Data retension policy (Configurable)](#data-retension-policy-configurable)
9. [Authentication & Authorization](#authentication--authorization)
10. [Additional Enhancements](#additional-enhancements)
11. [Non-Functional Requirements](#non-functional-requirements)

---

# **Introduction**

The **Test User Management Service** provides unified APIs to:

* Create and manage test users
* Fetch users with **concurrency-safe** operations
* Register users in different app environments
* Mark users as onboarded/logged-in
* Generate backend and CMS tokens
* Track analytics
* Monitor service and server health

This service follows the same envelope format and standard conventions defined by the Backend team in their [WIKI](https://dev.azure.com/Sciensus/SciQ%20Digital%20Health%20Platform/_wiki/wikis/Digital.wiki/204/7.2.5-Backend-and-Frontend-messaging-structures)

---

# **System Scope**

### ✔ Test User Management

* JSON-based user storage
* Scenario tagging
* Fetch next available unconsumed user
* Concurrency-safe locking
* Update/delete/reset operations

### ✔ Token Management

* App backend tokens
* CMS tokens
* Environment-based tokens

### ✔ Environment Support

* Register user in environment backend
* Mark user logged in (skip onboarding)

### ✔ Health Tools

* DB health
* Server health
* Complete system health

### ✔ Analytics

* Total user count
* Scenario breakdown
* Consumed/unconsumed analysis

---

# **Technical Standards**

## **Request Headers**

| Header           | Type    | Description        |
| ---------------- | ------- | ------------------ |
| Accept-Language  | String  | Locale for content |
| Authorization    | String  | Bearer JWT         |
| X-Correlation-Id | String  | GUID per request   |
| X-Dry-Run        | Boolean | Default `false`    |

---

## **Request Envelopes**

### **Command**

```json
{
  "command": { }
}
```

### **Query**

```json
{
  "query": { }
}
```

---

## **Response Envelopes**

### **Base Response**

```json
{
  "errors": [],
  "warnings": [],
  "generated": null,
  "redirectUrl": null
}
```

### **Result Response**

```json
{
  "errors": [],
  "warnings": [],
  "items": [],
  "meta": {
    "pagination": {
      "count": 0,
      "skip": 0,
      "limit": 50
    }
  }
}
```

---

# **Data Model**

## **TestUser Table**

| Field     | Type      | Notes             |                             |
| --------- | --------- | ----------------- | --------------------------- |
| id        | UUID      | Primary key       |                             |
| scenario  | String    | Scenario label    |                             |
| data      | JSON     | User data         |                             |
| consumed  | Boolean   | Default: false    |                             |
| metadata  | JSONB     | Optional metadata |                             |
| createdAt | Timestamp | Auto-generated    |                             |
| updatedAt | Timestamp | Auto-generated    |                             |

---

## **Database Requirements**

* Support JSON
* Support row-level locking

---

# **Features & Use Cases**

### ✔ Create, fetch, update, delete test users

### ✔ Fetch next unconsumed user safely

### ✔ Track scenario-based analytics

### ✔ Validate DB health

### ✔ Manage tokens for multiple backend systems

### ✔ Register user in environment and skip onboarding

---

# **API Specifications**

All endpoints use the command/query envelope format.

---

# **Test User**

---

## **Create User (store the user in DB)**

**POST** `/test-user`

**Request**

```json
{
  "command": {
    "scenario": "confirm-order-flow",
    "data": {
      "email": "test@example.com",
      "password": "123456",
      "profile": {}
    }
  }
}
```

**Response**

```json
{
  "generated": "uuid"
}
```

---

## **Acquire User (Concurrency Safe)**

**POST** `/test-users/acquire`

**Request**

```json
{
  "query": {
    "scenario": "confirm-flow-user",
    "markAsConsumed": true,
    "count": 1
  }
}
```

**Response**

```json
{
  "items": [
    {
      "id": "uuid",
      "scenario": "new-user",
      "data": {},
      "consumed": true
    }
  ]
}
```

---

## **Update User**

**PATCH** `/test-user/{id}`

**Request**

```json
{
  "command": {
    "updates": {
      "data.profile.name": "Updated Name",
      "consumed": false
    }
  }
}
```
---

## **Delete User**

**DELETE** `/test-user/{id}`

---

## **Reset Consumed Users**

**POST** `/test-user/reset`

**Request**

```json
{
  "command": { "scenario": "confir-user-flow" }
}
```

---

# **Analytics**

---

## **Test User Database Analytics**

**POST** `/analytics/test-user`

**Response**

```json
{
  "items": [{
    "totalUsers": 1200,
    "consumedCount": 740,
    "unconsumedCount": 460,
    "distinctScenarios": 12,
    "scenarios": {
      "new-user": 500,
      "login-only": 200
    }
  }]
}
```

---

## **Consumed User List**

**POST** `/analytics/test-user/consumed`

**Response**

```json
{
  "items": [
    {
      "id": "uuid",
      "scenario": "new-user",
      "data": {},
      "consumed": true
    }
  ]
}
```
---

# **Health Endpoints**

---

## **DB Health**

**POST** `/health/db`

**Response**

```json
{
  "items": [
    {
      "status": "UP",
    }
  ]
}
```

---

## **System Health**

**POST** `/health/system`

Checks:

* Database
* Backend servers
* CMS
* Token services

---

## **Server Status**

**POST** `/health/servers`

**Request**

```json
{
  "query": {
    "servers": ["backend", "cms", "auth", "posthog"]
  }
}
```

**Re**

```json
{
  "query": {
    "servers": [
        {"backend": "up"}, {"cms": "up"}, {"posthog": "up"}, { ...etc }
    ]
  }
}
```
---

# **Token Services**

---

## **Get Backend Token**

**POST** `/tokens/backend`

```json
{
  "query": { "environment": "qa" }
}
```

---

## **Get CMS Token**

**POST** `/tokens/cms`

---

# **Environment Actions**

---

## **Register User in Environment DB**

**POST** `/environment/register-user`

**Request**
```json
{
  "command": {
    "userId": "UUID",
    "environment": "qa"
  }
}
```

---

## **Mark User as Logged-In (skip onboarding)**

**POST** `/:env-name/mark-logged-in`

**Request**

```json
{
  "command": {
    "userId": "UUID",
  }
}
```

---

# **Concurrency Rules**

The service MUST ensure:

### ✔ Two parallel requests NEVER get the same user

### ✔ Auto-locking and marking users consumed

### ✔ Timeout handling and retry-safe logic

---

# **Data retension policy (Configurable)**

- Any user which is not created or update in last 30 days (configurable) should get clean up from the user data base

# **Authentication & Authorization**

All endpoints except health checks must require some authentication and have a policy to define authorization:

* Require auth like **Bearer JWT**
* Recommended roles:

  * `viewer`
  * `admin`

---

# **Additional Enhancements**

### ✔ Bulk user upload support (CSV/JSON)
 - Make create/update/delet user payload to accept csv or user arrays

---

# **Non-Functional Requirements**

| Requirement | Value                   |
| ----------- | ----------------------- |
| Idle Response Time     | <5s                  |
| Concurrency | ≥50 automation threads  |
| Logging     | Correlation ID required |
| Backups     | Weekly                   |
