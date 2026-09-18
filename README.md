# Company Internal Complaint & Service Request Management Portal

## Implementation & Technical Design Document

**Tech Stack:** MERN\
**MongoDB \| Express.js \| React.js \| Node.js**\
**Cloudinary \| Nodemailer + Gmail SMTP \| JWT \| bcrypt**

------------------------------------------------------------------------

# 1. Problem

Company mein employees ko daily internal problems face hoti hain:

-   Laptop/Desktop issue
-   Wi-Fi/Network issue
-   Software installation
-   Access/permission issue
-   Basic HR/Admin service request

Agar employee ye problem WhatsApp, call ya email se batata hai, to:

-   request properly track nahi hoti
-   kis person ko problem assign hui pata nahi chalta
-   current status clear nahi hota
-   previous request ka record maintain karna difficult hota hai

### Actual problem we are solving

> **Employee ki internal problem ko ek centralized system mein record,
> assign aur resolve karna.**

------------------------------------------------------------------------

# 2. Solution

Hum ek internal portal create karenge:

``` text
Employee
   ↓
Complaint / Service Request Create
   ↓
Admin Request Check
   ↓
Admin → Agent ko Assign
   ↓
Agent → Problem Solve
   ↓
Agent → Resolved
   ↓
Employee → Request Close
```

Bas **ye core business problem** hai.

------------------------------------------------------------------------

# 3. Project Scope

Project ko intentionally small rakhenge.

### Actually implement karenge

-   3 user roles
-   Authentication & authorization
-   Employee request creation
-   Photo attachment
-   Cloudinary upload
-   Request ID
-   Admin assignment
-   Agent request handling
-   Request status workflow
-   Employee/Agent/Admin dashboards
-   Important email notifications
-   Basic comments/timeline
-   One small R&D feature --- smart category/priority suggestion

### Actually implement nahi karenge

-   Real-time chat / Socket.io
-   Complex SLA engine
-   Redis
-   Kafka
-   Microservices
-   Complex reporting system
-   Complicated analytics
-   Large AI system

**Goal: Small project + real company problem + proper architecture +
R&D.**

------------------------------------------------------------------------

# 4. Users & Roles

Only **3 roles**.

## Employee

Employee actual problem raise karega.

``` text
Register
Login
   ↓
Create Request
   ↓
View Own Requests
   ↓
Track Status
   ↓
Comment
   ↓
Close Request
```

## Agent

Support team ka member.

``` text
Login
   ↓
View Assigned Requests
   ↓
Start Work
   ↓
Update Status
   ↓
Add Resolution
   ↓
Resolve Request
```

Agent ko **sirf assigned requests** access hongi.

## Admin

System ka management user.

``` text
Login
   ↓
View Requests
   ↓
Create/Manage Agents
   ↓
Assign Request
   ↓
Monitor Status
```

Admin employee ki request ko suitable agent ko assign karega.

------------------------------------------------------------------------

# 5. Authentication Design

## Employee Registration

Employee khud register kar sakta hai:

``` text
Name
Email
Password
Confirm Password
       ↓
Register
```

Backend:

``` text
Validate
   ↓
Check email
   ↓
Hash password using bcrypt
   ↓
Create Employee
```

## Agent Registration

Agent ke liye **public registration nahi**.

``` text
Admin
  ↓
Create Agent
  ↓
Agent Account
  ↓
Agent Login
```

## Admin

Admin ka public registration nahi.

Initial Admin account seed/configuration se create hoga.

## Login

``` text
Email
Password
   ↓
POST /api/auth/login
   ↓
Verify credentials
   ↓
Generate JWT
   ↓
Role identify
   ↓
Dashboard
```

Role ke according:

``` text
EMPLOYEE → Employee Dashboard
AGENT → Agent Dashboard
ADMIN → Admin Dashboard
```

------------------------------------------------------------------------

# 6. Main Business Workflow

``` text
                    EMPLOYEE
                       │
                       │ Create Request
                       ↓
                     OPEN
                       │
                       │ Admin assigns
                       ↓
                   ASSIGNED
                       │
                       │ Agent starts
                       ↓
                 IN_PROGRESS
                       │
                       │ Problem solved
                       ↓
                   RESOLVED
                       │
                       │ Employee confirms
                       ↓
                    CLOSED
```

Agar problem solve nahi hui:

``` text
RESOLVED
   ↓
Employee says issue not fixed
   ↓
REOPEN
   ↓
IN_PROGRESS
```

Backend status transition ko control karega.

------------------------------------------------------------------------

# 7. Employee Request Creation

Employee dashboard par **Create Request**:

``` text
Request Type
Complaint / Service Request

Category
IT / Network / HR / Admin / Access

Title
Laptop WiFi not working

Description
WiFi connected but internet is not working.

Priority
Low / Medium / High

Attachment
Optional image

[Submit Request]
```

------------------------------------------------------------------------

# 8. Photo Upload --- Actual Implementation

Employee screenshot/photo attach karega:

``` text
wifi-error.jpg
```

Flow:

