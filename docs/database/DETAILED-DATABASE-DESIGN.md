# ITR Management System

# Detailed Database Design

- **Document ID:** ITR-DB-001
- **Version:** 0.1.0
- **Status:** Draft — Database Design Planning
- **Related SRS:** `docs/SRS.md`
- **Related Architecture:** `docs/architecture/SYSTEM-ARCHITECTURE.md`
- **Database Engine:** PostgreSQL 18
- **Database Type:** Multi-Tenant Relational Database
- **Product Type:** Multi-Tenant SaaS
- **Initial Market:** India

---

## Document Purpose

This document defines the detailed database design for the ITR Management System.

It translates the logical database architecture described in Section 7 of the System Architecture and Technical Design document into a detailed PostgreSQL database specification.

The document will define database tables, columns, data types, primary keys, foreign keys, unique constraints, indexes, tenant ownership, Row-Level Security (RLS), and data integrity requirements.

The detailed database design will serve as the reference for subsequent database schema implementation and migration development.

---

## 1. Database Design Principles and Standards

### 1.1 Purpose

This section establishes the design principles, naming conventions, data type standards, and structural rules that will govern the PostgreSQL database schema of the ITR Management System.

All database tables, columns, relationships, constraints, indexes, and migrations must follow these standards unless an explicitly documented exception is approved.

### 1.2 Database Design Principles

The database design shall follow these principles:

1. **Tenant Isolation:** Every tenant-owned record must be protected from unauthorized access by other CA firms.
2. **Data Integrity:** Primary keys, foreign keys, unique constraints, check constraints, and transactions must preserve valid relationships and business rules.
3. **Customer Identity Separation:** Central customer identity must remain separate from firm-specific customer relationships and records.
4. **Security by Design:** Sensitive tax, financial, identity, and authentication data must receive appropriate protection.
5. **Auditability:** Important business and security operations must maintain traceable records.
6. **Scalability:** Tables and indexes must support expected growth in customers, assessment years, ITR records, documents, and transactions.
7. **Maintainability:** Database structures must remain understandable, consistent, and suitable for controlled schema evolution.
8. **Data Consistency:** Financial and business records must use appropriate constraints and transaction controls.
9. **Privacy:** Data must be collected, stored, accessed, and retained according to defined business and privacy requirements.
10. **Recoverability:** Database backup, restoration, and disaster recovery requirements must be supported.

### 1.3 Database Naming Conventions

The following naming conventions are proposed:

| Database Object | Naming Convention | Example |
|---|---|---|
| Tables | Singular snake_case | `customer`, `firm_membership` |
| Columns | snake_case | `created_at`, `tenant_id` |
| Primary Keys | `id` | `id` |
| Foreign Keys | Referenced entity name plus `_id` | `firm_id` |
| Indexes | `idx_<table>_<columns>` | `idx_customer_firm_id` |
| Unique Constraints | `uq_<table>_<columns>` | `uq_firm_customer_pan` |
| Check Constraints | `chk_<table>_<rule>` | `chk_payment_amount_positive` |
| Foreign Key Constraints | `fk_<table>_<referenced_table>` | `fk_customer_firm` |
| Enum Types | Descriptive snake_case | `itr_status` |

All database object names must be lowercase and use underscores as word separators.

### 1.4 Primary Key Standards

1. Every main business table must have a primary key.
2. Primary key columns shall use UUID values.
3. UUID generation must be handled consistently through the application or database.
4. Primary keys must not expose sensitive business information.
5. Primary keys must remain immutable after record creation.

The specific UUID generation strategy will be finalized during schema implementation.

### 1.5 Foreign Key and Relationship Standards

1. Foreign keys must enforce valid relationships between related records.
2. Foreign key columns must use data types compatible with their referenced primary keys.
3. Tenant-owned relationships must use tenant-aware constraints wherever necessary.
4. Cascading deletion must not be used indiscriminately.
5. Financial, audit, filing, and legally relevant records must receive appropriate deletion protection.
6. Referential integrity must be enforced at the database level wherever applicable.

### 1.6 Timestamp and Date Standards

1. Timestamp columns must use PostgreSQL `TIMESTAMPTZ`.
2. Timestamps must be stored and processed consistently in UTC.
3. Business dates that do not represent a specific time must use PostgreSQL `DATE`.
4. Timestamp fields should use consistent names such as `created_at`, `updated_at`, and `deleted_at`.
5. Important business events must retain their actual event timestamps.

### 1.7 Financial Data Standards

1. Financial amounts must use PostgreSQL `NUMERIC` or `DECIMAL`, not floating-point data types.
2. Precision and scale must be explicitly defined for each financial column.
3. Currency must be represented explicitly where multi-currency support is applicable.
4. Payment allocations and invoice balances must maintain transactional consistency.
5. Historical financial records must not be silently overwritten.
6. Corrections must use controlled reversals, adjustments, or credit notes as appropriate.

### 1.8 Nullability and Default Value Standards

1. Columns must be `NOT NULL` when a valid record cannot exist without a value.
2. Nullable columns must have a clearly defined business reason.
3. Default values must represent valid business defaults.
4. Security-sensitive fields must not receive unsafe default values.
5. Application validation and database constraints must work together.

### 1.9 Tenant Ownership Standards

1. Every tenant-owned table must have a clearly defined ownership model.
2. Direct tenant-owned tables should contain `tenant_id` where appropriate.
3. Records whose ownership is inherited through a parent relationship must use tenant-aware foreign key constraints.
4. Tenant context must come from authenticated membership and trusted backend authorization.
5. PostgreSQL Row-Level Security must protect tenant-owned records.
6. Cross-tenant relationships must be prevented at the database level wherever applicable.

### 1.10 Schema Migration Standards

1. All schema changes must be version-controlled.
2. Database migrations must be reviewed before production execution.
3. Production migrations must run through a controlled deployment process.
4. Destructive migrations require additional review and an appropriate recovery plan.
5. RLS policies, triggers, functions, and database-specific objects must be included in migration governance.
6. Migration execution must be auditable.

### 1.11 Standards Governance

These conventions apply to all database design groups and subsequent schema implementation.

Any exception must be documented with its justification, security implications, data integrity impact, and approval decision.

---

## 2. Database Ownership and Table Inventory

### 2.1 Purpose

This section defines the logical ownership classification and initial table inventory for the ITR Management System.

Every database table must have a clearly identified ownership category, access boundary, and relationship to the platform, firm, user, or customer.

The ownership classification will guide tenant isolation, database constraints, PostgreSQL Row-Level Security (RLS), authorization, and schema implementation.

### 2.2 Database Ownership Categories

The database shall use the following logical ownership categories:

| Ownership Category | Description | Example |
|---|---|---|
| Platform-Owned | Records used to operate the SaaS platform across all firms | Platform configuration |
| Firm-Owned | Records belonging exclusively to one CA firm | Firm profile and firm settings |
| User-Owned | Central identity and authentication records | User account |
| Customer Identity-Owned | Central customer identity shared across authorized firm relationships | Customer identity |
| Firm-Customer-Owned | Customer records belonging to a specific firm relationship | Firm customer profile |
| System-Owned | Operational records generated for background processing and platform operations | Import job |
| Relationship-Owned | Records whose ownership is determined through a parent business relationship | ITR document and checklist item |

This classification is logical. The final database schema must explicitly define the tenant ownership and access enforcement mechanism for every table.

### 2.3 Multi-Firm Customer Identity Model

A customer shall have one central identity that may be associated with multiple CA firms.

The central customer identity must remain separate from each firm's customer relationship.

The following entities establish this model:

- `user`: Central login identity and authentication relationship.
- `customer`: Central customer identity and common identity information.
- `firm`: Registered CA firm and tenant boundary.
- `firm_customer`: Firm-specific relationship between a customer and a CA firm.

A customer may have multiple `firm_customer` records, but each record must belong to exactly one firm and one customer.

The following rules apply:

1. Customer login credentials must not be duplicated for each firm.
2. Each firm must maintain its own customer reference, onboarding status, assigned staff, and internal notes.
3. Each firm's ITR, document, workflow, financial, and communication records must remain isolated.
4. Customer access must be restricted to records belonging to their authorized firm relationships.
5. A firm must not access another firm's customer-specific records merely because the same central customer identity exists.
6. Firm switching must validate the customer's relationship with the selected firm.
7. Customer identity updates must not automatically expose firm-specific information across tenants.
8. Tax identity information, including PAN and other firm-maintained tax identifiers, must be stored within the relevant firm-customer relationship and must not be treated as globally shared customer identity data.
9. Each firm's tax identity records must be independently controlled, authorized, and audited.
10. The same customer's PAN may appear in separate firm-specific records where the customer is associated with multiple firms. Uniqueness must be enforced within the appropriate firm scope rather than globally.

### 2.4 Platform Administration Table Group

This group contains records required for platform-level administration.

| Proposed Table | Purpose | Ownership |
|---|---|---|
| `platform_user` | Platform Super Admin identity and administration | Platform-Owned |
| `platform_configuration` | Platform-wide configuration settings | Platform-Owned |
| `platform_audit_event` | Audit records for platform administration | Platform-Owned |
| `platform_feature_flag` | Platform-level feature configuration | Platform-Owned |

### 2.5 Firm and Tenant Management Table Group

| Proposed Table | Purpose | Ownership |
|---|---|---|
| `firm` | Registered CA firm and tenant identity | Platform-Owned |
| `firm_profile` | Firm-specific profile and branding | Firm-Owned |
| `firm_configuration` | Firm-specific settings | Firm-Owned |
| `firm_domain` | Firm-specific domain and branding configuration, if enabled | Firm-Owned |
| `firm_status_history` | Firm activation, suspension, and status history | Firm-Owned |

### 2.6 User Identity and Authentication Table Group

| Proposed Table | Purpose | Ownership |
|---|---|---|
| `user` | Central user identity | User-Owned |
| `user_credential` | Password credential information, if platform-managed | User-Owned |
| `user_session` | Active and historical session information | User-Owned |
| `user_mfa` | Multi-factor authentication configuration | User-Owned |
| `user_recovery_method` | Verified account recovery methods | User-Owned |
| `user_login_history` | Login and authentication history | User-Owned |
| `invitation` | User and customer invitation records | Relationship-Owned |

### 2.7 Firm Membership and Access Control Table Group

| Proposed Table | Purpose | Ownership |
|---|---|---|
| `firm_membership` | Relationship between a user and a firm | Firm-Owned |
| `role` | Firm-specific or predefined role definitions | Firm-Owned |
| `permission` | System permission definitions | Platform-Owned |
| `role_permission` | Role-to-permission mapping | Firm-Owned |
| `membership_role` | Roles assigned to a firm membership | Firm-Owned |
| `customer_access_grant` | Explicit customer access authorization where required | Firm-Owned |

### 2.8 Customer Identity and CRM Table Group

| Proposed Table | Purpose | Ownership |
|---|---|---|
| `customer` | Central customer identity and common account reference | Customer Identity-Owned |
| `customer_contact` | Central customer contact information required for account identity and login | Customer Identity-Owned |
| `firm_customer` | Firm-specific relationship between a customer and a CA firm | Firm-Owned |
| `customer_tax_identity` | Firm-specific PAN and other tax identity information | Firm-Owned |
| `customer_staff_assignment` | Staff assigned to a firm's customer | Firm-Owned |
| `customer_status_history` | Customer status history within a firm | Firm-Owned |
| `customer_relationship_history` | History of firm-customer relationship changes | Firm-Owned |
| `customer_duplicate_match` | Potential duplicate customer matches and review outcomes | Firm-Owned |

### 2.9 Assessment Year and ITR Management Table Group

| Proposed Table | Purpose | Ownership |
|---|---|---|
| `assessment_year` | Configurable assessment year definitions | Platform-Owned |
| `firm_assessment_year` | Firm-specific assessment year availability or configuration, if required | Firm-Owned |
| `customer_itr` | Firm-specific customer ITR records | Firm-Owned |
| `itr_status_history` | ITR status transition history | Firm-Owned |
| `itr_assignment` | Staff assignment for ITR processing | Firm-Owned |
| `itr_filing_details` | Filing acknowledgement and submission references | Firm-Owned |
| `itr_note` | Internal ITR-related notes | Firm-Owned |

### 2.10 Document Management Table Group

| Proposed Table | Purpose | Ownership |
|---|---|---|
| `document` | Document metadata and ownership references | Firm-Owned |
| `document_version` | Document version history | Firm-Owned |
| `document_review` | Document review and approval records | Firm-Owned |
| `document_access_log` | Document access history | Firm-Owned |
| `document_scan_result` | Malware scanning and quarantine results | Firm-Owned |

Actual document content shall be stored in private object storage rather than directly in PostgreSQL.

### 2.11 Checklist and Workflow Table Group

| Proposed Table | Purpose | Ownership |
|---|---|---|
| `checklist_template` | Firm-specific checklist templates | Firm-Owned |
| `checklist_item_template` | Checklist template items | Firm-Owned |
| `customer_checklist` | Customer or ITR-specific checklist instance | Firm-Owned |
| `customer_checklist_item` | Individual checklist item status and completion | Firm-Owned |
| `workflow_transition_history` | Workflow status transition history | Firm-Owned |

### 2.12 Communication and Notification Table Group

| Proposed Table | Purpose | Ownership |
|---|---|---|
| `conversation` | Firm-specific customer communication thread | Firm-Owned |
| `message` | Communication messages | Firm-Owned |
| `message_attachment` | Message attachment metadata | Firm-Owned |
| `notification` | User notification records | Relationship-Owned |
| `notification_template` | Notification templates | Firm-Owned |
| `notification_delivery` | Delivery status and provider references | Firm-Owned |
| `communication_preference` | Customer communication preferences, scoped appropriately | Relationship-Owned |

### 2.13 Financial Management Table Group

| Proposed Table | Purpose | Ownership |
|---|---|---|
| `customer_invoice` | Firm-issued customer invoices | Firm-Owned |
| `customer_payment` | Payments received from customers | Firm-Owned |
| `payment_allocation` | Allocation of payments against invoices | Firm-Owned |
| `customer_receipt` | Payment receipts | Firm-Owned |
| `credit_note` | Credit notes and invoice adjustments | Firm-Owned |
| `financial_adjustment` | Controlled financial adjustments and reversals | Firm-Owned |
| `financial_transaction` | Financial transaction ledger and audit references | Firm-Owned |

### 2.14 Refund Management Table Group

| Proposed Table | Purpose | Ownership |
|---|---|---|
| `customer_refund` | Refund claims and tracking records | Firm-Owned |
| `refund_tracking` | Refund status, received amount, and follow-up history | Firm-Owned |
| `refund_follow_up` | Refund follow-up actions and communication references | Firm-Owned |

### 2.15 SaaS Subscription and Billing Table Group

| Proposed Table | Purpose | Ownership |
|---|---|---|
| `subscription_plan` | SaaS subscription plan definitions | Platform-Owned |
| `plan_feature` | Features and usage limits associated with plans | Platform-Owned |
| `firm_subscription` | Firm subscription and activation details | Firm-Owned |
| `subscription_payment` | SaaS subscription payments | Firm-Owned |
| `subscription_history` | Subscription changes and renewals | Firm-Owned |
| `subscription_usage` | Usage tracking for subscription limits | Firm-Owned |
| `subscription_event` | Subscription lifecycle events | Firm-Owned |

SaaS subscription billing must remain separate from invoices and payments issued by a CA firm to its customers.

### 2.16 Import and Migration Table Group

| Proposed Table | Purpose | Ownership |
|---|---|---|
| `import_job` | Excel or CSV import job | Firm-Owned |
| `import_mapping` | Source-to-database column mappings | Firm-Owned |
| `import_row_result` | Individual import row processing results | Firm-Owned |
| `import_error` | Validation and import errors | Firm-Owned |

### 2.17 Reporting and Export Table Group

| Proposed Table | Purpose | Ownership |
|---|---|---|
| `report_definition` | Saved report definitions | Firm-Owned |
| `report_execution` | Report execution history | Firm-Owned |
| `data_export_job` | Data export requests and processing status | Firm-Owned |

### 2.18 Support and Ticketing Table Group

| Proposed Table | Purpose | Ownership |
|---|---|---|
| `support_ticket` | Customer or staff support requests | Firm-Owned |
| `support_ticket_message` | Ticket discussion and updates | Firm-Owned |
| `support_ticket_attachment` | Ticket attachment metadata | Firm-Owned |
| `platform_support_case` | Platform-level support cases involving firms | Platform-Owned |

### 2.19 Audit and Security Table Group

| Proposed Table | Purpose | Ownership |
|---|---|---|
| `audit_event` | Business activity and record change audit history | Firm-Owned |
| `security_event` | Security-related events | Relationship-Owned |
| `platform_security_event` | Platform-wide security events | Platform-Owned |
| `privileged_access_event` | Exceptional privileged access history | Platform-Owned |
| `privileged_access_grant` | Explicit, approved and time-limited grants of privileged access to firm resources | Platform-Owned |

### 2.20 System and Background Processing Table Group

| Proposed Table | Purpose | Ownership |
|---|---|---|
| `background_job_execution` | Background job execution metadata | System-Owned |
| `background_job_attempt` | Individual execution attempts and retry history for background jobs | System-Owned |
| `scheduled_job_execution` | Scheduled task execution history | System-Owned |
| `scheduled_job_definition` | Configuration and lifecycle of recurring or scheduled jobs | System-Owned |
| `webhook_event` | External webhook receipt and processing history | Relationship-Owned |
| `integration_configuration` | External provider configuration | Firm-Owned |
| `integration_delivery_log` | External integration delivery attempts | Firm-Owned |

### 2.21 Ownership Classification Requirements

Before physical schema implementation, every proposed table must be reviewed against the following requirements:

1. Its ownership category must be explicitly confirmed.
2. Its tenant boundary must be identified.
3. Its parent-child relationships must be documented.
4. Its access control requirements must be defined.
5. Its RLS applicability must be determined.
6. Tenant-aware foreign keys and uniqueness requirements must be identified.
7. Its data classification and retention requirements must be documented.

### 2.22 Table Inventory Governance

The table inventory in this section is an initial logical inventory.

It does not imply that every proposed table must be created exactly as listed. During detailed schema design, related entities may be consolidated or separated where justified by data integrity, security, performance, or maintainability requirements.

Any table addition, removal, consolidation, or ownership change must be documented and reviewed before implementation.

---

## 3. Detailed Table Definitions

### 3.1 Platform User

#### 3.1.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `platform_user` |
| Purpose | Maintains platform-level administrative authorization records |
| Ownership | Platform-Owned |
| Tenant Scope | Platform-level; not associated with a single firm |
| Primary Key | `id` |
| Related Tables | `user`, `platform_audit_event` |
| RLS Requirement | Platform-level access policy |

#### 3.1.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Primary key |
| `user_id` | UUID | No | None | References the central user account |
| `admin_status` | VARCHAR(30) | No | `'active'` | Platform administrator status |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Last modification timestamp |
| `created_by` | UUID | Yes | NULL | Platform user who created the record |
| `updated_by` | UUID | Yes | NULL | Platform user who last updated the record |

#### 3.1.3 Primary Key

- `id` shall be the primary key.
- The primary key shall use UUID.
- The primary key shall remain immutable after creation.

#### 3.1.4 Foreign Keys

| Column | References | Requirement |
|---|---|---|
| `user_id` | `user.id` | Required central user identity |
| `created_by` | `platform_user.id` | Optional creator reference |
| `updated_by` | `platform_user.id` | Optional updater reference |

#### 3.1.5 Unique Constraints

- `user_id` must be unique.
- A central user account may have at most one platform administration record.

#### 3.1.6 Check Constraints

`admin_status` must accept only approved values:

- `active`
- `suspended`
- `revoked`

Additional status values must be introduced through controlled schema changes.

#### 3.1.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `uq_platform_user_user_id` | `user_id` | Prevent duplicate platform administration records |
| `idx_platform_user_admin_status` | `admin_status` | Support platform administrator status queries |

#### 3.1.8 Security and Access Control

1. Only authorized platform administrators may manage platform administration records.
2. Platform administrator privileges must be enforced by backend authorization.
3. Platform administrators must use mandatory MFA.
4. Suspending or revoking a platform administrator must prevent further privileged operations.
5. Privileged actions must generate platform audit events.
6. Platform administration must not automatically grant unrestricted access to firm-owned customer tax information.
7. Platform-level authorization must remain separate from firm-level membership and permissions.
8. Database access must follow least-privilege principles.

#### 3.1.9 Data Integrity and Lifecycle

1. Every platform administrator must reference an existing central user account.
2. A user account must not be deleted while an active platform administration record depends on it.
3. Administrator revocation must preserve historical audit evidence.
4. Status changes must be recorded in the platform audit history.
5. Platform administrator creation and revocation must follow an authorized administrative process.

#### 3.1.10 Design Notes and Open Decisions

- Authentication credentials remain in the central `user` and associated authentication tables.
- `platform_user` represents platform-level administrative authorization, not a separate login identity.
- The exact MFA enforcement mechanism will be finalized during authentication implementation.
- The platform-level RLS and privileged database access model must be designed separately from firm-tenant RLS.
- The final schema must prevent privilege escalation through direct database or application access.

---

### 3.2 Platform Configuration

#### 3.2.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `platform_configuration` |
| Purpose | Stores global configuration settings applicable across the SaaS platform |
| Ownership | Platform-Owned |
| Tenant Scope | Platform-level |
| Primary Key | `id` |
| Related Tables | `platform_user`, `platform_audit_event` |
| RLS Requirement | Platform-level access policy |

#### 3.2.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Primary key |
| `config_key` | VARCHAR(150) | No | None | Unique configuration identifier |
| `config_value` | JSONB | No | None | Configuration value stored as structured JSON |
| `config_category` | VARCHAR(50) | No | `'general'` | Configuration category |
| `description` | TEXT | Yes | NULL | Purpose and description of the setting |
| `is_active` | BOOLEAN | No | TRUE | Indicates whether the configuration is active |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Last modification timestamp |
| `created_by` | UUID | Yes | NULL | Platform administrator who created the record |
| `updated_by` | UUID | Yes | NULL | Platform administrator who last updated the record |

#### 3.2.3 Primary Key

- `id` shall be the primary key.
- The primary key shall use UUID.
- The primary key shall remain immutable after creation.

#### 3.2.4 Foreign Keys

| Column | References | Requirement |
|---|---|---|
| `created_by` | `platform_user.id` | Optional creator reference |
| `updated_by` | `platform_user.id` | Optional updater reference |

#### 3.2.5 Unique Constraints

- `config_key` must be unique across the platform.
- Configuration keys must follow a consistent lowercase naming convention.
- A configuration key must identify one logical platform setting.

#### 3.2.6 Check Constraints

1. `config_key` must not be empty.
2. `config_category` must not be empty.
3. `config_value` must contain a valid JSON value.
4. Configuration values must be validated against the expected structure and data type defined for each configuration key.

#### 3.2.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `uq_platform_configuration_config_key` | `config_key` | Prevent duplicate configuration keys |
| `idx_platform_configuration_category` | `config_category` | Support category-based configuration retrieval |
| `idx_platform_configuration_active` | `is_active` | Support active configuration filtering |

#### 3.2.8 Configuration Categories

The following categories are proposed:

| Category | Example Settings |
|---|---|
| `general` | Platform name and general system settings |
| `security` | Global security policy parameters |
| `maintenance` | Maintenance mode and operational settings |
| `notification` | Global notification defaults |
| `system` | Platform-wide operational configuration |
| `limits` | Default platform-level operational limits |

Additional categories may be introduced through controlled configuration governance.

#### 3.2.9 Security and Access Control

1. Only authorized Platform Super Admin users may create, modify, activate, or deactivate global configurations.
2. Configuration access must be enforced through backend authorization.
3. Firm administrators and ordinary firm staff must not modify platform-level configurations.
4. Sensitive credentials, API keys, passwords, private keys, and encryption secrets must not be stored in this table.
5. External secrets must be maintained through an appropriate secrets manager or secure key-management service.
6. Security-sensitive configuration changes must require appropriate privileged authorization.
7. Configuration changes must generate platform audit events.
8. Configuration values must be validated before activation.
9. Configuration access must follow the principle of least privilege.

#### 3.2.10 Data Integrity and Lifecycle

1. Configuration keys must remain unique.
2. Configuration values must conform to their defined schemas.
3. Invalid configuration values must not be activated.
4. Configuration changes must be recorded in platform audit history.
5. Deactivation should be preferred over deletion when historical traceability is required.
6. Changes to critical security or operational settings must follow a controlled approval process.

#### 3.2.11 Design Notes and Open Decisions

- JSONB is proposed to support flexible structured configuration values.
- Configuration schemas and validation rules must be maintained in application code or a controlled configuration registry.
- Configuration changes must not bypass security authorization or audit requirements.
- A future implementation may introduce configuration version history if the platform requires rollback or historical comparison.
- Platform-wide configuration must remain separate from firm-specific configuration stored in `firm_configuration`.
- Configuration caching and invalidation strategies will be finalized during application implementation.
- The final implementation must ensure that configuration updates cannot silently change security-critical behaviour.

---

### 3.3 Platform Audit Event

#### 3.3.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `platform_audit_event` |
| Purpose | Records platform-level administrative, configuration, and privileged security activities |
| Ownership | Platform-Owned |
| Tenant Scope | Platform-level |
| Primary Key | `id` |
| Related Tables | `platform_user`, `platform_configuration`, `platform_feature_flag` |
| RLS Requirement | Platform-level restricted access policy |
| Record Modification | Append-only |

#### 3.3.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Primary key |
| `actor_platform_user_id` | UUID | Yes | NULL | Platform administrator responsible for the action |
| `actor_user_id` | UUID | Yes | NULL | Central user identity associated with the actor |
| `event_type` | VARCHAR(100) | No | None | Type of administrative or security event |
| `operation` | VARCHAR(50) | No | None | Operation performed |
| `target_entity_type` | VARCHAR(100) | Yes | NULL | Type of record affected |
| `target_entity_id` | UUID | Yes | NULL | Identifier of the affected record |
| `description` | TEXT | Yes | NULL | Human-readable event description |
| `event_data` | JSONB | No | `'{}'::jsonb` | Structured event details |
| `outcome` | VARCHAR(30) | No | None | Result of the operation |
| `ip_address` | INET | Yes | NULL | Source IP address where available |
| `user_agent` | TEXT | Yes | NULL | Client user-agent information where available |
| `correlation_id` | UUID | Yes | NULL | Identifier linking related operations |
| `occurred_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Time when the event occurred |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Time when the audit record was persisted |

#### 3.3.3 Primary Key

- `id` shall be the primary key.
- The primary key shall use UUID.
- The primary key shall remain immutable.

#### 3.3.4 Foreign Keys

| Column | References | Requirement |
|---|---|---|
| `actor_platform_user_id` | `platform_user.id` | Optional platform administrator reference |
| `actor_user_id` | `user.id` | Optional central user reference |

Both actor references may be NULL for system-generated events or events generated before an actor can be resolved.

At least one actor or system-origin indicator must be established by the application event contract. The final system-origin representation will be decided during implementation.

Historical audit records must be preserved when an associated user or administrator is deactivated or revoked.

#### 3.3.5 Unique Constraints

No general business-level unique constraint is required.

The application must prevent duplicate audit events when processing retried operations. A suitable event identifier or idempotency mechanism must be finalized during implementation.

#### 3.3.6 Check Constraints

1. `event_type` must not be empty.
2. `operation` must not be empty.
3. `outcome` must contain an approved value.
4. `event_data` must contain a valid JSON object.
5. The actor and system-origin rules must be validated before an event is persisted.

Proposed outcome values:

- `success`
- `failure`
- `denied`
- `pending`

#### 3.3.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `idx_platform_audit_event_occurred_at` | `occurred_at` | Time-based audit retrieval |
| `idx_platform_audit_event_actor` | `actor_platform_user_id`, `occurred_at` | Administrator activity history |
| `idx_platform_audit_event_type` | `event_type`, `occurred_at` | Event-type filtering |
| `idx_platform_audit_event_target` | `target_entity_type`, `target_entity_id` | Retrieve events for an affected record |
| `idx_platform_audit_event_correlation` | `correlation_id` | Trace related operations |

Indexes must be reviewed against actual audit-query patterns and expected data volume.

#### 3.3.8 Audit Event Categories

The following event categories are proposed:

| Category | Examples |
|---|---|
| `platform_admin` | Administrator creation, suspension, revocation |
| `configuration` | Global configuration changes |
| `feature_management` | Platform feature activation or deactivation |
| `privileged_access` | Exceptional access requests and approvals |
| `security` | Security policy changes and administrative security actions |
| `system_operation` | Important platform-level operational events |

#### 3.3.9 Security and Access Control

1. Audit events must be generated by trusted backend services.
2. Platform administrators may retrieve audit records only when authorized.
3. Ordinary firm users must not access platform-level audit records.
4. Application runtime roles must not have permission to update or delete historical audit records.
5. Audit records must be protected against unauthorized modification and deletion.
6. Database permissions must enforce append-only behaviour for the application audit-writing role.
7. Audit records must not contain passwords, OTP values, access tokens, refresh tokens, API secrets, encryption keys, or unnecessary sensitive tax information.
8. IP addresses and user-agent information must be handled according to applicable privacy and retention requirements.
9. Critical audit events must be retained according to the approved retention policy.
10. Audit retrieval and export operations must themselves be appropriately authorized and auditable.

#### 3.3.10 Data Integrity and Lifecycle

1. Audit events must be persisted reliably for important platform operations.
2. Failed administrative operations must be recorded where the audit event can be safely persisted.
3. Audit event timestamps must use UTC.
4. Audit records must not be silently overwritten.
5. Corrections to audit evidence must use a separately recorded corrective event rather than altering the original record.
6. Retention, archival, and disposal must follow an approved audit-retention policy.
7. Audit storage must be monitored for availability and capacity.

#### 3.3.11 Design Notes and Open Decisions

- The table is intended for platform-level events, not ordinary firm-specific business activity.
- Firm-level business and security events will be maintained separately in the appropriate tenant-scoped audit tables.
- Append-only database permissions and, where appropriate, external tamper-resistant audit storage will be evaluated during implementation.
- Event identifiers and idempotency handling must be finalized.
- The system must define how system-generated events are distinguished from human-generated events.
- Audit retention and archival periods remain open decisions.
- High-volume audit retrieval may require time-based partitioning after workload analysis.

---

### 3.4 Platform Feature Flag

#### 3.4.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `platform_feature_flag` |
| Purpose | Controls global availability of platform features |
| Ownership | Platform-Owned |
| Tenant Scope | Platform-level |
| Primary Key | `id` |
| Related Tables | `platform_user`, `platform_audit_event`, `subscription_plan`, `plan_feature` |
| RLS Requirement | Platform-level restricted access policy |

#### 3.4.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Primary key |
| `feature_key` | VARCHAR(150) | No | None | Unique feature identifier |
| `feature_name` | VARCHAR(200) | No | None | Human-readable feature name |
| `description` | TEXT | Yes | NULL | Feature description |
| `is_enabled` | BOOLEAN | No | FALSE | Global feature availability |
| `release_stage` | VARCHAR(30) | No | `'general'` | Feature release stage |
| `configuration` | JSONB | No | `'{}'::jsonb` | Non-secret feature-specific configuration |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Last modification timestamp |
| `created_by` | UUID | Yes | NULL | Platform administrator who created the record |
| `updated_by` | UUID | Yes | NULL | Platform administrator who last updated the record |

#### 3.4.3 Primary Key

- `id` shall be the primary key.
- The primary key shall use UUID.
- The primary key shall remain immutable.

#### 3.4.4 Foreign Keys

| Column | References | Requirement |
|---|---|---|
| `created_by` | `platform_user.id` | Optional creator reference |
| `updated_by` | `platform_user.id` | Optional updater reference |

#### 3.4.5 Unique Constraints

- `feature_key` must be unique across the platform.
- Feature keys must follow a consistent lowercase naming convention.
- A feature key must identify one logical platform capability.

#### 3.4.6 Check Constraints

1. `feature_key` must not be empty.
2. `feature_name` must not be empty.
3. `configuration` must contain a valid JSON object.
4. `release_stage` must contain an approved value.

Proposed release stages:

- `development`
- `beta`
- `general`
- `deprecated`

#### 3.4.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `uq_platform_feature_flag_feature_key` | `feature_key` | Prevent duplicate feature identifiers |
| `idx_platform_feature_flag_enabled` | `is_enabled` | Retrieve enabled features |
| `idx_platform_feature_flag_release_stage` | `release_stage` | Filter features by release stage |

#### 3.4.8 Feature Availability and Entitlement Rules

1. A globally disabled feature must not be made available through firm-level configuration.
2. A globally enabled feature is not automatically available to every firm.
3. Firm-level availability must also consider subscription plan entitlements and applicable authorization policies.
4. Feature flags must not grant permissions or bypass role-based access control.
5. Feature availability must be evaluated by trusted backend services.
6. Feature-specific configuration must be validated before activation.
7. Emergency disabling of a feature must be supported through an authorized administrative process.
8. Existing business records must remain accessible according to their normal authorization rules even if a feature is disabled.

#### 3.4.9 Security and Access Control

1. Only authorized Platform Super Admin users may create, modify, or activate global feature flags.
2. All changes must be enforced through backend authorization.
3. Firm administrators and staff must not modify global feature flags.
4. Feature flag configuration must not contain passwords, API keys, tokens, or other secrets.
5. Changes to feature availability must generate platform audit events.
6. Feature evaluation must not expose unauthorized information across tenants.
7. Feature flag caching must support timely invalidation when critical settings change.

#### 3.4.10 Data Integrity and Lifecycle

1. Feature keys must remain unique.
2. Feature configuration must follow a validated schema.
3. Disabling a feature must not silently delete or modify existing customer data.
4. Deprecated features must remain identifiable for migration and retirement planning.
5. Historical feature changes must be traceable through platform audit history.
6. Removal of a feature must follow a controlled deprecation process.

#### 3.4.11 Design Notes and Open Decisions

- This table controls global platform feature availability.
- Subscription entitlements will be defined separately through `subscription_plan` and `plan_feature`.
- Firm-specific overrides, if required, must use an explicit and auditable mechanism.
- Feature rollout percentages and selected-firm beta releases may require additional rollout configuration.
- The exact feature evaluation and caching strategy will be finalized during application implementation.
- Feature flags must never replace authentication, authorization, tenant isolation, or security controls.

---

### 3.5 Firm

#### 3.5.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `firm` |
| Purpose | Stores the core identity and lifecycle information of each CA firm using the platform |
| Ownership | Platform-Owned / Tenant Root |
| Tenant Scope | One record per registered firm |
| Primary Key | `id` |
| Related Tables | `firm_profile`, `firm_configuration`, `firm_domain`, `firm_status_history`, `firm_membership`, `firm_customer`, `firm_subscription` |
| RLS Requirement | Special tenant-root access policy; platform-level administrative access must be separately controlled |

#### 3.5.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Unique firm identifier and tenant identifier |
| `firm_code` | VARCHAR(30) | No | None | Unique system-generated firm reference |
| `legal_name` | VARCHAR(250) | No | None | Registered legal name of the firm |
| `display_name` | VARCHAR(200) | No | None | Name displayed in the application |
| `primary_email` | VARCHAR(254) | No | None | Primary administrative contact email |
| `primary_phone` | VARCHAR(20) | Yes | NULL | Primary contact phone number |
| `status` | VARCHAR(30) | No | `'pending'` | Current firm lifecycle status |
| `onboarded_at` | TIMESTAMPTZ | Yes | NULL | Date and time the firm completed onboarding |
| `suspended_at` | TIMESTAMPTZ | Yes | NULL | Date and time the firm was suspended |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Last modification timestamp |
| `created_by` | UUID | Yes | NULL | Platform administrator who created the firm record |
| `updated_by` | UUID | Yes | NULL | Platform administrator who last updated the record |

#### 3.5.3 Primary Key

- `id` shall be the primary key.
- UUID shall be used for the firm identifier.
- The firm ID shall also serve as the tenant identifier for firm-owned records.
- The firm ID must remain immutable after creation.

#### 3.5.4 Unique Constraints

- `firm_code` must be unique across the platform.
- Firm codes must be generated through a controlled backend process.
- Firm codes must not be reused after a firm is deactivated or deleted.

#### 3.5.5 Foreign Keys

| Column | References | Requirement |
|---|---|---|
| `created_by` | `platform_user.id` | Optional creator reference |
| `updated_by` | `platform_user.id` | Optional updater reference |

#### 3.5.6 Check Constraints

1. `firm_code` must not be empty.
2. `legal_name` must not be empty.
3. `display_name` must not be empty.
4. `primary_email` must not be empty.
5. `status` must contain an approved value.

Proposed firm lifecycle statuses:

- `pending`
- `active`
- `suspended`
- `inactive`
- `closed`

#### 3.5.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `uq_firm_firm_code` | `firm_code` | Prevent duplicate firm codes |
| `idx_firm_status` | `status` | Retrieve firms by lifecycle status |
| `idx_firm_created_at` | `created_at` | Support chronological firm searches |

#### 3.5.8 Tenant Ownership and Isolation

1. Each registered CA firm must have exactly one firm record.
2. The `firm.id` value shall be the tenant identifier used by firm-owned tables.
3. Firm-owned records must reference the appropriate firm through `firm_id`.
4. Tenant context must be established and validated by trusted backend services.
5. PostgreSQL Row-Level Security must enforce tenant isolation on applicable firm-owned tables.
6. Firm users must not be able to change their tenant identifier or access another firm's records.
7. Platform-level access must use a separately controlled administrative access path.
8. The firm record must not be treated as permission to access all platform data.

#### 3.5.9 Security and Access Control

1. Only authorized platform administrators or approved onboarding processes may create firm records.
2. Firm administrators may update only explicitly permitted firm information through authorized services.
3. Firm status changes must require appropriate authorization.
4. Suspended firms must be prevented from performing prohibited operations.
5. Suspension must not automatically delete customer, tax, document, financial, or audit records.
6. Firm contact information must be protected against unauthorized disclosure.
7. All privileged firm lifecycle changes must generate platform audit events.

#### 3.5.10 Data Integrity and Lifecycle

1. Firm identifiers must remain stable throughout the firm's lifecycle.
2. Firm codes must never be reassigned to another firm.
3. Firm status changes must be recorded in `firm_status_history`.
4. Onboarding and suspension timestamps must be maintained consistently with lifecycle transitions.
5. Firm closure must follow a controlled retention and data-export process.
6. Firm deletion must not cascade into uncontrolled deletion of legally or operationally required records.

#### 3.5.11 Design Notes and Open Decisions

- Firm profile and registration details will be maintained separately in `firm_profile`.
- Firm-specific application settings will be maintained in `firm_configuration`.
- Firm lifecycle transitions will be recorded in `firm_status_history`.
- Subscription status and payment information will be maintained separately.
- The exact firm onboarding, suspension, closure, and retention workflows will be finalized during application design.
- The firm table is the tenant root and requires particular care during database migration, RLS policy design, and administrative access implementation.

---

### 3.6 Firm Profile

#### 3.6.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `firm_profile` |
| Purpose | Stores detailed professional, registration, and business profile information of a CA firm |
| Ownership | Firm-Owned |
| Tenant Scope | One profile per firm |
| Primary Key | `id` |
| Related Tables | `firm`, `firm_configuration` |
| RLS Requirement | Tenant-level RLS using `firm_id` |

#### 3.6.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Primary key |
| `firm_id` | UUID | No | None | Associated firm and tenant identifier |
| `firm_registration_number` | VARCHAR(100) | Yes | NULL | Professional or firm registration reference, where applicable |
| `membership_or_registration_details` | JSONB | No | `'{}'::jsonb` | Structured professional registration information |
| `address_line_1` | VARCHAR(250) | Yes | NULL | Primary business address |
| `address_line_2` | VARCHAR(250) | Yes | NULL | Additional address details |
| `city` | VARCHAR(100) | Yes | NULL | City |
| `state` | VARCHAR(100) | Yes | NULL | State or region |
| `postal_code` | VARCHAR(20) | Yes | NULL | Postal code |
| `country_code` | CHAR(2) | No | `'IN'` | ISO country code |
| `website_url` | VARCHAR(500) | Yes | NULL | Official firm website |
| `logo_object_key` | VARCHAR(500) | Yes | NULL | Private object-storage reference for the firm logo |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Last modification timestamp |
| `created_by` | UUID | Yes | NULL | User who created the profile |
| `updated_by` | UUID | Yes | NULL | User who last updated the profile |

#### 3.6.3 Primary Key and Relationship

- `id` shall be the primary key and use UUID.
- `firm_id` shall reference `firm.id`.
- Each firm shall have at most one profile record.
- A unique constraint on `firm_id` shall enforce the one-to-one relationship.
- The profile must never be associated with a different firm by changing its `firm_id`.

#### 3.6.4 Foreign Keys

| Column | References | Requirement |
|---|---|---|
| `firm_id` | `firm.id` | Mandatory firm relationship |

The creator and updater references will be finalized when the user identity and firm membership relationships are fully specified. They must not introduce cross-tenant references.

#### 3.6.5 Unique Constraints

- `firm_id` must be unique.
- Firm registration numbers must not be assumed globally unique unless the relevant registration authority and identifier type establish that rule.

#### 3.6.6 Check Constraints

1. `country_code` must contain a valid two-character country code.
2. `membership_or_registration_details` must contain a JSON object.
3. Non-null address fields must not contain only whitespace.
4. A non-null website URL must pass application-level URL validation.

#### 3.6.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `uq_firm_profile_firm_id` | `firm_id` | Enforce one profile per firm |

#### 3.6.8 Tenant Isolation and Security

1. Every profile record must belong to exactly one firm.
2. All profile access must be restricted using `firm_id`.
3. PostgreSQL RLS must prevent one firm from reading or modifying another firm's profile.
4. Backend authorization must validate the user's firm membership before allowing profile changes.
5. Firm logos must be stored in private object storage.
6. Public access to logo objects must not be enabled by default.
7. Registration and professional information must be protected against unauthorized modification.
8. Platform-level access must follow a separately authorized administrative process.

#### 3.6.9 Data Integrity and Lifecycle

1. A profile cannot exist without its associated firm.
2. The firm relationship must remain stable.
3. Profile changes must update `updated_at`.
4. Profile creation and modification must be auditable.
5. Profile deletion must follow the firm's retention and closure policies.
6. Deleting a profile must not delete the associated firm or its business records.

#### 3.6.10 Design Notes and Open Decisions

- The `firm` table retains core tenant identity and lifecycle information.
- This table contains extended firm profile information.
- Registration information is structured to accommodate different professional registration types.
- Logo files are referenced through private object storage rather than stored directly in PostgreSQL.
- The exact registration fields and their validation rules will be finalized after reviewing the firm's onboarding requirements.
- The relationship between profile editors and firm memberships will be finalized in the user and membership table designs.

---

### 3.7 Firm Configuration

#### 3.7.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `firm_configuration` |
| Purpose | Stores configurable application and operational settings for each CA firm |
| Ownership | Firm-Owned |
| Tenant Scope | Firm-specific |
| Primary Key | `id` |
| Related Tables | `firm`, `firm_membership`, `audit_event` |
| RLS Requirement | Tenant-level RLS using `firm_id` |

#### 3.7.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Primary key |
| `firm_id` | UUID | No | None | Associated firm and tenant identifier |
| `config_key` | VARCHAR(150) | No | None | Unique configuration identifier within the firm |
| `config_value` | JSONB | No | None | Configuration value |
| `config_category` | VARCHAR(50) | No | `'general'` | Configuration category |
| `description` | TEXT | Yes | NULL | Description of the configuration |
| `is_active` | BOOLEAN | No | TRUE | Indicates whether the configuration is active |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Last modification timestamp |
| `created_by_membership_id` | UUID | Yes | NULL | Firm membership responsible for creation |
| `updated_by_membership_id` | UUID | Yes | NULL | Firm membership responsible for last modification |

#### 3.7.3 Primary Key and Relationships

- `id` shall be the primary key and use UUID.
- `firm_id` shall reference `firm.id`.
- Each configuration record must belong to exactly one firm.
- Configuration records must not be transferable between firms by changing `firm_id`.

#### 3.7.4 Foreign Keys

| Column | References | Requirement |
|---|---|---|
| `firm_id` | `firm.id` | Mandatory firm relationship |
| `created_by_membership_id` | `firm_membership.id` | Optional creating membership |
| `updated_by_membership_id` | `firm_membership.id` | Optional updating membership |

The creator and updater membership references must be validated against the same firm. This cross-table tenant consistency requirement will be enforced through appropriate composite foreign keys or equivalent database constraints in the physical schema.

#### 3.7.5 Unique Constraints

- The combination of `firm_id` and `config_key` must be unique.
- The same configuration key may exist independently for different firms.
- Configuration keys must follow a consistent lowercase naming convention.

#### 3.7.6 Check Constraints

1. `config_key` must not be empty.
2. `config_category` must not be empty.
3. `config_value` must contain a valid JSON object.
4. Configuration categories must use approved values.

Proposed categories:

- `general`
- `branding`
- `workflow`
- `notification`
- `document`
- `customer_portal`
- `financial`
- `reporting`
- `security`

#### 3.7.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `uq_firm_configuration_firm_key` | `firm_id`, `config_key` | Prevent duplicate keys within a firm |
| `idx_firm_configuration_firm_category` | `firm_id`, `config_category` | Retrieve settings by firm and category |
| `idx_firm_configuration_firm_active` | `firm_id`, `is_active` | Retrieve active settings for a firm |

#### 3.7.8 Tenant Isolation and Security

1. Every configuration record must belong to exactly one firm.
2. PostgreSQL RLS must restrict configuration access to the current tenant.
3. Backend services must validate firm membership and permissions before allowing configuration changes.
4. Firm administrators may modify only settings they are authorized to manage.
5. Firm staff must not modify administrative or security-sensitive settings without explicit permission.
6. Configuration values must be validated against approved schemas before use.
7. Secrets, passwords, API keys, tokens, and private credentials must never be stored in this table.
8. Sensitive integration credentials must be stored in an appropriate secrets-management service.
9. Configuration changes must generate tenant-scoped audit events.
10. Cached configuration must be invalidated when relevant settings change.

#### 3.7.9 Data Integrity and Lifecycle

1. A configuration record cannot exist without its associated firm.
2. Configuration keys must remain unique within their firm.
3. Every modification must update `updated_at`.
4. Invalid configuration values must not be activated.
5. Deactivation should be preferred over deletion when historical traceability is required.
6. Configuration changes must not silently alter or delete existing customer tax records, documents, or financial records.

#### 3.7.10 Design Notes and Open Decisions

- Platform-wide settings remain in `platform_configuration`.
- Firm-specific settings remain in `firm_configuration`.
- Firm-specific settings must not override mandatory platform security policies.
- Configuration schemas and validation rules will be maintained by the application.
- The exact configuration keys will be introduced as individual features are developed.
- Firm membership references must be constrained to the same tenant.
- Effective configuration precedence between platform defaults and firm-specific values will be finalized during application design.

---

### 3.8 Firm Domain

#### 3.8.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `firm_domain` |
| Purpose | Stores custom domain names associated with CA firms |
| Ownership | Firm-Owned |
| Tenant Scope | Firm-specific |
| Primary Key | `id` |
| Related Tables | `firm`, `firm_status_history`, `audit_event` |
| RLS Requirement | Tenant-level RLS using `firm_id`, with controlled platform-level domain administration |

#### 3.8.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Primary key |
| `firm_id` | UUID | No | None | Associated firm and tenant identifier |
| `domain_name` | VARCHAR(253) | No | None | Normalized custom domain name |
| `verification_status` | VARCHAR(30) | No | `'pending'` | Domain ownership verification status |
| `verification_method` | VARCHAR(30) | No | `'dns_txt'` | Verification method |
| `verification_token_hash` | VARCHAR(255) | Yes | NULL | Hash of the domain verification token |
| `verification_expires_at` | TIMESTAMPTZ | Yes | NULL | Expiration time of the verification challenge |
| `verified_at` | TIMESTAMPTZ | Yes | NULL | Successful verification timestamp |
| `is_primary` | BOOLEAN | No | FALSE | Indicates the firm's primary custom domain |
| `is_active` | BOOLEAN | No | FALSE | Indicates whether the domain is active |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Last modification timestamp |
| `created_by_membership_id` | UUID | Yes | NULL | Firm membership responsible for creating the record |
| `updated_by_membership_id` | UUID | Yes | NULL | Firm membership responsible for last modification |

#### 3.8.3 Primary Key and Relationships

- `id` shall be the primary key and use UUID.
- `firm_id` shall reference `firm.id`.
- Each domain record must belong to exactly one firm.
- A domain must not be transferred between firms through an ordinary update operation.

#### 3.8.4 Foreign Keys

| Column | References | Requirement |
|---|---|---|
| `firm_id` | `firm.id` | Mandatory firm relationship |
| `created_by_membership_id` | `firm_membership.id` | Optional creating membership |
| `updated_by_membership_id` | `firm_membership.id` | Optional updating membership |

Creator and updater memberships must belong to the same firm as the domain record. This must be enforced through appropriate composite foreign keys or equivalent database constraints.

#### 3.8.5 Unique Constraints

- `domain_name` must be globally unique across all firms.
- Domain names must be normalized before storage.
- Domain comparison must be case-insensitive.
- A firm must not have more than one primary active domain.
- The primary-domain rule must be enforced through an appropriate partial unique index.

#### 3.8.6 Check Constraints

1. `domain_name` must not be empty.
2. `verification_status` must contain an approved value.
3. `verification_method` must contain an approved value.
4. An active domain must have a successful verification status and a non-null `verified_at`.
5. A primary domain must be active and verified.

Proposed verification statuses:

- `pending`
- `verified`
- `failed`
- `expired`
- `revoked`

Proposed verification methods:

- `dns_txt`
- `dns_cname`

#### 3.8.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `uq_firm_domain_domain_name` | `domain_name` | Prevent a domain from being registered by multiple firms |
| `idx_firm_domain_firm_id` | `firm_id` | Retrieve domains belonging to a firm |
| `idx_firm_domain_firm_active` | `firm_id`, `is_active` | Retrieve active firm domains |
| `uq_firm_domain_primary_active` | `firm_id` where `is_primary = TRUE AND is_active = TRUE` | Allow only one primary active domain per firm |

#### 3.8.8 Domain Verification and Tenant Isolation

1. Domain ownership must be verified before activation.
2. Verification must use a controlled challenge-response process, such as a DNS TXT record.
3. Verification tokens must be cryptographically random and stored only as hashes.
4. Verification tokens must expire.
5. Verification attempts must be rate-limited.
6. A domain already associated with another firm must not be claimable.
7. Domain routing must resolve to the correct firm only after verification and activation.
8. The firm identity must be validated independently of user-supplied request headers.
9. A custom domain must never be treated as sufficient proof of user authorization or tenant membership.
10. Domain changes and verification outcomes must be audited.

#### 3.8.9 Security and Access Control

1. Only authorized firm administrators may request domain registration or removal.
2. Only trusted backend services may validate verification challenges and activate domains.
3. Firm users must not be able to bypass domain verification by directly modifying database records.
4. Platform administrators may manage domain conflicts through a controlled and audited process.
5. Domain deactivation or revocation must immediately prevent its use for new tenant routing.
6. Custom domains must use HTTPS with valid certificate management.
7. Domain verification tokens must never appear in application logs or audit-event payloads.
8. Domain configuration must not bypass normal authentication, authorization, or tenant isolation.

#### 3.8.10 Data Integrity and Lifecycle

1. Every domain record must belong to an existing firm.
2. Domain ownership must remain globally unique.
3. Verification status changes must be recorded in audit history.
4. Expired challenges must not be reusable.
5. Revoked domains must not remain active.
6. Removing a domain must not delete the associated firm or its business records.
7. Domain reassignment must require a controlled release and fresh verification process.

#### 3.8.11 Design Notes and Open Decisions

- Custom domains may be introduced after the initial SaaS launch.
- The platform's default domain will remain available independently of custom domains.
- DNS verification is the proposed initial ownership verification mechanism.
- Domain routing and TLS certificate provisioning will be implemented through controlled infrastructure services.
- The exact domain onboarding and removal workflow will be finalized during application design.
- Domain uniqueness must be enforced globally, not merely within an individual tenant.

---

### 3.9 Firm Status History

#### 3.9.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `firm_status_history` |
| Purpose | Maintains the historical record of firm lifecycle status changes |
| Ownership | Platform-Owned / Firm Lifecycle |
| Tenant Scope | Firm-specific record with controlled platform administration |
| Primary Key | `id` |
| Related Tables | `firm`, `platform_user`, `platform_audit_event` |
| RLS Requirement | Restricted lifecycle-history access; firm-level visibility only where explicitly authorized |

#### 3.9.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Primary key |
| `firm_id` | UUID | No | None | Firm whose status changed |
| `previous_status` | VARCHAR(30) | Yes | NULL | Status before the transition; NULL for initial registration |
| `new_status` | VARCHAR(30) | No | None | Status after the transition |
| `change_reason` | TEXT | Yes | NULL | Reason for the status change |
| `change_source` | VARCHAR(30) | No | `'platform_admin'` | Source of the status change |
| `changed_by_platform_user_id` | UUID | Yes | NULL | Platform administrator responsible for the change |
| `correlation_id` | UUID | Yes | NULL | Identifier linking related operations |
| `occurred_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Time at which the status transition occurred |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Record creation timestamp |

#### 3.9.3 Primary Key and Relationships

- `id` shall be the primary key and use UUID.
- `firm_id` shall reference `firm.id`.
- Each record shall represent one status transition.
- Historical records must not be overwritten when a firm's current status changes.

#### 3.9.4 Foreign Keys

| Column | References | Requirement |
|---|---|---|
| `firm_id` | `firm.id` | Mandatory firm reference |
| `changed_by_platform_user_id` | `platform_user.id` | Optional administrator reference |

The administrator reference may be NULL for approved automated system transitions. The change source must identify the origin of the transition.

#### 3.9.5 Unique Constraints

- No general uniqueness constraint is required for status history records.
- Each status transition must have its own unique record identifier.
- Duplicate transition prevention must be handled through reliable transaction and idempotency controls.

#### 3.9.6 Check Constraints

1. `new_status` must contain an approved firm lifecycle status.
2. A non-null `previous_status` must contain an approved lifecycle status.
3. `previous_status` and `new_status` must not be identical.
4. `change_source` must contain an approved value.

Proposed lifecycle statuses:

- `pending`
- `active`
- `suspended`
- `inactive`
- `closed`

Proposed change sources:

- `platform_admin`
- `system`
- `automated_process`

#### 3.9.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `idx_firm_status_history_firm_time` | `firm_id`, `occurred_at` | Retrieve a firm's status history chronologically |
| `idx_firm_status_history_new_status` | `new_status` | Find transitions into a particular status |
| `idx_firm_status_history_actor_time` | `changed_by_platform_user_id`, `occurred_at` | Review administrator status changes |
| `idx_firm_status_history_correlation_id` | `correlation_id` | Trace related operations |

#### 3.9.8 Status Transition and Integrity Rules

1. Every firm status change must generate a corresponding history record.
2. The current status in `firm.status` and the latest committed history transition must remain consistent.
3. Status updates and history insertion must occur within the same database transaction.
4. Invalid lifecycle transitions must be rejected by the application and appropriate database safeguards.
5. Initial firm registration must generate a history record with a NULL `previous_status`.
6. Automated transitions must identify their source.
7. A failed status change must not leave a history record that incorrectly represents a completed transition.

#### 3.9.9 Security and Access Control

1. Only authorized platform administrators or approved system processes may change firm lifecycle status.
2. Firm users must not modify their own firm's lifecycle history.
3. Firm administrators may view relevant lifecycle history only when explicitly authorized.
4. Historical records must be append-only.
5. The application runtime must not have permission to update or delete historical records through ordinary operations.
6. Privileged status changes must also generate appropriate platform audit events.
7. Sensitive administrative reasons must be protected from unauthorized disclosure.

#### 3.9.10 Data Integrity and Lifecycle

1. Every history record must reference an existing firm.
2. Historical status transitions must remain immutable.
3. Corrections must be represented by additional traceable events rather than silently changing existing records.
4. Timestamps must be stored in UTC.
5. Firm closure and retention policies must account for lifecycle history.
6. History records must be retained according to the platform's approved audit and legal retention policy.

#### 3.9.11 Design Notes and Open Decisions

- `firm.status` stores the firm's current lifecycle status.
- `firm_status_history` stores the historical sequence of status transitions.
- The exact permitted status-transition matrix will be finalized during business workflow design.
- Automated suspension and reactivation rules will be defined separately.
- The relationship between platform audit events and firm status history will be finalized during audit implementation.
- Database transaction boundaries and append-only permissions must preserve consistency between current status and historical records.

---

### 3.10 User

#### 3.10.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `user` |
| Purpose | Stores the central identity of every person who can access the platform |
| Ownership | User Identity-Owned |
| Tenant Scope | Global identity; not directly owned by an individual firm |
| Primary Key | `id` |
| Related Tables | `user_credential`, `user_session`, `user_mfa`, `firm_membership`, `customer`, `platform_user` |
| RLS Requirement | Restricted global identity access; no unrestricted firm-level access |

#### 3.10.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Unique central user identifier |
| `email` | VARCHAR(254) | Yes | NULL | Normalized email address |
| `email_verified_at` | TIMESTAMPTZ | Yes | NULL | Email verification timestamp |
| `phone_number` | VARCHAR(20) | Yes | NULL | Phone number in normalized international format |
| `phone_verified_at` | TIMESTAMPTZ | Yes | NULL | Phone verification timestamp |
| `display_name` | VARCHAR(200) | No | None | User's display name |
| `user_status` | VARCHAR(30) | No | `'pending'` | Overall user account status |
| `preferred_language` | VARCHAR(20) | No | `'en'` | User's preferred application language |
| `last_login_at` | TIMESTAMPTZ | Yes | NULL | Most recent successful login timestamp |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Account creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Last modification timestamp |
| `deleted_at` | TIMESTAMPTZ | Yes | NULL | Soft-deletion timestamp, where applicable |

#### 3.10.3 Primary Key

- `id` shall be the primary key.
- UUID shall be used as the central user identifier.
- The user ID shall remain stable throughout the account lifecycle.
- The same user ID must be referenced by all associated authentication and access-control records.

#### 3.10.4 Unique Constraints

- Email addresses must be normalized before comparison.
- A non-null normalized email address must be globally unique among active identity records.
- Phone numbers must not be assumed globally unique because a phone number may be shared by family members or business contacts.
- Email and phone verification status must be maintained independently.
- Unique email enforcement must account for the approved soft-deletion and account-recovery policy.

#### 3.10.5 Check Constraints

1. `display_name` must not be empty or contain only whitespace.
2. `user_status` must contain an approved value.
3. `preferred_language` must contain an approved language identifier.
4. At least one supported login or identity contact method must be available before account activation.
5. A verification timestamp must be NULL until the corresponding verification succeeds.

Proposed user statuses:

- `pending`
- `active`
- `suspended`
- `locked`
- `deactivated`

#### 3.10.6 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `uq_user_email_active` | Normalized email for active, non-deleted users | Prevent duplicate active email identities |
| `idx_user_phone_number` | `phone_number` | Support authorized phone-based identity lookup |
| `idx_user_status` | `user_status` | Retrieve users by account status |
| `idx_user_created_at` | `created_at` | Support account administration and reporting |

The exact partial unique index predicate and email normalization implementation will be finalized during physical schema design.

#### 3.10.7 Identity and Relationship Rules

1. A person must have one central user identity within the platform.
2. A user may have relationships with multiple CA firms.
3. Firm-specific staff access must be managed through `firm_membership`.
4. Customer access to a firm must be managed through the appropriate firm-customer relationship.
5. Platform administrator privileges must be represented separately through `platform_user`.
6. A central user identity must not automatically grant access to any firm.
7. Access to a firm must be validated independently for every request.
8. Removing a user's relationship with one firm must not automatically delete the user's central identity or relationships with other firms.

#### 3.10.8 Security and Access Control

1. This table must not store plaintext passwords, OTPs, recovery codes, or authentication secrets.
2. Authentication credentials must be maintained in `user_credential` and related authentication tables.
3. Access to central identity information must be restricted to authorized backend services.
4. Firm users must not obtain unrestricted access to the global user directory.
5. Customer contact information must not be exposed to unrelated firms.
6. Account suspension and deactivation must be enforced by authentication and authorization services.
7. Email and phone verification must be completed before relying on those channels for account recovery or sensitive authentication operations.
8. Changes to important identity attributes must be auditable.
9. Sensitive identity information must be excluded or appropriately masked in application logs.

#### 3.10.9 Data Integrity and Lifecycle

1. A user identity must remain stable even when the user changes firms.
2. Account status changes must follow controlled lifecycle transitions.
3. Email and phone verification timestamps must accurately represent successful verification.
4. `updated_at` must be maintained on relevant changes.
5. Soft deletion must not bypass applicable retention or legal requirements.
6. Deactivating a user must not silently delete associated tax records, documents, invoices, payments, or audit history.
7. Account recovery must not create duplicate identities for an existing person without a controlled identity-resolution process.

#### 3.10.10 Design Notes and Open Decisions

- This table represents the central identity of a person, not their permissions within a firm.
- Password hashes and authentication-specific data will be stored separately.
- The identity may be associated with platform administration, firm staff membership, and customer relationships through separate tables.
- Email normalization and account deduplication rules will be finalized during authentication design.
- The precise rules for account merging, email changes, phone changes, and account recovery remain open.
- The physical RLS and access strategy for global user identity data will be finalized during database security implementation.

---

### 3.11 User Credential

#### 3.11.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `user_credential` |
| Purpose | Stores password authentication credentials and credential lifecycle information |
| Ownership | User Identity-Owned |
| Tenant Scope | Global user identity |
| Primary Key | `id` |
| Related Tables | `user`, `user_session`, `user_login_history` |
| RLS Requirement | Restricted global identity access; accessible only through authorized authentication services |

#### 3.11.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Primary key |
| `user_id` | UUID | No | None | Associated central user |
| `credential_type` | VARCHAR(30) | No | `'password'` | Authentication credential type |
| `password_hash` | VARCHAR(255) | Yes | NULL | Secure password hash |
| `password_algorithm` | VARCHAR(50) | Yes | NULL | Password hashing algorithm identifier |
| `password_changed_at` | TIMESTAMPTZ | Yes | NULL | Last password change timestamp |
| `must_change_password` | BOOLEAN | No | FALSE | Indicates whether password change is required |
| `failed_attempt_count` | INTEGER | No | 0 | Failed authentication attempt counter |
| `locked_until` | TIMESTAMPTZ | Yes | NULL | Temporary credential lock expiration |
| `credential_status` | VARCHAR(30) | No | `'active'` | Credential lifecycle status |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Last modification timestamp |
| `revoked_at` | TIMESTAMPTZ | Yes | NULL | Credential revocation timestamp |

#### 3.11.3 Primary Key and Relationships

- `id` shall be the primary key and use UUID.
- `user_id` shall reference `user.id`.
- Each credential record must belong to exactly one central user.
- The initial implementation will support password credentials.
- The schema must allow future authentication methods without storing unrelated credential data in the same password fields.

#### 3.11.4 Foreign Keys

| Column | References | Requirement |
|---|---|---|
| `user_id` | `user.id` | Mandatory central user relationship |

#### 3.11.5 Unique Constraints

- The initial password-authentication design shall allow only one active password credential per user.
- Credential replacement must follow a controlled update or revocation process.
- Historical credential information must not expose previous plaintext passwords or reusable password hashes.

#### 3.11.6 Check Constraints

1. `credential_type` must contain an approved value.
2. `credential_status` must contain an approved value.
3. `failed_attempt_count` must not be negative.
4. A password credential must have a non-null password hash.
5. A password credential must have an approved password algorithm identifier.
6. A revoked credential must have a non-null `revoked_at`.
7. A credential marked active must not have a revocation timestamp.

Proposed credential types:

- `password`

Proposed credential statuses:

- `active`
- `locked`
- `revoked`
- `expired`

#### 3.11.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `uq_user_credential_user_type` | `user_id`, `credential_type` | Prevent duplicate credentials of the same type |
| `idx_user_credential_status` | `credential_status` | Retrieve credentials by status |
| `idx_user_credential_locked_until` | `locked_until` | Support temporary lock management |

The exact uniqueness predicate will be finalized during physical schema design if historical revoked credentials are retained in this table.

#### 3.11.8 Password Security Requirements

1. Passwords must never be stored in plaintext.
2. Passwords must be hashed using Argon2id or another approved modern password-hashing algorithm.
3. Unique salts must be generated and managed by the password-hashing implementation.
4. Password hashing parameters must follow current security recommendations.
5. Password hashes must never be returned through ordinary application APIs.
6. Password changes must require appropriate authentication and authorization.
7. Password reset operations must use verified recovery channels and short-lived reset challenges.
8. Passwords must not be written to application logs, audit payloads, or error messages.
9. Password verification must be performed only by trusted authentication services.

#### 3.11.9 Authentication Protection

1. Failed login attempts must be tracked and protected by rate limiting.
2. Temporary account locking must use controlled thresholds and expiration.
3. Credential lockout must not become an uncontrolled permanent account-denial mechanism.
4. Suspended or deactivated users must not authenticate successfully.
5. Credential revocation must prevent future authentication using that credential.
6. Security-sensitive credential changes must trigger appropriate audit and session-management actions.
7. Password reset and credential replacement must not expose whether unrelated users have registered accounts.

#### 3.11.10 Access Control and Data Protection

1. Only authorized authentication services may read or modify password credential records.
2. Firm administrators and staff must not directly retrieve password hashes.
3. Platform administrators must not receive unrestricted access to password hashes.
4. Database access to this table must be restricted to the minimum required privileges.
5. Credential data must not be included in ordinary user-profile responses.
6. Credential changes and revocations must be auditable without recording the actual secret or password hash in audit descriptions.

#### 3.11.11 Data Integrity and Lifecycle

1. A credential cannot exist without an associated user.
2. Credential changes must update `updated_at`.
3. Password changes must update `password_changed_at`.
4. Revoked credentials must remain unusable.
5. Credential history and retention requirements must be handled without unnecessarily retaining reusable password hashes.
6. User deactivation must trigger the appropriate credential and session revocation workflow.

#### 3.11.12 Design Notes and Open Decisions

- This table initially supports password-based authentication.
- OTP challenges, MFA secrets, and recovery methods will be maintained separately.
- Password hashing will use Argon2id as the proposed default.
- Authentication-provider integration may change the physical credential implementation later.
- Exact password policy, lockout thresholds, reset-token design, and credential-retention rules will be finalized during authentication security design.
- Authentication must remain independent of firm membership and tenant authorization.

---

### 3.12 User Session

#### 3.12.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `user_session` |
| Purpose | Stores authenticated session metadata and session lifecycle information |
| Ownership | User Identity-Owned |
| Tenant Scope | Global user identity |
| Primary Key | `id` |
| Related Tables | `user`, `user_credential`, `user_login_history`, `user_mfa` |
| RLS Requirement | Restricted global identity access; accessible only through authorized authentication services |

#### 3.12.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Primary key |
| `user_id` | UUID | No | None | Associated central user |
| `session_token_hash` | VARCHAR(255) | No | None | Cryptographic hash of the session token |
| `session_status` | VARCHAR(30) | No | `'active'` | Current session status |
| `authentication_method` | VARCHAR(30) | No | None | Method used to authenticate |
| `mfa_verified_at` | TIMESTAMPTZ | Yes | NULL | Time MFA verification succeeded, if required |
| `ip_address` | INET | Yes | NULL | IP address associated with the session |
| `user_agent` | TEXT | Yes | NULL | Client application and browser information |
| `device_label` | VARCHAR(150) | Yes | NULL | Human-readable device description |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Session creation timestamp |
| `last_activity_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Most recent validated session activity |
| `expires_at` | TIMESTAMPTZ | No | None | Session expiration timestamp |
| `revoked_at` | TIMESTAMPTZ | Yes | NULL | Session revocation timestamp |
| `revocation_reason` | VARCHAR(100) | Yes | NULL | Reason for session revocation |

#### 3.12.3 Primary Key and Relationships

- `id` shall be the primary key and use UUID.
- `user_id` shall reference `user.id`.
- Each session must belong to exactly one central user.
- A user may have multiple concurrent sessions across devices and browsers.
- Session identifiers must remain unique.

#### 3.12.4 Foreign Keys

| Column | References | Requirement |
|---|---|---|
| `user_id` | `user.id` | Mandatory central user relationship |

#### 3.12.5 Unique Constraints

- `session_token_hash` must be globally unique.
- Session token hashes must not be reused across different sessions.

#### 3.12.6 Check Constraints

1. `session_status` must contain an approved value.
2. `authentication_method` must contain an approved value.
3. `expires_at` must be later than `created_at`.
4. `last_activity_at` must not be earlier than `created_at`.
5. A revoked session must have a non-null `revoked_at`.
6. An active session must not have a revocation timestamp.

Proposed session statuses:

- `active`
- `expired`
- `revoked`

Proposed authentication methods:

- `password`
- `otp`
- `password_mfa`
- `oidc`

#### 3.12.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `uq_user_session_token_hash` | `session_token_hash` | Prevent duplicate session token hashes |
| `idx_user_session_user_status` | `user_id`, `session_status` | Retrieve sessions for a user |
| `idx_user_session_expires_at` | `expires_at` | Support session expiration processing |
| `idx_user_session_last_activity` | `last_activity_at` | Support inactivity and session cleanup |

#### 3.12.8 Session Security Requirements

1. Original session bearer tokens must never be stored in the database.
2. Session tokens must be cryptographically random and generated using a secure random generator.
3. Only token hashes must be persisted.
4. Session validation must verify the supplied token against its stored hash.
5. Sessions must have configurable expiration and inactivity limits.
6. Expired or revoked sessions must never authorize requests.
7. Session tokens must be protected against theft and replay.
8. Browser sessions should use appropriately configured secure, HttpOnly, and SameSite cookies.
9. Session rotation must be performed after authentication and other security-sensitive events where appropriate.
10. Session tokens must never appear in logs, URLs, analytics, or audit-event payloads.

#### 3.12.9 Session Revocation and Lifecycle

1. Users must be able to log out and revoke their current session.
2. Authorized security workflows must support revoking all sessions belonging to a user.
3. Password changes and account-security events must trigger the appropriate session-revocation policy.
4. Suspended, locked, or deactivated users must not continue accessing protected resources through existing sessions.
5. Session expiration must be enforced during authentication and authorization.
6. Background cleanup may remove expired session records according to the approved retention policy.
7. Session revocation must take effect within the defined security enforcement period.

#### 3.12.10 Tenant Isolation and Access Control

1. A session represents authentication of a central user, not authorization to a particular firm.
2. Firm access must be checked independently using the user's current firm membership or customer relationship.
3. A session must not grant access to all firms associated with the user automatically.
4. Firm switching must validate the relevant relationship and permissions.
5. Only authorized authentication services may read or modify session records.
6. Firm administrators must not retrieve session tokens or unrestricted session metadata belonging to users.
7. Platform-level session administration must follow a separately authorized and audited process.

#### 3.12.11 Data Integrity and Lifecycle

1. Every session must reference an existing user.
2. Session creation and revocation must be auditable.
3. `last_activity_at` must be updated through controlled session-validation logic.
4. Session expiration must be enforced independently of background cleanup.
5. Revocation records must preserve the reason and timestamp where applicable.
6. IP address and user-agent information must be retained only according to approved privacy and security retention policies.

#### 3.12.12 Design Notes and Open Decisions

- Session token hashing and validation mechanisms will be finalized during authentication implementation.
- Absolute session lifetime and inactivity timeout will be configurable according to security policy.
- The exact session rotation strategy will be defined during application security design.
- Concurrent-session limits may be introduced as a platform policy.
- Session invalidation across multiple application instances must be reliable.
- If an external identity provider manages sessions, the relationship between provider sessions and application sessions will be finalized during integration design.

---

### 3.13 User MFA

#### 3.13.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `user_mfa` |
| Purpose | Stores MFA enrollment information and authentication-factor lifecycle metadata |
| Ownership | User Identity-Owned |
| Tenant Scope | Global user identity |
| Primary Key | `id` |
| Related Tables | `user`, `user_session`, `user_login_history`, `user_recovery_method` |
| RLS Requirement | Restricted global identity access; accessible only through authorized authentication services |

#### 3.13.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Primary key |
| `user_id` | UUID | No | None | Associated central user |
| `mfa_method` | VARCHAR(30) | No | None | MFA authentication method |
| `secret_encrypted` | BYTEA | Yes | NULL | Encrypted MFA secret |
| `encryption_key_version` | VARCHAR(100) | Yes | NULL | Identifier of the encryption key version |
| `mfa_status` | VARCHAR(30) | No | `'pending'` | MFA enrollment status |
| `is_primary` | BOOLEAN | No | FALSE | Indicates the primary MFA method |
| `enrolled_at` | TIMESTAMPTZ | Yes | NULL | Successful enrollment timestamp |
| `last_verified_at` | TIMESTAMPTZ | Yes | NULL | Most recent successful MFA verification |
| `last_used_at` | TIMESTAMPTZ | Yes | NULL | Most recent use of this MFA method |
| `failed_attempt_count` | INTEGER | No | 0 | Failed verification attempt counter |
| `locked_until` | TIMESTAMPTZ | Yes | NULL | Temporary MFA verification lock expiration |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Last modification timestamp |
| `revoked_at` | TIMESTAMPTZ | Yes | NULL | MFA method revocation timestamp |

#### 3.13.3 Primary Key and Relationships

- `id` shall be the primary key and use UUID.
- `user_id` shall reference `user.id`.
- Each MFA record must belong to exactly one central user.
- A user may have multiple MFA methods where supported by platform policy.
- MFA methods must be independently enrolled, verified, and revocable.

#### 3.13.4 Foreign Keys

| Column | References | Requirement |
|---|---|---|
| `user_id` | `user.id` | Mandatory central user relationship |

#### 3.13.5 Unique Constraints

- Duplicate active enrollment of the same MFA method for the same user must be prevented where applicable.
- A user must have at most one primary active MFA method per supported method category.
- The physical schema must enforce uniqueness through appropriate partial unique indexes.

#### 3.13.6 Check Constraints

1. `mfa_method` must contain an approved value.
2. `mfa_status` must contain an approved value.
3. `failed_attempt_count` must not be negative.
4. An active MFA method must have completed enrollment and verification.
5. A revoked MFA method must have a non-null `revoked_at`.
6. A revoked MFA method must not remain marked as the primary active method.

Proposed MFA methods:

- `totp`

Proposed MFA statuses:

- `pending`
- `active`
- `locked`
- `revoked`

#### 3.13.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `idx_user_mfa_user_status` | `user_id`, `mfa_status` | Retrieve MFA methods for a user |
| `idx_user_mfa_user_primary` | `user_id`, `is_primary` | Retrieve the primary MFA method |
| `idx_user_mfa_locked_until` | `locked_until` | Support temporary lock management |

#### 3.13.8 MFA Enrollment and Verification Rules

1. MFA enrollment must begin with a pending state.
2. An MFA method must not become active until enrollment verification succeeds.
3. TOTP secrets must be generated using a cryptographically secure random generator.
4. The enrollment process must verify a valid TOTP code before activation.
5. MFA verification must enforce an appropriate time window and replay-protection policy.
6. Failed verification attempts must be rate-limited.
7. Repeated failed attempts must trigger controlled temporary protection.
8. Revoked MFA methods must not authenticate users.
9. MFA enrollment, replacement, and removal must require appropriate identity verification.
10. High-risk account changes must trigger additional authentication where required.

#### 3.13.9 Security and Data Protection

1. MFA secrets must be encrypted using an approved authenticated encryption mechanism.
2. Encryption keys must be managed through a dedicated key-management service.
3. Encryption keys must not be stored alongside encrypted MFA secrets.
4. The encryption key version must be retained to support controlled key rotation.
5. Plaintext MFA secrets must exist only temporarily within the authorized authentication process.
6. MFA secrets and OTP values must never appear in application logs, audit payloads, or error messages.
7. MFA secrets must not be returned through ordinary application APIs.
8. Only authorized authentication services may decrypt MFA secrets.
9. Platform administrators must not have unrestricted access to users' MFA secrets.
10. MFA changes and recovery operations must generate appropriate audit events.

#### 3.13.10 Access Control and Session Integration

1. Firm membership does not automatically establish MFA enrollment.
2. Mandatory MFA policies must be enforced according to user role and platform security policy.
3. Platform administrators must use mandatory MFA.
4. Firm administrators and staff must use MFA according to the approved firm security policy.
5. Successful MFA verification must be associated with the relevant authentication session.
6. An MFA-required session must not receive full authorization before the required verification succeeds.
7. Revoking an MFA method must trigger the appropriate security and session-management workflow.

#### 3.13.11 Data Integrity and Lifecycle

1. Every MFA record must reference an existing user.
2. Enrollment and revocation timestamps must accurately represent lifecycle events.
3. Changes must update `updated_at`.
4. MFA revocation must prevent future use of the affected method.
5. Re-enrollment must follow a fresh verification process.
6. Encryption-key rotation must preserve the ability to securely decrypt existing active secrets.
7. MFA recovery must follow a separately controlled recovery process.

#### 3.13.12 Design Notes and Open Decisions

- TOTP through authenticator applications is the proposed initial MFA mechanism.
- SMS OTP may be considered separately as an authentication or recovery channel, subject to risk assessment.
- Backup recovery codes will be handled through the recovery-method design.
- Exact TOTP parameters, replay protection, rate limits, and lockout thresholds will be finalized during authentication implementation.
- Key rotation and encryption implementation will be finalized during security architecture implementation.
- MFA enforcement policies will be defined separately for platform administrators, firm administrators, firm staff, and customers.

---

### 3.14 User Recovery Method

#### 3.14.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `user_recovery_method` |
| Purpose | Stores verified account recovery methods and their lifecycle information |
| Ownership | User Identity-Owned |
| Tenant Scope | Global user identity |
| Primary Key | `id` |
| Related Tables | `user`, `user_mfa`, `user_credential`, `user_session` |
| RLS Requirement | Restricted global identity access; accessible only through authorized authentication and recovery services |

#### 3.14.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Primary key |
| `user_id` | UUID | No | None | Associated central user |
| `recovery_method_type` | VARCHAR(30) | No | None | Type of recovery method |
| `recovery_value_encrypted` | BYTEA | Yes | NULL | Encrypted recovery contact information, where required |
| `recovery_code_hash` | VARCHAR(255) | Yes | NULL | Secure hash of a recovery code |
| `encryption_key_version` | VARCHAR(100) | Yes | NULL | Encryption key version identifier |
| `method_status` | VARCHAR(30) | No | `'pending'` | Recovery method lifecycle status |
| `is_verified` | BOOLEAN | No | FALSE | Indicates successful verification |
| `verified_at` | TIMESTAMPTZ | Yes | NULL | Successful verification timestamp |
| `last_used_at` | TIMESTAMPTZ | Yes | NULL | Most recent recovery use |
| `expires_at` | TIMESTAMPTZ | Yes | NULL | Optional recovery method expiration |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Last modification timestamp |
| `revoked_at` | TIMESTAMPTZ | Yes | NULL | Recovery method revocation timestamp |

#### 3.14.3 Primary Key and Relationships

- `id` shall be the primary key and use UUID.
- `user_id` shall reference `user.id`.
- Each recovery method must belong to exactly one central user.
- A user may register multiple recovery methods.
- Recovery methods must be independently verified, activated, and revoked.

#### 3.14.4 Foreign Keys

| Column | References | Requirement |
|---|---|---|
| `user_id` | `user.id` | Mandatory central user relationship |

#### 3.14.5 Unique Constraints

- Duplicate active recovery codes must not be allowed for the same user.
- A recovery code must identify only one recovery record.
- Recovery contact uniqueness must follow the approved account identity and recovery policy.
- The physical schema must use suitable unique indexes for active recovery records.

#### 3.14.6 Check Constraints

1. `recovery_method_type` must contain an approved value.
2. `method_status` must contain an approved value.
3. An active recovery method must be verified.
4. A revoked recovery method must have a non-null `revoked_at`.
5. A recovery code record must contain a non-null `recovery_code_hash`.
6. A contact-based recovery record must contain the required encrypted recovery value.
7. A verified recovery method must have a non-null `verified_at`.

Proposed recovery method types:

- `email`
- `phone`
- `backup_code`

Proposed method statuses:

- `pending`
- `active`
- `expired`
- `revoked`

#### 3.14.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `idx_user_recovery_method_user_status` | `user_id`, `method_status` | Retrieve recovery methods for a user |
| `idx_user_recovery_method_type` | `recovery_method_type` | Retrieve methods by type |
| `idx_user_recovery_method_expires_at` | `expires_at` | Support recovery expiration processing |

#### 3.14.8 Recovery Verification Rules

1. A recovery method must not become active until ownership verification succeeds.
2. Email and phone recovery must use verified contact channels.
3. Verification challenges must be cryptographically secure, short-lived, and single-use.
4. Recovery attempts must be rate-limited.
5. Backup recovery codes must be generated using a cryptographically secure random generator.
6. Backup recovery codes must be single-use.
7. Successful use of a backup recovery code must invalidate that code.
8. Recovery methods must be revocable independently.
9. Changes to recovery methods must require appropriate identity verification.
10. Recovery operations must not reveal whether unrelated accounts exist.

#### 3.14.9 Security and Data Protection

1. Recovery codes must never be stored in plaintext.
2. Recovery codes must be stored using an appropriate secure hashing mechanism.
3. Contact information used for recovery must be protected against unauthorized disclosure.
4. Encrypted recovery values must use approved authenticated encryption.
5. Encryption keys must be managed separately through an appropriate key-management service.
6. Plaintext recovery codes and verification challenges must not appear in logs or audit payloads.
7. Recovery endpoints must enforce rate limiting and abuse detection.
8. Recovery must not bypass mandatory identity verification or applicable MFA policies.
9. Recovery operations must generate appropriate security and audit events.
10. Platform administrators must not receive unrestricted access to recovery secrets.

#### 3.14.10 Access Control and Recovery Workflow

1. Only authorized authentication and recovery services may manage recovery methods.
2. A user must prove control of an appropriate recovery channel before using it.
3. Recovery must follow a controlled workflow with clear verification and completion states.
4. Successful account recovery may require password replacement and session revocation.
5. MFA recovery must follow additional verification requirements where applicable.
6. Recovery methods must not grant firm membership or customer access automatically.
7. Access to firm data must continue to be governed by independent authorization and tenant checks.

#### 3.14.11 Data Integrity and Lifecycle

1. Every recovery method must reference an existing user.
2. Verification and revocation timestamps must accurately reflect lifecycle events.
3. Expired and revoked methods must not be usable.
4. Changes must update `updated_at`.
5. Recovery code use must be recorded without retaining the original code.
6. Recovery methods must be retained or removed according to approved security and privacy policies.

#### 3.14.12 Design Notes and Open Decisions

- Email and phone recovery are proposed for verified account recovery.
- Backup recovery codes are proposed for MFA recovery.
- The number of backup codes issued per user will be finalized during authentication design.
- The exact verification challenge and rate-limiting mechanisms remain open.
- Recovery workflows must be designed to prevent account takeover and unauthorized cross-firm access.
- The relationship between MFA recovery, password reset, and session revocation will be finalized during authentication implementation.

---

### 3.15 User Login History

#### 3.15.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `user_login_history` |
| Purpose | Records successful and unsuccessful authentication attempts |
| Ownership | User Identity-Owned / System-Owned |
| Tenant Scope | Global user identity |
| Primary Key | `id` |
| Related Tables | `user`, `user_credential`, `user_session`, `user_mfa` |
| RLS Requirement | Restricted access through authorized authentication and security services |

#### 3.15.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Primary key |
| `user_id` | UUID | Yes | NULL | Identified user, if known |
| `login_identifier_hash` | VARCHAR(255) | Yes | NULL | Privacy-preserving hash of the submitted login identifier |
| `authentication_method` | VARCHAR(30) | No | None | Authentication method attempted |
| `login_result` | VARCHAR(30) | No | None | Authentication outcome |
| `failure_reason` | VARCHAR(100) | Yes | NULL | Categorized failure reason |
| `ip_address` | INET | Yes | NULL | Source IP address |
| `user_agent` | TEXT | Yes | NULL | Client and browser information |
| `correlation_id` | UUID | Yes | NULL | Identifier linking related authentication operations |
| `occurred_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Time of authentication attempt |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Record creation timestamp |

#### 3.15.3 Primary Key and Relationships

- `id` shall be the primary key and use UUID.
- `user_id` shall reference `user.id` when the user has been identified.
- `user_id` may be NULL when an authentication attempt cannot be associated with an existing user.
- Every record shall represent one authentication attempt.

#### 3.15.4 Foreign Keys

| Column | References | Requirement |
|---|---|---|
| `user_id` | `user.id` | Optional user reference |

#### 3.15.5 Unique Constraints

- No general unique constraint is required.
- Multiple authentication attempts may legitimately occur for the same user.
- Event duplication must be controlled through the authentication event-processing and idempotency design where necessary.

#### 3.15.6 Check Constraints

1. `authentication_method` must contain an approved value.
2. `login_result` must contain an approved value.
3. `failure_reason`, when present, must use an approved category.
4. A successful login must not contain a failure reason.

Proposed authentication methods:

- `password`
- `otp`
- `password_mfa`
- `oidc`

Proposed login results:

- `success`
- `failure`
- `blocked`
- `mfa_required`

#### 3.15.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `idx_user_login_history_user_time` | `user_id`, `occurred_at` | Retrieve login history for a user |
| `idx_user_login_history_ip_time` | `ip_address`, `occurred_at` | Investigate activity from an IP address |
| `idx_user_login_history_result_time` | `login_result`, `occurred_at` | Review failed and blocked authentication attempts |
| `idx_user_login_history_correlation_id` | `correlation_id` | Trace related authentication operations |

#### 3.15.8 Authentication Audit Requirements

1. Successful authentication attempts must generate login-history records.
2. Failed authentication attempts must generate login-history records.
3. Blocked attempts and MFA-required outcomes must be recorded.
4. Authentication methods must be identified accurately.
5. Failure reasons must use controlled categories rather than sensitive raw error messages.
6. Attempts involving unknown users must be recordable without creating a user account.
7. Authentication records must use UTC timestamps.
8. Login history must support investigation of suspicious authentication patterns.

#### 3.15.9 Security and Privacy Requirements

1. Passwords, OTP values, recovery codes, and session tokens must never be stored.
2. Raw login identifiers should not be stored unnecessarily.
3. The login identifier hash must use a privacy-preserving keyed hashing approach where appropriate.
4. IP addresses and user-agent information must be protected against unauthorized disclosure.
5. Access to login history must be restricted to authorized security and authentication services.
6. Firm administrators must not automatically receive access to global login history.
7. Login-history access must itself be auditable.
8. Login history must not be modifiable through ordinary application operations.
9. Suspicious activity detection must not rely exclusively on IP address because users may share networks or change addresses.

#### 3.15.10 Data Integrity and Lifecycle

1. Every login-history record must have a unique identifier.
2. Records must accurately reflect the authentication outcome.
3. Records must remain append-only.
4. Failed authentication attempts must not disappear merely because no user was identified.
5. Login history must follow approved security and privacy retention policies.
6. High-volume login history may require partitioning and archival as platform usage grows.

#### 3.15.11 Design Notes and Open Decisions

- Login history records authentication attempts, not general user activity.
- General business operations will be recorded through the separate audit-event design.
- Exact login failure categories will be finalized during authentication implementation.
- Privacy-preserving identifier hashing and retention policies will be finalized during security implementation.
- Automated detection and alerting rules will be defined separately.
- High-volume indexing and partitioning strategies will be reviewed before production deployment.

---

### 3.16 Invitation

#### 3.16.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `invitation` |
| Purpose | Manages invitations for users to join a firm or receive authorized platform access |
| Ownership | Relationship-Owned |
| Tenant Scope | Firm-scoped for firm invitations; platform-scoped for platform invitations |
| Primary Key | `id` |
| Related Tables | `firm`, `user`, `firm_membership`, `platform_user` |
| RLS Requirement | Firm invitations must be restricted to their owning firm; platform invitations require separate platform-level access controls |

#### 3.16.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Primary key |
| `firm_id` | UUID | Yes | NULL | Target firm for a firm invitation |
| `invited_email` | VARCHAR(255) | No | None | Email address of the intended recipient |
| `invitation_type` | VARCHAR(30) | No | None | Type of invitation |
| `invitation_token_hash` | VARCHAR(255) | No | None | Hash of the secure invitation token |
| `invitation_status` | VARCHAR(30) | No | `pending` | Current invitation status |
| `invited_by_user_id` | UUID | Yes | NULL | User who initiated the invitation |
| `invited_by_platform_user_id` | UUID | Yes | NULL | Platform administrator who initiated the invitation |
| `intended_role_id` | UUID | Yes | NULL | Role intended to be assigned after acceptance |
| `expires_at` | TIMESTAMPTZ | No | None | Invitation expiration timestamp |
| `accepted_at` | TIMESTAMPTZ | Yes | NULL | Invitation acceptance timestamp |
| `accepted_by_user_id` | UUID | Yes | NULL | User who accepted the invitation |
| `revoked_at` | TIMESTAMPTZ | Yes | NULL | Invitation revocation timestamp |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Last update timestamp |

#### 3.16.3 Primary Key and Relationships

- `id` shall be the primary key and use UUID.
- A firm invitation shall reference the target `firm`.
- A platform invitation shall not reference a firm unless a separately approved business requirement requires it.
- `invited_by_user_id` shall reference `user.id`.
- `invited_by_platform_user_id` shall reference `platform_user.id`.
- `intended_role_id` shall reference `role.id` when a role is applicable.
- `accepted_by_user_id` shall reference `user.id`.
- The intended role and invitation type must be validated against the invitation scope.

#### 3.16.4 Foreign Keys

| Column | References | Requirement |
|---|---|---|
| `firm_id` | `firm.id` | Required for firm invitations; NULL for platform invitations |
| `invited_by_user_id` | `user.id` | Optional, depending on invitation origin |
| `invited_by_platform_user_id` | `platform_user.id` | Optional, depending on invitation origin |
| `intended_role_id` | `role.id` | Optional where a role is applicable |
| `accepted_by_user_id` | `user.id` | Optional until acceptance |

#### 3.16.5 Unique Constraints

- `invitation_token_hash` must be globally unique.
- Multiple invitations may exist for the same email address over time.
- The system must prevent conflicting active invitations for the same recipient, invitation type, and firm or platform scope.
- An expired, revoked, or accepted invitation must not prevent a new valid invitation from being issued.

#### 3.16.6 Check Constraints

1. `invitation_type` must contain an approved value.
2. `invitation_status` must contain an approved value.
3. `invited_email` must not be empty.
4. `expires_at` must be later than `created_at`.
5. An accepted invitation must have `accepted_at` and `accepted_by_user_id`.
6. A revoked invitation must have `revoked_at`.
7. A firm invitation must have `firm_id`.
8. A platform invitation must follow platform-specific authorization rules.
9. The invitation origin must be unambiguous: it must not identify both a firm user and a platform user as the inviter.

Proposed invitation types:

- `firm_admin`
- `firm_staff`
- `customer`
- `platform_admin`

Proposed invitation statuses:

- `pending`
- `accepted`
- `expired`
- `revoked`

#### 3.16.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `uq_invitation_token_hash` | `invitation_token_hash` | Enforce unique invitation token hashes |
| `idx_invitation_firm_status` | `firm_id`, `invitation_status` | Retrieve invitations for a firm |
| `idx_invitation_email` | `invited_email` | Find invitations for a recipient |
| `idx_invitation_expiry` | `expires_at` | Identify expired invitations |
| `idx_invitation_status_created` | `invitation_status`, `created_at` | Review pending and historical invitations |

A partial unique index should be considered for active invitations, using normalized email addresses and the appropriate firm or platform scope.

#### 3.16.8 Invitation Security Requirements

1. Invitation tokens must be generated using a cryptographically secure random generator.
2. Only token hashes may be stored in the database.
3. Raw invitation tokens must be sent through an approved communication channel and must never be written to application logs.
4. Invitation tokens must be single-use.
5. Invitations must expire after a defined period.
6. Expired and revoked invitations must not be accepted.
7. The recipient's email identity must be verified before acceptance.
8. The system must validate the invitation's firm or platform scope before granting access.
9. The intended role must be validated against the inviter's authority.
10. Invitation acceptance and membership or platform-access creation must occur in a controlled transaction.
11. Invitation resending must not unintentionally create multiple valid tokens.
12. Invitation creation, resending, revocation, expiry and acceptance must be auditable.

#### 3.16.9 Tenant Isolation and Authorization

1. Firm invitations must always be associated with their owning firm.
2. Firm administrators must only manage invitations within their authorized firm.
3. Firm staff may invite users only when explicitly permitted by their assigned permissions.
4. One firm's invitation must never grant access to another firm.
5. Platform invitations must use a separate platform-level authorization process.
6. Invitation acceptance must not bypass firm membership or platform authorization checks.
7. A customer invitation must establish the appropriate firm-customer relationship and must not automatically grant staff permissions.
8. Cross-tenant access must be prevented through backend authorization and database tenant-isolation controls.

#### 3.16.10 Data Integrity and Lifecycle

1. Every invitation must have a unique identifier and token hash.
2. Invitation status transitions must follow an approved lifecycle.
3. An invitation may be accepted only once.
4. Acceptance must be recorded with the accepting user and timestamp.
5. Revoked invitations must remain unavailable for acceptance.
6. Expired invitations must be marked expired through a controlled process.
7. Historical invitation records must be retained according to the security and audit retention policy.
8. Changes to intended roles or invitation scope must require appropriate authorization and audit records.

#### 3.16.11 Design Notes and Open Decisions

- The invitation table supports firm and platform invitation workflows, but these workflows must remain separately authorized.
- `role` is a proposed reference; the final role model and role scope will be defined in the membership and authorization table designs.
- Exact invitation expiry periods and resend limits will be finalized during authentication and security implementation.
- Email delivery and verification will be handled through the notification and communication architecture.
- The precise rules for customer invitations and existing customer accounts will be finalized with the customer relationship design.
- The final physical design must review whether a single invitation table or separate firm and platform invitation tables provide the clearest security boundary.

---

### 3.17 Firm Membership

#### 3.17.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `firm_membership` |
| Purpose | Establishes a user's membership and access relationship with a CA firm |
| Ownership | Firm-Owned |
| Tenant Scope | Firm-scoped |
| Primary Key | `id` |
| Related Tables | `firm`, `user`, `membership_role`, `invitation` |
| RLS Requirement | Mandatory firm-level Row-Level Security using `firm_id` |

#### 3.17.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Primary key |
| `firm_id` | UUID | No | None | Owning firm and tenant identifier |
| `user_id` | UUID | No | None | Central user identity |
| `membership_type` | VARCHAR(30) | No | None | Type of firm membership |
| `membership_status` | VARCHAR(30) | No | `pending` | Current membership status |
| `joined_at` | TIMESTAMPTZ | Yes | NULL | Date and time membership became active |
| `suspended_at` | TIMESTAMPTZ | Yes | NULL | Membership suspension timestamp |
| `deactivated_at` | TIMESTAMPTZ | Yes | NULL | Membership deactivation timestamp |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Last update timestamp |
| `created_by_membership_id` | UUID | Yes | NULL | Membership that created this record |
| `updated_by_membership_id` | UUID | Yes | NULL | Membership that last updated this record |

#### 3.17.3 Primary Key and Relationships

- `id` shall be the primary key and use UUID.
- `firm_id` shall reference `firm.id`.
- `user_id` shall reference `user.id`.
- A user may have memberships with multiple firms.
- Each membership shall represent one user's relationship with one firm.
- Firm-specific permissions must be assigned through the membership and role-assignment model.
- A membership must not grant access to another firm's data.

#### 3.17.4 Foreign Keys

| Column | References | Requirement |
|---|---|---|
| `firm_id` | `firm.id` | Required |
| `user_id` | `user.id` | Required |
| `created_by_membership_id` | `firm_membership.id` | Optional; must belong to the same firm |
| `updated_by_membership_id` | `firm_membership.id` | Optional; must belong to the same firm |

The creator and updater references must be protected by same-firm relationship constraints, preferably using composite foreign keys involving `firm_id`.

#### 3.17.5 Unique Constraints

- A user must have no more than one membership record for the same firm.
- Proposed unique constraint: `(firm_id, user_id)`.
- Membership history should be maintained through status changes rather than creating duplicate membership records.
- If historical membership reactivation requires separate records, the lifecycle model must be reviewed before implementation.

#### 3.17.6 Check Constraints

1. `membership_type` must contain an approved value.
2. `membership_status` must contain an approved value.
3. An active membership must have `joined_at`.
4. A suspended membership must have `suspended_at`.
5. A deactivated membership must have `deactivated_at`.
6. A pending membership must not be treated as an authorized active membership.

Proposed membership types:

- `firm_admin`
- `firm_staff`

Proposed membership statuses:

- `pending`
- `active`
- `suspended`
- `deactivated`

#### 3.17.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `uq_firm_membership_firm_user` | `firm_id`, `user_id` | Prevent duplicate firm membership |
| `idx_firm_membership_user_status` | `user_id`, `membership_status` | Find a user's firm memberships |
| `idx_firm_membership_firm_status` | `firm_id`, `membership_status` | Retrieve active and inactive memberships within a firm |
| `idx_firm_membership_created_at` | `firm_id`, `created_at` | Support membership administration and reporting |

#### 3.17.8 Tenant Isolation Requirements

1. Every membership must contain a valid `firm_id`.
2. Membership records must be protected by firm-level RLS.
3. The application must establish the authorized firm context before accessing membership data.
4. A user must only access a firm when an active membership or another explicitly authorized relationship permits it.
5. Membership in one firm must not imply membership in another firm.
6. Composite foreign keys must enforce same-firm relationships for membership-owned records.
7. The database runtime role must not bypass tenant-isolation policies.
8. Firm switching must validate the user's membership and status before establishing the new firm context.

#### 3.17.9 Authorization Requirements

1. Membership establishes association with a firm; it does not independently define every permission.
2. Roles and permissions must be assigned through the membership-role authorization model.
3. Firm administrators may manage memberships only within their authorized firm.
4. Staff membership must not automatically grant firm-administration privileges.
5. Suspended or deactivated memberships must not authorize firm operations.
6. Platform administrators must use their separate platform authorization path.
7. Customer access must be managed through the separate customer relationship and access-grant model.
8. Membership creation, role changes, suspension, reactivation and deactivation must be auditable.

#### 3.17.10 Data Integrity and Lifecycle

1. Membership creation and invitation acceptance must follow controlled authorization workflows.
2. Membership activation must be recorded with an appropriate timestamp.
3. Suspension and deactivation must update the corresponding lifecycle fields.
4. Suspension must prevent new authorized operations under that membership.
5. Existing sessions or cached authorization associated with a suspended membership must be invalidated or revalidated.
6. Historical membership records must be retained according to the audit and retention policy.
7. Membership changes must not modify the user's central identity or credentials.
8. Firm closure must follow a controlled membership deactivation process.

#### 3.17.11 Design Notes and Open Decisions

- The central `user` table represents identity; `firm_membership` represents authorization association with a firm.
- The same user may have different roles in different firms.
- Customer relationships must remain separate from staff memberships.
- Detailed role assignment will be defined in `membership_role`.
- The policy for transferring, reactivating and permanently deactivating memberships will be finalized during authorization design.
- The final physical design must review whether membership types should remain in this table or be derived entirely from assigned roles.
- Same-firm composite foreign keys and RLS policies must be validated during database implementation.

---

### 3.18 Role

#### 3.18.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `role` |
| Purpose | Defines named authorization roles used to group permissions |
| Ownership | Platform-Owned / Firm-Owned, depending on role scope |
| Tenant Scope | Platform-global or firm-specific |
| Primary Key | `id` |
| Related Tables | `firm`, `role_permission`, `membership_role` |
| RLS Requirement | Firm-specific roles must be restricted by `firm_id`; platform-global roles require separate restricted access |

#### 3.18.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Primary key |
| `firm_id` | UUID | Yes | NULL | Owning firm for a firm-specific role; NULL for a platform-global role |
| `role_key` | VARCHAR(100) | No | None | Stable machine-readable role identifier |
| `role_name` | VARCHAR(150) | No | None | Display name of the role |
| `description` | TEXT | Yes | NULL | Purpose and responsibilities of the role |
| `role_scope` | VARCHAR(30) | No | None | Authorization scope |
| `is_system_role` | BOOLEAN | No | FALSE | Indicates a predefined system-managed role |
| `is_active` | BOOLEAN | No | TRUE | Indicates whether the role can be assigned |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Last update timestamp |
| `created_by_membership_id` | UUID | Yes | NULL | Firm membership that created the role |
| `updated_by_membership_id` | UUID | Yes | NULL | Firm membership that last updated the role |

#### 3.18.3 Primary Key and Relationships

- `id` shall be the primary key and use UUID.
- `firm_id` shall reference `firm.id` for firm-specific roles.
- A NULL `firm_id` shall identify a platform-global role.
- Firm-specific roles must be associated with exactly one firm.
- Platform-global roles must be managed through platform-level authorization.
- Roles shall group permissions through `role_permission`.
- Roles shall be assigned to firm memberships through `membership_role`.

#### 3.18.4 Foreign Keys

| Column | References | Requirement |
|---|---|---|
| `firm_id` | `firm.id` | Optional for platform-global roles |
| `created_by_membership_id` | `firm_membership.id` | Optional; same-firm constraint for firm roles |
| `updated_by_membership_id` | `firm_membership.id` | Optional; same-firm constraint for firm roles |

Creator and updater references for firm-specific roles must be validated against the same firm. Platform-global role management must use a separate platform authorization path.

#### 3.18.5 Unique Constraints

- Platform-global role keys must be unique within platform scope.
- Firm-specific role keys must be unique within their firm.
- Proposed uniqueness rules:
  - Unique `role_key` where `firm_id IS NULL`.
  - Unique `(firm_id, role_key)` where `firm_id IS NOT NULL`.
- The same role key may exist in different firms without creating a cross-tenant relationship.

#### 3.18.6 Check Constraints

1. `role_key` must not be empty.
2. `role_name` must not be empty.
3. `role_scope` must contain an approved value.
4. A platform-global role must have `firm_id IS NULL`.
5. A firm-specific role must have `firm_id IS NOT NULL`.
6. Platform-global roles must not use firm membership creator references.
7. System roles must not be modified or deleted through ordinary firm administration.

Proposed role scopes:

- `platform`
- `firm`

#### 3.18.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `uq_role_platform_key` | `role_key` | Unique platform-global role key; partial index |
| `uq_role_firm_key` | `firm_id`, `role_key` | Unique role key within a firm; partial index |
| `idx_role_firm_active` | `firm_id`, `is_active` | Retrieve active roles within a firm |
| `idx_role_scope` | `role_scope` | Identify roles by authorization scope |

#### 3.18.8 Authorization Requirements

1. Roles must group explicitly defined permissions.
2. A role assignment must not grant permissions outside its defined scope.
3. Firm administrators may manage firm-specific roles only when authorized.
4. Firm administrators must not modify platform-global roles.
5. System roles must be protected from unauthorized modification.
6. Deactivated roles must not be assigned to new memberships.
7. Changes to role definitions and permissions must be auditable.
8. The backend must evaluate permissions for each protected operation.
9. A role name alone must never be treated as proof of authorization.

#### 3.18.9 Tenant Isolation Requirements

1. Firm-specific roles must contain a valid `firm_id`.
2. Firm-specific roles must be protected by firm-level RLS.
3. Role-permission and membership-role relationships must preserve the role's scope.
4. A membership in one firm must not receive a role belonging to another firm.
5. Platform-global roles must remain outside ordinary firm-level role administration.
6. Composite foreign keys or equivalent database constraints must enforce same-firm relationships.

#### 3.18.10 Data Integrity and Lifecycle

1. Role creation must follow authorized administrative workflows.
2. Role keys must remain stable after creation.
3. Deactivation must be preferred over deletion when historical assignments exist.
4. Changes to role permissions must be recorded in audit history.
5. System roles must be managed through controlled platform processes.
6. Role changes must not silently bypass existing authorization checks or cached permission state.

#### 3.18.11 Design Notes and Open Decisions

- The initial system will require predefined roles for platform administrators, firm administrators and firm staff.
- Customer access must be modeled separately from staff membership roles.
- Exact predefined role keys and permission mappings will be finalized during authorization design.
- The possibility of custom firm roles will be retained, subject to controlled permission assignment.
- Platform-global and firm-specific roles must remain clearly separated.
- The final physical design must review partial unique indexes, role-scope constraints and same-firm composite foreign keys.

---

### 3.19 Permission

#### 3.19.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `permission` |
| Purpose | Defines individual actions that can be granted through authorization roles |
| Ownership | Platform-Owned |
| Tenant Scope | Platform-global permission catalogue |
| Primary Key | `id` |
| Related Tables | `role_permission`, `role` |
| RLS Requirement | Restricted platform-level access; firm users must not modify the permission catalogue |

#### 3.19.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Primary key |
| `permission_key` | VARCHAR(150) | No | None | Stable machine-readable permission identifier |
| `permission_name` | VARCHAR(150) | No | None | Human-readable permission name |
| `resource` | VARCHAR(100) | No | None | Business resource or module |
| `action` | VARCHAR(100) | No | None | Operation permitted on the resource |
| `description` | TEXT | Yes | NULL | Explanation of the permission |
| `permission_scope` | VARCHAR(30) | No | None | Authorization scope |
| `is_sensitive` | BOOLEAN | No | FALSE | Indicates whether the permission grants sensitive access |
| `is_active` | BOOLEAN | No | TRUE | Indicates whether the permission can be assigned |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Last update timestamp |

#### 3.19.3 Primary Key and Relationships

- `id` shall be the primary key and use UUID.
- `permission_key` shall be a stable identifier used by backend authorization checks.
- Permissions shall be associated with roles through `role_permission`.
- One permission may be assigned to multiple roles.
- A permission must not directly identify or authorize a particular user.
- Permission scope must be compatible with the scope of the role receiving it.

#### 3.19.4 Foreign Keys

- This table does not require a foreign key to `firm` or `user`.
- Role-to-permission relationships shall be defined in `role_permission`.
- Permission definitions must remain centrally controlled.

#### 3.19.5 Unique Constraints

- `permission_key` must be globally unique.
- Duplicate permission keys must not be permitted.
- Permission keys must remain stable because application authorization logic may depend on them.

#### 3.19.6 Check Constraints

1. `permission_key` must not be empty.
2. `permission_name` must not be empty.
3. `resource` must not be empty.
4. `action` must not be empty.
5. `permission_scope` must contain an approved value.
6. Proposed permission scopes:
   - `platform`
   - `firm`
7. A sensitive permission must require explicit authorization and appropriate audit coverage.

#### 3.19.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `uq_permission_key` | `permission_key` | Enforce unique permission identifiers |
| `idx_permission_resource_action` | `resource`, `action` | Retrieve permissions by resource and operation |
| `idx_permission_scope_active` | `permission_scope`, `is_active` | Retrieve active permissions by scope |
| `idx_permission_sensitive` | `is_sensitive` | Identify sensitive permissions for security review |

#### 3.19.8 Initial Permission Catalogue

The following are proposed permission examples. The final catalogue will be maintained as a controlled platform definition.

| Permission Key | Resource | Action | Scope |
|---|---|---|---|
| `firm.view` | firm | view | platform |
| `firm.create` | firm | create | platform |
| `firm.update` | firm | update | platform |
| `firm.suspend` | firm | suspend | platform |
| `customer.view` | customer | view | firm |
| `customer.create` | customer | create | firm |
| `customer.update` | customer | update | firm |
| `customer.deactivate` | customer | deactivate | firm |
| `itr.view` | itr | view | firm |
| `itr.create` | itr | create | firm |
| `itr.update` | itr | update | firm |
| `itr.assign` | itr | assign | firm |
| `itr.approve` | itr | approve | firm |
| `document.view` | document | view | firm |
| `document.upload` | document | upload | firm |
| `document.download` | document | download | firm |
| `document.delete` | document | delete | firm |
| `financial.invoice.view` | financial_invoice | view | firm |
| `financial.invoice.create` | financial_invoice | create | firm |
| `financial.payment.record` | financial_payment | record | firm |
| `financial.refund.view` | financial_refund | view | firm |
| `financial.refund.update` | financial_refund | update | firm |
| `membership.view` | membership | view | firm |
| `membership.manage` | membership | manage | firm |
| `role.view` | role | view | firm |
| `role.manage` | role | manage | firm |
| `report.view` | report | view | firm |
| `report.export` | report | export | firm |
| `audit.view` | audit | view | firm |
| `configuration.manage` | configuration | manage | firm |

#### 3.19.9 Authorization Requirements

1. Backend services must enforce permissions for every protected operation.
2. Permission checks must consider the authenticated user, active firm membership, role assignments and current firm context.
3. Permission names or UI visibility must never be treated as authorization by themselves.
4. Sensitive operations must require explicit permissions.
5. Platform permissions must not be granted through ordinary firm roles.
6. Firm permissions must not grant platform-level access.
7. Deactivated permissions must not be granted through new role assignments.
8. Authorization changes must be auditable.
9. The application must deny access by default when a permission is missing or cannot be evaluated.
10. Permission checks must be enforced server-side, not only in the frontend.

#### 3.19.10 Security and Tenant Isolation Requirements

1. The permission catalogue must be centrally managed.
2. Firm administrators must not create or modify platform permission definitions.
3. Firm-specific authorization must be restricted to the active firm context.
4. Sensitive permissions must receive additional security review.
5. Permissions must not bypass PostgreSQL RLS or backend tenant authorization.
6. Access to tax identities, financial records, documents and audit evidence must use appropriately scoped permissions.
7. Permission changes and sensitive permission assignments must generate audit evidence.

#### 3.19.11 Data Integrity and Lifecycle

1. Permission keys must be stable and unique.
2. Permission definitions must be introduced through controlled platform changes.
3. Deactivation must be preferred over deletion when a permission has historical assignments.
4. Permission changes must be reviewed for their effect on existing roles.
5. Application code and database permission definitions must remain consistent.
6. Automated tests must verify that protected operations enforce the intended permission.

#### 3.19.12 Design Notes and Open Decisions

- The permission catalogue is platform-controlled and shared across firms.
- Firms may have different role configurations, but they must use approved permission definitions.
- Custom firm roles will be supported through controlled role-permission assignments.
- Exact permission granularity will be reviewed as individual modules are designed.
- Record-level restrictions, such as staff accessing only assigned customers, require additional authorization rules and must not be assumed to be covered by ordinary resource permissions.
- Sensitive actions may require additional approval or separation-of-duties controls.
- The final catalogue will be reviewed against all application modules before implementation.

---

### 3.20 Role Permission

#### 3.20.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `role_permission` |
| Purpose | Defines which permissions are assigned to each authorization role |
| Ownership | Relationship-Owned |
| Tenant Scope | Platform-global or firm-specific, according to the associated role |
| Primary Key | `id` |
| Related Tables | `role`, `permission`, `firm` |
| RLS Requirement | Firm-specific role assignments must preserve firm isolation; platform role assignments require restricted platform access |

#### 3.20.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Primary key |
| `role_id` | UUID | No | None | Associated authorization role |
| `permission_id` | UUID | No | None | Assigned permission |
| `firm_id` | UUID | Yes | NULL | Firm scope for firm-specific role assignments |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Assignment creation timestamp |
| `created_by_membership_id` | UUID | Yes | NULL | Firm membership responsible for assignment |

#### 3.20.3 Primary Key and Relationships

- `id` shall be the primary key and use UUID.
- `role_id` shall reference `role.id`.
- `permission_id` shall reference `permission.id`.
- `firm_id` shall identify the tenant scope when the associated role is firm-specific.
- The assigned permission scope must be compatible with the associated role scope.
- A role may have multiple permissions.
- A permission may be assigned to multiple roles.
- The relationship must preserve the authorization scope of both records.

#### 3.20.4 Foreign Keys

| Column | References | Requirement |
|---|---|---|
| `role_id` | `role.id` | Required |
| `permission_id` | `permission.id` | Required |
| `firm_id` | `firm.id` | Required for firm-specific assignments; NULL for platform-global assignments |
| `created_by_membership_id` | `firm_membership.id` | Optional; same-firm constraint for firm assignments |

Composite foreign keys or equivalent database constraints must ensure that firm-specific assignments reference a role belonging to the same firm.

#### 3.20.5 Unique Constraints

- A role must not receive the same permission more than once.
- Proposed unique constraint: `(role_id, permission_id)`.
- Duplicate role-permission assignments must be prevented.
- Historical changes must be retained through audit records rather than duplicate active assignments.

#### 3.20.6 Check Constraints

1. Required relationship identifiers must not be NULL.
2. Firm-specific assignments must have a valid `firm_id`.
3. Platform-global assignments must follow platform-level authorization rules.
4. A firm role must not receive a platform-scoped permission.
5. A platform role may receive only permissions explicitly approved for platform scope.
6. Firm creator references must belong to the same firm.

#### 3.20.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `uq_role_permission_role_permission` | `role_id`, `permission_id` | Prevent duplicate assignments |
| `idx_role_permission_permission` | `permission_id` | Find roles associated with a permission |
| `idx_role_permission_firm` | `firm_id` | Retrieve firm-scoped assignments |

#### 3.20.8 Authorization Requirements

1. Only authorized administrators may assign permissions to roles.
2. Firm administrators may modify permissions only for roles within their own firm.
3. Platform-global role permissions must be managed through controlled platform administration.
4. Firm roles must never receive platform-level permissions.
5. Deactivated permissions must not be newly assigned.
6. Deactivated roles must not receive new permission assignments.
7. Permission changes must take effect according to the approved authorization-cache invalidation strategy.
8. Every permission assignment or removal must generate an audit event.
9. High-risk permission assignments must require additional authorization where policy demands it.
10. Backend services must evaluate the resulting permissions before allowing protected operations.

#### 3.20.9 Tenant Isolation Requirements

1. Firm-specific assignments must remain within the owning firm.
2. RLS must protect firm-specific relationship records.
3. Composite foreign keys must enforce same-firm relationships.
4. Platform-global assignments must not become accessible through ordinary firm-level queries.
5. Firm administrators must not manipulate another firm's role-permission relationships.
6. Database runtime access must not bypass tenant-isolation controls.

#### 3.20.10 Data Integrity and Lifecycle

1. Every assignment must reference an existing role and permission.
2. Duplicate assignments must be prevented.
3. Permission removal must be controlled and auditable.
4. Role deactivation must prevent further assignment changes through ordinary firm administration.
5. Changes must not silently leave authorization caches with outdated permissions.
6. Historical authorization changes must remain available for security investigation.

#### 3.20.11 Design Notes and Open Decisions

- This table implements the many-to-many relationship between roles and permissions.
- Platform-global and firm-specific authorization relationships must remain separate in their enforcement.
- Record-level access restrictions, such as staff access to assigned customers, will require additional authorization logic.
- The final physical schema must validate composite foreign keys and the appropriate RLS policies.
- Permission changes and their effective timestamps will be covered by the audit design.
- The final authorization model must be tested for privilege escalation and cross-tenant access.

---

### 3.21 Membership Role

#### 3.21.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `membership_role` |
| Purpose | Assigns authorization roles to individual firm memberships |
| Ownership | Relationship-Owned |
| Tenant Scope | Firm-scoped |
| Primary Key | `id` |
| Related Tables | `firm`, `firm_membership`, `role`, `user` |
| RLS Requirement | Mandatory firm-level Row-Level Security using `firm_id` |

#### 3.21.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Primary key |
| `firm_id` | UUID | No | None | Owning firm and tenant identifier |
| `membership_id` | UUID | No | None | Firm membership receiving the role |
| `role_id` | UUID | No | None | Assigned authorization role |
| `assigned_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Role assignment timestamp |
| `assigned_by_membership_id` | UUID | Yes | NULL | Membership responsible for assignment |
| `expires_at` | TIMESTAMPTZ | Yes | NULL | Optional role assignment expiry |
| `revoked_at` | TIMESTAMPTZ | Yes | NULL | Role revocation timestamp |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Record creation timestamp |

#### 3.21.3 Primary Key and Relationships

- `id` shall be the primary key and use UUID.
- `firm_id` shall reference `firm.id`.
- `membership_id` shall reference `firm_membership.id`.
- `role_id` shall reference `role.id`.
- `assigned_by_membership_id` shall reference `firm_membership.id`.
- Every firm membership may receive multiple roles.
- Every firm-specific role may be assigned to multiple memberships within its owning firm.
- Role assignments must not cross firm boundaries.
- Platform-global roles must not be assigned through this table.

#### 3.21.4 Foreign Keys

| Column | References | Requirement |
|---|---|---|
| `firm_id` | `firm.id` | Required |
| `membership_id` | `firm_membership.id` | Required; same-firm relationship |
| `role_id` | `role.id` | Required; must be a firm-specific role belonging to the same firm |
| `assigned_by_membership_id` | `firm_membership.id` | Optional; must belong to the same firm |

Composite foreign keys involving `firm_id` must enforce same-firm membership and role relationships.

#### 3.21.5 Unique Constraints

- A membership must not receive the same role more than once as an active assignment.
- Proposed partial unique index: `(membership_id, role_id)` where `revoked_at IS NULL`.
- Historical revoked assignments may remain available for audit purposes.
- A role assignment must not be reactivated by creating conflicting active records.

#### 3.21.6 Check Constraints

1. Required relationship identifiers must not be NULL.
2. `expires_at`, when present, must be later than `assigned_at`.
3. `revoked_at`, when present, must not be earlier than `assigned_at`.
4. A role assignment must reference a firm-specific role.
5. The membership, assigned role and assigning membership must belong to the same firm.
6. An expired or revoked role assignment must not authorize new operations.

#### 3.21.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `uq_membership_role_active` | `membership_id`, `role_id` | Prevent duplicate active role assignments; partial index |
| `idx_membership_role_firm` | `firm_id` | Retrieve role assignments within a firm |
| `idx_membership_role_role` | `role_id` | Find memberships assigned to a role |
| `idx_membership_role_expiry` | `expires_at` | Identify assignments approaching expiry |

#### 3.21.8 Authorization Requirements

1. Only authorized administrators may assign roles to firm memberships.
2. A firm administrator may manage role assignments only within their authorized firm.
3. A user must have an active firm membership before the assigned role can authorize firm operations.
4. A suspended or deactivated membership must not receive effective permissions.
5. Expired or revoked role assignments must not grant permissions.
6. Role assignments must be evaluated together with the role's active status and associated permissions.
7. Platform-level roles must use a separate platform authorization mechanism.
8. Assignment, revocation and expiry events must be auditable.
9. High-risk role assignments must require additional authorization where policy demands it.
10. The backend must enforce the effective permission set for each protected operation.

#### 3.21.9 Tenant Isolation Requirements

1. Every record must contain a valid `firm_id`.
2. RLS must restrict records to the authorized firm.
3. Composite foreign keys must enforce same-firm membership and role relationships.
4. A user must not receive another firm's role through membership manipulation.
5. Platform-global authorization must remain separate from firm membership authorization.
6. Database runtime access must not bypass tenant-isolation policies.

#### 3.21.10 Data Integrity and Lifecycle

1. Role assignments must be created through controlled authorization workflows.
2. Role assignment and revocation must generate audit events.
3. Expired assignments must no longer contribute to effective permissions.
4. Revoked assignments must remain available for historical review.
5. Changes to roles or permissions must be reflected in authorization-cache invalidation.
6. Membership suspension or deactivation must invalidate effective access even if role assignments remain active.
7. Historical assignments must follow approved audit-retention policies.

#### 3.21.11 Design Notes and Open Decisions

- This table implements the relationship between firm memberships and authorization roles.
- Multiple roles per membership are supported.
- The same user may have different roles across different firms.
- Platform administrators must use a separate authorization path.
- Temporary role assignments may be supported through `expires_at`.
- Exact role activation, expiry and revocation processing will be finalized during authorization implementation.
- The final physical design must validate composite foreign keys, partial unique indexes and RLS policies.
- Security testing must include privilege escalation, cross-firm role assignment and revoked-role access attempts.

---

### 3.22 Customer Access Grant

#### 3.22.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `customer_access_grant` |
| Purpose | Defines explicit user access to individual customer relationships within a firm |
| Ownership | Firm-Owned |
| Tenant Scope | Firm-scoped |
| Primary Key | `id` |
| Related Tables | `firm`, `user`, `firm_membership`, `firm_customer` |
| RLS Requirement | Mandatory firm-level Row-Level Security using `firm_id` |

#### 3.22.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Primary key |
| `firm_id` | UUID | No | None | Owning firm and tenant identifier |
| `membership_id` | UUID | No | None | Firm membership receiving access |
| `firm_customer_id` | UUID | No | None | Customer relationship to which access is granted |
| `access_level` | VARCHAR(30) | No | None | Scope of customer access |
| `grant_reason` | TEXT | Yes | NULL | Business justification for granting access |
| `granted_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Grant creation timestamp |
| `granted_by_membership_id` | UUID | No | None | Authorized membership granting access |
| `expires_at` | TIMESTAMPTZ | Yes | NULL | Optional access expiration |
| `revoked_at` | TIMESTAMPTZ | Yes | NULL | Access revocation timestamp |
| `revoked_by_membership_id` | UUID | Yes | NULL | Membership responsible for revocation |
| `revocation_reason` | TEXT | Yes | NULL | Reason for revocation |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Last update timestamp |

#### 3.22.3 Primary Key and Relationships

- `id` shall be the primary key and use UUID.
- `firm_id` shall reference `firm.id`.
- `membership_id` shall reference `firm_membership.id`.
- `firm_customer_id` shall reference `firm_customer.id`.
- `granted_by_membership_id` shall reference `firm_membership.id`.
- `revoked_by_membership_id` shall reference `firm_membership.id`.
- Each grant shall associate one firm membership with one firm-customer relationship.
- Multiple customer access grants may exist for the same membership, allowing access to multiple customers.
- A membership may have different access grants for different customers.

#### 3.22.4 Foreign Keys

| Column | References | Requirement |
|---|---|---|
| `firm_id` | `firm.id` | Required |
| `membership_id` | `firm_membership.id` | Required; same-firm constraint |
| `firm_customer_id` | `firm_customer.id` | Required; same-firm constraint |
| `granted_by_membership_id` | `firm_membership.id` | Required; same-firm constraint |
| `revoked_by_membership_id` | `firm_membership.id` | Optional; same-firm constraint |

Composite foreign keys involving `firm_id` must ensure that the membership, customer relationship and granting membership all belong to the same firm.

#### 3.22.5 Unique Constraints

- Duplicate active grants for the same membership, customer relationship and access level must be prevented.
- Proposed partial unique index: `(membership_id, firm_customer_id, access_level)` where `revoked_at IS NULL`.
- Historical revoked grants may remain available for audit purposes.
- Expired grants must not authorize access even if their records remain active.

#### 3.22.6 Check Constraints

1. `access_level` must contain an approved value.
2. `grant_reason`, when provided, must not contain only whitespace.
3. `expires_at`, when present, must be later than `granted_at`.
4. `revoked_at`, when present, must not be earlier than `granted_at`.
5. A revoked grant must have an appropriate revocation record.
6. The membership, customer relationship and granting membership must belong to the same firm.
7. An expired or revoked grant must not authorize customer access.

Proposed access levels:

- `view`
- `manage`
- `full`

The final access-level definitions must be mapped to explicitly approved operations and must not independently bypass role permissions.

#### 3.22.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `uq_customer_access_grant_active` | `membership_id`, `firm_customer_id`, `access_level` | Prevent duplicate active grants; partial index |
| `idx_customer_access_grant_firm` | `firm_id` | Retrieve grants within a firm |
| `idx_customer_access_grant_membership` | `membership_id`, `revoked_at` | Retrieve customer access assigned to a membership |
| `idx_customer_access_grant_customer` | `firm_customer_id` | Identify users with access to a customer |
| `idx_customer_access_grant_expiry` | `expires_at` | Identify grants approaching expiry |

#### 3.22.8 Authorization Requirements

1. Customer access grants must be evaluated together with the user's active firm membership and assigned role permissions.
2. A grant must not independently provide access to operations that the user's role does not authorize.
3. A user must have the required role permission and a valid customer access grant when customer-level assignment restrictions apply.
4. Firm administrators may grant access only within their authorized firm.
5. Staff must not grant themselves additional customer access unless explicitly authorized.
6. Customer access must be denied when the grant is expired or revoked.
7. Suspended or deactivated memberships must not retain effective customer access.
8. Customer access grants must be revocable without changing the customer's identity or firm relationship.
9. Grant creation, modification and revocation must generate audit events.
10. Access to sensitive tax identifiers, financial information and private documents must additionally enforce their specific permissions and security policies.

#### 3.22.9 Tenant Isolation Requirements

1. Every grant must contain a valid `firm_id`.
2. RLS must restrict access grants to the owning firm.
3. Composite foreign keys must enforce same-firm membership and customer relationships.
4. A user must never receive access to another firm's customer through a manipulated grant.
5. Customer identity records must not become globally accessible merely because a firm-specific grant exists.
6. Database runtime access must not bypass tenant-isolation policies.
7. Backend authorization must validate the active firm context for every protected customer operation.

#### 3.22.10 Data Integrity and Lifecycle

1. Every grant must have a unique identifier and valid relationship references.
2. Grant creation must follow an authorized workflow.
3. Grant expiration must prevent further authorized operations.
4. Revocation must be recorded with the responsible membership and timestamp.
5. Historical grants must remain available for security and audit investigations.
6. Membership suspension or deactivation must invalidate effective customer access.
7. Changes to grants must invalidate relevant authorization caches.
8. Customer relationship termination must trigger a controlled review of associated access grants.

#### 3.22.11 Design Notes and Open Decisions

- This table supports record-level customer access restrictions.
- It complements, rather than replaces, role-based permissions.
- The initial design supports explicit customer-level grants.
- The final implementation must define whether firm administrators have implicit access to all firm customers or require explicit grants.
- The exact meaning of `view`, `manage` and `full` must be defined through authorization rules.
- Customer ownership, staff assignment and temporary access policies will be reviewed together.
- The final physical design must validate same-firm composite foreign keys, partial unique indexes and RLS policies.
- Security tests must cover unauthorized customer access, cross-firm access, expired grants and revoked grants.

---

### 3.23 Customer

#### 3.23.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `customer` |
| Purpose | Stores the central identity of an individual or customer account |
| Ownership | Customer Identity-Owned |
| Tenant Scope | Global customer identity |
| Primary Key | `id` |
| Related Tables | `user`, `customer_contact`, `firm_customer` |
| RLS Requirement | Restricted global identity access; no unrestricted firm-level customer directory access |

#### 3.23.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Primary key |
| `user_id` | UUID | No | None | Associated central user account |
| `customer_reference` | VARCHAR(50) | No | None | Globally unique internal customer reference |
| `customer_type` | VARCHAR(30) | No | `individual` | Type of customer identity |
| `customer_status` | VARCHAR(30) | No | `pending` | Overall customer identity status |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Last update timestamp |
| `deleted_at` | TIMESTAMPTZ | Yes | NULL | Soft-deletion timestamp, where legally permitted |

#### 3.23.3 Primary Key and Relationships

- `id` shall be the primary key and use UUID.
- `user_id` shall reference `user.id`.
- A customer identity must be associated with one central user account.
- One customer identity may have relationships with multiple CA firms through `firm_customer`.
- Customer identity must remain independent of individual firm relationships.
- Firm-specific tax information must not be stored in this table.
- The same customer must not need separate central login identities merely because they work with multiple CA firms.

#### 3.23.4 Foreign Keys

| Column | References | Requirement |
|---|---|---|
| `user_id` | `user.id` | Required |

#### 3.23.5 Unique Constraints

- `user_id` must be unique if the initial design maintains one customer identity per user account.
- `customer_reference` must be globally unique.
- PAN and other tax identifiers must not be used as global uniqueness constraints.
- Customer identity duplication and account merging must follow a controlled review process.

#### 3.23.6 Check Constraints

1. `customer_reference` must not be empty.
2. `customer_type` must contain an approved value.
3. `customer_status` must contain an approved value.
4. `deleted_at`, when present, must not be earlier than `created_at`.

Proposed customer types:

- `individual`
- `business`
- `other`

Proposed customer statuses:

- `pending`
- `active`
- `suspended`
- `deactivated`

The exact supported customer types will be finalized against the business requirements.

#### 3.23.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `uq_customer_user_id` | `user_id` | Prevent duplicate customer identities for the same user |
| `uq_customer_reference` | `customer_reference` | Enforce unique customer references |
| `idx_customer_status` | `customer_status` | Retrieve customers by identity status |
| `idx_customer_created_at` | `created_at` | Support customer registration reporting |

#### 3.23.8 Identity and Data Ownership Requirements

1. Central customer identity must be independent of any particular CA firm.
2. The central identity must not contain firm-specific customer notes, assignments, workflow status or tax records.
3. PAN and other firm-maintained tax identifiers must remain in `customer_tax_identity`.
4. Firm-specific customer relationships must be maintained through `firm_customer`.
5. Central contact information must follow the approved identity and contact-management model.
6. Customer identity changes must not automatically modify firm-specific tax or business records.
7. Customer account duplication and merging must require a controlled and auditable process.

#### 3.23.9 Security and Privacy Requirements

1. Customer identity information must be protected against unauthorized disclosure.
2. Firm users must not receive unrestricted access to the global customer identity directory.
3. Backend services must verify the user's authorized customer relationship before disclosing customer information.
4. Customer identity access must follow applicable privacy and data-protection requirements.
5. Customer account suspension must be handled separately from firm-specific customer relationship suspension.
6. Sensitive personal information must not be unnecessarily duplicated across firm-owned records.
7. Identity access and changes must be auditable where required.

#### 3.23.10 Data Integrity and Lifecycle

1. Every customer identity must have a unique identifier.
2. Customer identity creation must follow an authorized registration or onboarding workflow.
3. Customer status changes must follow an approved lifecycle.
4. Soft deletion must not bypass legal retention requirements.
5. Customer identity deletion must not automatically cascade-delete firm-owned tax, financial, filing or audit records.
6. Account merging must preserve valid firm relationships and historical references.
7. Customer suspension must trigger appropriate authentication and access-control checks.

#### 3.23.11 Design Notes and Open Decisions

- The customer table represents the central identity of a customer, not the customer's relationship with a particular CA firm.
- A customer may be associated with multiple CA firms through separate `firm_customer` records.
- One central user account is intended to support customer login across associated firms.
- Firm-specific PAN and tax identity records remain separately owned by each firm.
- The relationship between `customer` and `user` assumes a one-to-one association in the initial design.
- Business customers may eventually require multiple authorized individual users; this requirement will need a separate organization-contact and authorization model.
- Account recovery, customer identity merging and duplicate detection will be finalized during the customer identity design review.
- The final physical design must review global identity access, soft deletion and the one-to-one user relationship.

---

### 3.24 Customer Contact

#### 3.24.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `customer_contact` |
| Purpose | Stores customer contact details and their verification status |
| Ownership | Customer Identity-Owned |
| Tenant Scope | Global customer identity |
| Primary Key | `id` |
| Related Tables | `customer`, `user` |
| RLS Requirement | Restricted global identity access; no unrestricted firm-level access |

#### 3.24.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Primary key |
| `customer_id` | UUID | No | None | Associated customer identity |
| `contact_type` | VARCHAR(30) | No | None | Type of contact information |
| `contact_value_encrypted` | BYTEA | No | None | Encrypted contact value |
| `contact_value_hash` | VARCHAR(255) | Yes | NULL | Keyed hash for privacy-preserving matching |
| `is_primary` | BOOLEAN | No | FALSE | Indicates the customer's primary contact |
| `is_verified` | BOOLEAN | No | FALSE | Indicates successful contact verification |
| `verified_at` | TIMESTAMPTZ | Yes | NULL | Contact verification timestamp |
| `is_active` | BOOLEAN | No | TRUE | Indicates whether the contact method is active |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Last update timestamp |
| `deleted_at` | TIMESTAMPTZ | Yes | NULL | Soft-deletion timestamp |

#### 3.24.3 Primary Key and Relationships

- `id` shall be the primary key and use UUID.
- `customer_id` shall reference `customer.id`.
- One customer may have multiple contact records.
- A contact record must belong to exactly one customer.
- Contact information must remain independent of firm-specific customer relationships.
- Central login identity and authentication channels must remain consistent with the central `user` account.

#### 3.24.4 Foreign Keys

| Column | References | Requirement |
|---|---|---|
| `customer_id` | `customer.id` | Required |

#### 3.24.5 Unique Constraints

- Duplicate active contact records for the same customer and contact type must be prevented where appropriate.
- Only one active primary email contact should exist per customer.
- Only one active primary phone contact should exist per customer.
- Global contact uniqueness must not be assumed.
- The same email address or phone number may be associated with multiple customer identities where permitted by the account policy.

#### 3.24.6 Check Constraints

1. `contact_type` must contain an approved value.
2. `contact_value_encrypted` must not be empty.
3. A verified contact must have `verified_at`.
4. An unverified contact must not be treated as an authenticated contact channel.
5. An inactive contact must not be used for new notifications or authentication.
6. A deleted contact must not remain an active primary contact.

Proposed contact types:

- `email`
- `phone`

#### 3.24.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `idx_customer_contact_customer` | `customer_id` | Retrieve contact methods for a customer |
| `idx_customer_contact_type_active` | `contact_type`, `is_active` | Retrieve active contact methods by type |
| `idx_customer_contact_value_hash` | `contact_value_hash` | Support privacy-preserving contact matching |
| `uq_customer_contact_primary_email` | `customer_id` | Enforce one active primary email; partial index |
| `uq_customer_contact_primary_phone` | `customer_id` | Enforce one active primary phone; partial index |

#### 3.24.8 Contact Verification Requirements

1. Email addresses must be verified before being used as verified email channels.
2. Phone numbers must be verified before being used as verified phone channels.
3. Verification must use short-lived, single-use challenges.
4. Verification codes must never be stored in plaintext.
5. Verification attempts must be rate-limited.
6. Contact changes must require appropriate re-verification.
7. Verification timestamps must be recorded.
8. Contact verification must not automatically grant firm membership or business-record access.

#### 3.24.9 Security and Privacy Requirements

1. Contact values must be encrypted at rest using approved authenticated encryption.
2. Encryption keys must be managed separately through an approved key-management service.
3. Key versions must be tracked to support encryption-key rotation.
4. Raw contact information must not appear unnecessarily in logs or audit descriptions.
5. Contact lookup must use an approved keyed-hashing mechanism where matching is required.
6. Access to contact information must be restricted to authorized identity and communication services.
7. Ordinary firm users must not receive unrestricted access to global customer contact records.
8. Contact disclosure and modification must follow applicable privacy requirements.

#### 3.24.10 Data Integrity and Lifecycle

1. Each contact record must belong to an existing customer.
2. Primary-contact changes must be handled transactionally.
3. A contact must be verified before being used as a verified communication channel.
4. Deactivated contacts must not receive new notifications.
5. Contact history must be retained according to approved privacy and audit policies.
6. Contact replacement must not silently alter historical business records.
7. Deletion must follow legal retention and account-lifecycle requirements.

#### 3.24.11 Design Notes and Open Decisions

- This table stores central customer contact information rather than firm-specific contact records.
- The relationship between customer contact information and central user login channels must be finalized to avoid inconsistent copies of email and phone data.
- Whether contact values should be stored exclusively in `user` and referenced here, or maintained as a separate customer contact collection, requires a final identity-model review.
- Encryption-key versioning and hashing algorithms will be finalized during security implementation.
- Contact verification and notification delivery will integrate with the authentication and communication services.
- The final physical design must review primary-contact uniqueness, duplicate handling, encryption, and soft deletion.

---

### 3.25 Firm Customer

#### 3.25.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `firm_customer` |
| Purpose | Establishes a firm-specific business relationship with a central customer identity |
| Ownership | Firm-Owned |
| Tenant Scope | Firm-scoped |
| Primary Key | `id` |
| Related Tables | `firm`, `customer`, `customer_tax_identity`, `customer_staff_assignment`, `customer_status_history`, `customer_access_grant` |
| RLS Requirement | Mandatory firm-level Row-Level Security using `firm_id` |

#### 3.25.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Primary key |
| `firm_id` | UUID | No | None | Owning firm and tenant identifier |
| `customer_id` | UUID | No | None | Central customer identity |
| `firm_customer_reference` | VARCHAR(50) | No | None | Customer reference within the firm |
| `relationship_status` | VARCHAR(30) | No | `pending` | Current firm-customer relationship status |
| `onboarded_at` | TIMESTAMPTZ | Yes | NULL | Date and time customer onboarding was completed |
| `relationship_started_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Start of the firm-customer relationship |
| `relationship_ended_at` | TIMESTAMPTZ | Yes | NULL | End of the firm-customer relationship |
| `relationship_end_reason` | TEXT | Yes | NULL | Reason for ending the relationship |
| `internal_notes` | TEXT | Yes | NULL | Firm-specific internal customer notes |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Last update timestamp |
| `created_by_membership_id` | UUID | Yes | NULL | Firm membership responsible for creation |
| `updated_by_membership_id` | UUID | Yes | NULL | Firm membership responsible for the last update |

#### 3.25.3 Primary Key and Relationships

- `id` shall be the primary key and use UUID.
- `firm_id` shall reference `firm.id`.
- `customer_id` shall reference `customer.id`.
- Each record shall represent one customer relationship with one CA firm.
- A customer may have relationships with multiple firms.
- Each firm must independently manage its relationship with a customer.
- Firm-specific tax information must be associated with this relationship rather than stored in the central customer identity.
- Customer identity information must not be duplicated unnecessarily across firms.

#### 3.25.4 Foreign Keys

| Column | References | Requirement |
|---|---|---|
| `firm_id` | `firm.id` | Required |
| `customer_id` | `customer.id` | Required |
| `created_by_membership_id` | `firm_membership.id` | Optional; must belong to the same firm |
| `updated_by_membership_id` | `firm_membership.id` | Optional; must belong to the same firm |

Composite foreign keys involving `firm_id` must enforce same-firm creator and updater relationships.

#### 3.25.5 Unique Constraints

- A central customer must have no more than one current relationship record per firm under the initial design.
- Proposed unique constraint: `(firm_id, customer_id)`.
- `firm_customer_reference` must be unique within its firm.
- The same customer reference may exist in different firms.
- Customer identity and PAN must not be used as global uniqueness constraints.

#### 3.25.6 Check Constraints

1. `firm_customer_reference` must not be empty.
2. `relationship_status` must contain an approved value.
3. `relationship_ended_at`, when present, must not be earlier than `relationship_started_at`.
4. An ended relationship must have an appropriate end reason where required by business policy.
5. An active relationship must not have an ending timestamp.
6. An inactive or ended relationship must not authorize new customer operations.

Proposed relationship statuses:

- `pending`
- `active`
- `suspended`
- `inactive`
- `ended`

#### 3.25.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `uq_firm_customer_firm_customer` | `firm_id`, `customer_id` | Prevent duplicate customer relationships within a firm |
| `uq_firm_customer_reference` | `firm_id`, `firm_customer_reference` | Enforce firm-specific customer reference uniqueness |
| `idx_firm_customer_status` | `firm_id`, `relationship_status` | Retrieve customers by relationship status |
| `idx_firm_customer_customer` | `customer_id` | Identify associated firm relationships through authorized identity services |
| `idx_firm_customer_created_at` | `firm_id`, `created_at` | Support firm customer reporting |

#### 3.25.8 Tenant Isolation Requirements

1. Every record must contain a valid `firm_id`.
2. RLS must restrict access to the owning firm.
3. All firm-specific customer records must reference the appropriate `firm_customer.id`.
4. Composite foreign keys must enforce same-firm relationships.
5. A customer relationship in one firm must not expose another firm's relationship records.
6. Customer identity access must be mediated through authorized application services.
7. The database runtime role must not bypass tenant-isolation policies.
8. Firm switching must validate the customer's relationship with the selected firm.
9. Customer records must not be retrieved using an unvalidated customer ID alone.

#### 3.25.9 Customer Relationship and Authorization Requirements

1. Customer onboarding must create or associate a central customer identity and establish a firm-specific relationship.
2. A customer may use the same central login account across multiple associated firms.
3. Each firm must independently manage its customer status and relationship lifecycle.
4. Staff assignments must reference the appropriate firm-customer relationship.
5. Customer access grants must be associated with the appropriate firm-customer relationship.
6. A suspended or ended relationship must not authorize ordinary customer operations.
7. Firm-specific internal notes must not be visible to other firms or customers.
8. Customer access must be validated for every protected operation.

#### 3.25.10 Data Integrity and Lifecycle

1. Every firm-customer relationship must reference an existing firm and customer.
2. Customer onboarding and relationship creation must follow an authorized workflow.
3. Relationship status changes must be recorded in `customer_status_history`.
4. Relationship termination must not automatically delete tax records, ITR filings, documents, invoices, payments or audit records.
5. Relationship termination must trigger a controlled review of staff assignments and customer access grants.
6. Historical customer relationships must be retained according to legal, contractual and audit requirements.
7. Rejoining a firm must follow the approved reactivation or historical relationship policy.
8. Changes to firm-specific customer information must generate appropriate audit records.

#### 3.25.11 Security and Privacy Requirements

1. Firm-specific customer information must be protected by tenant isolation.
2. Internal notes must be accessible only to appropriately authorized firm personnel.
3. Customer identity and contact information must not be copied into unrestricted firm records unnecessarily.
4. Tax identifiers must be stored in the firm-owned `customer_tax_identity` table.
5. Access to sensitive tax and financial information must require additional authorization.
6. Customer relationship access must be checked together with role permissions and applicable customer-level access grants.
7. Firm administrators must not access another firm's customer information.
8. Relationship creation, modification, suspension and termination must be auditable.

#### 3.25.12 Design Notes and Open Decisions

- This table is the central relationship boundary between a customer identity and a CA firm.
- One central customer identity may have multiple firm-customer relationships.
- PAN uniqueness will be enforced within the appropriate firm scope, not globally.
- Firm-specific customer references and internal notes must remain independently controlled.
- The initial design assumes one relationship record per customer per firm.
- The policy for rejoining a firm and preserving historical relationships will be finalized during lifecycle design.
- Customer relationship termination and retention rules must be reviewed against applicable legal requirements.
- The final physical schema must validate composite foreign keys, RLS policies and customer-access authorization.

---

### 3.26 Customer Tax Identity

#### 3.26.1 Table Information

| Attribute | Definition |
|---|---|
| Table Name | `customer_tax_identity` |
| Purpose | Stores tax identification information maintained by a CA firm for its customer |
| Ownership | Firm-Owned |
| Tenant Scope | Firm-scoped |
| Primary Key | `id` |
| Related Tables | `firm`, `firm_customer` |
| RLS Requirement | Mandatory firm-level Row-Level Security using `firm_id` |

#### 3.26.2 Column Definitions

| Column | PostgreSQL Data Type | Nullable | Default | Description |
|---|---|---|---|---|
| `id` | UUID | No | Generated UUID | Primary key |
| `firm_id` | UUID | No | None | Owning firm and tenant identifier |
| `firm_customer_id` | UUID | No | None | Associated firm-customer relationship |
| `tax_identifier_type` | VARCHAR(30) | No | None | Type of tax identifier |
| `tax_identifier_encrypted` | BYTEA | No | None | Encrypted tax identifier |
| `tax_identifier_hash` | VARCHAR(255) | No | None | Keyed hash used for uniqueness and matching |
| `encryption_key_version` | VARCHAR(100) | No | None | Encryption key version reference |
| `is_primary` | BOOLEAN | No | TRUE | Indicates the primary identifier of this type |
| `verification_status` | VARCHAR(30) | No | `unverified` | Verification status |
| `verified_at` | TIMESTAMPTZ | Yes | NULL | Verification timestamp |
| `created_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | `CURRENT_TIMESTAMP` | Last update timestamp |
| `created_by_membership_id` | UUID | Yes | NULL | Firm membership responsible for creation |
| `updated_by_membership_id` | UUID | Yes | NULL | Firm membership responsible for last update |
| `deleted_at` | TIMESTAMPTZ | Yes | NULL | Controlled soft-deletion timestamp |

#### 3.26.3 Primary Key and Relationships

- `id` shall be the primary key and use UUID.
- `firm_id` shall reference `firm.id`.
- `firm_customer_id` shall reference `firm_customer.id`.
- Each tax identity record must belong to exactly one firm-customer relationship.
- A firm-customer relationship may have multiple tax identifiers of different approved types.
- Tax identity records must never be treated as globally shared customer identity information.
- The same PAN may exist in separate firm-specific records for different firms.

#### 3.26.4 Foreign Keys

| Column | References | Requirement |
|---|---|---|
| `firm_id` | `firm.id` | Required |
| `firm_customer_id` | `firm_customer.id` | Required; same-firm constraint |
| `created_by_membership_id` | `firm_membership.id` | Optional; same-firm constraint |
| `updated_by_membership_id` | `firm_membership.id` | Optional; same-firm constraint |

Composite foreign keys involving `firm_id` must enforce same-firm relationships for the customer and membership references.

#### 3.26.5 Unique Constraints

- Tax identifier uniqueness must be enforced within the appropriate firm and identifier type.
- Proposed unique constraint: `(firm_id, tax_identifier_type, tax_identifier_hash)` for non-deleted records.
- A customer must not have conflicting duplicate records for the same identifier type within the same firm.
- The same tax identifier may exist in separate firm-specific records for different firms.
- Global PAN uniqueness must not be enforced.

The hash must use a controlled keyed-hashing mechanism. Ordinary unkeyed hashing must not be used as the sole protection for predictable identifiers such as PAN.

#### 3.26.6 Check Constraints

1. `tax_identifier_type` must contain an approved value.
2. `tax_identifier_encrypted` must not be empty.
3. `tax_identifier_hash` must not be empty.
4. `encryption_key_version` must not be empty.
5. `verification_status` must contain an approved value.
6. A verified record must have `verified_at`.
7. A non-verified record must not be represented as verified.
8. Deleted records must not participate in active uniqueness rules.

Proposed identifier types:

- `pan`
- `tan`
- `gstin`
- `other`

Proposed verification statuses:

- `unverified`
- `pending`
- `verified`
- `rejected`

The supported identifier types may be extended as business requirements evolve.

#### 3.26.7 Indexes

| Index | Columns | Purpose |
|---|---|---|
| `uq_customer_tax_identity_firm_type_hash` | `firm_id`, `tax_identifier_type`, `tax_identifier_hash` | Enforce firm-scoped identifier uniqueness; partial index |
| `idx_customer_tax_identity_customer` | `firm_id`, `firm_customer_id` | Retrieve tax identifiers for a firm customer |
| `idx_customer_tax_identity_type_status` | `firm_id`, `tax_identifier_type`, `verification_status` | Retrieve identifiers by type and verification status |
| `idx_customer_tax_identity_created_at` | `firm_id`, `created_at` | Support controlled reporting and review |

#### 3.26.8 Encryption and Key Management Requirements

1. PAN and other sensitive tax identifiers must be encrypted at rest.
2. Authenticated encryption must be used.
3. Encryption keys must be managed through an approved key-management service.
4. Encryption keys must not be stored alongside encrypted tax identifiers.
5. Encryption-key versions must be recorded to support key rotation.
6. Key rotation must not require exposing plaintext identifiers to ordinary application users.
7. Only authorized backend services may decrypt tax identifiers.
8. Encryption and decryption operations must be restricted and auditable.
9. The keyed hash must use a separately managed secret key.
10. Hash-key rotation must preserve the ability to enforce uniqueness and perform authorized matching.

#### 3.26.9 Access Control Requirements

1. Access to tax identifiers must require explicit sensitive-data permissions.
2. Firm users must only access identifiers belonging to their own firm.
3. Access must also be restricted by the user's authorized customer relationship and applicable record-level grants.
4. Ordinary customer lists and search results must display masked identifiers.
5. Full PAN disclosure must require an authorized business operation.
6. Unauthorized access attempts must be denied and appropriately logged.
7. Firm administrators must not access another firm's tax identifiers.
8. Customer-facing access must be explicitly defined and must not expose internal firm data.

#### 3.26.10 Audit and Privacy Requirements

1. Creation, modification and deletion of tax identity records must generate audit events.
2. Sensitive identifier viewing or decryption must be auditable according to the approved access policy.
3. Audit records must not contain plaintext PAN or other sensitive identifiers.
4. Application logs must not contain plaintext tax identifiers.
5. Search and matching operations must use approved protected lookup mechanisms.
6. Tax identity records must follow applicable Indian tax, privacy and record-retention requirements.
7. Unauthorized modification or deletion must be prevented.

#### 3.26.11 Data Integrity and Lifecycle

1. Every tax identity record must reference a valid firm-customer relationship.
2. The firm and customer relationship must belong to the same tenant.
3. Verification status changes must be controlled and auditable.
4. Identifier changes must preserve appropriate historical evidence.
5. Soft deletion must not bypass legal retention requirements.
6. Tax identity changes must not silently modify historical ITR filing records.
7. Duplicate identifiers within the same firm must be detected and handled through a controlled process.
8. Deleted or superseded records must not be used for new filing operations.

#### 3.26.12 Design Notes and Open Decisions

- Tax identity records are firm-owned and associated with `firm_customer`.
- PAN uniqueness is enforced within the firm, not globally.
- Encryption, keyed hashing and key management must be implemented as separate security concerns.
- The exact PAN validation and normalization rules will be finalized during implementation.
- The policy for multiple identifiers of the same type per customer will be reviewed.
- Tax identifier history and replacement procedures must preserve filing integrity.
- The final physical design must validate composite foreign keys, RLS policies, partial unique indexes and encryption-key rotation procedures.
- Security testing must include cross-tenant PAN lookup, unauthorized decryption, duplicate PAN insertion and sensitive-data leakage through logs.

### 3.27 `customer_staff_assignment`

**Purpose**

The `customer_staff_assignment` table records the assignment of firm staff members to customers associated with the same CA firm.

It supports customer ownership, staff workload management, operational responsibility and customer-level access control.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Each assignment must belong to one firm.
- A staff member may be assigned only to customers belonging to the same firm.
- Mandatory PostgreSQL Row-Level Security (RLS) using `firm_id`.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `firm_customer_id` | UUID | No | Customer relationship within the firm |
| `membership_id` | UUID | No | Staff membership assigned to the customer |
| `assignment_role` | VARCHAR(30) | No | Responsibility type |
| `assignment_status` | VARCHAR(30) | No | Assignment lifecycle status |
| `assigned_at` | TIMESTAMPTZ | No | Assignment timestamp |
| `assigned_by_membership_id` | UUID | Yes | Membership that created the assignment |
| `ended_at` | TIMESTAMPTZ | Yes | Assignment end timestamp |
| `ended_by_membership_id` | UUID | Yes | Membership that ended the assignment |
| `end_reason` | TEXT | Yes | Reason for ending the assignment |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`.
- `(firm_id, firm_customer_id)` references the corresponding firm-customer relationship.
- `(firm_id, membership_id)` references the corresponding firm membership.
- `(firm_id, assigned_by_membership_id)` references the assigning membership.
- `(firm_id, ended_by_membership_id)` references the membership that ended the assignment.

Composite foreign keys or equivalent database-enforced constraints must ensure that all referenced memberships and customer relationships belong to the same firm.

**Assignment Roles**

Initial supported values:

- `primary`: Main staff member responsible for the customer.
- `secondary`: Additional staff member supporting the customer.
- `reviewer`: Staff member responsible for review.

The assignment-role catalogue may be expanded through controlled application and database changes.

**Assignment Statuses**

Initial supported values:

- `active`
- `ended`
- `cancelled`

An active assignment must not have an `ended_at` value. An ended or cancelled assignment must have an appropriate end timestamp and, where applicable, an end reason.

**Constraints and Uniqueness**

- A staff membership must belong to the same firm as the customer relationship.
- Only eligible, active firm staff memberships may receive new assignments.
- A membership must not receive duplicate active assignments for the same customer and assignment role.
- A customer may have multiple active staff assignments.
- A customer should have no more than one active primary assignment at a time.
- Historical assignments must be retained rather than overwritten.

A partial unique index should enforce one active primary assignment per firm-customer relationship.

**Indexes**

Proposed indexes:

- Index on `(firm_id, firm_customer_id, assignment_status)`.
- Index on `(firm_id, membership_id, assignment_status)`.
- Index on `(firm_id, assigned_at)`.
- Partial unique index for one active primary assignment per `firm_customer_id`.
- Index on `(firm_id, assignment_role, assignment_status)` where useful for workload reporting.

**Tenant Isolation and RLS**

- Every record must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Cross-firm assignment references must be rejected by database constraints.
- Staff must not gain access to a customer merely because an assignment record exists; required role permissions and customer access rules must also be satisfied.
- Firm administrators may manage assignments according to their permissions.
- Platform-level support access must follow separately controlled and audited procedures.

**Business and Security Rules**

- Assignment and reassignment must be performed through authorized backend services.
- Reassignment should end the previous assignment and create a new assignment record within a controlled transaction.
- Assignment changes must be auditable.
- Ending a staff membership must trigger a review or controlled termination of its active customer assignments.
- Assignment changes must invalidate relevant authorization caches.
- Staff must not be able to assign customers to themselves or others unless their permissions explicitly allow it.
- Historical assignments must remain available to authorized reviewers.

**Data Integrity and Lifecycle**

- Assignment creation, reassignment and termination must preserve historical records.
- Assignment timestamps must use UTC.
- Deleting a staff membership or customer relationship must not silently cascade-delete assignment history.
- Assignment records must be retained according to the applicable business and legal retention policy.

**Design Notes and Open Decisions**

- Whether firm administrators receive implicit access to all firm customers or require explicit assignments will be finalized in the authorization design.
- Whether a reviewer can also be the primary staff member requires a firm-level policy decision.
- Whether multiple reviewers may be assigned simultaneously should be configurable or explicitly defined.
- The system should support reassignment history and workload reporting without losing previous responsibility records.

### 3.28 `customer_status_history`

**Purpose**

The `customer_status_history` table records changes to a customer's relationship status within a particular CA firm.

It provides a historical record of customer onboarding, activation, suspension, reactivation, deactivation and relationship termination.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Each history record belongs to one firm-customer relationship.
- Mandatory PostgreSQL Row-Level Security (RLS) using `firm_id`.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `firm_customer_id` | UUID | No | Customer relationship within the firm |
| `previous_status` | VARCHAR(30) | Yes | Status before the change; null for initial status |
| `new_status` | VARCHAR(30) | No | Status after the change |
| `change_reason` | TEXT | Yes | Reason for the status change |
| `change_source` | VARCHAR(30) | No | Source of the change |
| `changed_by_membership_id` | UUID | Yes | Firm membership responsible for the change |
| `correlation_id` | UUID | Yes | Identifier linking related operations |
| `occurred_at` | TIMESTAMPTZ | No | Time the status change occurred |
| `created_at` | TIMESTAMPTZ | No | Time the history record was created |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`.
- `(firm_id, firm_customer_id)` references the corresponding firm-customer relationship.
- `(firm_id, changed_by_membership_id)` references the responsible firm membership.

Composite foreign keys or equivalent database-enforced constraints must ensure that the customer relationship and responsible membership belong to the same firm.

A null `changed_by_membership_id` is permitted for system-generated or automated changes.

**Supported Customer Relationship Statuses**

The initial status catalogue is:

- `pending`
- `active`
- `suspended`
- `inactive`
- `ended`

These values must remain consistent with the status values defined for `firm_customer`.

**Change Sources**

Initial supported values:

- `firm_admin`
- `firm_staff`
- `system`
- `automated_process`
- `platform_admin`

The application must validate that the source and actor are consistent. Platform administrators must not be represented as firm memberships.

**Constraints and Integrity Rules**

- `new_status` is mandatory.
- `previous_status` may be null only for initial status establishment.
- A status change must not record the same previous and new status.
- Initial status establishment must be distinguishable from subsequent transitions.
- A system-generated change may have no firm membership actor.
- A user-initiated change must identify the responsible actor.
- The current `firm_customer.relationship_status` must agree with the latest applicable status-history record.

**Indexes**

Proposed indexes:

- Index on `(firm_id, firm_customer_id, occurred_at DESC)`.
- Index on `(firm_id, new_status, occurred_at DESC)`.
- Index on `(firm_id, changed_by_membership_id, occurred_at DESC)` where useful.
- Index on `correlation_id`.

**Tenant Isolation and RLS**

- Every record must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Cross-firm customer and actor references must be rejected.
- Only authorized firm users may retrieve customer status history.
- Platform-level access must follow separately controlled and audited procedures.

**Business and Security Rules**

- Customer status changes must be performed through authorized backend services.
- Updating the current customer status and inserting its history record must occur in the same database transaction.
- Failed status changes must not leave a history record suggesting that the change succeeded.
- Status transitions must follow an approved transition policy.
- Status changes must be auditable.
- Staff must not be able to change a customer's status unless their permissions explicitly allow it.
- Customer suspension or termination must trigger the appropriate access-control and workflow checks.

**Data Integrity and Lifecycle**

- History records must be append-only.
- Application runtime permissions must not permit ordinary users to update or delete history records.
- Corrections must be recorded through additional controlled events rather than silently overwriting history.
- Timestamps must use UTC.
- Customer relationship deletion must not silently cascade-delete status history.
- Retention and archival must follow the applicable business and legal retention policy.

**Design Notes and Open Decisions**

- The exact permitted status-transition matrix will be finalized in the workflow design.
- Whether platform-initiated status changes should also generate a separate `platform_audit_event` will be defined in the audit design.
- The method for identifying the authoritative latest status-history record will be finalized during implementation.
- Status changes caused by bulk import must identify the import process and retain the relevant correlation reference.

### 3.29 `customer_relationship_history`

**Purpose**

The `customer_relationship_history` table records significant changes to the business relationship between a customer and a CA firm.

It preserves historical information about relationship lifecycle events and important relationship-level changes without overwriting previous records.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Each record belongs to one firm-customer relationship.
- Mandatory PostgreSQL Row-Level Security (RLS) using `firm_id`.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `firm_customer_id` | UUID | No | Customer relationship within the firm |
| `event_type` | VARCHAR(50) | No | Type of relationship event |
| `previous_value` | JSONB | Yes | Previous relationship information |
| `new_value` | JSONB | Yes | Updated relationship information |
| `event_description` | TEXT | Yes | Human-readable explanation |
| `event_source` | VARCHAR(30) | No | Source of the event |
| `recorded_by_membership_id` | UUID | Yes | Firm membership responsible for recording the event |
| `correlation_id` | UUID | Yes | Identifier linking related operations |
| `occurred_at` | TIMESTAMPTZ | No | Time the business event occurred |
| `created_at` | TIMESTAMPTZ | No | Time the history record was created |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`.
- `(firm_id, firm_customer_id)` references the corresponding firm-customer relationship.
- `(firm_id, recorded_by_membership_id)` references the responsible firm membership.

Composite foreign keys or equivalent database-enforced constraints must ensure that referenced customer relationships and memberships belong to the same firm.

A null `recorded_by_membership_id` is permitted for system-generated or automated events.

**Initial Event Types**

The initial event catalogue includes:

- `relationship_created`
- `relationship_updated`
- `relationship_activated`
- `relationship_suspended`
- `relationship_reactivated`
- `relationship_ended`
- `relationship_reopened`
- `engagement_terms_changed`
- `relationship_owner_changed`

Additional event types may be introduced through controlled application and database changes.

**Event Sources**

Initial supported values:

- `firm_admin`
- `firm_staff`
- `system`
- `automated_process`
- `platform_admin`

The application must validate that the event source and actor are consistent. Platform administrators must not be represented as firm memberships.

**Constraints and Data Integrity**

- `event_type` and `event_source` are mandatory.
- `previous_value` and `new_value`, when provided, must contain valid JSON objects.
- At least one of `previous_value`, `new_value` or `event_description` must provide meaningful event information.
- An event must reference an existing firm-customer relationship.
- A user-initiated event must identify its responsible actor.
- System-generated events must be distinguishable from user-initiated events.

**Indexes**

Proposed indexes:

- Index on `(firm_id, firm_customer_id, occurred_at DESC)`.
- Index on `(firm_id, event_type, occurred_at DESC)`.
- Index on `(firm_id, recorded_by_membership_id, occurred_at DESC)` where useful.
- Index on `correlation_id`.

**Tenant Isolation and RLS**

- Every record must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Cross-firm customer and actor references must be rejected.
- Only authorized firm users may retrieve relationship history.
- Platform-level access must follow separately controlled and audited procedures.

**Business and Security Rules**

- Significant relationship changes must be recorded through authorized backend services.
- Relationship updates and corresponding history insertion must occur in the same database transaction.
- Reassignment, suspension, reactivation and termination must retain historical records.
- Changes must be auditable and traceable to an actor or system process.
- Sensitive customer information must not be copied unnecessarily into JSON history fields.
- JSON values must be restricted to approved fields and validated against event-specific schemas.
- Secrets, passwords, OTPs, authentication tokens and unrestricted tax identifiers must never be stored in event payloads.

**Data Integrity and Lifecycle**

- History records must be append-only.
- Ordinary application runtime permissions must not permit updates or deletion of history records.
- Corrections must be recorded as additional events rather than silently overwriting existing history.
- Timestamps must use UTC.
- Deleting a firm-customer relationship must not silently cascade-delete its history.
- Retention and archival must follow applicable business and legal requirements.

**Design Notes and Open Decisions**

- The exact list of significant relationship events will be finalized with the customer lifecycle and workflow design.
- Whether engagement terms should eventually be stored in a dedicated versioned table rather than JSON history payloads remains open.
- The distinction between status changes and broader relationship changes must be maintained: status transitions belong in `customer_status_history`, while broader business relationship events belong here.
- The relationship history must not become a replacement for the formal security audit trail in `audit_event`.
- The treatment of bulk-import-generated relationship events will be finalized in the import design.

### 3.30 `customer_duplicate_match`

**Purpose**

The `customer_duplicate_match` table records potential duplicate customer matches identified within a CA firm.

It supports duplicate detection during customer onboarding, Excel imports and customer data maintenance. It also records review decisions so that possible duplicates can be resolved by authorized personnel.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Both customer relationships involved in a match must belong to the same firm.
- Mandatory PostgreSQL Row-Level Security (RLS) using `firm_id`.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `source_firm_customer_id` | UUID | No | Customer relationship being checked |
| `matched_firm_customer_id` | UUID | No | Potential matching customer relationship |
| `match_type` | VARCHAR(30) | No | Type of matching evidence |
| `match_score` | NUMERIC(5,2) | Yes | Matching confidence score |
| `match_details` | JSONB | No | Approved matching evidence and details |
| `review_status` | VARCHAR(30) | No | Review lifecycle status |
| `reviewed_by_membership_id` | UUID | Yes | Membership that reviewed the match |
| `reviewed_at` | TIMESTAMPTZ | Yes | Review timestamp |
| `review_notes` | TEXT | Yes | Review decision explanation |
| `created_at` | TIMESTAMPTZ | No | Detection record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`.
- `(firm_id, source_firm_customer_id)` references the corresponding firm-customer relationship.
- `(firm_id, matched_firm_customer_id)` references the corresponding firm-customer relationship.
- `(firm_id, reviewed_by_membership_id)` references the responsible firm membership.

Composite foreign keys or equivalent database-enforced constraints must ensure that both customer relationships and the reviewing membership belong to the same firm.

**Match Types**

Initial supported values:

- `pan_match`
- `name_match`
- `email_match`
- `phone_match`
- `composite_match`
- `manual_review`

Matching evidence must be handled according to its sensitivity. Sensitive tax identifiers must not be stored in plaintext in `match_details`.

**Review Statuses**

Initial supported values:

- `pending`
- `confirmed_duplicate`
- `not_duplicate`
- `requires_more_information`
- `dismissed`

A potential match must remain pending until an authorized reviewer makes a decision.

**Constraints and Data Integrity**

- Source and matched customer relationships must be different.
- Both relationships must belong to the same firm.
- `match_score`, when provided, must be between 0 and 100.
- `match_details` must be a JSON object.
- A completed review must identify the reviewer and review timestamp.
- A pending review must not contain a completed review timestamp.
- Duplicate candidate pairs should be normalized so that the same pair is not recorded twice in opposite directions for the same detection context.

**Indexes**

Proposed indexes:

- Index on `(firm_id, source_firm_customer_id, review_status)`.
- Index on `(firm_id, matched_firm_customer_id, review_status)`.
- Index on `(firm_id, review_status, created_at DESC)`.
- Index on `(firm_id, match_type)`.
- Index on `(firm_id, reviewed_by_membership_id, reviewed_at)` where useful.

**Tenant Isolation and RLS**

- Every record must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Cross-firm matching must be rejected.
- Only authorized firm users may view or review duplicate candidates.
- Platform-level access must follow separately controlled and audited procedures.

**Business and Security Rules**

- Duplicate detection must not automatically merge customer identities or firm-customer relationships.
- An authorized reviewer must confirm a duplicate before any merge or consolidation operation.
- A confirmed duplicate decision must trigger a separate controlled workflow.
- Matching algorithms must use approved data fields and configurable thresholds.
- Sensitive tax identifiers must be compared using controlled, firm-scoped cryptographic hashes where appropriate.
- Full tax identifiers must not be exposed in matching results or ordinary review screens.
- Duplicate review decisions must be auditable.
- Dismissed or rejected matches must remain available for appropriate historical review.
- Repeated detection jobs should avoid creating unnecessary duplicate candidate records.

**Data Integrity and Lifecycle**

- Detection records and review decisions must preserve the history of the review process.
- Changes to review status must be controlled and auditable.
- Customer records must not be deleted merely because a potential duplicate is identified.
- Any eventual customer merge must preserve document references, ITR records, financial records, audit history and legal retention requirements.
- Timestamps must use UTC.
- Retention and archival must follow the applicable business and legal retention policy.

**Design Notes and Open Decisions**

- The exact matching algorithm, scoring weights and thresholds will be finalized during the customer import and duplicate-detection design.
- Whether a separate review-history table is required for multiple review decisions will be assessed during detailed schema review.
- The canonical direction and uniqueness strategy for candidate customer pairs must be finalized during physical schema implementation.
- A confirmed duplicate must not automatically authorize merging records.
- The merge workflow, if introduced, must be separately designed with approval, rollback and audit requirements.

### 3.31 `assessment_year`

**Purpose**

The `assessment_year` table maintains the master list of assessment years supported by the ITR Management System.

It provides a centralized reference for assessment-year information used by firms, customer ITR records, workflows, reports and financial or operational records where applicable.

Assessment years must be configurable so that new years can be introduced without changing the database schema.

**Ownership and Tenant Scope**

- Ownership: Platform-Owned.
- Tenant scope: Global; no `firm_id` column.
- Assessment-year definitions are maintained centrally.
- Firms independently enable or disable supported assessment years through the `firm_assessment_year` table.
- Access to create or modify assessment-year definitions is restricted to authorized platform administrators.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `assessment_year_code` | VARCHAR(10) | No | Unique assessment-year identifier, such as `2025-26` |
| `assessment_year_name` | VARCHAR(50) | No | Display name of the assessment year |
| `financial_year_start` | DATE | No | First date of the corresponding financial year |
| `financial_year_end` | DATE | No | Last date of the corresponding financial year |
| `assessment_year_start` | DATE | No | First date of the assessment year |
| `assessment_year_end` | DATE | No | Last date of the assessment year |
| `filing_start_date` | DATE | Yes | Official or configured filing start date |
| `filing_due_date` | DATE | Yes | Applicable filing due date |
| `extended_due_date` | DATE | Yes | Extended filing due date, where applicable |
| `assessment_year_status` | VARCHAR(30) | No | Lifecycle status |
| `is_current` | BOOLEAN | No | Indicates the configured current assessment year |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `created_by_platform_user_id` | UUID | Yes | Platform administrator who created the record |
| `updated_by_platform_user_id` | UUID | Yes | Platform administrator who last updated the record |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `created_by_platform_user_id` references `platform_user.id`.
- `updated_by_platform_user_id` references `platform_user.id`.

**Initial Assessment Years**

The initial system configuration must support:

- Assessment Year `2025-26`
- Assessment Year `2026-27`
- Assessment Year `2027-28`

These are initial master-data records, not hardcoded application limitations.

Future assessment years must be added through authorized configuration or administration workflows.

**Assessment Year Statuses**

Initial supported values:

- `upcoming`
- `active`
- `closed`
- `archived`

The status must be managed through an approved lifecycle process.

**Constraints and Data Integrity**

- `assessment_year_code` must be unique.
- Assessment-year names and codes must not be empty.
- `financial_year_start` must be earlier than `financial_year_end`.
- `assessment_year_start` must be earlier than `assessment_year_end`.
- Filing dates, when provided, must follow the applicable date-validation rules.
- An extended due date must not precede the original filing due date.
- Only one assessment year may be marked as current at a time.
- Date ranges and assessment-year codes must be validated against the applicable Indian tax-year conventions.
- Closed or archived assessment years must not be silently deleted or reused.

**Indexes**

Proposed indexes:

- Unique index on `assessment_year_code`.
- Index on `assessment_year_status`.
- Index on `financial_year_start` and `financial_year_end`.
- Partial unique index allowing only one row where `is_current = TRUE`.

**Tenant Isolation and Access Control**

- This is a global reference table and does not use firm-level tenant RLS.
- Only authorized platform administrators may create, modify or archive assessment-year definitions.
- Firm users may retrieve assessment-year definitions required for their authorized operations.
- Firms must not modify global assessment-year master data.
- Firm-specific availability and configuration must be controlled through `firm_assessment_year`.

**Business and Security Rules**

- Assessment years must be managed as data rather than hardcoded application constants.
- The application must validate that an assessment year is available to a firm before creating or processing its ITR records.
- Changes to due dates and lifecycle status must be auditable.
- Changes to filing dates must not silently overwrite historical filing or compliance evidence.
- The system must distinguish global assessment-year availability from firm-specific enablement.
- Official statutory dates must be verified before being configured; the system must not assume that all ITR categories share the same due date.

**Data Integrity and Lifecycle**

- Assessment-year master records should be retained for historical reporting and audit purposes.
- Existing ITR records must retain their assessment-year references even when an assessment year is closed or archived.
- Assessment-year records must not be cascade-deleted when referenced by customer ITR records.
- All timestamps must use UTC.
- Changes to assessment-year master data must preserve an audit trail.

**Design Notes and Open Decisions**

- The relationship between financial years and assessment years must be validated against Indian income-tax terminology and applicable statutory rules.
- Filing due dates may differ by taxpayer category and filing type. If the product needs to manage these differences, a dedicated filing-deadline configuration table may be required.
- Whether multiple assessment years may be marked as active for operational processing will be defined separately from the single `is_current` designation.
- The meaning of `is_current` must be finalized: it may represent the system's default assessment year rather than the only year available for processing.
- The initial assessment-year records must be verified before production deployment.

### 3.32 `firm_assessment_year`

**Purpose**

The `firm_assessment_year` table defines which assessment years are enabled for each CA firm.

It establishes a firm-specific relationship between the global `assessment_year` master records and individual firms.

It allows firms to control their operational assessment-year availability without modifying the platform-wide assessment-year configuration.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Each record belongs to one CA firm.
- Mandatory PostgreSQL Row-Level Security (RLS) using `firm_id`.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `assessment_year_id` | UUID | No | Reference to global assessment-year master |
| `is_enabled` | BOOLEAN | No | Indicates whether the firm has enabled the assessment year |
| `firm_display_name` | VARCHAR(100) | Yes | Optional firm-specific display label |
| `configuration` | JSONB | No | Firm-specific assessment-year configuration |
| `enabled_at` | TIMESTAMPTZ | Yes | Time the assessment year was enabled |
| `disabled_at` | TIMESTAMPTZ | Yes | Time the assessment year was disabled |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `created_by_membership_id` | UUID | Yes | Firm membership that created the record |
| `updated_by_membership_id` | UUID | Yes | Firm membership that last updated the record |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`.
- `assessment_year_id` references `assessment_year.id`.
- `(firm_id, created_by_membership_id)` references the corresponding firm membership.
- `(firm_id, updated_by_membership_id)` references the corresponding firm membership.

Composite foreign keys or equivalent database-enforced constraints must ensure that the creating and updating memberships belong to the same firm.

**Uniqueness**

- A firm must have only one configuration record for a given assessment year.
- Unique constraint on `(firm_id, assessment_year_id)`.
- Disabling and subsequently re-enabling an assessment year must update the existing configuration record rather than create duplicate records.
- Historical enablement and disablement events must be preserved through the audit mechanism.

**Constraints and Data Integrity**

- `firm_id` and `assessment_year_id` are mandatory.
- `is_enabled` must be explicitly defined.
- `configuration` must contain a valid JSON object.
- If `is_enabled = TRUE`, `enabled_at` must be populated and `disabled_at` must be null.
- If `is_enabled = FALSE` after a previous enablement, `disabled_at` must be populated.
- A firm cannot enable an assessment year that is not available in the global assessment-year master configuration.
- The global assessment-year status must be considered when evaluating whether the firm can process new ITR records.

**Indexes**

Proposed indexes:

- Unique index on `(firm_id, assessment_year_id)`.
- Index on `(firm_id, is_enabled)`.
- Index on `(assessment_year_id, is_enabled)`.
- Index on `(firm_id, enabled_at)` where useful.

**Tenant Isolation and RLS**

- Every record must contain `firm_id`.
- RLS must restrict access to the current tenant.
- A firm must not read or modify another firm's assessment-year configuration.
- Cross-firm membership references must be rejected.
- Global assessment-year master records must remain protected from firm-level modification.

**Business and Security Rules**

- Firm administrators may enable or disable assessment years according to their permissions.
- Firm staff may retrieve assessment-year configurations required for authorized work.
- Assessment-year availability must be checked before creating new customer ITR records.
- Disabling an assessment year must not delete existing ITR records or historical data.
- Existing records for a disabled assessment year must remain accessible to authorized users, subject to applicable business rules.
- Changes to enablement status and configuration must be audited.
- Firm-specific settings must not override platform-level statutory restrictions or global assessment-year closure.
- Assessment-year enablement must not be treated as authorization to access every customer's records.

**Data Integrity and Lifecycle**

- Assessment-year configuration records must be retained for historical reference.
- Disabling an assessment year must preserve its existing relationships and records.
- Re-enablement must preserve the existing configuration history through audit events.
- Deleting a firm must not automatically delete historical assessment-year information required for legal or audit purposes.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- Whether firms may configure different filing deadlines or workflow settings for each assessment year will be finalized with the workflow and compliance design.
- The `configuration` JSONB structure must be validated against an approved configuration schema.
- The distinction between global assessment-year status and firm-specific enablement must be enforced in backend services.
- The exact policy for processing existing ITR records after an assessment year is globally closed must be finalized.
- Assessment-year enablement and disablement must be recorded in the formal audit trail.

### 3.33 `customer_itr`

**Purpose**

The `customer_itr` table stores the primary ITR case or work record for a customer for a specific assessment year within a CA firm.

It connects the firm-customer relationship with the applicable assessment year and provides the central reference for ITR status, assignments, filing details, documents, checklists, communications and related operational records.

A customer may have multiple ITR records across different assessment years and, where the business rules permit, multiple filing records for the same assessment year.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Every ITR record must belong to one firm.
- Mandatory PostgreSQL Row-Level Security (RLS) using `firm_id`.
- The customer relationship and assessment-year configuration must belong to the same firm.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `firm_customer_id` | UUID | No | Customer relationship within the firm |
| `assessment_year_id` | UUID | No | Global assessment-year reference |
| `firm_assessment_year_id` | UUID | No | Firm-specific assessment-year configuration |
| `itr_reference_number` | VARCHAR(50) | No | Unique ITR case reference within the firm |
| `itr_type` | VARCHAR(30) | Yes | ITR form or return category |
| `taxpayer_category` | VARCHAR(30) | Yes | Applicable taxpayer category |
| `itr_status` | VARCHAR(30) | No | Current ITR workflow status |
| `priority` | VARCHAR(20) | No | Operational priority |
| `assigned_membership_id` | UUID | Yes | Primary staff membership responsible for the ITR |
| `received_at` | TIMESTAMPTZ | Yes | Date and time the ITR work was received |
| `work_started_at` | TIMESTAMPTZ | Yes | Date and time processing began |
| `completed_at` | TIMESTAMPTZ | Yes | Date and time internal processing was completed |
| `filed_at` | TIMESTAMPTZ | Yes | Date and time the return was filed |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `created_by_membership_id` | UUID | Yes | Firm membership that created the record |
| `updated_by_membership_id` | UUID | Yes | Firm membership that last updated the record |
| `deleted_at` | TIMESTAMPTZ | Yes | Controlled soft-deletion timestamp, if permitted |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`.
- `(firm_id, firm_customer_id)` references the corresponding firm-customer relationship.
- `(firm_id, assessment_year_id)` must reference an assessment year available to the firm through `firm_assessment_year`.
- `(firm_id, firm_assessment_year_id)` references the corresponding firm assessment-year configuration.
- `(firm_id, assigned_membership_id)` references the corresponding firm membership.
- `(firm_id, created_by_membership_id)` references the creating firm membership.
- `(firm_id, updated_by_membership_id)` references the updating firm membership.

Composite foreign keys or equivalent database-enforced constraints must prevent cross-firm references.

The assessment-year reference and firm-assessment-year reference must be consistent: both must identify the same assessment year.

**ITR Statuses**

The initial workflow status catalogue should include:

- `received`
- `documents_pending`
- `under_preparation`
- `under_review`
- `correction_required`
- `ready_for_filing`
- `filed`
- `completed`
- `on_hold`
- `cancelled`

These statuses represent the operational ITR workflow. The permitted transitions will be defined separately in the workflow design.

**Priority Values**

Initial supported values:

- `low`
- `normal`
- `high`
- `urgent`

Priority must not override authorization, statutory filing requirements or workflow controls.

**Constraints and Uniqueness**

- `itr_reference_number` must be unique within a firm.
- Customer and assessment-year references are mandatory.
- The assessment year must be enabled for the firm when a new ITR record is created.
- The assigned membership, when provided, must belong to the same firm.
- `completed_at` must not precede `created_at`.
- `filed_at` must not precede `created_at`.
- A filed ITR must have the required filing evidence or filing details before it is considered successfully filed.
- A cancelled or soft-deleted record must not be silently reused.
- Whether multiple active ITR cases are permitted for one customer and assessment year must follow an explicit business rule.

A unique constraint on `(firm_id, itr_reference_number)` is required.

**Indexes**

Proposed indexes:

- Unique index on `(firm_id, itr_reference_number)`.
- Index on `(firm_id, firm_customer_id, assessment_year_id)`.
- Index on `(firm_id, itr_status, created_at DESC)`.
- Index on `(firm_id, assigned_membership_id, itr_status)`.
- Index on `(firm_id, assessment_year_id, itr_status)`.
- Index on `(firm_id, priority, created_at DESC)`.
- Partial index on active ITR records where `deleted_at IS NULL`.

**Tenant Isolation and RLS**

- Every record must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Cross-firm customer, assessment-year configuration and membership references must be rejected.
- Firm-level role permissions and customer-level access grants must be enforced by backend authorization.
- Staff assignments must not independently grant permissions beyond the approved authorization model.
- Platform-level access must follow separately controlled and audited procedures.

**Business and Security Rules**

- An ITR record must be created only for a valid firm-customer relationship.
- The firm must have the corresponding assessment year enabled when creating new ITR work.
- The system must retain the assessment-year reference even after the assessment year is closed or disabled.
- ITR status changes must follow an approved workflow transition policy.
- Status updates and their corresponding history records must be committed in the same transaction.
- Assignment and reassignment must be authorized and auditable.
- A filed status must not be set merely because an employee marks the work as complete; filing evidence and applicable filing details must be verified.
- The system must distinguish internal preparation completion from actual filing.
- Sensitive taxpayer information must be protected through backend authorization, database controls and appropriate encryption.
- All important ITR operations must generate appropriate audit evidence.

**Data Integrity and Lifecycle**

- ITR records must not be cascade-deleted when a customer relationship or assessment-year configuration changes.
- Historical ITR information must remain available to authorized users.
- Soft deletion, if permitted, must follow a controlled retention and recovery policy.
- ITR status history, filing details, documents and financial records must preserve their relationships to the ITR case.
- All timestamps must use UTC.
- Corrections must preserve previous filing and operational evidence rather than silently replacing it.

**Design Notes and Open Decisions**

- Whether one customer may have multiple ITR cases for the same assessment year must be finalized. Revised returns, updated returns and other filing scenarios may require separate case or filing records.
- The relationship between `itr_type`, taxpayer category and applicable filing rules must be finalized in the tax-domain design.
- The final ITR status catalogue and transition matrix will be defined in the workflow design.
- Filing acknowledgements, acknowledgement numbers, ITR-V and related filing evidence will be specified in `itr_filing_details`.
- Whether the primary staff assignment should be maintained directly here or derived entirely from `customer_staff_assignment` and `itr_assignment` must be reviewed to avoid conflicting assignment information.
- The system must distinguish an internal ITR workflow record from an actual income-tax return submission to the government.

### 3.34 `itr_status_history`

**Purpose**

The `itr_status_history` table maintains an immutable history of status changes for each customer ITR record.

It provides traceability for the ITR preparation and filing workflow, supports operational monitoring and enables authorized reviewers to reconstruct the sequence of status transitions.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Each history record belongs to one ITR case within a firm.
- Mandatory PostgreSQL Row-Level Security (RLS) using `firm_id`.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `customer_itr_id` | UUID | No | Related ITR case |
| `previous_status` | VARCHAR(30) | Yes | Status before the transition |
| `new_status` | VARCHAR(30) | No | Status after the transition |
| `change_reason` | TEXT | Yes | Reason or explanation for the transition |
| `change_source` | VARCHAR(30) | No | Source of the status change |
| `changed_by_membership_id` | UUID | Yes | Firm membership responsible for the change |
| `correlation_id` | UUID | Yes | Identifier linking related operations |
| `occurred_at` | TIMESTAMPTZ | No | Time the transition occurred |
| `created_at` | TIMESTAMPTZ | No | Time the history record was created |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`.
- `(firm_id, customer_itr_id)` references the corresponding customer ITR record.
- `(firm_id, changed_by_membership_id)` references the responsible firm membership.

Composite foreign keys or equivalent database-enforced constraints must ensure that the ITR record and responsible membership belong to the same firm.

A null `changed_by_membership_id` is permitted for system-generated or automated transitions.

**Supported Status Values**

The `previous_status` and `new_status` fields must use the same status catalogue defined for `customer_itr`:

- `received`
- `documents_pending`
- `under_preparation`
- `under_review`
- `correction_required`
- `ready_for_filing`
- `filed`
- `completed`
- `on_hold`
- `cancelled`

The status catalogue must be maintained consistently across the current ITR record and its history.

**Change Sources**

Initial supported values:

- `firm_admin`
- `firm_staff`
- `system`
- `automated_process`
- `platform_admin`

The application must validate that the source and actor are consistent. Platform administrators must not be represented as firm memberships.

**Constraints and Data Integrity**

- `customer_itr_id` and `new_status` are mandatory.
- `previous_status` may be null only for the initial status establishment.
- `previous_status` and `new_status` must not be identical.
- A user-initiated transition must identify the responsible membership.
- A system-generated transition must be distinguishable from a user-initiated transition.
- Status values must belong to the approved ITR status catalogue.
- The latest applicable history record must agree with the current `customer_itr.itr_status`.

**Indexes**

Proposed indexes:

- Index on `(firm_id, customer_itr_id, occurred_at DESC)`.
- Index on `(firm_id, new_status, occurred_at DESC)`.
- Index on `(firm_id, changed_by_membership_id, occurred_at DESC)`.
- Index on `correlation_id`.

**Tenant Isolation and RLS**

- Every record must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Cross-firm ITR and membership references must be rejected.
- Only authorized firm users may retrieve ITR status history.
- Platform-level access must follow separately controlled and audited procedures.

**Business and Security Rules**

- Every successful ITR status transition must create a corresponding history record.
- Updating `customer_itr.itr_status` and inserting its history record must occur in the same database transaction.
- The backend must validate that the requested transition is permitted by the workflow transition policy.
- Users must not be able to bypass workflow authorization by directly modifying the current status.
- Failed transitions must not create records indicating that the transition succeeded.
- Status changes must be auditable and traceable to a user or system process.
- Important transitions, including filing, cancellation and completion, may require additional authorization or supporting evidence.
- The application must preserve the reason for important corrections, rejections and workflow exceptions.

**Data Integrity and Lifecycle**

- History records must be append-only.
- Ordinary application runtime permissions must not permit users to update or delete history records.
- Corrections must be represented by additional controlled events rather than overwriting previous history.
- ITR deletion or archival must not silently cascade-delete its status history.
- All timestamps must use UTC.
- Retention and archival must follow applicable business, legal and audit requirements.

**Design Notes and Open Decisions**

- The exact permitted status-transition matrix will be finalized in the workflow design.
- The treatment of initial status establishment must be consistent across ITR creation and history insertion.
- Whether automated status changes require a dedicated system actor reference will be finalized in the audit design.
- Whether status transitions should include additional structured metadata will be evaluated without duplicating sensitive customer information.
- Filing-related transitions must be consistent with the filing evidence stored in `itr_filing_details`.
- The system must distinguish ITR workflow history from the broader security and administrative audit trail maintained in `audit_event`.

### 3.35 `itr_assignment`

**Purpose**

The `itr_assignment` table records staff assignments for individual customer ITR cases within a CA firm.

It supports assignment, reassignment, review responsibility, workload management and accountability for ITR preparation and filing activities.

A customer-level staff assignment does not automatically imply responsibility for every ITR case belonging to that customer.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Each assignment belongs to one ITR case within one firm.
- Mandatory PostgreSQL Row-Level Security (RLS) using `firm_id`.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `customer_itr_id` | UUID | No | Related ITR case |
| `membership_id` | UUID | No | Staff membership assigned to the ITR |
| `assignment_role` | VARCHAR(30) | No | Responsibility assigned |
| `assignment_status` | VARCHAR(30) | No | Assignment lifecycle status |
| `assigned_at` | TIMESTAMPTZ | No | Assignment timestamp |
| `assigned_by_membership_id` | UUID | Yes | Membership that created the assignment |
| `ended_at` | TIMESTAMPTZ | Yes | Assignment end timestamp |
| `ended_by_membership_id` | UUID | Yes | Membership that ended the assignment |
| `end_reason` | TEXT | Yes | Reason for ending the assignment |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`.
- `(firm_id, customer_itr_id)` references the corresponding customer ITR record.
- `(firm_id, membership_id)` references the corresponding firm membership.
- `(firm_id, assigned_by_membership_id)` references the assigning firm membership.
- `(firm_id, ended_by_membership_id)` references the membership that ended the assignment.

Composite foreign keys or equivalent database-enforced constraints must ensure that all referenced ITR records and memberships belong to the same firm.

**Assignment Roles**

Initial supported values:

- `preparer`: Responsible for preparing the ITR.
- `reviewer`: Responsible for reviewing the prepared ITR.
- `approver`: Responsible for authorized internal approval.
- `coordinator`: Responsible for coordinating ITR-related activities.

Additional roles may be introduced through controlled application and database changes.

**Assignment Statuses**

Initial supported values:

- `active`
- `ended`
- `cancelled`

An active assignment must not have an `ended_at` value. An ended or cancelled assignment must have an appropriate end timestamp.

**Constraints and Uniqueness**

- An assigned membership must belong to the same firm as the ITR case.
- Only eligible, active firm memberships may receive new assignments.
- Duplicate active assignments for the same ITR case, membership and assignment role must be prevented.
- An ITR case may have multiple active assignments with different roles.
- Historical assignments must be retained.
- Whether multiple preparers, reviewers or approvers may be active simultaneously must follow an explicit firm-level policy.

A partial unique index should prevent duplicate active assignments for the same `(customer_itr_id, membership_id, assignment_role)` combination.

**Indexes**

Proposed indexes:

- Index on `(firm_id, customer_itr_id, assignment_status)`.
- Index on `(firm_id, membership_id, assignment_status)`.
- Index on `(firm_id, assignment_role, assignment_status)`.
- Index on `(firm_id, assigned_at DESC)`.
- Index on `(firm_id, ended_at)` where useful.

**Tenant Isolation and RLS**

- Every record must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Cross-firm ITR and membership references must be rejected.
- Staff assignments must not independently grant permissions beyond the approved authorization model.
- Access to an ITR case must still satisfy role permissions and customer-level access requirements.
- Platform-level access must follow separately controlled and audited procedures.

**Business and Security Rules**

- Assignment and reassignment must be performed through authorized backend services.
- Reassignment should end the previous assignment and create a new assignment record within a controlled transaction.
- Assignment changes must be auditable.
- Ending a staff membership must trigger a review or controlled termination of its active ITR assignments.
- Assignment changes must invalidate relevant authorization caches.
- Staff must not be able to assign ITR cases to themselves or other staff unless explicitly authorized.
- Internal approval must not be confused with actual income-tax return filing.
- An approver must satisfy the firm's applicable segregation-of-duties policy.

**Data Integrity and Lifecycle**

- Assignment history must be preserved rather than overwritten.
- Assignment creation, reassignment and termination must maintain consistent timestamps.
- Deleting a membership or ITR record must not silently cascade-delete assignment history.
- All timestamps must use UTC.
- Historical assignment records must remain available to authorized reviewers.

**Design Notes and Open Decisions**

- Whether the same membership may act as preparer and reviewer for one ITR case must be defined by the segregation-of-duties policy.
- Whether an ITR case requires at least one preparer and one reviewer before filing readiness can be achieved will be finalized in the workflow design.
- Whether multiple simultaneous preparers or reviewers are supported must be decided.
- The relationship between `customer_staff_assignment`, `customer_itr.assigned_membership_id` and this table must be reviewed to eliminate redundant assignment information.
- Assignment changes must be represented in the formal audit trail.

### 3.36 `itr_filing_details`

**Purpose**

The `itr_filing_details` table stores the actual income-tax return filing information associated with a customer ITR case.

It records filing references, acknowledgement details, filing dates, submission information and relevant filing evidence.

The table distinguishes actual return filing from internal ITR preparation, review and approval activities.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Each filing-details record belongs to one ITR case within one firm.
- Mandatory PostgreSQL Row-Level Security (RLS) using `firm_id`.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `customer_itr_id` | UUID | No | Related ITR case |
| `filing_type` | VARCHAR(30) | No | Type of return filing |
| `filing_status` | VARCHAR(30) | No | Filing lifecycle status |
| `acknowledgement_number` | VARCHAR(100) | Yes | Income-tax return acknowledgement number |
| `filing_date` | DATE | Yes | Actual return filing date |
| `filing_timestamp` | TIMESTAMPTZ | Yes | Actual filing timestamp, when available |
| `itr_form_type` | VARCHAR(30) | Yes | ITR form submitted |
| `submission_mode` | VARCHAR(30) | Yes | Filing submission method |
| `verification_status` | VARCHAR(30) | No | Return verification status |
| `verification_date` | DATE | Yes | Date of verification |
| `verification_reference` | VARCHAR(100) | Yes | Verification reference, where applicable |
| `filing_portal_reference` | VARCHAR(150) | Yes | External portal reference, where available |
| `filing_evidence_document_id` | UUID | Yes | Related filing evidence document |
| `filing_notes` | TEXT | Yes | Relevant filing notes |
| `recorded_by_membership_id` | UUID | Yes | Membership that recorded the filing details |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`.
- `(firm_id, customer_itr_id)` references the corresponding customer ITR record.
- `(firm_id, recorded_by_membership_id)` references the responsible firm membership.
- `(firm_id, filing_evidence_document_id)` must reference a document belonging to the same firm.

Composite foreign keys or equivalent database-enforced constraints must prevent cross-firm references.

**Filing Types**

Initial supported values:

- `original`
- `revised`
- `belated`
- `updated`
- `other`

The supported filing types must be reviewed against applicable Indian income-tax filing rules before production deployment.

**Filing Statuses**

Initial supported values:

- `pending`
- `submitted`
- `acknowledged`
- `rejected`
- `cancelled`

The application must distinguish an attempted submission from a successfully acknowledged filing.

**Verification Statuses**

Initial supported values:

- `not_started`
- `pending`
- `verified`
- `failed`
- `not_applicable`

Verification status must represent the applicable return verification process and must not be inferred merely from submission.

**Submission Modes**

Initial supported values:

- `manual`
- `authorized_integration`
- `other`

The system must not claim direct government portal integration unless an authorized integration has actually been implemented.

**Constraints and Data Integrity**

- `customer_itr_id` and `filing_type` are mandatory.
- Filing dates must be valid and consistent with the related filing event.
- `verification_date` must not precede the filing date.
- An acknowledged filing must have the required acknowledgement reference and filing date.
- A verified filing must have the required verification evidence or reference, where applicable.
- Filing reference uniqueness must be scoped according to the applicable government reference rules and firm ownership.
- Filing evidence must belong to the same firm as the ITR case.
- A filing attempt must not be represented as successful solely because a staff member manually changes its status.

**Indexes**

Proposed indexes:

- Index on `(firm_id, customer_itr_id)`.
- Index on `(firm_id, acknowledgement_number)` where the value is present.
- Index on `(firm_id, filing_date)`.
- Index on `(firm_id, filing_status, filing_date)`.
- Index on `(firm_id, verification_status)`.

**Tenant Isolation and RLS**

- Every record must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Cross-firm ITR, document and membership references must be rejected.
- Only authorized users may create, view or modify filing details.
- Customer portal access must follow customer relationship and record-level authorization rules.
- Platform-level access must follow separately controlled and audited procedures.

**Business and Security Rules**

- Filing information must be recorded through authorized backend services.
- Actual filing must be distinguished from internal workflow completion.
- Acknowledgement information must be recorded only when supported by reliable filing evidence.
- The application must preserve evidence of the filing event.
- Filing corrections and subsequent revised or updated returns must not silently overwrite earlier filing evidence.
- Filing information changes must be auditable.
- Sensitive filing references must be protected from unauthorized disclosure.
- Filing evidence documents must be stored in private object storage and accessed through authorized, short-lived access mechanisms.
- The system must not automatically claim successful government submission without verified integration or reliable supporting evidence.

**Data Integrity and Lifecycle**

- Filing records must preserve historical filing events.
- Previous filing evidence must remain available to authorized reviewers.
- Revised, belated and updated filing events must maintain their relationship to the appropriate ITR case.
- Document deletion or archival must not silently destroy required filing evidence.
- All timestamps must use UTC.
- Retention must follow applicable legal, business and audit requirements.

**Design Notes and Open Decisions**

- Whether one ITR case can have multiple filing attempts or filing records must be finalized. The design should support original and subsequent filings without overwriting earlier evidence.
- Whether filing attempts should eventually be stored in a separate `itr_filing_attempt` table must be assessed during detailed schema review.
- The exact acknowledgement and verification requirements must be validated against applicable Indian income-tax procedures.
- The relationship between filing status and `customer_itr.itr_status` must be defined in the workflow design.
- The final evidence-document relationship must be checked against the document model.
- Government portal integration remains outside the initial scope unless separately approved and implemented.

### 3.37 `itr_note`

**Purpose**

The `itr_note` table stores internal notes associated with a customer ITR case within a CA firm.

It supports communication and collaboration among authorized firm administrators, preparers, reviewers and other staff members involved in ITR processing.

Notes are internal operational records and must not automatically become visible to customers.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Each note belongs to one ITR case within one firm.
- Mandatory PostgreSQL Row-Level Security (RLS) using `firm_id`.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `customer_itr_id` | UUID | No | Related ITR case |
| `note_type` | VARCHAR(30) | No | Category of note |
| `note_content` | TEXT | No | Note content |
| `visibility` | VARCHAR(20) | No | Intended visibility |
| `is_pinned` | BOOLEAN | No | Indicates whether the note is pinned |
| `created_by_membership_id` | UUID | No | Firm membership that created the note |
| `updated_by_membership_id` | UUID | Yes | Firm membership that last updated the note |
| `created_at` | TIMESTAMPTZ | No | Note creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `deleted_at` | TIMESTAMPTZ | Yes | Controlled soft-deletion timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`.
- `(firm_id, customer_itr_id)` references the corresponding customer ITR record.
- `(firm_id, created_by_membership_id)` references the creating firm membership.
- `(firm_id, updated_by_membership_id)` references the updating firm membership.

Composite foreign keys or equivalent database-enforced constraints must ensure that the referenced ITR case and memberships belong to the same firm.

**Note Types**

Initial supported values:

- `general`: General operational observations.
- `preparation`: Notes related to ITR preparation.
- `review`: Internal review observations.
- `correction`: Information about required corrections.
- `follow_up`: Notes about pending follow-up activities.
- `compliance`: Relevant compliance observations.

Additional categories may be introduced through controlled application changes.

**Visibility Values**

Initial supported values:

- `internal`: Visible only to appropriately authorized firm users.
- `restricted`: Visible only to explicitly authorized users with the required access.

Customer-visible communication must use the dedicated communication module rather than this table.

**Constraints and Data Integrity**

- `customer_itr_id` is mandatory.
- `note_content` must not be empty or contain only whitespace.
- `note_type` must belong to the approved catalogue.
- `visibility` must belong to the approved visibility catalogue.
- `created_by_membership_id` must identify the original author.
- Updates must preserve the original creator and creation timestamp.
- Deleted notes must not appear in ordinary active-note queries.
- The system must prevent unauthorized modification of notes created by other users.

**Indexes**

Proposed indexes:

- Index on `(firm_id, customer_itr_id, created_at DESC)`.
- Index on `(firm_id, customer_itr_id, note_type)`.
- Index on `(firm_id, created_by_membership_id, created_at DESC)`.
- Partial index on `(firm_id, customer_itr_id, is_pinned)` for active pinned notes.

**Tenant Isolation and RLS**

- Every note must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Cross-firm ITR and membership references must be rejected.
- Backend authorization must verify the user's access to the associated customer and ITR case.
- Restricted notes must require additional authorization beyond ordinary ITR access.
- Platform-level access must follow separately controlled and audited procedures.

**Business and Security Rules**

- Notes must be created and accessed through authorized backend services.
- Internal notes must never be exposed through customer APIs or customer portal queries.
- Restricted notes must be accessible only to explicitly authorized personnel.
- Notes may contain sensitive taxpayer or business information and must be protected accordingly.
- Note creation, modification, deletion and access to restricted notes must be auditable.
- The original author and creation timestamp must be preserved.
- Note content must be validated and safely rendered to prevent stored cross-site scripting (XSS).
- Notes must not be used to store passwords, OTPs, session tokens or other authentication secrets.
- Customer communications and formal filing evidence must be maintained in their dedicated modules.

**Data Integrity and Lifecycle**

- Note records should use controlled soft deletion where retention requirements permit.
- Ordinary deletion must not silently remove legally or operationally required evidence.
- Changes to note content must be traceable through the audit mechanism.
- All timestamps must use UTC.
- Retention and archival must follow the applicable business and legal retention policy.

**Design Notes and Open Decisions**

- Whether notes should support threaded discussions or comments will be evaluated separately.
- Whether note content should be versioned in a dedicated history table will be finalized during schema review.
- Whether attachments should be supported will be handled through the document and attachment model rather than storing files directly in this table.
- The exact permission requirements for creating, editing, pinning and deleting restricted notes will be defined in the authorization design.
- The system must clearly distinguish internal ITR notes from customer-facing messages and formal audit events.

### 3.38 `document`

**Purpose**

The `document` table maintains the central metadata and ownership information for documents uploaded to or generated within the ITR Management System.

It stores document metadata and references to files held in private cloud object storage. Actual file contents must never be stored directly in this database table.

Documents may be associated with customers, ITR cases, filing records and other supported business entities.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned for firm-managed business documents.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Every firm-managed document must belong to one CA firm.
- Mandatory PostgreSQL Row-Level Security (RLS) using `firm_id`.
- Central customer identity documents, if supported, must follow a separately defined ownership and authorization model.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `firm_customer_id` | UUID | Yes | Associated firm-customer relationship |
| `customer_itr_id` | UUID | Yes | Associated ITR case |
| `document_category` | VARCHAR(50) | No | Document category |
| `document_type` | VARCHAR(50) | No | Specific document type |
| `document_name` | VARCHAR(255) | No | User-visible document name |
| `original_file_name` | VARCHAR(255) | Yes | Original uploaded filename |
| `storage_provider` | VARCHAR(30) | No | Private object storage provider |
| `storage_bucket` | VARCHAR(255) | No | Storage bucket or container reference |
| `storage_object_key` | VARCHAR(1024) | No | Private object storage key |
| `mime_type` | VARCHAR(150) | No | Detected media type |
| `file_extension` | VARCHAR(20) | Yes | File extension |
| `file_size_bytes` | BIGINT | No | File size in bytes |
| `file_checksum` | VARCHAR(255) | No | Cryptographic file integrity checksum |
| `document_status` | VARCHAR(30) | No | Document lifecycle status |
| `access_classification` | VARCHAR(30) | No | Document sensitivity classification |
| `uploaded_by_membership_id` | UUID | Yes | Firm membership that uploaded the document |
| `uploaded_by_customer` | BOOLEAN | No | Indicates whether the customer uploaded it |
| `uploaded_at` | TIMESTAMPTZ | No | Upload timestamp |
| `last_accessed_at` | TIMESTAMPTZ | Yes | Last recorded access timestamp |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `deleted_at` | TIMESTAMPTZ | Yes | Controlled soft-deletion timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`.
- `(firm_id, firm_customer_id)` references the corresponding firm-customer relationship.
- `(firm_id, customer_itr_id)` references the corresponding customer ITR record.
- `(firm_id, uploaded_by_membership_id)` references the corresponding firm membership.

Composite foreign keys or equivalent database-enforced constraints must prevent cross-firm references.

Where both customer and ITR references are provided, the ITR must belong to the same customer relationship.

**Document Categories**

Initial supported categories:

- `identity`
- `income`
- `deduction`
- `banking`
- `tax_compliance`
- `filing_evidence`
- `financial`
- `other`

**Document Types**

Examples of supported document types:

- PAN card
- Aadhaar document
- Form 16
- Salary slip
- Bank statement
- Interest certificate
- Investment proof
- Insurance premium receipt
- Home loan interest certificate
- Capital gains statement
- AIS or tax information statement
- ITR acknowledgement
- ITR-V
- Other supporting documents

The document-type catalogue must be configurable or maintained through controlled application configuration.

**Document Statuses**

Initial supported values:

- `upload_pending`
- `quarantined`
- `scanning`
- `available`
- `rejected`
- `archived`
- `deleted`

A document must not become available for ordinary access until the required security validation and malware scanning have succeeded.

**Access Classification**

Initial supported values:

- `standard`
- `confidential`
- `highly_sensitive`

Access classification must influence authorization and handling requirements, but must not replace role-based and record-level authorization.

**Constraints and Data Integrity**

- `firm_id` is mandatory.
- Document names must not be empty.
- File size must be greater than zero and within the configured upload limit.
- Storage object keys must be generated by the application and must not be based directly on untrusted filenames.
- Storage object keys must be unique within their storage bucket or container.
- File checksums must be generated using an approved cryptographic hash algorithm.
- MIME types must be verified using file-content inspection rather than trusting the supplied filename or HTTP content type.
- A document must not be marked `available` until required validation and malware scanning have succeeded.
- At least one supported business association must be provided, subject to the approved document ownership model.

**Indexes**

Proposed indexes:

- Index on `(firm_id, firm_customer_id, document_status)`.
- Index on `(firm_id, customer_itr_id, document_status)`.
- Index on `(firm_id, document_category, document_type)`.
- Index on `(firm_id, uploaded_at DESC)`.
- Index on `(firm_id, uploaded_by_membership_id, uploaded_at DESC)`.
- Unique index on `(storage_bucket, storage_object_key)`.

**Tenant Isolation and RLS**

- Every firm-managed document must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Cross-firm customer, ITR and membership references must be rejected.
- Document access must also satisfy backend role permissions and customer-level authorization.
- A valid document identifier must never be sufficient by itself to retrieve a file.
- Platform-level support access must follow separately controlled and audited procedures.

**Business and Security Rules**

- Actual files must be stored in private cloud object storage.
- Public bucket access must be disabled.
- Database records must contain storage references, not file contents.
- File downloads must be authorized through backend services.
- Short-lived signed URLs or equivalent controlled download mechanisms may be used after authorization.
- Uploaded files must undergo file type validation, size validation and malware scanning.
- Unverified or suspicious files must remain quarantined.
- File names must be sanitized before display or processing.
- Storage object keys must be unpredictable and must not expose customer PAN, Aadhaar or other sensitive identifiers.
- Sensitive documents must use encryption at rest and protected transport.
- Access to highly sensitive documents must require additional authorization and generate audit evidence.
- Document downloads, previews, replacement and deletion must be auditable.
- The system must not expose storage credentials or unrestricted object storage URLs to customers or staff.
- File access must be restricted to authorized users of the associated firm and customer records.

**Data Integrity and Lifecycle**

- Document metadata and file storage operations must be coordinated to avoid inconsistent records.
- Failed uploads must not leave documents marked as available.
- Document deletion must follow a controlled process that considers retention requirements and legal holds.
- Historical filing evidence must not be silently removed.
- Documents referenced by ITR filing records must be protected against unauthorized deletion.
- All timestamps must use UTC.
- Retention and archival must follow applicable legal, business and audit requirements.

**Design Notes and Open Decisions**

- Whether customer identity documents should be centrally owned or always firm-specific must be finalized against the existing customer identity and tenant-isolation model.
- The relationship between `document`, `document_version`, `document_review` and `document_scan_result` will be defined in the subsequent document-management tables.
- The initial cloud object storage provider remains to be selected.
- The maximum file size, supported file types and retention periods must be configurable.
- The document replacement process must preserve previous versions and audit evidence.
- The system must define how documents associated with multiple business records are represented without weakening tenant isolation.
- Encryption key management, malware scanning infrastructure and quarantine lifecycle must be finalized before implementation.

### 3.39 `document_version`

**Purpose**

The `document_version` table maintains the version history of documents stored in the ITR Management System.

Whenever a document is replaced or a new version is uploaded, the system must create a new version record rather than overwrite the existing file.

This preserves historical evidence and supports document review, auditability and recovery.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Every version must belong to a document owned by the same firm.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `document_id` | UUID | No | Parent document record |
| `version_number` | INTEGER | No | Sequential version number |
| `storage_provider` | VARCHAR(30) | No | Object storage provider |
| `storage_bucket` | VARCHAR(255) | No | Storage bucket or container |
| `storage_object_key` | VARCHAR(1024) | No | Private object storage key |
| `mime_type` | VARCHAR(150) | No | Detected media type |
| `file_extension` | VARCHAR(20) | Yes | File extension |
| `file_size_bytes` | BIGINT | No | File size in bytes |
| `file_checksum` | VARCHAR(255) | No | Cryptographic integrity checksum |
| `version_status` | VARCHAR(30) | No | Version lifecycle status |
| `change_reason` | TEXT | Yes | Reason for uploading a new version |
| `uploaded_by_membership_id` | UUID | Yes | Firm membership responsible for upload |
| `uploaded_by_customer` | BOOLEAN | No | Indicates customer upload |
| `uploaded_at` | TIMESTAMPTZ | No | Upload timestamp |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, document_id)` references `(firm_id, id)` in `document`.
- `(firm_id, uploaded_by_membership_id)` references the corresponding firm membership.

Composite foreign keys must prevent a document version from referencing a document or uploader belonging to another firm.

**Version Number Rules**

- Version numbering starts at `1`.
- Version numbers increase sequentially for each document.
- A document must not have duplicate version numbers.
- Version creation must be performed transactionally with the parent document's current-version update.
- Concurrent uploads must not create duplicate or conflicting version numbers.
- Version numbering must not be reused after a version is archived or deleted.

**Version Statuses**

Initial supported values:

- `upload_pending`
- `quarantined`
- `scanning`
- `available`
- `rejected`
- `archived`

A version must not become available until the required validation and malware scanning have succeeded.

**Constraints and Data Integrity**

- `version_number` must be greater than zero.
- `file_size_bytes` must be greater than zero.
- File checksum must be generated by the application using an approved cryptographic hash algorithm.
- Storage object keys must be unique within their storage bucket or container.
- A document must have at most one current available version.
- The parent document's current version reference must point to a version belonging to that same document and firm.
- A version must not be physically overwritten by another version.
- Historical versions must retain their original upload and checksum information.

**Indexes**

Proposed indexes:

- Unique index on `(firm_id, document_id, version_number)`.
- Unique index on `(storage_bucket, storage_object_key)`.
- Index on `(firm_id, document_id, uploaded_at DESC)`.
- Index on `(firm_id, version_status)`.
- Index on `(firm_id, uploaded_by_membership_id, uploaded_at DESC)`.

**Tenant Isolation and RLS**

- Every version must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Composite foreign keys must enforce same-firm document and membership relationships.
- Access to a version must also satisfy the authorization rules of its parent document.
- A version identifier alone must never grant access to the underlying file.

**Business and Security Rules**

- Each new upload or replacement creates a separate version record.
- Previous versions must remain available to authorized reviewers according to retention policy.
- Only the latest approved and available version should normally be presented as the current document.
- A rejected or quarantined version must not replace the currently available version.
- Version downloads and previews must be authorized through the backend.
- Historical versions must use private object storage.
- Each version must have its own storage object key.
- Replacing a document must not remove previous audit evidence.
- File validation and malware scanning must be performed for every version.
- Version creation, approval, rejection, archival and access must be auditable.

**Data Integrity and Lifecycle**

- Version creation and parent document updates must be coordinated transactionally.
- Failed uploads must not create a falsely available version.
- A failed replacement must leave the previously available version unchanged.
- Historical versions must be protected against unauthorized modification or deletion.
- Retention and legal hold requirements must apply to relevant historical versions.
- Physical deletion from object storage must follow an approved retention and deletion process.

**Design Notes and Open Decisions**

- The `document` table must eventually identify its current version through a controlled reference such as `current_version_id`.
- The exact approval process for making a new version current must be finalized.
- The system must define whether customers can view historical versions or only the current version.
- Version retention periods and archival rules must be finalized.
- The handling of concurrent uploads and version-number allocation must be implemented safely.
- Whether document metadata changes create a new file version or only an audit event must be defined.

### 3.40 `document_review`

**Purpose**

The `document_review` table records staff reviews of documents submitted or maintained within the ITR Management System.

It maintains the review outcome, reviewer identity, comments, timestamps and references to the document version that was reviewed.

A review decision must apply to a specific document version so that approval of an older version cannot automatically approve a subsequently uploaded version.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Every review must belong to a document and version within the same firm.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `document_id` | UUID | No | Document being reviewed |
| `document_version_id` | UUID | No | Specific document version reviewed |
| `reviewed_by_membership_id` | UUID | No | Firm membership performing the review |
| `review_status` | VARCHAR(30) | No | Review outcome |
| `review_comments` | TEXT | Yes | Review comments |
| `rejection_reason` | TEXT | Yes | Reason for rejection |
| `requested_correction` | TEXT | Yes | Required correction or additional information |
| `reviewed_at` | TIMESTAMPTZ | No | Review completion timestamp |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, document_id)` references the corresponding document.
- `(firm_id, document_version_id)` references the corresponding document version.
- `(firm_id, reviewed_by_membership_id)` references the corresponding firm membership.

Composite foreign keys must enforce same-firm relationships.

The database and application must also ensure that `document_version_id` belongs to the specified `document_id`, not merely to the same firm.

**Review Statuses**

Initial supported values:

- `pending`
- `approved`
- `rejected`
- `correction_required`
- `not_applicable`

Review status must represent the review decision, not the malware scanning or upload status of the document.

**Constraints and Data Integrity**

- Reviewer membership is mandatory.
- Review status must be a supported value.
- An approved review must reference a version that has passed the required security validation.
- A rejected review must include a rejection reason.
- A correction-required review must include a correction request.
- `reviewed_at` must be recorded when a review is completed.
- A review must not be silently reassigned to another document version.
- Review records must preserve the identity of the reviewer and the reviewed version.

**Indexes**

Proposed indexes:

- Index on `(firm_id, document_id, reviewed_at DESC)`.
- Index on `(firm_id, document_version_id, reviewed_at DESC)`.
- Index on `(firm_id, reviewed_by_membership_id, reviewed_at DESC)`.
- Index on `(firm_id, review_status, reviewed_at DESC)`.

**Tenant Isolation and RLS**

- Every review must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Composite foreign keys must prevent cross-firm document, version and reviewer references.
- Review access must also satisfy the permissions associated with the document and its customer relationship.
- Customers must not be allowed to access internal staff review comments unless an explicitly authorized customer-facing communication mechanism is provided.

**Business and Security Rules**

- Only authorized firm staff may review documents.
- Reviewers must be authorized for the relevant customer or document.
- A reviewer must be able to approve, reject or request corrections according to their permissions.
- Approval must apply only to the exact version reviewed.
- Uploading a newer version must not inherit approval from an earlier version.
- Review decisions must be auditable.
- Review comments must not contain unnecessary passwords, authentication codes or other secrets.
- Rejection and correction messages intended for customers must be handled through an approved customer-facing communication workflow.
- Internal review comments must remain private to authorized firm users.

**Data Integrity and Lifecycle**

- Review creation and review outcome recording must be reliable and transactional.
- Completed review decisions must not be silently overwritten.
- If a decision needs correction, the system must preserve the original decision and record the correction through an auditable process.
- Historical review records must be retained according to the applicable retention policy.
- Review history must remain available even when a document is replaced or archived.

**Design Notes and Open Decisions**

- Whether multiple reviewers may review the same version must be finalized.
- Whether a second-level approval is required for highly sensitive documents must be defined.
- The relationship between document approval and ITR workflow progression must be finalized.
- The system must define whether reviewers can revise their own decisions or must create a new review record.
- Customer-facing correction requests must be separated from internal review notes.
- The precise permission catalogue for document review actions must be defined.

### 3.41 `document_access_log`

**Purpose**

The `document_access_log` table records access to firm-managed documents and their versions.

It provides traceable evidence of who accessed a document, which operation was performed, when it occurred and whether the operation succeeded or failed.

The table supports security monitoring, privacy investigations, compliance reviews and document access auditing.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Records must be append-only.
- Access logs must not be editable or deletable by ordinary firm users.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `document_id` | UUID | No | Accessed document |
| `document_version_id` | UUID | Yes | Accessed document version, when applicable |
| `actor_user_id` | UUID | Yes | Central user identity performing the operation |
| `actor_membership_id` | UUID | Yes | Firm membership context used for access |
| `actor_customer_id` | UUID | Yes | Customer identity, when the customer performs the operation |
| `access_action` | VARCHAR(50) | No | Operation performed |
| `access_result` | VARCHAR(30) | No | Result of the access attempt |
| `denial_reason` | VARCHAR(255) | Yes | Reason for denied access |
| `ip_address` | INET | Yes | Request source IP address |
| `user_agent` | TEXT | Yes | Client user-agent information |
| `correlation_id` | UUID | Yes | Request or operation correlation identifier |
| `occurred_at` | TIMESTAMPTZ | No | Time of the access event |
| `created_at` | TIMESTAMPTZ | No | Log record creation timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, document_id)` references the corresponding document.
- `(firm_id, document_version_id)` references the corresponding document version.
- `(firm_id, actor_membership_id)` references the corresponding firm membership.
- `actor_user_id` references `user.id`.
- `actor_customer_id` references `customer.id`.

Composite foreign keys must enforce same-firm relationships for document, version and membership references.

The actor fields must follow an explicitly defined actor model. A customer action and a staff membership action must not be ambiguously attributed.

System-originated operations must also be identifiable through a controlled actor representation.

**Access Actions**

Initial supported values:

- `view`
- `preview`
- `download`
- `upload`
- `replace`
- `delete`
- `archive`
- `restore`
- `share`
- `access_denied`

The action catalogue may be extended through controlled application changes.

**Access Results**

Initial supported values:

- `success`
- `denied`
- `failure`

**Constraints and Data Integrity**

- `firm_id` is mandatory.
- `document_id` is mandatory.
- Access action and result must be supported values.
- A version reference, when supplied, must belong to the specified document and firm.
- Actor attribution must be unambiguous.
- Denied operations should capture an appropriate denial reason without exposing sensitive authorization details.
- The event timestamp must represent the actual operation time.
- Log records must not contain document contents, passwords, OTPs, session tokens or encryption keys.

**Indexes**

Proposed indexes:

- Index on `(firm_id, document_id, occurred_at DESC)`.
- Index on `(firm_id, actor_membership_id, occurred_at DESC)`.
- Index on `(firm_id, actor_user_id, occurred_at DESC)`.
- Index on `(firm_id, access_action, occurred_at DESC)`.
- Index on `(firm_id, access_result, occurred_at DESC)`.
- Index on `correlation_id`.
- Index on `occurred_at` to support retention and archival operations.

**Tenant Isolation and RLS**

- Every record must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Cross-firm document, version and membership references must be rejected.
- Firm users must only retrieve logs they are authorized to review.
- Platform-level security investigation access must use separately controlled and audited procedures.
- The application runtime must not have permission to update or delete access-log records.

**Business and Security Rules**

- Successful document views, previews and downloads must generate access records.
- Denied access attempts must be recorded where the request can be reliably attributed.
- Upload, replacement, archival, restoration and deletion operations must be recorded.
- Staff and customer access must be distinguishable.
- Access logging must occur through trusted backend services, not through client-submitted claims.
- Access-log generation must not expose document contents or sensitive identifiers.
- Logs must not be returned through ordinary customer-facing document APIs.
- Sensitive access events must be available to authorized security and audit reviewers.
- Logging failures must be monitored and must not silently undermine required audit evidence.

**Data Integrity and Lifecycle**

- Access logs must be append-only.
- Ordinary application roles must not be able to modify or delete historical records.
- Corrections must be represented by additional events rather than overwriting existing records.
- Access events should be written reliably, with a defined strategy for handling logging failures.
- Retention and archival must follow approved security, privacy and legal requirements.
- High-volume access logs may require time-based partitioning as the system grows.
- IP addresses and user-agent information must be protected and retained only as necessary.

**Design Notes and Open Decisions**

- The final actor model must distinguish platform users, firm members, customers, system processes and integration services.
- Whether failed access attempts with no confirmed firm context are stored in a separate platform security log must be finalized.
- The exact list of events requiring mandatory logging must be defined.
- Log retention, partitioning and archival policies must be established.
- Tamper-resistant external audit storage may be required for stronger evidence preservation.
- The relationship between this table and the general `audit_event` table must be defined to avoid duplicate or incomplete audit records.
- High-volume logging must be designed so that audit reliability does not unnecessarily degrade document access performance.

### 3.42 `document_scan_result`

**Purpose**

The `document_scan_result` table records the results of security scanning and file validation performed on uploaded documents and their versions.

It provides evidence of malware detection, scanning status, validation outcomes, scanner information and the time at which each scan was performed.

Every uploaded file version must undergo the required security checks before becoming available for ordinary access.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Every scan result must reference a document and its corresponding version within the same firm.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `document_id` | UUID | No | Associated document |
| `document_version_id` | UUID | No | Specific version scanned |
| `scan_type` | VARCHAR(50) | No | Type of security scan |
| `scan_status` | VARCHAR(30) | No | Scan execution status |
| `scan_result` | VARCHAR(30) | No | Security assessment outcome |
| `scanner_name` | VARCHAR(150) | No | Scanner or validation service name |
| `scanner_version` | VARCHAR(100) | Yes | Scanner software version |
| `signature_database_version` | VARCHAR(100) | Yes | Malware signature database version |
| `file_checksum` | VARCHAR(255) | No | Checksum of the scanned file |
| `threat_name` | VARCHAR(255) | Yes | Detected threat name, when applicable |
| `scan_details` | JSONB | No | Structured scan information |
| `error_details` | TEXT | Yes | Technical error information |
| `started_at` | TIMESTAMPTZ | Yes | Scan start timestamp |
| `completed_at` | TIMESTAMPTZ | Yes | Scan completion timestamp |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, document_id)` references the corresponding document.
- `(firm_id, document_version_id)` references the corresponding document version.
- Composite foreign keys must prevent cross-firm references.
- The database and application must ensure that the specified document version belongs to the specified document.

**Scan Types**

Initial supported values:

- `malware`
- `file_type_validation`
- `file_integrity`
- `content_security`
- `other`

**Scan Statuses**

Initial supported values:

- `pending`
- `queued`
- `running`
- `completed`
- `failed`
- `cancelled`
- `timeout`

**Scan Results**

Initial supported values:

- `clean`
- `threat_detected`
- `suspicious`
- `invalid_file`
- `inconclusive`
- `error`

**Constraints and Data Integrity**

- Document and document version references are mandatory.
- Scan type, status and result must use supported values.
- `scan_details` must contain a valid JSON object.
- The checksum must correspond to the exact file version scanned.
- A completed scan must have a completion timestamp.
- A completion timestamp must not precede the start timestamp.
- A failed or timed-out scan must not be treated as a successful security validation.
- A threat detection must preserve the scanner's reported threat classification where available.
- Scan records must not contain file contents, credentials or encryption secrets.

**Indexes**

Proposed indexes:

- Index on `(firm_id, document_version_id, scan_type, created_at DESC)`.
- Index on `(firm_id, document_id, created_at DESC)`.
- Index on `(firm_id, scan_status, created_at DESC)`.
- Index on `(firm_id, scan_result, created_at DESC)`.
- Index on `(firm_id, threat_name)` where threat information is present.

**Tenant Isolation and RLS**

- Every scan result must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Composite foreign keys must enforce same-firm document and version relationships.
- Scan results must only be retrieved by authorized firm users or controlled system services.
- Platform-level security investigation access must follow separately controlled and audited procedures.

**Business and Security Rules**

- Every uploaded file version must undergo the required security validation.
- Malware scanning must run before ordinary document access is permitted.
- Files awaiting scanning must remain quarantined.
- Files identified as malicious must remain quarantined or be rejected.
- A scanner failure, timeout or inconclusive result must not be interpreted as a clean result.
- The application must verify the checksum of the scanned file against the checksum recorded for the document version.
- A new file version must undergo a fresh scan even if an earlier version passed.
- Scan results must be generated by trusted background processing services.
- Users must not be allowed to manually mark a failed or suspicious scan as clean.
- Any authorized override process must require explicit privileged approval and generate security audit evidence.
- Scanner credentials and infrastructure details must not be exposed through customer or staff interfaces.

**Data Integrity and Lifecycle**

- Each scan execution must create a traceable result record.
- Re-scanning a file must create a new scan record rather than silently replacing historical results.
- Scan results must remain associated with the exact document version.
- The document version must only become available after all required security checks have passed.
- Scanner failures and suspicious results must generate appropriate operational alerts.
- Historical scan evidence must be retained according to security and legal retention policies.
- Scan processing must be coordinated with background job execution and document lifecycle management.

**Design Notes and Open Decisions**

- The malware scanning provider and infrastructure remain to be selected.
- Required scanning types and acceptance criteria must be finalized.
- The system must define whether a new scan is required when malware signatures are updated.
- Re-scanning policies for previously stored documents must be established.
- Quarantine retention and secure deletion policies must be defined.
- The relationship between scan results, document status and document-version status must be enforced consistently.
- Scanner outage handling, retry policies and operational alerting must be finalized before implementation.

### 3.43 `checklist_template`

**Purpose**

The `checklist_template` table stores reusable checklist definitions that CA firms use to manage customer document requirements and ITR preparation tasks.

A checklist template defines the overall checklist structure and its applicability. Individual checklist items will be maintained separately in `checklist_item_template`.

Templates allow firms to standardize document collection and preparation procedures while retaining the flexibility to configure firm-specific requirements.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Each firm manages its own checklist templates.
- Platform-provided default templates, if introduced, must follow a separately defined ownership and publishing model.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `template_code` | VARCHAR(100) | No | Firm-specific template identifier |
| `template_name` | VARCHAR(255) | No | Display name of the checklist |
| `description` | TEXT | Yes | Purpose and scope of the checklist |
| `assessment_year_id` | UUID | Yes | Applicable assessment year |
| `taxpayer_category` | VARCHAR(50) | Yes | Applicable taxpayer category |
| `itr_type` | VARCHAR(50) | Yes | Applicable ITR type |
| `template_status` | VARCHAR(30) | No | Template lifecycle status |
| `is_default` | BOOLEAN | No | Indicates the firm's default template |
| `version_number` | INTEGER | No | Template version number |
| `created_by_membership_id` | UUID | No | Firm membership creating the template |
| `updated_by_membership_id` | UUID | Yes | Firm membership updating the template |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `archived_at` | TIMESTAMPTZ | Yes | Archival timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`.
- `(firm_id, created_by_membership_id)` references the corresponding firm membership.
- `(firm_id, updated_by_membership_id)` references the corresponding firm membership.
- `assessment_year_id` references `assessment_year.id`.

Firm-specific references must be protected through composite foreign keys.

**Template Statuses**

Initial supported values:

- `draft`
- `active`
- `inactive`
- `archived`

**Constraints and Data Integrity**

- `firm_id` is mandatory.
- Template code and name must not be empty.
- `version_number` must be greater than zero.
- Template code must be unique within the firm.
- A firm may have multiple templates for different assessment years and taxpayer categories.
- An archived template must not be used to create new customer checklists.
- The applicable assessment year must exist in the platform assessment-year catalogue.
- Template applicability must be validated before assigning it to an ITR case.

**Indexes**

Proposed indexes:

- Unique index on `(firm_id, template_code)`.
- Index on `(firm_id, template_status)`.
- Index on `(firm_id, assessment_year_id, taxpayer_category)`.
- Index on `(firm_id, is_default)`.

**Tenant Isolation and RLS**

- Every firm-specific template must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Firm-specific templates must never be visible or editable by another firm.
- Platform-provided templates, if implemented, must be accessed through a separately authorized mechanism.
- Template access must also follow firm membership and permission rules.

**Business and Security Rules**

- Authorized firm administrators and staff may create templates according to their permissions.
- Templates may define requirements for documents, information collection and preparation tasks.
- Checklist items must be maintained in `checklist_item_template`.
- Templates must support assessment-year-specific requirements.
- Templates may be configured for different taxpayer categories and ITR types.
- Changes to an existing template must not silently alter checklists already assigned to customers.
- Template versions must preserve the historical requirements applicable when a customer checklist was created.
- Deactivating or archiving a template must not delete existing customer checklist records.
- Template changes must be auditable.

**Data Integrity and Lifecycle**

- Draft templates may be edited by authorized users.
- Active templates may be assigned to new customer ITR cases.
- Template changes that affect existing workflows must follow a controlled versioning process.
- Archiving must preserve historical checklist references.
- Template deletion must be restricted where existing customer checklists depend on it.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The exact taxpayer-category catalogue must be finalized.
- The system must define whether a template applies to one or multiple ITR types.
- Template versioning and activation rules must be finalized.
- The system must define whether firms can customize platform-provided default templates.
- The process for selecting a template automatically for a new ITR case must be defined.
- The relationship between checklist templates and assessment-year-specific statutory requirements must be reviewed before implementation.

### 3.44 `checklist_item_template`

**Purpose**

The `checklist_item_template` table stores individual checklist items belonging to a firm-specific checklist template.

Each item represents a document requirement, information request or task that must be completed as part of customer onboarding, document collection or ITR preparation.

The table allows firms to configure the order, applicability, mandatory status and completion requirements of individual checklist items.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Every checklist item must belong to a checklist template owned by the same firm.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `checklist_template_id` | UUID | No | Parent checklist template |
| `item_code` | VARCHAR(100) | No | Template-specific item identifier |
| `item_name` | VARCHAR(255) | No | Display name of the checklist item |
| `description` | TEXT | Yes | Detailed requirement or instructions |
| `item_type` | VARCHAR(30) | No | Document, information or task requirement |
| `document_category` | VARCHAR(50) | Yes | Associated document category |
| `document_type` | VARCHAR(50) | Yes | Expected document type |
| `is_mandatory` | BOOLEAN | No | Indicates whether completion is required |
| `display_order` | INTEGER | No | Display sequence within the template |
| `customer_visible` | BOOLEAN | No | Indicates whether the item is visible to customers |
| `requires_staff_review` | BOOLEAN | No | Indicates whether staff review is required |
| `allow_multiple_files` | BOOLEAN | No | Indicates whether multiple documents may satisfy the item |
| `item_configuration` | JSONB | No | Additional validated item-specific configuration |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `deleted_at` | TIMESTAMPTZ | Yes | Controlled soft-deletion timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, checklist_template_id)` references the corresponding checklist template.
- Composite foreign keys must prevent an item from referencing a template belonging to another firm.

**Item Types**

Initial supported values:

- `document`
- `information`
- `task`

**Constraints and Data Integrity**

- `firm_id` is mandatory.
- Parent checklist template is mandatory.
- Item code and item name must not be empty.
- `display_order` must be zero or greater.
- `item_configuration` must contain a valid JSON object.
- A document item must define an appropriate document category or document type.
- Non-document items must not be treated as file-upload requirements.
- Item codes must be unique within their parent template.
- Deleted items must not appear in newly generated customer checklists.
- Item configuration must be validated against the supported schema for its item type.

**Indexes**

Proposed indexes:

- Index on `(firm_id, checklist_template_id, display_order)`.
- Unique index on `(firm_id, checklist_template_id, item_code)`.
- Index on `(firm_id, checklist_template_id, deleted_at)`.
- Index on `(firm_id, item_type)`.

**Tenant Isolation and RLS**

- Every item must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Composite foreign keys must enforce same-firm template ownership.
- Access to checklist items must also follow the authorization rules of the parent template.

**Business and Security Rules**

- Authorized firm users may create and configure checklist items.
- Items must belong to an existing firm checklist template.
- Item ordering must determine the presentation sequence in customer and staff portals.
- Mandatory items must be completed before the corresponding checklist can be considered fully complete, subject to approved exception rules.
- Customer visibility must be independent of internal staff permissions.
- Internal-only tasks and instructions must never be exposed through customer-facing APIs.
- Items requiring staff review must not be considered approved merely because a customer uploaded a document.
- Multiple-file requirements must be handled through the customer checklist item and document relationship.
- Template changes must not silently modify previously generated customer checklist instances.
- Item configuration must not contain passwords, authentication tokens or other secrets.

**Data Integrity and Lifecycle**

- Checklist items must be managed through controlled template editing.
- Changes to active templates must follow the approved template-versioning process.
- Existing customer checklist instances must preserve the original item requirements.
- Deleting an item must not remove historical customer checklist responses or associated documents.
- Historical checklist references must remain traceable.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The supported document-category and document-type catalogues must be finalized.
- The system must define whether document items can accept multiple document types.
- Conditional checklist items based on taxpayer category or other conditions may require a defined rules model.
- The exact configuration schemas for document, information and task items must be established.
- The process for migrating existing customer checklist instances when a template is revised must be defined.
- The relationship between checklist completion, document approval and ITR workflow progression must be finalized.

### 3.45 `customer_checklist`

**Purpose**

The `customer_checklist` table stores an actual checklist instance assigned to a customer's ITR case.

A checklist instance is generated from a firm-owned `checklist_template` and contains the requirements applicable to a particular customer and assessment year.

It provides a parent record for individual checklist responses maintained in `customer_checklist_item`.

The checklist instance must preserve the template version and applicable requirements used at the time of generation.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Every checklist must belong to a firm-customer relationship and an ITR case within the same firm.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `firm_customer_id` | UUID | No | Associated firm-customer relationship |
| `customer_itr_id` | UUID | No | Associated ITR case |
| `checklist_template_id` | UUID | No | Source checklist template |
| `template_version_number` | INTEGER | No | Template version used to generate the checklist |
| `checklist_name` | VARCHAR(255) | No | Checklist display name |
| `checklist_status` | VARCHAR(30) | No | Overall checklist status |
| `assigned_to_membership_id` | UUID | Yes | Responsible firm staff member |
| `generated_at` | TIMESTAMPTZ | No | Checklist generation timestamp |
| `due_at` | TIMESTAMPTZ | Yes | Checklist completion deadline |
| `completed_at` | TIMESTAMPTZ | Yes | Checklist completion timestamp |
| `created_by_membership_id` | UUID | No | Membership generating the checklist |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `deleted_at` | TIMESTAMPTZ | Yes | Controlled soft-deletion timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, firm_customer_id)` references the corresponding firm-customer relationship.
- `(firm_id, customer_itr_id)` references the corresponding customer ITR record.
- `(firm_id, checklist_template_id)` references the corresponding checklist template.
- `(firm_id, assigned_to_membership_id)` references the corresponding firm membership.
- `(firm_id, created_by_membership_id)` references the corresponding firm membership.

Composite foreign keys must prevent cross-firm references.

The application and database must ensure that the associated ITR case belongs to the specified firm-customer relationship.

**Checklist Statuses**

Initial supported values:

- `draft`
- `active`
- `in_progress`
- `pending_review`
- `completed`
- `cancelled`
- `archived`

**Constraints and Data Integrity**

- `firm_id` is mandatory.
- Customer, ITR case and source template references are mandatory.
- Template version number must be greater than zero.
- Checklist name must not be empty.
- Completion timestamp must be present when status is `completed`.
- The source template must belong to the same firm.
- The source template version must be identifiable and historically available.
- A checklist must not be generated from an inactive or archived template unless an explicitly authorized exception applies.
- The customer and ITR case relationship must be consistent.

**Indexes**

Proposed indexes:

- Index on `(firm_id, customer_itr_id, checklist_status)`.
- Index on `(firm_id, firm_customer_id, created_at DESC)`.
- Index on `(firm_id, assigned_to_membership_id, checklist_status)`.
- Index on `(firm_id, checklist_template_id, template_version_number)`.
- Index on `(firm_id, due_at)`.

**Tenant Isolation and RLS**

- Every checklist must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Composite foreign keys must enforce same-firm customer, ITR, template and membership references.
- Checklist access must also satisfy firm membership, role permissions and customer-level access grants.
- Customer-facing access must expose only checklist information explicitly designated as customer-visible.

**Business and Security Rules**

- A checklist is generated from an approved firm template.
- Generation must create the corresponding `customer_checklist_item` records.
- The source template version must be preserved.
- Subsequent template changes must not silently alter an existing checklist.
- Staff assignments must follow the firm's authorization rules.
- Customers may view and respond to checklist items permitted for customer access.
- Internal tasks and staff-only instructions must remain inaccessible through customer-facing APIs.
- Checklist completion must consider mandatory items and required staff reviews.
- An incomplete or rejected mandatory document must prevent checklist completion unless an authorized exception is recorded.
- Checklist generation and item creation must be performed transactionally.
- Checklist cancellation and archival must preserve historical evidence.

**Data Integrity and Lifecycle**

- Checklist creation and generation of its child items must succeed or fail together.
- Checklist status transitions must be validated by the backend.
- Completion status must reflect the actual state of required checklist items.
- A checklist must not be marked completed merely because all documents were uploaded; required reviews and approvals must also be satisfied.
- Changes to checklist requirements must follow a controlled process.
- Deletion must not cascade to uploaded documents, reviews or historical ITR records.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The precise snapshot strategy for preserving template and item definitions must be finalized.
- Whether a customer may have multiple active checklists for the same ITR case must be defined.
- Checklist reassignment and deadline changes must be auditable.
- The process for adding newly required items to an existing checklist must be established.
- The relationship between checklist completion and ITR status transitions must be finalized.
- Authorized exception handling for mandatory checklist requirements must be defined.

### 3.46 `customer_checklist_item`

**Purpose**

The `customer_checklist_item` table stores the individual requirements and their completion status within a customer's checklist.

Each record represents one checklist requirement generated from a `checklist_item_template`.

It tracks customer responses, document submissions, staff review outcomes and completion timestamps.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Every checklist item must belong to a customer checklist owned by the same firm.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `customer_checklist_id` | UUID | No | Parent customer checklist |
| `checklist_item_template_id` | UUID | Yes | Source template item, when available |
| `item_code` | VARCHAR(100) | No | Preserved template item code |
| `item_name` | VARCHAR(255) | No | Preserved checklist item name |
| `item_description` | TEXT | Yes | Preserved requirement description |
| `item_type` | VARCHAR(30) | No | Document, information or task |
| `is_mandatory` | BOOLEAN | No | Indicates whether completion is required |
| `customer_visible` | BOOLEAN | No | Indicates whether the customer can view the item |
| `item_status` | VARCHAR(30) | No | Current completion status |
| `customer_response` | TEXT | Yes | Customer-provided response or information |
| `staff_review_status` | VARCHAR(30) | No | Staff review outcome |
| `reviewed_by_membership_id` | UUID | Yes | Staff member reviewing the item |
| `reviewed_at` | TIMESTAMPTZ | Yes | Review timestamp |
| `completed_at` | TIMESTAMPTZ | Yes | Item completion timestamp |
| `display_order` | INTEGER | No | Display sequence |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `deleted_at` | TIMESTAMPTZ | Yes | Controlled soft-deletion timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, customer_checklist_id)` references the corresponding customer checklist.
- `(firm_id, checklist_item_template_id)` references the corresponding checklist item template when present.
- `(firm_id, reviewed_by_membership_id)` references the corresponding firm membership when present.

Composite foreign keys must prevent cross-firm references.

The application must verify that a source template item belongs to the template version used to generate the parent customer checklist.

**Item Types**

Initial supported values:

- `document`
- `information`
- `task`

**Item Statuses**

Initial supported values:

- `pending`
- `submitted`
- `in_review`
- `correction_required`
- `completed`
- `not_applicable`
- `cancelled`

**Staff Review Statuses**

Initial supported values:

- `not_required`
- `pending`
- `approved`
- `rejected`
- `correction_required`

**Constraints and Data Integrity**

- `firm_id` and parent checklist references are mandatory.
- Item code and item name must not be empty.
- `display_order` must be zero or greater.
- A completed item must have a completion timestamp.
- A review decision must have a reviewer and review timestamp.
- An item requiring review must not be considered fully completed until the required review succeeds.
- A mandatory item must not be marked `not_applicable` without an authorized exception.
- Customer responses must be stored separately from internal staff comments.
- Source template references must remain consistent with the parent checklist's template version.

**Indexes**

Proposed indexes:

- Index on `(firm_id, customer_checklist_id, display_order)`.
- Index on `(firm_id, customer_checklist_id, item_status)`.
- Index on `(firm_id, item_status, updated_at DESC)`.
- Index on `(firm_id, reviewed_by_membership_id, reviewed_at DESC)`.
- Index on `(firm_id, checklist_item_template_id)`.

**Tenant Isolation and RLS**

- Every checklist item must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Composite foreign keys must enforce same-firm parent, template and reviewer references.
- Customer access must be restricted to items belonging to an authorized firm-customer relationship.
- Customer-visible and internal-only fields must be separated through backend authorization and response filtering.

**Business and Security Rules**

- Checklist items must be generated from the approved template version.
- The item name, description, mandatory status, visibility and ordering must preserve the applicable template definition.
- Customers may submit information or documents only for items authorized for customer interaction.
- Document submissions must reference records managed through the document-management model.
- Multiple files may be associated with one item when the template permits them.
- Staff review decisions must be recorded separately from customer submission status.
- Rejected submissions must support correction requests.
- Internal staff comments must not be exposed as customer responses.
- Customers must not be allowed to approve their own submitted documents.
- Completion must follow the item's type-specific requirements.
- All changes to item status and review outcomes must be auditable.

**Data Integrity and Lifecycle**

- Checklist item creation must occur in the same transaction as checklist generation.
- Status transitions must be validated by the backend.
- Repeated submissions must preserve relevant document and response history.
- Item completion must reflect actual submission and review requirements.
- Changes to a template must not silently alter existing checklist item requirements.
- Deletion must preserve historical checklist and document evidence.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- A separate document-to-checklist-item association table may be needed to support multiple uploaded documents per item and preserve submission history.
- The exact customer response model for structured information items must be finalized.
- The relationship between item completion and document approval must be defined.
- The system must define how authorized exceptions to mandatory requirements are recorded.
- Whether item-level due dates and staff assignments are required must be finalized.
- The relationship between checklist item review and `document_review` must be defined to avoid duplicate review decisions.
- A dedicated checklist item history table may be needed to preserve every status transition.

### 3.47 `workflow_transition_history`

**Purpose**

The `workflow_transition_history` table records significant workflow status transitions within the ITR Management System.

It provides a historical record of workflow changes affecting ITR cases, checklists and other supported business processes.

The table supports operational traceability, audit investigations, workflow monitoring and verification of authorized status transitions.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Records must be append-only.
- Every firm-specific workflow transition must belong to the same firm as its associated business record.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `workflow_entity_type` | VARCHAR(50) | No | Type of business entity |
| `workflow_entity_id` | UUID | No | Identifier of the affected business record |
| `workflow_name` | VARCHAR(100) | No | Workflow being tracked |
| `previous_status` | VARCHAR(50) | Yes | Status before transition |
| `new_status` | VARCHAR(50) | No | Status after transition |
| `transition_reason` | TEXT | Yes | Reason for the transition |
| `transition_source` | VARCHAR(50) | No | Source initiating the transition |
| `changed_by_membership_id` | UUID | Yes | Firm membership initiating the transition |
| `correlation_id` | UUID | Yes | Related request or business operation identifier |
| `transition_data` | JSONB | No | Additional validated transition information |
| `occurred_at` | TIMESTAMPTZ | No | Time the transition occurred |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`.
- `(firm_id, changed_by_membership_id)` references the corresponding firm membership when the transition is initiated by a firm member.

Because the affected entity may be an ITR case, checklist or another supported business record, its reference must be validated through a controlled entity-specific mechanism.

A generic entity identifier must not be treated as a substitute for database-enforced referential integrity.

**Workflow Entity Types**

Initial supported values:

- `customer_itr`
- `customer_checklist`
- `customer_checklist_item`
- `document`
- `firm_customer`

Additional entity types may be introduced through controlled schema and application changes.

**Transition Sources**

Initial supported values:

- `firm_admin`
- `firm_staff`
- `customer`
- `system`
- `automated_process`
- `authorized_platform_operation`

System and automated transitions must have an identifiable system actor rather than being attributed to an ordinary user.

**Constraints and Data Integrity**

- `firm_id` is mandatory.
- Workflow entity type, workflow name and new status must not be empty.
- `transition_data` must contain a valid JSON object.
- Previous and new statuses must not be identical.
- The transition timestamp must be recorded.
- Actor attribution must be consistent with the transition source.
- The transition must be valid under the applicable workflow rules.
- Cross-firm entity references must be rejected.
- Status transitions must not be recorded without the corresponding business operation succeeding, except where a separately defined failure-event mechanism applies.

**Indexes**

Proposed indexes:

- Index on `(firm_id, workflow_entity_type, workflow_entity_id, occurred_at DESC)`.
- Index on `(firm_id, workflow_name, occurred_at DESC)`.
- Index on `(firm_id, new_status, occurred_at DESC)`.
- Index on `(firm_id, changed_by_membership_id, occurred_at DESC)`.
- Index on `correlation_id`.
- Index on `occurred_at` for retention and archival operations.

**Tenant Isolation and RLS**

- Every firm-specific transition record must contain `firm_id`.
- RLS must restrict access to the current tenant.
- The application must validate that the affected entity belongs to the specified firm.
- Firm users must only retrieve workflow history they are authorized to view.
- Platform-level access must follow separately controlled and audited procedures.

**Business and Security Rules**

- Workflow transitions must be initiated through authorized backend services.
- The application must validate the current status before applying a transition.
- Only permitted transitions may be performed.
- The business-record status update and its corresponding history record must be committed in the same database transaction.
- Concurrent transitions must be handled safely to prevent conflicting status changes.
- Each transition must preserve the previous status and new status.
- Transition reasons must be recorded where required by business rules.
- Automated transitions must be distinguishable from human-initiated transitions.
- Workflow history must not be editable through ordinary firm administration interfaces.
- Important transitions must also generate appropriate general audit events where required.

**Data Integrity and Lifecycle**

- Workflow history must be append-only.
- Historical records must not be silently overwritten or deleted.
- Corrective actions must be represented by new records.
- The latest valid transition should be consistent with the current status of the associated business entity.
- Transition history must remain available when the business entity is completed, cancelled or archived.
- Retention and archival must follow approved legal and business requirements.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The exact workflow catalogue and permitted transition matrices must be finalized.
- The relationship between this table and `itr_status_history`, `customer_status_history` and checklist-specific status history must be reviewed to avoid duplicate records.
- The generic entity-reference strategy must be finalized, including database-level integrity enforcement.
- The system must define how system-originated actors are represented.
- Concurrency control and transaction retry policies must be finalized.
- The relationship between workflow history and general `audit_event` records must be established.
- Whether workflow transitions require approval or segregation of duties must be defined for sensitive operations.

### 3.48 `conversation`

**Purpose**

The `conversation` table stores communication threads between customers and CA firms within the ITR Management System.

A conversation provides a common communication context for messages exchanged between a customer and authorized firm users.

Conversations may relate to a customer relationship, a specific ITR case or a general customer enquiry.

Individual messages will be stored separately in the `message` table.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Every conversation must belong to one firm-customer relationship.
- A conversation must not be shared across firms.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `firm_customer_id` | UUID | No | Associated firm-customer relationship |
| `customer_itr_id` | UUID | Yes | Related ITR case, if applicable |
| `conversation_reference` | VARCHAR(100) | No | Firm-specific conversation reference |
| `conversation_subject` | VARCHAR(255) | No | Conversation subject |
| `conversation_type` | VARCHAR(30) | No | Conversation category |
| `conversation_status` | VARCHAR(30) | No | Current conversation status |
| `created_by_user_id` | UUID | No | User who initiated the conversation |
| `assigned_to_membership_id` | UUID | Yes | Responsible firm staff member |
| `last_message_at` | TIMESTAMPTZ | Yes | Timestamp of the latest message |
| `closed_at` | TIMESTAMPTZ | Yes | Conversation closure timestamp |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `deleted_at` | TIMESTAMPTZ | Yes | Controlled soft-deletion timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, firm_customer_id)` references the corresponding firm-customer relationship.
- `(firm_id, customer_itr_id)` references the corresponding customer ITR record when present.
- `(firm_id, assigned_to_membership_id)` references the corresponding firm membership when present.
- `created_by_user_id` references `user.id`.

Composite foreign keys must prevent cross-firm references.

The initiating user must be validated as an authorized customer or firm member associated with the conversation's firm-customer relationship.

**Conversation Types**

Initial supported values:

- `general_enquiry`
- `itr_related`
- `document_request`
- `document_clarification`
- `payment_related`
- `refund_related`
- `support`
- `other`

**Conversation Statuses**

Initial supported values:

- `open`
- `in_progress`
- `waiting_for_customer`
- `waiting_for_firm`
- `resolved`
- `closed`

**Constraints and Data Integrity**

- `firm_id` is mandatory.
- `firm_customer_id` is mandatory.
- Conversation reference and subject must not be empty.
- Conversation reference must be unique within the firm.
- If an ITR case is referenced, it must belong to the same firm-customer relationship.
- The initiating user must have a valid relationship with the conversation.
- A closed conversation must have a closure timestamp.
- A reopened conversation must follow an authorized status transition.
- Last-message timestamp must reflect the latest successfully recorded message.

**Indexes**

Proposed indexes:

- Unique index on `(firm_id, conversation_reference)`.
- Index on `(firm_id, firm_customer_id, updated_at DESC)`.
- Index on `(firm_id, customer_itr_id)`.
- Index on `(firm_id, conversation_status, updated_at DESC)`.
- Index on `(firm_id, assigned_to_membership_id, conversation_status)`.
- Index on `(firm_id, last_message_at DESC)`.

**Tenant Isolation and RLS**

- Every conversation must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Composite foreign keys must enforce same-firm customer, ITR and membership relationships.
- Customers may access only conversations belonging to their own firm-customer relationship.
- Firm staff access must satisfy membership permissions and customer-level access rules.
- Platform-level support access must follow separately controlled and audited procedures.

**Business and Security Rules**

- Customers may initiate conversations through the customer portal.
- Authorized firm staff may initiate conversations on behalf of a customer.
- A conversation may be associated with an ITR case or remain a general customer communication thread.
- Conversation access must be validated on every backend operation.
- Assigning a conversation to a staff member must not independently grant customer-record access.
- Message creation must validate the conversation's current status and participant permissions.
- Closing or resolving a conversation must not delete its messages.
- Reopening must follow an authorized workflow.
- Internal staff-only notes must not be stored as customer-visible messages.
- Sensitive tax documents should be exchanged through the secure document-management workflow rather than unrestricted message attachments.
- Conversation creation, assignment, closure and reopening must be auditable.

**Data Integrity and Lifecycle**

- Conversation creation must be transactionally consistent.
- Message creation must update the conversation's last-message timestamp reliably.
- Conversation status changes must follow controlled transitions.
- Closing a conversation must preserve its communication history.
- Soft deletion must not cascade to messages or document records.
- Retention and archival must follow approved communication and legal requirements.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The participant model must be finalized, including whether multiple firm staff members can participate in one conversation.
- The relationship between conversations and customer-facing notifications must be defined.
- Whether customers can reopen resolved or closed conversations must be established.
- Internal notes and customer-visible messages must remain strictly separated.
- The system must define whether conversations support multiple ITR cases or exactly one associated ITR case.
- Message retention and archival requirements must be finalized.
- The handling of conversations after a customer-firm relationship ends must be defined.

### 3.49 `message`

**Purpose**

The `message` table stores individual messages exchanged within a conversation between customers and authorized firm users.

Each message belongs to one conversation and preserves its sender, content, message type, delivery status and timestamps.

The table supports customer enquiries, ITR-related communication, document clarification and other authorized business communications.

Internal staff notes must be handled separately from customer-visible messages through explicit visibility controls.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Every message must belong to a conversation owned by the same firm.
- Messages must never be shared across firms.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `conversation_id` | UUID | No | Parent conversation |
| `sender_user_id` | UUID | No | User identity of the sender |
| `sender_membership_id` | UUID | Yes | Firm membership context for staff messages |
| `message_type` | VARCHAR(30) | No | Type of message |
| `message_content` | TEXT | No | Message content |
| `visibility` | VARCHAR(30) | No | Message visibility classification |
| `message_status` | VARCHAR(30) | No | Message processing status |
| `reply_to_message_id` | UUID | Yes | Parent message when replying |
| `edited_at` | TIMESTAMPTZ | Yes | Last edit timestamp |
| `sent_at` | TIMESTAMPTZ | No | Message creation or sending timestamp |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `deleted_at` | TIMESTAMPTZ | Yes | Controlled soft-deletion timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, conversation_id)` references the corresponding conversation.
- `(firm_id, sender_membership_id)` references the corresponding firm membership when the sender is a firm member.
- `sender_user_id` references `user.id`.
- `(firm_id, reply_to_message_id)` references another message within the same firm.

Composite foreign keys must prevent cross-firm references.

The application must validate that the sender is an authorized participant in the parent conversation.

For staff messages, the sender user and membership must identify the same person.

**Message Types**

Initial supported values:

- `text`
- `system`
- `internal_note`
- `automated_notification`

**Message Visibility**

Initial supported values:

- `customer_visible`
- `internal`

Customer-visible messages may be accessed by the authorized customer and firm participants.

Internal messages must remain accessible only to authorized firm personnel.

**Message Statuses**

Initial supported values:

- `draft`
- `sent`
- `failed`
- `deleted`

Delivery and read-receipt statuses will be maintained separately through the notification and message-delivery design.

**Constraints and Data Integrity**

- `firm_id` and `conversation_id` are mandatory.
- Sender user must be identified.
- Message content must not be empty for ordinary text messages.
- Message type and visibility must use supported values.
- A reply must reference a message in the same conversation.
- Staff membership references must belong to the same firm.
- Internal messages must never be exposed through customer-facing APIs.
- Edited and deleted messages must preserve appropriate audit evidence.
- System-generated messages must have an identifiable system-origin actor.

**Indexes**

Proposed indexes:

- Index on `(firm_id, conversation_id, sent_at DESC)`.
- Index on `(firm_id, sender_user_id, sent_at DESC)`.
- Index on `(firm_id, sender_membership_id, sent_at DESC)`.
- Index on `(firm_id, message_status, sent_at DESC)`.
- Index on `(firm_id, reply_to_message_id)`.
- Index on `(firm_id, visibility, sent_at DESC)`.

**Tenant Isolation and RLS**

- Every message must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Composite foreign keys must enforce same-firm conversation, sender membership and reply references.
- Customers may retrieve only customer-visible messages from their own authorized conversations.
- Firm staff access must satisfy membership permissions and customer-level access grants.
- Internal messages must be excluded from customer-facing query results at the backend authorization layer, not merely hidden in the user interface.

**Business and Security Rules**

- A message must be created only by an authorized conversation participant or trusted system service.
- The sender must be authorized for the parent conversation at the time of sending.
- Customer-visible and internal messages must remain strictly separated.
- Internal staff notes must not trigger customer notifications.
- Messages must be associated with the correct conversation.
- Replies must remain within the same conversation.
- Message content must be safely validated and rendered to prevent stored cross-site scripting (XSS).
- Message length and permitted content must follow configurable application limits.
- Messages must not contain passwords, OTPs, session tokens or encryption keys.
- Sensitive tax documents must use the secure document-management workflow rather than unrestricted message content.
- Message creation, editing, deletion and access must be auditable.
- Editing a sent message must preserve the original content through an approved version-history or audit mechanism.

**Data Integrity and Lifecycle**

- Message creation and conversation last-message timestamp updates must be coordinated reliably.
- Failed message delivery must not be represented as successful delivery.
- Deleted messages must follow a controlled retention process.
- Replies must retain their relationship to the original message.
- Conversation closure must not automatically delete message history.
- Historical messages must remain available to authorized reviewers according to retention requirements.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must decide whether edited messages require a dedicated `message_revision` table.
- The precise system-message and automated-message actor model must be finalized.
- The relationship between message sending and `notification_delivery` must be defined.
- Message delivery, read receipts and recipient-specific status tracking must be designed separately.
- The system must define whether multiple staff users can participate in one conversation.
- Internal-note editing and deletion permissions must be established.
- Message retention, archival and legal-hold requirements must be finalized.
- The system must define how messages are handled when a customer-firm relationship ends.

### 3.50 `message_attachment`

**Purpose**

The `message_attachment` table stores references to documents attached to individual messages within a conversation.

It connects the messaging system with the existing document-management system. Actual file storage, scanning, access control and version management remain the responsibility of the document tables and private object storage.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Every attachment must belong to a message and a document within the same firm.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `message_id` | UUID | No | Parent message |
| `document_id` | UUID | No | Referenced document |
| `document_version_id` | UUID | Yes | Specific document version attached |
| `attachment_name` | VARCHAR(255) | No | Display name for the attachment |
| `display_order` | INTEGER | No | Attachment ordering within the message |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `created_by_user_id` | UUID | No | User who attached the document |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, message_id)` references `message`.
- `(firm_id, document_id)` references `document`.
- `(firm_id, document_id, document_version_id)` must reference a version belonging to the specified document.
- `created_by_user_id` references `user.id`.

Composite foreign keys must prevent cross-firm references.

**Constraints and Data Integrity**

- Every attachment must reference an existing message and document.
- The referenced document must belong to the same firm as the message.
- If a document version is supplied, it must belong to the referenced document.
- `attachment_name` must not be empty.
- `display_order` must be zero or greater.
- Duplicate references to the same document version within one message should be prevented unless an explicit business requirement permits them.
- The application must verify that the sender is authorized to attach the document.
- Attachments must not bypass document access permissions.

**Indexes**

Proposed indexes:

- Index on `(firm_id, message_id, display_order)`.
- Index on `(firm_id, document_id)`.
- Index on `(firm_id, document_version_id)`.
- Index on `(firm_id, created_by_user_id, created_at DESC)`.

**Tenant Isolation and RLS**

- Every attachment must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Composite foreign keys must enforce same-firm message and document relationships.
- Customer access must be validated through the parent conversation and message visibility.
- Internal messages and their attachments must never be exposed to customers.
- Document-level authorization must still be enforced when an attachment is opened or downloaded.

**Business and Security Rules**

- Attachments must reference documents already registered in the document-management system.
- Uploads must follow the secure document upload, quarantine, malware scanning and approval workflow.
- The system must not store public file URLs or unrestricted object-storage paths in this table.
- Download access must use backend authorization and short-lived access mechanisms.
- Attaching a document must not grant additional access to that document beyond the permissions of the conversation and document.
- Customers must not attach documents to conversations belonging to another firm.
- Internal staff attachments must remain internal when the associated message is internal.
- Attachment creation and removal must be auditable.
- Removing an attachment from a message must not automatically delete the underlying document.

**Data Integrity and Lifecycle**

- Attachment creation must be coordinated with message creation where both occur in one operation.
- Document retention must remain independent of message attachment records.
- Deleting or closing a conversation must not automatically delete referenced documents.
- Document version references must preserve the exact version shared in the message.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must decide whether attachments can be added after a message has been sent.
- The system must define whether a message can contain multiple versions of the same document.
- The system must determine whether attachment removal is implemented through soft deletion or a separate removal-history mechanism.
- The system must define customer visibility rules for documents attached to internal messages.
- Attachment size and count limits must be configurable.
- The relationship between attachment access logs and document access logs must be finalized.

### 3.51 `notification`

**Purpose**

The `notification` table stores in-app notifications generated for customers and authorized firm users.

Notifications inform recipients about relevant business events, including new messages, document requests, ITR status changes, checklist updates, payment events and other important activities.

This table stores the notification record and its recipient-specific read status. Actual email, SMS and other external delivery attempts will be managed through the notification delivery mechanism.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped for firm-related notifications.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Every firm-related notification must belong to one firm.
- Platform-wide notifications, if required, must use a separately controlled platform notification mechanism rather than bypassing tenant isolation.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `recipient_user_id` | UUID | No | User receiving the notification |
| `firm_customer_id` | UUID | Yes | Related firm-customer relationship, when applicable |
| `notification_type` | VARCHAR(100) | No | Notification event type |
| `notification_title` | VARCHAR(255) | No | Notification heading |
| `notification_body` | TEXT | No | Notification message |
| `reference_entity_type` | VARCHAR(100) | Yes | Related business entity type |
| `reference_entity_id` | UUID | Yes | Related business record identifier |
| `notification_data` | JSONB | No | Additional structured notification metadata |
| `priority` | VARCHAR(20) | No | Notification priority |
| `notification_status` | VARCHAR(30) | No | Notification lifecycle status |
| `read_at` | TIMESTAMPTZ | Yes | Time the recipient read the notification |
| `expires_at` | TIMESTAMPTZ | Yes | Optional notification expiry |
| `created_at` | TIMESTAMPTZ | No | Notification creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, firm_customer_id)` references the corresponding firm-customer relationship when applicable.
- `recipient_user_id` references `user.id`.

The application must validate that the recipient is an authorized member or customer associated with the specified firm.

For customer notifications, the recipient must have an active or otherwise explicitly authorized relationship with the relevant firm.

For staff notifications, the recipient must have an eligible firm membership.

**Notification Types**

Initial supported event types include:

- `new_message`
- `document_requested`
- `document_uploaded`
- `document_review_completed`
- `itr_status_changed`
- `checklist_updated`
- `payment_received`
- `invoice_created`
- `refund_status_changed`
- `task_assigned`
- `task_due`
- `system_announcement`
- `other`

The notification type catalogue should remain configurable and centrally governed.

**Priority**

Initial supported values:

- `low`
- `normal`
- `high`
- `urgent`

**Notification Statuses**

Initial supported values:

- `unread`
- `read`
- `dismissed`
- `expired`

**Constraints and Data Integrity**

- `firm_id` and `recipient_user_id` are mandatory.
- Notification title and body must not be empty.
- Notification type, priority and status must use approved values.
- `notification_data` must contain a valid JSON object.
- `read_at` must be populated when status is `read`.
- An expired notification must not be treated as an active notification.
- The recipient must be authorized within the relevant firm.
- Related entity references must be validated by the application against the allowed entity catalogue and tenant.
- Notification metadata must not contain passwords, OTPs, session tokens or unnecessary sensitive tax information.

**Indexes**

Proposed indexes:

- Index on `(firm_id, recipient_user_id, notification_status, created_at DESC)`.
- Index on `(firm_id, notification_type, created_at DESC)`.
- Index on `(firm_id, priority, created_at DESC)`.
- Index on `(firm_id, firm_customer_id, created_at DESC)`.
- Index on `expires_at`.
- Index on `(firm_id, reference_entity_type, reference_entity_id)`.

**Tenant Isolation and RLS**

- Every firm-related notification must contain `firm_id`.
- RLS must restrict access to the current tenant.
- A recipient may retrieve only notifications addressed to their own user identity.
- Staff access must additionally satisfy firm membership and permission checks.
- Customer access must additionally satisfy the relevant firm-customer relationship and customer authorization rules.
- A notification must never expose information from another firm.
- Platform-wide notifications must use a separate, explicitly authorized access policy.

**Business and Security Rules**

- Notifications must be generated only from authorized business events or trusted system processes.
- The backend must determine the recipient; clients must not be allowed to choose arbitrary recipient identities.
- Notification generation must not grant access to the referenced business record.
- When a recipient opens a notification, the backend must independently authorize access to the referenced record.
- Internal messages and restricted staff notes must never generate customer-visible notifications containing their confidential content.
- Notification text must avoid unnecessary PAN, financial or other sensitive information.
- Notification content must be safely rendered to prevent stored XSS.
- Duplicate notifications caused by retried background jobs must be controlled through idempotency.
- Reading or dismissing a notification must not modify the underlying business record.
- Notification creation and recipient access must be auditable.

**Data Integrity and Lifecycle**

- Notification generation should be coordinated with the originating business transaction using a reliable event or outbox mechanism where appropriate.
- Notification delivery failure must not delete the in-app notification.
- Read and dismissal actions must update the appropriate recipient-specific state.
- Expired notifications must follow a controlled retention and cleanup policy.
- Notification history must be retained according to the approved retention requirements.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must decide whether notifications are always single-recipient records or whether a separate notification-recipient model is required for bulk notifications.
- The system must define whether notification content is generated immediately or from approved templates.
- The relationship between this table and `notification_delivery` must be finalized.
- The system must define notification deduplication keys and idempotency rules.
- The system must determine whether dismissed notifications remain available in notification history.
- Platform-wide announcements require a separate platform-level notification design.
- The system must define notification retention, archival and user preferences integration.
- The system must determine whether read receipts require additional device or channel metadata.

### 3.52 `notification_template`

**Purpose**

The `notification_template` table stores reusable templates used to generate in-app notifications and notification delivery content.

Templates define notification titles, message bodies and supported variables for events such as new messages, document requests, ITR status changes, payment updates and other business activities.

Templates must be validated before activation and rendered using approved, structured data.

**Ownership and Tenant Scope**

- Ownership: Platform-Owned or Firm-Owned, depending on template scope.
- Platform templates provide centrally maintained defaults.
- Firm templates allow authorized firm administrators to customize permitted notification content.
- Firm-specific templates must contain `firm_id`.
- Platform templates must have `firm_id` set to `NULL`.
- Platform and firm template access must follow separate authorization policies.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | Yes | Owning firm; null for platform templates |
| `template_key` | VARCHAR(150) | No | Stable template identifier |
| `template_name` | VARCHAR(255) | No | Human-readable template name |
| `notification_type` | VARCHAR(100) | No | Associated notification event type |
| `channel` | VARCHAR(30) | No | Intended notification channel |
| `language_code` | VARCHAR(10) | No | Template language |
| `title_template` | VARCHAR(255) | Yes | Notification title template |
| `body_template` | TEXT | No | Notification body template |
| `supported_variables` | JSONB | No | Approved template variables and metadata |
| `template_status` | VARCHAR(30) | No | Template lifecycle status |
| `version_number` | INTEGER | No | Template version |
| `created_by_membership_id` | UUID | Yes | Firm membership that created the template |
| `updated_by_membership_id` | UUID | Yes | Firm membership that last updated the template |
| `created_at` | TIMESTAMPTZ | No | Creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `archived_at` | TIMESTAMPTZ | Yes | Archival timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id` when the template is firm-owned.
- `(firm_id, created_by_membership_id)` references the corresponding firm membership for firm templates.
- `(firm_id, updated_by_membership_id)` references the corresponding firm membership for firm templates.

Platform templates must not use firm membership references.

**Template Channels**

Initial supported values:

- `in_app`
- `email`
- `sms`

Additional channels may be introduced through controlled schema changes.

**Template Statuses**

Initial supported values:

- `draft`
- `active`
- `inactive`
- `archived`

**Constraints and Data Integrity**

- `template_key`, `template_name`, `notification_type` and `body_template` must not be empty.
- `version_number` must be greater than zero.
- `supported_variables` must contain a valid JSON object.
- Template scope must be consistent with `firm_id`.
- Platform templates must not reference firm memberships.
- Firm templates must use creator and updater memberships from the same firm.
- Template keys must be unique within their respective ownership scopes and channel/language combinations, as finalized in the uniqueness strategy.
- Only validated templates may be activated.
- Active templates must have a supported notification type, channel and language.

**Indexes**

Proposed indexes:

- Unique index on `(template_key, channel, language_code)` for platform templates.
- Unique index on `(firm_id, template_key, channel, language_code)` for firm templates.
- Index on `(firm_id, notification_type, template_status)`.
- Index on `(channel, language_code, template_status)`.
- Index on `updated_at`.

The two uniqueness scopes must be implemented using separate partial unique indexes because platform templates have a null `firm_id`.

**Tenant Isolation and Access Control**

- Platform templates must be accessible only through authorized platform-level operations.
- Firm templates must be protected by RLS using `firm_id`.
- Firm administrators may manage firm templates according to permissions.
- Firm staff may use active templates but must not modify them without authorization.
- A firm must not modify platform-owned templates.
- Template selection must not permit a firm to access another firm's templates.

**Business and Security Rules**

- Templates must use an approved template-rendering engine.
- Variable names must be explicitly declared in `supported_variables`.
- Rendering must use trusted structured business data rather than arbitrary client-supplied values.
- Missing or invalid required variables must cause controlled rendering failure.
- Template content must be safely escaped or sanitized according to its output channel.
- Templates must not include passwords, OTPs, session tokens, encryption keys or unnecessary sensitive tax information.
- Template changes and activation must be auditable.
- Previously generated notifications must not change when a template is subsequently edited.
- Platform defaults may be used when a firm has not configured an eligible override.
- Firm customization must not override platform security notices or mandatory legal disclosures.

**Data Integrity and Lifecycle**

- Template versions must be preserved when content changes.
- Activating a new version must not silently modify already generated notifications.
- Archived templates must not be selected for new notification generation.
- Deactivation must preserve historical notification records.
- Template updates and activation must be performed transactionally.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must finalize template precedence between platform defaults and firm-specific overrides.
- The system must decide whether template versions are stored in this table or in a separate `notification_template_version` table.
- The supported template variable catalogue and validation rules must be defined.
- The system must determine whether firms can customize all channels or only selected channels.
- The system must define language fallback behavior when a requested language template is unavailable.
- The system must determine whether templates support conditional content and localization formatting.
- The system must define which notifications require mandatory platform-controlled wording.
- The system must finalize template retention and archival requirements.

### 3.53 `notification_delivery`

**Purpose**

The `notification_delivery` table records external notification delivery attempts through channels such as email and SMS.

It tracks delivery status, provider references, retry attempts, failure details and timestamps.

The in-app notification record remains independent of external delivery. A failed email or SMS delivery must not delete or invalidate the corresponding in-app notification.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned for firm-related notifications.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Every delivery record must reference a notification belonging to the same firm.
- Platform-wide notification delivery, if required, must use a separately controlled platform-level mechanism.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `notification_id` | UUID | No | Parent notification |
| `notification_template_id` | UUID | Yes | Template used to generate the delivery content |
| `channel` | VARCHAR(30) | No | Delivery channel |
| `recipient_address_encrypted` | BYTEA | Yes | Encrypted destination address or phone number |
| `recipient_address_hash` | VARCHAR(255) | Yes | Keyed hash for controlled matching and deduplication |
| `provider_name` | VARCHAR(100) | Yes | External delivery provider |
| `provider_message_id` | VARCHAR(255) | Yes | Provider-generated message reference |
| `delivery_status` | VARCHAR(30) | No | Current delivery status |
| `attempt_number` | INTEGER | No | Delivery attempt sequence |
| `idempotency_key` | VARCHAR(255) | No | Unique key for delivery deduplication |
| `scheduled_at` | TIMESTAMPTZ | Yes | Scheduled delivery time |
| `attempted_at` | TIMESTAMPTZ | Yes | Time the provider delivery attempt started |
| `accepted_at` | TIMESTAMPTZ | Yes | Time the provider accepted the request |
| `delivered_at` | TIMESTAMPTZ | Yes | Confirmed delivery time, if available |
| `failed_at` | TIMESTAMPTZ | Yes | Failure timestamp |
| `failure_code` | VARCHAR(100) | Yes | Normalized failure code |
| `failure_reason` | TEXT | Yes | Sanitized failure description |
| `provider_response` | JSONB | No | Sanitized provider response metadata |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, notification_id)` references the corresponding notification.
- `(firm_id, notification_template_id)` references the corresponding firm template where applicable.

Platform-owned templates may be referenced through a separately validated platform-template relationship because they do not have a firm identifier.

**Delivery Channels**

Initial supported values:

- `email`
- `sms`

Additional channels may be introduced through controlled schema changes.

**Delivery Statuses**

Initial supported values:

- `queued`
- `processing`
- `accepted`
- `delivered`
- `failed`
- `retry_scheduled`
- `cancelled`
- `undeliverable`

A provider accepting a request must not automatically be interpreted as confirmed delivery.

**Constraints and Data Integrity**

- `firm_id` and `notification_id` are mandatory.
- `attempt_number` must be greater than zero.
- `idempotency_key` must not be empty.
- Delivery channel and status must use approved values.
- `provider_response` must contain a valid JSON object.
- Delivery timestamps must follow valid chronological order.
- `delivered_at` must be populated only when delivery is confirmed.
- Failure details must be populated when a terminal failure is recorded.
- Recipient information must be protected and must not be stored in plaintext unnecessarily.
- Delivery attempts must reference the correct notification and tenant.

**Indexes**

Proposed indexes:

- Index on `(firm_id, notification_id, created_at DESC)`.
- Index on `(firm_id, delivery_status, scheduled_at)`.
- Index on `(firm_id, channel, delivery_status, created_at DESC)`.
- Unique index on `(firm_id, idempotency_key)`.
- Index on `(provider_name, provider_message_id)`.
- Index on `(firm_id, recipient_address_hash, created_at DESC)`.

**Tenant Isolation and RLS**

- Every firm-related delivery record must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Composite foreign keys must enforce same-firm notification references.
- Firm users must not access delivery records belonging to another firm.
- Customers must not directly access provider responses or internal failure details.
- Platform-level provider operations must use controlled service authorization.

**Business and Security Rules**

- Delivery attempts must be created by trusted backend services or authorized background jobs.
- Clients must not directly initiate arbitrary provider delivery operations.
- Provider credentials and API secrets must be stored in a secrets manager, never in this table.
- Recipient addresses must be encrypted using approved encryption and key-management controls.
- Recipient hashes must use a separately managed keyed hashing mechanism.
- Provider responses must be sanitized before persistence.
- Provider callbacks and webhooks must be authenticated and validated.
- Duplicate provider callbacks must be handled idempotently.
- Retry processing must use bounded retry policies and appropriate backoff.
- Permanent failures must not be retried indefinitely.
- Sensitive notification content must not be unnecessarily copied into provider response metadata or failure logs.
- Delivery attempts, retries, failures and provider status changes must be auditable.

**Data Integrity and Lifecycle**

- Each delivery attempt must have a traceable attempt number.
- Retry attempts must preserve the history of previous attempts.
- A retry must not overwrite the evidence of an earlier failure.
- Provider acceptance and confirmed delivery must remain distinct states.
- Delivery processing must support recovery after worker failure.
- Notification delivery must be coordinated reliably with the background-job and outbox mechanisms.
- Retention and archival must follow the approved communication data-retention policy.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must decide whether each retry creates a new row or updates a delivery record while maintaining a separate attempt-history table. The preferred approach is to preserve each attempt independently.
- The system must define provider selection and failover policies.
- The system must define retry limits, exponential backoff and permanent failure classification.
- The system must finalize webhook authentication and callback verification.
- The system must determine whether provider-specific response metadata requires a separate table.
- The system must define how platform-owned templates are referenced.
- The system must finalize encryption-key rotation and recipient-address retention.
- The system must determine how delivery records integrate with `notification` and background-job execution records.

### 3.54 `communication_preference`

**Purpose**

The `communication_preference` table stores communication channel preferences for customers and firm users within a specific CA firm.

It allows users to control optional notifications through supported channels such as email, SMS and in-app notifications.

Communication preferences must be respected by notification-generation and delivery services, subject to mandatory security, legal and operational communication requirements.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Each preference record belongs to one user and one firm.
- A user's preferences in one firm must not affect their preferences in another firm.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `user_id` | UUID | No | User whose preferences are stored |
| `communication_category` | VARCHAR(50) | No | Category of communication |
| `channel` | VARCHAR(30) | No | Communication channel |
| `is_enabled` | BOOLEAN | No | Whether the optional communication is enabled |
| `preference_source` | VARCHAR(30) | No | Source of the preference change |
| `updated_by_user_id` | UUID | Yes | User who last changed the preference |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`.
- `user_id` references `user.id`.
- `updated_by_user_id` references `user.id` when supplied.

The application must validate that the user has an authorized relationship with the specified firm.

A customer may manage their own preferences. Authorized firm administrators may manage preferences only where the approved business policy permits it.

**Communication Categories**

Initial supported values:

- `general_updates`
- `itr_updates`
- `document_updates`
- `payment_updates`
- `refund_updates`
- `marketing`
- `system_announcements`

Security alerts and legally mandatory communications must be handled through separately governed mandatory-notification rules.

**Communication Channels**

Initial supported values:

- `in_app`
- `email`
- `sms`

Additional channels may be introduced through controlled schema changes.

**Preference Sources**

Initial supported values:

- `user`
- `firm_admin`
- `system`
- `import`

The source must identify how the preference was established or last changed.

**Constraints and Data Integrity**

- `firm_id`, `user_id`, `communication_category` and `channel` are mandatory.
- `is_enabled` must have an explicit Boolean value.
- Category, channel and source must use approved values.
- Only one preference record may exist for each `(firm_id, user_id, communication_category, channel)` combination.
- The user must be associated with the relevant firm.
- Preference updates must preserve the previous value through appropriate audit evidence.
- Mandatory notifications must not be disabled through ordinary preference settings.

**Indexes**

Proposed indexes:

- Unique index on `(firm_id, user_id, communication_category, channel)`.
- Index on `(firm_id, user_id)`.
- Index on `(firm_id, communication_category, is_enabled)`.
- Index on `(firm_id, channel, is_enabled)`.
- Index on `updated_at`.

**Tenant Isolation and RLS**

- Every preference record must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Users may retrieve and update only their own preferences unless an explicitly authorized administrative operation permits otherwise.
- Firm administrators must not access or change preferences belonging to another firm.
- Customer and staff access must follow their respective firm relationships and authorization policies.
- Platform-level mandatory notification policies must remain outside firm-level preference overrides.

**Business and Security Rules**

- Users must be able to manage permitted communication preferences through an authenticated interface.
- The backend must enforce ownership and authorization for every preference change.
- Notification services must check preferences before sending optional communications.
- A disabled channel must suppress optional notifications through that channel.
- Mandatory security, authentication, legal and critical operational notifications must follow centrally governed rules.
- Marketing communications must follow applicable consent and opt-out requirements.
- Preference changes must take effect according to a defined processing policy.
- Unauthorized preference changes must be denied and recorded.
- Preference changes must be auditable.
- Imported preferences must be validated and must not silently override a user's existing explicit choices.

**Data Integrity and Lifecycle**

- Preference creation and updates must be transactional.
- Missing preference records must follow a documented default-preference policy.
- Preference changes must not alter historical notification or delivery records.
- User deactivation and firm relationship termination must follow the applicable preference-retention policy.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must finalize default preferences for newly registered customers and firm users.
- The system must decide whether preferences are user-managed, firm-managed or configurable by both under defined rules.
- The system must define mandatory notification categories and their permitted channels.
- The system must determine whether consent evidence requires a separate history table.
- The system must define how preference changes interact with already queued notifications.
- The system must establish marketing consent, withdrawal and compliance requirements.
- The system must decide whether communication preferences need language, quiet-hour or frequency settings.
- The system must define whether a central user preference can supplement, but never replace, firm-specific preferences.

### 3.55 `customer_invoice`

**Purpose**

The `customer_invoice` table stores invoices issued by a CA firm to its customers for professional services and other approved billable activities.

It provides the financial record for amounts charged, applicable taxes, payment obligations and invoice lifecycle status.

Invoices must be maintained independently for each firm and customer relationship. Financial amounts must use exact decimal types rather than floating-point types.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Every invoice must belong to one firm and one firm-customer relationship.
- A firm must not access or modify invoices belonging to another firm.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `firm_customer_id` | UUID | No | Customer relationship receiving the invoice |
| `invoice_number` | VARCHAR(100) | No | Firm-specific invoice reference |
| `invoice_date` | DATE | No | Invoice issue date |
| `due_date` | DATE | No | Payment due date |
| `invoice_status` | VARCHAR(30) | No | Invoice lifecycle status |
| `currency_code` | CHAR(3) | No | ISO currency code |
| `subtotal_amount` | NUMERIC(18,2) | No | Amount before taxes and adjustments |
| `discount_amount` | NUMERIC(18,2) | No | Total discount |
| `taxable_amount` | NUMERIC(18,2) | No | Amount subject to applicable taxes |
| `tax_amount` | NUMERIC(18,2) | No | Total tax amount |
| `other_charges_amount` | NUMERIC(18,2) | No | Other permitted charges |
| `rounding_adjustment` | NUMERIC(18,2) | No | Invoice rounding adjustment |
| `total_amount` | NUMERIC(18,2) | No | Final invoice amount |
| `amount_paid` | NUMERIC(18,2) | No | Total amount allocated as paid |
| `balance_amount` | NUMERIC(18,2) | No | Outstanding invoice balance |
| `invoice_description` | TEXT | Yes | Description of billed services |
| `customer_notes` | TEXT | Yes | Customer-visible invoice notes |
| `internal_notes` | TEXT | Yes | Firm-internal invoice notes |
| `issued_at` | TIMESTAMPTZ | Yes | Time the invoice was formally issued |
| `cancelled_at` | TIMESTAMPTZ | Yes | Invoice cancellation timestamp |
| `created_by_membership_id` | UUID | No | Firm membership creating the invoice |
| `updated_by_membership_id` | UUID | Yes | Firm membership last updating the invoice |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `deleted_at` | TIMESTAMPTZ | Yes | Controlled soft-deletion timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, firm_customer_id)` references `firm_customer`.
- `(firm_id, created_by_membership_id)` references the creating firm membership.
- `(firm_id, updated_by_membership_id)` references the updating firm membership.

Composite foreign keys must prevent cross-firm references.

**Invoice Statuses**

Initial supported values:

- `draft`
- `issued`
- `partially_paid`
- `paid`
- `overdue`
- `cancelled`
- `written_off`

The application must determine overdue status using the due date, outstanding balance and approved business rules.

**Constraints and Data Integrity**

- `invoice_number` must not be empty.
- Invoice number must be unique within its firm.
- `due_date` must not precede `invoice_date`.
- All monetary columns must be non-negative, except `rounding_adjustment`, which may be positive or negative.
- `amount_paid` and `balance_amount` must not be negative.
- `balance_amount` must reconcile with the invoice total and allocated payments.
- Invoice totals must reconcile with the applicable subtotal, discounts, taxes, charges and rounding adjustment.
- `currency_code` must use a valid supported currency code.
- Issued invoices must have an issue timestamp.
- Cancelled invoices must preserve cancellation evidence.
- Financial calculations must use decimal arithmetic.

**Indexes**

Proposed indexes:

- Unique index on `(firm_id, invoice_number)`.
- Index on `(firm_id, firm_customer_id, invoice_date DESC)`.
- Index on `(firm_id, invoice_status, due_date)`.
- Index on `(firm_id, due_date)`.
- Index on `(firm_id, created_at DESC)`.
- Index on `(firm_id, created_by_membership_id, created_at DESC)`.

**Tenant Isolation and RLS**

- Every invoice must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Composite foreign keys must enforce same-firm customer and membership relationships.
- Firm staff must have appropriate financial permissions to view, create, update or cancel invoices.
- Customers may view only invoices belonging to their authorized firm-customer relationship.
- Internal notes must never be exposed through customer-facing APIs.
- Platform administration must not automatically grant unrestricted access to firm financial records.

**Business and Security Rules**

- Invoice creation and issuance must follow firm authorization and approval rules.
- Issued invoices must not be silently overwritten.
- Changes to issued financial documents must follow a controlled correction, cancellation or credit-note process.
- Payment allocations must be recorded through the payment-management workflow.
- Invoice balance must be reconciled against payment allocations rather than manually adjusted without evidence.
- Customer-facing invoice information must be separated from internal notes.
- Invoice generation must preserve the applicable firm identity, invoice number, tax details and currency.
- Tax calculations must follow the applicable configured tax rules; this table is not itself a tax-calculation engine.
- Invoice creation, issuance, adjustment and cancellation must be auditable.
- Sensitive financial information must be protected by authorization and tenant isolation.

**Data Integrity and Lifecycle**

- Invoice creation must be transactional.
- Issuance must preserve a reliable snapshot of the invoice details provided to the customer.
- Payment allocation changes must reconcile the outstanding balance.
- Cancelled or issued invoices must retain historical evidence.
- Soft deletion must not erase financial or legal records that must be retained.
- Invoice numbering must be concurrency-safe and firm-specific.
- All timestamps must use UTC; invoice dates and due dates must use business dates.

**Design Notes and Open Decisions**

- The system must define firm-specific invoice numbering formats and financial-year sequences.
- The system must decide whether invoice line items require a dedicated `customer_invoice_item` table.
- The system must finalize tax breakdown storage, including CGST, SGST, IGST and other applicable components where relevant.
- The system must define whether discounts and other charges are stored at invoice-header or line-item level.
- The system must define invoice amendment, cancellation and credit-note workflows.
- The system must decide how invoice PDF snapshots are generated, stored and versioned.
- The system must define rounding precision and reconciliation rules.
- The system must determine whether multiple currencies are supported initially or only INR.
- The relationship between invoice status, payment allocations and overdue processing must be finalized.

### 3.56 `customer_payment`

**Purpose**

The `customer_payment` table stores payments received by a CA firm from its customers.

It records the payment amount, payment date, payment method, transaction reference and payment lifecycle status.

Payment receipt and invoice allocation must remain separate operations. A payment may be received before an invoice is created, may cover multiple invoices or may remain partially or completely unallocated.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Every payment must belong to one firm and one firm-customer relationship.
- Payments must never be accessible across firms.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `firm_customer_id` | UUID | No | Customer relationship making the payment |
| `payment_reference` | VARCHAR(100) | No | Firm-specific payment reference |
| `payment_date` | DATE | No | Business date payment was received |
| `payment_amount` | NUMERIC(18,2) | No | Amount received |
| `currency_code` | CHAR(3) | No | ISO currency code |
| `payment_method` | VARCHAR(30) | No | Payment method |
| `payment_status` | VARCHAR(30) | No | Payment lifecycle status |
| `transaction_reference` | VARCHAR(255) | Yes | Bank or payment-provider transaction reference |
| `payment_provider` | VARCHAR(100) | Yes | Payment provider, where applicable |
| `payment_description` | TEXT | Yes | Payment description |
| `internal_notes` | TEXT | Yes | Firm-internal payment notes |
| `received_at` | TIMESTAMPTZ | Yes | Time payment was received or confirmed |
| `recorded_by_membership_id` | UUID | No | Firm membership recording the payment |
| `updated_by_membership_id` | UUID | Yes | Firm membership last updating the record |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `reversed_at` | TIMESTAMPTZ | Yes | Payment reversal timestamp |
| `deleted_at` | TIMESTAMPTZ | Yes | Controlled soft-deletion timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, firm_customer_id)` references `firm_customer`.
- `(firm_id, recorded_by_membership_id)` references the recording firm membership.
- `(firm_id, updated_by_membership_id)` references the updating firm membership.

Composite foreign keys must prevent cross-firm references.

**Payment Methods**

Initial supported values:

- `cash`
- `bank_transfer`
- `upi`
- `credit_card`
- `debit_card`
- `cheque`
- `payment_gateway`
- `other`

**Payment Statuses**

Initial supported values:

- `pending`
- `received`
- `failed`
- `partially_reversed`
- `reversed`

The application must distinguish a payment that has been initiated from one that has actually been received or confirmed.

**Constraints and Data Integrity**

- `payment_reference` must not be empty.
- Payment reference must be unique within its firm.
- `payment_amount` must be greater than zero.
- Currency must use a supported ISO currency code.
- Payment method and status must use approved values.
- `received_at` must be populated when status is `received`.
- `reversed_at` must be populated when status is `reversed`.
- A payment must not be treated as received solely because a payment gateway request was initiated.
- Transaction references must follow appropriate uniqueness rules where reliable provider evidence exists.
- All monetary calculations must use exact decimal arithmetic.

**Indexes**

Proposed indexes:

- Unique index on `(firm_id, payment_reference)`.
- Index on `(firm_id, firm_customer_id, payment_date DESC)`.
- Index on `(firm_id, payment_status, payment_date DESC)`.
- Index on `(firm_id, transaction_reference)`.
- Index on `(firm_id, payment_method, payment_date DESC)`.
- Index on `(firm_id, created_at DESC)`.
- Index on `(firm_id, recorded_by_membership_id, created_at DESC)`.

**Tenant Isolation and RLS**

- Every payment must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Composite foreign keys must enforce same-firm customer and membership references.
- Firm staff must have appropriate financial permissions to record, view, modify or reverse payments.
- Customers may view only payments belonging to their authorized firm-customer relationship.
- Internal notes and sensitive provider details must not be exposed through customer-facing APIs.
- Platform administration must not automatically grant unrestricted access to firm payment information.

**Business and Security Rules**

- Payments must be recorded only by authorized firm personnel or trusted payment-provider integrations.
- Payment confirmation must be supported by reliable evidence.
- Gateway callbacks must be authenticated and processed idempotently.
- Duplicate payment notifications must not create duplicate payment records.
- Payment amounts must not be changed silently after confirmation.
- Corrections and reversals must preserve the original payment record and its history.
- Payment allocation must be handled through `payment_allocation`.
- A payment must not be considered fully allocated merely because it has been received.
- Advance and unallocated payments must be supported.
- Payment receipt, modification, failure and reversal must be auditable.
- Sensitive banking and transaction information must be protected.

**Data Integrity and Lifecycle**

- Payment creation and confirmation must follow a controlled transactional workflow.
- Payment reversals must preserve historical payment evidence.
- Invoice balances must be reconciled through payment allocations.
- Failed or pending payments must not reduce outstanding invoice balances.
- A payment must not be physically deleted if it represents a confirmed financial transaction.
- Reconciliation discrepancies must be identifiable and reviewable.
- All timestamps must use UTC; payment dates must use business dates.

**Design Notes and Open Decisions**

- The system must define how cash and cheque payments are confirmed.
- The system must decide whether payment gateway integration is included in the initial release.
- The system must define partial and full payment reversal workflows.
- The system must determine how refunds relate to reversed payments.
- The system must define payment reconciliation and evidence requirements.
- The system must finalize whether transaction references are unique per provider or per firm.
- The system must define handling of foreign-currency payments, if supported.
- The system must determine whether payment receipts are generated automatically.
- The system must define how unallocated and advance payments are displayed and managed.

### 3.57 `payment_allocation`

**Purpose**

The `payment_allocation` table records how a customer's received payment is applied against one or more invoices issued by the same CA firm.

It provides the financial relationship between `customer_payment` and `customer_invoice`.

The table supports partial invoice payments, payments covering multiple invoices, advance payments and controlled allocation reversals.

A payment receipt and its allocation must remain separate financial events.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Every allocation must reference a payment and invoice belonging to the same firm.
- The payment and invoice must belong to the same firm-customer relationship.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `customer_payment_id` | UUID | No | Payment being allocated |
| `customer_invoice_id` | UUID | No | Invoice receiving the allocation |
| `allocation_reference` | VARCHAR(100) | No | Firm-specific allocation reference |
| `allocated_amount` | NUMERIC(18,2) | No | Amount applied to the invoice |
| `currency_code` | CHAR(3) | No | Allocation currency |
| `allocation_status` | VARCHAR(30) | No | Allocation lifecycle status |
| `allocated_at` | TIMESTAMPTZ | No | Allocation timestamp |
| `allocated_by_membership_id` | UUID | No | Firm membership performing the allocation |
| `reversed_at` | TIMESTAMPTZ | Yes | Allocation reversal timestamp |
| `reversed_by_membership_id` | UUID | Yes | Firm membership reversing the allocation |
| `reversal_reason` | TEXT | Yes | Reason for reversing the allocation |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, customer_payment_id)` references `customer_payment`.
- `(firm_id, customer_invoice_id)` references `customer_invoice`.
- `(firm_id, allocated_by_membership_id)` references the allocating firm membership.
- `(firm_id, reversed_by_membership_id)` references the reversing firm membership.

Composite foreign keys must enforce same-firm relationships.

The application must additionally validate that the payment and invoice belong to the same `firm_customer_id`.

**Allocation Statuses**

Initial supported values:

- `active`
- `reversed`

A reversed allocation must preserve its original amount and historical evidence.

**Constraints and Data Integrity**

- `allocation_reference` must not be empty.
- Allocation reference must be unique within its firm.
- `allocated_amount` must be greater than zero.
- Currency must match the payment and invoice currencies unless an explicitly designed currency-conversion workflow is implemented.
- The payment and invoice must belong to the same firm.
- The payment and invoice must belong to the same firm-customer relationship.
- Active allocations must not exceed the confirmed available payment balance.
- Active allocations must not exceed the invoice's outstanding balance.
- Reversed allocations must have a reversal timestamp, reversing membership and reason.
- Reversed allocations must not contribute to invoice paid amounts.
- Concurrent allocation operations must be protected against over-allocation.

**Indexes**

Proposed indexes:

- Unique index on `(firm_id, allocation_reference)`.
- Index on `(firm_id, customer_payment_id, allocation_status)`.
- Index on `(firm_id, customer_invoice_id, allocation_status)`.
- Index on `(firm_id, allocated_at DESC)`.
- Index on `(firm_id, allocated_by_membership_id, allocated_at DESC)`.
- Index on `(firm_id, allocation_status, allocated_at DESC)`.

**Tenant Isolation and RLS**

- Every allocation must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Composite foreign keys must enforce same-firm payment, invoice and membership references.
- Cross-customer allocations must be rejected.
- Firm staff must have appropriate financial permissions to allocate or reverse payments.
- Customers may view only allocations associated with their authorized invoices and payments.
- Platform administration must not automatically grant unrestricted access to allocation records.

**Business and Security Rules**

- Only confirmed or received payments may be allocated.
- Pending or failed payments must not be allocated.
- A payment may be allocated across multiple invoices belonging to the same customer and firm.
- An invoice may receive allocations from multiple payments.
- Advance payments may remain unallocated until an invoice is available.
- Allocation must be performed transactionally.
- The system must recalculate or reconcile invoice balances after allocation.
- Reversals must create preserved historical evidence rather than deleting allocation records.
- Reversal must restore the corresponding available payment balance and invoice outstanding balance.
- Unauthorized allocation and reversal operations must be denied and audited.
- Financial allocation operations must use exact decimal arithmetic.

**Data Integrity and Lifecycle**

- Payment and invoice balance calculations must be derived from valid active allocations.
- Allocation operations must lock or otherwise safely coordinate the relevant payment and invoice balances to prevent concurrent over-allocation.
- Reversed allocations must remain available for financial reconciliation and audit.
- Reversal and reallocation must be treated as distinct events.
- Allocation records must not be physically deleted after financial confirmation.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must decide whether allocation changes require a separate `payment_allocation_history` table.
- The system must define how allocation reversal and subsequent reallocation are represented.
- The system must finalize the transaction-locking and concurrency-control strategy.
- The system must determine how advance payments are displayed before invoice allocation.
- The system must define whether overpayments can remain as customer credit.
- The system must finalize currency-conversion rules, if multiple currencies are supported.
- The system must define reconciliation reports for payments, invoices and allocations.
- The system must determine how credit notes and refunds affect existing allocations.

### 3.58 `customer_receipt`

**Purpose**

The `customer_receipt` table stores formal receipts issued by a CA firm to acknowledge payments received from customers.

A receipt provides evidence of payment received and must reference the corresponding payment record.

A receipt is not an invoice and must not independently change invoice balances or payment allocations.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Every receipt must belong to one firm and one firm-customer relationship.
- Receipts must never be accessible across firms.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `firm_customer_id` | UUID | No | Customer relationship making the payment |
| `customer_payment_id` | UUID | No | Payment acknowledged by the receipt |
| `receipt_number` | VARCHAR(100) | No | Firm-specific receipt reference |
| `receipt_date` | DATE | No | Receipt issue date |
| `receipt_amount` | NUMERIC(18,2) | No | Amount acknowledged |
| `currency_code` | CHAR(3) | No | ISO currency code |
| `payment_method` | VARCHAR(30) | No | Payment method recorded |
| `transaction_reference` | VARCHAR(255) | Yes | Related bank or provider transaction reference |
| `receipt_status` | VARCHAR(30) | No | Receipt lifecycle status |
| `receipt_description` | TEXT | Yes | Description of payment |
| `issued_at` | TIMESTAMPTZ | Yes | Formal receipt issuance timestamp |
| `cancelled_at` | TIMESTAMPTZ | Yes | Receipt cancellation timestamp |
| `cancellation_reason` | TEXT | Yes | Reason for cancellation |
| `issued_by_membership_id` | UUID | No | Firm membership issuing the receipt |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, firm_customer_id)` references `firm_customer`.
- `(firm_id, customer_payment_id)` references `customer_payment`.
- `(firm_id, issued_by_membership_id)` references the issuing firm membership.

Composite foreign keys must prevent cross-firm references.

The application must validate that the referenced payment belongs to the same firm-customer relationship.

**Receipt Statuses**

Initial supported values:

- `draft`
- `issued`
- `cancelled`

**Constraints and Data Integrity**

- `receipt_number` must not be empty.
- Receipt number must be unique within its firm.
- `receipt_amount` must be greater than zero.
- Currency must match the referenced payment currency.
- Receipt amount must not exceed the confirmed payment amount.
- The receipt's customer and firm must match the referenced payment.
- Issued receipts must have an issuance timestamp.
- Cancelled receipts must preserve cancellation timestamp and reason.
- Receipt amounts must use exact decimal arithmetic.
- A payment may have only one active full-payment receipt unless a separately approved partial-receipt model is introduced.

**Indexes**

Proposed indexes:

- Unique index on `(firm_id, receipt_number)`.
- Index on `(firm_id, customer_payment_id)`.
- Index on `(firm_id, firm_customer_id, receipt_date DESC)`.
- Index on `(firm_id, receipt_status, receipt_date DESC)`.
- Index on `(firm_id, issued_by_membership_id, created_at DESC)`.

**Tenant Isolation and RLS**

- Every receipt must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Composite foreign keys must enforce same-firm payment, customer and membership references.
- Firm staff must have appropriate financial permissions to create, issue, view or cancel receipts.
- Customers may view only receipts belonging to their authorized firm-customer relationship.
- Platform administration must not automatically grant unrestricted access to firm receipt records.

**Business and Security Rules**

- Receipts may be issued only for confirmed or received payments.
- Receipt issuance must be authorized and auditable.
- A receipt must accurately reflect the associated payment amount, currency and payment method.
- Receipt generation must not create a new payment or alter payment allocations.
- Issued receipts must not be silently overwritten.
- Corrections must follow a controlled cancellation and reissuance process.
- Customer-facing receipts must contain appropriate firm and payment details.
- Receipt PDFs must be stored through the private document-management system.
- Public file URLs and unrestricted storage paths must not be exposed.
- Receipt access, issuance and cancellation must be auditable.

**Data Integrity and Lifecycle**

- Receipt issuance must be transactional.
- A cancelled receipt must retain its original financial evidence.
- Cancelling a receipt must not automatically reverse the underlying payment.
- Payment reversal must follow the separate payment-reversal workflow.
- Receipt numbering must be concurrency-safe and firm-specific.
- Receipt records must be retained according to applicable financial and legal retention requirements.
- All timestamps must use UTC; receipt dates must use business dates.

**Design Notes and Open Decisions**

- The system must decide whether partial-payment receipts are supported.
- The system must determine whether one payment can generate multiple receipts.
- The system must define receipt numbering formats and financial-year sequences.
- The system must determine whether receipt PDF snapshots require a separate version-history table.
- The system must define how cancelled receipts appear in customer portals and financial reports.
- The system must finalize receipt generation and document-storage integration.
- The system must define how receipts relate to payment reversals, refunds and credit notes.

### 3.59 `credit_note`

**Purpose**

The `credit_note` table stores credit notes issued by a CA firm to reduce or adjust amounts previously invoiced to a customer.

Credit notes may be issued for billing corrections, service discounts, cancellations, overcharging or other approved financial adjustments.

A credit note must reference the original invoice and preserve the financial history of both records.

Issuing a credit note must not silently modify the original invoice.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Every credit note must belong to one firm and one firm-customer relationship.
- Credit notes must never be accessible across firms.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `firm_customer_id` | UUID | No | Customer relationship receiving the credit |
| `customer_invoice_id` | UUID | No | Original invoice being adjusted |
| `credit_note_number` | VARCHAR(100) | No | Firm-specific credit note reference |
| `credit_note_date` | DATE | No | Credit note issue date |
| `credit_note_type` | VARCHAR(30) | No | Reason or category of credit |
| `credit_note_status` | VARCHAR(30) | No | Credit note lifecycle status |
| `currency_code` | CHAR(3) | No | ISO currency code |
| `subtotal_amount` | NUMERIC(18,2) | No | Credit amount before tax |
| `tax_amount` | NUMERIC(18,2) | No | Associated tax adjustment |
| `total_credit_amount` | NUMERIC(18,2) | No | Total credit note amount |
| `reason` | TEXT | No | Explanation for issuing the credit |
| `internal_notes` | TEXT | Yes | Firm-internal notes |
| `issued_at` | TIMESTAMPTZ | Yes | Formal issuance timestamp |
| `cancelled_at` | TIMESTAMPTZ | Yes | Cancellation timestamp |
| `cancellation_reason` | TEXT | Yes | Reason for cancellation |
| `created_by_membership_id` | UUID | No | Firm membership creating the credit note |
| `approved_by_membership_id` | UUID | Yes | Firm membership approving the credit note |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, firm_customer_id)` references `firm_customer`.
- `(firm_id, customer_invoice_id)` references `customer_invoice`.
- `(firm_id, created_by_membership_id)` references the creating firm membership.
- `(firm_id, approved_by_membership_id)` references the approving firm membership.

Composite foreign keys must prevent cross-firm references.

The application must validate that the original invoice belongs to the specified firm-customer relationship.

**Credit Note Types**

Initial supported values:

- `billing_correction`
- `service_discount`
- `invoice_cancellation`
- `overcharge_adjustment`
- `other`

**Credit Note Statuses**

Initial supported values:

- `draft`
- `pending_approval`
- `approved`
- `issued`
- `cancelled`

**Constraints and Data Integrity**

- `credit_note_number` must not be empty.
- Credit note number must be unique within its firm.
- Credit amounts must be greater than zero.
- Currency must match the original invoice currency.
- Credit note amounts must use exact decimal arithmetic.
- The credit note must reference an invoice belonging to the same customer and firm.
- The total credit amount must reconcile with the subtotal and tax adjustment.
- The total credit issued against an invoice must not exceed the permitted remaining creditable amount under the approved financial rules.
- Issued credit notes must have an issuance timestamp.
- Cancelled credit notes must preserve cancellation timestamp and reason.
- Approval requirements must follow the firm's financial authorization policy.

**Indexes**

Proposed indexes:

- Unique index on `(firm_id, credit_note_number)`.
- Index on `(firm_id, customer_invoice_id, credit_note_date DESC)`.
- Index on `(firm_id, firm_customer_id, credit_note_date DESC)`.
- Index on `(firm_id, credit_note_status, credit_note_date DESC)`.
- Index on `(firm_id, created_by_membership_id, created_at DESC)`.

**Tenant Isolation and RLS**

- Every credit note must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Composite foreign keys must enforce same-firm invoice, customer and membership relationships.
- Firm staff must have appropriate financial permissions to create, approve, issue or cancel credit notes.
- Customers may view only credit notes belonging to their authorized firm-customer relationship.
- Internal notes must never be exposed through customer-facing APIs.
- Platform administration must not automatically grant unrestricted access to firm financial records.

**Business and Security Rules**

- Credit notes must be created only by authorized firm personnel.
- Approval and issuance must follow the firm's financial approval policy.
- Issued invoices must not be silently modified to reflect credit adjustments.
- Credit notes must preserve a reference to the original invoice.
- Credit notes must not be used as a substitute for recording actual payment refunds.
- The effect of a credit note on outstanding invoice balances must be calculated through the approved financial adjustment and reconciliation workflow.
- Issued credit notes must not be silently overwritten.
- Corrections must follow a controlled cancellation and reissuance process.
- Customer-facing credit notes must contain appropriate firm, invoice and adjustment details.
- Credit note PDFs must use private document storage.
- Creation, approval, issuance and cancellation must be auditable.

**Data Integrity and Lifecycle**

- Credit note creation and issuance must be transactional.
- Credit note numbering must be concurrency-safe and firm-specific.
- Issued credit notes must retain historical financial evidence.
- Cancelling a credit note must not erase its original record.
- Credit notes must not automatically create cash refunds.
- Any refund resulting from a credit must be recorded separately through the payment/refund workflow.
- Credit note records must be retained according to applicable financial and legal retention requirements.
- All timestamps must use UTC; credit note dates must use business dates.

**Design Notes and Open Decisions**

- The system must decide whether credit note line items require a separate `credit_note_item` table.
- The system must finalize the relationship between credit notes and invoice tax components.
- The system must define the creditable balance calculation when payments and previous credits exist.
- The system must define whether partial credit notes are supported.
- The system must establish approval thresholds and segregation-of-duties requirements.
- The system must determine how credit notes affect invoice outstanding balances and customer account balances.
- The system must define whether credit notes can be issued against multiple invoices.
- The system must determine PDF generation, document versioning and retention requirements.

### 3.60 `financial_adjustment`

**Purpose**

The `financial_adjustment` table stores authorized financial adjustments made to a customer's account within a CA firm.

It supports corrections and adjustments that cannot be represented directly through invoices, payments, payment allocations or credit notes.

Examples include approved opening-balance corrections, accounting corrections, write-offs and other controlled account adjustments.

Every adjustment must preserve its reason, financial effect, authorization and audit history.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Every adjustment must belong to one firm and one firm-customer relationship.
- Adjustments must never be accessible across firms.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `firm_customer_id` | UUID | No | Customer relationship affected |
| `adjustment_reference` | VARCHAR(100) | No | Firm-specific adjustment reference |
| `adjustment_type` | VARCHAR(50) | No | Type of financial adjustment |
| `adjustment_direction` | VARCHAR(20) | No | Debit or credit direction |
| `adjustment_amount` | NUMERIC(18,2) | No | Adjustment amount |
| `currency_code` | CHAR(3) | No | ISO currency code |
| `adjustment_date` | DATE | No | Business date of adjustment |
| `adjustment_status` | VARCHAR(30) | No | Adjustment lifecycle status |
| `reason` | TEXT | No | Explanation and justification |
| `reference_entity_type` | VARCHAR(100) | Yes | Related business record type |
| `reference_entity_id` | UUID | Yes | Related business record identifier |
| `internal_notes` | TEXT | Yes | Firm-internal notes |
| `created_by_membership_id` | UUID | No | Firm membership creating the adjustment |
| `approved_by_membership_id` | UUID | Yes | Firm membership approving the adjustment |
| `approved_at` | TIMESTAMPTZ | Yes | Approval timestamp |
| `posted_at` | TIMESTAMPTZ | Yes | Financial posting timestamp |
| `reversed_at` | TIMESTAMPTZ | Yes | Reversal timestamp |
| `reversal_reason` | TEXT | Yes | Explanation for reversal |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, firm_customer_id)` references `firm_customer`.
- `(firm_id, created_by_membership_id)` references the creating firm membership.
- `(firm_id, approved_by_membership_id)` references the approving firm membership.

Any related financial record must be validated to belong to the same firm and customer relationship.

Polymorphic references must use an approved entity catalogue and controlled application-level validation.

**Adjustment Types**

Initial supported values:

- `opening_balance`
- `accounting_correction`
- `write_off`
- `balance_correction`
- `other`

**Adjustment Directions**

Initial supported values:

- `debit`
- `credit`

The financial meaning and effect of each direction must be defined consistently across customer account reporting.

**Adjustment Statuses**

Initial supported values:

- `draft`
- `pending_approval`
- `approved`
- `posted`
- `reversed`
- `rejected`

**Constraints and Data Integrity**

- `adjustment_reference` must not be empty.
- Adjustment reference must be unique within its firm.
- `adjustment_amount` must be greater than zero.
- Currency must use a supported ISO currency code.
- Adjustment type, direction and status must use approved values.
- Reason must not be empty.
- Approved adjustments must have an approver and approval timestamp.
- Posted adjustments must have a posting timestamp.
- Reversed adjustments must preserve reversal timestamp and reason.
- Creator and approver must belong to the same firm.
- The system must prevent unauthorized or duplicate financial posting.

**Indexes**

Proposed indexes:

- Unique index on `(firm_id, adjustment_reference)`.
- Index on `(firm_id, firm_customer_id, adjustment_date DESC)`.
- Index on `(firm_id, adjustment_status, adjustment_date DESC)`.
- Index on `(firm_id, adjustment_type, adjustment_date DESC)`.
- Index on `(firm_id, created_by_membership_id, created_at DESC)`.
- Index on `(firm_id, reference_entity_type, reference_entity_id)`.

**Tenant Isolation and RLS**

- Every adjustment must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Composite foreign keys must enforce same-firm customer and membership relationships.
- Firm staff must have appropriate financial permissions to create, approve, post or reverse adjustments.
- Customers must not access internal adjustment notes or unauthorized financial correction details.
- Platform administration must not automatically grant unrestricted access to firm financial records.

**Business and Security Rules**

- Financial adjustments must be created only by authorized firm personnel.
- Adjustments must require a documented business reason.
- Approval requirements must follow firm-defined financial authorization thresholds.
- The creator must not approve their own adjustment where segregation-of-duties policy prohibits it.
- Only approved adjustments may be posted.
- Posted adjustments must not be silently modified or deleted.
- Corrections to posted adjustments must follow a controlled reversal and replacement process.
- Adjustment posting must be idempotent.
- Adjustments must not be used to bypass invoice, payment, credit-note or refund workflows.
- Customer account balances must be recalculated or reconciled after posting.
- All adjustment creation, approval, posting and reversal operations must be auditable.

**Data Integrity and Lifecycle**

- Adjustment creation, approval and posting must follow controlled transactional operations.
- Draft and rejected adjustments must not affect customer financial balances.
- Only valid posted adjustments may affect account balances.
- Reversal must preserve the original adjustment and its financial evidence.
- A reversal must create an appropriate opposite financial effect.
- Financial reconciliation must be able to identify the original adjustment and its reversal.
- Posted adjustment records must not be physically deleted.
- All monetary values must use exact decimal arithmetic.
- All timestamps must use UTC; adjustment dates must use business dates.

**Design Notes and Open Decisions**

- The system must define the precise debit and credit effects on customer account balances.
- The system must determine whether adjustments require a dedicated double-entry ledger model.
- The system must finalize approval thresholds and segregation-of-duties rules.
- The system must define opening-balance migration and verification procedures.
- The system must determine how adjustments interact with invoices, payments, credit notes and refunds.
- The system must decide whether financial adjustment line items are required.
- The system must define reversal and replacement workflows.
- The system must determine whether a separate immutable financial journal is required as the authoritative accounting record.

### 3.61 `financial_transaction`

**Purpose**

The `financial_transaction` table records financial movements affecting a customer's account within a CA firm.

It provides a centralized, auditable record of posted financial activity originating from supported business operations, including invoices, payments, credit notes, financial adjustments and refunds.

This table is intended to support customer account statements, financial reconciliation, transaction history and balance calculations.

It must not become an independent mechanism for creating financial effects that bypass the underlying business records.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Every transaction must belong to one firm and one firm-customer relationship.
- Financial transactions must never be accessible across firms.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `firm_customer_id` | UUID | No | Customer relationship affected |
| `transaction_reference` | VARCHAR(100) | No | Firm-specific transaction reference |
| `transaction_type` | VARCHAR(50) | No | Originating financial transaction type |
| `transaction_direction` | VARCHAR(20) | No | Debit or credit direction |
| `transaction_amount` | NUMERIC(18,2) | No | Posted transaction amount |
| `currency_code` | CHAR(3) | No | ISO currency code |
| `transaction_date` | DATE | No | Financial business date |
| `transaction_status` | VARCHAR(30) | No | Transaction status |
| `source_entity_type` | VARCHAR(100) | No | Type of originating business record |
| `source_entity_id` | UUID | No | Identifier of originating business record |
| `description` | TEXT | No | Transaction description |
| `correlation_id` | UUID | Yes | Identifier linking related operations |
| `posted_at` | TIMESTAMPTZ | No | Financial posting timestamp |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `created_by_membership_id` | UUID | Yes | Firm membership responsible for the operation |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, firm_customer_id)` references `firm_customer`.
- `(firm_id, created_by_membership_id)` references the responsible firm membership when applicable.

The originating business record must be validated against an approved entity catalogue.

The source record must belong to the same firm and customer relationship. Because source references are polymorphic, database-level referential integrity requires an additional design decision.

**Transaction Types**

Initial supported values:

- `invoice_issued`
- `payment_received`
- `credit_note_issued`
- `financial_adjustment_posted`
- `refund_processed`
- `payment_reversed`
- `credit_note_reversed`
- `adjustment_reversed`
- `other`

**Transaction Directions**

Initial supported values:

- `debit`
- `credit`

The financial effect of each transaction type and direction must be defined consistently in the customer account ledger rules.

**Transaction Statuses**

Initial supported values:

- `posted`
- `reversed`

Only posted transactions may contribute to customer account balance calculations. Reversal records must preserve the original transaction and its relationship to the reversing transaction.

**Constraints and Data Integrity**

- `transaction_reference` must not be empty.
- Transaction reference must be unique within its firm.
- Transaction amount must be greater than zero.
- Currency must use a supported ISO currency code.
- Transaction type, direction and status must use approved values.
- Source entity type and identifier must be present.
- Posted timestamp must be present.
- Source records must be validated before transaction creation.
- Duplicate posting of the same business event must be prevented through idempotency controls.
- Reversal transactions must be linked to the original transaction through a relationship to be finalized in the physical schema.

**Indexes**

Proposed indexes:

- Unique index on `(firm_id, transaction_reference)`.
- Index on `(firm_id, firm_customer_id, transaction_date DESC)`.
- Index on `(firm_id, transaction_type, transaction_date DESC)`.
- Index on `(firm_id, source_entity_type, source_entity_id)`.
- Index on `(firm_id, transaction_status, transaction_date DESC)`.
- Index on `(firm_id, correlation_id)`.

**Tenant Isolation and RLS**

- Every transaction must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Composite foreign keys must enforce same-firm customer and membership relationships.
- Firm financial permissions must govern transaction viewing and reporting.
- Customers must only see transactions permitted by the customer-facing financial access policy.
- Platform administration must not automatically grant unrestricted access to firm financial transactions.

**Business and Security Rules**

- Transactions must be generated by authorized financial business operations.
- A transaction must not be posted independently without a valid originating business event.
- Posting must be idempotent and transactionally consistent with the originating business operation.
- Posted transaction records must not be silently edited or deleted.
- Corrections must use controlled reversal and replacement transactions.
- Reversal operations must preserve the original transaction and its audit evidence.
- Transaction amounts must not be used as manually editable customer balances.
- Financial reports must use consistent debit and credit conventions.
- Sensitive payment or tax information must not be copied unnecessarily into transaction descriptions.
- All financial posting and reversal operations must be auditable.

**Data Integrity and Lifecycle**

- The originating business operation and its financial transaction must be committed atomically where possible.
- Failed business operations must not leave partially posted financial transactions.
- Transaction history must remain available for reconciliation and audit.
- Reversal records must provide a traceable link to the original transaction.
- Financial balances must be calculated or reconciled from the authoritative financial records.
- All monetary values must use exact decimal arithmetic.
- All timestamps must use UTC; transaction dates must use business dates.

**Design Notes and Open Decisions**

- The system must determine whether this table represents a customer subledger or a complete double-entry accounting journal.
- If double-entry accounting is required, a separate journal header and journal line model may be more appropriate.
- The system must define debit and credit conventions for each transaction type.
- The system must decide whether one source business event can generate multiple transaction records.
- The system must define the relationship between this table and payment allocations.
- The system must finalize reversal relationships and prevent duplicate reversal.
- The system must determine whether transaction amounts are stored as one-sided positive amounts or signed amounts.
- The system must define the authoritative source for customer balances and account statements.
- The system must determine whether immutable transaction records require a separate external audit or ledger mechanism.

### 3.62 `customer_refund`

**Purpose**

The `customer_refund` table records refunds issued by a CA firm to a customer.

It supports refunds arising from excess payments, cancelled services, approved credit balances, payment reversals or other authorized refund situations.

A refund must remain independently traceable to its originating payment, customer account and approval process.

Recording a refund must not silently modify or delete the original payment or its allocation history.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Every refund must belong to one firm and one firm-customer relationship.
- Refund records must never be accessible across firms.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `firm_customer_id` | UUID | No | Customer relationship receiving the refund |
| `customer_payment_id` | UUID | Yes | Original payment, where applicable |
| `refund_reference` | VARCHAR(100) | No | Firm-specific refund reference |
| `refund_type` | VARCHAR(40) | No | Reason or category of refund |
| `refund_amount` | NUMERIC(18,2) | No | Amount to be refunded |
| `currency_code` | CHAR(3) | No | ISO currency code |
| `refund_method` | VARCHAR(40) | No | Method used to issue the refund |
| `refund_status` | VARCHAR(30) | No | Refund lifecycle status |
| `refund_reason` | TEXT | No | Explanation for the refund |
| `transaction_reference` | VARCHAR(255) | Yes | Bank, gateway or payment-provider reference |
| `requested_at` | TIMESTAMPTZ | No | Refund request timestamp |
| `approved_by_membership_id` | UUID | Yes | Firm membership approving the refund |
| `approved_at` | TIMESTAMPTZ | Yes | Approval timestamp |
| `processed_by_membership_id` | UUID | Yes | Firm membership processing the refund |
| `processed_at` | TIMESTAMPTZ | Yes | Processing timestamp |
| `failed_at` | TIMESTAMPTZ | Yes | Failure timestamp |
| `failure_reason` | TEXT | Yes | Explanation for processing failure |
| `cancelled_at` | TIMESTAMPTZ | Yes | Cancellation timestamp |
| `cancellation_reason` | TEXT | Yes | Explanation for cancellation |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `created_by_membership_id` | UUID | No | Firm membership creating the refund request |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, firm_customer_id)` references `firm_customer`.
- `(firm_id, customer_payment_id)` references `customer_payment` when an original payment is specified.
- `(firm_id, approved_by_membership_id)` references `firm_membership` when an approver is specified.
- `(firm_id, processed_by_membership_id)` references `firm_membership` when a processor is specified.
- `(firm_id, created_by_membership_id)` references `firm_membership`.

When an original payment is specified, it must belong to the same firm and customer relationship as the refund.

**Refund Types**

Initial supported values:

- `excess_payment`
- `service_cancellation`
- `credit_balance`
- `payment_reversal`
- `other`

**Refund Methods**

Initial supported values:

- `original_payment_method`
- `bank_transfer`
- `upi`
- `cash`
- `cheque`
- `payment_gateway`
- `other`

**Refund Statuses**

Initial supported values:

- `requested`
- `pending_approval`
- `approved`
- `processing`
- `processed`
- `failed`
- `cancelled`

**Constraints and Data Integrity**

- `refund_reference` must not be empty.
- Refund reference must be unique within its firm.
- `refund_amount` must be greater than zero.
- Currency must use a supported ISO currency code.
- Refund type, method and status must use approved values.
- Refund reason must not be empty.
- Approved refunds must have an approver and approval timestamp.
- Processed refunds must have a processor and processing timestamp.
- Failed refunds must preserve failure details.
- Cancelled refunds must preserve cancellation details.
- Refund amount must not exceed the eligible refundable amount.
- Concurrent refund requests must not cause the refundable balance to be exceeded.
- Duplicate processing of the same refund must be prevented through idempotency controls.

**Indexes**

Proposed indexes:

- Unique index on `(firm_id, refund_reference)`.
- Index on `(firm_id, firm_customer_id, requested_at DESC)`.
- Index on `(firm_id, customer_payment_id)`.
- Index on `(firm_id, refund_status, requested_at DESC)`.
- Index on `(firm_id, refund_type, requested_at DESC)`.
- Index on `(firm_id, transaction_reference)`.
- Index on `(firm_id, created_by_membership_id, created_at DESC)`.

**Tenant Isolation and RLS**

- Every refund must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Composite foreign keys must enforce same-firm customer, payment and membership relationships.
- Firm financial permissions must govern refund creation, approval, processing and viewing.
- Customers must only see refund information permitted by the customer-facing financial access policy.
- Platform administration must not automatically grant unrestricted access to firm refund records.

**Business and Security Rules**

- Refunds may only be requested by authorized firm personnel or through an explicitly approved customer request workflow.
- Every refund must have a documented reason.
- Refund approval must follow firm-defined authorization thresholds.
- The creator must not approve their own refund where segregation-of-duties policy prohibits it.
- Only approved refunds may enter processing.
- A refund must not be marked processed merely because a payment provider accepted a request; successful completion must be established according to the selected refund method.
- Failed refunds must remain visible and must not silently reduce customer balances.
- A refund must not directly delete or alter the original payment.
- Refunds must be reconciled against the relevant payment, customer account and financial transaction records.
- All refund lifecycle operations must be auditable.

**Data Integrity and Lifecycle**

- Refund request, approval and processing must follow controlled transactional operations.
- Refundable balance calculations must account for prior successful refunds and pending operations where necessary.
- Only successfully processed refunds may create the corresponding final financial effect.
- Failed or cancelled refunds must not be treated as completed refunds.
- A processed refund must not be silently edited or deleted.
- Corrections must use controlled reversal or adjustment procedures.
- Refund records must remain available for financial reconciliation and audit.
- All monetary values must use exact decimal arithmetic.
- All timestamps must use UTC; refund dates must use business dates.

**Design Notes and Open Decisions**

- The system must define whether refunds are always linked to an original payment or may also originate from a customer credit balance.
- The system must determine whether partial refunds and multiple refunds against one payment are supported.
- The system must define how pending refund requests reserve refundable balances.
- The system must determine how payment-provider refund callbacks and idempotency are handled.
- The system must define how refunds interact with payment reversals, credit notes and financial adjustments.
- The system must determine whether a separate refund attempt/history table is required.
- The system must define the authoritative financial transaction generated by a successful refund.
- The system must finalize approval thresholds and segregation-of-duties rules.
- The system must define refund reconciliation and customer notification requirements.

### 3.63 `refund_tracking`

**Purpose**

The `refund_tracking` table records the operational tracking history and processing progress of customer refunds.

It provides visibility into refund processing stages, payment-provider updates, bank processing references and reconciliation outcomes.

The table complements `customer_refund` and must not independently create, modify or reverse financial balances.

A refund may have multiple tracking records as it progresses through internal processing and external payment systems.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Every tracking record must belong to one firm and one customer refund.
- Tracking information must never be accessible across firms.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `customer_refund_id` | UUID | No | Related refund |
| `tracking_reference` | VARCHAR(150) | No | Internal tracking reference |
| `tracking_stage` | VARCHAR(50) | No | Processing stage |
| `tracking_status` | VARCHAR(30) | No | Status of the tracking event |
| `provider_name` | VARCHAR(100) | Yes | Payment provider or financial institution |
| `provider_reference` | VARCHAR(255) | Yes | External processing reference |
| `provider_event_id` | VARCHAR(255) | Yes | External event identifier, where available |
| `event_description` | TEXT | Yes | Description of the processing update |
| `event_data` | JSONB | No | Validated non-sensitive event metadata |
| `occurred_at` | TIMESTAMPTZ | No | Time the event occurred |
| `recorded_at` | TIMESTAMPTZ | No | Time the event was recorded |
| `recorded_by_membership_id` | UUID | Yes | Firm membership recording a manual update |
| `correlation_id` | UUID | Yes | Identifier connecting related processing operations |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, customer_refund_id)` references `customer_refund`.
- `(firm_id, recorded_by_membership_id)` references `firm_membership` when a staff member records the event.

System-generated provider events may have no staff membership reference.

Each tracking record must belong to the same firm as its parent refund.

**Tracking Stages**

Initial supported values:

- `refund_requested`
- `approval_completed`
- `processing_started`
- `provider_submitted`
- `provider_accepted`
- `bank_processing`
- `refund_completed`
- `processing_failed`
- `refund_cancelled`
- `reconciliation_completed`
- `manual_update`

**Tracking Statuses**

Initial supported values:

- `pending`
- `in_progress`
- `completed`
- `failed`
- `cancelled`
- `information_required`

**Constraints and Data Integrity**

- Tracking reference must not be empty.
- Tracking stage and status must use approved values.
- Event data must contain valid JSON.
- Provider event identifiers must be unique within the appropriate provider and firm scope when supplied.
- Provider references must be stored without exposing credentials or sensitive payment information.
- Recorded timestamp must not precede the actual event timestamp without an explicitly permitted correction workflow.
- Staff-recorded events must reference a membership belonging to the same firm.
- Duplicate provider events must be detected and handled idempotently.

**Indexes**

Proposed indexes:

- Index on `(firm_id, customer_refund_id, occurred_at DESC)`.
- Index on `(firm_id, tracking_stage, occurred_at DESC)`.
- Index on `(firm_id, tracking_status, occurred_at DESC)`.
- Index on `(firm_id, provider_reference)`.
- Unique or appropriately scoped index on `(firm_id, provider_name, provider_event_id)` where provider event identifiers are available.
- Index on `(firm_id, correlation_id)`.
- Index on `(firm_id, recorded_by_membership_id, recorded_at DESC)`.

**Tenant Isolation and RLS**

- Every tracking record must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Composite foreign keys must enforce same-firm refund and membership relationships.
- Only authorized firm personnel and approved backend services may create or retrieve tracking records.
- Customers may see only the approved refund progress information intended for the customer portal.
- Internal provider responses and operational notes must not automatically be exposed to customers.
- Platform administration must not automatically grant unrestricted access to firm refund tracking information.

**Business and Security Rules**

- Tracking events must be generated by authorized backend operations, verified provider callbacks or authorized staff actions.
- External provider callbacks must be authenticated and validated before being recorded.
- Provider event processing must be idempotent.
- A provider's acceptance of a refund request must not automatically be interpreted as successful refund completion.
- Tracking records must not directly update financial balances.
- Financial status changes must be performed through the authorized `customer_refund` lifecycle.
- Tracking events must preserve the source of each update.
- Manual updates must identify the responsible staff member.
- Provider response payloads must be sanitized and must not contain credentials, access tokens or unnecessary sensitive customer data.
- All material tracking updates must be auditable.

**Data Integrity and Lifecycle**

- Tracking records should be append-only.
- Corrections must be recorded as additional events rather than silently overwriting previous tracking history.
- Duplicate external events must not produce duplicate processing effects.
- Tracking history must remain available for refund investigation and reconciliation.
- Failed provider callbacks must be retried or escalated through controlled background processing.
- The latest tracking state must be consistent with the authoritative refund lifecycle status.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must decide whether every provider callback creates a separate tracking record.
- The system must define the provider callback authentication and verification mechanism.
- The system must determine whether raw provider payloads are retained or only sanitized metadata.
- The system must define the authoritative mapping between tracking stages and refund statuses.
- The system must decide how late, duplicate and out-of-order provider events are handled.
- The system must define whether reconciliation outcomes require a separate reconciliation table.
- The system must determine which tracking events are visible to customers.
- The system must define retention and archival requirements for provider event history.

### 3.64 `refund_follow_up`

**Purpose**

The `refund_follow_up` table records follow-up activities required to investigate, resolve or monitor customer refund requests.

It supports follow-up work for delayed refunds, failed processing, missing provider updates, bank reconciliation issues and customer enquiries.

Each follow-up must be associated with a refund and retain its assignment, due date, status and resolution history.

This table manages operational tasks only. It must not independently modify refund amounts, payment records or customer financial balances.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Every follow-up must belong to one firm and one customer refund.
- Follow-up records must never be accessible across firms.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm and tenant identifier |
| `customer_refund_id` | UUID | No | Related customer refund |
| `follow_up_reference` | VARCHAR(100) | No | Firm-specific follow-up reference |
| `follow_up_type` | VARCHAR(50) | No | Reason or category of follow-up |
| `follow_up_status` | VARCHAR(30) | No | Follow-up lifecycle status |
| `priority` | VARCHAR(20) | No | Follow-up priority |
| `subject` | VARCHAR(255) | No | Follow-up subject |
| `description` | TEXT | No | Details of required follow-up |
| `assigned_to_membership_id` | UUID | Yes | Staff member responsible |
| `created_by_membership_id` | UUID | No | Staff member creating the follow-up |
| `due_at` | TIMESTAMPTZ | Yes | Follow-up deadline |
| `next_action_at` | TIMESTAMPTZ | Yes | Planned next action time |
| `completed_at` | TIMESTAMPTZ | Yes | Completion timestamp |
| `resolution_summary` | TEXT | Yes | Summary of the outcome |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `deleted_at` | TIMESTAMPTZ | Yes | Controlled soft-deletion timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, customer_refund_id)` references `customer_refund`.
- `(firm_id, assigned_to_membership_id)` references `firm_membership` when an assignee is specified.
- `(firm_id, created_by_membership_id)` references `firm_membership`.

Assigned staff must be active and authorized to perform refund follow-up work within the relevant firm.

**Follow-Up Types**

Initial supported values:

- `processing_delay`
- `provider_confirmation`
- `bank_reconciliation`
- `failed_refund`
- `customer_enquiry`
- `documentation_required`
- `other`

**Follow-Up Statuses**

Initial supported values:

- `open`
- `in_progress`
- `waiting_for_provider`
- `waiting_for_customer`
- `resolved`
- `cancelled`

**Priority Levels**

Initial supported values:

- `low`
- `normal`
- `high`
- `urgent`

**Constraints and Data Integrity**

- Follow-up reference must not be empty.
- Follow-up reference must be unique within its firm.
- Subject and description must not be empty.
- Follow-up type, status and priority must use approved values.
- A resolved follow-up must have a completion timestamp and resolution summary.
- An open or in-progress follow-up must not have a completion timestamp.
- Assigned staff and creator must belong to the same firm.
- A follow-up must reference an existing refund belonging to the same firm.

**Indexes**

Proposed indexes:

- Unique index on `(firm_id, follow_up_reference)`.
- Index on `(firm_id, customer_refund_id, created_at DESC)`.
- Index on `(firm_id, assigned_to_membership_id, follow_up_status, due_at)`.
- Index on `(firm_id, follow_up_status, due_at)`.
- Index on `(firm_id, priority, due_at)`.
- Index on `(firm_id, created_by_membership_id, created_at DESC)`.

**Tenant Isolation and RLS**

- Every follow-up must contain `firm_id`.
- RLS must restrict access to the current tenant.
- Composite foreign keys must enforce same-firm refund and membership relationships.
- Only authorized firm personnel may create, assign, update or resolve follow-ups.
- Customer-facing users must not access internal follow-up notes or staff assignments.
- Platform administration must not automatically grant unrestricted access to firm follow-up records.

**Business and Security Rules**

- Follow-ups may be created manually or generated by approved refund monitoring processes.
- Every follow-up must have a clear business purpose.
- Assignment must be restricted to authorized firm staff.
- Reassignment must preserve the previous assignment history through an appropriate history mechanism.
- Due dates and next-action times must support operational monitoring.
- Overdue follow-ups must be identifiable for staff dashboards and reporting.
- Resolution must include a meaningful outcome.
- Cancelling a follow-up must not cancel the underlying refund.
- Completing a follow-up must not automatically mark a refund as processed.
- All material follow-up operations must be auditable.

**Data Integrity and Lifecycle**

- Follow-up creation and assignment must be recorded transactionally.
- Changes to status, priority, assignee and due date must be traceable.
- Resolved follow-ups must retain their resolution details.
- Reopened follow-ups must preserve previous completion history through an appropriate event or history record.
- Follow-ups must not be physically deleted where retention or audit obligations require preservation.
- Refund closure must trigger a review of any remaining open follow-ups.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must determine whether follow-up assignment history requires a dedicated table.
- The system must define whether multiple follow-ups may remain open for one refund.
- The system must establish default due dates and escalation rules.
- The system must define whether overdue follow-ups generate notifications.
- The system must decide how automatic follow-ups are generated from refund tracking events.
- The system must define follow-up reopening and cancellation rules.
- The system must determine whether follow-up comments and attachments require separate tables.
- The system must define retention and reporting requirements for completed follow-ups.

### 3.65 `subscription_plan`

**Purpose**

The `subscription_plan` table defines the subscription plans offered by the ITR Management System SaaS platform to CA firms.

It stores plan identity, pricing, billing intervals, availability and lifecycle information.

Examples may include:

- Starter
- Professional
- Business
- Enterprise

The table must support future pricing and plan changes without requiring application code or database schema changes for every new plan.

Subscription plans define commercial entitlements. They must not independently grant application permissions or bypass security controls.

**Ownership and Tenant Scope**

- Ownership: Platform-Owned.
- Tenant scope: Platform-level.
- No `firm_id` column.
- No firm-level RLS policy.
- Access restricted to authorized platform administrators and approved backend services.
- Firm administrators may view only the plan information and pricing made available to them.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `plan_code` | VARCHAR(50) | No | Unique stable plan identifier |
| `plan_name` | VARCHAR(150) | No | Display name of the subscription plan |
| `plan_description` | TEXT | Yes | Description of the plan |
| `billing_interval` | VARCHAR(20) | No | Billing frequency |
| `billing_interval_count` | INTEGER | No | Number of billing intervals |
| `base_price` | NUMERIC(18,2) | No | Base subscription price |
| `currency_code` | CHAR(3) | No | ISO currency code |
| `trial_period_days` | INTEGER | No | Default trial duration |
| `plan_status` | VARCHAR(30) | No | Plan lifecycle status |
| `is_public` | BOOLEAN | No | Whether the plan is publicly available for selection |
| `display_order` | INTEGER | No | Order in which the plan is displayed |
| `effective_from` | TIMESTAMPTZ | Yes | Date and time the plan becomes effective |
| `effective_until` | TIMESTAMPTZ | Yes | Date and time the plan stops being effective |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `created_by_platform_user_id` | UUID | Yes | Platform administrator creating the plan |
| `updated_by_platform_user_id` | UUID | Yes | Platform administrator updating the plan |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `created_by_platform_user_id` references `platform_user.id`.
- `updated_by_platform_user_id` references `platform_user.id`.

**Billing Intervals**

Initial supported values:

- `monthly`
- `quarterly`
- `half_yearly`
- `yearly`
- `one_time`

The billing interval count must be a positive integer.

**Plan Statuses**

Initial supported values:

- `draft`
- `active`
- `inactive`
- `deprecated`
- `archived`

Only active plans within their effective dates may be offered for new subscriptions.

Existing subscriptions must retain their historical plan and pricing information when a plan is changed or deprecated.

**Constraints and Data Integrity**

- `plan_code` must not be empty.
- `plan_code` must be unique.
- `plan_name` must not be empty.
- Base price must be zero or greater.
- Billing interval count must be greater than zero.
- Trial period days must be zero or greater.
- Display order must be zero or greater.
- Currency must use a supported ISO currency code.
- Effective end date must be later than effective start date when both are provided.
- Plan status must use an approved value.
- One-time plans must follow separately defined billing and renewal rules.

**Indexes**

Proposed indexes:

- Unique index on `plan_code`.
- Index on `(plan_status, is_public)`.
- Index on `(display_order, plan_status)`.
- Index on `(effective_from, effective_until)`.

**Business and Security Rules**

- Only authorized platform administrators may create or modify subscription plans.
- Firm administrators must not be able to modify platform plan definitions.
- Published plan changes must not silently alter the agreed commercial terms of existing subscriptions.
- Pricing changes must preserve historical subscription pricing.
- Deprecated plans must remain available for historical reporting.
- Subscription plans must not directly grant application permissions.
- Feature availability must be determined through the subscription entitlement model and backend authorization.
- Promotional pricing, discounts and taxes must follow separately controlled billing rules.
- All plan creation, modification, activation and deprecation operations must be audited.

**Data Integrity and Lifecycle**

- Draft plans may be modified before publication.
- Published plan changes must preserve historical pricing and subscription references.
- Existing subscriptions must retain their agreed plan version or pricing snapshot.
- Plans must not be physically deleted when referenced by subscriptions or billing records.
- Plan deprecation must not automatically cancel existing subscriptions.
- All financial amounts must use exact decimal arithmetic.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must define the initial subscription tiers and their commercial pricing.
- The system must decide whether plan prices include or exclude applicable taxes.
- The system must determine whether pricing requires a separate versioned plan-pricing table.
- The system must define trial eligibility and trial extension rules.
- The system must determine how discounts, coupons and promotional pricing are represented.
- The system must define currency support and whether multi-currency billing is required.
- The system must determine whether one-time plans are needed at launch.
- The system must define plan upgrade, downgrade and renewal pricing behavior.
- The system must ensure historical subscriptions retain their original commercial terms.

### 3.66 `plan_feature`

**Purpose**

The `plan_feature` table defines the features, capabilities and usage entitlements associated with each SaaS subscription plan.

It allows the platform to control which application features are available to firms under different subscription plans.

Examples include:

- Maximum number of firm staff members.
- Maximum number of customers.
- Maximum document storage.
- Assessment-year availability.
- Advanced reporting.
- Excel import functionality.
- Customer portal access.
- Automated notifications.
- Financial reporting.
- API and integration access.

Feature entitlements must be evaluated by the backend. Hiding a feature in the user interface must never be treated as an authorization control.

**Ownership and Tenant Scope**

- Ownership: Platform-Owned.
- Tenant scope: Platform-level.
- No `firm_id` column.
- No firm-level RLS policy.
- Access restricted to authorized platform administrators and backend services.
- Firm administrators may view the entitlements applicable to their own subscription.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `subscription_plan_id` | UUID | No | Associated subscription plan |
| `feature_key` | VARCHAR(100) | No | Stable feature identifier |
| `feature_name` | VARCHAR(150) | No | Human-readable feature name |
| `feature_description` | TEXT | Yes | Description of the feature |
| `is_included` | BOOLEAN | No | Whether the feature is included |
| `limit_type` | VARCHAR(30) | No | Type of entitlement limit |
| `limit_value` | NUMERIC(18,2) | Yes | Configured feature limit |
| `limit_unit` | VARCHAR(30) | Yes | Unit associated with the limit |
| `feature_configuration` | JSONB | No | Additional validated feature configuration |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `created_by_platform_user_id` | UUID | Yes | Platform administrator creating the record |
| `updated_by_platform_user_id` | UUID | Yes | Platform administrator updating the record |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `subscription_plan_id` references `subscription_plan.id`.
- `created_by_platform_user_id` references `platform_user.id`.
- `updated_by_platform_user_id` references `platform_user.id`.

A feature key must identify a consistent application capability across all plans.

**Limit Types**

Initial supported values:

- `none` — No numerical limit applies.
- `unlimited` — The feature has no configured usage ceiling.
- `fixed` — A specific numerical limit applies.
- `boolean` — The feature is either included or excluded.
- `configurable` — The feature uses a validated configuration rather than a single numerical limit.

**Examples of Feature Entitlements**

| Feature Key | Example Entitlement | Example Limit |
|---|---|---|
| `staff_users` | Maximum firm staff accounts | 5 |
| `customers` | Maximum managed customers | 500 |
| `document_storage` | Document storage capacity | 50 GB |
| `assessment_years` | Enabled assessment years | 3 |
| `excel_import` | Excel import functionality | Included |
| `advanced_reporting` | Advanced reports | Included |
| `customer_portal` | Customer portal | Included |
| `api_access` | API access | Excluded |
| `automated_notifications` | Automated notifications | 1,000 per month |

These are illustrative values only. Actual commercial limits must be configured through approved platform administration.

**Constraints and Data Integrity**

- Each feature must belong to an existing subscription plan.
- Feature key must not be empty.
- Feature name must not be empty.
- Feature key must be unique within a subscription plan.
- `limit_value` must not be negative when supplied.
- Limit type must use an approved value.
- Fixed limits must have a valid limit value and unit where applicable.
- Boolean entitlements must use the inclusion flag consistently.
- Unlimited entitlements must not be interpreted as zero.
- Feature configuration must contain a valid JSON object.
- Feature configuration must be validated against the approved feature schema.
- Creator and updater references must point to valid platform administrators.

**Indexes**

Proposed indexes:

- Unique index on `(subscription_plan_id, feature_key)`.
- Index on `(feature_key, is_included)`.
- Index on `(subscription_plan_id, is_included)`.

**Business and Security Rules**

- Only authorized platform administrators may define or modify plan entitlements.
- Firms must not modify their own plan feature definitions.
- Backend authorization must verify feature availability before executing restricted operations.
- Usage limits must be enforced by backend services, not only by the frontend.
- A disabled feature must remain unavailable even if a client attempts to invoke its API directly.
- A global platform feature flag may disable a feature even when the subscription includes it.
- A subscription entitlement must not override platform security restrictions or role-based permissions.
- Usage calculations must use consistent definitions and avoid counting the same operation multiple times.
- Changes to published plan entitlements must preserve historical subscription agreements through versioning or subscription snapshots.
- All entitlement changes must be audited.

**Data Integrity and Lifecycle**

- Feature definitions must use stable feature keys.
- Removing a feature from a new plan version must not erase historical entitlement information.
- Plan entitlement changes must follow controlled publication procedures.
- Usage limits must be reconciled with actual usage records.
- Entitlement evaluation must be deterministic and consistent across backend services.
- Cached entitlement changes must trigger appropriate cache invalidation.
- Archived subscription plans must retain their historical feature definitions.

**Design Notes and Open Decisions**

- The system must define the complete feature catalogue and stable feature keys.
- The system must determine whether feature definitions should be separated into a global `feature_catalogue` table.
- The system must define usage measurement and reset periods for each limited feature.
- The system must decide how storage limits, customer limits and staff limits are calculated.
- The system must define whether usage limits are hard restrictions or configurable warning thresholds.
- The system must determine how plan upgrades and downgrades affect existing usage.
- The system must define the precedence between platform feature flags, subscription entitlements and firm-specific configuration.
- The system must determine whether published entitlements require a separate version history table.
- The system must ensure that historical subscriptions retain their agreed entitlement terms.

### 3.67 `firm_subscription`

**Purpose**

The `firm_subscription` table manages the subscription relationship between a CA firm and the ITR Management System SaaS platform.

It records the subscription plan selected by a firm, its commercial terms, billing period, activation status and subscription lifecycle.

A firm may have multiple historical subscription records, but only the permitted number of simultaneously active subscriptions may be maintained according to platform policy.

The table must preserve historical subscription terms when a firm upgrades, downgrades, renews or changes plans.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned, managed through platform subscription administration.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Each subscription belongs to one CA firm.
- Platform administrators may perform authorized subscription management operations.
- Firm administrators may view their own subscription information but must not modify platform-controlled billing status or commercial terms.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Subscribing CA firm |
| `subscription_plan_id` | UUID | No | Selected subscription plan |
| `subscription_reference` | VARCHAR(100) | No | Unique subscription reference |
| `subscription_status` | VARCHAR(30) | No | Current subscription lifecycle status |
| `billing_interval` | VARCHAR(20) | No | Agreed billing frequency |
| `billing_interval_count` | INTEGER | No | Number of billing intervals |
| `agreed_price` | NUMERIC(18,2) | No | Agreed subscription price |
| `currency_code` | CHAR(3) | No | ISO currency code |
| `trial_started_at` | TIMESTAMPTZ | Yes | Trial start timestamp |
| `trial_ends_at` | TIMESTAMPTZ | Yes | Trial end timestamp |
| `subscription_started_at` | TIMESTAMPTZ | Yes | Subscription activation timestamp |
| `current_period_start` | TIMESTAMPTZ | Yes | Current billing period start |
| `current_period_end` | TIMESTAMPTZ | Yes | Current billing period end |
| `next_billing_at` | TIMESTAMPTZ | Yes | Next scheduled billing timestamp |
| `cancel_at_period_end` | BOOLEAN | No | Whether cancellation is scheduled at period end |
| `cancelled_at` | TIMESTAMPTZ | Yes | Cancellation timestamp |
| `cancellation_reason` | TEXT | Yes | Reason for cancellation |
| `ended_at` | TIMESTAMPTZ | Yes | Subscription end timestamp |
| `previous_subscription_id` | UUID | Yes | Previous subscription when replaced |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `created_by_platform_user_id` | UUID | Yes | Platform administrator creating the subscription |
| `updated_by_platform_user_id` | UUID | Yes | Platform administrator updating the subscription |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`.
- `subscription_plan_id` references `subscription_plan.id`.
- `previous_subscription_id` references `firm_subscription.id`.
- `created_by_platform_user_id` references `platform_user.id`.
- `updated_by_platform_user_id` references `platform_user.id`.

Subscription replacement and plan-change relationships must preserve the original subscription history.

**Subscription Statuses**

Initial supported values:

- `pending`
- `trialing`
- `active`
- `past_due`
- `suspended`
- `cancelled`
- `expired`
- `terminated`

**Billing Intervals**

Initial supported values:

- `monthly`
- `quarterly`
- `half_yearly`
- `yearly`
- `one_time`

The agreed interval and price must be preserved even if the underlying subscription plan changes later.

**Constraints and Data Integrity**

- Subscription reference must not be empty.
- Subscription reference must be unique within the firm.
- Agreed price must be zero or greater.
- Billing interval count must be positive.
- Currency must use a supported ISO currency code.
- Current billing period end must be later than its start when both are supplied.
- Trial end must be later than trial start when both are supplied.
- Cancellation timestamps and end timestamps must be consistent with the subscription lifecycle.
- A previous subscription reference must not point to the same record.
- A firm must not have overlapping active subscriptions unless explicitly permitted by platform policy.
- Subscription changes must not overwrite historical commercial terms.

**Indexes**

Proposed indexes:

- Unique index on `(firm_id, subscription_reference)`.
- Index on `(firm_id, subscription_status)`.
- Index on `(subscription_plan_id, subscription_status)`.
- Index on `(subscription_status, current_period_end)`.
- Index on `(subscription_status, next_billing_at)`.
- Index on `(firm_id, created_at DESC)`.
- Index on `previous_subscription_id`.

A suitable partial unique index or controlled subscription lifecycle mechanism must enforce the permitted number of current subscriptions per firm.

**Tenant Isolation and RLS**

- Every subscription must contain `firm_id`.
- RLS must restrict firm access to the current tenant.
- Composite foreign keys must enforce firm-scoped relationships.
- Platform subscription operations must use separately authorized administrative access.
- Firm administrators may view their subscription details through authorized backend services.
- Firm users must not be able to alter subscription status, agreed price or platform-controlled entitlements.
- Platform administration must not automatically grant unrestricted access to firm customer tax information.

**Business and Security Rules**

- Subscription creation and activation must be performed through authorized platform workflows.
- Subscription activation must follow the approved payment, trial or manual activation process.
- Plan changes must preserve the previous subscription's history and commercial terms.
- Upgrades and downgrades must follow explicitly defined effective-date and proration policies.
- Subscription cancellation must follow the agreed cancellation policy.
- Scheduled cancellation must not immediately disable access before the applicable period ends.
- Expiry and suspension must trigger the appropriate entitlement and access-control updates.
- Subscription status must not be treated as a substitute for firm account security status.
- All subscription lifecycle operations must be auditable.
- Subscription entitlements must be evaluated by backend services using the applicable plan and agreed subscription terms.

**Data Integrity and Lifecycle**

- Subscription creation and activation must be transactional.
- Subscription renewal must preserve billing-period history.
- Subscription replacement must maintain a traceable relationship to the previous subscription.
- A cancelled or expired subscription must not be silently reactivated without an authorized lifecycle operation.
- Historical subscriptions must remain available for financial reporting and audit.
- Subscription termination must not delete firm data.
- Data retention and export must follow the platform's contractual and legal policies.
- All timestamps must use UTC.
- All monetary values must use exact decimal arithmetic.

**Design Notes and Open Decisions**

- The system must define whether one firm may maintain multiple simultaneous subscriptions.
- The system must finalize trial eligibility and trial extension rules.
- The system must determine whether renewals update the same subscription record or create separate subscription-period records.
- The system must define upgrade and downgrade proration.
- The system must determine whether subscriptions are activated manually or through a payment gateway at launch.
- The system must define payment failure, grace period and suspension rules.
- The system must determine how historical plan pricing and feature entitlements are snapshotted.
- The system must define cancellation and reactivation policies.
- The system must determine how subscription lifecycle events are synchronized with `subscription_history` and `subscription_event`.
- The system must define how firm closure and subscription termination interact.

### 3.68 `subscription_payment`

**Purpose**

The `subscription_payment` table records payments made by CA firms to the ITR Management System SaaS platform for subscription charges.

It supports manual payment recording initially and can later support authorized payment gateway integrations.

This table represents platform revenue collection from CA firms. It must remain separate from `customer_payment`, which records payments made by tax customers to their respective CA firms.

**Ownership and Tenant Scope**

- Ownership: Platform-Owned, with a firm relationship.
- Tenant scope: Firm-associated, platform-controlled.
- `firm_id` identifies the subscribing firm.
- Access is restricted to authorized platform billing administrators and backend services.
- A firm administrator may view only their own subscription payment information.
- Firm users must not create, approve or modify platform subscription payment records.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Subscribing CA firm |
| `firm_subscription_id` | UUID | No | Related subscription |
| `payment_reference` | VARCHAR(100) | No | Unique platform payment reference |
| `payment_amount` | NUMERIC(18,2) | No | Amount received |
| `currency_code` | CHAR(3) | No | ISO currency code |
| `payment_method` | VARCHAR(40) | No | Payment method |
| `payment_status` | VARCHAR(30) | No | Payment lifecycle status |
| `payment_date` | DATE | Yes | Business date of payment |
| `received_at` | TIMESTAMPTZ | Yes | Timestamp payment was received |
| `provider_name` | VARCHAR(100) | Yes | Payment gateway or provider |
| `provider_transaction_reference` | VARCHAR(255) | Yes | External transaction reference |
| `provider_order_reference` | VARCHAR(255) | Yes | External order reference |
| `billing_period_start` | TIMESTAMPTZ | Yes | Period covered by payment |
| `billing_period_end` | TIMESTAMPTZ | Yes | End of covered billing period |
| `invoice_reference` | VARCHAR(100) | Yes | Related platform invoice reference |
| `description` | TEXT | Yes | Payment description |
| `failure_reason` | TEXT | Yes | Failure details when applicable |
| `reversed_at` | TIMESTAMPTZ | Yes | Payment reversal timestamp |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `recorded_by_platform_user_id` | UUID | Yes | Platform administrator recording payment |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, firm_subscription_id)` references the corresponding firm subscription using a same-firm composite foreign key.
- `recorded_by_platform_user_id` references `platform_user.id`.

Provider transaction references must follow an approved uniqueness and idempotency policy.

**Payment Methods**

Initial supported values:

- `bank_transfer`
- `upi`
- `credit_card`
- `debit_card`
- `payment_gateway`
- `cheque`
- `other`

**Payment Statuses**

Initial supported values:

- `pending`
- `received`
- `failed`
- `partially_reversed`
- `reversed`

**Constraints and Data Integrity**

- Payment reference must not be empty.
- Payment reference must be unique within the platform or the applicable firm scope, as finalized in the schema.
- Payment amount must be greater than zero.
- Currency must use a supported ISO currency code.
- Payment method and status must use approved values.
- Received payments must have a received timestamp.
- Reversed payments must preserve reversal evidence.
- Billing period end must be later than its start when both are provided.
- Subscription and firm identifiers must match.
- Duplicate provider payment events must not create duplicate payment records.
- Provider transaction identifiers must be validated within their provider-specific scope.

**Indexes**

Proposed indexes:

- Unique index on `payment_reference`.
- Index on `(firm_id, firm_subscription_id, payment_status)`.
- Index on `(firm_id, payment_date DESC)`.
- Index on `(payment_status, created_at DESC)`.
- Index on `(provider_name, provider_transaction_reference)`.
- Index on `(firm_id, billing_period_start, billing_period_end)`.
- Index on `recorded_by_platform_user_id`.

**Tenant Isolation and RLS**

- Every payment must contain `firm_id`.
- Platform-level access must be restricted to authorized billing administrators and backend services.
- Firm administrators may retrieve only payments belonging to their own firm.
- Firm-scoped retrieval must validate `firm_id` against the authenticated firm context.
- Composite foreign keys must ensure that the payment and subscription belong to the same firm.
- Platform billing permissions must be separate from firm-level financial permissions.
- Platform administrators must not automatically receive unrestricted access to firm customer financial records.

**Business and Security Rules**

- Payments must be recorded only through authorized billing workflows.
- Manual payment confirmation must require appropriate platform authorization and supporting evidence.
- Payment gateway callbacks must be authenticated and verified.
- Payment processing must be idempotent.
- Provider acceptance of a payment request must not automatically mean that payment has been received.
- Only confirmed payments may satisfy subscription payment obligations.
- Failed payments must not activate or renew a subscription.
- Payment reversals must preserve the original payment record.
- Subscription payment records must not store card numbers, CVV values, banking passwords or payment credentials.
- Provider transaction references must be protected from unauthorized disclosure.
- All payment creation, confirmation, failure and reversal operations must be auditable.

**Data Integrity and Lifecycle**

- Payment records must preserve the original payment evidence.
- Payment confirmation and corresponding subscription billing updates must be transactionally consistent.
- Duplicate payment callbacks must not create duplicate financial effects.
- Reversals must be recorded without deleting the original payment.
- Partial reversals must preserve the original amount and record the reversed amount through an appropriate reversal model.
- Failed and pending payments must remain distinguishable from received payments.
- Historical subscription payments must remain available for platform financial reporting and reconciliation.
- All monetary values must use exact decimal arithmetic.
- All timestamps must use UTC; payment dates must use business dates.

**Design Notes and Open Decisions**

- The system must define whether platform subscription payments are always associated with a platform-generated invoice.
- The system must determine whether one payment may cover multiple subscription periods or invoices.
- The system must define the payment gateway and integration model.
- The system must decide whether payment attempts require a separate history table.
- The system must define partial payment and partial reversal handling.
- The system must establish provider transaction idempotency and uniqueness rules.
- The system must determine whether a separate platform refund table is required for subscription payment refunds.
- The system must define payment reconciliation and receipt requirements.
- The system must decide whether manual bank-transfer confirmation is the initial payment method.
- The system must define the relationship between payment confirmation and subscription activation or renewal.

### 3.69 `subscription_history`

**Purpose**

The `subscription_history` table records the historical lifecycle changes and important modifications made to a firm's SaaS subscription.

It provides an auditable timeline of subscription events, including:

- Subscription creation and activation.
- Trial commencement and completion.
- Plan upgrades and downgrades.
- Pricing and billing interval changes.
- Subscription renewal.
- Payment-related suspension.
- Cancellation and expiry.
- Reactivation and termination.

The table preserves historical subscription information and must not replace the current subscription state stored in `firm_subscription`.

**Ownership and Tenant Scope**

- Ownership: Platform-Owned, with a firm relationship.
- Tenant scope: Firm-associated, platform-controlled.
- `firm_id` identifies the affected firm.
- Access is restricted to authorized platform billing administrators and backend services.
- Firm administrators may view subscription history belonging to their own firm.
- Firm users must not modify or delete subscription history.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Affected firm |
| `firm_subscription_id` | UUID | No | Related subscription |
| `event_type` | VARCHAR(60) | No | Subscription event category |
| `previous_status` | VARCHAR(30) | Yes | Status before the event |
| `new_status` | VARCHAR(30) | Yes | Status after the event |
| `previous_plan_id` | UUID | Yes | Previous subscription plan |
| `new_plan_id` | UUID | Yes | New subscription plan |
| `previous_price` | NUMERIC(18,2) | Yes | Previous agreed price |
| `new_price` | NUMERIC(18,2) | Yes | New agreed price |
| `currency_code` | CHAR(3) | Yes | Currency for recorded price values |
| `previous_billing_interval` | VARCHAR(20) | Yes | Previous billing frequency |
| `new_billing_interval` | VARCHAR(20) | Yes | New billing frequency |
| `change_reason` | TEXT | Yes | Reason for the subscription change |
| `event_data` | JSONB | No | Additional validated event metadata |
| `changed_by_platform_user_id` | UUID | Yes | Platform administrator responsible |
| `change_source` | VARCHAR(40) | No | Origin of the event |
| `correlation_id` | UUID | Yes | Identifier linking related operations |
| `occurred_at` | TIMESTAMPTZ | No | Time the subscription event occurred |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, firm_subscription_id)` references `firm_subscription` using a same-firm composite foreign key.
- `previous_plan_id` references `subscription_plan.id` when supplied.
- `new_plan_id` references `subscription_plan.id` when supplied.
- `changed_by_platform_user_id` references `platform_user.id` when supplied.

System-generated events may have no platform administrator reference.

**Event Types**

Initial supported values:

- `subscription_created`
- `subscription_activated`
- `trial_started`
- `trial_ended`
- `plan_upgraded`
- `plan_downgraded`
- `pricing_changed`
- `billing_interval_changed`
- `subscription_renewed`
- `payment_failed`
- `subscription_suspended`
- `subscription_reactivated`
- `cancellation_scheduled`
- `subscription_cancelled`
- `subscription_expired`
- `subscription_terminated`
- `other`

**Change Sources**

Initial supported values:

- `platform_admin`
- `system`
- `payment_gateway`
- `automated_process`
- `authorized_integration`

**Constraints and Data Integrity**

- Event type must not be empty.
- Event type and change source must use approved values.
- Previous and new status values must use the approved subscription status catalogue when supplied.
- Previous and new prices must not be negative.
- Event data must contain a valid JSON object.
- Plan references must point to valid subscription plans.
- The subscription and firm identifiers must match.
- Every event must have an occurrence timestamp.
- Changes that affect subscription status, plan or pricing must preserve sufficient previous and new values for audit.
- System-generated events must identify their source clearly.
- Historical events must not be silently modified or deleted.

**Indexes**

Proposed indexes:

- Index on `(firm_id, firm_subscription_id, occurred_at DESC)`.
- Index on `(firm_id, event_type, occurred_at DESC)`.
- Index on `(firm_id, changed_by_platform_user_id, occurred_at DESC)`.
- Index on `(firm_id, change_source, occurred_at DESC)`.
- Index on `(firm_id, correlation_id)`.
- Index on `(occurred_at)` for platform-wide historical reporting.

**Tenant Isolation and RLS**

- Every history record must contain `firm_id`.
- Firm-facing queries must be restricted to the authenticated firm's subscription.
- Composite foreign keys must enforce same-firm subscription relationships.
- Platform billing administrators must use separately authorized platform access.
- Firm administrators may view their own subscription history through approved backend services.
- Firm users must not create, modify or delete subscription history.
- Platform access must not automatically grant access to unrelated firm tax or customer records.

**Business and Security Rules**

- Subscription lifecycle changes must generate corresponding history records.
- History generation must occur in the same transaction as the corresponding subscription state change wherever possible.
- Previous and new values must accurately represent the state transition.
- Manual administrative changes must identify the responsible platform administrator.
- Automated changes must identify the system or integration source.
- Payment gateway events must be verified before generating subscription history.
- History records must not contain payment credentials, secrets or unnecessary customer tax information.
- Historical subscription records must remain available for authorized billing disputes and audit.
- All subscription changes must be traceable through correlation identifiers where applicable.

**Data Integrity and Lifecycle**

- History records should be append-only.
- Subscription updates and their history records must be transactionally consistent.
- Failed subscription changes must not create misleading successful history events.
- Corrections must be represented by additional events rather than overwriting historical records.
- Historical pricing must preserve the currency and billing context applicable at the time.
- Retention and archival must follow platform financial and audit policies.
- All timestamps must use UTC.
- The latest valid subscription state must remain consistent with `firm_subscription`.

**Design Notes and Open Decisions**

- The system must decide whether subscription history stores every lifecycle event or only material changes.
- The system must define the event catalogue and status transition rules.
- The system must determine whether a dedicated immutable event identifier or idempotency key is required.
- The system must define how failed and retried subscription operations are represented.
- The system must decide whether complete plan entitlement snapshots are required for historical subscriptions.
- The system must define how subscription history relates to `subscription_event`.
- The system must determine whether provider callback payload references are retained.
- The system must define retention, archival and tamper-resistance requirements for subscription history.

### 3.70 `subscription_usage`

**Purpose**

The `subscription_usage` table records the consumption of subscription-based features and resource limits by each CA firm.

It supports entitlement enforcement, usage dashboards, subscription limit monitoring and billing-related reporting.

Examples include:

- Number of active customer relationships.
- Number of active firm staff accounts.
- Document storage consumption.
- Monthly notification usage.
- Assessment-year usage.
- API request consumption.
- Other subscription-controlled resources.

Usage records must be measurable, traceable and consistent with the entitlement definitions in `plan_feature`.

This table must not independently grant permissions or modify subscription entitlements.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned, with platform-controlled entitlement management.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Each usage record belongs to one firm.
- Platform billing and entitlement services may access usage through separately authorized backend operations.
- Firm administrators may view usage applicable to their own firm.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm |
| `firm_subscription_id` | UUID | No | Related subscription |
| `feature_key` | VARCHAR(100) | No | Feature or resource being measured |
| `usage_period_start` | TIMESTAMPTZ | No | Start of measurement period |
| `usage_period_end` | TIMESTAMPTZ | No | End of measurement period |
| `usage_quantity` | NUMERIC(20,4) | No | Measured resource consumption |
| `usage_unit` | VARCHAR(30) | No | Unit of measurement |
| `usage_limit` | NUMERIC(20,4) | Yes | Applicable entitlement limit |
| `usage_status` | VARCHAR(30) | No | Usage monitoring status |
| `last_calculated_at` | TIMESTAMPTZ | No | Most recent calculation timestamp |
| `calculation_version` | VARCHAR(50) | Yes | Version of the usage calculation logic |
| `usage_metadata` | JSONB | No | Additional validated usage information |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, firm_subscription_id)` references `firm_subscription` using a same-firm composite foreign key.

The feature key must correspond to an approved feature entitlement.

The applicable usage limit must be determined using the subscription terms effective during the measurement period, rather than assuming that the firm's current plan was active throughout the entire period.

**Usage Units**

Initial supported examples:

- `count`
- `bytes`
- `kilobytes`
- `megabytes`
- `gigabytes`
- `requests`
- `messages`
- `minutes`
- `other`

Each feature must have a consistent measurement unit.

**Usage Statuses**

Initial supported values:

- `within_limit`
- `approaching_limit`
- `limit_reached`
- `exceeded`
- `unlimited`
- `not_applicable`

**Constraints and Data Integrity**

- Feature key must not be empty.
- Usage quantity must not be negative.
- Usage limit must not be negative when supplied.
- Usage period end must be later than its start.
- Usage unit must use an approved value.
- Usage status must use an approved value.
- Usage metadata must contain a valid JSON object.
- Subscription and firm identifiers must match.
- Usage calculations must use consistent definitions and units.
- Duplicate period summaries must be prevented using an appropriate unique constraint based on firm, subscription, feature and measurement period.

**Indexes**

Proposed indexes:

- Unique index on `(firm_id, firm_subscription_id, feature_key, usage_period_start, usage_period_end)`.
- Index on `(firm_id, feature_key, usage_period_start DESC)`.
- Index on `(firm_id, usage_status)`.
- Index on `(firm_id, last_calculated_at)`.
- Index on `(firm_subscription_id, feature_key)`.

**Tenant Isolation and RLS**

- Every usage record must contain `firm_id`.
- RLS must restrict firm access to the current tenant.
- Composite foreign keys must enforce same-firm subscription relationships.
- Backend entitlement services must control usage calculation and updates.
- Firm administrators may retrieve their own usage information through authorized APIs.
- Firm users must not be able to modify usage quantities or entitlement limits directly.
- Platform administration must use separately authorized access.

**Business and Security Rules**

- Usage must be calculated using approved and documented measurement rules.
- Usage calculations must be repeatable and idempotent.
- Usage limits must be enforced by backend services.
- Frontend indicators must not be treated as enforcement controls.
- Resource consumption must be attributed to the correct firm.
- Duplicate events must not inflate usage.
- Deleted, archived or reversed records must follow defined usage adjustment rules.
- Usage changes must be traceable to their source events where necessary.
- Approaching or exceeded limits may trigger notifications according to platform policy.
- Unlimited entitlements must not be represented as an ordinary zero usage limit.
- Usage reporting must not expose another firm's consumption data.

**Data Integrity and Lifecycle**

- Usage summaries must be recalculated or reconciled against authoritative source records.
- Period boundaries must be consistent with the subscription billing and entitlement rules.
- Usage records must preserve the measurement period and calculation version.
- Usage changes resulting from corrections must be traceable.
- Current usage must be consistent with the underlying resource consumption.
- Historical usage must remain available for subscription disputes and reporting.
- Usage aggregation must account for concurrent operations and prevent double counting.
- All timestamps must use UTC.
- Resource quantities must use appropriate precision and units.

**Design Notes and Open Decisions**

- The system must decide whether usage is stored as periodic summaries, individual usage events or both.
- A separate immutable `subscription_usage_event` table may be required for reliable event-level accounting.
- The system must define whether usage periods follow calendar months or subscription billing periods.
- The system must define exact measurement rules for customers, staff, storage and notifications.
- The system must determine how deleted documents and inactive customers affect usage.
- The system must define hard-limit enforcement versus warning thresholds.
- The system must decide how plan upgrades and downgrades affect usage limits during an existing period.
- The system must define usage correction and reconciliation procedures.
- The system must determine how usage is handled during trials, suspension and expired subscriptions.
- The system must define retention and archival requirements for historical usage.

### 3.71 `subscription_event`

**Purpose**

The `subscription_event` table records subscription-related business and integration events generated by the ITR Management System SaaS platform.

It provides a durable event record for subscription lifecycle processing, payment integration, background jobs, notifications and entitlement updates.

Examples include:

- Subscription created or activated.
- Trial started or ended.
- Payment received or failed.
- Subscription renewed.
- Plan upgraded or downgraded.
- Subscription suspended or reactivated.
- Cancellation scheduled or completed.
- Payment gateway callback received.
- Subscription entitlement recalculated.

This table supports reliable event processing and idempotency. It must not replace the authoritative subscription, payment or subscription-history records.

**Ownership and Tenant Scope**

- Ownership: Platform-Owned, with a firm relationship.
- Tenant scope: Firm-associated, platform-controlled.
- `firm_id` identifies the affected firm.
- Access is restricted to authorized platform services and administrators.
- Firm administrators may see only approved subscription event information for their own firm.
- Firm users must not create, modify or delete platform subscription events.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | Yes | Affected firm, where applicable |
| `firm_subscription_id` | UUID | Yes | Related subscription, where applicable |
| `event_type` | VARCHAR(100) | No | Event category |
| `event_source` | VARCHAR(50) | No | Origin of the event |
| `event_status` | VARCHAR(30) | No | Processing status |
| `event_reference` | VARCHAR(150) | No | Internal event reference |
| `external_event_id` | VARCHAR(255) | Yes | External provider event identifier |
| `idempotency_key` | VARCHAR(255) | Yes | Duplicate-processing prevention key |
| `event_payload` | JSONB | No | Validated event metadata |
| `correlation_id` | UUID | Yes | Identifier linking related operations |
| `occurred_at` | TIMESTAMPTZ | No | Time the event occurred |
| `received_at` | TIMESTAMPTZ | No | Time the platform received the event |
| `processed_at` | TIMESTAMPTZ | Yes | Successful processing timestamp |
| `processing_attempts` | INTEGER | No | Number of processing attempts |
| `last_error_code` | VARCHAR(100) | Yes | Last processing error code |
| `last_error_message` | TEXT | Yes | Sanitized error description |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id` when supplied.
- `(firm_id, firm_subscription_id)` references `firm_subscription` when both are supplied.

Platform-wide events may not belong to a specific firm or subscription.

When a subscription is specified, the firm identifier must match the subscription's firm.

**Event Types**

Initial supported values:

- `subscription_created`
- `subscription_activated`
- `trial_started`
- `trial_ended`
- `payment_received`
- `payment_failed`
- `subscription_renewed`
- `plan_upgraded`
- `plan_downgraded`
- `subscription_suspended`
- `subscription_reactivated`
- `cancellation_scheduled`
- `subscription_cancelled`
- `subscription_expired`
- `entitlement_updated`
- `payment_gateway_callback`
- `other`

**Event Sources**

Initial supported values:

- `platform_application`
- `payment_gateway`
- `background_job`
- `platform_admin`
- `authorized_integration`
- `automated_process`

**Event Statuses**

Initial supported values:

- `received`
- `pending`
- `processing`
- `processed`
- `failed`
- `ignored`
- `dead_letter`

**Constraints and Data Integrity**

- Event type, source and status must use approved values.
- Event reference must not be empty.
- Event reference must be unique according to its defined platform scope.
- Event payload must contain a valid JSON object.
- Processing attempts must not be negative.
- Processed events must have a processing timestamp.
- Failed events must preserve appropriate error information.
- External event identifiers must follow provider-specific uniqueness rules.
- Idempotency keys must prevent duplicate business processing.
- Subscription and firm identifiers must match when both are supplied.

**Indexes**

Proposed indexes:

- Unique index on `event_reference`.
- Index on `(firm_id, firm_subscription_id, occurred_at DESC)`.
- Index on `(event_status, created_at)`.
- Index on `(event_type, occurred_at DESC)`.
- Index on `(event_source, occurred_at DESC)`.
- Index on `(correlation_id)`.
- Index on `(external_event_id)`.
- Index on `(event_status, processing_attempts, created_at)`.

Provider-specific uniqueness constraints must include the provider or source scope where required.

**Tenant Isolation and RLS**

- Firm-associated events must contain the correct `firm_id`.
- Firm-facing access must be restricted to the authenticated firm's subscription events.
- Platform-wide events must use a separate platform-authorized access path.
- Composite foreign keys must enforce same-firm subscription relationships.
- Firm administrators must not access raw platform integration payloads unless explicitly authorized.
- Platform administration must use least-privilege access.
- Event payloads must not expose unrelated firm data.

**Business and Security Rules**

- Events must be generated by authorized platform services, verified integrations or authorized platform administrators.
- External payment gateway callbacks must be authenticated and validated.
- Event processing must be idempotent.
- Duplicate events must not create duplicate payments, renewals or subscription state changes.
- Event processing must use controlled background workers.
- Failed processing must support safe retries.
- Events that repeatedly fail must be moved to a controlled dead-letter workflow.
- Sensitive credentials, payment details and unnecessary customer data must not be stored in event payloads.
- Event processing must generate appropriate subscription history and audit records.
- Events must not directly grant permissions or bypass subscription entitlement evaluation.

**Data Integrity and Lifecycle**

- Event receipt and durable persistence must occur before asynchronous processing.
- Event processing must use transactional updates wherever possible.
- A failed event must not leave partial subscription or payment changes.
- Processing attempts and failure details must be traceable.
- Successful processing must record a completion timestamp.
- Duplicate delivery must be safely recognized.
- Events must be retained according to platform audit and integration policies.
- Processed events must not be silently deleted or rewritten.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must determine whether this table is an inbound event inbox, an internal domain event store or both.
- Separate inbox and outbox tables may be required for reliable event delivery.
- The system must define the exact event catalogue and event versioning strategy.
- The system must define provider-specific callback authentication and replay protection.
- The system must establish idempotency key generation and retention.
- The system must define retry intervals, maximum attempts and dead-letter recovery.
- The system must determine whether raw provider payloads are retained or sanitized before storage.
- The system must define event ordering and concurrency handling.
- The system must determine how events generate `subscription_history` records.
- The system must define retention, archival and tamper-resistance requirements.

### 3.72 `import_job`

**Purpose**

The `import_job` table records and manages Excel and other approved data-import operations performed by CA firms.

It provides traceability for the complete import lifecycle, including file upload, validation, preview, execution, completion, failure and cancellation.

The initial implementation will primarily support customer master-data imports from Excel files.

Future imports may include:

- Customer records.
- Customer contact information.
- Customer tax identity information.
- Historical customer references.
- Other approved firm-specific business records.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Each import job belongs to exactly one CA firm.
- Only authorized firm administrators and staff may initiate or review imports according to assigned permissions.
- Platform administrators must use separately authorized access.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm |
| `import_reference` | VARCHAR(100) | No | Firm-specific import reference |
| `import_type` | VARCHAR(50) | No | Type of data being imported |
| `source_file_name` | VARCHAR(255) | No | Original uploaded file name |
| `storage_provider` | VARCHAR(50) | No | Private storage provider |
| `storage_bucket` | VARCHAR(255) | No | Private storage bucket |
| `storage_object_key` | VARCHAR(500) | No | Private storage object key |
| `file_mime_type` | VARCHAR(150) | No | Detected file MIME type |
| `file_size_bytes` | BIGINT | No | Uploaded file size |
| `file_checksum` | VARCHAR(128) | No | File integrity checksum |
| `import_status` | VARCHAR(30) | No | Current import lifecycle status |
| `total_rows` | INTEGER | No | Total data rows detected |
| `valid_rows` | INTEGER | No | Rows passing validation |
| `invalid_rows` | INTEGER | No | Rows failing validation |
| `imported_rows` | INTEGER | No | Successfully imported rows |
| `skipped_rows` | INTEGER | No | Rows intentionally skipped |
| `duplicate_rows` | INTEGER | No | Rows identified as duplicates |
| `uploaded_by_membership_id` | UUID | No | Membership that initiated the import |
| `started_at` | TIMESTAMPTZ | Yes | Import execution start |
| `completed_at` | TIMESTAMPTZ | Yes | Import completion time |
| `failed_at` | TIMESTAMPTZ | Yes | Import failure time |
| `failure_summary` | TEXT | Yes | Sanitized failure description |
| `cancelled_at` | TIMESTAMPTZ | Yes | Cancellation timestamp |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`.
- `(firm_id, uploaded_by_membership_id)` references `firm_membership` through a same-firm composite foreign key.

**Import Types**

Initial supported values:

- `customer_master`
- `customer_contact`
- `customer_tax_identity`
- `historical_customer_reference`
- `other`

Only explicitly approved import types may be processed.

**Import Statuses**

Initial supported values:

- `uploaded`
- `pending_validation`
- `validating`
- `validation_completed`
- `validation_failed`
- `awaiting_confirmation`
- `processing`
- `completed`
- `completed_with_errors`
- `failed`
- `cancelled`

**Constraints and Data Integrity**

- Import reference must not be empty.
- File name and storage object key must not be empty.
- File size must be positive and within configured limits.
- Row counters must not be negative.
- Invalid, valid, duplicate and imported row counts must remain consistent with the import-processing rules.
- Completed jobs must have a completion timestamp.
- Failed jobs must preserve a failure timestamp and sanitized failure summary.
- Cancelled jobs must preserve a cancellation timestamp.
- The uploaded file must belong to the same firm as the import job.
- File checksum must be generated and verified by trusted backend services.

**Indexes**

Proposed indexes:

- Unique index on `(firm_id, import_reference)`.
- Index on `(firm_id, import_status, created_at DESC)`.
- Index on `(firm_id, import_type, created_at DESC)`.
- Index on `(firm_id, uploaded_by_membership_id, created_at DESC)`.
- Index on `(import_status, created_at)` for authorized background processing.
- Index on `(firm_id, completed_at)`.

**Tenant Isolation and RLS**

- Every import job must contain `firm_id`.
- RLS must restrict firm users to their own import jobs.
- Composite foreign keys must enforce same-firm membership references.
- Import processing workers must establish the correct tenant context.
- Background processing must not rely solely on a client-supplied firm identifier.
- Platform-level operational access must be separately authorized and audited.

**Business and Security Rules**

- Only authorized firm users may initiate imports.
- Uploads must be stored in private object storage.
- File extension, actual file format, MIME type and file signature must be validated.
- Only approved file formats and configured file-size limits are permitted.
- Uploaded files must be scanned for malware before processing.
- Files must remain quarantined until required security checks pass.
- Excel formulas and macros must not be executed.
- Imported values must be treated as untrusted input.
- Validation and preview must occur before committing imported records.
- Users must be able to review validation results before confirming an import.
- Duplicate detection must follow approved firm-specific business rules.
- Import execution must use controlled transactions or documented batch-processing strategies.
- Failed imports must not silently leave inconsistent customer records.
- Import workers must be idempotent and safe against duplicate job execution.
- Sensitive tax identifiers must be encrypted and must never appear in ordinary import logs or error reports.

**Data Integrity and Lifecycle**

- An import job must preserve its original uploaded-file reference and checksum.
- Validation must finish before an import can be confirmed for execution.
- Row counts must be updated consistently as processing progresses.
- Successful and failed row outcomes must be traceable.
- Import completion must reflect actual committed records.
- Cancellation must prevent further processing where safely possible.
- Temporary files and intermediate processing data must follow defined cleanup policies.
- Imported business records must preserve their normal audit and ownership relationships.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must determine supported Excel formats and worksheet-selection rules.
- The system must define column mapping and header-detection behavior.
- A separate `import_mapping` table will store reusable firm-specific column mappings.
- A separate `import_row_result` table will record individual row processing outcomes.
- A separate `import_error` table will preserve structured validation and processing errors.
- The system must define whether imports are atomic or processed in controlled batches.
- The system must define rollback and recovery behavior for partially completed batch imports.
- The system must define duplicate detection and customer matching rules.
- The system must define maximum file size, row count and processing duration.
- The system must define temporary file retention and secure deletion.
- The system must define how cancelled and failed jobs can be retried safely.

### 3.73 `import_mapping`

**Purpose**

The `import_mapping` table stores reusable column-mapping configurations for data-import operations.

It defines how source spreadsheet columns correspond to approved fields in the ITR Management System.

For example:

| Excel Column | System Field |
|---|---|
| Client Name | customer_name |
| PAN Number | pan |
| Mobile | phone_number |
| Email ID | email |
| Assessment Year | assessment_year |

The mapping configuration helps standardize imports and reduce repeated manual configuration.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Each mapping belongs to one CA firm.
- Only authorized firm administrators and staff may create, update, activate or use mappings.
- Mappings must not be shared across firms without an explicitly authorized platform-level template mechanism.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm |
| `mapping_name` | VARCHAR(150) | No | User-friendly mapping name |
| `mapping_code` | VARCHAR(100) | No | Firm-specific mapping identifier |
| `import_type` | VARCHAR(50) | No | Applicable import type |
| `source_format` | VARCHAR(30) | No | Spreadsheet or file format |
| `mapping_configuration` | JSONB | No | Source-to-system field mappings |
| `mapping_version` | INTEGER | No | Mapping version number |
| `mapping_status` | VARCHAR(30) | No | Mapping lifecycle status |
| `is_default` | BOOLEAN | No | Whether this is the default mapping |
| `description` | TEXT | Yes | Mapping description |
| `created_by_membership_id` | UUID | No | Creating firm membership |
| `updated_by_membership_id` | UUID | No | Last updating firm membership |
| `created_at` | TIMESTAMPTZ | No | Creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `archived_at` | TIMESTAMPTZ | Yes | Archival timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`.
- `(firm_id, created_by_membership_id)` references `firm_membership` through a same-firm composite foreign key.
- `(firm_id, updated_by_membership_id)` references `firm_membership` through a same-firm composite foreign key.

**Import Types**

Initial supported values:

- `customer_master`
- `customer_contact`
- `customer_tax_identity`
- `historical_customer_reference`
- `other`

**Source Formats**

Initial supported values:

- `xlsx`
- `xls`
- `csv`

Only formats supported by the import-processing service may be configured.

**Mapping Statuses**

Initial supported values:

- `draft`
- `active`
- `inactive`
- `archived`

**Constraints and Data Integrity**

- Mapping name and code must not be empty.
- Mapping version must be greater than zero.
- Mapping configuration must contain a valid JSON object.
- Import type and source format must use approved values.
- A mapping code must be unique within its firm.
- Only one active default mapping may exist for a given firm and import type.
- Mapping configuration must reference approved system fields.
- Duplicate source-column assignments must be rejected unless explicitly supported.
- Required system fields must be mapped or supplied through approved defaults.
- Archived mappings must not be used for new imports.

**Indexes**

Proposed indexes:

- Unique index on `(firm_id, mapping_code)`.
- Index on `(firm_id, import_type, mapping_status)`.
- Index on `(firm_id, is_default)`.
- Index on `(firm_id, created_at DESC)`.
- Partial unique index on `(firm_id, import_type)` where `is_default = TRUE` and `mapping_status = 'active'`.

**Tenant Isolation and RLS**

- Every mapping must contain `firm_id`.
- RLS must restrict mapping access to the current firm.
- Creator and updater memberships must belong to the same firm.
- Import-processing services must verify that the selected mapping belongs to the import job's firm.
- Firm users must never be able to retrieve or apply another firm's mapping.
- Platform-provided templates, if introduced, must use a separate platform-owned model or explicitly defined access mechanism.

**Business and Security Rules**

- Mapping configuration must be validated before activation.
- Only approved system fields may be selected.
- Sensitive fields such as PAN must be handled through authorized import services.
- Mapping configuration must never contain passwords, API keys or other secrets.
- Source spreadsheet values must always be treated as untrusted input.
- Formula execution and macro execution are prohibited.
- A mapping must be compatible with its declared import type and source format.
- Existing imports must retain the mapping version used during their execution.
- Changes to an existing mapping must not silently alter historical import results.
- Mapping creation, modification, activation and archival must be auditable.

**Data Integrity and Lifecycle**

- Mapping configurations must be versioned.
- Active mappings must pass schema and business-rule validation.
- Changes must preserve historical versions or an equivalent immutable snapshot.
- Import jobs must record the exact mapping version used.
- Archived mappings must remain available for historical reference.
- Deletion of mappings must not invalidate completed or historical imports.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must determine whether mapping versions are stored in this table or in a separate mapping-version table.
- The system must define the approved system-field catalogue for each import type.
- The system must define source-column normalization and matching rules.
- The system must decide whether default values and transformation rules are supported.
- The system must define date, number, currency and text conversion rules.
- The system must define how unmapped columns are handled.
- The system must define whether firm-specific mappings can be copied from platform-provided templates.
- The system must define mapping compatibility checks when the underlying customer data model changes.

### 3.74 `import_row_result`

**Purpose**

The `import_row_result` table records the processing outcome of each individual data row within an import job.

It provides row-level traceability for validation, duplicate detection, import execution and error resolution.

Examples include:

- A customer row imported successfully.
- A row rejected because the PAN format is invalid.
- A row skipped because the customer already exists.
- A row requiring manual review because a potential duplicate was detected.
- A row failing because a required field is missing.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Every row result belongs to one import job and one CA firm.
- Only authorized firm users and import-processing services may access row-level results.
- Platform-level operational access must be separately authorized and audited.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm |
| `import_job_id` | UUID | No | Related import job |
| `source_row_number` | INTEGER | No | Original spreadsheet row number |
| `row_status` | VARCHAR(30) | No | Processing outcome |
| `source_row_reference` | VARCHAR(150) | Yes | Reference to the source row or staged record |
| `normalized_data` | JSONB | Yes | Validated and normalized row data |
| `created_entity_type` | VARCHAR(100) | Yes | Type of business record created |
| `created_entity_id` | UUID | Yes | Identifier of the created business record |
| `matched_entity_type` | VARCHAR(100) | Yes | Type of existing record matched |
| `matched_entity_id` | UUID | Yes | Identifier of the matched record |
| `duplicate_match_status` | VARCHAR(30) | Yes | Duplicate detection outcome |
| `processing_attempts` | INTEGER | No | Number of processing attempts |
| `processed_at` | TIMESTAMPTZ | Yes | Processing completion timestamp |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, import_job_id)` references `import_job` through a same-firm composite foreign key.

The created and matched entity references are polymorphic. Their integrity must be validated by the backend or by an approved entity-specific association design.

**Row Statuses**

Initial supported values:

- `pending`
- `validating`
- `valid`
- `invalid`
- `duplicate`
- `awaiting_review`
- `ready_to_import`
- `importing`
- `imported`
- `skipped`
- `failed`
- `cancelled`

**Duplicate Match Statuses**

Initial supported values:

- `not_checked`
- `no_match`
- `possible_match`
- `confirmed_duplicate`
- `not_duplicate`
- `requires_review`

**Constraints and Data Integrity**

- Source row number must be positive.
- Processing attempts must not be negative.
- Row status must use an approved value.
- Duplicate match status must use an approved value.
- Normalized data, when supplied, must be a valid JSON object.
- An imported row must have a processing timestamp and an appropriate created-record reference.
- A skipped row must have an appropriate skip reason recorded in the associated error or result details.
- Each source row must have at most one authoritative result within an import job.
- Created and matched entity references must belong to the same firm.
- Sensitive tax identifiers must not be stored in plaintext within normalized data.

**Indexes**

Proposed indexes:

- Unique index on `(firm_id, import_job_id, source_row_number)`.
- Index on `(firm_id, import_job_id, row_status)`.
- Index on `(firm_id, row_status, created_at)`.
- Index on `(import_job_id, duplicate_match_status)`.
- Index on `(created_entity_type, created_entity_id)`.
- Index on `(matched_entity_type, matched_entity_id)`.
- Index on `(import_job_id, processed_at)`.

**Tenant Isolation and RLS**

- Every row result must contain `firm_id`.
- RLS must restrict access to the current firm.
- Composite foreign keys must enforce same-firm import-job ownership.
- Import workers must establish the correct tenant context before processing.
- Created and matched records must be checked against the same firm.
- Firm users must not be able to modify processing outcomes directly through ordinary customer-management APIs.
- Authorized import services must perform controlled updates.

**Business and Security Rules**

- Every accepted source row must receive an identifiable processing result.
- Row validation must complete before the row is eligible for import.
- Invalid rows must not create business records.
- Possible duplicates must follow the firm's approved duplicate-review policy.
- Confirmed duplicates must not be silently imported as new customers.
- Row processing must be idempotent.
- Retrying a row must not create duplicate business records.
- Row-level errors must be stored in the associated `import_error` records.
- Sensitive values must be masked in ordinary screens and logs.
- Normalized data must contain only fields required for validation and processing.
- Internal processing metadata must not be exposed to customer-facing APIs.

**Data Integrity and Lifecycle**

- Row results must preserve their original source row number.
- Processing outcomes must remain traceable to the parent import job.
- Row counts must reconcile with the parent import job's counters.
- Successful imports must reference the created business record.
- Failed and skipped rows must preserve their reasons.
- Corrections and retries must be traceable.
- Historical import results must remain available according to retention policy.
- Temporary source data and normalized data must follow secure cleanup and retention rules.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must determine whether normalized row data is stored permanently or only during import processing.
- The system must define the maximum permitted size of normalized row data.
- The system must determine how row-level sensitive values are encrypted or excluded.
- The system must define whether one row can create multiple business records.
- The system must define the entity-reference strategy for different import types.
- The system must determine whether retry attempts update the existing row result or create separate attempt-history records.
- The system must define duplicate matching thresholds and manual-review procedures.
- The system must define whether skipped rows are counted separately from invalid and duplicate rows.
- The system must establish reconciliation rules between row results and import-job summary counters.

### 3.75 `import_error`

**Purpose**

The `import_error` table records structured validation errors, data-quality issues and processing failures encountered during an import job.

It helps firm staff identify problems, understand why a row could not be imported and take corrective action.

Examples include:

- Missing mandatory customer name.
- Invalid PAN format.
- Invalid email or phone number.
- Unsupported assessment year.
- Duplicate customer detected.
- Invalid date or financial value.
- Unauthorized or unsupported source field.
- Database or processing failure.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-scoped.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Each error belongs to one firm's import job.
- Only authorized firm users and import-processing services may access error details.
- Platform-level operational access must be separately authorized and audited.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Owning firm |
| `import_job_id` | UUID | No | Related import job |
| `import_row_result_id` | UUID | Yes | Related spreadsheet row result |
| `error_code` | VARCHAR(100) | No | Machine-readable error identifier |
| `error_category` | VARCHAR(50) | No | Error classification |
| `error_severity` | VARCHAR(20) | No | Error severity |
| `error_message` | TEXT | No | Human-readable error description |
| `source_column_name` | VARCHAR(255) | Yes | Source spreadsheet column |
| `target_field_name` | VARCHAR(150) | Yes | Corresponding system field |
| `source_row_number` | INTEGER | Yes | Original spreadsheet row number |
| `error_details` | JSONB | No | Additional sanitized error metadata |
| `resolution_status` | VARCHAR(30) | No | Error resolution state |
| `resolution_notes` | TEXT | Yes | Explanation of corrective action |
| `resolved_by_membership_id` | UUID | Yes | Membership that resolved the error |
| `resolved_at` | TIMESTAMPTZ | Yes | Resolution timestamp |
| `created_at` | TIMESTAMPTZ | No | Error creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, import_job_id)` references `import_job` through a same-firm composite foreign key.
- `(firm_id, import_row_result_id)` references `import_row_result` through a same-firm composite foreign key when a row result is supplied.
- `(firm_id, resolved_by_membership_id)` references `firm_membership` through a same-firm composite foreign key when a resolving user is supplied.

An error without a specific row result may represent an import-level validation or processing failure.

When a row result is supplied, it must belong to the same import job.

**Error Categories**

Initial supported values:

- `file_validation`
- `mapping_validation`
- `required_field`
- `data_format`
- `data_integrity`
- `duplicate_detection`
- `authorization`
- `business_rule`
- `processing`
- `system`
- `other`

**Error Severities**

Initial supported values:

- `info`
- `warning`
- `error`
- `critical`

**Resolution Statuses**

Initial supported values:

- `open`
- `under_review`
- `resolved`
- `ignored`
- `requires_reimport`

**Constraints and Data Integrity**

- Error code must not be empty.
- Error message must not be empty.
- Error category and severity must use approved values.
- Resolution status must use an approved value.
- Source row number must be positive when supplied.
- Error details must contain a valid JSON object.
- A resolved error must have a resolution timestamp.
- A supplied resolving membership must belong to the same firm.
- A supplied row result must belong to the same import job.
- Error records must not contain plaintext passwords, OTPs, authentication tokens or unnecessary sensitive tax information.

**Indexes**

Proposed indexes:

- Index on `(firm_id, import_job_id, created_at)`.
- Index on `(firm_id, import_job_id, error_severity)`.
- Index on `(firm_id, import_row_result_id)`.
- Index on `(firm_id, error_code)`.
- Index on `(firm_id, resolution_status, created_at)`.
- Index on `(firm_id, resolved_by_membership_id, resolved_at)`.

**Tenant Isolation and RLS**

- Every error record must contain `firm_id`.
- RLS must restrict access to the current firm.
- Composite foreign keys must enforce same-firm relationships.
- Import workers must establish the correct tenant context.
- Error reports must never expose another firm's data.
- Firm users may view and resolve errors only when authorized.
- Platform operational access must use separately authorized and audited procedures.

**Business and Security Rules**

- Errors must be generated by trusted import-processing services.
- Error codes must be stable and suitable for programmatic handling.
- Error messages must be understandable without exposing internal implementation details.
- Sensitive source values must not be copied into error messages.
- Error details must contain only sanitized diagnostic metadata.
- Errors must be associated with the relevant import job and, where applicable, the specific source row.
- Warnings must be distinguishable from blocking errors.
- Critical errors may prevent the entire import from proceeding.
- Resolving an error must not automatically reimport a row unless an authorized retry operation is explicitly initiated.
- Error resolution and retry operations must be auditable.
- Database errors must not expose SQL statements, credentials or internal infrastructure details to ordinary firm users.

**Data Integrity and Lifecycle**

- Error records must preserve their original creation time and source context.
- Resolution changes must preserve sufficient history for audit purposes.
- Repeated processing failures must remain traceable.
- Error counts must reconcile with the relevant row results and import-job summary.
- Resolved errors must not be silently deleted.
- Historical errors must follow the platform's retention and archival policies.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must define whether resolved errors are updated in place or tracked through a separate error-resolution history table.
- The system must define which error categories block import execution.
- The system must define how warnings affect import approval.
- The system must establish standard error codes and message templates.
- The system must determine whether one row can have multiple errors of the same type.
- The system must define how import-level errors without a row result are displayed.
- The system must define retry behavior after error resolution.
- The system must define error-report export formats and sensitive-data masking.
- The system must determine retention periods for detailed error metadata.

### 3.76 `report_definition`

**Purpose**

The `report_definition` table stores metadata and configuration for reports available within the ITR Management System.

It supports predefined platform reports and authorized firm-specific report definitions.

Examples include:

- Customer summary report.
- ITR filing status report.
- Pending document report.
- Staff workload report.
- Invoice and payment report.
- Refund tracking report.
- Assessment-year performance report.
- Customer import summary report.

The table defines report availability and configuration. It must not itself grant access to underlying business data.

**Ownership and Tenant Scope**

- Ownership: Platform-Owned or Firm-Owned, depending on the report definition.
- Tenant scope: Global or firm-scoped.
- `firm_id` is nullable:
  - `NULL` represents a platform-provided report definition.
  - A populated value represents a firm-specific report definition.
- Firm-specific records must use mandatory tenant RLS.
- Platform-owned definitions must use a separately authorized platform access policy.
- Firm users may access platform-provided reports only when explicitly enabled and authorized.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | Yes | Owning firm for firm-specific reports |
| `report_key` | VARCHAR(100) | No | Stable report identifier |
| `report_name` | VARCHAR(200) | No | Display name |
| `report_description` | TEXT | Yes | Report description |
| `report_category` | VARCHAR(50) | No | Report classification |
| `report_type` | VARCHAR(30) | No | Standard or configurable report |
| `report_configuration` | JSONB | No | Validated report configuration |
| `required_permissions` | JSONB | No | Required permission identifiers |
| `supported_filters` | JSONB | No | Approved filter definitions |
| `report_status` | VARCHAR(30) | No | Report lifecycle status |
| `is_exportable` | BOOLEAN | No | Whether export is permitted |
| `is_sensitive` | BOOLEAN | No | Whether sensitive data is involved |
| `version_number` | INTEGER | No | Report definition version |
| `created_by_membership_id` | UUID | Yes | Creating firm membership |
| `updated_by_membership_id` | UUID | Yes | Updating firm membership |
| `created_by_platform_user_id` | UUID | Yes | Creating platform administrator |
| `updated_by_platform_user_id` | UUID | Yes | Updating platform administrator |
| `created_at` | TIMESTAMPTZ | No | Creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `archived_at` | TIMESTAMPTZ | Yes | Archival timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id` when the definition is firm-owned.
- Firm creator and updater references must use same-firm composite foreign keys to `firm_membership`.
- Platform creator and updater references must reference `platform_user.id`.

The creator and updater references must follow an explicit ownership rule. A firm-owned report must not be attributed to a platform administrator unless a separately authorized platform operation is recorded.

**Report Categories**

Initial supported values:

- `customer`
- `itr`
- `document`
- `checklist`
- `staff`
- `financial`
- `refund`
- `subscription`
- `import`
- `audit`
- `operational`
- `other`

**Report Types**

Initial supported values:

- `standard`
- `configurable`

**Report Statuses**

Initial supported values:

- `draft`
- `active`
- `inactive`
- `deprecated`
- `archived`

**Constraints and Data Integrity**

- Report key and name must not be empty.
- Report category, type and status must use approved values.
- Report configuration must contain a valid JSON object.
- Required permissions and supported filters must contain validated JSON arrays or objects according to their defined schemas.
- Version number must be greater than zero.
- A firm-owned definition must have a firm identifier.
- A platform-owned definition must not contain firm membership creator references.
- Firm-owned definitions must not contain unauthorized platform ownership references.
- Report keys must be unique within their ownership scope.
- Only active report definitions may be used for new report executions.

**Indexes**

Proposed indexes:

- Partial unique index on `(report_key)` where `firm_id IS NULL`.
- Partial unique index on `(firm_id, report_key)` where `firm_id IS NOT NULL`.
- Index on `(firm_id, report_category, report_status)`.
- Index on `(report_status, report_category)`.
- Index on `(firm_id, updated_at DESC)`.
- Index on `(is_sensitive, report_status)`.

**Tenant Isolation and RLS**

- Firm-specific report definitions must contain `firm_id`.
- RLS must restrict firm-owned definitions to the current tenant.
- Platform-owned definitions must use a separately controlled access path.
- Platform reports must not automatically provide cross-tenant data access.
- Report execution must independently enforce tenant context and data permissions.
- Firm users must not alter platform-provided report definitions.
- Firm-specific report configurations must not weaken platform security restrictions.

**Business and Security Rules**

- Reports must be generated through approved backend reporting services.
- Users must have the required permissions before accessing a report.
- Report execution must enforce firm-level tenant isolation.
- Sensitive reports must require additional authorization where appropriate.
- Report configuration must use approved fields, filters, joins and aggregations.
- Arbitrary SQL execution from user-provided report configurations is prohibited.
- Report definitions must not bypass RLS or backend authorization.
- Export permissions must be checked independently from view permissions.
- Sensitive customer information must be masked or excluded where appropriate.
- Changes to report definitions must be versioned and auditable.
- Platform-defined mandatory restrictions must not be overridden by firm configuration.

**Data Integrity and Lifecycle**

- Report definitions must retain a stable report key.
- Changes must preserve historical versions or equivalent immutable snapshots.
- Historical report executions must retain the definition version used.
- Deprecated reports must not be silently substituted with different report logic.
- Archived definitions must remain available for historical reference.
- Deletion must not invalidate existing report execution records.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must define the approved report catalogue and supported data sources.
- The system must decide whether report query logic is stored in code, configuration or a controlled combination.
- The system must define supported filters, grouping, sorting and aggregation rules.
- The system must determine how report definition versions are preserved.
- The system must define firm-specific customization limits.
- The system must define sensitive-report approval and export policies.
- The system must determine whether platform reports can be copied and customized by firms.
- The system must define report localization and date/number formatting.
- The system must define the relationship between report definitions and report execution records.
- The system must ensure report queries remain efficient and cannot cause unrestricted database scans.

### 3.77 `report_execution`

**Purpose**

The `report_execution` table records individual report-generation requests and their execution lifecycle.

It provides traceability for report access, execution, performance, failures and generated results.

Examples include:

- A firm administrator generating an ITR filing status report.
- A staff member generating a pending document report.
- A firm administrator exporting an invoice summary.
- A platform administrator generating an authorized platform-level operational report.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned or Platform-Owned, according to the report execution context.
- Tenant scope: Firm-scoped for firm reports; platform-controlled for platform reports.
- `firm_id` is nullable only for authorized platform-level executions.
- Firm-specific executions must use mandatory tenant RLS.
- Platform-level executions must use a separate, restricted access policy.
- Firm users may view only executions belonging to their own firm and permitted user scope.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | Yes | Owning firm, where applicable |
| `report_definition_id` | UUID | No | Related report definition |
| `report_version_number` | INTEGER | No | Definition version used |
| `execution_reference` | VARCHAR(100) | No | Execution reference |
| `requested_by_user_id` | UUID | No | User who requested the report |
| `requested_by_membership_id` | UUID | Yes | Firm membership, for firm executions |
| `requested_by_platform_user_id` | UUID | Yes | Platform administrator, for platform executions |
| `execution_type` | VARCHAR(30) | No | Interactive or scheduled execution |
| `execution_status` | VARCHAR(30) | No | Execution lifecycle status |
| `requested_filters` | JSONB | No | Validated report filter values |
| `execution_parameters` | JSONB | No | Approved execution parameters |
| `result_row_count` | BIGINT | Yes | Number of generated result rows |
| `result_storage_provider` | VARCHAR(50) | Yes | Private result storage provider |
| `result_storage_bucket` | VARCHAR(255) | Yes | Private result storage bucket |
| `result_storage_object_key` | VARCHAR(500) | Yes | Private result storage object key |
| `result_file_format` | VARCHAR(30) | Yes | Generated file format |
| `result_file_size_bytes` | BIGINT | Yes | Generated result size |
| `result_checksum` | VARCHAR(128) | Yes | Result integrity checksum |
| `started_at` | TIMESTAMPTZ | Yes | Execution start time |
| `completed_at` | TIMESTAMPTZ | Yes | Successful completion time |
| `failed_at` | TIMESTAMPTZ | Yes | Failure timestamp |
| `failure_code` | VARCHAR(100) | Yes | Sanitized failure code |
| `failure_summary` | TEXT | Yes | Sanitized failure description |
| `execution_duration_ms` | BIGINT | Yes | Execution duration |
| `correlation_id` | UUID | Yes | Related operation identifier |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `report_definition_id` references `report_definition.id`.
- `firm_id` references `firm.id` when supplied.
- `requested_by_user_id` references `user.id`.
- Firm membership references must use same-firm composite foreign keys.
- Platform administrator references must reference `platform_user.id`.

The execution ownership must be consistent with the referenced report definition and requesting actor.

For firm executions, the report definition must be either:
- A platform-provided report explicitly available to the firm; or
- A report definition owned by the same firm.

**Execution Types**

Initial supported values:

- `interactive`
- `scheduled`
- `background`
- `platform_operation`

**Execution Statuses**

Initial supported values:

- `queued`
- `processing`
- `completed`
- `completed_with_warnings`
- `failed`
- `cancelled`
- `expired`

**Supported Result Formats**

Initial supported values:

- `csv`
- `xlsx`
- `pdf`
- `json`

Only approved formats should be enabled for each report.

**Constraints and Data Integrity**

- Execution reference must not be empty.
- Report version number must be positive.
- Result row count, file size and execution duration must not be negative.
- Requested filters and execution parameters must contain valid JSON objects.
- Completed executions must have a completion timestamp.
- Failed executions must have a failure timestamp.
- Result storage details must be supplied consistently when a file is generated.
- Result checksum must be verified before the result is made available.
- Firm executions must have a valid same-firm membership or an explicitly authorized customer actor, according to the supported report-access model.
- Platform executions must have a valid platform administrator or approved system actor.
- Actor references must follow one unambiguous ownership rule.

**Indexes**

Proposed indexes:

- Unique index on `execution_reference`.
- Index on `(firm_id, created_at DESC)`.
- Index on `(firm_id, report_definition_id, created_at DESC)`.
- Index on `(firm_id, execution_status, created_at)`.
- Index on `(requested_by_user_id, created_at DESC)`.
- Index on `(requested_by_membership_id, created_at DESC)`.
- Index on `(report_definition_id, execution_status)`.
- Index on `(correlation_id)`.
- Index on `(execution_status, created_at)` for authorized background workers.

**Tenant Isolation and RLS**

- Firm report executions must contain the correct `firm_id`.
- RLS must restrict firm users to their own tenant's executions.
- Platform-level executions must use a separately authorized access path.
- Report definition ownership must be validated before execution.
- Report queries must independently enforce tenant isolation.
- Firm users must not retrieve another firm's report results or execution metadata.
- Background workers must establish trusted tenant context.
- Access to generated result files must be authorized independently from access to execution metadata.

**Business and Security Rules**

- Every report request must pass backend permission checks.
- Report execution must use the approved report definition version.
- User-provided filters must be validated against supported filter definitions.
- Arbitrary SQL or unapproved query fragments must never be accepted.
- Large reports must use controlled background processing.
- Result files must be stored in private object storage.
- Public object-storage URLs are prohibited.
- File access must use short-lived authorized download links.
- Sensitive report exports must follow additional authorization and audit requirements.
- Expired results must no longer be accessible.
- Failed executions must preserve sanitized error information.
- Duplicate execution requests must be handled using an appropriate idempotency strategy.
- Report execution and download activity must be auditable.
- Report results must not contain data outside the requester's authorized tenant and record scope.

**Data Integrity and Lifecycle**

- The execution record must be created before background processing begins.
- Execution status must follow controlled lifecycle transitions.
- Completion must occur only after result generation and integrity checks succeed.
- Failed and cancelled executions must preserve their history.
- Generated files must follow configured retention and cleanup policies.
- Historical execution metadata must be retained according to audit requirements.
- Report definition changes must not alter already completed execution records.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must determine which reports execute synchronously and which use background jobs.
- The system must define report execution timeouts and resource limits.
- The system must determine whether result data is stored as a file, database snapshot or both.
- The system must define report result retention and expiration periods.
- The system must establish idempotency and retry rules.
- The system must determine whether users can rerun a historical report with the same filters.
- The system must define scheduled-report execution and ownership.
- The system must establish the actor model for platform, firm, customer and system executions.
- The system must define report download audit requirements.
- The system must define how large report exports are partitioned or streamed.

### 3.78 `data_export_job`

**Purpose**

The `data_export_job` table records and manages data-export requests from the ITR Management System.

It supports controlled export of authorized business records into approved file formats.

Examples include:

- Customer master-data export.
- Customer contact export.
- ITR case-data export.
- Document inventory export.
- Invoice and payment export.
- Refund tracking export.
- Audit and operational data export.
- Authorized data portability requests.

This table manages export requests and their lifecycle. It does not itself authorize access to the underlying records.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned or Platform-Owned, according to the export context.
- Tenant scope: Firm-scoped for firm exports; platform-controlled for platform exports.
- `firm_id` is nullable only for authorized platform-level exports.
- Firm-specific exports must use mandatory tenant RLS.
- Platform-level exports must use a separate restricted access policy.
- A firm administrator may request exports only for their own firm and within their assigned permissions.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | Yes | Owning firm, where applicable |
| `export_reference` | VARCHAR(100) | No | Export request reference |
| `export_type` | VARCHAR(50) | No | Type of data being exported |
| `export_format` | VARCHAR(30) | No | Requested output format |
| `export_scope` | JSONB | No | Approved data selection criteria |
| `export_filters` | JSONB | No | Validated export filters |
| `export_status` | VARCHAR(30) | No | Export lifecycle status |
| `requested_by_user_id` | UUID | No | Requesting user |
| `requested_by_membership_id` | UUID | Yes | Firm membership for firm exports |
| `requested_by_platform_user_id` | UUID | Yes | Platform administrator for platform exports |
| `approval_required` | BOOLEAN | No | Whether additional approval is required |
| `approved_by_membership_id` | UUID | Yes | Firm approver, where applicable |
| `approved_by_platform_user_id` | UUID | Yes | Platform approver, where applicable |
| `approved_at` | TIMESTAMPTZ | Yes | Approval timestamp |
| `rejection_reason` | TEXT | Yes | Reason for rejection |
| `record_count` | BIGINT | Yes | Number of exported records |
| `result_storage_provider` | VARCHAR(50) | Yes | Private storage provider |
| `result_storage_bucket` | VARCHAR(255) | Yes | Private storage bucket |
| `result_storage_object_key` | VARCHAR(500) | Yes | Private storage object key |
| `result_file_size_bytes` | BIGINT | Yes | Generated file size |
| `result_checksum` | VARCHAR(128) | Yes | File integrity checksum |
| `requested_at` | TIMESTAMPTZ | No | Request timestamp |
| `started_at` | TIMESTAMPTZ | Yes | Processing start |
| `completed_at` | TIMESTAMPTZ | Yes | Processing completion |
| `expires_at` | TIMESTAMPTZ | Yes | Download expiration |
| `downloaded_at` | TIMESTAMPTZ | Yes | Latest successful download time |
| `failed_at` | TIMESTAMPTZ | Yes | Failure timestamp |
| `failure_code` | VARCHAR(100) | Yes | Sanitized failure code |
| `failure_summary` | TEXT | Yes | Sanitized failure description |
| `correlation_id` | UUID | Yes | Related operation identifier |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id` when supplied.
- `requested_by_user_id` references `user.id`.
- Firm membership references must use same-firm composite foreign keys.
- Platform administrator references must reference `platform_user.id`.

Approval references must follow the export ownership and approval policy. An export must not be approved by an unauthorized actor or by its requester where segregation of duties is required.

**Export Types**

Initial supported values:

- `customer_master`
- `customer_contact`
- `customer_tax_identity`
- `customer_itr`
- `document_inventory`
- `financial_records`
- `refund_records`
- `audit_records`
- `operational_data`
- `customer_data_portability`
- `other`

**Export Formats**

Initial supported values:

- `csv`
- `xlsx`
- `pdf`
- `json`

Only approved formats should be enabled for each export type.

**Export Statuses**

Initial supported values:

- `requested`
- `pending_approval`
- `approved`
- `rejected`
- `queued`
- `processing`
- `completed`
- `failed`
- `cancelled`
- `expired`

**Constraints and Data Integrity**

- Export reference must not be empty.
- Export type and format must use approved values.
- Export scope and filters must contain valid JSON objects.
- Record count and file size must not be negative.
- Completed exports must have a completion timestamp.
- Expired exports must have an expiration timestamp.
- Approval timestamps and approver references must be consistent.
- Firm exports must have a valid firm actor and tenant context.
- Platform exports must have a valid platform or authorized system actor.
- Storage details must be supplied consistently when a result file exists.
- Export result checksums must be verified before download access is granted.

**Indexes**

Proposed indexes:

- Unique index on `export_reference`.
- Index on `(firm_id, export_status, requested_at DESC)`.
- Index on `(firm_id, export_type, requested_at DESC)`.
- Index on `(requested_by_user_id, requested_at DESC)`.
- Index on `(requested_by_membership_id, requested_at DESC)`.
- Index on `(export_status, requested_at)` for authorized background workers.
- Index on `(expires_at)`.
- Index on `(correlation_id)`.

**Tenant Isolation and RLS**

- Firm-specific exports must contain the correct `firm_id`.
- RLS must restrict firm users to their own firm's export jobs.
- Platform-level exports must use a separate restricted access policy.
- Export filters must be evaluated under the authenticated tenant context.
- Export services must independently verify access to every exported data category.
- Customer tax identity, financial and document data must require appropriate permissions.
- Generated files must not be accessible through public URLs.
- Background workers must establish trusted tenant context.
- Download access must be authorized independently from access to export metadata.

**Business and Security Rules**

- Export requests must pass backend permission checks.
- Sensitive exports must require additional approval where platform or firm policy demands it.
- Export approval must not automatically grant permission to access unrelated records.
- Export filters must use approved fields and operators.
- Arbitrary SQL execution through export configuration is prohibited.
- Data must be restricted to the requester's authorized firm and record scope.
- Sensitive identifiers must be masked or excluded unless full-value export is explicitly authorized.
- Export files must be generated in private object storage.
- Short-lived, authorized download links must be used.
- Download access must be logged.
- Export files must have configurable expiration and secure deletion.
- Failed and cancelled exports must not leave publicly accessible files.
- Export requests and approval decisions must be auditable.
- Customer data portability requests must follow identity verification and applicable legal requirements.
- Exports must not include passwords, authentication tokens, MFA secrets or other credentials.

**Data Integrity and Lifecycle**

- Export metadata must be persisted before processing begins.
- Approval must occur before processing when required.
- Processing must use a consistent and authorized data scope.
- Export results must preserve the filters and export definition used.
- File generation must complete before download access is enabled.
- Expired exports must no longer be downloadable.
- Failed and cancelled exports must preserve their history.
- Generated files must be securely removed according to retention policy.
- Historical export metadata must remain available for audit and compliance purposes.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must define which export categories require approval.
- The system must establish role-specific export permissions.
- The system must define sensitive-field masking and full-value export rules.
- The system must define export file size and record-count limits.
- The system must determine whether large exports require partitioning or streaming.
- The system must define file expiration and retention periods.
- The system must establish download-count and repeated-download policies.
- The system must define customer data portability verification and approval workflows.
- The system must determine whether exports need immutable snapshots for reproducibility.
- The system must define the relationship between `report_execution` and `data_export_job` to avoid duplicating report-export functionality.
- The system must establish export audit-event requirements.

### 3.79 `support_ticket`

**Purpose**

The `support_ticket` table stores support requests submitted by CA firms to the ITR Management System platform support team.

It provides a structured mechanism for reporting technical problems, requesting assistance and tracking support resolution.

Examples include:

- Login and authentication problems.
- Firm configuration assistance.
- Import-processing failures.
- Subscription and billing queries.
- Document upload problems.
- Security incident reporting.
- Feature requests.
- General platform assistance.

This table represents support between a CA firm and the SaaS platform. It is separate from customer conversations maintained within an individual CA firm.

**Ownership and Tenant Scope**

- Ownership: Platform-Owned, with a firm relationship.
- Tenant scope: Firm-associated and platform-controlled.
- `firm_id` identifies the requesting firm.
- Firm administrators may create and view support tickets belonging to their own firm.
- Authorized platform support personnel may access tickets according to their assigned support permissions.
- Firm staff must not access another firm's support tickets.
- Platform support access to firm business data must not be implied by ticket access.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Requesting firm |
| `ticket_reference` | VARCHAR(100) | No | Human-readable ticket reference |
| `ticket_subject` | VARCHAR(255) | No | Support request subject |
| `ticket_description` | TEXT | No | Initial problem description |
| `ticket_category` | VARCHAR(50) | No | Support classification |
| `ticket_priority` | VARCHAR(20) | No | Support priority |
| `ticket_status` | VARCHAR(30) | No | Ticket lifecycle status |
| `requester_user_id` | UUID | No | User who submitted the request |
| `assigned_platform_user_id` | UUID | Yes | Assigned platform support administrator |
| `related_entity_type` | VARCHAR(100) | Yes | Related business or platform entity type |
| `related_entity_id` | UUID | Yes | Related entity identifier |
| `resolution_summary` | TEXT | Yes | Final resolution description |
| `resolved_by_platform_user_id` | UUID | Yes | Platform administrator who resolved the ticket |
| `resolved_at` | TIMESTAMPTZ | Yes | Resolution timestamp |
| `closed_at` | TIMESTAMPTZ | Yes | Closure timestamp |
| `reopened_at` | TIMESTAMPTZ | Yes | Most recent reopening timestamp |
| `first_response_at` | TIMESTAMPTZ | Yes | First support response timestamp |
| `due_at` | TIMESTAMPTZ | Yes | Target resolution or response deadline |
| `created_at` | TIMESTAMPTZ | No | Ticket creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `deleted_at` | TIMESTAMPTZ | Yes | Controlled soft-deletion timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`.
- `requester_user_id` references `user.id`.
- `assigned_platform_user_id` references `platform_user.id`.
- `resolved_by_platform_user_id` references `platform_user.id`.

The requester must have an active and authorized relationship with the requesting firm when creating the ticket.

Firm membership or customer relationship authorization must be validated by the backend.

Related entity references are polymorphic and must be validated against an approved entity catalogue. A related entity must not expose another firm's information.

**Ticket Categories**

Initial supported values:

- `technical_issue`
- `authentication`
- `account_configuration`
- `data_import`
- `document_management`
- `subscription_billing`
- `security_incident`
- `feature_request`
- `general_enquiry`
- `other`

**Ticket Priorities**

Initial supported values:

- `low`
- `normal`
- `high`
- `urgent`
- `critical`

**Ticket Statuses**

Initial supported values:

- `open`
- `assigned`
- `in_progress`
- `waiting_for_firm`
- `waiting_for_platform`
- `resolved`
- `closed`
- `reopened`
- `cancelled`

**Constraints and Data Integrity**

- Ticket reference and subject must not be empty.
- Ticket description must not be empty.
- Category, priority and status must use approved values.
- Requester must be authorized for the associated firm.
- Assigned and resolving platform users must reference valid platform administration records.
- Resolved tickets must have a resolution timestamp and appropriate resolution information.
- Closed tickets must have a closure timestamp.
- Reopened tickets must preserve their previous resolution history.
- A ticket's firm identifier must remain consistent throughout its lifecycle.
- Related entity references must be validated against the requesting firm's authorized records.

**Indexes**

Proposed indexes:

- Unique index on `ticket_reference`.
- Index on `(firm_id, ticket_status, created_at DESC)`.
- Index on `(firm_id, ticket_category, created_at DESC)`.
- Index on `(firm_id, ticket_priority, created_at DESC)`.
- Index on `(requester_user_id, created_at DESC)`.
- Index on `(assigned_platform_user_id, ticket_status, priority)`.
- Index on `(ticket_status, due_at)`.
- Index on `(firm_id, resolved_at DESC)`.

**Tenant Isolation and RLS**

- Every support ticket must contain `firm_id`.
- Firm-facing RLS must restrict access to the requesting firm's tickets.
- Platform support access must use a separate, explicitly authorized policy.
- Platform support personnel must have least-privilege access.
- Access to a support ticket must not automatically grant access to the firm's customers, tax identifiers, documents or financial records.
- Related entity access must be independently authorized.
- All exceptional platform access must be recorded and audited.

**Business and Security Rules**

- Only authorized users may submit support tickets for a firm.
- Platform support staff must be assigned appropriate support permissions.
- Critical security incidents must follow a separate security incident response procedure.
- Sensitive tax information and authentication credentials must not be included unnecessarily in ticket descriptions.
- Attachments must use the approved private document-storage and malware-scanning processes.
- Platform support personnel must not request passwords, OTPs or authentication secrets.
- Support access to production business data must require an approved and auditable exceptional-access process.
- Ticket assignment, priority changes, status changes and resolution must be auditable.
- Support tickets must not be used as a substitute for platform audit or security-event records.
- Customer-facing firm support conversations must remain separate from platform support tickets.

**Data Integrity and Lifecycle**

- Every ticket must have a unique reference.
- Ticket status changes must follow an approved transition process.
- First response, resolution and closure timestamps must reflect actual events.
- Reopening a ticket must preserve previous resolution information.
- Resolved tickets must remain available for historical reference.
- Soft deletion must not silently remove security or compliance evidence.
- Ticket closure and archival must follow retention policies.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must determine whether support tickets can be created by firm staff as well as firm administrators.
- The system must define support SLA targets by priority and category.
- The system must determine whether platform support teams require a separate `platform_support_team` and membership model.
- The system must define ticket escalation and reassignment rules.
- The system must define the relationship between support tickets and platform support cases.
- The system must determine whether security incidents require a dedicated incident-management workflow.
- The system must define customer-visible versus internal support notes.
- The system must establish attachment limits and retention policies.
- The system must define ticket reopening and closure policies.
- The system must establish support access auditing and exceptional production-data access procedures.

### 3.80 `support_ticket_message`

**Purpose**

The `support_ticket_message` table stores individual messages exchanged within a platform support ticket.

It maintains the communication history between the requesting CA firm and authorized platform support personnel.

Examples include:

- Initial support clarification.
- Additional information supplied by the firm.
- Troubleshooting instructions from platform support.
- Technical updates.
- Resolution explanations.
- Follow-up questions.
- Internal platform support notes.

Internal support notes must remain separate from messages visible to the requesting firm.

**Ownership and Tenant Scope**

- Ownership: Platform-Owned, with a firm relationship inherited from the parent support ticket.
- Tenant scope: Firm-associated and platform-controlled.
- Tenant identifier: `firm_id`.
- Mandatory RLS for firm-facing access.
- Every message belongs to exactly one support ticket.
- Firm users may access only messages belonging to tickets they are authorized to view.
- Platform support personnel may access messages according to their support permissions.
- Internal platform notes must never be exposed through ordinary firm-facing APIs.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Associated firm |
| `support_ticket_id` | UUID | No | Parent support ticket |
| `sender_user_id` | UUID | No | Sending user |
| `sender_platform_user_id` | UUID | Yes | Platform administrator, where applicable |
| `message_type` | VARCHAR(30) | No | Message classification |
| `message_visibility` | VARCHAR(30) | No | Customer-facing or internal |
| `message_content` | TEXT | No | Message body |
| `message_status` | VARCHAR(30) | No | Message lifecycle status |
| `reply_to_message_id` | UUID | Yes | Parent message being answered |
| `edited_at` | TIMESTAMPTZ | Yes | Last edit timestamp |
| `sent_at` | TIMESTAMPTZ | Yes | Message sending timestamp |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `deleted_at` | TIMESTAMPTZ | Yes | Controlled soft-deletion timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, support_ticket_id)` references `support_ticket` through a same-firm composite foreign key.
- `sender_user_id` references `user.id`.
- `sender_platform_user_id` references `platform_user.id`.
- `(firm_id, support_ticket_id, reply_to_message_id)` must reference a message belonging to the same ticket, using an appropriate composite self-referencing constraint.

Sender attribution must follow an explicit actor model:

- Firm-side messages must identify an authorized firm user.
- Platform-side messages must identify an authorized platform administrator.
- System-generated messages must use an explicitly defined system actor representation.

The sender must be authorized for the associated support ticket.

**Message Types**

Initial supported values:

- `text`
- `system`
- `internal_note`
- `automated_update`
- `resolution`

**Message Visibility**

Initial supported values:

- `firm_visible`
- `platform_internal`

**Message Statuses**

Initial supported values:

- `draft`
- `sent`
- `failed`
- `deleted`

**Constraints and Data Integrity**

- Message content must not be empty for sent messages.
- Message type, visibility and status must use approved values.
- Sender attribution must identify one valid actor type.
- Platform internal messages must be created by authorized platform personnel or approved system services.
- Firm-visible messages must follow the support ticket's firm authorization rules.
- Reply references must point to messages in the same support ticket.
- Edited and deleted messages must preserve appropriate historical evidence.
- Sent messages must have a sending timestamp.
- A firm user must never be able to create or modify platform-internal messages.

**Indexes**

Proposed indexes:

- Index on `(firm_id, support_ticket_id, sent_at)`.
- Index on `(firm_id, support_ticket_id, message_visibility, sent_at)`.
- Index on `(sender_user_id, created_at DESC)`.
- Index on `(sender_platform_user_id, created_at DESC)`.
- Index on `(support_ticket_id, reply_to_message_id)`.
- Index on `(firm_id, message_status, created_at)`.

**Tenant Isolation and RLS**

- Every message must contain the parent ticket's `firm_id`.
- RLS must restrict firm access to authorized support tickets belonging to the current firm.
- Composite foreign keys must enforce same-firm parent relationships.
- Firm-facing APIs must filter out `platform_internal` messages.
- Platform support access must use separately authorized policies.
- Background services must establish the correct tenant context.
- Message attachments must independently enforce document authorization.
- No firm user may retrieve another firm's support conversation.

**Business and Security Rules**

- Only authorized participants may send messages.
- Firm users may send only firm-visible messages.
- Platform support personnel may send firm-visible replies or platform-internal notes according to their permissions.
- Internal notes must never trigger customer-facing notifications.
- System messages must have identifiable and auditable origins.
- Message content must be validated and safely rendered to prevent stored cross-site scripting (XSS).
- Passwords, OTPs, authentication tokens and other secrets must not be included in messages.
- Sensitive customer tax information must not be unnecessarily copied into support conversations.
- Attachments must use the approved document-management and malware-scanning workflow.
- Editing, deletion and visibility changes must be auditable.
- Message creation and related ticket activity updates must be transactionally consistent.

**Data Integrity and Lifecycle**

- Every message must belong to a valid support ticket.
- Messages must preserve sender and timestamp information.
- Reply relationships must remain within the same ticket.
- Internal notes must retain their restricted visibility.
- Message edits must preserve sufficient historical information for audit purposes.
- Soft deletion must not silently erase required support or compliance evidence.
- Ticket closure must not automatically delete its messages.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must decide whether message edits are allowed after sending.
- A separate `support_ticket_message_history` table may be required for immutable edit history.
- The system must define the exact actor model for firm users, platform users and system-generated messages.
- The system must determine whether multiple participants from one firm can participate in a ticket.
- The system must define attachment limits and supported file types.
- The system must define whether internal notes can mention or reference external support systems.
- The system must establish notification rules for new replies.
- The system must define message retention and archival policies.
- The system must determine whether a separate support-ticket event history table is required.

### 3.81 `support_ticket_attachment`

**Purpose**

The `support_ticket_attachment` table stores metadata linking documents to messages within platform support tickets.

It allows authorized firm users and platform support personnel to exchange supporting files without bypassing the platform's document security controls.

Examples include:

- Screenshots of application errors.
- Sanitized Excel import samples.
- Error reports.
- Approved diagnostic files.
- Configuration examples.
- Supporting documents requested by platform support.

The actual file content must remain in the existing private document-storage system.

**Ownership and Tenant Scope**

- Ownership: Platform-Owned, with a firm relationship inherited from the parent support ticket.
- Tenant scope: Firm-associated and platform-controlled.
- Tenant identifier: `firm_id`.
- Mandatory RLS for firm-facing access.
- Every attachment belongs to one support ticket message.
- Firm users may access only attachments associated with messages they are authorized to view.
- Platform support access must follow separately authorized support permissions.
- Internal-message attachments must never be exposed through firm-facing APIs.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Associated firm |
| `support_ticket_id` | UUID | No | Parent support ticket |
| `support_ticket_message_id` | UUID | No | Message containing the attachment |
| `document_id` | UUID | No | Related document record |
| `document_version_id` | UUID | Yes | Specific document version |
| `attachment_name` | VARCHAR(255) | No | Display name |
| `attachment_description` | TEXT | Yes | Attachment description |
| `display_order` | INTEGER | No | Display sequence |
| `uploaded_by_user_id` | UUID | No | Uploading user |
| `created_at` | TIMESTAMPTZ | No | Attachment creation timestamp |
| `deleted_at` | TIMESTAMPTZ | Yes | Controlled removal timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `(firm_id, support_ticket_id)` references `support_ticket` through a same-firm composite foreign key.
- `(firm_id, support_ticket_id, support_ticket_message_id)` must reference a message belonging to the same ticket.
- `(firm_id, document_id)` references the document record through a same-firm composite foreign key.
- `(firm_id, document_id, document_version_id)` must reference a version belonging to the specified document.
- `uploaded_by_user_id` references `user.id`.

The uploading user must be authorized to send or receive the parent message and access the associated document.

**Constraints and Data Integrity**

- Attachment name must not be empty.
- Display order must be nonnegative.
- Attachment must reference a valid message and document.
- The document must belong to the same firm as the support ticket.
- The document version, when supplied, must belong to the referenced document.
- An attachment must not reference a platform-internal message for firm-visible delivery.
- Duplicate attachment references within the same message must be prevented according to the attachment uniqueness policy.
- Deleted attachments must retain sufficient historical metadata for audit purposes.

**Indexes**

Proposed indexes:

- Index on `(firm_id, support_ticket_id, created_at)`.
- Index on `(firm_id, support_ticket_message_id, display_order)`.
- Index on `(firm_id, document_id)`.
- Index on `(document_id, document_version_id)`.
- Index on `(uploaded_by_user_id, created_at DESC)`.
- Partial index on `(firm_id, support_ticket_id)` where `deleted_at IS NULL`.

**Tenant Isolation and RLS**

- Every attachment must contain `firm_id`.
- RLS must restrict access to the current firm's authorized support messages.
- Composite foreign keys must enforce same-firm relationships.
- Attachment access must be checked against the parent message visibility.
- Platform-internal message attachments must remain restricted to authorized platform personnel.
- Document access must be independently authorized.
- Short-lived authorized download links must be used.
- Public object-storage access is prohibited.

**Business and Security Rules**

- Attachments must be uploaded through authorized support workflows.
- Actual files must use the existing private document-management and storage infrastructure.
- File type, extension, MIME type, signature and size must be validated.
- Malware scanning and quarantine rules must be enforced before attachment availability.
- Untrusted files must not be made available for download.
- Sensitive files must follow the applicable document access classification.
- Attachment downloads must be logged through the document-access auditing mechanism.
- Internal support attachments must not be included in customer-facing notifications.
- Removing an attachment from a message must not automatically delete the underlying document.
- Uploading users must not be able to bypass document permissions through message attachment references.
- Attachment creation, removal and download activity must be auditable.

**Data Integrity and Lifecycle**

- Attachment metadata must be created only after the document has been registered through the document-management workflow.
- Attachment references must remain consistent with the parent message and ticket.
- Document versions must remain identifiable.
- Removal must preserve historical evidence.
- Deleting a support ticket must not automatically delete the underlying documents.
- Document retention and legal-hold policies must take precedence over attachment removal.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must decide whether support attachments should reuse the existing `document` table or use a dedicated platform support-document model.
- The system must define maximum file size and allowed file types.
- The system must establish whether a document can be attached to multiple support messages.
- The system must define attachment replacement and version-selection rules.
- The system must define firm-visible versus platform-internal attachment handling.
- The system must establish retention and deletion policies.
- The system must determine whether platform support attachments require a separate security classification.
- The system must define how attachment access is represented in document-access logs.

### 3.82 `platform_support_case`

**Purpose**

The `platform_support_case` table records privileged platform-level support and investigation cases involving CA firms.

It provides a controlled mechanism for platform administrators and authorized support personnel to investigate technical problems, resolve operational issues and perform approved support activities.

Examples include:

- Tenant configuration investigations.
- Complex Excel import failures.
- Firm account recovery assistance.
- Tenant data integrity investigations.
- Production troubleshooting.
- Security-related support investigations.
- Approved exceptional access to firm data.

This table does not grant access by itself. Every privileged operation must pass a separate authorization and approval process.

**Ownership and Tenant Scope**

- Ownership: Platform-Owned, with an explicit firm relationship.
- Tenant scope: Firm-associated and platform-controlled.
- Tenant identifier: `firm_id`.
- Platform support personnel may access cases according to their assigned permissions.
- Firm administrators may view appropriate case information relating to their own firm.
- Access to tenant business data must remain separately authorized.
- Exceptional access must be justified, time-limited and auditable.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Associated firm |
| `case_reference` | VARCHAR(100) | No | Unique support case reference |
| `case_type` | VARCHAR(50) | No | Support case classification |
| `case_priority` | VARCHAR(20) | No | Case priority |
| `case_status` | VARCHAR(30) | No | Case lifecycle status |
| `case_title` | VARCHAR(255) | No | Case title |
| `case_description` | TEXT | No | Detailed case description |
| `business_justification` | TEXT | No | Reason for platform intervention |
| `requested_by_user_id` | UUID | Yes | Requesting firm user |
| `created_by_platform_user_id` | UUID | Yes | Platform administrator creating the case |
| `assigned_to_platform_user_id` | UUID | Yes | Assigned support administrator |
| `approved_by_platform_user_id` | UUID | Yes | Authorizing platform administrator |
| `approval_status` | VARCHAR(30) | No | Approval state |
| `approval_reason` | TEXT | Yes | Approval justification |
| `approved_at` | TIMESTAMPTZ | Yes | Approval timestamp |
| `started_at` | TIMESTAMPTZ | Yes | Investigation start |
| `resolved_at` | TIMESTAMPTZ | Yes | Resolution timestamp |
| `closed_at` | TIMESTAMPTZ | Yes | Case closure timestamp |
| `resolution_summary` | TEXT | Yes | Resolution details |
| `correlation_id` | UUID | Yes | Related operation correlation |
| `created_at` | TIMESTAMPTZ | No | Creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `deleted_at` | TIMESTAMPTZ | Yes | Controlled soft-deletion timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`.
- `requested_by_user_id` references `user.id`.
- `created_by_platform_user_id` references `platform_user.id`.
- `assigned_to_platform_user_id` references `platform_user.id`.
- `approved_by_platform_user_id` references `platform_user.id`.

The requester must have an authorized relationship with the associated firm.

The case creator, assignee and approver must have appropriate platform permissions.

**Case Types**

Initial supported values:

- `technical_investigation`
- `tenant_configuration`
- `data_integrity`
- `data_import`
- `account_recovery`
- `security_investigation`
- `production_troubleshooting`
- `privileged_data_access`
- `other`

**Case Priorities**

- `low`
- `normal`
- `high`
- `urgent`
- `critical`

**Case Statuses**

- `open`
- `assigned`
- `awaiting_approval`
- `approved`
- `in_progress`
- `waiting_for_firm`
- `waiting_for_platform`
- `resolved`
- `closed`
- `rejected`
- `cancelled`

**Approval Statuses**

- `not_required`
- `pending`
- `approved`
- `rejected`
- `revoked`
- `expired`

**Constraints and Data Integrity**

- Case reference must be unique.
- Case title, description and business justification must not be empty.
- Case type, priority, status and approval status must use approved values.
- Approval records must identify the approving platform administrator.
- The case creator and approver should be different individuals for privileged production-data access.
- A case cannot enter an approved state without the required approval.
- An investigation cannot start before required approvals are completed.
- Closed cases must have a closure timestamp.
- Resolved cases must have a resolution summary.
- Approval revocation or expiry must prevent subsequent privileged operations.
- Case status changes must be recorded in an immutable history mechanism.

**Indexes**

Proposed indexes:

- Unique index on `case_reference`.
- Index on `(firm_id, case_status, created_at DESC)`.
- Index on `(firm_id, case_type, created_at DESC)`.
- Index on `(case_priority, case_status)`.
- Index on `(assigned_to_platform_user_id, case_status)`.
- Index on `(approval_status, created_at)`.
- Index on `(created_by_platform_user_id, created_at DESC)`.
- Index on `correlation_id`.

**Tenant Isolation and RLS**

- Every case must identify the associated firm.
- Firm administrators may access only cases associated with their own firm.
- Platform support personnel must use separately authorized platform-level access.
- Firm-level RLS must not be bypassed merely because a platform support case exists.
- A support case does not automatically grant permission to inspect customer tax records, documents or financial information.
- Any approved exceptional access must be restricted to the specific firm, resources, purpose and time period.
- All privileged operations must be associated with the relevant support case.

**Business and Security Rules**

- Platform support cases must have a documented business justification.
- Privileged access must require explicit approval from an authorized independent platform administrator.
- The requester, operator and approver must be separately identifiable.
- Emergency access must follow a documented break-glass procedure.
- Emergency access must be time-limited and generate immediate security alerts.
- Access must be restricted to the minimum data necessary.
- Production data must not be copied into local development environments without an approved data-handling process.
- Support personnel must not use customer credentials or impersonate customers to bypass authorization.
- Sensitive tax identifiers and financial records must remain protected.
- Every privileged read, modification, export or administrative action must generate appropriate audit evidence.
- Revoking or closing a support case must terminate any associated temporary access grants.
- Support case approval must never replace ordinary backend authorization checks.

**Data Integrity and Lifecycle**

- Case creation, approval, assignment, resolution and closure must be recorded.
- Approval decisions must remain historically traceable.
- A rejected or revoked approval must not be overwritten.
- Case closure must not delete associated audit or security records.
- Privileged-access evidence must be retained according to the security and compliance retention policy.
- Sensitive case descriptions and attachments must have restricted access.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must define the exact approval matrix for different privileged operations.
- A separate `privileged_access_grant` table will likely be required to record approved temporary access.
- A separate `platform_support_case_history` table may be required for immutable case lifecycle history.
- The system must define emergency access duration and notification requirements.
- The system must determine whether approval requires one or two independent platform administrators.
- The system must define whether firms can view all case details or only customer-facing updates.
- The system must define support case retention and archival policies.
- The system must establish how support cases connect to `support_ticket`, `privileged_access_event`, `platform_audit_event` and `platform_security_event`.

### 3.83 `audit_event`

**Purpose**

The `audit_event` table stores an immutable record of significant business, administrative, financial and security-relevant operations performed within a CA firm's tenant.

It provides evidence of who performed an action, what operation occurred, which business record was affected, when it happened and whether the operation succeeded.

Examples include:

- Customer creation, modification and deactivation.
- Customer PAN creation, viewing, modification and deletion.
- ITR record creation and status changes.
- Document upload, download, review and deletion.
- Checklist updates and approvals.
- Invoice creation, issue, cancellation and correction.
- Payment receipt, allocation, reversal and refund.
- Staff membership and role changes.
- Customer access grant creation and revocation.
- Financial exports and sensitive data access.
- Changes to firm configuration and permissions.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Strictly firm-specific.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL Row-Level Security (RLS).
- Every firm-level audit event must belong to exactly one firm.
- Firm administrators may retrieve authorized audit records belonging to their firm.
- Firm staff may access audit information only when explicitly permitted.
- Customers must not access internal firm audit records.
- Platform administrators must use separately authorized platform-level audit access.
- Audit records must be append-only.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Tenant identifier |
| `actor_type` | VARCHAR(30) | No | Actor classification |
| `actor_user_id` | UUID | Yes | User who performed the operation |
| `actor_membership_id` | UUID | Yes | Firm membership associated with the actor |
| `actor_customer_id` | UUID | Yes | Customer identity associated with the actor |
| `event_category` | VARCHAR(50) | No | Audit event category |
| `event_type` | VARCHAR(100) | No | Specific event |
| `operation` | VARCHAR(50) | No | Operation performed |
| `target_entity_type` | VARCHAR(100) | Yes | Affected entity type |
| `target_entity_id` | UUID | Yes | Affected record identifier |
| `event_description` | TEXT | Yes | Human-readable description |
| `event_data` | JSONB | No | Structured event metadata |
| `outcome` | VARCHAR(30) | No | Operation result |
| `ip_address` | INET | Yes | Originating IP address |
| `user_agent` | TEXT | Yes | Client information |
| `correlation_id` | UUID | Yes | Related request or workflow |
| `occurred_at` | TIMESTAMPTZ | No | Time the event occurred |
| `created_at` | TIMESTAMPTZ | No | Time the audit record was persisted |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`.
- `(firm_id, actor_membership_id)` references `firm_membership` through a same-firm composite foreign key.
- `actor_user_id` references `user.id`.
- `actor_customer_id` references `customer.id`.

Actor attribution must follow an explicit actor model:

- `firm_user`: requires a valid user and firm membership belonging to the same firm.
- `customer`: requires a valid customer identity and an authorized relationship with the firm.
- `system`: identifies an approved system process or service.
- `platform_user`: requires a separately authorized platform actor and must be used only for approved platform-originated operations affecting the firm.

The actor type must be consistent with the populated actor references. The design must prevent ambiguous or contradictory actor attribution.

**Event Categories**

Initial supported values:

- `authentication`
- `authorization`
- `customer_management`
- `itr_management`
- `document_management`
- `checklist_management`
- `communication`
- `financial`
- `refund`
- `membership`
- `role_management`
- `configuration`
- `data_import`
- `data_export`
- `subscription`
- `support`
- `security`
- `system`
- `other`

**Operations**

Initial supported values:

- `create`
- `view`
- `update`
- `delete`
- `download`
- `upload`
- `approve`
- `reject`
- `assign`
- `revoke`
- `export`
- `login`
- `logout`
- `permission_change`
- `status_change`
- `other`

**Outcome Values**

- `success`
- `failure`
- `denied`
- `pending`

**Constraints and Data Integrity**

- `firm_id`, actor type, event category, event type, operation, outcome and timestamps must be populated.
- `event_data` must be a valid JSON object.
- Event type and operation values must be non-empty and follow the approved event catalogue.
- Actor references must be consistent with the actor type.
- Firm-user membership references must belong to the same firm.
- Customer actors must have an authorized relationship with the firm at the time of the operation.
- Failed and denied operations must be recorded wherever sufficient trusted context exists.
- Events must not be silently overwritten or deleted.
- Corrections must be represented through additional audit events.
- Event identifiers and correlation identifiers must support reliable traceability and idempotency.

**Indexes**

Proposed indexes:

- Index on `(firm_id, occurred_at DESC)`.
- Index on `(firm_id, actor_type, occurred_at DESC)`.
- Index on `(firm_id, actor_membership_id, occurred_at DESC)`.
- Index on `(firm_id, actor_customer_id, occurred_at DESC)`.
- Index on `(firm_id, event_category, occurred_at DESC)`.
- Index on `(firm_id, event_type, occurred_at DESC)`.
- Index on `(firm_id, target_entity_type, target_entity_id)`.
- Index on `(firm_id, correlation_id)`.
- Index on `(firm_id, outcome, occurred_at DESC)`.

**Tenant Isolation and RLS**

- Every event must have a valid `firm_id`.
- RLS must restrict retrieval to the current authorized firm.
- Firm context must be established transactionally for every database operation.
- Cross-firm references must be rejected.
- Platform-wide audit events must be stored separately in `platform_audit_event`.
- Firm users must not access another firm's audit records.
- Audit retrieval and export operations must themselves be audited.
- The application runtime must not have unrestricted access to other tenants' audit data.

**Business and Security Rules**

- Audit records must be generated by trusted backend services, not directly by client-supplied event payloads.
- Audit events must be recorded for security-sensitive and financially significant operations.
- Sensitive identifier viewing, including PAN reveal, must be auditable.
- Document downloads and sensitive exports must generate audit evidence.
- Role, permission and access-grant changes must be recorded.
- Failed authorization attempts must be logged where safely possible.
- Audit records must not contain passwords, OTPs, session tokens, recovery codes, encryption keys or unnecessary personal/tax information.
- Sensitive metadata must be masked or minimized.
- Application users must not be able to modify or delete audit records.
- Database permissions must enforce append-only behavior.
- Audit retrieval must be restricted and monitored.
- High-value audit evidence should be periodically copied to independently protected storage.

**Data Integrity and Lifecycle**

- Audit events must be persisted in the same transaction as the relevant business operation whenever practical and necessary for consistency.
- Failed operations must be recorded through a reliable separate mechanism when the original transaction rolls back.
- Event timestamps must use UTC.
- Audit records must remain available for the approved retention period.
- Retention and archival must comply with applicable legal and contractual requirements.
- Audit records must not be removed merely because a related business record is soft-deleted.
- High-volume audit data may require time-based partitioning.
- Database backups and independent audit archives must be included in recovery planning.

**Design Notes and Open Decisions**

- The system must finalize the actor model, particularly system and platform-originated actions.
- The exact event catalogue must be maintained centrally.
- The system must determine which read operations require mandatory audit records.
- The system must define retention periods by event category.
- The system must decide whether audit events are partitioned monthly or quarterly.
- The system must define tamper-resistant external audit storage and integrity verification.
- The system must define audit-event idempotency and duplicate prevention.
- The system must determine whether business audit records and security-event records share any common event infrastructure.
- The system must establish how audit export and privileged audit access are approved and recorded.

### 3.84 `security_event`

**Purpose**

The `security_event` table stores security-relevant events detected within a CA firm's tenant.

It supports security monitoring, investigation, incident response and detection of suspicious activity.

Examples include:

- Repeated failed login attempts.
- Suspicious authentication patterns.
- Unauthorized access attempts.
- Customer or staff account lockouts.
- Unusual document download activity.
- Repeated access-denied events.
- Suspicious customer data exports.
- MFA verification failures.
- Potential session hijacking.
- Attempts to access another firm's resources.

**Ownership and Tenant Scope**

- Ownership: Firm-Owned.
- Tenant scope: Firm-specific.
- Tenant identifier: `firm_id`.
- Mandatory PostgreSQL RLS.
- Every firm-attributable security event must belong to one firm.
- Firm administrators may view authorized security events belonging to their firm.
- Firm staff access must be restricted to explicitly authorized security permissions.
- Customers must not access internal security-event records.
- Platform-wide events must be recorded separately in `platform_security_event`.
- Security events must be append-only.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Associated firm |
| `event_type` | VARCHAR(100) | No | Security event classification |
| `event_category` | VARCHAR(50) | No | Security event category |
| `severity` | VARCHAR(20) | No | Risk severity |
| `event_status` | VARCHAR(30) | No | Investigation status |
| `actor_user_id` | UUID | Yes | Associated user |
| `actor_membership_id` | UUID | Yes | Associated firm membership |
| `actor_customer_id` | UUID | Yes | Associated customer identity |
| `affected_user_id` | UUID | Yes | User affected by the event |
| `affected_entity_type` | VARCHAR(100) | Yes | Affected entity type |
| `affected_entity_id` | UUID | Yes | Affected entity identifier |
| `ip_address` | INET | Yes | Source IP address |
| `user_agent` | TEXT | Yes | Client information |
| `event_description` | TEXT | No | Security event description |
| `event_data` | JSONB | No | Structured event metadata |
| `detection_source` | VARCHAR(50) | No | Event detection mechanism |
| `correlation_id` | UUID | Yes | Related operation identifier |
| `detected_at` | TIMESTAMPTZ | No | Detection timestamp |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`.
- `(firm_id, actor_membership_id)` references `firm_membership` through a same-firm composite foreign key.
- `actor_user_id` references `user.id`.
- `actor_customer_id` references `customer.id`.
- `affected_user_id` references `user.id`.

Actor attribution must distinguish firm users, customers, system processes and unknown external actors.

Events originating from an unauthenticated request must still be recordable without inventing a user identity.

**Event Categories**

Initial supported values:

- `authentication`
- `authorization`
- `account_security`
- `session_security`
- `mfa_security`
- `data_access`
- `document_security`
- `data_export`
- `tenant_isolation`
- `configuration_security`
- `suspicious_activity`
- `other`

**Example Event Types**

- `repeated_login_failures`
- `invalid_password_attempt`
- `mfa_verification_failure`
- `account_locked`
- `session_revoked`
- `suspicious_session_activity`
- `unauthorized_resource_access`
- `cross_tenant_access_attempt`
- `suspicious_document_download`
- `sensitive_data_export`
- `privilege_escalation_attempt`
- `account_recovery_attempt`
- `suspicious_activity_detected`

**Severity Values**

- `info`
- `low`
- `medium`
- `high`
- `critical`

**Event Statuses**

- `new`
- `under_investigation`
- `confirmed`
- `false_positive`
- `resolved`
- `closed`

**Constraints and Data Integrity**

- Event type, category, severity and detection source must use approved values.
- Event description must not be empty.
- `event_data` must be a valid JSON object.
- Actor references must be consistent with the event's origin.
- Firm membership references must belong to the same firm.
- Security events must preserve their original detection timestamp.
- Event status changes must be recorded in a separate immutable investigation history.
- Security event records must not be silently modified or deleted.
- Unknown actors must remain distinguishable from system-generated events.

**Indexes**

Proposed indexes:

- Index on `(firm_id, detected_at DESC)`.
- Index on `(firm_id, severity, detected_at DESC)`.
- Index on `(firm_id, event_category, detected_at DESC)`.
- Index on `(firm_id, event_type, detected_at DESC)`.
- Index on `(firm_id, event_status, detected_at DESC)`.
- Index on `(firm_id, actor_user_id, detected_at DESC)`.
- Index on `(firm_id, affected_user_id, detected_at DESC)`.
- Index on `(firm_id, ip_address, detected_at DESC)`.
- Index on `(firm_id, correlation_id)`.

**Tenant Isolation and RLS**

- Every firm-attributable event must have a valid `firm_id`.
- RLS must restrict firm-level access to the current tenant.
- Security monitoring services must establish the correct firm context.
- Cross-tenant access attempts must be attributable to the target firm where safely possible.
- A firm must not be able to inspect security events belonging to another firm.
- Platform-wide security events must be stored separately.
- Security-event retrieval must require explicit authorization and be audited.

**Business and Security Rules**

- Security events must be generated by trusted authentication, authorization and monitoring services.
- Repeated failures and suspicious patterns should trigger appropriate detection rules.
- Critical security events must generate alerts to authorized personnel.
- Security-event records must not include passwords, OTPs, access tokens, recovery codes or encryption secrets.
- Sensitive values must be masked or excluded.
- Event detection must not rely solely on client-supplied information.
- Security investigation must preserve the original event.
- Confirmed incidents must follow the organization's incident-response process.
- Security events must not automatically grant access to affected customer data.
- Automated detection and response must be controlled and auditable.

**Data Integrity and Lifecycle**

- Event creation must be reliable even when the triggering business operation fails.
- Security records must be append-only.
- Investigation status and resolution details must be maintained separately from original event data.
- Timestamps must use UTC.
- High-volume event data may require time-based partitioning.
- Retention and archival must follow the approved security and legal retention policy.
- Security-event storage must be included in backup and recovery planning.

**Design Notes and Open Decisions**

- The system must define security-event detection thresholds.
- The system must define severity classification rules.
- A separate `security_event_history` table may be required for investigation updates.
- The system must determine which events require immediate alerts.
- The system must define event correlation and deduplication strategies.
- The system must determine whether automated account suspension or session revocation is appropriate for particular event types.
- The system must define security-event retention and archival periods.
- The system must establish how security events relate to `audit_event`, `platform_security_event` and incident-response workflows.

### 3.85 `platform_security_event`

**Purpose**

The `platform_security_event` table records security-related events affecting the overall ITR Management SaaS platform.

It supports platform-level security monitoring, incident investigation, threat detection and response.

Examples include:

- Suspicious platform administrator login attempts.
- Repeated failed platform authentication.
- Attempts to bypass platform-level authorization.
- Unauthorized access to platform administration functions.
- Suspicious tenant provisioning or suspension activity.
- Platform-wide configuration tampering attempts.
- Privileged account compromise indicators.
- Attempts to access multiple firms without authorization.
- Infrastructure or integration security incidents.
- Suspected platform-wide data exposure.

**Ownership and Tenant Scope**

- Ownership: Platform-Owned.
- Tenant scope: Global platform scope.
- Tenant identifier: No mandatory `firm_id`.
- Platform-level restricted access policy.
- This table must not use ordinary firm-level RLS.
- Access must be restricted to specifically authorized platform security personnel.
- Firm administrators and ordinary firm users must not access platform security events.
- A firm identifier may be recorded as contextual information when an event relates to a particular firm.
- Platform security records must be append-only.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `event_type` | VARCHAR(100) | No | Security event classification |
| `event_category` | VARCHAR(50) | No | Security event category |
| `severity` | VARCHAR(20) | No | Risk severity |
| `event_status` | VARCHAR(30) | No | Investigation status |
| `actor_type` | VARCHAR(30) | No | Actor classification |
| `actor_user_id` | UUID | Yes | Associated central user |
| `actor_platform_user_id` | UUID | Yes | Associated platform administrator |
| `affected_firm_id` | UUID | Yes | Related firm, if applicable |
| `affected_entity_type` | VARCHAR(100) | Yes | Affected entity type |
| `affected_entity_id` | UUID | Yes | Affected entity identifier |
| `ip_address` | INET | Yes | Source IP address |
| `user_agent` | TEXT | Yes | Client information |
| `event_description` | TEXT | No | Security event description |
| `event_data` | JSONB | No | Structured event metadata |
| `detection_source` | VARCHAR(50) | No | Event detection mechanism |
| `correlation_id` | UUID | Yes | Related operation identifier |
| `detected_at` | TIMESTAMPTZ | No | Detection timestamp |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `actor_user_id` references `user.id`.
- `actor_platform_user_id` references `platform_user.id`.
- `affected_firm_id` references `firm.id`.

Actor attribution must distinguish:

- `platform_user`: authenticated platform administrator.
- `firm_user`: user associated with a firm.
- `customer`: customer identity.
- `system`: trusted platform process.
- `external`: unidentified or unauthenticated external actor.

The actor type must be consistent with the available actor references.

Unknown external actors must be recordable without requiring an existing user record.

**Event Categories**

Initial supported values:

- `platform_authentication`
- `platform_authorization`
- `platform_account_security`
- `privileged_access`
- `tenant_isolation`
- `platform_configuration`
- `infrastructure_security`
- `integration_security`
- `data_protection`
- `suspicious_activity`
- `incident_response`
- `other`

**Example Event Types**

- `platform_login_failure`
- `platform_admin_account_locked`
- `platform_mfa_failure`
- `privileged_access_attempt`
- `unauthorized_platform_operation`
- `cross_tenant_access_attempt`
- `tenant_provisioning_anomaly`
- `platform_configuration_tampering`
- `suspicious_admin_activity`
- `platform_data_exposure_suspected`
- `integration_credential_compromise`
- `infrastructure_security_alert`

**Severity Values**

- `info`
- `low`
- `medium`
- `high`
- `critical`

**Event Statuses**

- `new`
- `under_investigation`
- `confirmed`
- `false_positive`
- `resolved`
- `closed`

**Constraints and Data Integrity**

- Event type, category, severity, actor type and detection source must use approved values.
- Event description must not be empty.
- `event_data` must be a valid JSON object.
- Actor references must be consistent with actor type.
- `affected_firm_id`, when supplied, must reference an existing firm.
- Platform security events must not depend on successful user authentication.
- Original detection details and timestamps must remain immutable.
- Investigation updates must be recorded separately.
- Duplicate events must be managed through a controlled event-correlation or deduplication mechanism.

**Indexes**

Proposed indexes:

- Index on `(detected_at DESC)`.
- Index on `(severity, detected_at DESC)`.
- Index on `(event_category, detected_at DESC)`.
- Index on `(event_type, detected_at DESC)`.
- Index on `(event_status, detected_at DESC)`.
- Index on `(actor_platform_user_id, detected_at DESC)`.
- Index on `(actor_user_id, detected_at DESC)`.
- Index on `(affected_firm_id, detected_at DESC)`.
- Index on `(ip_address, detected_at DESC)`.
- Index on `correlation_id`.

**Platform-Level Access Control**

- Access must be restricted to authorized platform security roles.
- Platform administrators without security permissions must not automatically receive unrestricted access.
- Security-event retrieval and exports must be audited.
- Database permissions must prevent ordinary application roles from modifying or deleting security records.
- Platform-level RLS or equivalent restricted database policies must be evaluated separately from firm-level RLS.
- Emergency access must follow the approved privileged-access procedure.

**Business and Security Rules**

- Events must be generated by trusted platform authentication, authorization, monitoring and infrastructure services.
- Critical events must trigger appropriate alerts.
- Cross-tenant access attempts must be recorded even when authorization denies the operation.
- Platform administrator activity must be monitored.
- Sensitive configuration values, passwords, tokens, encryption keys and recovery codes must never be stored in event payloads.
- Security investigations must preserve original event evidence.
- Confirmed incidents must follow a documented incident-response process.
- Platform security records must not be exposed through firm-facing APIs.
- Automated responses must be authorized, controlled and auditable.

**Data Integrity and Lifecycle**

- Security-event creation must be reliable even when the triggering operation fails.
- Original event records must be append-only.
- Investigation status changes must be separately recorded.
- Timestamps must use UTC.
- High-volume records may require time-based partitioning.
- Security records must be protected through backup and recovery procedures.
- Retention and archival must follow the approved security and legal retention policy.
- Critical evidence may require independent tamper-resistant storage.

**Design Notes and Open Decisions**

- The system must define platform security event detection rules and severity thresholds.
- A separate `platform_security_event_history` table may be required.
- The system must define alerting and escalation rules for critical events.
- The system must define event correlation and deduplication.
- The system must decide whether security incidents require a separate incident-management entity.
- The system must define retention, archival and tamper-resistance requirements.
- The system must establish the relationship between platform security events, platform audit events and privileged access records.
- The system must define the precise database policy for platform-level security records.

### 3.86 `privileged_access_event`

**Purpose**

The `privileged_access_event` table records privileged access to sensitive tenant resources by platform administrators or other specially authorized personnel.

It provides traceability for exceptional access granted during support investigations, production troubleshooting, security incidents and approved administrative operations.

Examples include:

- Temporary access to a firm's customer records.
- Approved inspection of sensitive tax information.
- Access to a firm's documents for troubleshooting.
- Exceptional production database investigation.
- Emergency access during a security incident.
- Privileged data export for an approved investigation.
- Revocation or expiry of temporary access.

This table records privileged-access activity. It does not itself grant access or replace authorization controls.

**Ownership and Tenant Scope**

- Ownership: Platform-Owned, with an explicit firm relationship.
- Tenant scope: Firm-associated and platform-controlled.
- Tenant identifier: `firm_id`.
- Platform-level restricted access policy.
- No ordinary firm-level RLS access.
- Firm administrators may view approved summaries relating to their own firm where policy permits.
- Only authorized platform security and compliance personnel may retrieve complete privileged-access evidence.
- Every privileged operation must be linked to an approved support case or authorized emergency procedure.
- Records must be append-only.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Firm whose resources are accessed |
| `platform_support_case_id` | UUID | Yes | Related approved support case |
| `platform_user_id` | UUID | No | Platform administrator performing the action |
| `access_grant_reference` | UUID | No | Identifier of the associated access grant |
| `access_type` | VARCHAR(50) | No | Type of privileged access |
| `resource_type` | VARCHAR(100) | No | Accessed resource classification |
| `resource_id` | UUID | Yes | Specific resource identifier |
| `access_action` | VARCHAR(50) | No | Operation performed |
| `access_purpose` | TEXT | No | Business justification |
| `approval_status` | VARCHAR(30) | No | Approval state |
| `access_result` | VARCHAR(30) | No | Operation result |
| `ip_address` | INET | Yes | Originating IP address |
| `user_agent` | TEXT | Yes | Client information |
| `correlation_id` | UUID | Yes | Related operation identifier |
| `occurred_at` | TIMESTAMPTZ | No | Time of privileged action |
| `created_at` | TIMESTAMPTZ | No | Record persistence timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`.
- `platform_support_case_id` references `platform_support_case.id`.
- `platform_user_id` references `platform_user.id`.

A separate `privileged_access_grant` table must be introduced to manage the actual authorization grant.

The access grant must identify:

- The authorized platform administrator.
- The affected firm.
- The approved support case or emergency justification.
- The permitted resource scope.
- The permitted operations.
- The approval authority.
- The grant start and expiry times.
- The revocation status.

The `access_grant_reference` must reference a valid grant. The database design must finalize its foreign key once the grant table is defined.

**Access Types**

Initial supported values:

- `read_only`
- `sensitive_data_read`
- `document_access`
- `data_export`
- `configuration_access`
- `administrative_operation`
- `emergency_access`

**Resource Types**

Initial supported values:

- `customer`
- `customer_tax_identity`
- `customer_itr`
- `document`
- `financial_record`
- `firm_configuration`
- `membership`
- `audit_record`
- `other`

**Access Actions**

- `view`
- `read`
- `download`
- `export`
- `update`
- `approve`
- `revoke`
- `other`

**Approval Statuses**

- `approved`
- `emergency_authorized`
- `denied`

**Access Results**

- `success`
- `denied`
- `failure`

**Constraints and Data Integrity**

- Every event must identify the affected firm and acting platform administrator.
- Access purpose must not be empty.
- Access type, resource type, action, approval status and result must use approved values.
- The access grant must belong to the same firm and platform administrator.
- The support case, when supplied, must belong to the same firm.
- The requested action must fall within the approved grant scope.
- Access outside the approved time window must be denied.
- Expired or revoked grants must not authorize further operations.
- Emergency access must have an identifiable emergency justification.
- Failed and denied privileged-access attempts must be recorded.
- Event records must not be modified or deleted.

**Indexes**

Proposed indexes:

- Index on `(firm_id, occurred_at DESC)`.
- Index on `(platform_user_id, occurred_at DESC)`.
- Index on `(access_grant_reference, occurred_at DESC)`.
- Index on `(platform_support_case_id, occurred_at DESC)`.
- Index on `(firm_id, resource_type, resource_id)`.
- Index on `(firm_id, access_action, occurred_at DESC)`.
- Index on `(approval_status, occurred_at DESC)`.
- Index on `(access_result, occurred_at DESC)`.
- Index on `correlation_id`.

**Privileged Access Control**

- A privileged-access event must never independently authorize an operation.
- The backend must validate an active grant before each privileged action.
- Grant scope must be limited to the approved firm, resource and permitted actions.
- Access must be time-limited and automatically expire.
- Approval must be independent of the operator for sensitive production-data access.
- Emergency access must generate immediate alerts and receive retrospective review.
- Access revocation must take effect promptly.
- All sensitive reads, modifications and exports must be recorded.
- Platform administrators must not use privileged access to bypass tenant isolation.
- Privileged actions must be associated with a support case or documented emergency authorization.

**Tenant Isolation and Security Rules**

- Every privileged event must identify the firm whose data or resources are involved.
- Cross-firm access must be denied unless separately authorized through an explicitly approved process.
- Platform support access must not create a permanent tenant membership.
- Firm-level RLS must remain enforced for normal application operations.
- Any exceptional access mechanism must use a narrowly scoped and controlled database authorization path.
- Sensitive tax identifiers must not be copied into event descriptions or metadata.
- Passwords, OTPs, access tokens, encryption keys and other secrets must never be recorded.
- Retrieval of privileged-access evidence must itself be audited.
- Database permissions must prevent unauthorized modification or deletion.

**Data Integrity and Lifecycle**

- Privileged-access events must be recorded through trusted backend services.
- Event creation must be reliable even when the privileged operation fails.
- Grant creation, approval, revocation and expiry must be traceable.
- Original event timestamps and actor information must remain immutable.
- All timestamps must use UTC.
- Records must follow the approved security and compliance retention policy.
- High-value privileged-access evidence should be copied to independently protected storage.
- Backup and recovery procedures must include privileged-access evidence.

**Design Notes and Open Decisions**

- The separate `privileged_access_grant` table must be fully specified.
- The exact approval matrix must be finalized.
- The system must define the maximum access-grant duration.
- Emergency access authorization and retrospective review procedures must be defined.
- The system must decide whether privileged access requires two-person approval.
- The exact resource and operation catalogue must be maintained centrally.
- The system must define whether sensitive reads require individual event records or controlled aggregation.
- The system must define alerting for unusual privileged-access patterns.
- The relationship between this table, `platform_support_case`, `platform_audit_event` and `platform_security_event` must be finalized.
- Retention, archival and tamper-resistant evidence requirements must be established.

### 3.87 `background_job_execution`

**Purpose**

The `background_job_execution` table stores execution records for asynchronous background jobs processed by the ITR Management System.

It provides operational visibility into job execution, processing attempts, failures, retries and completion.

Examples include:

- Excel customer data imports.
- Document malware scanning.
- Report generation.
- Data exports.
- Email and SMS notification processing.
- Invoice and payment reconciliation.
- Refund status synchronization.
- Checklist generation.
- Subscription usage calculations.
- Background data maintenance.
- Automated workflow processing.

This table records execution history and operational metadata. It is not itself the job queue or the authoritative business record.

**Ownership and Tenant Scope**

- Ownership: System-Owned, with an optional firm relationship.
- Tenant scope: Firm-associated or platform-wide.
- Tenant identifier: `firm_id`, nullable for platform-wide jobs.
- Firm-associated records must be protected by tenant-aware RLS.
- Platform-wide job records require separately restricted platform-level access.
- Workers must establish the correct tenant context before processing firm-specific jobs.
- Job execution records must be retained according to operational and audit requirements.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | Yes | Associated firm |
| `job_reference` | VARCHAR(150) | No | Unique job reference |
| `job_type` | VARCHAR(100) | No | Job classification |
| `job_name` | VARCHAR(255) | No | Human-readable job name |
| `job_queue` | VARCHAR(100) | No | Processing queue |
| `job_status` | VARCHAR(30) | No | Current execution status |
| `job_priority` | VARCHAR(20) | No | Processing priority |
| `job_payload` | JSONB | No | Validated job metadata |
| `job_result` | JSONB | Yes | Processing result metadata |
| `idempotency_key` | VARCHAR(255) | Yes | Duplicate execution protection |
| `correlation_id` | UUID | Yes | Related workflow identifier |
| `parent_job_id` | UUID | Yes | Parent job reference |
| `attempt_number` | INTEGER | No | Current attempt number |
| `maximum_attempts` | INTEGER | No | Maximum permitted attempts |
| `worker_identifier` | VARCHAR(150) | Yes | Worker processing the job |
| `requested_by_user_id` | UUID | Yes | Requesting user |
| `requested_by_membership_id` | UUID | Yes | Requesting firm membership |
| `started_at` | TIMESTAMPTZ | Yes | Execution start |
| `completed_at` | TIMESTAMPTZ | Yes | Execution completion |
| `failed_at` | TIMESTAMPTZ | Yes | Failure timestamp |
| `next_retry_at` | TIMESTAMPTZ | Yes | Scheduled retry time |
| `execution_duration_ms` | BIGINT | Yes | Execution duration |
| `error_code` | VARCHAR(100) | Yes | Structured failure code |
| `error_summary` | TEXT | Yes | Sanitized failure summary |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`, when applicable.
- `parent_job_id` references `background_job_execution.id`.
- `requested_by_user_id` references `user.id`.
- `(firm_id, requested_by_membership_id)` references `firm_membership` through a same-firm composite foreign key.

For firm-associated jobs, the requesting membership must belong to the same firm.

Platform-wide jobs must use a separately validated platform or system actor representation.

**Example Job Types**

- `customer_import`
- `document_scan`
- `report_generation`
- `data_export`
- `notification_delivery`
- `payment_reconciliation`
- `refund_synchronization`
- `checklist_generation`
- `subscription_usage_calculation`
- `workflow_processing`
- `data_maintenance`
- `other`

**Job Statuses**

- `queued`
- `scheduled`
- `processing`
- `completed`
- `completed_with_warnings`
- `failed`
- `retry_scheduled`
- `cancelled`
- `dead_letter`

**Job Priorities**

- `low`
- `normal`
- `high`
- `urgent`

**Constraints and Data Integrity**

- Job reference must be unique.
- Job type, queue, status and priority must use approved values.
- `job_payload` and `job_result`, when supplied, must be valid JSON objects.
- Attempt number must be nonnegative.
- Maximum attempts must be positive.
- Attempt number must not exceed the permitted execution policy.
- Execution timestamps must follow a valid lifecycle.
- Completion and failure states must have corresponding timestamps.
- Execution duration must be nonnegative.
- Parent job references must not create circular relationships.
- Idempotency keys must be unique within their defined scope.
- Failed jobs must retain sanitized failure information.
- Job payloads must not contain plaintext passwords, OTPs, session tokens or encryption secrets.

**Indexes**

Proposed indexes:

- Unique index on `job_reference`.
- Index on `(firm_id, job_status, created_at DESC)`.
- Index on `(job_queue, job_status, created_at)`.
- Index on `(job_type, job_status, created_at DESC)`.
- Index on `(job_priority, job_status, created_at)`.
- Index on `(next_retry_at, job_status)`.
- Index on `(correlation_id)`.
- Index on `(parent_job_id)`.
- Index on `(requested_by_user_id, created_at DESC)`.
- Partial index for jobs awaiting processing or retry.

**Business and Security Rules**

- Background workers must obtain jobs through the approved queue-processing mechanism.
- The database execution record must not be treated as a replacement for the actual queue.
- Workers must validate job type and payload before processing.
- Firm-specific jobs must execute within the correct tenant context.
- A worker must never reuse a previous job's tenant context.
- Job execution must be idempotent wherever possible.
- Retries must follow controlled retry and backoff policies.
- Permanently failing jobs must move to a dead-letter state for investigation.
- Job payloads must contain references rather than unnecessary copies of sensitive business data.
- Workers must use least-privilege service identities.
- Errors must be sanitized before storage.
- Job execution, cancellation and retry operations must be auditable.
- Long-running jobs must support monitoring and controlled cancellation.
- Concurrent workers must not process the same job without an approved locking or claim mechanism.

**Data Integrity and Lifecycle**

- Job submission and queue publication must use a reliable transactional mechanism, preferably a transactional outbox where required.
- Execution status updates must be consistent with the queue-processing lifecycle.
- Failed jobs must retain sufficient evidence for troubleshooting.
- Retry attempts must be independently traceable.
- Completed job records must not be silently rewritten.
- Large job outputs must be stored in private object storage rather than directly in JSONB.
- Retention and archival must follow operational and compliance requirements.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must determine whether each retry updates this record or creates a separate execution-attempt record. A separate `background_job_attempt` table is recommended for complete attempt history.
- The system must define queue names and worker responsibilities.
- The system must establish retry counts, backoff intervals and dead-letter policies.
- The system must define job locking, leasing and concurrency controls.
- The system must determine whether job payloads require encryption.
- The system must define execution timeouts and cancellation behavior.
- The system must define job monitoring, alerting and operational dashboards.
- The system must establish how job records relate to `scheduled_job_execution`, `subscription_event`, `import_job` and other business-process tables.
- The system must define retention and archival policies for execution history.

### 3.88 `scheduled_job_execution`

**Purpose**

The `scheduled_job_execution` table stores execution records for jobs initiated by a schedule or recurring automation rule.

It provides traceability for scheduled operations, including execution time, outcome, failures and retries.

Examples include:

- Daily pending ITR reminders.
- Upcoming filing deadline notifications.
- Scheduled customer follow-ups.
- Subscription renewal processing.
- Periodic subscription usage calculations.
- Scheduled financial reconciliation.
- Daily or monthly report generation.
- Expired session and temporary-data cleanup.
- Document retention and archival processing.
- Periodic security monitoring tasks.

This table records scheduled execution history. The actual schedule definition and next-run calculation must be managed separately.

**Ownership and Tenant Scope**

- Ownership: System-Owned, with an optional firm relationship.
- Tenant scope: Firm-associated or platform-wide.
- Tenant identifier: `firm_id`, nullable for platform-wide schedules.
- Firm-associated records must be protected through tenant-aware RLS.
- Platform-wide records require separately restricted access policies.
- A scheduled execution must establish the correct tenant context before processing.
- Schedule definitions and execution records must remain distinguishable.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | Yes | Associated firm |
| `schedule_reference` | VARCHAR(150) | No | Schedule reference |
| `scheduled_job_name` | VARCHAR(255) | No | Human-readable job name |
| `job_type` | VARCHAR(100) | No | Scheduled job classification |
| `schedule_type` | VARCHAR(30) | No | Scheduling method |
| `scheduled_for` | TIMESTAMPTZ | No | Intended execution time |
| `started_at` | TIMESTAMPTZ | Yes | Actual execution start |
| `completed_at` | TIMESTAMPTZ | Yes | Execution completion |
| `failed_at` | TIMESTAMPTZ | Yes | Failure timestamp |
| `execution_status` | VARCHAR(30) | No | Execution outcome |
| `execution_attempt` | INTEGER | No | Attempt number |
| `maximum_attempts` | INTEGER | No | Maximum permitted attempts |
| `timezone_name` | VARCHAR(100) | Yes | Schedule's configured timezone |
| `schedule_configuration` | JSONB | No | Validated schedule metadata |
| `execution_result` | JSONB | Yes | Execution result metadata |
| `error_code` | VARCHAR(100) | Yes | Structured failure code |
| `error_summary` | TEXT | Yes | Sanitized failure description |
| `background_job_execution_id` | UUID | Yes | Related background job |
| `correlation_id` | UUID | Yes | Related workflow identifier |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`, when applicable.
- `background_job_execution_id` references `background_job_execution.id`.

The referenced background job must have the same tenant scope.

Platform-wide scheduled executions must not reference firm-specific background jobs unless an explicitly authorized platform workflow requires it.

**Schedule Types**

Initial supported values:

- `one_time`
- `recurring`
- `cron_based`
- `event_triggered`
- `system_maintenance`

**Job Types**

Initial supported values:

- `itr_reminder`
- `filing_deadline_notification`
- `customer_follow_up`
- `subscription_renewal`
- `subscription_usage_calculation`
- `financial_reconciliation`
- `report_generation`
- `session_cleanup`
- `temporary_file_cleanup`
- `document_archival`
- `security_monitoring`
- `other`

**Execution Statuses**

- `scheduled`
- `queued`
- `processing`
- `completed`
- `completed_with_warnings`
- `failed`
- `retry_scheduled`
- `skipped`
- `cancelled`
- `missed`

**Constraints and Data Integrity**

- Schedule reference must be valid and traceable.
- Job type, schedule type and execution status must use approved values.
- `schedule_configuration` and `execution_result`, when supplied, must be valid JSON objects.
- Execution attempt must be nonnegative.
- Maximum attempts must be positive.
- Completion and failure timestamps must match the execution lifecycle.
- The associated background job must belong to the same tenant scope.
- Duplicate execution of the same scheduled occurrence must be prevented through an idempotency mechanism.
- Schedule time and timezone must be interpreted consistently.
- Execution results must not contain secrets or unnecessary sensitive data.

**Indexes**

Proposed indexes:

- Index on `(firm_id, scheduled_for)`.
- Index on `(execution_status, scheduled_for)`.
- Index on `(firm_id, job_type, scheduled_for DESC)`.
- Index on `(schedule_reference, scheduled_for DESC)`.
- Index on `(background_job_execution_id)`.
- Index on `(correlation_id)`.
- Partial index for scheduled and retry-pending executions.

**Business and Security Rules**

- Only approved schedule definitions may initiate scheduled executions.
- Scheduled execution must use the configured schedule and timezone.
- Firm-specific schedules must execute within the correct tenant context.
- Platform-wide schedules must use authorized system identities.
- Duplicate execution of a scheduled occurrence must be prevented.
- Long-running or delayed executions must be monitored.
- Retry policies must follow controlled backoff and failure handling.
- Missed executions must be identifiable and handled according to the job's business rules.
- Scheduled notifications must respect communication preferences and applicable consent requirements.
- Filing deadline notifications must use the correct assessment year and applicable deadline configuration.
- Scheduled subscription operations must not bypass payment or subscription authorization rules.
- All execution attempts and administrative changes must be auditable.

**Data Integrity and Lifecycle**

- Every scheduled execution must retain its intended execution timestamp.
- Actual execution timestamps must be recorded separately.
- Schedule changes must not rewrite historical execution records.
- Execution status must remain consistent with the related background job.
- A retry must be traceable to the original scheduled occurrence.
- Missed or cancelled executions must retain their original scheduling context.
- All timestamps must use UTC, while the configured timezone must be retained where needed to interpret local schedules.
- Retention and archival must follow operational requirements.

**Design Notes and Open Decisions**

- A separate `scheduled_job_definition` table is required to store recurring schedule definitions, enablement status and next execution time.
- The system must define the supported scheduling syntax and recurrence rules.
- The system must establish timezone and daylight-saving handling.
- The system must define missed-execution recovery and catch-up behavior.
- The system must determine whether retries update the same execution record or create separate attempt records.
- The system must define how scheduled executions create `background_job_execution` records.
- The system must define locking and concurrency controls to prevent duplicate execution.
- The system must establish schedule ownership and authorization rules.
- The system must define monitoring, alerting and retention policies.

### 3.89 `webhook_event`

**Purpose**

The `webhook_event` table stores metadata and processing information for webhook events exchanged between the ITR Management System and external services.

It supports reliable integration processing, event verification, idempotency, retries and troubleshooting.

Examples include:

- Payment gateway payment confirmations.
- Payment failure notifications.
- Refund status updates.
- Email delivery and bounce notifications.
- SMS delivery reports.
- Authorized external integration callbacks.
- Subscription payment events.
- Other approved inbound or outbound webhook events.

**Ownership and Tenant Scope**

- Ownership: System-Owned, with an optional firm relationship.
- Tenant scope: Platform-wide or firm-associated.
- Tenant identifier: `firm_id`, nullable for platform-wide events.
- Firm-associated events must be protected by tenant-aware RLS.
- Platform-wide events require separate restricted access policies.
- Incoming webhook processing must not trust a tenant identifier supplied by an external request without independent verification.
- Webhook records must preserve sufficient evidence for security investigations and reliable retries.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | Yes | Associated firm, if applicable |
| `webhook_reference` | VARCHAR(150) | No | Internal webhook reference |
| `provider_name` | VARCHAR(100) | No | External service provider |
| `integration_type` | VARCHAR(50) | No | Integration classification |
| `webhook_direction` | VARCHAR(20) | No | Incoming or outgoing |
| `event_type` | VARCHAR(100) | No | Provider event classification |
| `external_event_id` | VARCHAR(255) | Yes | Provider event identifier |
| `idempotency_key` | VARCHAR(255) | Yes | Duplicate-processing protection |
| `webhook_status` | VARCHAR(30) | No | Processing status |
| `signature_status` | VARCHAR(30) | No | Signature verification outcome |
| `payload_hash` | VARCHAR(255) | Yes | Payload integrity reference |
| `payload_encrypted` | BYTEA | Yes | Protected payload, if retention is required |
| `encryption_key_version` | VARCHAR(100) | Yes | Encryption key version |
| `response_status_code` | INTEGER | Yes | HTTP response status |
| `processing_attempts` | INTEGER | No | Number of processing attempts |
| `received_at` | TIMESTAMPTZ | Yes | Incoming event receipt time |
| `sent_at` | TIMESTAMPTZ | Yes | Outgoing event transmission time |
| `processed_at` | TIMESTAMPTZ | Yes | Successful processing time |
| `next_retry_at` | TIMESTAMPTZ | Yes | Scheduled retry time |
| `last_error_code` | VARCHAR(100) | Yes | Structured error code |
| `last_error_summary` | TEXT | Yes | Sanitized error description |
| `correlation_id` | UUID | Yes | Related business operation |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`, when applicable.

Provider-specific business references should be maintained through the appropriate integration or business-event records rather than unrestricted polymorphic foreign keys.

For example, a payment gateway callback may be linked to a corresponding `customer_payment` or `subscription_payment` through a validated integration-processing relationship.

**Webhook Directions**

- `incoming`
- `outgoing`

**Integration Types**

- `payment_gateway`
- `email_provider`
- `sms_provider`
- `authorized_external_api`
- `other`

**Webhook Statuses**

- `received`
- `signature_verification_pending`
- `verified`
- `processing`
- `processed`
- `failed`
- `retry_scheduled`
- `ignored`
- `dead_letter`

**Signature Verification Statuses**

- `not_applicable`
- `pending`
- `verified`
- `invalid`
- `failed`

**Constraints and Data Integrity**

- Webhook reference, provider name, integration type, direction, event type and status must be populated.
- Direction, integration type and status must use approved values.
- Processing attempts must be nonnegative.
- Payload hash must use an approved integrity algorithm.
- Encrypted payload and encryption key version must be consistent when payload retention is enabled.
- Successfully processed incoming webhooks must have a verified signature whenever the provider supports signature verification.
- Unverified or invalid events must not initiate sensitive business operations.
- External event identifiers must be unique within the provider and integration scope where supplied.
- Idempotency keys must prevent duplicate business processing.
- Webhook processing timestamps must follow the event lifecycle.
- Error descriptions must not expose secrets or sensitive customer information.

**Indexes**

Proposed indexes:

- Unique index on `webhook_reference`.
- Index on `(provider_name, external_event_id)`.
- Index on `(provider_name, webhook_status, received_at DESC)`.
- Index on `(firm_id, webhook_status, created_at DESC)`.
- Index on `(integration_type, event_type, received_at DESC)`.
- Index on `(webhook_direction, webhook_status, created_at DESC)`.
- Index on `(next_retry_at, webhook_status)`.
- Index on `(correlation_id)`.
- Unique index on the approved provider-scoped idempotency key where applicable.

**Business and Security Rules**

- Incoming webhooks must be verified using the provider's documented authentication mechanism.
- Signature verification must use the correct provider-specific signing method.
- Timestamp validation and replay protection must be enforced where supported.
- Provider credentials and signing secrets must be stored in an approved secrets-management service.
- Secrets must never be stored in webhook payloads or error descriptions.
- Unverified webhooks must not update payments, refunds, subscriptions or other business records.
- Processing must be idempotent.
- Duplicate callbacks must not create duplicate payments, refunds or subscription changes.
- External payloads must be treated as untrusted input.
- Payload size and content type must be validated.
- Webhook processing must use controlled asynchronous jobs where appropriate.
- Failed processing must follow retry and dead-letter policies.
- Provider responses must not be treated as proof of a business outcome unless independently verified.
- Outgoing webhook delivery must use authenticated HTTPS endpoints and controlled retry policies.
- Webhook administrative actions and processing outcomes must be auditable.

**Data Integrity and Lifecycle**

- Incoming events must be persisted before asynchronous business processing.
- Acknowledgement to the provider must follow the provider's delivery protocol.
- Business state changes and event-processing outcomes must be coordinated transactionally.
- Duplicate event delivery must remain safe.
- Out-of-order events must be detected and handled according to the relevant business rules.
- Original event receipt information must remain traceable.
- Payload retention must be minimized and protected through encryption.
- High-volume webhook records may require time-based partitioning.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must determine whether incoming and outgoing webhooks share one table or use separate event and delivery tables.
- A separate `webhook_delivery_attempt` table may be required for complete retry history.
- The system must define provider-specific signature verification and replay-protection rules.
- The system must establish payload encryption, retention and deletion policies.
- The system must define provider event ordering and deduplication behavior.
- The system must decide how webhook events link to payment, refund and subscription business records.
- The system must define retry, backoff and dead-letter policies.
- The system must establish provider rate limits and endpoint security requirements.
- The system must define monitoring and alerting for repeated webhook failures.

### 3.90 `integration_configuration`

**Purpose**

The `integration_configuration` table stores configuration metadata for external services and integrations used by the ITR Management System.

Examples include:

- Payment gateway integrations.
- Email delivery providers.
- SMS providers.
- Authorized third-party APIs.
- Document verification services.
- Other approved external services.

**Ownership and Tenant Scope**

- Ownership: System-Owned, with platform-wide or firm-specific scope.
- Tenant scope: Platform-wide or firm-associated.
- Tenant identifier: `firm_id`, nullable for platform-wide integrations.
- Firm-specific configurations must be protected by tenant-aware RLS.
- Platform-wide configurations require separate restricted access policies.
- A firm must not be able to view or modify another firm's integration configuration.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | Yes | Associated firm, if applicable |
| `integration_code` | VARCHAR(100) | No | Unique integration identifier within its scope |
| `integration_name` | VARCHAR(150) | No | Display name |
| `integration_type` | VARCHAR(50) | No | Integration category |
| `provider_name` | VARCHAR(100) | No | External service provider |
| `environment` | VARCHAR(20) | No | Sandbox or production |
| `configuration_data` | JSONB | No | Non-secret integration settings |
| `credential_reference` | VARCHAR(255) | Yes | Reference to externally managed credentials |
| `integration_status` | VARCHAR(30) | No | Current integration status |
| `is_enabled` | BOOLEAN | No | Whether the integration is enabled |
| `last_verified_at` | TIMESTAMPTZ | Yes | Last successful connection verification |
| `verification_status` | VARCHAR(30) | No | Latest verification outcome |
| `last_error_code` | VARCHAR(100) | Yes | Last structured verification error |
| `last_error_summary` | TEXT | Yes | Sanitized error description |
| `created_at` | TIMESTAMPTZ | No | Creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `created_by_membership_id` | UUID | Yes | Firm membership that created the configuration |
| `updated_by_membership_id` | UUID | Yes | Firm membership that last updated it |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`, when applicable.
- `created_by_membership_id` and `updated_by_membership_id` reference `firm_membership.id` for firm-owned configurations.
- Firm membership references must be validated against the same firm.
- Platform-wide configurations must use an appropriate platform actor model rather than firm membership references.

**Integration Types**

- `payment_gateway`
- `email`
- `sms`
- `external_api`
- `document_verification`
- `other`

**Environments**

- `sandbox`
- `production`

**Integration Statuses**

- `draft`
- `pending_verification`
- `active`
- `inactive`
- `suspended`
- `failed`
- `revoked`

**Verification Statuses**

- `not_verified`
- `pending`
- `verified`
- `failed`
- `expired`

**Constraints and Data Integrity**

- Integration code, name, type, provider and environment must be populated.
- Configuration data must be a valid JSON object.
- Integration type, environment, status and verification status must use approved values.
- Integration codes must be unique within their platform or firm scope.
- A production integration must not be activated without the required verification.
- An enabled integration must have an appropriate active status.
- Firm-specific actor references must belong to the same firm.
- Credential references must point to approved secrets-management records.
- Configuration changes must be validated against provider-specific schemas.

**Indexes**

Proposed indexes:

- Unique index on platform-wide `integration_code` where `firm_id IS NULL`.
- Unique index on `(firm_id, integration_code)` where `firm_id IS NOT NULL`.
- Index on `(firm_id, integration_type, integration_status)`.
- Index on `(provider_name, environment)`.
- Index on `(integration_status, is_enabled)`.
- Index on `(verification_status, last_verified_at)`.

**Business and Security Rules**

- API keys, passwords, client secrets, signing keys and access tokens must never be stored directly in this table.
- Credentials must be stored in an approved secrets-management service.
- Only a protected credential reference may be stored in the database.
- Credential references must not be exposed through ordinary API responses or logs.
- Integration configuration access must require appropriate permissions.
- Firm administrators may manage only integrations explicitly delegated to their firm.
- Platform-wide integration configuration must remain under platform administration.
- Production integrations must use HTTPS and provider-approved authentication.
- Provider-specific credentials must be isolated by firm wherever the provider supports separate credentials.
- Integration changes must generate audit records.
- Credential rotation and revocation must be supported.
- Failed verification must not silently activate an integration.

**Data Integrity and Lifecycle**

- Configuration changes must be validated before activation.
- Verification results must record the time and outcome of the latest verification.
- Disabling an integration must stop new operations without deleting historical event records.
- Revoked credentials must no longer authorize new integration requests.
- Historical changes may be retained in audit records or a dedicated configuration history table.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The exact secrets-management service and credential-reference format remain to be selected.
- The system must define whether platform-wide integrations can be overridden by individual firms.
- The system must establish provider-specific configuration schemas and validation rules.
- The system must define the permitted firm administrator permissions for integration management.
- A separate `integration_configuration_history` table may be required for versioned configuration snapshots.
- The system must determine whether integration health checks run synchronously or through background jobs.
- The system must define credential rotation, expiry and revocation workflows.
- The system must determine whether one firm can maintain multiple configurations for the same provider and environment.

### 3.91 `integration_delivery_log`

**Purpose**

The `integration_delivery_log` table stores operational records of requests and responses exchanged between the ITR Management System and external integration providers.

It provides traceability for integration operations, delivery attempts, failures, retries and troubleshooting.

Examples include:

- Payment gateway API requests.
- Email and SMS provider requests.
- Refund processing requests.
- External API requests.
- Document verification requests.
- Provider callback acknowledgements.
- Other authorized integration operations.

**Ownership and Tenant Scope**

- Ownership: System-Owned, with an optional firm relationship.
- Tenant scope: Platform-wide or firm-associated.
- Tenant identifier: `firm_id`, nullable for platform-wide operations.
- Firm-associated records must be protected by tenant-aware RLS.
- Platform-wide records require separate restricted access policies.
- Each record must be associated with the relevant integration configuration where applicable.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | Yes | Associated firm, if applicable |
| `integration_configuration_id` | UUID | Yes | Related integration configuration |
| `delivery_reference` | VARCHAR(150) | No | Internal delivery reference |
| `operation_type` | VARCHAR(100) | No | Integration operation |
| `direction` | VARCHAR(20) | No | Outgoing or incoming |
| `provider_name` | VARCHAR(100) | No | External service provider |
| `endpoint_reference` | VARCHAR(255) | Yes | Sanitized endpoint identifier |
| `http_method` | VARCHAR(10) | Yes | HTTP method, where applicable |
| `request_reference` | VARCHAR(255) | Yes | Internal request or correlation reference |
| `external_request_id` | VARCHAR(255) | Yes | Provider request identifier |
| `delivery_status` | VARCHAR(30) | No | Delivery outcome |
| `attempt_number` | INTEGER | No | Attempt sequence number |
| `http_status_code` | INTEGER | Yes | HTTP response status |
| `request_metadata` | JSONB | Yes | Sanitized request metadata |
| `response_metadata` | JSONB | Yes | Sanitized response metadata |
| `request_payload_hash` | VARCHAR(255) | Yes | Request integrity reference |
| `response_payload_hash` | VARCHAR(255) | Yes | Response integrity reference |
| `duration_ms` | BIGINT | Yes | Request duration in milliseconds |
| `error_code` | VARCHAR(100) | Yes | Structured error code |
| `error_summary` | TEXT | Yes | Sanitized error description |
| `started_at` | TIMESTAMPTZ | No | Attempt start time |
| `completed_at` | TIMESTAMPTZ | Yes | Attempt completion time |
| `correlation_id` | UUID | Yes | Related business operation |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`, when applicable.
- `integration_configuration_id` references `integration_configuration.id`, when applicable.
- Firm-associated integration configuration references must belong to the same firm.
- Platform-wide integration records must follow separate platform access policies.

**Delivery Directions**

- `outgoing`
- `incoming`

**Delivery Statuses**

- `pending`
- `processing`
- `succeeded`
- `failed`
- `retry_scheduled`
- `timeout`
- `rejected`
- `cancelled`

**Constraints and Data Integrity**

- Delivery reference, operation type, direction, provider and status must be populated.
- Attempt number must be greater than zero.
- Duration, when supplied, must be nonnegative.
- Request and response metadata must be valid JSON objects when present.
- HTTP status codes must fall within the valid HTTP status-code range.
- Completion time must not precede start time.
- Firm and integration configuration references must satisfy tenant consistency.
- Delivery attempts must have unique references within their defined scope.
- Each retry must be distinguishable from previous attempts.
- Failed attempts must retain a structured failure reason where available.

**Indexes**

Proposed indexes:

- Unique index on `delivery_reference`.
- Index on `(firm_id, created_at DESC)`.
- Index on `(integration_configuration_id, created_at DESC)`.
- Index on `(provider_name, delivery_status, created_at DESC)`.
- Index on `(operation_type, delivery_status, created_at DESC)`.
- Index on `(correlation_id)`.
- Index on `(request_reference)`.
- Index on `(external_request_id)` where supplied.

**Business and Security Rules**

- Request and response bodies must not be stored in unrestricted logs.
- Passwords, access tokens, API keys, authorization headers, signing secrets and sensitive customer information must be redacted.
- If payload evidence is necessary, it must be stored through a separately approved encrypted retention mechanism.
- Integration logs must not become an alternative source of business truth.
- Provider responses must be validated before updating business records.
- Failed delivery must follow the applicable retry and backoff policy.
- Duplicate requests must be prevented through idempotency controls where supported.
- Integration delivery logs must be accessible only to authorized platform or firm personnel.
- Access to sensitive integration diagnostics must be audited.
- Logs must not expose another firm's integration activity.

**Data Integrity and Lifecycle**

- Each delivery attempt must have an independent record.
- A retry must create a new attempt record rather than overwrite the previous attempt.
- Delivery records should be immutable after completion, except for narrowly controlled operational annotations if required.
- Integration delivery outcomes must be traceable through request and correlation references.
- Long-running delivery operations must be monitored.
- Old operational records must follow defined retention and archival policies.
- High-volume records may require time-based partitioning.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must determine whether `integration_delivery_log` records one row per attempt or whether a separate delivery parent and attempt model is required.
- The system must define the relationship between webhook events, background jobs and integration delivery attempts.
- The system must establish request and response metadata retention limits.
- The system must define provider-specific retry, timeout and idempotency policies.
- The system must decide whether sanitized payload snapshots are necessary for selected integration types.
- The system must establish log partitioning, archival and deletion policies.
- The system must define monitoring and alerting for repeated failures and provider outages.

### 3.92 `privileged_access_grant`

**Purpose**

The `privileged_access_grant` table records explicit, approved grants of temporary privileged access by authorized platform personnel to firm-specific resources.

It supports controlled support operations, troubleshooting, security investigations and emergency access.

A support case or approval request does not itself authorize access. An active grant must exist before the privileged operation is permitted.

**Ownership and Tenant Scope**

- Ownership: Platform-Owned, firm-associated.
- Tenant scope: Firm-specific.
- Tenant identifier: `firm_id`, mandatory.
- Access administration: Authorized platform security and support roles.
- Firm-associated records must follow a dedicated platform-controlled access policy.
- Ordinary firm users must not be able to create, approve, extend or modify privileged access grants.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | No | Firm whose resources may be accessed |
| `support_case_id` | UUID | No | Related platform support case |
| `platform_user_id` | UUID | No | Platform user receiving access |
| `grant_reference` | UUID | No | Unique grant reference |
| `access_type` | VARCHAR(50) | No | Type of privileged access |
| `resource_scope` | JSONB | No | Explicitly permitted resource scope |
| `permitted_actions` | JSONB | No | Explicitly permitted actions |
| `purpose` | TEXT | No | Business justification |
| `approval_status` | VARCHAR(30) | No | Approval outcome |
| `approved_by_platform_user_id` | UUID | Yes | Approving platform user |
| `approval_reason` | TEXT | Yes | Approval decision details |
| `approved_at` | TIMESTAMPTZ | Yes | Approval timestamp |
| `grant_status` | VARCHAR(30) | No | Current grant status |
| `valid_from` | TIMESTAMPTZ | No | Grant start time |
| `expires_at` | TIMESTAMPTZ | No | Mandatory expiry time |
| `revoked_at` | TIMESTAMPTZ | Yes | Revocation timestamp |
| `revoked_by_platform_user_id` | UUID | Yes | Platform user revoking access |
| `revocation_reason` | TEXT | Yes | Revocation justification |
| `created_at` | TIMESTAMPTZ | No | Creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `correlation_id` | UUID | Yes | Related operation reference |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`.
- `support_case_id` references `platform_support_case.id`.
- `platform_user_id` references `platform_user.id`.
- `approved_by_platform_user_id` references `platform_user.id`.
- `revoked_by_platform_user_id` references `platform_user.id`.
- The support case must belong to the same firm.
- The grant recipient, approver and revoking actor must be valid platform users with the appropriate permissions.

**Access Types**

- `read_only`
- `document_access`
- `data_correction`
- `configuration_access`
- `diagnostic_access`
- `emergency_access`
- `other`

**Approval Statuses**

- `pending`
- `approved`
- `rejected`
- `not_required`

**Grant Statuses**

- `pending`
- `active`
- `expired`
- `revoked`
- `rejected`
- `cancelled`

**Constraints and Data Integrity**

- Firm, support case, recipient, access type, resource scope, purpose and expiry must be populated.
- Resource scope and permitted actions must be valid JSON objects or arrays according to their defined schemas.
- Expiry must be later than the grant start time.
- An approved grant must have an approver and approval timestamp unless an explicitly approved emergency policy applies.
- The grant recipient must not approve their own grant.
- Revoked grants must contain revocation details.
- A grant must not become active before approval.
- Expired or revoked grants must not authorize further operations.
- Access must be denied when the grant does not explicitly cover the requested resource and action.

**Indexes**

Proposed indexes:

- Unique index on `grant_reference`.
- Index on `(firm_id, grant_status, expires_at)`.
- Index on `(platform_user_id, grant_status, expires_at)`.
- Index on `(support_case_id)`.
- Index on `(approved_by_platform_user_id, approved_at DESC)`.
- Index on `(correlation_id)`.

**Business and Security Rules**

- Privileged access must follow least-privilege principles.
- Access must be restricted to the specified firm, resource scope and permitted actions.
- Every grant must have a mandatory expiry.
- Access must be time-limited and automatically denied after expiry.
- Approval must be independent of the requesting platform user.
- Emergency access must follow a separately documented break-glass policy.
- Emergency access must generate immediate security alerts and mandatory retrospective review.
- Every privileged operation must be recorded in `privileged_access_event`.
- The event's `access_grant_reference` must identify the grant authorizing that operation.
- A grant must not bypass tenant isolation or database security controls.
- Sensitive document and tax information access must require explicit scope and authorization.
- Revocation must take effect promptly across active sessions and processing operations.
- Grant creation, approval, rejection, extension and revocation must be auditable.
- Privileged access must never be granted merely because a support case exists.

**Data Integrity and Lifecycle**

- Grant creation and approval must be recorded as controlled state transitions.
- An active grant must be checked at the time of every privileged operation.
- Grant expiry must be enforced even if a background expiry job is delayed.
- Revocation must immediately prevent new privileged operations.
- Historical grants and their associated audit evidence must be retained according to the security retention policy.
- Grant records must not be physically deleted as part of routine revocation.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must define the exact resource-scope and permitted-action schemas.
- The system must establish approval requirements for each access type.
- The system must define maximum grant durations.
- The system must specify emergency access and retrospective approval procedures.
- The system must define how grants are checked by backend authorization and database access controls.
- The system must determine how active operations are terminated after revocation.
- The system must establish alerting and periodic review requirements.
- The system must define whether grant extensions require fresh approval or a new grant.

### 3.93 `background_job_attempt`

**Purpose**

The `background_job_attempt` table stores the individual execution attempts of background jobs.

It provides a detailed operational history of job processing, including retries, failures, timeouts and successful completion.

Examples include:

- Excel import processing.
- Document malware scanning.
- Customer notifications.
- ITR deadline reminders.
- Refund status synchronization.
- Report generation.
- Subscription usage calculations.
- Scheduled maintenance operations.

**Ownership and Tenant Scope**

- Ownership: System-Owned, with an optional firm relationship.
- Tenant scope: Platform-wide or firm-associated.
- Tenant identifier: `firm_id`, nullable for platform-wide jobs.
- Firm-associated records must be protected by tenant-aware RLS.
- Platform-wide records require separate restricted access policies.
- Every attempt must belong to a valid background job execution.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | Yes | Associated firm, if applicable |
| `background_job_execution_id` | UUID | No | Parent background job execution |
| `attempt_number` | INTEGER | No | Sequential attempt number |
| `attempt_reference` | VARCHAR(150) | No | Unique attempt reference |
| `worker_reference` | VARCHAR(150) | Yes | Worker processing the attempt |
| `attempt_status` | VARCHAR(30) | No | Attempt outcome |
| `started_at` | TIMESTAMPTZ | No | Attempt start time |
| `completed_at` | TIMESTAMPTZ | Yes | Attempt completion time |
| `duration_ms` | BIGINT | Yes | Execution duration in milliseconds |
| `result_summary` | JSONB | Yes | Sanitized execution result |
| `error_code` | VARCHAR(100) | Yes | Structured failure code |
| `error_summary` | TEXT | Yes | Sanitized failure description |
| `retryable` | BOOLEAN | No | Whether the failure permits retry |
| `retry_scheduled_at` | TIMESTAMPTZ | Yes | Scheduled retry time |
| `correlation_id` | UUID | Yes | Related business operation |
| `created_at` | TIMESTAMPTZ | No | Record creation timestamp |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`, when applicable.
- `background_job_execution_id` references `background_job_execution.id`.
- The parent job and attempt must have the same tenant scope.
- The parent job must be the authoritative source for overall job status.

**Attempt Statuses**

- `queued`
- `processing`
- `completed`
- `completed_with_warnings`
- `failed`
- `timeout`
- `cancelled`
- `retry_scheduled`

**Constraints and Data Integrity**

- Parent job, attempt number, attempt reference and status must be populated.
- Attempt number must be greater than zero.
- Attempt number must be unique within its parent job.
- Attempt reference must be unique.
- Duration must be nonnegative when supplied.
- Completion time must not precede start time.
- Result summary must be a valid JSON object when present.
- A successful attempt must have a completion timestamp.
- A failed attempt must contain a structured failure reason where available.
- Retry scheduling must be consistent with the retryable flag.
- Firm scope must match the parent background job.

**Indexes**

Proposed indexes:

- Unique index on `attempt_reference`.
- Unique index on `(background_job_execution_id, attempt_number)`.
- Index on `(firm_id, created_at DESC)`.
- Index on `(background_job_execution_id, attempt_status)`.
- Index on `(attempt_status, started_at DESC)`.
- Index on `(retryable, retry_scheduled_at)`.
- Index on `(correlation_id)`.

**Business and Security Rules**

- Every execution attempt must create a separate record.
- Previous attempts must never be overwritten by retries.
- Attempt history must remain traceable to its parent job.
- Tenant context must be established before processing firm-associated jobs.
- A job must not access another firm's records.
- Retry decisions must follow defined retry and backoff policies.
- Non-retryable failures must not be automatically retried.
- Repeated failures must be eligible for dead-letter handling and operational alerts.
- Job payloads and result summaries must not expose credentials, tokens or unnecessary sensitive customer information.
- Job execution must be idempotent where business operations could otherwise be duplicated.
- Cancellation and timeout handling must not leave business records in inconsistent states.
- Access to job diagnostics must be restricted and audited.

**Data Integrity and Lifecycle**

- Attempt creation and parent job status changes must be coordinated safely.
- Each attempt must retain its original execution outcome.
- Parent job status must reflect the authoritative execution lifecycle.
- A successful attempt must not be invalidated by a later retry.
- Attempts must remain available for operational troubleshooting and audit requirements.
- High-volume attempt records may require time-based partitioning.
- Retention and archival must follow the background job operational retention policy.
- All timestamps must use UTC.

**Design Notes and Open Decisions**

- The system must define which failures are retryable for each job type.
- The system must establish retry limits and exponential backoff policies.
- The system must define how a timed-out worker is prevented from continuing to modify business records.
- The system must establish dead-letter queue and manual replay procedures.
- The system must determine whether worker heartbeat and lease information belong in this table or a separate execution-control mechanism.
- The system must define the relationship between attempt records and operational monitoring.
- The system must establish retention and partitioning policies.

### 3.94 `scheduled_job_definition`

**Purpose**

The `scheduled_job_definition` table stores the configuration and lifecycle of scheduled jobs.

It defines recurring, one-time and calendar-based jobs that the system must execute automatically.

Examples include:

- ITR filing deadline reminders.
- Customer follow-up reminders.
- Subscription renewal processing.
- Subscription usage calculations.
- Refund reconciliation.
- Report generation.
- Expired session cleanup.
- Temporary file cleanup.
- Document archival.
- Security monitoring and maintenance.

**Ownership and Tenant Scope**

- Ownership: System-Owned, with an optional firm relationship.
- Tenant scope: Platform-wide or firm-associated.
- Tenant identifier: `firm_id`, nullable for platform-wide schedules.
- Firm-associated definitions must be protected by tenant-aware RLS.
- Platform-wide definitions require separate restricted access policies.
- Firm administrators may manage only schedules explicitly delegated to their firm.
- System-critical schedules must remain under platform control.

**Proposed Columns**

| Column | Data Type | Nullable | Description |
|---|---|---|---|
| `id` | UUID | No | Primary key |
| `firm_id` | UUID | Yes | Associated firm, if applicable |
| `schedule_reference` | VARCHAR(150) | No | Unique schedule reference |
| `schedule_name` | VARCHAR(200) | No | Human-readable schedule name |
| `job_type` | VARCHAR(100) | No | Type of scheduled job |
| `schedule_type` | VARCHAR(30) | No | Scheduling method |
| `schedule_expression` | VARCHAR(255) | Yes | Cron or equivalent expression |
| `timezone` | VARCHAR(100) | No | IANA timezone identifier |
| `schedule_configuration` | JSONB | No | Additional scheduling parameters |
| `execution_configuration` | JSONB | No | Job execution parameters |
| `schedule_status` | VARCHAR(30) | No | Current schedule status |
| `is_enabled` | BOOLEAN | No | Whether the schedule is enabled |
| `starts_at` | TIMESTAMPTZ | Yes | Schedule activation time |
| `ends_at` | TIMESTAMPTZ | Yes | Optional schedule end time |
| `next_run_at` | TIMESTAMPTZ | Yes | Next calculated execution time |
| `last_run_at` | TIMESTAMPTZ | Yes | Most recent execution time |
| `last_execution_id` | UUID | Yes | Most recent execution record |
| `maximum_concurrent_executions` | INTEGER | No | Allowed concurrent executions |
| `created_at` | TIMESTAMPTZ | No | Creation timestamp |
| `updated_at` | TIMESTAMPTZ | No | Last update timestamp |
| `created_by_membership_id` | UUID | Yes | Creating firm membership |
| `updated_by_membership_id` | UUID | Yes | Updating firm membership |

**Primary Key and Foreign Keys**

- Primary key: `id`.
- `firm_id` references `firm.id`, when applicable.
- `last_execution_id` references `scheduled_job_execution.id`.
- `created_by_membership_id` and `updated_by_membership_id` reference `firm_membership.id` for firm-managed schedules.
- Firm-specific references must satisfy same-firm integrity.
- Platform-managed schedules must use an appropriate platform actor model.

**Schedule Types**

- `one_time`
- `recurring`
- `cron_based`
- `calendar_based`
- `event_triggered`

**Schedule Statuses**

- `draft`
- `active`
- `paused`
- `completed`
- `disabled`
- `cancelled`
- `failed`

**Constraints and Data Integrity**

- Schedule reference, name, job type, schedule type, timezone and status must be populated.
- Schedule reference must be unique.
- Schedule and execution configuration fields must contain valid JSON objects.
- Maximum concurrent executions must be greater than zero.
- End time must be later than start time when both are provided.
- Recurring and cron-based schedules must contain valid scheduling expressions.
- One-time schedules must contain a valid execution time.
- Timezone must be a valid IANA timezone identifier.
- Next execution time must be calculated using the configured timezone and scheduling rules.
- Firm-specific actor references must belong to the same firm.
- Disabled or cancelled schedules must not generate new executions.

**Indexes**

Proposed indexes:

- Unique index on `schedule_reference`.
- Index on `(firm_id, schedule_status, is_enabled)`.
- Index on `(schedule_type, schedule_status)`.
- Index on `(next_run_at)` for enabled schedules.
- Index on `(job_type, schedule_status)`.
- Index on `(last_run_at DESC)`.
- Index on `(last_execution_id)` where supplied.

**Business and Security Rules**

- Schedule definitions must be separate from individual scheduled executions.
- Every scheduled occurrence must create or reference an individual `scheduled_job_execution` record.
- Schedule changes must not overwrite historical execution records.
- Only authorized users or system processes may create or modify schedules.
- Firm schedules must never execute with another firm's tenant context.
- Platform-wide schedules must run under restricted system permissions.
- Scheduling expressions and execution configuration must be validated before activation.
- Concurrent execution limits must be enforced.
- Duplicate occurrences must be prevented using deterministic occurrence identifiers or idempotency keys.
- Schedule changes, pauses, activation and cancellation must be auditable.
- Execution failures must follow the relevant retry and monitoring policies.
- Timezone and daylight-saving transitions must be handled explicitly.

**Data Integrity and Lifecycle**

- Schedule definition changes must be versioned or captured in audit history.
- Each execution must preserve the schedule configuration version used at execution time.
- The scheduler must safely identify due schedules and prevent multiple workers from claiming the same occurrence.
- Missed executions must follow an explicit catch-up or skip policy.
- Paused schedules must preserve their configuration.
- Completed one-time schedules must not execute again.
- All timestamps must use UTC, while the configured timezone controls calendar-based scheduling.

**Design Notes and Open Decisions**

- The system must define whether schedule configuration changes require a separate `scheduled_job_definition_version` table.
- The system must establish the exact cron implementation and expression validation rules.
- The system must define daylight-saving and timezone transition behavior.
- The system must establish missed-run, catch-up and skip policies.
- The system must define concurrency control and schedule-claiming mechanisms.
- The system must determine which schedules firms can manage independently.
- The system must define schedule versioning and audit requirements.
- The system must establish how scheduled occurrences are linked to `scheduled_job_execution` and `background_job_execution` records.