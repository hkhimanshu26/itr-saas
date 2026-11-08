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

### 2.20 System and Background Processing Table Group

| Proposed Table | Purpose | Ownership |
|---|---|---|
| `background_job_execution` | Background job execution metadata | System-Owned |
| `scheduled_job_execution` | Scheduled task execution history | System-Owned |
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