``` text
React
   ↓
multipart/form-data
   ↓
Express API
   ↓
Multer
   ↓
Cloudinary
   ↓
Image URL
   ↓
MongoDB
```

MongoDB mein image store nahi karenge. Only:

``` text
attachmentUrl
```

store hoga.

Cloudinary image storage/delivery handle karega.

------------------------------------------------------------------------

# 9. Request ID

Request create hote hi backend unique ID generate karega:

``` text
REQ-2026-0001
REQ-2026-0002
REQ-2026-0003
```

Employee aur Admin dono is ID se request identify kar sakte hain.

------------------------------------------------------------------------

# 10. Admin Assignment

``` text
Request
REQ-2026-0001
Laptop WiFi not working

Category: Network
Priority: High

Employee:
Rahul

Assign Agent:
[ Ajay - Network Support ]

[ Assign ]
```

Backend:

``` text
assignedTo = agentId
status = ASSIGNED
assignedAt = current time
```

Assignment ke baad Agent dashboard mein request show hogi.

------------------------------------------------------------------------

# 11. Agent Workflow

``` text
My Assigned Requests

REQ-2026-0001
Laptop WiFi not working
High
ASSIGNED
```

Agent:

``` text
[ Start Work ]
      ↓
IN_PROGRESS
      ↓
Investigate Problem
      ↓
Add Resolution Note
      ↓
[ Resolve ]
      ↓
RESOLVED
```

Example resolution:

> Wi-Fi driver was updated and network connectivity was restored.

------------------------------------------------------------------------

# 12. Dashboards

## Employee Dashboard

``` text
My Requests

Total
Open
In Progress
Resolved
Closed
```

Recent requests:

``` text
REQ-0001
WiFi not working
IN_PROGRESS

REQ-0002
Software installation
RESOLVED
```

## Agent Dashboard

``` text
Assigned Requests

Assigned
In Progress
Resolved
```

Agent ko sirf uske assigned requests milenge.

## Admin Dashboard

``` text
Total Requests
Open
Assigned
In Progress
Resolved
Closed
```

Admin ka main purpose:

> **Request ko monitor aur assign karna.**

Complex analytics nahi.

------------------------------------------------------------------------

# 13. Comments / Request Timeline

Separate chat system nahi banayenge.

``` text
Employee:
WiFi is still not working.

Agent:
I will check the network configuration.

Agent:
Configuration updated. Please check again.
```

Database:

``` text
comments
```

Har comment:

``` text
requestId
userId
message
createdAt
```

Ye request ka simple communication/history maintain karega.

------------------------------------------------------------------------

# 14. Email Notifications

Important updates ke liye **Nodemailer + Gmail SMTP** use karenge.

### Request Created

Employee:

> Your request REQ-2026-0001 has been created.

### Request Assigned

Employee + Agent:

> Request REQ-2026-0001 has been assigned.

### Request Resolved

Employee:

> Your request REQ-2026-0001 has been resolved.

Flow:

``` text
Backend
   ↓
Notification Service
   ↓
Nodemailer
   ↓
Gmail SMTP
   ↓
Email
```

Bas important events par email.

------------------------------------------------------------------------

# 15. R&D --- One Small Practical Feature

## Smart Category + Priority Suggestion

Employee description:

> "My laptop is connected to WiFi but internet is not working."

System:

``` text
Description
     ↓
Classification Logic
     ↓
Suggested Category → Network
Suggested Priority → Medium
```

Employee suggestion ko check karke submit karega.

Ye **suggestion** hoga, automatic final decision nahi.

Core request system AI ke bina bhi work karega.

------------------------------------------------------------------------

# 16. Database Design

Only required collections:

``` text
users
requests
comments
notifications
```

## Users

``` text
users
 ├── name
 ├── email
 ├── passwordHash
 ├── role
 ├── isActive
 └── createdAt
```

Roles:

``` text
EMPLOYEE
AGENT
ADMIN
```

## Requests

``` text
requests
 ├── requestId
 ├── title
 ├── description
 ├── requestType
 ├── category
 ├── priority
 ├── status
 ├── createdBy
 ├── assignedTo
 ├── attachmentUrl
 ├── resolutionNote
 ├── createdAt
 ├── assignedAt
 ├── resolvedAt
 └── closedAt
```

## Comments

``` text
comments
 ├── requestId
 ├── userId
 ├── message
 └── createdAt
```

## Notifications

``` text
notifications
 ├── userId
 ├── message
 ├── type
 ├── isRead
 └── createdAt
```

------------------------------------------------------------------------

# 17. API Design

## Authentication

``` text
POST /api/auth/register
POST /api/auth/login
POST /api/auth/logout
POST /api/auth/forgot-password
POST /api/auth/reset-password
```

## Requests

``` text
POST   /api/requests
GET    /api/requests
GET    /api/requests/:id

PATCH  /api/requests/:id/assign
PATCH  /api/requests/:id/status
PATCH  /api/requests/:id/reopen
```

## Comments

``` text
GET  /api/requests/:id/comments
POST /api/requests/:id/comments
```

## Agents

``` text
POST  /api/users/agents
GET   /api/users/agents
PATCH /api/users/:id
```

