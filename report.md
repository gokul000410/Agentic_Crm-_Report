# Agentic CRM — Architecture and PostgreSQL Database Schema

## 1. Introduction

A Customer Relationship Management (CRM) system is used to manage information and interactions related to leads, customers, companies, sales activities, marketing campaigns, and follow-ups.

An **Agentic CRM** extends a traditional CRM by integrating AI agents that can understand business data, reason about tasks, use tools, and perform multi-step actions with limited human intervention.

A traditional CRM mainly helps users **store, manage, search, and analyze information**.

An Agentic CRM can additionally:

* Analyze incoming leads
* Qualify leads
* Calculate lead scores
* Analyze customer history
* Generate personalized communication
* Schedule follow-ups
* Update CRM records
* Search company knowledge
* Recommend next actions
* Execute approved business workflows

The basic concept is:

```text
Traditional CRM
User → CRM → Database

Agentic CRM
User → CRM → AI Agent → LLM → Tools → Database / External Services
```

---

# 2. Objective of the Agentic CRM

The main objective is to reduce repetitive manual CRM operations and allow AI agents to assist employees in customer and sales workflows.

The system should be able to:

1. Capture leads.
2. Store customer and company information.
3. Analyze lead quality.
4. Understand customer history.
5. Recommend or execute follow-up actions.
6. Manage marketing campaigns.
7. Generate personalized emails.
8. Retrieve information from company documents using RAG.
9. Maintain an audit trail of AI actions.
10. Allow humans to approve sensitive actions.

---

# 3. High-Level Architecture

The proposed architecture consists of several layers.

```text
                         ┌───────────────────────┐
                         │        USERS          │
                         │ Sales / Admin /       │
                         │ Marketing / Manager   │
                         └───────────┬───────────┘
                                     │
                                     ▼
                         ┌───────────────────────┐
                         │    React Frontend     │
                         │                       │
                         │ Dashboard             │
                         │ Leads                 │
                         │ Contacts              │
                         │ Campaigns             │
                         │ AI Assistant          │
                         └───────────┬───────────┘
                                     │
                                  REST API
                                     │
                                     ▼
                         ┌───────────────────────┐
                         │    FastAPI Backend    │
                         │                       │
                         │ Authentication        │
                         │ CRM APIs              │
                         │ Validation            │
                         │ Business Logic        │
                         └───────────┬───────────┘
                                     │
                                     ▼
                    ┌────────────────────────────────┐
                    │       AGENT ORCHESTRATOR       │
                    │            LangGraph           │
                    └───────────────┬────────────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              │                     │                     │
              ▼                     ▼                     ▼
       ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
       │ Lead Agent   │      │Customer Agent│      │Campaign Agent│
       └──────┬───────┘      └──────┬───────┘      └──────┬───────┘
              │                     │                     │
              └─────────────────────┼─────────────────────┘
                                    ▼
                           ┌─────────────────┐
                           │    LLM / SLM    │
                           │                 │
                           │ Reasoning       │
                           │ Classification  │
                           │ Generation      │
                           └────────┬────────┘
                                    │
                              Tool Calling
                                    │
          ┌─────────────────────────┼────────────────────────┐
          │                         │                        │
          ▼                         ▼                        ▼
   ┌──────────────┐        ┌─────────────────┐       ┌──────────────┐
   │ CRM Tools    │        │ Communication   │       │ RAG Tools    │
   │              │        │ Tools           │       │              │
   │ get_lead()   │        │ send_email()    │       │ Search docs  │
   │ update_lead()│        │ send_sms()      │       │ Retrieve FAQ │
   │ create_task()│        │ schedule_call() │       │ Product info │
   └──────┬───────┘        └────────┬────────┘       └──────┬───────┘
          │                          │                       │
          ▼                          ▼                       ▼
   ┌──────────────┐        ┌─────────────────┐       ┌──────────────┐
   │ PostgreSQL   │        │ Email / SMS API │       │  pgvector    │
   │ CRM Database │        │                 │       │ Vector Store │
   └──────────────┘        └─────────────────┘       └──────────────┘
```