## Notifications

``` text
GET   /api/notifications
PATCH /api/notifications/:id/read
```

------------------------------------------------------------------------

# 18. High-Level Architecture

``` text
                         USERS
                           │
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
         Employee        Agent         Admin
             └─────────────┼─────────────┘
                           ↓
                      React.js
                           │
                        HTTPS
                           │
                           ↓
                  Node.js + Express
                           │
        ┌──────────────────┼──────────────────┐
        ↓                  ↓                  ↓
      Auth             Request Module      User Module
                           │
                 ┌─────────┼─────────┐
                 ↓         ↓         ↓
              Assign     Status   Comments
                           │
                           ↓
                        MongoDB
                           │
              ┌────────────┴────────────┐
              ↓                         ↓
         Cloudinary                 Nodemailer
         Photo Storage              Gmail SMTP
```

------------------------------------------------------------------------

# 19. Backend Structure

``` text
backend/
│
└── src/
    │
    ├── config/
    │   ├── db.js
    │   └── cloudinary.js
    │
    ├── models/
    │   ├── User.js
    │   ├── Request.js
    │   ├── Comment.js
    │   └── Notification.js
    │
    ├── controllers/
    │   ├── auth.controller.js
    │   ├── request.controller.js
    │   ├── user.controller.js
    │   ├── comment.controller.js
    │   └── notification.controller.js
    │
    ├── routes/
    │   ├── auth.routes.js
    │   ├── request.routes.js
    │   ├── user.routes.js
    │   ├── comment.routes.js
    │   └── notification.routes.js
    │
    ├── middleware/
    │   ├── auth.js
    │   ├── role.js
    │   └── upload.js
    │
    ├── services/
    │   ├── cloudinary.service.js
    │   └── email.service.js
    │
    ├── utils/
    │
    └── app.js
│
└── server.js
```

------------------------------------------------------------------------

# 20. Frontend Structure

``` text
frontend/
│
├── components/
├── pages/
│
│   ├── auth/
│   │   ├── Login.jsx
│   │   └── Register.jsx
│   │
│   ├── employee/
│   │   ├── Dashboard.jsx
│   │   ├── CreateRequest.jsx
│   │   └── RequestDetails.jsx
│   │
│   ├── agent/
│   │   ├── Dashboard.jsx
│   │   └── RequestDetails.jsx
│   │
│   └── admin/
│       ├── Dashboard.jsx
│       ├── Requests.jsx
│       └── Agents.jsx
│
├── services/
│   └── api.js
│
├── context/
├── hooks/
└── App.jsx
```

------------------------------------------------------------------------

# 21. Actual Implementation Sequence

Yahi sequence follow karenge.

### Step 1 --- Requirement Finalization

``` text
Problem
↓
Users
↓
Roles
↓
Workflow
↓
Features
```

### Step 2 --- Database Design

``` text
User Schema
Request Schema
Comment Schema
Notification Schema
```

### Step 3 --- Backend Setup

``` text
Node.js
Express
MongoDB
Environment Variables
Project Structure
```

### Step 4 --- Authentication

``` text
Register
Login
bcrypt
JWT
Middleware
Role Authorization
```

### Step 5 --- Request Module

``` text
Create Request
Request ID
Get Requests
Request Details
```

### Step 6 --- Cloudinary

``` text
Multer
   ↓
Cloudinary
   ↓
attachmentUrl
```

### Step 7 --- Admin Assignment

``` text
Admin
 ↓
View Request
 ↓
Select Agent
 ↓
Assign
```

### Step 8 --- Agent Workflow

``` text
Assigned
 ↓
In Progress
 ↓
Resolution
 ↓
Resolved
```

### Step 9 --- Employee Workflow

``` text
View Request
 ↓
Track Status
 ↓
Comment
 ↓
Close / Reopen
```

### Step 10 --- Email

``` text
Nodemailer
 ↓
Gmail SMTP
 ↓
Important notifications
```

### Step 11 --- R&D

``` text
Description
 ↓
Smart classification
 ↓
Category + Priority suggestion
```

### Step 12 --- React Frontend

``` text
Auth UI
 ↓
Employee Dashboard
 ↓
Agent Dashboard
 ↓
Admin Dashboard
 ↓
Request UI
```

### Step 13 --- Integration

``` text
React
   ↕
REST API
   ↕
Express/Node
   ↕
MongoDB
   +
Cloudinary
   +
Gmail SMTP
```

### Step 14 --- Testing

``` text
Authentication
Role access
Request creation
Image upload
Assignment
Status flow
Email
R&D
```

### Step 15 --- Deployment

``` text
Frontend → Vercel
Backend  → Render
Database → MongoDB Atlas
Images   → Cloudinary
Email    → Gmail SMTP
```

------------------------------------------------------------------------

# Final Scope

**Project principle:** Small project + real company problem + proper
architecture + one practical R&D feature.

**Team explanation sequence:**

> Problem → Proposed Solution → Users/Roles → Business Workflow →
> Requirements → Architecture → Database → APIs → Backend Implementation
> → Frontend → External Integrations → R&D → Testing → Deployment.