---

# 4. Architecture Layers

## 4.1 Frontend Layer

### Technology

React

The frontend provides the interface for CRM users.

Main modules:

```text
Dashboard
Leads
Contacts
Companies
Campaigns
Deals
Tasks
Activities
AI Assistant
Analytics
```

---

## 4.2 Backend Layer

### Technology

Python + FastAPI

FastAPI provides REST APIs between the frontend, AI agents, and database.

Example APIs:

```text
POST   /api/leads
GET    /api/leads
GET    /api/leads/{id}
PUT    /api/leads/{id}

POST   /api/contacts
GET    /api/contacts/{id}

POST   /api/campaigns
GET    /api/campaigns

POST   /api/tasks
GET    /api/activities
```

The backend handles:

* Authentication
* Authorization
* Validation
* CRUD operations
* Business logic
* AI requests
* Database communication

---

## 4.3 Agent Orchestration Layer

### Technology

LangGraph

LangGraph can be used to control the workflow of AI agents.

Example:

```text
New Lead
   ↓
Analyze Lead
   ↓
Retrieve Customer History
   ↓
Calculate Lead Score
   ↓
Decision
   │
   ├── High Score
   │      ↓
   │   Sales Follow-up
   │
   └── Low Score
          ↓
      Nurture Campaign
```

The orchestrator determines which agent or tool should execute the next step.

---

## 4.4 AI Agent Layer

Different agents can specialize in different CRM operations.

### Lead Qualification Agent

Responsibilities:

* Analyze lead information
* Calculate lead score
* Determine lead priority
* Recommend next action

### Customer Agent

Responsibilities:

* Retrieve customer history
* Summarize interactions
* Identify customer requirements
* Provide customer context

### Campaign Agent

Responsibilities:

* Analyze campaigns
* Segment customers
* Generate campaign content
* Analyze campaign results

### Follow-up Agent

Responsibilities:

* Identify pending follow-ups
* Create tasks
* Generate follow-up messages
* Schedule activities

---

## 4.5 LLM / SLM Layer

The LLM or SLM provides language understanding and generation.

It can perform:

* Classification
* Summarization
* Information extraction
* Email generation
* Natural language understanding
* Reasoning over retrieved information

Example:

```text
Lead Information
       ↓
      LLM
       ↓
Lead Score = 87
Priority = High
Next Action = Sales Call
```

---

## 4.6 Tool Layer

Agents should interact with the system through controlled tools rather than directly executing arbitrary database operations.

Example tools:

```text
get_lead()
search_leads()
get_customer_history()
update_lead()
create_task()
create_activity()
send_email()
schedule_followup()
search_knowledge_base()
```

Example:

```text
Agent:
"I need the customer's previous interactions."

        ↓

get_customer_history(contact_id)

        ↓

PostgreSQL

        ↓

Customer history

        ↓

Agent continues workflow
```

---

## 4.7 Database Layer

### Technology

PostgreSQL

PostgreSQL stores the main CRM information:

```text
Users
Companies
Contacts
Leads
Campaigns
Activities
Tasks
Deals
Messages
Agents
Agent Runs
Agent Tool Calls
```

PostgreSQL can also be extended with **pgvector** for vector similarity search.

---

## 4.8 RAG Layer

The CRM can use RAG to retrieve information from company documents.

Example documents:

```text
Product Documentation
Pricing
FAQs
Sales Guidelines
Company Policies
Technical Documentation
```

Workflow:

```text
Customer Question
       ↓
Create Embedding
       ↓
Search pgvector
       ↓
Retrieve Relevant Documents
       ↓
LLM
       ↓
Generate Answer
```

---

## 4.9 Communication Layer

Agents can use external APIs for communication.

```text
Agent
 │
 ├── Email → SMTP / Email API
 │
 ├── SMS → SMS API
 │
 └── Other approved communication APIs
```

---

# 5. Example Agentic CRM Workflow

Suppose a customer submits a website form:

```text
Name: Rahul
Company: ABC Technologies
Requirement: AI Chatbot
Budget: ₹5,00,000
```

The workflow can be:

```text
Website Form
     ↓
FastAPI
     ↓
Create Lead
     ↓
Lead Qualification Agent
     ↓
LLM
     ↓
Analyze Lead
     ↓
Lead Score = 87
     ↓
Qualified
     ↓
Customer Agent
     ↓
Retrieve Customer History
     ↓
Follow-up Agent
     ↓
Generate Personalized Email
     ↓
Email Tool
     ↓
Email API
     ↓
Customer
     ↓
Create Activity
     ↓
Schedule Follow-up
```

---

# 6. PostgreSQL Database Schema

The database can be divided into two major groups.

### CRM entities

```text
users
companies
contacts
leads
campaigns
campaign_leads
activities
tasks
deals
messages
```

### AI/Agent entities

```text
agents
agent_runs
agent_tools
agent_tool_calls
knowledge_documents
document_chunks
approval_requests
```

---

# 7. Entity Relationship Overview

```text
                         USERS
                           │
             ┌─────────────┼──────────────┐
             │             │              │
             ▼             ▼              ▼
         CAMPAIGNS       LEADS           TASKS
             │             │
             │             ├──────────────┐
             │             │              │
             ▼             ▼              ▼
     CAMPAIGN_LEADS     CONTACTS      ACTIVITIES
                           │
                           ▼
                       COMPANIES
                           │
                           ▼
                         DEALS


                         AGENTS
                           │
                           ▼
                      AGENT_RUNS
                           │
                    ┌──────┴───────┐
                    ▼              ▼
             TOOL_CALLS      APPROVAL_REQUESTS
                    │
                    ▼
                AGENT_TOOLS


             KNOWLEDGE_DOCUMENTS
                    │
                    ▼
              DOCUMENT_CHUNKS
                    │
                    ▼
                 PGVECTOR
```

---

# 8. SQL Schema

## 8.1 Users

```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    role VARCHAR(50) NOT NULL DEFAULT 'sales',
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

---

## 8.2 Companies

```sql
CREATE TABLE companies (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    industry VARCHAR(100),
    website VARCHAR(255),
    phone VARCHAR(30),
    company_size INTEGER,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

---

## 8.3 Contacts

```sql
CREATE TABLE contacts (
    id BIGSERIAL PRIMARY KEY,
    company_id BIGINT REFERENCES companies(id) ON DELETE SET NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100),
    email VARCHAR(255),
    phone VARCHAR(30),
    job_title VARCHAR(150),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

---

## 8.4 Leads

```sql
CREATE TABLE leads (
    id BIGSERIAL PRIMARY KEY,

    contact_id BIGINT REFERENCES contacts(id)
        ON DELETE SET NULL,

    company_id BIGINT REFERENCES companies(id)
        ON DELETE SET NULL,

    source VARCHAR(100),
    status VARCHAR(50) DEFAULT 'new',

    lead_score INTEGER CHECK (
        lead_score >= 0 AND lead_score <= 100
    ),

    intent_score NUMERIC(5,4),

    estimated_budget NUMERIC(15,2),

    assigned_to BIGINT REFERENCES users(id)
        ON DELETE SET NULL,

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,

    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

---

# 9. Campaigns

```sql
CREATE TABLE campaigns (
    id BIGSERIAL PRIMARY KEY,

    name VARCHAR(255) NOT NULL,

    campaign_type VARCHAR(50),

    description TEXT,

    status VARCHAR(50) DEFAULT 'draft',

    start_date DATE,
    end_date DATE,

    created_by BIGINT REFERENCES users(id)
        ON DELETE SET NULL,

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,

    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

---

# 10. Campaign Leads

This represents a many-to-many relationship between campaigns and leads.

```sql
CREATE TABLE campaign_leads (
    campaign_id BIGINT REFERENCES campaigns(id)
        ON DELETE CASCADE,

    lead_id BIGINT REFERENCES leads(id)
        ON DELETE CASCADE,

    status VARCHAR(50) DEFAULT 'added',

    joined_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,

    PRIMARY KEY (campaign_id, lead_id)
);
```

Relationship:

```text
Campaign N ───────── N Lead
```

---

# 11. Activities

```sql
CREATE TABLE activities (
    id BIGSERIAL PRIMARY KEY,

    lead_id BIGINT REFERENCES leads(id)
        ON DELETE CASCADE,

    contact_id BIGINT REFERENCES contacts(id)
        ON DELETE SET NULL,

    user_id BIGINT REFERENCES users(id)
        ON DELETE SET NULL,

    activity_type VARCHAR(50) NOT NULL,

    subject VARCHAR(255),

    description TEXT,

    status VARCHAR(50) DEFAULT 'pending',

    scheduled_at TIMESTAMP WITH TIME ZONE,

    completed_at TIMESTAMP WITH TIME ZONE,

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

Examples:

```text
email
call
meeting
sms
follow_up
note
ai_action
```

---

# 12. Deals

```sql
CREATE TABLE deals (
    id BIGSERIAL PRIMARY KEY,

    lead_id BIGINT REFERENCES leads(id)
        ON DELETE SET NULL,

    company_id BIGINT REFERENCES companies(id)
        ON DELETE SET NULL,

    title VARCHAR(255) NOT NULL,

    amount NUMERIC(15,2),

    stage VARCHAR(100),

    probability NUMERIC(5,2),

    expected_close_date DATE,

    assigned_to BIGINT REFERENCES users(id)
        ON DELETE SET NULL,

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,

    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

---

# 13. Tasks

```sql
CREATE TABLE tasks (
    id BIGSERIAL PRIMARY KEY,

    lead_id BIGINT REFERENCES leads(id)
        ON DELETE CASCADE,

    assigned_to BIGINT REFERENCES users(id)
        ON DELETE SET NULL,

    title VARCHAR(255) NOT NULL,

    description TEXT,

    status VARCHAR(50) DEFAULT 'pending',

    priority VARCHAR(30) DEFAULT 'medium',

    due_date TIMESTAMP WITH TIME ZONE,

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,

    completed_at TIMESTAMP WITH TIME ZONE
);
```

The AI agent can automatically create tasks such as:

```text
"Call Rahul within 24 hours."
```

---

# 14. Messages

```sql
CREATE TABLE messages (
    id BIGSERIAL PRIMARY KEY,

    lead_id BIGINT REFERENCES leads(id)
        ON DELETE CASCADE,

    contact_id BIGINT REFERENCES contacts(id)
        ON DELETE SET NULL,

    channel VARCHAR(50) NOT NULL,

    direction VARCHAR(20) NOT NULL,

    subject VARCHAR(255),

    message TEXT,

    status VARCHAR(50),

    sent_at TIMESTAMP WITH TIME ZONE,

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

---

# 15. AI Agents

This table stores the AI agents available in the system.

```sql
CREATE TABLE agents (
    id BIGSERIAL PRIMARY KEY,

    name VARCHAR(100) NOT NULL,

    description TEXT,

    agent_type VARCHAR(100),

    model_name VARCHAR(100),

    is_active BOOLEAN DEFAULT TRUE,

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

Examples:

```text
Lead Qualification Agent
Customer Agent
Campaign Agent
Follow-up Agent
```

---

# 16. Agent Runs

An agent run represents one execution of an AI agent.

```sql
CREATE TABLE agent_runs (
    id BIGSERIAL PRIMARY KEY,

    agent_id BIGINT REFERENCES agents(id)
        ON DELETE SET NULL,

    lead_id BIGINT REFERENCES leads(id)
        ON DELETE SET NULL,

    contact_id BIGINT REFERENCES contacts(id)
        ON DELETE SET NULL,

    task TEXT NOT NULL,

    input_data JSONB,

    output_data JSONB,

    status VARCHAR(50) DEFAULT 'running',

    started_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,

    completed_at TIMESTAMP WITH TIME ZONE,

    error_message TEXT
);
```

Example input:

```json
{
    "budget": 500000,
    "industry": "Technology",
    "requirement": "AI chatbot"
}
```

Example output:

```json
{
    "lead_score": 87,
    "classification": "high_value",
    "next_action": "sales_call"
}
```

---

# 17. Agent Tools

```sql
CREATE TABLE agent_tools (
    id BIGSERIAL PRIMARY KEY,

    name VARCHAR(100) UNIQUE NOT NULL,

    description TEXT NOT NULL,

    tool_type VARCHAR(50),

    is_active BOOLEAN DEFAULT TRUE,

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

Example tools:

```text
get_lead
search_leads
get_customer_history
update_lead
create_task
send_email
schedule_followup
search_knowledge_base
```

---

# 18. Agent Tool Calls

This table records which tools an agent actually used.

```sql
CREATE TABLE agent_tool_calls (
    id BIGSERIAL PRIMARY KEY,

    agent_run_id BIGINT REFERENCES agent_runs(id)
        ON DELETE CASCADE,

    tool_id BIGINT REFERENCES agent_tools(id)
        ON DELETE SET NULL,

    arguments JSONB,

    result JSONB,

    status VARCHAR(50),

    started_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,

    completed_at TIMESTAMP WITH TIME ZONE
);
```

Example:

```text
Agent Run #5001

1. get_lead(101)
2. get_customer_history(25)
3. update_lead(101)
4. send_email(...)
5. create_task(101)
```

---

# 19. Knowledge Documents

For RAG:

```sql
CREATE TABLE knowledge_documents (
    id BIGSERIAL PRIMARY KEY,

    title VARCHAR(255) NOT NULL,

    content TEXT NOT NULL,

    document_type VARCHAR(100),

    metadata JSONB,

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,

    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

---

# 20. Document Chunks

Documents are split into smaller chunks for retrieval.

```sql
CREATE TABLE document_chunks (
    id BIGSERIAL PRIMARY KEY,

    document_id BIGINT REFERENCES knowledge_documents(id)
        ON DELETE CASCADE,

    chunk_index INTEGER NOT NULL,

    content TEXT NOT NULL,

    metadata JSONB,

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

If pgvector is used:

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

Then:

```sql
ALTER TABLE document_chunks
ADD COLUMN embedding VECTOR(1536);
```

The exact vector dimension should match the embedding model being used.

---

# 21. Human Approval

Sensitive actions can require human approval.

```sql
CREATE TABLE approval_requests (
    id BIGSERIAL PRIMARY KEY,

    agent_run_id BIGINT REFERENCES agent_runs(id)
        ON DELETE CASCADE,

    action_type VARCHAR(100) NOT NULL,

    requested_action JSONB,

    status VARCHAR(50) DEFAULT 'pending',

    reviewed_by BIGINT REFERENCES users(id)
        ON DELETE SET NULL,

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,

    reviewed_at TIMESTAMP WITH TIME ZONE
);
```

Example:

```text
AI Agent
   ↓
Generate campaign
   ↓
Approval Required
   ↓
Marketing Manager
   ↓
Approve
   ↓
Send Campaign
```

---

# 22. Complete Database Structure

The final database can be represented as:

```text
CRM DATABASE
│
├── USERS
│
├── COMPANIES
│     │
│     └── CONTACTS
│            │
│            └── LEADS
│                  │
│                  ├── ACTIVITIES
│                  ├── TASKS
│                  ├── DEALS
│                  ├── MESSAGES
│                  └── CAMPAIGN_LEADS
│
├── CAMPAIGNS
│
└── AI SYSTEM
      │
      ├── AGENTS
      │      │
      │      └── AGENT_RUNS
      │             │
      │             └── AGENT_TOOL_CALLS
      │
      ├── AGENT_TOOLS
      │
      ├── APPROVAL_REQUESTS
      │
      └── RAG
             │
             ├── KNOWLEDGE_DOCUMENTS
             │
             └── DOCUMENT_CHUNKS
                         │
                         └── EMBEDDINGS
```

---

# 23. Complete Technology Stack

| Component            | Technology             |
| -------------------- | ---------------------- |
| Frontend             | React                  |
| Backend              | FastAPI                |
| Programming Language | Python                 |
| Database             | PostgreSQL             |
| ORM                  | SQLAlchemy             |
| Agent Orchestration  | LangGraph              |
| LLM                  | OpenAI API / Other LLM |
| SLM                  | Optional local SLM     |
| Vector Search        | pgvector               |
| RAG                  | LangChain / LangGraph  |
| Cache                | Redis                  |
| Background Jobs      | Celery                 |
| Authentication       | JWT                    |
| Email                | SMTP / Email API       |
| Containerization     | Docker                 |
| Deployment           | AWS                    |

For the first POC, the stack can be simplified to:

```text
React
   ↓
FastAPI
   ↓
LangGraph
   ↓
LLM
   ↓
PostgreSQL + pgvector
   ↓
Email API
```

---

# 24. Security Architecture

The AI agent should not have unrestricted access to the database.

Instead:

```text
AI Agent
    ↓
Controlled Tool
    ↓
Backend Validation
    ↓
Authorization
    ↓
Database
```

For example, instead of allowing:

```text
AI → arbitrary SQL
```

use:

```text
AI
 ↓
update_lead(lead_id, status)
 ↓
FastAPI validates request
 ↓
Permission check
 ↓
PostgreSQL
```

Security mechanisms should include:

* JWT authentication
* Role-based access control
* Input validation
* API authentication
* Database permissions
* Audit logs
* Agent execution logs
* Tool permissions
* Human approval for sensitive actions
* Environment variables for secrets

---

# 25. Example End-to-End Flow

A complete example:

```text
1. Customer fills website form
            ↓
2. FastAPI receives lead
            ↓
3. Lead stored in PostgreSQL
            ↓
4. Lead Agent starts
            ↓
5. Agent retrieves customer information
            ↓
6. LLM analyzes lead
            ↓
7. Lead score calculated
            ↓
8. Agent decides next action
            ↓
9. Customer Agent retrieves history
            ↓
10. Follow-up Agent generates email
            ↓
11. Human approval if required
            ↓
12. Email Tool sends email
            ↓
13. Activity stored in PostgreSQL
            ↓
14. Follow-up task created
            ↓
15. Agent Run stored for auditing
```

---

# 26. Why PostgreSQL?

PostgreSQL is suitable because it supports:

* Relational data
* Foreign keys
* Transactions
* JSON/JSONB
* Complex queries
* Indexing
* Full-text search
* Extensions such as pgvector

The combination of:

```text
PostgreSQL + pgvector
```

allows the CRM to store both:

```text
Structured CRM Data
        +
Vector Embeddings
```

in the same database.

---

# 27. Final Architecture

The complete proposed system is:

```text
                         USER
                           │
                           ▼
                       React UI
                           │
                           ▼
                      FastAPI API
                           │
             ┌─────────────┴──────────────┐
             │                            │
             ▼                            ▼
       Normal CRM APIs              AI Request
             │                            │
             ▼                            ▼
       PostgreSQL                 LangGraph
                                          │
                                          ▼
                                    AI Agents
                                          │
                                          ▼
                                      LLM/SLM
                                          │
                             ┌────────────┼────────────┐
                             │            │            │
                             ▼            ▼            ▼
                         CRM Tools    RAG Tools    Email Tools
                             │            │            │
                             ▼            ▼            ▼
                        PostgreSQL     pgvector     Email API
                             │
                             ▼
                        Agent Runs
                             │
                             ▼
                       Audit / Logs
```

---

# 28. Conclusion

An Agentic CRM combines traditional CRM functionality with AI agents and automation.

The main components are:

```text
Frontend
   +
Backend
   +
CRM Database
   +
AI Agents
   +
LLM/SLM
   +
Tools
   +
RAG
   +
External APIs
```

The **PostgreSQL database** stores both traditional CRM information and information about AI operations.

The most important distinction is:

```text
Users = Human actors

Agents = AI actors

Leads / Contacts / Companies / Campaigns = CRM data

Agent Runs / Tool Calls = AI activity and audit information
```

This architecture allows the CRM to move from a system where employees only **manage information** to a system where AI agents can **analyze information, use controlled tools, and execute defined workflows** while maintaining human oversight where required.