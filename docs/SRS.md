# ITR Management System
## Software Requirements Specification (SRS)

**Document ID:** ITR-SaaS-SRS-001  
**Version:** 0.1.0  
**Status:** Draft — Requirements Planning  
**Product Type:** Multi-Tenant Software as a Service (SaaS)  
**Initial Market:** India  
**Target Users:** CA Firms, Firm Administrators, Staff and Customers  
**Database:** PostgreSQL  
**Last Updated:** 5 November 2026

---

## Document Control

| Field | Details |
|---|---|
| Document Name | Software Requirements Specification |
| Product Name | ITR Management System |
| Document ID | ITR-SaaS-SRS-001 |
| Current Version | 0.1.0 |
| Document Status | Draft |
| Prepared For | Product Development and Stakeholders |
| Approval Status | Pending Review |

## 1. Introduction

### 1.1 Purpose

This Software Requirements Specification (SRS) defines the functional requirements, non-functional requirements, business rules, security controls, workflows, data requirements and acceptance criteria for the ITR Management System.

The document serves as the primary reference for product design, architecture, development, testing, deployment and future maintenance.

All implementation and release decisions must be traceable to approved requirements in this document or formally approved change requests.

### 1.2 Product Vision

The ITR Management System is intended to be a secure, scalable, commercial, multi-tenant SaaS platform that enables multiple Chartered Accountant (CA) firms to manage their income tax return preparation and related business operations.

The platform will provide separate, role-appropriate experiences for platform administrators, CA firm administrators, firm staff and customers.

It will centralize customer management, assessment-year records, document collection, ITR preparation workflows, review and approval, filing records, invoices, payments, communications, reporting and administrative operations.

### 1.3 Primary Objectives

The primary objectives are:

1. Provide a centralized platform for CA firms to manage customers and income tax return workflows.
2. Support multiple CA firms with strict tenant-level data isolation.
3. Provide secure customer and staff portals.
4. Simplify customer onboarding and document collection.
5. Track ITR preparation, review, filing and completion.
6. Manage customer invoices, payments and refunds.
7. Provide reporting, analytics and operational dashboards.
8. Support configurable firm-specific workflows and branding.
9. Introduce AI-assisted productivity features with human oversight.
10. Establish a secure, maintainable and commercially scalable SaaS foundation.

### 1.4 Intended Audience

This document is intended for:

- Product owner and business stakeholders.
- Software architects and developers.
- UI/UX designers.
- Quality assurance and testing engineers.
- Security and infrastructure engineers.
- CA firms participating in product evaluation and pilot testing.
- Legal and compliance advisors.
- Platform operations and support teams.

---

## 2. Document Management

### 2.1 Versioning Policy

The SRS will use semantic-style document versioning:

- **Major version:** Significant approved changes to product scope or requirements.
- **Minor version:** Addition or substantial modification of requirements.
- **Patch version:** Clarifications, corrections and editorial improvements.

### 2.2 Requirement Identification

Each requirement will receive a unique identifier.

Examples:

- `FR-CUS-001` — Functional requirement for customer management.
- `FR-ITR-001` — Functional requirement for ITR workflows.
- `SEC-TEN-001` — Security requirement for tenant isolation.
- `NFR-PERF-001` — Non-functional performance requirement.
- `INT-001` — External integration requirement.
- `NFR-TEST-001` — Non-functional testing requirement.
- `AC-001` — System acceptance criterion.

Requirement identifiers will remain stable even if their descriptions are updated.

### 2.3 Requirement Priorities

Requirements will use the following priority classifications:

| Priority | Meaning |
|---|---|
| Critical | Essential for security, legal compliance or core platform operation |
| High | Necessary for the primary product experience |
| Medium | Important but can be delivered in a later development phase |
| Low | Enhancement or future capability |

### 2.4 Requirement Traceability

Every approved requirement should be traceable to:

1. Its business objective.
2. Relevant user roles.
3. Related workflows and business rules.
4. Architecture and database components.
5. Implementation tasks.
6. Test cases.
7. Acceptance evidence.
8. Release version.

### 2.5 Change Management

Changes to approved requirements must be documented with:

- Change request identifier.
- Description and justification.
- Impact assessment.
- Affected requirements.
- Security and compliance implications.
- Approval status.
- Document version in which the change is incorporated.

No significant requirement change should be implemented without reviewing its impact on existing workflows, data structures, security and tenant isolation.

---

## 3. Product Overview

### 3.1 Product Description

The ITR Management System is a cloud-based, multi-tenant SaaS platform designed primarily for Indian Chartered Accountant (CA) firms.

It will provide a centralized environment for managing customers, assessment years, income tax return preparation, document collection, review workflows, filing records, billing, payments, communication and reporting.

Each CA firm will operate within its own logically isolated tenant environment.

The platform will support multiple firms without requiring separate application deployments for each firm.

### 3.2 Business Model

The product will operate as a commercial SaaS business.

The platform owner will provide subscription-based access to CA firms.

The commercial model will support:

- Starter, Professional and Enterprise subscription plans.
- Monthly and annual billing.
- Configurable customer and staff limits.
- Storage allowances and additional capacity.
- Optional modules and add-on services.
- Negotiated enterprise pricing.
- Trials, discounts and authorized pricing overrides.

Subscription billing between the platform owner and CA firms will remain separate from invoices and payments that CA firms manage for their own customers.

### 3.3 Target Market

The initial target market is India.

Primary customers include:

- Individual Chartered Accountants.
- Small CA practices.
- Medium-sized CA firms.
- Large CA firms with multiple staff members.
- CA firms managing large customer portfolios.

The architecture will remain ready for future localization and geographical expansion.

### 3.4 User Categories

The platform will support four primary user categories.

#### 3.4.1 Platform Super Admin

Responsible for overall SaaS platform administration, including:

- CA firm onboarding and approval.
- Subscription and plan management.
- Platform-wide configuration.
- Usage monitoring.
- System health and operational monitoring.
- Security and compliance governance.
- Support and incident management.
- Authorized emergency access.

#### 3.4.2 Firm Admin

Responsible for managing an individual CA firm's operations, including:

- Firm profile and branding.
- Staff invitations and permissions.
- Customer management.
- ITR workflow configuration.
- Document requirements.
- Invoices and payments.
- Reports and dashboards.
- Firm-level integrations and settings.

#### 3.4.3 Firm Staff

Responsible for authorized operational activities, including:

- Customer record management.
- Document collection and verification.
- ITR preparation.
- Data entry and validation.
- Review and approval according to assigned permissions.
- Customer communication.
- Follow-up activities.
- Authorized financial and reporting operations.

#### 3.4.4 Customer

Responsible for interacting with their associated CA firm through the customer portal.

Customers will be able to:

- View their ITR status.
- Upload requested documents.
- Review document requirements.
- Respond to staff queries.
- View invoices and payment records.
- Track refunds and acknowledgements.
- Access historical assessment-year records.
- Communicate with authorized firm staff.
- Manage their profile and verified contact information.

### 3.5 System Architecture Overview

The platform will initially follow a modular monolith architecture.

The proposed technology foundation includes:

- Frontend: Next.js, React and TypeScript.
- Backend: NestJS and TypeScript.
- Database: PostgreSQL.
- ORM and data access: Prisma with controlled SQL.
- Database security: PostgreSQL Row-Level Security.
- UI foundation: Tailwind CSS and shadcn/ui.
- Background jobs: Redis and BullMQ.
- Document storage: Private cloud object storage.
- API documentation: OpenAPI/Swagger.
- Testing: Jest and Playwright.
- Deployment: Docker-based cloud infrastructure.

The architecture will support future scaling and extraction of independent services where justified.

### 3.6 Supported Assessment Years

The system will initially support:

- AY 2025–26.
- AY 2026–27.
- AY 2027–28.

Assessment years must be configurable by authorized Platform Super Admin users.

Historical records must remain accessible according to applicable permissions and retention policies even when an assessment year is no longer active.

### 3.7 Initial Product Boundaries

The initial product will focus on managing CA firm operations and supporting ITR preparation.

The following capabilities are excluded from the initial release:

- Automated ITR filing or submission to the Income Tax Department.
- Direct government refund API integrations.
- Automatic AIS/26AS retrieval.
- A comprehensive automated tax calculation engine.
- Native Android and iOS applications.

Assisted tax calculations, manual filing record management, document processing and the AI-assisted features defined in Section 5.27 are included in the initial release, subject to their respective requirements, security controls, human review and approval requirements. AI capabilities shall remain assistive and shall not independently perform consequential tax or workflow actions.

### 3.8 Deployment Model

The platform will use a cloud-portable architecture supporting AWS or Azure.

One primary cloud provider will be selected for the first production deployment.

The deployment architecture will support:

- Separate development, testing, staging and production environments.
- Multiple application instances.
- Managed database services.
- Private object storage.
- Background job processing.
- Centralized monitoring and logging.
- Automated backups and disaster recovery.
- CI/CD pipelines and controlled releases.

### 3.9 Key Product Principles

The following principles will guide product design and implementation:

1. Security and tenant isolation by design.
2. Privacy and compliance by design.
3. Role-based access and least privilege.
4. Configurable workflows with mandatory security safeguards.
5. Reliable and auditable financial records.
6. Customer-friendly and accessible interfaces.
7. Maintainable modular architecture.
8. Automated testing and controlled releases.
9. Human oversight of AI-assisted tax operations.
10. Long-term scalability and extensibility.

---

## 4. User Roles and Portal Responsibilities

### 4.1 Overview

The ITR Management System will provide four primary user categories, each with a dedicated portal and controlled access to platform functionality.

The four categories are:

1. Platform Super Admin.
2. Firm Admin.
3. Firm Staff.
4. Customer.

Access will be governed by role-based access control (RBAC), configurable permissions, tenant isolation and least-privilege principles.

### 4.2 Platform Super Admin Portal

#### 4.2.1 Purpose

The Platform Super Admin Portal will provide centralized control over the SaaS platform and its tenant organizations.

#### 4.2.2 Responsibilities

The Platform Super Admin may perform authorized activities including:

- Review and approve CA firm registration requests.
- Manage firm activation, suspension and reactivation.
- Create and manage subscription plans.
- Configure platform-wide assessment years.
- Monitor platform usage and service health.
- Manage platform-level configuration.
- Manage feature availability and platform policies.
- Monitor security alerts and audit events.
- Manage support operations.
- Configure integrations and service providers.
- Review subscription usage and exceptions.
- Manage platform-level operational and compliance controls.

#### 4.2.3 Access Restrictions

- Platform administration must require mandatory MFA.
- Privileged actions must be recorded in tamper-resistant audit logs.
- Administrative access must follow least-privilege principles.
- Emergency access must be justified, time-limited and audited.
- Platform administration must not automatically grant unrestricted access to CA firms' tax documents.
- Sensitive tenant data access must follow an approved privileged-access process.

### 4.3 Firm Admin Portal

#### 4.3.1 Purpose

The Firm Admin Portal will allow each CA firm to manage its own organization, users, customers, financial operations and business workflows.

#### 4.3.2 Responsibilities

The Firm Admin may perform authorized activities including:

- Manage firm profile, branding and configuration.
- Invite and manage firm staff.
- Assign predefined roles and configurable permissions.
- Add and manage customers.
- Invite customers to activate their accounts.
- Configure assessment-year availability.
- Configure document checklists and workflows.
- Assign customers and work to staff.
- Monitor ITR preparation and filing progress.
- Manage customer invoices and payments.
- Manage firm CRM and lead pipelines.
- Access firm reports and analytics.
- Configure firm integrations.
- Manage customer communications.
- Review firm-level audit activities.

#### 4.3.3 Access Restrictions

- A Firm Admin must only access their own firm's tenant data.
- A Firm Admin cannot grant permissions beyond their own authority.
- A Firm Admin cannot access another firm's private records.
- Mandatory platform security policies cannot be overridden.
- Sensitive administrative actions must be audited.

### 4.4 Firm Staff Portal

#### 4.4.1 Purpose

The Firm Staff Portal will provide authorized employees and associates with tools to perform assigned CA firm operations.

#### 4.4.2 Responsibilities

Depending on assigned permissions, staff may:

- View and manage assigned customers.
- Update customer information.
- Request and upload documents.
- Review uploaded documents.
- Prepare ITR records.
- Enter and validate tax-related information.
- Update permitted workflow statuses.
- Submit returns for review.
- Review returns when assigned reviewer permissions.
- Communicate with customers.
- Manage follow-up tasks.
- Create or update invoices where authorized.
- Access permitted reports.

#### 4.4.3 Access Restrictions

- Staff access must be restricted to their assigned customers where configured.
- Staff must not access records outside their authorized tenant or permissions.
- Staff cannot grant permissions to themselves.
- Staff cannot mark a return as filed without the required authorization and filing evidence.
- Financial, document and approval permissions must be independently configurable.
- Staff access and actions must be audited.

### 4.5 Customer Portal

#### 4.5.1 Purpose

The Customer Portal will allow customers to securely interact with their associated CA firms and access their own tax-related services.

#### 4.5.2 Responsibilities

Customers may:

- View their profile and verified contact details.
- View their associated CA firm relationships.
- Switch between authorized firm contexts.
- View assessment-year records.
- Check ITR preparation and filing status.
- View required document checklists.
- Upload requested documents.
- Review document submission status.
- Respond to document replacement requests.
- Communicate with authorized firm staff.
- View invoices and payment history.
- Make online payments where enabled.
- View receipts and acknowledgements.
- Track manually recorded refund information.
- Access permitted historical tax records.
- Submit support tickets.
- Manage permitted notification preferences.

#### 4.5.3 Access Restrictions

- Customers may only access their own authorized records.
- Customers cannot access internal firm notes or another customer's information.
- Customers cannot modify staff-controlled ITR filing statuses.
- Customers cannot approve or file income tax returns through the platform.
- Contact information changes must undergo verification.
- Customers cannot access another firm's internal records merely because they share a customer identity.

### 4.6 Cross-Portal Access Principles

The following principles apply across all portals:

1. Every request must be authenticated and authorized.
2. Tenant ownership must be verified for every tenant-scoped operation.
3. Backend authorization must be enforced independently of frontend visibility.
4. Sensitive actions must require appropriate permissions and, where necessary, additional verification.
5. Role and permission changes must be audited.
6. Access must be revoked promptly when accounts are deactivated or permissions change.
7. Cross-tenant access attempts must be denied and logged.
8. Shared customer identity must not imply shared access to firm-specific records.
9. Platform-level administrative privileges must remain separate from ordinary tenant operations.
10. All portal interfaces must follow accessibility and security requirements.

### 4.7 User Authentication Summary

| User Category | Login Method | MFA Policy |
|---|---|---|
| Platform Super Admin | Email and password | Mandatory |
| Firm Admin | Email and password | Mandatory |
| Firm Staff | Email and password | Mandatory |
| Customer | Password or verified email/SMS OTP | Optional |

Authentication, password management, OTP handling, session security and account recovery will be specified in the dedicated security requirements section.

### 4.8 Future User Categories

The architecture may support additional controlled user categories in future releases, such as:

- External auditors.
- Compliance reviewers.
- Authorized consultants.
- Temporary support personnel.

Such categories will require explicit business approval, permission definitions, tenant boundaries and security reviews before implementation.

---

## 5. Functional Requirements

### 5.1 CA Firm Registration and Onboarding

#### 5.1.1 Overview

The platform must support controlled registration and onboarding of CA firms.

A CA firm must submit a registration request and receive approval from the Platform Super Admin before its tenant account becomes active.

The system must support secure creation of the initial Firm Admin account through an invitation-based activation process.

#### 5.1.2 Functional Requirements

**FR-FRM-001: CA Firm Registration Request**

- Priority: Critical
- Actor: Prospective CA Firm

Description:

The system shall allow an authorized representative of a CA firm to submit a registration request containing the required firm and contact information.

Business Rules:

1. Each registration request must receive a unique reference ID.
2. Required information must be validated before submission.
3. Duplicate or potentially conflicting firm registrations must be flagged for review.
4. Submission must not automatically activate a tenant.
5. Registration information must be protected against unauthorized access.

Acceptance Criteria:

- A valid registration request can be submitted successfully.
- Invalid or incomplete mandatory information is rejected with clear validation messages.
- A unique registration reference is generated.
- The request is available to authorized Platform Super Admin users.
- The submitting representative receives an acknowledgement.

**FR-FRM-002: Platform Admin Approval**

- Priority: Critical
- Actor: Platform Super Admin

Description:

The system shall allow authorized Platform Super Admin users to review, approve or reject CA firm registration requests.

Business Rules:

1. Only authorized Platform Super Admin users may approve or reject requests.
2. Approval and rejection actions must be recorded in the audit log.
3. Rejected requests must include a reason.
4. Approval must initiate controlled tenant provisioning.
5. Tenant creation must be idempotent to prevent duplicate tenant records.

Acceptance Criteria:

- An authorized administrator can review submitted firm information.
- An administrator can approve or reject a request.
- Rejection requires a recorded reason.
- Approval creates exactly one tenant account.
- The approval decision and administrator identity are recorded.
- Unauthorized users cannot approve registrations.

**FR-FRM-003: Initial Firm Admin Invitation**

- Priority: Critical
- Actor: Platform System

Description:

After successful firm approval, the system shall generate a secure invitation for the initial Firm Admin.

Business Rules:

1. Invitations must be sent to the verified or approved contact email.
2. Invitation links must be single-use and time-limited.
3. Invitation tokens must be securely generated and stored.
4. Expired, revoked or previously used invitations must not be accepted.
5. The recipient must create their own password.
6. Mandatory MFA enrollment must be completed before privileged access is granted.

Acceptance Criteria:

- Firm approval triggers the invitation process.
- The recipient can activate an account using a valid invitation.
- An expired or reused invitation is rejected.
- The recipient establishes their own password.
- Mandatory MFA enrollment is enforced.
- Successful activation is recorded in the audit log.

**FR-FRM-004: Firm Tenant Provisioning**

- Priority: Critical
- Actor: Platform System

Description:

The system shall provision a logically isolated tenant environment for every approved CA firm.

Business Rules:

1. Every firm must have a unique internal tenant identifier.
2. Tenant-owned records must be associated with the correct tenant.
3. Tenant creation must establish required default configurations.
4. Provisioning failures must be detectable and recoverable.
5. A failed provisioning operation must not leave an incorrectly activated tenant.
6. Tenant provisioning must not expose another firm's information.

Acceptance Criteria:

- Each approved firm receives a unique tenant identifier.
- Required default settings are created.
- Tenant provisioning can be safely retried.
- Duplicate tenant creation is prevented.
- Cross-tenant data access is denied.
- Provisioning activity is auditable.

**FR-FRM-005: Firm Activation and Suspension**

- Priority: Critical
- Actor: Platform Super Admin

Description:

The system shall allow authorized Platform Super Admin users to activate, suspend and reactivate CA firm accounts.

Business Rules:

1. Only authorized administrators may change firm activation status.
2. Suspension must not automatically delete tenant data.
3. Existing records must remain protected and recoverable according to retention policies.
4. Suspension must prevent unauthorized new operational activity.
5. Permitted read-only access, historical access and support access must follow explicitly defined suspension policies.
6. Every status change must be audited.

Acceptance Criteria:

- An authorized administrator can suspend a firm.
- Suspended firms cannot perform prohibited operational activities.
- Existing records remain preserved.
- An authorized administrator can reactivate an eligible firm.
- All status changes and reasons are recorded.

### 5.2 User Account Management and Authentication

#### 5.2.1 Overview

The system must provide secure account creation, authentication, verification, recovery and lifecycle management for Platform Admins, Firm Admins, Firm Staff and Customers.

Authentication and authorization must be handled centrally and enforced by the backend.

#### 5.2.2 Functional Requirements

**FR-AUTH-001: Role-Based Account Creation**

- Priority: Critical
- Actors: Platform Super Admin, Firm Admin

Description:

The system shall support controlled creation of user accounts according to authorized onboarding workflows.

Business Rules:

1. Initial Firm Admin accounts must be created through secure platform invitations.
2. Firm Staff accounts must be invited by authorized Firm Admin users.
3. Customer accounts must be initiated through a CA firm's customer invitation.
4. Public self-registration is excluded from the initial release.
5. Account creation must validate the creator's authority and tenant context.
6. A user must not gain access to a tenant merely by knowing its identifier.

Acceptance Criteria:

- Each permitted user category follows its approved onboarding process.
- Unauthorized account creation is rejected.
- Staff invitations are restricted to the issuing firm.
- Customer invitations are linked to the intended firm relationship.
- Account creation is recorded in the audit log.

**FR-AUTH-002: Staff Authentication**

- Priority: Critical
- Actors: Platform Super Admin, Firm Admin, Firm Staff

Description:

The system shall authenticate privileged and staff users using registered email addresses and passwords, followed by mandatory MFA.

Business Rules:

1. Email addresses must be verified.
2. Passwords must meet the approved security policy.
3. MFA must be mandatory for all privileged and staff accounts.
4. Password storage must use a secure password-hashing algorithm such as Argon2id.
5. Login attempts must be rate-limited.
6. Authentication failures must not reveal whether an account exists.
7. Successful and suspicious login activity must be logged.

Acceptance Criteria:

- Valid credentials allow the user to proceed to MFA verification.
- Invalid credentials are rejected.
- Users without completed MFA enrollment cannot access protected portals.
- Repeated failed attempts trigger configured protection.
- Login events are auditable.

**FR-AUTH-003: Customer Authentication**

- Priority: Critical
- Actor: Customer

Description:

The system shall allow customers to authenticate using either password-based login or OTP-based login.

Business Rules:

1. Customers may choose an enabled authentication method.
2. OTPs must be sent only to verified registered email addresses or mobile numbers.
3. OTPs must be short-lived and single-use.
4. OTP requests and verification attempts must be rate-limited.
5. OTPs must never be stored or logged in plaintext.
6. Customer authentication must not automatically grant access to every CA firm associated with a matching PAN.
7. Optional customer MFA must be supported according to platform policy.

Acceptance Criteria:

- A customer can authenticate using valid password credentials.
- A customer can authenticate using a valid email or SMS OTP.
- Expired or reused OTPs are rejected.
- Unverified contact methods cannot be used for OTP login.
- Excessive OTP requests trigger protective controls.
- Authentication does not grant unauthorized firm access.

**FR-AUTH-004: Password Policy and Recovery**

- Priority: Critical
- Actor: All password-based users

Description:

The system shall enforce a secure password policy and provide controlled password recovery.

Business Rules:

1. Minimum password length shall be 12 characters.
2. Common and compromised passwords must be blocked where supported.
3. Arbitrary periodic password changes must not be required.
4. Password reset links or codes must be time-limited and single-use.
5. Password changes must invalidate appropriate existing sessions.
6. Recovery processes must not bypass mandatory MFA requirements for privileged accounts.

Acceptance Criteria:

- Passwords shorter than the minimum length are rejected.
- Known compromised passwords are rejected where the screening service is available.
- Authorized users can complete secure password recovery.
- Expired and reused recovery tokens are rejected.
- Password changes generate appropriate security notifications and audit events.

**FR-AUTH-005: Multi-Factor Authentication**

- Priority: Critical
- Actors: Platform Super Admin, Firm Admin, Firm Staff

Description:

The system shall enforce mandatory MFA for all privileged and staff users.

Business Rules:

1. MFA enrollment is mandatory before privileged access.
2. Secure authenticator-based MFA must be supported.
3. Recovery codes or equivalent secure recovery methods must be available.
4. MFA recovery must follow identity verification and audit procedures.
5. No ordinary administrator may bypass mandatory MFA.
6. Emergency recovery must use a separately controlled and audited process.

Acceptance Criteria:

- Staff and privileged users cannot access protected functionality without mandatory MFA.
- Valid MFA challenges allow access.
- Invalid or expired challenges are rejected.
- Recovery procedures are protected and auditable.
- MFA settings and recovery actions are logged.

**FR-AUTH-006: Session and Device Management**

- Priority: Critical
- Actor: All authenticated users

Description:

The system shall provide secure session lifecycle and device management.

Business Rules:

1. Sessions must have configurable inactivity and absolute expiration limits.
2. Users must be able to view and revoke eligible active sessions.
3. Password or MFA changes must trigger appropriate session invalidation.
4. Administrators must be able to revoke sessions within their authorized scope.
5. New or suspicious login notifications must be supported.
6. Session identifiers must be securely generated and protected.
7. Session activity must be audited without logging authentication secrets.

Acceptance Criteria:

- Expired sessions cannot access protected resources.
- Users can revoke their active sessions.
- Authorized administrators can revoke sessions.
- Password and MFA changes invalidate applicable sessions.
- Session revocation takes effect on subsequent protected requests.
- Session management events are recorded.

**FR-AUTH-007: Suspicious Login and Account Protection**

- Priority: Critical
- Actor: Platform Security Services

Description:

The system shall detect and respond to suspicious authentication activity.

Business Rules:

1. Configurable rate limits and progressive delays must be applied.
2. Risk-based account protection must be supported.
3. Unusual login patterns and repeated failures must generate security signals.
4. Automated restrictions must allow controlled recovery from false positives.
5. A location change alone must not be treated as proof of malicious activity.
6. Security events must be available to authorized security administrators.

Acceptance Criteria:

- Repeated failed attempts trigger configured protections.
- Suspicious login activity generates an appropriate security event.
- Security administrators can review relevant events.
- Legitimate users have a controlled recovery path.
- Security actions are recorded in the audit log.

---

### 5.3 Staff Management and Permission Administration

#### 5.3.1 Overview

The system shall allow Firm Admins to invite, activate, manage, assign and deactivate staff accounts within their own CA firm.

The permission model shall support predefined roles combined with configurable permissions.

#### 5.3.2 Functional Requirements

**FR-STAFF-001: Staff Invitation**

- Priority: Critical
- Actor: Firm Admin

Description:

The system shall allow authorized Firm Admins to invite new staff members.

Business Rules:

1. Staff invitations must be associated with the issuing firm's tenant.
2. Invitations must be sent through verified email addresses.
3. Invitation tokens must be single-use and time-limited.
4. Staff must establish their own passwords.
5. Mandatory MFA enrollment must be completed before privileged staff access.
6. Unauthorized users must not invite staff.

Acceptance Criteria:

- Authorized Firm Admins can invite staff.
- Invitations are associated with the correct firm.
- Expired, revoked and reused invitations are rejected.
- Staff can activate accounts securely.
- Invitation activity is audited.

**FR-STAFF-002: Predefined Staff Roles**

- Priority: High
- Actor: Firm Admin

Description:

The system shall provide predefined staff roles that can be assigned to employees.

Initial role categories may include:

- Data Entry Executive.
- Document Verification Executive.
- ITR Preparation Executive.
- ITR Reviewer.
- Finance Executive.
- Customer Support Executive.
- General Staff.

Business Rules:

1. Predefined roles must have documented default permissions.
2. Firm Admins may assign eligible roles within their authority.
3. Role assignments must not grant access beyond the assigning administrator's authority.
4. Role definitions must be maintained centrally or within approved firm configuration boundaries.

Acceptance Criteria:

- Firm Admins can assign eligible predefined roles.
- Each role provides its configured permissions.
- Unauthorized role assignments are rejected.
- Role assignments are recorded in the audit log.

**FR-STAFF-003: Configurable Permissions**

- Priority: Critical
- Actor: Firm Admin

Description:

The system shall support configurable permissions for authorized staff members.

Permission categories shall include:

1. Customer record access.
2. Customer assignment management.
3. Document viewing and uploading.
4. Document acceptance and rejection.
5. ITR preparation and editing.
6. ITR review and approval.
7. Workflow status updates.
8. Invoice and payment management.
9. Financial record access.
10. Customer communication.
11. Reporting and data exports.
12. CRM and lead management.
13. Administrative configuration.

Business Rules:

1. Permissions must be enforced by the backend.
2. Frontend visibility alone must not determine authorization.
3. Permissions must be restricted to the user's tenant.
4. Customer-level assignment restrictions must be supported.
5. Firm Admins cannot grant permissions beyond their own authority.
6. Permission changes must take effect according to defined session and authorization policies.
7. Sensitive permission changes must be audited.

Acceptance Criteria:

- Authorized Firm Admins can configure eligible permissions.
- Unauthorized permission combinations are rejected.
- Staff cannot access features outside their permissions.
- API requests cannot bypass permission restrictions.
- Permission changes are recorded.

**FR-STAFF-004: Customer Assignment Restrictions**

- Priority: Critical
- Actor: Firm Admin

Description:

The system shall allow Firm Admins to restrict staff access to assigned customers.

Business Rules:

1. Customer assignments must belong to the same firm.
2. Staff must not access unassigned customers when assignment restrictions apply.
3. Authorized supervisors may receive broader access within their tenant.
4. Customer reassignment must update applicable access permissions.
5. Assignment changes must be audited.

Acceptance Criteria:

- Firm Admins can assign customers to staff.
- Assigned staff can access permitted customer records.
- Unassigned staff cannot access restricted records.
- Reassignment updates access appropriately.
- Cross-tenant customer assignments are rejected.

**FR-STAFF-005: Staff Account Suspension and Deactivation**

- Priority: Critical
- Actor: Firm Admin

Description:

The system shall allow authorized Firm Admins to suspend or deactivate staff accounts.

Business Rules:

1. Deactivated accounts must lose access to protected firm resources.
2. Active sessions must be revoked according to the account lifecycle policy.
3. Historical staff actions must remain in audit records.
4. Deactivation must not delete customer records or financial history.
5. Staff reassignment must be supported where necessary.

Acceptance Criteria:

- Authorized Firm Admins can deactivate staff.
- Deactivated staff cannot access protected resources.
- Existing sessions are invalidated.
- Historical activity remains available to authorized auditors.
- Account lifecycle actions are audited.

### 5.4 Customer Identity and Firm Relationships

#### 5.4.1 Overview

The system shall support a shared customer identity that can be associated with multiple independent CA firms.

A customer's identity must remain distinct from their firm-specific relationship, tax records, invoices, documents and communications.

#### 5.4.2 Functional Requirements

**FR-CUS-001: Customer Creation**

- Priority: Critical
- Actor: Firm Admin or Authorized Firm Staff

Description:

The system shall allow authorized firm users to create customer records within their own tenant.

Business Rules:

1. Customer creation must require authorized access.
2. Required identity and contact information must be validated.
3. PAN must be handled as sensitive personal information.
4. Duplicate customer records must be detected within the firm.
5. Customer creation must not grant access based on PAN alone.
6. Customer identity and firm-specific records must be logically separated.

Acceptance Criteria:

- Authorized users can create customer records.
- Invalid mandatory information is rejected.
- Potential duplicates are flagged.
- Customer records are associated with the correct firm.
- Unauthorized users cannot create customers.

**FR-CUS-002: Customer Identity Matching**

- Priority: Critical
- Actor: Platform Identity Services

Description:

The system shall support controlled matching of customer identities across CA firms.

Business Rules:

1. PAN may be used as an identity-matching attribute.
2. PAN must be encrypted or otherwise strongly protected at rest.
3. Access to PAN must be restricted and audited.
4. Email and mobile numbers must be verified before being treated as verified contact methods.
5. Conflicting or uncertain identity matches must require manual review.
6. PAN alone must never authorize account access or identity linking.
7. Matching results must not expose another firm's private customer records.

Acceptance Criteria:

- Potential identity matches can be detected.
- Conflicting identity information is flagged.
- Unauthorized identity linking is prevented.
- Manual review is available for uncertain matches.
- Sensitive identity access is audited.

**FR-CUS-003: Multiple Firm Relationships**

- Priority: Critical
- Actor: Platform Identity Services

Description:

The system shall allow one customer identity to have relationships with multiple CA firms.

Business Rules:

1. Every firm relationship must have its own unique identifier.
2. Each relationship must belong to exactly one tenant.
3. ITR records, documents, invoices and communications must remain firm-specific.
4. Customer access to each relationship must be separately authorized.
5. Deleting one firm relationship must not automatically delete the shared customer identity or another firm's relationship.
6. Historical firm relationships must follow applicable retention policies.

Acceptance Criteria:

- One customer identity can be associated with multiple firms.
- Each relationship remains isolated.
- Customer access is restricted to authorized relationships.
- Data from one firm cannot be accessed through another firm's relationship.
- Relationship changes are audited.

**FR-CUS-004: Customer Invitation and Activation**

- Priority: Critical
- Actor: Firm Admin or Authorized Firm Staff

Description:

The system shall allow firms to invite customers to activate their accounts or link an existing verified customer identity.

Business Rules:

1. Invitations must be associated with the intended firm relationship.
2. Invitation tokens must be single-use and time-limited.
3. Existing identity linking must require secure identity verification.
4. A customer must not gain access to another firm through an invitation.
5. Public self-registration is excluded from the initial release.
6. Invitation acceptance must be recorded.

Acceptance Criteria:

- Authorized firm users can invite customers.
- New customers can activate their accounts securely.
- Existing customers can link a new firm relationship after verification.
- Expired or reused invitations are rejected.
- Unauthorized relationship linking is prevented.

**FR-CUS-005: Customer Profile Management**

- Priority: High
- Actor: Customer and Authorized Firm Staff

Description:

The system shall support controlled management of customer profile information.

Business Rules:

1. Customers may update permitted profile fields.
2. Email and mobile number changes must undergo verification.
3. Sensitive identity changes may require staff review.
4. Firm-specific customer information must remain tenant-scoped.
5. Changes must be recorded in the audit history.

Acceptance Criteria:

- Customers can update permitted information.
- Contact changes require verification.
- Unauthorized sensitive changes are rejected.
- Authorized staff can review relevant changes.
- Profile changes are audited.

**FR-CUS-006: Customer Relationship Switching**

- Priority: High
- Actor: Customer

Description:

The system shall allow customers associated with multiple firms to switch between authorized firm relationships.

Business Rules:

1. Only verified and authorized relationships may be displayed.
2. Switching context must not transfer permissions between firms.
3. The active firm context must be explicitly established and verified by the backend.
4. Previous firm data must not remain accessible through cached or stale requests after switching.
5. Each relationship must display clear firm identity information.

Acceptance Criteria:

- Customers can view their authorized firm relationships.
- Customers can switch between authorized relationships.
- Each context displays only its permitted records.
- Unauthorized firm relationships cannot be selected.
- Context-switching activity is appropriately audited.

### 5.5 Multi-Tenant Isolation and Authorization

#### 5.5.1 Overview

The platform shall enforce strict tenant isolation across application services, database operations, document storage, background processing, caching, integrations and reporting.

Tenant isolation is a critical security requirement and a mandatory release acceptance criterion.

#### 5.5.2 Functional and Security Requirements

**SEC-TEN-001: Tenant Identification**

- Priority: Critical

Description:

Every CA firm shall be assigned a unique internal tenant identifier.

Business Rules:

1. Tenant identifiers must be generated and managed by trusted platform services.
2. Client-supplied tenant identifiers must never be trusted without authorization checks.
3. Tenant context must be established from authenticated and authorized relationships.
4. Tenant identity must be validated for every tenant-scoped operation.

Acceptance Criteria:

- Every tenant has a unique identifier.
- Unauthorized tenant identifiers are rejected.
- Tenant context cannot be arbitrarily changed by clients.
- Tenant access is verified on every protected operation.

**SEC-TEN-002: PostgreSQL Row-Level Security**

- Priority: Critical

Description:

The database shall enforce tenant-level data isolation using PostgreSQL Row-Level Security (RLS) on applicable tenant-owned tables.

Business Rules:

1. RLS policies must restrict access to the authorized tenant.
2. FORCE ROW LEVEL SECURITY must be applied where appropriate.
3. Runtime database roles must not own protected tables or possess BYPASSRLS privileges.
4. Tenant context must be established safely within database transactions.
5. Connection pooling must not permit tenant context leakage.
6. RLS policies must be tested independently of application authorization.

Acceptance Criteria:

- Queries cannot access another tenant's protected rows.
- RLS remains effective under the production runtime database role.
- Cross-tenant access attempts are denied.
- Transaction and connection-pooling tests demonstrate isolation.
- RLS policy changes are reviewed and tested.

**SEC-TEN-003: Tenant-Aware Database Relationships**

- Priority: Critical

Description:

The database shall enforce tenant ownership across related records using appropriate constraints and relationships.

Business Rules:

1. Tenant-owned records must contain appropriate tenant references.
2. Composite tenant-aware foreign keys must be used where required.
3. Cross-tenant record relationships must be prevented.
4. Database constraints must complement application-level validation.
5. Schema migrations must preserve tenant isolation.

Acceptance Criteria:

- Invalid cross-tenant relationships are rejected by the database.
- Referential integrity is maintained.
- Tenant ownership cannot be silently changed.
- Migration tests verify tenant-aware constraints.

**SEC-TEN-004: Backend Authorization**

- Priority: Critical

Description:

The backend shall independently authorize every protected operation.

Business Rules:

1. Authentication alone must not grant access to tenant records.
2. Authorization must consider role, permissions, tenant context, customer assignment and resource ownership.
3. Direct object references must not bypass authorization.
4. API endpoints must reject unauthorized access regardless of frontend visibility.
5. Sensitive operations may require additional verification or approval.

Acceptance Criteria:

- Unauthorized API requests are rejected.
- Changing record identifiers cannot expose other customers' records.
- Role and assignment restrictions are enforced.
- Authorization failures are recorded appropriately.

**SEC-TEN-005: Tenant Isolation Across Platform Services**

- Priority: Critical

Description:

Tenant isolation shall apply consistently across all platform components.

Business Rules:

1. Cache keys and cached data must be tenant-aware.
2. Background jobs must carry validated tenant context.
3. Document storage access must verify tenant and customer authorization.
4. Reports and exports must be tenant-scoped.
5. Search results must respect tenant and user permissions.
6. Notifications must not disclose another tenant's information.
7. Integrations must use appropriately scoped credentials and configurations.

Acceptance Criteria:

- Cross-tenant access is prevented in every applicable service.
- Background jobs cannot process records under an incorrect tenant.
- Cached data cannot leak between firms.
- Search, reports, exports and document downloads enforce tenant isolation.
- Security tests cover all relevant service boundaries.

**SEC-TEN-006: Cross-Tenant Security Testing**

- Priority: Critical

Description:

The system shall maintain automated and periodic security tests designed to identify cross-tenant access vulnerabilities.

Business Rules:

1. Tests must cover database, API, documents, search, caching and background jobs.
2. Tests must include attempts to manipulate identifiers and tenant context.
3. Critical tenant-isolation failures must block production release.
4. Security regressions must be tracked until remediation.
5. Security testing must use controlled synthetic test data.

Acceptance Criteria:

- Automated cross-tenant tests run during relevant release pipelines.
- Unauthorized access attempts are consistently denied.
- Critical isolation failures prevent release approval.
- Test evidence is retained and traceable to requirements.

---

### 5.6 Assessment Year and ITR Workflow Management

#### 5.6.1 Overview

The system shall support configurable assessment years and firm-specific ITR workflows.

Each customer may have multiple assessment-year records. Each assessment-year record must maintain its own tax information, documents, preparation progress, review history and filing information.

#### 5.6.2 Functional Requirements

**FR-AY-001: Assessment Year Configuration**

- Priority: Critical
- Actor: Platform Super Admin

Description:

The system shall allow authorized Platform Super Admin users to create, activate, deactivate and manage assessment years.

Initial assessment years:

- AY 2025–26.
- AY 2026–27.
- AY 2027–28.

Business Rules:

1. Assessment years must be configurable without application code changes.
2. Each assessment year must have a unique internal identifier.
3. Firms must be able to enable supported assessment years.
4. Deactivating an assessment year must not delete historical records.
5. Historical information must remain accessible according to permissions and retention policies.
6. Assessment-year configuration changes must be audited.

Acceptance Criteria:

- Authorized administrators can create assessment years.
- Firms can enable available assessment years.
- Inactive years cannot receive prohibited new work.
- Historical records remain accessible to authorized users.
- Assessment-year changes are audited.

**FR-AY-002: Firm Assessment Year Activation**

- Priority: High
- Actor: Firm Admin

Description:

The system shall allow Firm Admins to enable or disable assessment years for their firm from the years made available by the platform.

Business Rules:

1. Firms cannot create unauthorized platform-level assessment years.
2. Disabling an assessment year must not delete existing records.
3. Active work must be handled according to configured deactivation rules.
4. Changes must be recorded in the audit log.

Acceptance Criteria:

- Firm Admins can enable available assessment years.
- Unauthorized years cannot be enabled.
- Existing records are preserved when a year is disabled.
- Configuration changes are audited.

**FR-AY-003: Customer Assessment-Year Records**

- Priority: Critical
- Actor: Authorized Firm Admin or Firm Staff

Description:

The system shall allow authorized users to create and manage customer records for individual assessment years.

Business Rules:

1. Every record must belong to a valid customer-firm relationship.
2. Every record must reference a valid assessment year.
3. Duplicate records for the same customer, firm and assessment year must be controlled.
4. Historical records must remain separate from records belonging to other assessment years.
5. Customer and firm ownership must be validated for every operation.

Acceptance Criteria:

- Authorized users can create assessment-year records.
- Duplicate records are prevented or flagged according to business rules.
- Records remain associated with the correct customer, firm and year.
- Cross-tenant access is rejected.

**FR-AY-004: ITR Type and Filing Information**

- Priority: High
- Actor: Authorized Firm Staff

Description:

The system shall support recording the applicable ITR type and related filing information for each assessment-year record.

Business Rules:

1. ITR types must be configurable and maintained according to applicable tax rules.
2. Applicable ITR types must be selectable for each record.
3. Eligibility and applicability must be validated using approved business rules.
4. Changes to ITR type must be recorded.
5. The system must not assume that selecting an ITR type establishes filing eligibility.

Acceptance Criteria:

- Authorized users can select an applicable ITR type.
- Invalid or unsupported selections are rejected.
- Changes are recorded in the audit history.
- ITR applicability can be reviewed by authorized staff.

### 5.7 ITR Preparation, Review and Filing Workflow

#### 5.7.1 Overview

The system shall provide a controlled workflow for managing income tax return preparation from initial document collection through review, manual filing and completion.

#### 5.7.2 Standard Workflow Statuses

The initial standard statuses shall include:

1. Documents Pending.
2. Documents Received.
3. In Preparation.
4. Under Review.
5. Changes Required.
6. Ready for Filing.
7. Filed.
8. Completed.

Firms may define limited additional statuses subject to platform rules.

#### 5.7.3 Functional Requirements

**FR-ITR-001: ITR Workflow Creation**

- Priority: Critical
- Actor: Authorized Firm Admin or Firm Staff

Description:

The system shall allow authorized users to initiate an ITR workflow for a customer and assessment year.

Business Rules:

1. Every workflow must be linked to the correct customer-firm relationship.
2. Every workflow must reference an assessment year.
3. Initial status must follow the approved workflow configuration.
4. Duplicate active workflows must be controlled.
5. Workflow creation must be audited.

Acceptance Criteria:

- Authorized users can initiate an ITR workflow.
- Invalid customer or assessment-year associations are rejected.
- Initial workflow status is assigned correctly.
- Workflow creation is recorded.

**FR-ITR-002: ITR Preparation**

- Priority: Critical
- Actor: Authorized Firm Staff

Description:

The system shall allow authorized staff to prepare and update ITR-related information for a customer.

Business Rules:

1. Staff must possess the required ITR preparation permissions.
2. Customer and assessment-year associations must be validated.
3. Changes must preserve appropriate history.
4. Financial and tax values must use precise decimal data types.
5. Validation failures must be clearly communicated.
6. Sensitive tax information must be protected.

Acceptance Criteria:

- Authorized staff can update permitted ITR information.
- Unauthorized changes are rejected.
- Invalid data is flagged.
- Changes are recorded in the audit history.
- Data remains associated with the correct customer and assessment year.

**FR-ITR-003: Document Readiness Verification**

- Priority: Critical
- Actor: Authorized Firm Staff

Description:

The system shall support verification of required customer documents before an ITR workflow proceeds.

Business Rules:

1. Document requirements must follow the applicable checklist.
2. Mandatory documents must be distinguished from optional documents.
3. Rejected or replacement-requested documents must remain identifiable.
4. Document readiness must be based on authorized review decisions.
5. AI-generated document assessments must not independently establish final acceptance.

Acceptance Criteria:

- Staff can review document readiness.
- Missing mandatory documents are identified.
- Rejected documents are flagged.
- Readiness status reflects the current document checklist.
- Review decisions are audited.

**FR-ITR-004: ITR Review and Approval**

- Priority: Critical
- Actor: Authorized Reviewer or Firm Admin

Description:

The system shall support a controlled review and approval process before an ITR is marked Ready for Filing.

Business Rules:

1. Review permissions must be explicitly assigned.
2. Reviewers must be able to record findings and request corrections.
3. Changes Required must return the workflow to an appropriate preparation stage.
4. Approval must not automatically mark the return as Filed.
5. Review decisions must be recorded with user identity and timestamp.
6. Separation of preparation and review responsibilities must be configurable.

Acceptance Criteria:

- Authorized reviewers can review ITR records.
- Review findings can be recorded.
- Corrections can be requested.
- Unauthorized users cannot approve returns.
- Approved records can progress to Ready for Filing.
- Review history is retained.

**FR-ITR-005: Workflow Status Management**

- Priority: Critical
- Actor: Authorized Firm Staff and Firm Admin

Description:

The system shall enforce valid workflow status transitions according to role permissions and firm configuration.

Business Rules:

1. Standard system statuses must remain identifiable.
2. Custom statuses must not bypass mandatory workflow controls.
3. Customers must not directly modify staff-controlled filing statuses.
4. Invalid status transitions must be rejected.
5. Authorized exceptions must require appropriate justification.
6. All status changes must be audited.

Acceptance Criteria:

- Valid status transitions are permitted.
- Invalid transitions are rejected.
- Unauthorized status changes are blocked.
- Custom statuses follow configured rules.
- Status history is available to authorized users.

**FR-ITR-006: Manual Filing Record**

- Priority: Critical
- Actor: Authorized Firm Staff

Description:

The system shall allow authorized staff to record filing information after the income tax return has been submitted through the official Income Tax Department portal outside the platform.

Business Rules:

1. The platform will not independently submit income tax returns in the initial release.
2. Filing information must be entered by authorized staff.
3. Filing date and acknowledgement information must be validated.
4. Filing evidence, such as an acknowledgement document, may be uploaded.
5. The Filed status must not be assigned merely because an ITR is Ready for Filing.
6. Filing information must be auditable.

Acceptance Criteria:

- Authorized staff can record filing information.
- Filing date and acknowledgement details are captured.
- Supporting acknowledgement documents can be attached.
- Unauthorized filing-status changes are rejected.
- Filing records are linked to the correct customer and assessment year.

**FR-ITR-007: ITR Completion**

- Priority: High
- Actor: Authorized Firm Staff or Firm Admin

Description:

The system shall support controlled completion of an ITR workflow after the required post-filing activities have been satisfied.

Business Rules:

1. A return must be Filed before it can become Completed.
2. Applicable post-filing tasks must be verified.
3. Completion must require appropriate permissions.
4. Completed records must preserve their filing history.
5. Reopening a completed record must require authorization and justification.

Acceptance Criteria:

- Only eligible filed records can be completed.
- Unauthorized completion is rejected.
- Completion decisions are audited.
- Authorized reopening is supported.
- Historical workflow information remains available.

**FR-ITR-008: Workflow Assignment and Follow-up**

- Priority: High
- Actor: Firm Admin or Authorized Firm Staff

Description:

The system shall support assigning ITR workflows to staff and managing related follow-up activities.

Business Rules:

1. Assignments must remain within the firm's tenant.
2. Assigned staff must possess appropriate permissions.
3. Reassignment must update access according to policy.
4. Due dates and follow-up tasks must be supported.
5. Assignment changes must be audited.

Acceptance Criteria:

- Authorized users can assign workflows.
- Unauthorized assignments are rejected.
- Staff can view their permitted work.
- Follow-up tasks and due dates can be managed.
- Assignment history is retained.

**FR-ITR-009: Workflow History and Audit**

- Priority: Critical
- Actor: Platform System

Description:

The system shall maintain a historical record of important ITR workflow activities.

Business Rules:

1. Status changes must record previous and new values.
2. User identity and timestamps must be recorded.
3. Relevant review decisions and filing events must be retained.
4. Audit records must not be editable through ordinary user interfaces.
5. Access to audit information must be permission-controlled.

Acceptance Criteria:

- Authorized users can review permitted workflow history.
- Important changes include actor and timestamp.
- Unauthorized modification of audit records is prevented.
- Audit history remains tenant-scoped.

---

### 5.8 Document Management and Customer Document Lifecycle

#### 5.8.1 Overview

The system shall provide secure document management for customers and authorized CA firm staff.

Documents may include identity records, PAN-related documents, Form 16, bank statements, investment proofs, income records, tax computation documents, ITR acknowledgements and other supporting records.

The system shall maintain document ownership, tenant association, assessment-year association, version history, review status and access history.

#### 5.8.2 Functional Requirements

**FR-DOC-001: Document Upload**

- Priority: Critical
- Actors: Customer, Authorized Firm Staff

Description:

The system shall allow customers and authorized firm staff to upload documents against permitted customer records and assessment years.

Business Rules:

1. Upload permission must be verified before accepting a document.
2. Every document must be associated with the correct tenant and customer relationship.
3. Assessment-year association must be supported where applicable.
4. File extensions, MIME types, file signatures and file sizes must be validated.
5. Configurable upload size and file-type restrictions must be enforced.
6. Files must be stored in private object storage.
7. Uploaded files must not become publicly accessible.
8. Malware scanning and quarantine must be performed before documents become available for normal use.
9. Upload failures must not create incomplete or misleading document records.

Acceptance Criteria:

- Authorized users can upload permitted documents.
- Unsupported file types and oversized files are rejected.
- Files are associated with the correct tenant and customer.
- Unscanned or suspicious files remain quarantined.
- Unauthorized users cannot upload against unrelated customer records.
- Upload activity is audited.

**FR-DOC-002: Document Categorization**

- Priority: High
- Actor: Authorized Firm Staff

Description:

The system shall support categorization and classification of customer documents.

Initial categories may include:

- Identity and PAN documents.
- Income documents.
- Form 16 and salary records.
- Bank statements.
- Investment and deduction proofs.
- Capital gains records.
- House property documents.
- Business and professional income records.
- Tax payment records.
- ITR acknowledgements.
- Other supporting documents.

Business Rules:

1. Document categories must be configurable.
2. Documents may be associated with multiple relevant business classifications where required.
3. Category changes must be audited.
4. Document classification must not automatically imply acceptance or verification.

Acceptance Criteria:

- Authorized staff can categorize documents.
- Documents can be filtered by category.
- Invalid category assignments are rejected.
- Classification changes are recorded.

**FR-DOC-003: Document Review and Verification**

- Priority: Critical
- Actor: Authorized Firm Staff

Description:

The system shall allow authorized staff to review uploaded documents and record acceptance, rejection or replacement requests.

Business Rules:

1. Only authorized reviewers may make formal review decisions.
2. Rejected documents must include an appropriate reason.
3. Replacement requests must be communicated to the customer.
4. Document review status must be distinct from ITR workflow status.
5. Review decisions must be retained in document history.
6. Automated or AI-assisted checks must not independently establish final acceptance.

Acceptance Criteria:

- Authorized staff can accept or reject documents.
- Rejected documents require a reason.
- Replacement requests can be created.
- Customers can see permitted review outcomes.
- Review decisions and reviewer identities are audited.

**FR-DOC-004: Document Version Management**

- Priority: Critical
- Actor: Platform System

Description:

The system shall maintain version history when a document is replaced or updated.

Business Rules:

1. Replacing a document must create a new version rather than silently overwrite the existing version.
2. Previous versions must remain subject to access and retention policies.
3. Each version must record its uploader and timestamp.
4. The current version must be clearly identifiable.
5. Version history must be tenant-scoped.
6. Unauthorized users must not restore or modify previous versions.

Acceptance Criteria:

- Replacing a document creates a new version.
- Previous versions remain identifiable.
- Authorized users can view permitted version history.
- Version changes are audited.
- Cross-tenant version access is denied.

**FR-DOC-005: Secure Document Download and Viewing**

- Priority: Critical
- Actors: Customer, Authorized Firm Staff

Description:

The system shall allow authorized users to view or download documents through controlled access mechanisms.

Business Rules:

1. Authorization must be checked before every protected document access.
2. Download links must be short-lived and securely generated.
3. Public permanent document URLs must not be used.
4. Tenant, customer relationship and document permissions must be verified.
5. Document access must be logged.
6. Sensitive documents must not be exposed through browser caches or unauthorized previews.

Acceptance Criteria:

- Authorized users can access permitted documents.
- Unauthorized document requests are rejected.
- Expired download links cannot be reused.
- Documents cannot be accessed by changing record identifiers.
- Document access events are recorded.

**FR-DOC-006: Document Deletion and Retention**

- Priority: Critical
- Actor: Authorized Firm Admin or Platform System

Description:

The system shall support controlled document deletion, retention and legal-hold procedures.

Business Rules:

1. Deletion must follow approved retention policies.
2. Legal holds must prevent prohibited deletion.
3. Document deletion must require appropriate authorization.
4. Audit history must be retained even when document content is deleted, subject to applicable law.
5. Secure deletion must account for versions, replicas and applicable backup-expiry policies.
6. Deletion of one firm's relationship must not affect another firm's independent documents.

Acceptance Criteria:

- Authorized deletion requests can be processed.
- Retention rules are enforced.
- Legal holds prevent prohibited deletion.
- Deletion actions are audited.
- Unauthorized deletion is rejected.

**FR-DOC-007: Document Integrity and Security**

- Priority: Critical
- Actor: Platform System

Description:

The system shall protect document integrity, confidentiality and availability.

Business Rules:

1. Documents must be encrypted at rest and in transit.
2. Cryptographic checksums or equivalent integrity verification must be supported.
3. Storage permissions must follow least privilege.
4. Document storage must support appropriate backup and recovery.
5. Malware scanning failures must not silently release files.
6. Document metadata must not expose unnecessary sensitive information.

Acceptance Criteria:

- Stored documents are encrypted.
- Integrity checks can detect unauthorized modification or corruption.
- Storage access follows authorized policies.
- Failed security scans result in quarantine.
- Recovery procedures preserve document integrity.

### 5.9 Document Requirement Checklists

#### 5.9.1 Overview

The system shall provide configurable document checklists to guide customers and staff through the collection of required information for each customer and assessment year.

#### 5.9.2 Functional Requirements

**FR-CHECK-001: Standard Document Checklist**

- Priority: High
- Actor: Platform System

Description:

The system shall provide standard document checklist templates based on applicable customer and ITR requirements.

Business Rules:

1. Checklist templates must support assessment-year applicability.
2. Requirements must distinguish mandatory and optional documents.
3. Different customer and income categories may have different requirements.
4. Checklist changes must not silently remove existing submission history.
5. Standard templates must be configurable by authorized platform administrators.

Acceptance Criteria:

- Standard checklists can be generated.
- Applicable requirements are displayed.
- Mandatory and optional items are distinguished.
- Historical checklist information remains available.

**FR-CHECK-002: Firm-Specific Checklist Configuration**

- Priority: High
- Actor: Firm Admin

Description:

The system shall allow Firm Admins to customize document checklist requirements for their firm.

Business Rules:

1. Firms may add, remove or modify permitted additional requirements.
2. Platform-mandated security or compliance requirements cannot be overridden.
3. Checklist changes must be versioned.
4. Existing customer submissions must retain their historical context.
5. Checklist configuration must remain tenant-specific.

Acceptance Criteria:

- Firm Admins can customize permitted requirements.
- Changes apply according to configured effective dates.
- Existing submission history is preserved.
- Unauthorized cross-tenant checklist changes are prevented.

**FR-CHECK-003: Checklist Progress Tracking**

- Priority: High
- Actor: Customer and Authorized Firm Staff

Description:

The system shall track the completion and review status of each customer's document checklist.

Business Rules:

1. Checklist items must support pending, received, accepted, rejected and replacement-requested states as applicable.
2. Progress must reflect actual document review decisions.
3. Customers may view permitted checklist information.
4. Staff must be able to identify outstanding mandatory items.
5. Checklist changes and status updates must be audited.

Acceptance Criteria:

- Customers can view their permitted checklist.
- Staff can identify missing mandatory documents.
- Progress updates reflect document submissions and reviews.
- Unauthorized users cannot change checklist states.

### 5.10 Customer Communication and Messaging

#### 5.10.1 Overview

The system shall provide secure communication between customers and their associated CA firm staff.

Communication shall include built-in messaging and configurable notifications through email, SMS and WhatsApp providers.

#### 5.10.2 Functional Requirements

**FR-MSG-001: Built-in Customer Messaging**

- Priority: High
- Actors: Customer and Authorized Firm Staff

Description:

The system shall provide a messaging facility connecting customers with authorized staff.

Business Rules:

1. Messages must be associated with the correct customer-firm relationship.
2. Assessment-year association must be supported where applicable.
3. Access must be limited to authorized participants.
4. Internal firm notes must remain separate from customer-visible messages.
5. Message history must be retained according to applicable policies.
6. Message attachments must follow document security requirements.

Acceptance Criteria:

- Authorized users can exchange messages.
- Messages appear only in the correct firm context.
- Unauthorized users cannot read another customer's messages.
- Internal notes are not exposed to customers.
- Message activity is audited.

**FR-MSG-002: Message Attachments**

- Priority: Critical
- Actors: Customer and Authorized Firm Staff

Description:

The system shall support secure attachments in customer messages.

Business Rules:

1. Attachment upload must follow document validation and malware-scanning policies.
2. Attachment access must be restricted to authorized conversation participants.
3. Attachments must be stored privately.
4. Attachment deletion and retention must follow approved policies.

Acceptance Criteria:

- Authorized participants can attach permitted files.
- Unsupported or malicious files are rejected or quarantined.
- Unauthorized users cannot access attachments.
- Attachment actions are audited.

**FR-MSG-003: Email, SMS and WhatsApp Notifications**

- Priority: High
- Actor: Platform Notification Services

Description:

The system shall send configurable notifications for important customer and firm activities through approved email, SMS and WhatsApp providers.

Business Rules:

1. Notifications must use approved templates.
2. Provider integrations must support delivery tracking.
3. Failed deliveries must support controlled retries.
4. WhatsApp templates must comply with applicable provider requirements.
5. Customer preferences and applicable consent requirements must be respected.
6. Sensitive PAN, passwords, OTPs and detailed tax information must not appear in ordinary notification content.
7. OTP delivery must use dedicated secure authentication flows.

Acceptance Criteria:

- Configured notifications are generated for applicable events.
- Delivery status is tracked.
- Failed notifications follow retry rules.
- Customer preferences are respected.
- Notification content does not disclose prohibited sensitive information.

**FR-MSG-004: Notification Preferences and History**

- Priority: Medium
- Actor: Customer and Firm Admin

Description:

The system shall support permitted notification preferences and maintain notification history.

Business Rules:

1. Customers may manage eligible notification preferences.
2. Mandatory security notifications cannot be disabled where policy requires delivery.
3. Firm branding and approved templates must be supported.
4. Delivery records must include provider status and timestamps.
5. Notification history must remain tenant-scoped.

Acceptance Criteria:

- Customers can update permitted preferences.
- Mandatory notifications remain enabled.
- Authorized firm administrators can manage firm templates.
- Delivery history can be reviewed by authorized users.

**FR-MSG-005: Notification Retry and Provider Failover**

- Priority: High
- Actor: Platform Notification Services

Description:

The system shall support retry handling and provider failover for notification delivery.

Business Rules:

1. Retry policies must be configurable.
2. Duplicate notifications must be prevented where possible through idempotency controls.
3. Provider failures must be monitored.
4. Sensitive data must not be unnecessarily included in queued jobs.
5. Delivery failures must be visible to authorized administrators.

Acceptance Criteria:

- Temporary failures trigger controlled retries.
- Duplicate delivery is minimized.
- Provider outages generate operational alerts.
- Delivery attempts are traceable.

### 5.11 Customer Relationship Management and Lead Management

#### 5.11.1 Overview

The system shall provide firm-level CRM capabilities for managing prospective customers, existing customer relationships, follow-ups and service opportunities.

#### 5.11.2 Functional Requirements

**FR-CRM-001: Lead Creation and Management**

- Priority: Medium
- Actor: Firm Admin or Authorized Firm Staff

Description:

The system shall allow firms to create and manage prospective customer records.

Business Rules:

1. Leads must belong to the correct firm tenant.
2. Contact information must be validated.
3. Duplicate leads must be detected within the firm.
4. Leads must remain distinct from activated customer accounts until conversion.
5. Access must follow assigned staff permissions.

Acceptance Criteria:

- Authorized users can create leads.
- Duplicate leads are flagged.
- Lead ownership and assignment are recorded.
- Unauthorized tenant access is rejected.

**FR-CRM-002: Lead Assignment and Follow-up**

- Priority: Medium
- Actor: Firm Admin or Authorized Firm Staff

Description:

The system shall allow leads to be assigned to staff with follow-up activities and due dates.

Business Rules:

1. Lead assignments must remain within the firm's tenant.
2. Follow-up tasks must support due dates and completion status.
3. Reassignment must be audited.
4. Staff must only view leads permitted by their access scope.

Acceptance Criteria:

- Authorized users can assign leads.
- Follow-up tasks can be created and updated.
- Overdue activities can be identified.
- Assignment history is retained.

**FR-CRM-003: Lead Conversion**

- Priority: High
- Actor: Firm Admin or Authorized Firm Staff

Description:

The system shall support converting an eligible lead into a customer relationship.

Business Rules:

1. Conversion must use customer identity-matching and duplicate-detection rules.
2. Existing customer identities must be linked through secure verification.
3. Lead history must be preserved.
4. Conversion must not create duplicate shared customer identities.
5. Conversion must be associated with the correct firm tenant.

Acceptance Criteria:

- Eligible leads can be converted.
- Potential duplicate identities are flagged.
- Existing verified identities can be linked appropriately.
- Lead history remains available.
- Conversion is audited.

**FR-CRM-004: Customer Follow-up and Task Management**

- Priority: Medium
- Actor: Firm Admin or Authorized Firm Staff

Description:

The system shall provide customer-related tasks and follow-up management.

Business Rules:

1. Tasks must support assignment, priority, due date and status.
2. Tasks must be linked to authorized customer or lead records.
3. Reminders and overdue notifications must be supported.
4. Access must follow tenant and assignment permissions.
5. Task history must be auditable.

Acceptance Criteria:

- Authorized users can create and assign tasks.
- Tasks can be tracked by status and due date.
- Overdue tasks can be identified.
- Unauthorized users cannot access restricted tasks.
- Task updates are audited.

---

### 5.12 Customer Invoicing and Financial Management

#### 5.12.1 Overview

The system shall provide financial management capabilities for CA firms to manage customer invoices, payments, outstanding balances, credit notes, refunds and financial reporting.

Customer financial transactions must remain separate from SaaS subscription transactions between the platform owner and CA firms.

All financial records must be tenant-scoped, auditable and protected against unauthorized modification.

#### 5.12.2 Functional Requirements

**FR-FIN-001: Customer Invoice Creation**

- Priority: Critical
- Actor: Firm Admin or Authorized Firm Staff

Description:

The system shall allow authorized firm users to create invoices for customers.

Business Rules:

1. Every invoice must belong to one firm tenant and customer relationship.
2. Each invoice must have a unique firm-specific invoice number.
3. Invoice numbers must follow configurable firm numbering rules.
4. Invoice line items must support descriptions, quantities, rates, discounts and applicable taxes.
5. Financial calculations must use precise decimal arithmetic.
6. Invoice totals must be validated before issuance.
7. Issued invoices must not be silently overwritten.
8. Invoice creation and modification must be audited.

Acceptance Criteria:

- Authorized users can create invoices.
- Invoice numbers are unique within the firm.
- Invalid financial calculations are rejected.
- Issued invoices can be downloaded as PDF.
- Invoice history is preserved.
- Unauthorized invoice creation is blocked.

**FR-FIN-002: Invoice Status Management**

- Priority: High
- Actor: Authorized Firm Staff

Description:

The system shall support invoice lifecycle management.

Initial statuses shall include:

- Draft.
- Issued.
- Partially Paid.
- Paid.
- Overdue.
- Cancelled.
- Written Off.

Business Rules:

1. Status changes must follow valid financial rules.
2. Issued invoices must retain their original issuance history.
3. Cancelled and written-off invoices must require authorization and justification.
4. Overdue status must be determined using due dates and outstanding balances.
5. Financial history must remain available after cancellation or write-off.

Acceptance Criteria:

- Authorized users can manage permitted invoice statuses.
- Invalid status transitions are rejected.
- Overdue invoices can be identified.
- Historical invoice information remains available.
- Status changes are audited.

**FR-FIN-003: Payment Recording**

- Priority: Critical
- Actor: Firm Admin or Authorized Finance Staff

Description:

The system shall allow authorized users to record payments received from customers.

Supported payment methods shall include:

- Cash.
- Bank transfer.
- UPI.
- Cheque.
- Other configured payment methods.

Business Rules:

1. Each payment must have a unique transaction reference within the applicable firm.
2. Payment amount and date must be validated.
3. Reference numbers and supporting details must be captured where applicable.
4. Payment records must be associated with the correct customer and tenant.
5. Duplicate payment entries must be detected or flagged.
6. Payment records must not be silently overwritten.
7. Payment recording must be audited.

Acceptance Criteria:

- Authorized users can record payments.
- Supported payment methods can be selected.
- Invalid amounts and references are rejected or flagged.
- Payments are associated with the correct customer.
- Payment history is retained.

**FR-FIN-004: Partial Payment and Allocation**

- Priority: Critical
- Actor: Authorized Finance Staff

Description:

The system shall support partial payments and allocation of received payments against one or more eligible customer invoices.

Business Rules:

1. A payment may be allocated partially or fully.
2. The system must calculate remaining invoice balances accurately.
3. Unallocated payments must remain identifiable.
4. Over-allocation must be prevented.
5. Payment reversals must be recorded through controlled adjustment entries.
6. Allocation changes must be audited.

Acceptance Criteria:

- Partial payments can be recorded.
- Payments can be allocated to eligible invoices.
- Outstanding balances are recalculated correctly.
- Over-allocation is rejected.
- Allocation history is retained.

**FR-FIN-005: Payment Receipts**

- Priority: High
- Actor: Authorized Firm Staff and Customer

Description:

The system shall generate payment receipts for recorded customer payments.

Business Rules:

1. Each receipt must have a unique reference.
2. Receipts must reflect the actual recorded payment.
3. Receipt documents must be associated with the relevant customer and firm.
4. Receipt corrections must preserve historical records.
5. Customers may access only their authorized receipts.

Acceptance Criteria:

- Receipts can be generated for recorded payments.
- Receipts can be downloaded as PDF.
- Receipt details match the underlying payment.
- Unauthorized receipt access is rejected.

**FR-FIN-006: Credit Notes and Financial Adjustments**

- Priority: High
- Actor: Authorized Firm Admin or Finance Staff

Description:

The system shall support credit notes and controlled financial adjustments.

Business Rules:

1. Credit notes must reference the relevant invoice where applicable.
2. Each credit note must have a unique reference.
3. Financial adjustments must require authorization and a reason.
4. Original invoices and transactions must remain preserved.
5. Adjustments must not silently rewrite historical financial records.
6. Applicable tax and accounting rules must be reviewed before implementation.

Acceptance Criteria:

- Authorized users can create credit notes.
- Credit notes are linked to appropriate invoices.
- Financial adjustments preserve original records.
- Unauthorized adjustments are rejected.
- All adjustments are audited.

**FR-FIN-007: Customer Outstanding and Overdue Tracking**

- Priority: High
- Actor: Firm Admin, Authorized Finance Staff and Customer

Description:

The system shall calculate and display outstanding customer balances and overdue invoices.

Business Rules:

1. Outstanding balances must reflect issued invoices, allocated payments and approved adjustments.
2. Overdue calculations must use configured due dates.
3. Customer visibility must be restricted to their own financial records.
4. Firm reports must remain tenant-scoped.
5. Financial totals must use precise decimal arithmetic.

Acceptance Criteria:

- Outstanding balances are calculated correctly.
- Overdue invoices can be filtered.
- Customers can view their permitted outstanding balances.
- Firm administrators can view authorized financial summaries.
- Cross-tenant financial access is prevented.

**FR-FIN-008: Financial Transaction Audit and Reversal**

- Priority: Critical
- Actor: Platform System and Authorized Finance Staff

Description:

The system shall maintain an auditable financial transaction history.

Business Rules:

1. Issued invoices and recorded payments must not be silently deleted or overwritten.
2. Corrections must use controlled reversal, adjustment or cancellation entries.
3. Every transaction must record actor, timestamp, amount and relevant reference.
4. Financial records must preserve appropriate historical states.
5. Access to financial audit records must be restricted.
6. Audit records must be protected against ordinary user modification.

Acceptance Criteria:

- Financial transactions retain historical records.
- Corrections preserve original transaction information.
- Every adjustment identifies the responsible user.
- Unauthorized modification is prevented.
- Audit history is available to authorized users.

### 5.13 Financial Reconciliation and Reporting

#### 5.13.1 Overview

The system shall provide financial reconciliation and reporting capabilities to help firms monitor customer billing, payment collection and outstanding balances.

#### 5.13.2 Functional Requirements

**FR-REC-001: Payment Reconciliation**

- Priority: High
- Actor: Authorized Finance Staff

Description:

The system shall support reconciliation of recorded customer payments against invoices and available external payment or bank records.

Business Rules:

1. Reconciliation must not automatically modify financial records without authorization.
2. Unmatched transactions must be identifiable.
3. Differences must be recorded and investigated.
4. Reconciliation adjustments must follow controlled financial workflows.
5. Reconciliation activity must be auditable.

Acceptance Criteria:

- Authorized users can review payment reconciliation information.
- Matched and unmatched entries can be identified.
- Differences can be recorded.
- Unauthorized financial adjustments are prevented.

**FR-REC-002: Financial Reports**

- Priority: High
- Actor: Firm Admin and Authorized Finance Staff

Description:

The system shall provide tenant-scoped financial reports.

Initial reports shall include:

- Invoice register.
- Payment register.
- Outstanding receivables.
- Overdue invoices.
- Customer-wise revenue.
- Period-wise collections.
- Payment method summaries.
- Credit notes and adjustments.
- Refund reports.

Business Rules:

1. Reports must respect user permissions.
2. Financial filters must support date ranges and relevant customer or transaction attributes.
3. Exported reports must remain tenant-scoped.
4. Sensitive financial information must be protected.
5. Report generation and export activity must be audited.

Acceptance Criteria:

- Authorized users can generate permitted reports.
- Report totals reconcile with underlying records.
- Unauthorized financial information is not displayed.
- Exports are recorded in audit history.

### 5.14 Refund Management

#### 5.14.1 Overview

The system shall provide manual refund tracking for customers whose income tax refunds are recorded or monitored by their associated CA firm.

The initial release shall not depend on direct government refund API integrations.

#### 5.14.2 Functional Requirements

**FR-REF-001: Refund Record Creation**

- Priority: High
- Actor: Authorized Firm Staff

Description:

The system shall allow authorized staff to create refund tracking records for customer assessment-year records.

Business Rules:

1. Each refund record must belong to the correct customer, firm and assessment year.
2. Expected refund amounts must be distinguishable from received amounts.
3. Refund records must not be treated as official government confirmation unless supported by appropriate evidence.
4. Changes must be audited.

Acceptance Criteria:

- Authorized users can create refund records.
- Refund records are linked to the correct assessment year.
- Expected and received amounts are displayed separately.
- Unauthorized records cannot be created.

**FR-REF-002: Refund Status and History**

- Priority: High
- Actor: Authorized Firm Staff and Customer

Description:

The system shall support refund status tracking and historical updates.

Possible statuses shall include:

- Not Yet Received.
- Expected.
- Partially Received.
- Received.
- Delayed.
- Under Verification.
- Discrepancy Identified.
- Closed.

Business Rules:

1. Status changes must be authorized.
2. Expected and received amounts must be separately maintained.
3. Partial refunds must be supported.
4. Evidence and remarks may be attached.
5. Historical status changes must remain available.
6. Customers may view only their authorized refund information.

Acceptance Criteria:

- Authorized staff can update refund statuses.
- Partial receipts can be recorded.
- Historical refund changes are preserved.
- Customers can view permitted refund information.
- Unauthorized changes are rejected.

**FR-REF-003: Refund Discrepancy and Follow-up**

- Priority: Medium
- Actor: Authorized Firm Staff

Description:

The system shall support follow-up activities when refund amounts, expected dates or recorded receipts differ.

Business Rules:

1. Discrepancies must be recorded with relevant details.
2. Follow-up tasks must support assignment and due dates.
3. Supporting evidence may be attached.
4. Escalation and reminders must be supported.
5. All follow-up activity must be auditable.

Acceptance Criteria:

- Staff can record refund discrepancies.
- Follow-up tasks can be assigned.
- Overdue follow-ups can be identified.
- Evidence and resolution history are preserved.

### 5.15 Assisted Tax Calculation and Validation

#### 5.15.1 Overview

The system shall provide basic assisted tax calculation and validation capabilities to support CA staff during return preparation.

These capabilities must remain subject to professional review and approved tax rules.

The initial release shall not claim to provide a comprehensive automated tax engine.

#### 5.15.2 Functional Requirements

**FR-TAX-001: Basic Tax Calculation Support**

- Priority: High
- Actor: Authorized Firm Staff

Description:

The system shall provide configurable basic calculation tools to assist staff with supported tax calculations.

Business Rules:

1. Calculations must use approved and version-controlled tax rules.
2. Applicable assessment year and relevant tax regime must be identified.
3. Calculation inputs and outputs must be traceable.
4. Financial calculations must use precise decimal arithmetic.
5. Unsupported or uncertain cases must be flagged for professional review.
6. Calculations must not automatically submit or file returns.

Acceptance Criteria:

- Authorized staff can perform supported calculations.
- Assessment-year rules are applied correctly.
- Calculation inputs and outputs are retained.
- Unsupported cases are flagged.
- Calculation results can be reviewed by authorized professionals.

**FR-TAX-002: Tax Data Validation**

- Priority: High
- Actor: Authorized Firm Staff

Description:

The system shall validate supported tax-related information for completeness, consistency and permitted business rules.

Business Rules:

1. Validation rules must be version-controlled.
2. Validation failures must provide understandable messages.
3. Warnings must be distinguished from blocking errors.
4. Validation must not silently modify customer tax information.
5. Rule changes must undergo review and testing.

Acceptance Criteria:

- Supported validations run against entered information.
- Invalid information is flagged.
- Warnings and blocking errors are distinguishable.
- Validation results can be reviewed.
- Rule changes are traceable.

**FR-TAX-003: Calculation Review and Approval**

- Priority: Critical
- Actor: Authorized ITR Reviewer

Description:

The system shall allow authorized reviewers to examine assisted tax calculations before their results are used in an approved ITR workflow.

Business Rules:

1. Calculation review must be permission-controlled.
2. Review decisions must be recorded.
3. Changes to calculation inputs must trigger appropriate recalculation.
4. Previously approved results must retain their historical version.
5. AI-assisted suggestions must not independently approve tax calculations.

Acceptance Criteria:

- Authorized reviewers can review calculations.
- Review findings can be recorded.
- Material input changes invalidate or flag affected approval.
- Historical calculation versions remain available.
- Unauthorized approval is rejected.

### 5.16 Separation of Customer and SaaS Subscription Financials

**FR-SUBFIN-001: Financial Domain Separation**

- Priority: Critical
- Actor: Platform System

Description:

The system shall maintain complete separation between CA firm customer billing and SaaS subscription billing.

Business Rules:

1. Customer invoices and payments belong to the CA firm's financial domain.
2. SaaS subscription invoices and payments belong to the platform's subscription domain.
3. Customer financial records must not be used as platform subscription payment records.
4. Access permissions must remain separate.
5. Reporting and reconciliation must respect the relevant financial domain.
6. Domain separation must be enforced at the application and database levels.

Acceptance Criteria:

- Customer financial records are independent of subscription records.
- Unauthorized cross-domain access is rejected.
- Each financial domain has separate permissions and reporting.
- Separation is verified through integration and security tests.

---

### 5.17 Excel Import and Bulk Data Processing

#### 5.17.1 Overview

The system shall provide secure bulk import capabilities for CA firms to import customer master data and other supported records from Excel and CSV files.

The import process shall include file validation, column mapping, data validation, duplicate detection, preview, controlled processing, error reporting and import history.

#### 5.17.2 Functional Requirements

**FR-IMP-001: Standard Import Template**

- Priority: High
- Actor: Firm Admin or Authorized Firm Staff

Description:

The system shall provide downloadable standard templates for supported data import operations.

Business Rules:

1. Templates must identify mandatory and optional fields.
2. Supported data formats and field descriptions must be documented.
3. Templates must include appropriate examples.
4. Template versions must be maintained.
5. Import templates must not contain real customer personal information.

Acceptance Criteria:

- Authorized users can download the appropriate template.
- Mandatory fields are clearly identified.
- Template instructions are available.
- Unsupported template versions are identified.

**FR-IMP-002: Flexible File Upload**

- Priority: Critical
- Actor: Firm Admin or Authorized Firm Staff

Description:

The system shall support importing data from permitted `.xlsx` and `.csv` files.

Business Rules:

1. File extensions, content signatures and file sizes must be validated.
2. Malicious or unsupported files must be rejected.
3. Temporary import files must be stored securely.
4. Import access must be tenant-scoped.
5. Temporary files must be deleted according to approved retention rules.
6. Large imports must be processed using controlled background jobs.

Acceptance Criteria:

- Authorized users can upload supported files.
- Invalid files are rejected with clear messages.
- Temporary files are protected.
- Imports are associated with the correct firm.
- Upload activity is audited.

**FR-IMP-003: Column Mapping**

- Priority: High
- Actor: Firm Admin or Authorized Firm Staff

Description:

The system shall allow users to map source spreadsheet columns to supported system fields.

Business Rules:

1. Mandatory system fields must be mapped before processing.
2. Field data types must be validated.
3. Mapping configurations may be saved for reuse within the same firm.
4. Mapping configurations must not expose another firm's data.
5. Invalid mappings must be identified before import.

Acceptance Criteria:

- Users can map spreadsheet columns.
- Mandatory fields must be mapped.
- Invalid mappings are flagged.
- Authorized users can reuse permitted mapping configurations.

**FR-IMP-004: Data Validation and Duplicate Detection**

- Priority: Critical
- Actor: Platform Import Services

Description:

The system shall validate imported records and detect potential duplicates before committing data.

Business Rules:

1. Mandatory fields must be validated.
2. Data types and permitted formats must be checked.
3. PAN matching must follow the approved sensitive identity-matching process.
4. Duplicate detection must consider firm-specific records.
5. Potential identity conflicts must be flagged for manual review.
6. Validation must not silently alter source data.
7. Existing records must not be overwritten without an explicitly authorized operation.

Acceptance Criteria:

- Invalid records are identified.
- Duplicate records are flagged.
- Potential identity conflicts are reported.
- Existing records remain unchanged during preview.
- Validation results are available before import.

**FR-IMP-005: Import Preview and Approval**

- Priority: Critical
- Actor: Firm Admin or Authorized Firm Staff

Description:

The system shall provide a preview of validated import data before processing.

Business Rules:

1. Preview must identify valid, invalid, duplicate and potentially conflicting records.
2. Users must be able to review import results before committing.
3. Unauthorized users must not approve imports.
4. Preview data must remain protected and tenant-scoped.
5. The approved import scope must be recorded.

Acceptance Criteria:

- Users can review import previews.
- Validation errors are clearly displayed.
- Unauthorized approval is rejected.
- Only approved records are processed.

**FR-IMP-006: Controlled Import Processing**

- Priority: Critical
- Actor: Platform Import Services

Description:

The system shall process approved imports using controlled, recoverable and tenant-aware operations.

Business Rules:

1. Import processing must prevent cross-tenant writes.
2. Batch processing must support large files.
3. Jobs must be idempotent or use equivalent duplicate-prevention controls.
4. Partial failures must be identifiable.
5. Failed imports must support controlled recovery or rollback according to the import transaction design.
6. Existing customer information must not be silently overwritten.
7. Import processing must be auditable.

Acceptance Criteria:

- Approved records are imported into the correct tenant.
- Duplicate processing is prevented.
- Failed rows are identifiable.
- Import recovery follows defined rules.
- Existing records remain protected.
- Import activity is recorded.

**FR-IMP-007: Import Error Reports**

- Priority: High
- Actor: Firm Admin or Authorized Firm Staff

Description:

The system shall generate detailed import error reports.

Business Rules:

1. Reports must identify affected rows and fields.
2. Error messages must explain the validation problem.
3. Reports must not expose data outside the issuing firm.
4. Corrected files must be eligible for re-import.
5. Import reports must follow secure export policies.

Acceptance Criteria:

- Error reports are generated for failed records.
- Users can identify and correct errors.
- Corrected records can be re-imported.
- Unauthorized users cannot access import reports.

**FR-IMP-008: Import History and Reconciliation**

- Priority: High
- Actor: Firm Admin

Description:

The system shall maintain import history and provide reconciliation information.

Business Rules:

1. Each import operation must have a unique reference.
2. History must record uploader, time, file metadata, status and processing results.
3. Imported record counts must be available.
4. Failed and skipped records must be distinguishable.
5. Import history must be tenant-scoped and auditable.

Acceptance Criteria:

- Authorized users can view import history.
- Import results can be reconciled.
- Processing failures are traceable.
- Import history cannot be accessed by another tenant.

### 5.18 Historical Customer Data Migration

#### 5.18.1 Overview

The system shall support phased migration of existing customer information and historical ITR records from CA firms' existing systems and files.

Migration must preserve data integrity, historical relationships and document ownership.

#### 5.18.2 Functional Requirements

**FR-MIG-001: Phased Data Migration**

- Priority: Critical
- Actor: Firm Admin and Authorized Migration Personnel

Description:

The system shall support phased migration of existing firm data.

Initial migration phases may include:

1. Customer master data.
2. Customer-firm relationships.
3. Historical assessment-year records.
4. ITR preparation and filing history.
5. Financial records.
6. Customer documents and metadata.
7. Permitted historical communications and supporting information.

Business Rules:

1. Migration must be performed within an authorized tenant context.
2. Data must be mapped to approved system structures.
3. Historical records must retain relevant dates and relationships.
4. Migration must not expose another firm's records.
5. Migration progress must be traceable.

Acceptance Criteria:

- Authorized users can initiate approved migration phases.
- Migrated data is associated with the correct tenant.
- Historical relationships are preserved.
- Migration results are recorded.

**FR-MIG-002: Migration Preview and Validation**

- Priority: Critical
- Actor: Authorized Migration Personnel

Description:

The system shall validate source data and provide a preview before committing migrated records.

Business Rules:

1. Source data must be mapped and validated.
2. Duplicate and conflicting records must be identified.
3. Missing required information must be reported.
4. Existing production records must not be silently overwritten.
5. Migration approval must be recorded.

Acceptance Criteria:

- Migration preview is available.
- Validation errors and conflicts are reported.
- Existing records remain protected.
- Authorized users can approve the migration scope.

**FR-MIG-003: Migration Reconciliation and Rollback**

- Priority: Critical
- Actor: Authorized Migration Personnel

Description:

The system shall support reconciliation and controlled rollback of migration operations.

Business Rules:

1. Source and destination record counts must be compared.
2. Failed and skipped records must be reported.
3. Migration batches must be traceable.
4. Rollback must not delete unrelated records or subsequent legitimate changes.
5. Recovery operations must be authorized and audited.

Acceptance Criteria:

- Migration results can be reconciled.
- Failed batches can be identified.
- Authorized rollback procedures are available.
- Unrelated records remain unaffected.
- Migration actions are audited.

### 5.19 Dashboards and Reporting

#### 5.19.1 Overview

The system shall provide dashboards and reporting capabilities for Platform Super Admins, Firm Admins, Firm Staff and Customers according to their access permissions.

Reports must use authorized, tenant-scoped data.

#### 5.19.2 Functional Requirements

**FR-REP-001: Platform Administration Dashboard**

- Priority: High
- Actor: Platform Super Admin

Description:

The system shall provide a platform-level dashboard for monitoring SaaS operations.

Information may include:

- Total registered firms.
- Active, suspended and pending firms.
- Subscription plan distribution.
- Subscription status and renewals.
- Platform usage.
- Storage consumption.
- Operational health indicators.
- Security alerts.
- Background job health.
- Platform-level support activity.

Business Rules:

1. Platform metrics must be restricted to authorized administrators.
2. Sensitive tenant records must not be exposed through aggregate dashboards.
3. Administrative access to tenant-level information must follow privileged-access policies.
4. Dashboard access must be audited where appropriate.

Acceptance Criteria:

- Authorized administrators can view platform metrics.
- Unauthorized users cannot access platform administration dashboards.
- Tenant-sensitive details are protected.
- Metrics reflect the underlying platform data.

**FR-REP-002: Firm Administration Dashboard**

- Priority: High
- Actor: Firm Admin

Description:

The system shall provide each CA firm with an operational dashboard.

Information may include:

- Total customers.
- Active assessment-year records.
- ITR workflow status distribution.
- Pending documents.
- Returns under preparation.
- Returns under review.
- Returns ready for filing.
- Filed and completed returns.
- Staff workload.
- Outstanding invoices.
- Payment collections.
- Refund follow-ups.
- Open support tickets.

Business Rules:

1. Dashboard information must be restricted to the current firm.
2. Staff-level details must respect permission policies.
3. Financial metrics must reconcile with underlying records.
4. Dashboard filters must support relevant assessment years and periods.

Acceptance Criteria:

- Firm Admins can view their firm's operational metrics.
- Other firms' information is not accessible.
- Dashboard totals are consistent with authorized records.
- Filters produce accurate results.

**FR-REP-003: Staff Productivity Dashboard**

- Priority: Medium
- Actor: Firm Admin and Authorized Supervisor

Description:

The system shall provide permitted staff productivity information.

Information may include:

- Assigned customers.
- Active tasks.
- Completed tasks.
- Pending document reviews.
- ITR preparation workload.
- Review workload.
- Overdue follow-ups.
- Relevant activity metrics.

Business Rules:

1. Staff analytics must remain tenant-scoped.
2. Access must follow supervisory permissions.
3. Metrics must be based on recorded activities.
4. Productivity metrics must not be presented as definitive measures of professional performance without appropriate context.

Acceptance Criteria:

- Authorized supervisors can view permitted staff metrics.
- Unauthorized staff cannot access restricted colleague information.
- Metrics are traceable to recorded activities.

**FR-REP-004: Customer Dashboard**

- Priority: High
- Actor: Customer

Description:

The system shall provide customers with an overview of their authorized tax-related activities.

Information may include:

- Current assessment-year status.
- Pending document requirements.
- Recent document submissions.
- ITR preparation and filing progress.
- Invoices and outstanding balances.
- Payment history.
- Refund information.
- Recent customer messages.
- Relevant acknowledgements and receipts.

Business Rules:

1. Dashboard information must be restricted to the authenticated customer and active firm relationship.
2. Switching firm context must update all displayed data.
3. Internal firm notes must not be displayed.
4. Historical information must follow access and retention policies.

Acceptance Criteria:

- Customers can view their permitted information.
- Unauthorized customer records are not displayed.
- Firm context switching does not expose stale information.
- Dashboard information matches the underlying records.

**FR-REP-005: Standard Reports**

- Priority: High
- Actor: Firm Admin and Authorized Firm Staff

Description:

The system shall provide predefined operational and financial reports.

Initial reports shall include:

- Customer register.
- Assessment-year customer list.
- ITR status report.
- Pending document report.
- Staff assignment report.
- Staff productivity report.
- Invoice register.
- Payment register.
- Outstanding receivables.
- Refund tracking report.
- Customer activity report.
- Audit activity report.

Business Rules:

1. Reports must respect role and permission restrictions.
2. Reports must be tenant-scoped.
3. Assessment-year and date filters must be supported where applicable.
4. Report totals must reconcile with underlying data.

Acceptance Criteria:

- Authorized users can generate permitted reports.
- Filters produce correct results.
- Cross-tenant data is excluded.
- Report information is traceable.

**FR-REP-006: Custom Report Builder**

- Priority: High
- Actor: Firm Admin and Authorized Users

Description:

The system shall provide a configurable report builder for authorized users.

Business Rules:

1. Users may select permitted data fields.
2. Filters, sorting, grouping and column selection must be supported.
3. Sensitive fields must remain permission-controlled.
4. Reports must not bypass tenant isolation.
5. Saved report configurations must remain firm-specific.
6. Expensive reports may be processed asynchronously.

Acceptance Criteria:

- Authorized users can build permitted custom reports.
- Unauthorized fields cannot be selected.
- Report filters and groupings work correctly.
- Saved configurations are tenant-scoped.

**FR-REP-007: Report Export**

- Priority: High
- Actor: Authorized Firm Admin and Firm Staff

Description:

The system shall support exporting authorized reports into supported formats such as Excel, CSV and PDF.

Business Rules:

1. Exported data must follow the same access controls as on-screen reports.
2. Exports must be tenant-scoped.
3. Large exports may be generated through background jobs.
4. Download links must be short-lived and authorized.
5. Sensitive exports must be audited.
6. Export files must follow approved retention and deletion policies.

Acceptance Criteria:

- Authorized users can export permitted reports.
- Exported data matches the selected report.
- Unauthorized fields and records are excluded.
- Download links expire according to policy.
- Export activity is logged.

**FR-REP-008: Scheduled Reports**

- Priority: Medium
- Actor: Firm Admin

Description:

The system shall support scheduling recurring reports for authorized recipients.

Business Rules:

1. Schedules must support approved frequencies.
2. Recipients must be authorized to access the report.
3. Scheduled exports must respect current permissions.
4. Failed report generation must be recorded.
5. Sensitive information must not be sent through insecure channels.

Acceptance Criteria:

- Authorized users can configure report schedules.
- Reports are generated according to configured schedules.
- Unauthorized recipients are rejected.
- Scheduled report activity is traceable.

### 5.20 Global Search and Record Discovery

#### 5.20.1 Overview

The system shall provide permission-aware search and record discovery capabilities across supported business modules.

#### 5.20.2 Functional Requirements

**FR-SRCH-001: Customer and Record Search**

- Priority: High
- Actor: Authorized Users

Description:

The system shall support searching for customers and permitted business records.

Searchable information may include:

- Customer name.
- Permitted customer reference.
- Masked PAN or approved matching attributes.
- Verified contact details where authorized.
- Assessment year.
- ITR reference.
- Invoice reference.
- Document category.
- Workflow status.

Business Rules:

1. Search results must respect tenant boundaries.
2. Search results must respect user permissions and customer assignments.
3. Sensitive fields must not be exposed to unauthorized users.
4. Search queries must be validated and protected against abuse.
5. Search must not bypass backend authorization.

Acceptance Criteria:

- Authorized users can search permitted records.
- Unauthorized records are excluded.
- Sensitive information is masked or restricted appropriately.
- Search access is tenant-scoped.

**FR-SRCH-002: Search Filters and Autocomplete**

- Priority: Medium
- Actor: Authorized Users

Description:

The system shall support filters and autocomplete for supported search operations.

Business Rules:

1. Filters must use permitted record attributes.
2. Autocomplete suggestions must respect authorization.
3. Search results must remain tenant-scoped.
4. Search performance must be monitored.

Acceptance Criteria:

- Users can filter permitted search results.
- Autocomplete does not disclose unauthorized information.
- Search results remain consistent with access permissions.

**FR-SRCH-003: Search Performance and Indexing**

- Priority: Medium
- Actor: Platform System

Description:

The system shall support efficient search using appropriate database indexes and PostgreSQL full-text search capabilities.

Business Rules:

1. Search indexes must be maintained as records change.
2. Search performance must be monitored.
3. Indexing must not weaken tenant isolation.
4. More specialized search infrastructure may be introduced only when justified by measured requirements.

Acceptance Criteria:

- Common search operations meet approved performance targets.
- Indexes remain consistent with source records.
- Search results respect tenant and permission boundaries.

---

### 5.21 SaaS Subscription and Plan Management

#### 5.21.1 Overview

The system shall provide subscription management capabilities for the platform owner to manage CA firm subscriptions.

The initial release shall support manual subscription activation, payment recording and renewal management. Online subscription payments and automated renewals may be introduced in future releases.

SaaS subscription financial records must remain separate from CA firms' customer invoices and payments.

#### 5.21.2 Functional Requirements

**FR-SAAS-001: Subscription Plan Management**

- Priority: Critical
- Actor: Platform Super Admin

Description:

The system shall support configurable subscription plans for CA firms.

Initial plans shall include:

- Starter.
- Professional.
- Enterprise.

Business Rules:

1. Each plan must have configurable features and usage limits.
2. Plans must support pricing and billing-period configuration.
3. Plans may include add-ons.
4. Platform Super Admins may update plan configurations.
5. Changes to plans must not silently modify historical subscription transactions.
6. Plan configuration changes must be audited.

Acceptance Criteria:

- Authorized administrators can create and manage plans.
- Plan features and limits can be configured.
- Firms can be assigned eligible plans.
- Historical subscription information remains available.
- Configuration changes are audited.

**FR-SAAS-002: Firm Subscription Activation**

- Priority: Critical
- Actor: Platform Super Admin

Description:

The system shall allow authorized administrators to activate, renew, upgrade, downgrade and suspend CA firm subscriptions.

Business Rules:

1. Subscription activation must follow the approved firm onboarding process.
2. Each subscription must have a defined plan and validity period.
3. Manual payment recording must be supported.
4. Subscription changes must preserve historical information.
5. Suspension and reactivation must require appropriate authorization.
6. All subscription actions must be audited.

Acceptance Criteria:

- Authorized administrators can activate subscriptions.
- Subscription validity is tracked.
- Plan changes are recorded.
- Suspended firms follow configured access policies.
- Subscription history is available.

**FR-SAAS-003: Subscription Payment and Renewal Tracking**

- Priority: High
- Actor: Platform Super Admin

Description:

The system shall support manual recording and tracking of SaaS subscription payments and renewals.

Business Rules:

1. Subscription payments must belong to the correct CA firm.
2. Payment method, amount, date and reference must be recorded.
3. Renewals must be tracked against subscription validity.
4. Outstanding subscription payments must be identifiable.
5. Subscription payment corrections must preserve transaction history.
6. Customer financial records must not be mixed with SaaS subscription transactions.

Acceptance Criteria:

- Administrators can record subscription payments.
- Renewal dates and payment history are visible.
- Outstanding subscription payments can be identified.
- Unauthorized financial changes are rejected.
- Subscription transactions are audited.

**FR-SAAS-004: Subscription Plan Changes**

- Priority: High
- Actor: Platform Super Admin

Description:

The system shall support controlled subscription plan changes for CA firms.

Business Rules:

1. Upgrades and downgrades must follow configured effective-date rules.
2. Usage limits and entitlements must update according to the approved plan.
3. Existing records must not be deleted because of a plan change.
4. Changes must be auditable.
5. Any prorated billing calculations must follow separately approved business rules.

Acceptance Criteria:

- Authorized administrators can change firm plans.
- Updated entitlements are applied correctly.
- Existing customer data remains available.
- Plan change history is retained.

### 5.22 Subscription Usage and Limit Enforcement

#### 5.22.1 Overview

The system shall monitor firm usage and enforce subscription entitlements without compromising access to existing customer information.

#### 5.22.2 Functional Requirements

**FR-LIMIT-001: Usage Tracking**

- Priority: Critical
- Actor: Platform System

Description:

The system shall track firm usage against configured subscription limits.

Usage metrics may include:

- Active customers.
- Staff accounts.
- Storage consumption.
- Assessment-year records.
- Monthly or periodic usage.
- Enabled features.
- Other plan-specific entitlements.

Business Rules:

1. Usage must be calculated separately for each firm.
2. Usage metrics must be traceable.
3. Platform administrators must be able to configure applicable limits.
4. Usage records must not expose another firm's information.
5. Usage updates must be monitored for accuracy.

Acceptance Criteria:

- Usage is tracked per tenant.
- Configured limits are applied.
- Authorized administrators can review usage.
- Cross-tenant usage access is prevented.

**FR-LIMIT-002: Usage Warnings and Thresholds**

- Priority: High
- Actor: Platform System

Description:

The system shall notify firms when usage approaches configured subscription limits.

Business Rules:

1. Warning thresholds must be configurable.
2. Notifications must be sent to authorized firm administrators.
3. Usage warnings must not interrupt existing work.
4. Warning history must be retained.
5. Approved overages and exceptions must be recorded.

Acceptance Criteria:

- Usage thresholds trigger warnings.
- Authorized firm administrators receive appropriate notifications.
- Warning history is available.
- Approved exceptions are reflected in usage enforcement.

**FR-LIMIT-003: Grace Period and Overage Management**

- Priority: Critical
- Actor: Platform Super Admin

Description:

The system shall support configurable grace periods and approved usage overages.

Business Rules:

1. Grace periods must be configurable.
2. Authorized administrators may approve overages.
3. Approved exceptions must have defined scope and validity.
4. New usage may be restricted after applicable grace periods expire without approval.
5. Existing customer records, ITRs, documents, invoices and historical information must not be blocked solely because of subscription limits.
6. Subscription enforcement decisions must be audited.

Acceptance Criteria:

- Grace periods are enforced.
- Approved overages are supported.
- Unauthorized overages are restricted according to policy.
- Existing customer records remain accessible.
- Enforcement actions are auditable.

**FR-LIMIT-004: Subscription Expiry and Suspension**

- Priority: Critical
- Actor: Platform System and Platform Super Admin

Description:

The system shall manage expired, suspended and reactivated firm subscriptions.

Business Rules:

1. Expiry rules must follow configured subscription terms.
2. Renewal reminders must be supported.
3. Access restrictions must be configurable and policy-based.
4. Existing customer records and historical data must remain protected and accessible according to applicable policy.
5. Reactivation must restore authorized access.
6. Suspension and reactivation must be audited.

Acceptance Criteria:

- Expired subscriptions are identified.
- Authorized restrictions are applied.
- Existing data is preserved.
- Reactivated firms regain permitted access.
- All actions are traceable.

### 5.23 CA Firm Onboarding and Tenant Administration

#### 5.23.1 Overview

The system shall provide a controlled onboarding and administration process for CA firms.

Each approved firm shall receive an independent tenant environment within the shared SaaS platform.

#### 5.23.2 Functional Requirements

**FR-TENANT-001: Firm Registration Request**

- Priority: Critical
- Actor: Prospective CA Firm

Description:

The system shall allow prospective CA firms to submit registration requests for platform access.

Business Rules:

1. Registration requests must be stored separately from active tenant records until approval.
2. Required firm information must be validated.
3. Duplicate or suspicious registration requests must be flagged.
4. Registration requests must not automatically activate a tenant.
5. Submission must be acknowledged.

Acceptance Criteria:

- Prospective firms can submit registration requests.
- Requests enter a pending state.
- Unauthorized tenant activation is prevented.
- Registration activity is recorded.

**FR-TENANT-002: Firm Approval and Tenant Creation**

- Priority: Critical
- Actor: Platform Super Admin

Description:

The system shall allow Platform Super Admins to review and approve or reject firm registration requests.

Business Rules:

1. Tenant creation must occur only after approval.
2. Initial Firm Admin onboarding must use a secure invitation.
3. Invitations must be time-limited and single-use.
4. The invited administrator must set their own password and complete mandatory MFA setup.
5. Approval and rejection decisions must be audited.
6. Rejected requests must not create active tenants.

Acceptance Criteria:

- Authorized administrators can approve or reject requests.
- Approved firms receive a new tenant.
- Initial Firm Admin invitations are securely generated.
- Unapproved firms cannot access tenant services.
- Approval history is retained.

**FR-TENANT-003: Firm Profile Management**

- Priority: High
- Actor: Firm Admin and Platform Super Admin

Description:

The system shall support management of firm profile information.

Business Rules:

1. Firm profiles must remain tenant-specific.
2. Changes to sensitive firm information must require appropriate verification.
3. Authorized users may update permitted contact and business details.
4. Platform-level changes must be restricted to authorized administrators.
5. Changes must be audited.

Acceptance Criteria:

- Authorized users can update permitted firm information.
- Unauthorized changes are rejected.
- Firm profile history is recorded.

**FR-TENANT-004: Firm Suspension and Reactivation**

- Priority: Critical
- Actor: Platform Super Admin

Description:

The system shall allow authorized Platform Super Admins to suspend and reactivate firm tenants.

Business Rules:

1. Suspension must require authorization and a recorded reason.
2. Suspension behavior must follow defined access policies.
3. Existing tenant data must be preserved.
4. Reactivation must restore eligible access.
5. Emergency suspension actions must generate appropriate security alerts.
6. All actions must be audited.

Acceptance Criteria:

- Authorized administrators can suspend and reactivate firms.
- Suspension reasons are recorded.
- Tenant data remains protected.
- Reactivation restores permitted operations.
- Actions are auditable.

### 5.24 Customer Support and Issue Management

#### 5.24.1 Overview

The system shall provide an integrated support-ticket facility for customers and CA firm staff.

Support records must remain tenant-isolated. Internal notes and platform-only information must not be visible to unauthorized customers.

#### 5.24.2 Functional Requirements

**FR-SUP-001: Support Ticket Creation**

- Priority: High
- Actor: Customer or Authorized Firm Staff

Description:

The system shall allow customers and authorized staff to create support tickets.

Business Rules:

1. Tickets must be associated with the correct customer and tenant where applicable.
2. Ticket categories and priorities must be configurable.
3. Ticket descriptions and attachments must follow security policies.
4. Customers must not be able to view another customer's tickets.
5. Ticket creation must be recorded.

Acceptance Criteria:

- Authorized users can create tickets.
- Tickets are linked to the correct context.
- Unauthorized access is rejected.
- Ticket creation is audited.

**FR-SUP-002: Ticket Assignment and Workflow**

- Priority: High
- Actor: Firm Admin and Authorized Support Staff

Description:

The system shall support ticket assignment and status management.

Initial statuses may include:

- Open.
- Assigned.
- In Progress.
- Waiting for Customer.
- Escalated.
- Resolved.
- Closed.

Business Rules:

1. Ticket assignments must remain within the appropriate tenant or authorized platform support scope.
2. Assignment permissions must be enforced.
3. Status transitions must be controlled.
4. Escalations must be traceable.
5. Ticket history must be retained.

Acceptance Criteria:

- Authorized staff can assign tickets.
- Ticket statuses follow permitted transitions.
- Escalations can be managed.
- Ticket history is available to authorized users.

**FR-SUP-003: Internal Notes and Customer Replies**

- Priority: Critical
- Actor: Authorized Support Staff and Customer

Description:

The system shall distinguish internal support notes from customer-visible replies.

Business Rules:

1. Internal notes must never be exposed to customers.
2. Customer replies must be clearly distinguished from internal notes.
3. Access to support conversations must follow tenant permissions.
4. Attachments must follow document security controls.
5. Communication history must be audited.

Acceptance Criteria:

- Authorized staff can create internal notes.
- Customers can see only permitted replies.
- Internal notes remain inaccessible to customers.
- Unauthorized access is rejected.

**FR-SUP-004: Support SLA and Escalation**

- Priority: Medium
- Actor: Firm Admin and Platform Super Admin

Description:

The system shall support configurable support response and resolution targets.

Business Rules:

1. SLA targets must be configurable by priority and ticket category.
2. Overdue tickets must be identifiable.
3. Escalation rules must be configurable.
4. SLA events must be recorded.
5. Tenant support reporting must remain isolated.

Acceptance Criteria:

- SLA targets can be configured.
- Overdue tickets are identified.
- Escalations are triggered according to policy.
- SLA history is available.

**FR-SUP-005: Knowledge Base**

- Priority: Medium
- Actor: Firm Admin and Platform Super Admin

Description:

The system shall provide a knowledge base for approved support articles and frequently asked questions.

Business Rules:

1. Article visibility must be configurable.
2. Platform-level and firm-specific articles must be distinguishable.
3. Articles must support version history.
4. Unauthorized users must not access restricted articles.
5. Article updates must be audited.

Acceptance Criteria:

- Authorized administrators can manage articles.
- Users can search permitted knowledge content.
- Firm-specific content remains tenant-scoped.
- Article history is preserved.

### 5.25 Firm Branding and White-Label Configuration

#### 5.25.1 Overview

The system shall support configurable branding for CA firms so that each firm can provide a branded customer experience.

#### 5.25.2 Functional Requirements

**FR-BRAND-001: Firm Branding Configuration**

- Priority: High
- Actor: Firm Admin

Description:

The system shall allow authorized Firm Admins to configure permitted branding information.

Supported settings may include:

- Firm name.
- Logo.
- Contact information.
- Brand colours.
- Portal identity.
- Approved email templates.
- Other permitted branding elements.

Business Rules:

1. Branding must remain tenant-specific.
2. Uploaded logos and assets must follow file validation and security policies.
3. Platform-level branding and legal disclosures must remain protected.
4. Branding changes must not compromise accessibility or usability.
5. Changes must be audited.

Acceptance Criteria:

- Firm Admins can update permitted branding.
- Branding changes apply only to the correct firm.
- Invalid assets are rejected.
- Platform-required elements remain visible.

**FR-BRAND-002: Customer Portal Branding**

- Priority: High
- Actor: Firm Admin

Description:

The system shall display approved firm branding in the associated customer portal.

Business Rules:

1. Customer portal branding must reflect the active firm relationship.
2. Switching firms must update branding and firm context.
3. Branding must not override security indicators or required platform disclosures.
4. Unauthorized branding access is prevented.

Acceptance Criteria:

- Customers see the correct firm's branding.
- Switching firm contexts updates branding.
- Other firms' branding and records are not exposed.

**FR-BRAND-003: Custom Domain Management**

- Priority: Medium
- Actor: Firm Admin and Platform Super Admin

Description:

The system shall support optional custom domains for participating CA firms.

Business Rules:

1. Custom domains must undergo ownership verification.
2. HTTPS must be enforced.
3. Domain configuration must be validated.
4. Domain changes must be auditable.
5. A custom domain must not bypass authentication or tenant isolation.
6. Platform administrators must be able to disable invalid or unsafe configurations.

Acceptance Criteria:

- Authorized firms can request custom domains.
- Domain ownership is verified.
- HTTPS is enforced.
- Invalid configurations are rejected.
- Domain changes are audited.

### 5.26 Platform Administration and Privileged Operations

#### 5.26.1 Overview

The system shall provide a protected administration interface for Platform Super Admins to manage SaaS operations, policies, subscriptions, tenant configuration, system monitoring and security.

Platform administration must follow least-privilege principles.

#### 5.26.2 Functional Requirements

**FR-ADMIN-001: Platform Administration Dashboard**

- Priority: Critical
- Actor: Platform Super Admin

Description:

The system shall provide a centralized platform administration dashboard.

Capabilities may include:

- Firm onboarding approvals.
- Tenant status management.
- Subscription plans and renewals.
- Usage monitoring.
- Platform configuration.
- Feature flags.
- System health and operational alerts.
- Support management.
- Security events.
- Administrative activity history.

Business Rules:

1. Access must require mandatory MFA.
2. Platform administration permissions must be restricted.
3. Sensitive operations must require appropriate re-verification or approval.
4. Administrative activity must be audited.
5. Platform dashboards must not expose unrestricted customer tax information.

Acceptance Criteria:

- Authorized administrators can access permitted platform functions.
- Unauthorized users are rejected.
- Sensitive operations follow additional controls.
- Administrative activity is auditable.

**FR-ADMIN-002: Platform Configuration Management**

- Priority: High
- Actor: Platform Super Admin

Description:

The system shall support controlled configuration of platform-level settings.

Business Rules:

1. Configuration changes must be permission-controlled.
2. Critical settings must support validation before activation.
3. Configuration history must be retained.
4. Secrets must not be stored as ordinary application configuration values.
5. Configuration changes must be auditable.

Acceptance Criteria:

- Authorized administrators can update permitted settings.
- Invalid configuration is rejected.
- Historical configuration changes are available.
- Secrets remain protected.

**FR-ADMIN-003: Emergency and Privileged Tenant Access**

- Priority: Critical
- Actor: Authorized Platform Super Admin

Description:

The system shall support exceptional, controlled platform-level access to tenant resources when operationally or legally justified.

Business Rules:

1. Privileged access must follow least-privilege principles.
2. Access must require MFA and appropriate re-verification.
3. Justification and authorization must be recorded.
4. Access must be time-limited wherever possible.
5. Sensitive tax document access must not be automatically granted by the Super Admin role.
6. Emergency access must generate security alerts and detailed audit trails.
7. Access must be revocable immediately.
8. Emergency access must not bypass applicable legal or privacy obligations.

Acceptance Criteria:

- Privileged access requires authorization.
- Justification is recorded.
- Access expires or is revoked according to policy.
- Sensitive access is traceable.
- Emergency actions generate alerts.
- Unauthorized access is prevented.

**FR-ADMIN-004: Feature Flag and Tenant-Level Configuration**

- Priority: High
- Actor: Platform Super Admin

Description:

The system shall support controlled feature enablement and configuration at platform and tenant levels.

Business Rules:

1. Feature flags must be centrally managed.
2. Tenant-specific exceptions must be recorded.
3. Security-critical controls must not be disabled through ordinary feature flags.
4. Changes must support controlled rollout and rollback.
5. Feature configuration changes must be audited.

Acceptance Criteria:

- Authorized administrators can manage feature flags.
- Tenant-specific settings are isolated.
- Security controls remain protected.
- Changes can be rolled back.
- Configuration history is retained.

### 5.27 AI-Assisted Features

The initial release shall include controlled AI-assisted productivity features to help authorized users process documents, organize work, and prepare drafts. AI outputs are suggestions only and must not replace professional judgment, customer confirmation, or authorized staff approval.

**Functional requirements**

- **FR-AI-001 — Document Data Extraction:** The system shall support AI-assisted extraction of relevant information from supported uploaded documents, subject to file validation, security scanning, and authorized access.
- **FR-AI-002 — Extraction Review:** Extracted values shall be presented for human verification. Users shall be able to correct, reject, or confirm extracted information before it is saved as verified business data.
- **FR-AI-003 — Checklist Suggestions:** The system shall support AI-generated checklist suggestions based on the applicable assessment year, ITR workflow, and available customer information. Authorized users must be able to review and modify suggestions.
- **FR-AI-004 — Drafting Assistance:** The system shall support drafting assistance for permitted communications, document summaries, and internal notes. Drafts must remain editable and require user approval before being sent or treated as final.
- **FR-AI-005 — Human Approval:** AI-generated outputs shall not independently submit an ITR, mark an ITR as filed, approve a tax position, issue a customer communication, or complete another consequential workflow action.
- **FR-AI-006 — Source Traceability:** Where technically supported, extracted values and AI-generated suggestions shall identify their source documents or input information so users can verify them.
- **FR-AI-007 — Failure Handling:** The system shall clearly communicate when AI processing is unavailable, incomplete, or unsuccessful and allow users to continue the relevant workflow manually.
- **FR-AI-008 — Tenant Isolation:** AI processing shall enforce the same tenant, firm, customer, and role-based access restrictions as the rest of the platform. Information from one firm or customer shall not be exposed to another.
- **FR-AI-009 — Auditability:** The system shall record relevant AI-assisted actions, including the initiating user, affected record, operation type, timestamp, and human review or approval outcome.
- **FR-AI-010 — Provider Abstraction:** AI capabilities shall use an integration layer that allows the underlying AI provider or model to be changed without redesigning core business workflows.

**AI data protection and governance**

- Customer tax documents and personal information shall be processed only through approved AI services and configurations.
- AI providers shall not be permitted to use customer information for model training unless the platform's authorized data-governance policy and applicable consent explicitly permit it.
- AI processing shall follow the platform's retention, deletion, encryption, access-control, and audit requirements.
- AI outputs shall be clearly identified as AI-generated or AI-assisted.
- The platform shall not represent AI-generated tax interpretations or calculations as professional advice or verified tax conclusions.
- Firms shall be able to control which AI-assisted capabilities are enabled for their users, subject to platform-level governance.
- AI features shall be covered by security, privacy, accuracy, usability, and regression testing before release.

---

# 6. Security and Access Control Requirements

## 6.1 Security Objectives and Principles

### 6.1.1 Overview

The system shall be designed with security as a foundational requirement rather than as an optional feature.

The platform shall protect customer information, tax records, financial data, documents, authentication credentials and business information belonging to CA firms.

The security architecture shall follow these principles:

1. Security by design and by default.
2. Least-privilege access.
3. Zero-trust authorization.
4. Defense in depth.
5. Strict tenant isolation.
6. Secure handling of sensitive information.
7. Complete accountability for privileged operations.
8. Secure software development and deployment.
9. Continuous security monitoring.
10. Privacy by design.

### 6.1.2 Functional and Security Requirements

**SR-SEC-001: Security Architecture**

- Priority: Critical
- Actor: Platform System

Description:

The system shall implement a layered security architecture covering application, infrastructure, database, network, identity and data-storage layers.

Business Rules:

1. All security-sensitive operations must require authorization.
2. Security controls must be enforced on the server side.
3. Client-side restrictions must not be treated as security controls.
4. Sensitive operations must be logged.
5. Security controls must be tested before production releases.

Acceptance Criteria:

- Security controls exist at each applicable layer.
- Unauthorized requests are rejected.
- Security-sensitive operations are traceable.
- Security testing is included in release validation.

**SR-SEC-002: Secure Defaults**

- Priority: Critical
- Actor: Platform System

Description:

The system shall use secure default configurations for newly created accounts, tenants, services and infrastructure components.

Business Rules:

1. New accounts must receive only the permissions required for their assigned role.
2. New tenants must be isolated by default.
3. New storage resources must be private by default.
4. Administrative access must require mandatory MFA.
5. Debugging and development configurations must not be enabled in production.
6. Default credentials and publicly accessible administrative interfaces are prohibited.

Acceptance Criteria:

- New accounts receive restricted permissions.
- New tenants are isolated.
- Private storage is enforced.
- Production configuration follows approved security baselines.

## 6.2 Authentication and Identity Management

### 6.2.1 Overview

The system shall provide secure authentication for Platform Super Admins, Firm Admins, Firm Staff and Customers.

Authentication shall use a replaceable standards-compatible identity provider wherever practical.

### 6.2.2 Functional Requirements

**SR-AUTH-001: User Authentication**

- Priority: Critical
- Actor: All Users

Description:

The system shall authenticate users before granting access to protected platform functionality.

Business Rules:

1. Staff and administrators shall authenticate using email and password with mandatory MFA.
2. Customers shall be able to authenticate using password or OTP sent to a verified email address or mobile number.
3. Customer OTP authentication must not rely on unverified contact details.
4. Authentication must identify the user's active firm relationship and permitted role.
5. Failed authentication attempts must be rate-limited.
6. Authentication errors must not reveal whether a particular email address or account exists.
7. Passwords must never be stored in plaintext.

Acceptance Criteria:

- Valid credentials allow permitted access.
- Invalid credentials are rejected.
- MFA is enforced for privileged staff.
- OTP verification is required where applicable.
- Authentication attempts are logged securely.

**SR-AUTH-002: Password Security**

- Priority: Critical
- Actor: Platform System

Description:

The system shall enforce secure password creation, storage, verification and reset procedures.

Business Rules:

1. Passwords must be stored using a strong password-hashing algorithm such as Argon2id.
2. Passwords must never be stored or logged in plaintext.
3. Password policy must support strong passphrases.
4. Common and compromised passwords should be rejected.
5. Password reset tokens must be single-use and time-limited.
6. Password reset must invalidate or appropriately re-evaluate existing sessions.
7. Password changes must be recorded without exposing password contents.

Acceptance Criteria:

- Passwords are securely hashed.
- Weak or compromised passwords are rejected according to policy.
- Reset tokens expire and cannot be reused.
- Password changes are auditable.

**SR-AUTH-003: Multi-Factor Authentication**

- Priority: Critical
- Actor: Platform Super Admin, Firm Admin and Firm Staff

Description:

The system shall enforce multi-factor authentication for privileged staff and administrative accounts.

Business Rules:

1. MFA must be mandatory for Platform Super Admins.
2. MFA must be mandatory for Firm Admins and Firm Staff.
3. Customers may use optional MFA in addition to supported password or OTP login methods.
4. MFA recovery must follow a secure identity-verification process.
5. MFA changes must require appropriate verification.
6. MFA enrollment, recovery and removal must be audited.
7. Emergency recovery must not become an unrestricted authentication bypass.

Acceptance Criteria:

- Privileged accounts cannot access protected services without required MFA.
- MFA enrollment and recovery work securely.
- Unauthorized MFA changes are rejected.
- MFA-related events are recorded.

**SR-AUTH-004: Account Invitation and Activation**

- Priority: Critical
- Actor: Platform Super Admin and Firm Admin

Description:

The system shall provide secure account invitation and activation workflows.

Business Rules:

1. Invitations must be sent to verified or appropriately validated contact details.
2. Invitation tokens must be single-use and time-limited.
3. Invitations must be associated with the intended tenant and role.
4. Expired or revoked invitations must not activate accounts.
5. Account activation must follow the assigned role's authentication requirements.
6. Invitation events must be audited.

Acceptance Criteria:

- Authorized administrators can invite users.
- Invitations cannot be reused.
- Invitations cannot grant unauthorized tenant access.
- Expired invitations are rejected.
- Activation history is retained.

**SR-AUTH-005: Account Recovery**

- Priority: Critical
- Actor: All Users

Description:

The system shall provide secure account recovery procedures.

Business Rules:

1. Recovery must use verified contact channels or approved identity-verification procedures.
2. Recovery tokens must be single-use and time-limited.
3. Recovery must not disclose sensitive account information.
4. Privileged account recovery must require stronger verification.
5. Suspicious recovery attempts must generate appropriate alerts.
6. Recovery actions must be audited.

Acceptance Criteria:

- Authorized users can recover accounts.
- Unauthorized recovery attempts are rejected.
- Privileged account recovery follows stronger controls.
- Recovery events are recorded.

**SR-AUTH-006: Identity Provider Integration**

- Priority: High
- Actor: Platform System

Description:

The system shall support integration with an OIDC/OAuth 2.0-compatible identity provider.

Business Rules:

1. Authentication provider integration must be replaceable without redesigning business modules.
2. Token signatures, issuers, audiences and expiry must be validated.
3. Authentication secrets must be protected.
4. Token and session handling must follow approved security standards.
5. Provider failures must not grant unauthorized access.

Acceptance Criteria:

- Identity provider integration follows approved standards.
- Invalid tokens are rejected.
- Provider configuration is secure.
- Authentication failures are handled safely.

## 6.3 Authorization and Role-Based Access Control

### 6.3.1 Overview

The system shall implement centralized, server-side authorization using role-based access control and configurable permissions.

Authorization shall consider:

- User identity.
- Assigned role.
- Tenant membership.
- Customer assignment.
- Resource ownership.
- Requested action.
- Applicable business rules.
- Resource sensitivity.

### 6.3.2 Functional Requirements

**SR-RBAC-001: Role-Based Access Control**

- Priority: Critical
- Actor: Platform System

Description:

The system shall support the following primary roles:

1. Platform Super Admin.
2. Firm Admin.
3. Firm Staff.
4. Customer.

Business Rules:

1. Permissions must be assigned according to role.
2. Role permissions must be enforced on the backend.
3. Users must not gain permissions by changing client-side requests.
4. Role changes must take effect according to defined session and token policies.
5. Privileged role assignments must be audited.

Acceptance Criteria:

- Each role receives only permitted functionality.
- Unauthorized operations are rejected.
- Role changes are recorded.
- Backend authorization is independently tested.

**SR-RBAC-002: Configurable Staff Permissions**

- Priority: High
- Actor: Firm Admin

Description:

The system shall allow Firm Admins to configure permitted staff permissions using approved permission categories.

Business Rules:

1. Predefined roles must be available.
2. Configurable permissions must not override platform security boundaries.
3. Staff access may be restricted by customer assignment.
4. Sensitive financial, document and administrative operations must require explicit permissions.
5. Permission changes must be audited.

Acceptance Criteria:

- Authorized Firm Admins can configure permitted permissions.
- Staff receive only assigned permissions.
- Unauthorized permission escalation is prevented.
- Permission history is retained.

**SR-RBAC-003: Customer Assignment Restrictions**

- Priority: Critical
- Actor: Platform System

Description:

The system shall restrict staff access to customers according to firm-level assignment and authorization rules.

Business Rules:

1. Staff must access only customers they are authorized to handle.
2. Assignment restrictions must apply to APIs, search, reports and document access.
3. Unassigned customer records must not be exposed through indirect queries.
4. Authorized reassignment must be recorded.
5. Emergency exceptions must follow controlled authorization procedures.

Acceptance Criteria:

- Assigned staff can access permitted customers.
- Unassigned customer access is rejected unless explicitly authorized.
- Search and reports respect assignment restrictions.
- Reassignment history is available.

**SR-RBAC-004: Resource-Level Authorization**

- Priority: Critical
- Actor: Platform System

Description:

The system shall verify authorization for every protected resource and operation.

Business Rules:

1. Access checks must apply to individual records, files and actions.
2. Knowing or guessing a resource identifier must not grant access.
3. Resource authorization must include tenant and ownership checks.
4. Sensitive operations must require appropriate permissions.
5. Authorization failures must be handled securely.

Acceptance Criteria:

- Unauthorized resource access is rejected.
- Direct identifier manipulation does not bypass authorization.
- Resource-level checks are covered by security tests.

**SR-RBAC-005: Separation of Duties**

- Priority: High
- Actor: Platform System

Description:

The system shall support separation of duties for sensitive business and administrative operations.

Business Rules:

1. Sensitive actions may require review or approval by another authorized user.
2. Approval requirements must be configurable where applicable.
3. A user must not approve their own action where separation of duties is required.
4. Approval and rejection decisions must be recorded.
5. Emergency exceptions must be explicitly authorized and audited.

Acceptance Criteria:

- Configured approval workflows are enforced.
- Self-approval is prevented where prohibited.
- Approval history is retained.

## 6.4 Multi-Tenant Isolation and Database Security

### 6.4.1 Overview

The platform shall operate as a multi-tenant SaaS system with strict logical isolation between CA firms.

A security failure affecting one tenant must not expose another tenant's customers, documents, tax information, financial records or internal communications.

Tenant isolation shall be enforced at the application and database layers.

### 6.4.2 Functional and Technical Requirements

**SR-TENANT-SEC-001: Tenant Identification**

- Priority: Critical
- Actor: Platform System

Description:

The system shall identify and validate the active tenant context for every tenant-scoped operation.

Business Rules:

1. Tenant context must be derived from authenticated and authorized membership.
2. Client-supplied tenant identifiers must never be trusted without validation.
3. Tenant context must be explicitly established for database operations.
4. Missing or invalid tenant context must cause protected operations to fail closed.
5. Tenant context must not leak between requests or database transactions.

Acceptance Criteria:

- Every tenant-scoped request has a validated tenant context.
- Missing tenant context is rejected.
- Tenant context cannot be manipulated to access another firm.
- Context isolation is tested under concurrent requests.

**SR-TENANT-SEC-002: Database Row-Level Security**

- Priority: Critical
- Actor: Platform Database

Description:

The system shall implement PostgreSQL Row-Level Security (RLS) for tenant-owned data.

Business Rules:

1. Tenant-owned tables must have appropriate tenant identifiers.
2. RLS policies must enforce tenant-level data access.
3. FORCE ROW LEVEL SECURITY must be used where appropriate.
4. Application runtime database roles must not own protected tables or have BYPASSRLS privileges.
5. Tenant context must be set transaction-locally and safely.
6. Database migrations and privileged maintenance tasks must use separately controlled roles.
7. RLS policies must be reviewed and tested during database changes.

Acceptance Criteria:

- Cross-tenant queries are blocked by RLS.
- Runtime database credentials cannot bypass RLS.
- Tenant context is transaction-scoped.
- RLS behavior is verified through automated integration tests.

**SR-TENANT-SEC-003: Tenant-Aware Relationships and Constraints**

- Priority: Critical
- Actor: Platform Database

Description:

The system shall enforce tenant-aware relationships between tenant-owned database records.

Business Rules:

1. Tenant-scoped relationships must include tenant ownership validation.
2. Composite foreign keys must be used where necessary to prevent cross-tenant references.
3. Tenant identifiers must be consistent across related records.
4. Shared customer identity records must remain separate from firm-specific relationships.
5. Database constraints must prevent invalid cross-tenant associations.

Acceptance Criteria:

- Cross-tenant foreign-key relationships are rejected.
- Tenant ownership is consistent across related records.
- Shared customer identity does not expose firm-specific records.

**SR-TENANT-SEC-004: Tenant Isolation Across Application Services**

- Priority: Critical
- Actor: Platform System

Description:

Tenant isolation shall apply consistently across all application functionality.

Business Rules:

1. Tenant checks must apply to APIs, background jobs, search, reports, exports, messaging and document operations.
2. Background jobs must carry validated tenant context.
3. Cache keys must include appropriate tenant scope.
4. Search indexes must preserve tenant and authorization filters.
5. Logs and monitoring must avoid exposing sensitive cross-tenant information.
6. Shared infrastructure must not imply shared data authorization.

Acceptance Criteria:

- Tenant isolation is verified across all major modules.
- Background jobs cannot operate under an incorrect tenant.
- Cache and search results remain tenant-scoped.
- Automated cross-tenant security tests pass.

**SR-TENANT-SEC-005: Tenant Isolation Security Testing**

- Priority: Critical
- Actor: QA and Security Team

Description:

The system shall include dedicated tests to verify tenant isolation and prevent cross-tenant data access.

Business Rules:

1. Tests must cover read, create, update, delete and export operations.
2. Tests must cover direct API requests and manipulated identifiers.
3. Tests must cover document downloads and background processing.
4. Tests must cover database connection pooling and concurrent requests.
5. Tenant isolation regressions must block production release.

Acceptance Criteria:

- Automated cross-tenant tests are maintained.
- Unauthorized access attempts fail.
- Connection-pooling context leakage is tested.
- Security test failures block release.

---

## 6.5 Data Encryption and Sensitive Information Protection

### 6.5.1 Overview

The system shall protect sensitive customer, tax, financial, authentication and firm information through encryption, secure key management and controlled data access.

### 6.5.2 Functional and Technical Requirements

**SR-DATA-001: Encryption in Transit**

- Priority: Critical
- Actor: Platform System

Description:

All communications involving the platform shall use secure encrypted transport.

Business Rules:

1. HTTPS shall be enforced for browser and API communication.
2. TLS 1.2 or higher shall be required, with TLS 1.3 preferred where supported.
3. Insecure HTTP requests must be redirected or rejected.
4. Internal service communications must use appropriate encryption.
5. Secure cookie attributes and transport security headers must be configured.

Acceptance Criteria:

- Protected endpoints use HTTPS.
- Insecure communication is rejected or redirected safely.
- Security configuration is tested.

**SR-DATA-002: Encryption at Rest**

- Priority: Critical
- Actor: Platform System

Description:

The system shall encrypt sensitive information stored in databases, backups, file storage and other persistent storage systems.

Business Rules:

1. Database storage and backups must use encryption at rest.
2. Document storage must use encryption at rest.
3. Encryption must use approved, modern cryptographic algorithms.
4. Encryption configuration must be centrally managed.
5. Encryption failures must not result in unprotected data being stored.

Acceptance Criteria:

- Persistent storage is encrypted.
- Backup encryption is verified.
- Encryption configurations are documented and reviewed.

**SR-DATA-003: Sensitive Field-Level Encryption**

- Priority: Critical
- Actor: Platform System

Description:

The system shall support additional protection for particularly sensitive fields, including PAN and other designated personal or financial identifiers.

Business Rules:

1. Sensitive fields must be encrypted using an approved field-level encryption approach.
2. Encryption keys must be stored separately from application data.
3. Search and matching operations must use approved privacy-preserving methods where necessary.
4. Plaintext sensitive values must not appear in ordinary application logs.
5. Sensitive values must be masked in user interfaces unless full display is specifically authorized.
6. Key rotation must be supported.

Acceptance Criteria:

- Designated fields are encrypted.
- Unauthorized users cannot retrieve plaintext values.
- Masking is applied where required.
- Key rotation can be performed safely.

**SR-DATA-004: Encryption Key Management**

- Priority: Critical
- Actor: Platform System

Description:

The system shall use a secure key-management mechanism for encryption keys and secrets.

Business Rules:

1. Production keys must be managed using an approved KMS or equivalent key-management service.
2. Keys must not be hardcoded in source code.
3. Keys must not be committed to Git repositories.
4. Key access must be restricted and audited.
5. Key rotation and revocation procedures must be documented.
6. Different environments must use appropriately separated keys.
7. Key loss and recovery procedures must be defined.

Acceptance Criteria:

- Production keys are stored securely.
- Unauthorized key access is prevented.
- Key access is audited.
- Rotation and recovery procedures are tested.

**SR-DATA-005: Sensitive Data Exposure Prevention**

- Priority: Critical
- Actor: Platform System

Description:

The system shall minimize unnecessary exposure of sensitive information across user interfaces, APIs, logs, exports and notifications.

Business Rules:

1. API responses must return only fields required for the requested operation.
2. Sensitive identifiers must be masked where full values are unnecessary.
3. Passwords, OTPs, tokens and encryption keys must never be logged.
4. Notifications must not include confidential tax information unnecessarily.
5. Error messages must not expose internal database or infrastructure details.
6. Sensitive exports must require authorization.

Acceptance Criteria:

- Sensitive values are not exposed in unauthorized responses.
- Logs and notifications are reviewed for data leakage.
- Exports enforce access permissions.

## 6.6 Document and File Security

### 6.6.1 Overview

The system shall protect uploaded tax documents, identity documents, financial records and other attachments against unauthorized access, malicious content and accidental disclosure.

### 6.6.2 Functional and Technical Requirements

**SR-FILE-001: Private Document Storage**

- Priority: Critical
- Actor: Platform System

Description:

All customer and firm documents shall be stored in private object storage or an equivalent protected storage service.

Business Rules:

1. Publicly accessible document storage is prohibited.
2. Documents must be associated with the appropriate tenant and authorized customer relationship.
3. Storage paths and object identifiers must not independently grant access.
4. Access must be validated by the backend.
5. Document access must be audited.

Acceptance Criteria:

- Documents are not publicly accessible.
- Unauthorized document requests are rejected.
- Tenant and customer authorization is enforced.

**SR-FILE-002: File Validation and Upload Security**

- Priority: Critical
- Actor: Platform System

Description:

The system shall validate uploaded files before accepting them into the document-management workflow.

Business Rules:

1. Allowed file types must be configurable.
2. File extensions, MIME types and actual file signatures must be validated.
3. File size limits must be enforced.
4. Potentially dangerous file types must be rejected or handled under a separately approved policy.
5. Uploads must be protected against malicious filenames and path manipulation.
6. Upload authorization must be checked before processing.

Acceptance Criteria:

- Invalid files are rejected.
- File size restrictions are enforced.
- Unauthorized uploads are blocked.
- Validation failures are logged safely.

**SR-FILE-003: Malware Scanning and Quarantine**

- Priority: Critical
- Actor: Platform System

Description:

Uploaded files shall undergo malware scanning before being made available for normal use.

Business Rules:

1. Newly uploaded files must initially remain in a restricted processing state.
2. Files must be scanned using an approved malware-scanning solution.
3. Files identified as malicious must be quarantined.
4. Failed or incomplete scans must not be treated as successful.
5. Authorized administrators must be able to review quarantine events.
6. Scan results must be recorded.

Acceptance Criteria:

- Uploaded files are scanned.
- Malicious files are quarantined.
- Files with incomplete scans are not released.
- Scan history is available to authorized personnel.

**SR-FILE-004: Secure Document Access and Download**

- Priority: Critical
- Actor: Platform System

Description:

The system shall provide controlled access to stored documents.

Business Rules:

1. Document access must require authenticated and authorized requests.
2. Short-lived signed URLs may be issued after backend authorization.
3. Signed URLs must expire within configured limits.
4. Signed URLs must not be publicly discoverable through application listings or logs.
5. Document downloads must respect tenant, customer and staff assignment restrictions.
6. Sensitive document access must be audited.

Acceptance Criteria:

- Authorized users can access permitted files.
- Expired links cannot be reused.
- Unauthorized document access is rejected.
- Access events are recorded.

**SR-FILE-005: Document Versioning and Integrity**

- Priority: High
- Actor: Platform System

Description:

The system shall preserve document versions and maintain document integrity.

Business Rules:

1. Replacing a document must not silently destroy its previous version.
2. Document versions must retain relevant metadata.
3. Integrity verification using cryptographic hashes must be supported.
4. Unauthorized document modification must be prevented.
5. Version history must follow applicable retention policies.

Acceptance Criteria:

- Document replacement creates an appropriate version record.
- Previous versions remain available to authorized users.
- Integrity checks can identify unexpected changes.
- Version history is auditable.

**SR-FILE-006: Secure Temporary File Handling**

- Priority: High
- Actor: Platform System

Description:

The system shall securely manage temporary files created during uploads, downloads, imports, exports and document processing.

Business Rules:

1. Temporary files must be stored in restricted locations.
2. Temporary files must have defined expiration and cleanup policies.
3. Sensitive temporary content must not be exposed through public endpoints.
4. Failed processing must not leave unrestricted temporary files.
5. Temporary-file access must be limited to authorized services.

Acceptance Criteria:

- Temporary files are access-controlled.
- Expired temporary files are cleaned up.
- Failed processing does not expose sensitive content.

## 6.7 Audit Logging and Accountability

### 6.7.1 Overview

The system shall maintain comprehensive, tamper-resistant audit records for security-sensitive and business-critical operations.

Audit records shall support investigations, compliance reviews, operational accountability and incident response.

### 6.7.2 Functional and Technical Requirements

**SR-AUDIT-001: Comprehensive Audit Logging**

- Priority: Critical
- Actor: Platform System

Description:

The system shall record significant user, administrative, financial and system activities.

Auditable events shall include:

1. Login and logout.
2. Failed authentication attempts.
3. MFA enrollment and recovery.
4. User creation and deactivation.
5. Role and permission changes.
6. Tenant creation, suspension and reactivation.
7. Customer record creation and modification.
8. ITR workflow transitions.
9. Document uploads, downloads, replacements and deletions.
10. Invoice and payment operations.
11. Subscription changes.
12. Data exports and imports.
13. Privileged access and emergency operations.
14. Security configuration changes.

Business Rules:

1. Audit events must identify the actor and applicable tenant context.
2. Events must include timestamps and relevant resource references.
3. Audit records must not unnecessarily contain passwords, OTPs, tokens or plaintext sensitive information.
4. Audit records must be protected against unauthorized alteration.
5. Audit collection must not depend solely on client-side events.

Acceptance Criteria:

- Required activities generate audit events.
- Unauthorized users cannot modify audit records.
- Audit events contain sufficient investigation context.
- Sensitive secrets are excluded.

**SR-AUDIT-002: Audit Log Integrity and Retention**

- Priority: Critical
- Actor: Platform System

Description:

The system shall protect audit logs against unauthorized alteration and premature deletion.

Business Rules:

1. Audit records must be append-only or protected by equivalent integrity controls.
2. Audit retention must follow configured platform and legal policies.
3. Privileged access to audit records must be restricted.
4. Audit records must be included in appropriate backup and recovery procedures.
5. Retention exceptions and legal holds must be supported where required.

Acceptance Criteria:

- Unauthorized audit modification is prevented.
- Retention policies are applied.
- Authorized investigators can retrieve required records.
- Audit integrity is periodically verified.

**SR-AUDIT-003: Audit Search and Review**

- Priority: High
- Actor: Authorized Firm Admin and Platform Super Admin

Description:

The system shall provide searchable audit information according to user authorization.

Business Rules:

1. Firm Admins must see only permitted firm-level audit information.
2. Platform administrators must access platform-level events according to their permissions.
3. Search filters may include actor, date, event category and resource.
4. Sensitive data must remain masked where appropriate.
5. Audit exports must require authorization and be recorded.

Acceptance Criteria:

- Authorized users can search permitted audit records.
- Cross-tenant audit access is prevented.
- Unauthorized exports are rejected.
- Audit searches and exports are traceable.

## 6.8 Security Monitoring and Incident Management

### 6.8.1 Overview

The system shall continuously monitor relevant security events and provide mechanisms to identify, investigate and respond to security incidents.

### 6.8.2 Functional and Technical Requirements

**SR-MON-001: Security Event Monitoring**

- Priority: Critical
- Actor: Platform System

Description:

The system shall monitor security-relevant events across application, database, infrastructure and authentication services.

Business Rules:

1. Failed login patterns must be monitored.
2. Suspicious account activity must be detectable.
3. Repeated authorization failures must be monitored.
4. Unusual privileged access must be detectable.
5. Security monitoring must avoid collecting unnecessary sensitive content.
6. Monitoring failures must be identifiable.

Acceptance Criteria:

- Defined security events are monitored.
- Monitoring alerts can be generated.
- Monitoring failures are detectable.
- Sensitive information is protected.

**SR-MON-002: Security Alerts**

- Priority: Critical
- Actor: Platform System

Description:

The system shall generate security alerts for defined suspicious or high-risk events.

Example events:

- Repeated failed login attempts.
- Suspicious password recovery.
- Unusual MFA changes.
- Privileged role changes.
- Emergency tenant access.
- Unusual document download activity.
- Repeated cross-tenant authorization failures.
- Unexpected database or infrastructure access.

Business Rules:

1. Alert severity must be configurable.
2. Alerts must be routed to authorized recipients.
3. Alerts must not disclose sensitive information unnecessarily.
4. Alert acknowledgement and resolution must be tracked.
5. Critical alerts must follow defined escalation procedures.

Acceptance Criteria:

- Defined security events generate alerts.
- Alerts reach authorized recipients.
- Alert status and history are retained.

**SR-MON-003: Security Incident Management**

- Priority: Critical
- Actor: Platform Super Admin and Authorized Security Personnel

Description:

The system shall support the recording, investigation, escalation and resolution of security incidents.

Business Rules:

1. Incidents must have severity and status classifications.
2. Incident records must include relevant timestamps and evidence references.
3. Access to incident records must be restricted.
4. Response actions must be documented.
5. Relevant tenant administrators must be notified according to incident policy and applicable legal obligations.
6. Incident closure must include resolution details.

Acceptance Criteria:

- Incidents can be created and tracked.
- Authorized personnel can investigate incidents.
- Response actions are recorded.
- Incident history is retained.

**SR-MON-004: Incident Response and Recovery**

- Priority: Critical
- Actor: Platform Super Admin and Operations Team

Description:

The system shall support coordinated response to security incidents, including containment, investigation, recovery and post-incident review.

Business Rules:

1. Incident response procedures must be documented.
2. Compromised accounts and credentials must be capable of being disabled or revoked.
3. Affected sessions and access tokens must be revocable.
4. Affected services may be isolated where necessary.
5. Recovery actions must preserve evidence where appropriate.
6. Post-incident reviews must identify corrective actions.
7. Incident response must be coordinated with backup and disaster recovery procedures.

Acceptance Criteria:

- Incident containment actions can be performed.
- Compromised access can be revoked.
- Recovery actions are traceable.
- Post-incident corrective actions are recorded.

---

## 6.9 Privacy, Consent and Personal Data Governance

### 6.9.1 Overview

The system shall protect personal information belonging to customers, firm staff, CA firms and other individuals whose information is processed through the platform.

The platform shall support applicable Indian data protection and privacy requirements, including the Digital Personal Data Protection Act, 2023 and applicable rules and regulations, as they become applicable.

Privacy controls shall be incorporated into system design, data processing, customer communication and administrative workflows.

### 6.9.2 Functional Requirements

**SR-PRIV-001: Personal Data Inventory and Classification**

- Priority: Critical
- Actor: Platform System

Description:

The system shall classify information according to its sensitivity and intended use.

Data categories shall include:

1. Personal identification information.
2. PAN and tax-related information.
3. Financial information.
4. Authentication and security information.
5. Customer documents.
6. Communication records.
7. Firm business information.
8. Operational and audit information.

Business Rules:

1. Each category must have defined access and handling requirements.
2. Sensitive data must receive stronger protection.
3. Data collection must be limited to legitimate business purposes.
4. Data classification must be documented.
5. Classification changes must be reviewed and recorded.

Acceptance Criteria:

- Data categories are documented.
- Sensitive data is identified.
- Access and protection rules are defined.

**SR-PRIV-002: Consent and Privacy Notice Management**

- Priority: Critical
- Actor: Platform System and Authorized Firm Admin

Description:

The system shall support appropriate privacy notices and consent collection wherever required.

Business Rules:

1. Privacy notices must identify relevant data processing purposes.
2. Consent must be recorded where consent is the applicable legal basis.
3. Consent records must include relevant notice version, timestamp, user reference and purpose.
4. Consent withdrawal must be supported where applicable.
5. Withdrawal must not automatically terminate processing that has another valid legal basis.
6. Privacy notices and consent wording must be version-controlled.
7. Firms must not be able to remove mandatory platform privacy disclosures.

Acceptance Criteria:

- Applicable notices can be presented.
- Consent records are retained.
- Consent withdrawal is supported where applicable.
- Historical notice versions remain traceable.

**SR-PRIV-003: Purpose Limitation and Data Minimization**

- Priority: Critical
- Actor: Platform System

Description:

The system shall process personal information only for documented and authorized purposes.

Business Rules:

1. Data collection must be limited to necessary information.
2. Information must not be reused for unrelated purposes without an appropriate legal basis.
3. Optional information must be distinguishable from mandatory information.
4. Access must be restricted according to business purpose and role.
5. Unnecessary collection and retention must be avoided.

Acceptance Criteria:

- Data collection fields have documented purposes.
- Unnecessary information is not required.
- Access follows defined processing purposes.

**SR-PRIV-004: Customer Privacy Rights and Requests**

- Priority: Critical
- Actor: Customer and Authorized Privacy Administrator

Description:

The system shall provide a controlled process for handling applicable personal data rights requests.

Supported request categories may include:

- Access to personal information.
- Correction of inaccurate information.
- Updating personal information.
- Withdrawal of consent where applicable.
- Erasure where legally permitted.
- Grievance or privacy complaint submission.

Business Rules:

1. Requests must be linked to a verified requester.
2. Requests must have status, submission date and resolution history.
3. Requests must be routed to the responsible data fiduciary or authorized representative according to the applicable arrangement.
4. Requests must follow applicable legal timelines.
5. Requests must be reviewed for legal holds, statutory retention obligations and third-party rights.
6. Requests must not result in unauthorized access to another customer's information.

Acceptance Criteria:

- Applicable privacy requests can be submitted and tracked.
- Requester identity is verified.
- Authorized personnel can process requests.
- Decisions and resolution details are recorded.

**SR-PRIV-005: Data Processing Accountability**

- Priority: High
- Actor: Platform Super Admin and Firm Admin

Description:

The system shall support documentation of relevant data processing activities and responsibilities.

Business Rules:

1. Platform and firm responsibilities must be defined according to contractual and legal arrangements.
2. Data processing purposes must be documented.
3. Relevant service providers and subprocessors must be identifiable.
4. Data access and processing activities must be traceable.
5. Material changes to processing arrangements must be reviewed.

Acceptance Criteria:

- Processing responsibilities are documented.
- Relevant service providers can be identified.
- Processing records are available to authorized personnel.

**SR-PRIV-006: Privacy Grievance Management**

- Priority: High
- Actor: Customer and Authorized Privacy Administrator

Description:

The system shall support submission, assignment, investigation and resolution of privacy-related grievances.

Business Rules:

1. Grievances must have unique reference numbers.
2. Grievance status and communication history must be retained.
3. Access must be restricted to authorized personnel.
4. Resolution must follow applicable policies and legal requirements.
5. Grievance handling must be auditable.

Acceptance Criteria:

- Customers can submit privacy grievances.
- Authorized personnel can manage grievances.
- Resolution history is retained.

## 6.10 Data Retention, Archival, Export and Deletion

### 6.10.1 Overview

The system shall provide controlled data lifecycle management covering retention, archival, backup, export and deletion.

Retention periods shall be configurable according to applicable legal obligations, firm policies, contractual commitments and platform requirements.

### 6.10.2 Functional Requirements

**SR-RET-001: Configurable Data Retention Policies**

- Priority: Critical
- Actor: Platform Super Admin

Description:

The system shall support configurable retention policies for different data categories.

Business Rules:

1. Retention periods must be configurable by data category.
2. Platform policies may define minimum mandatory retention requirements.
3. Firms may configure longer retention periods where permitted.
4. Legal and regulatory retention obligations must take precedence over ordinary deletion requests.
5. Retention changes must be audited.
6. Retention policies must cover documents, financial records, communications, audit logs and other applicable records.

Acceptance Criteria:

- Retention policies can be configured.
- Different data categories can have different periods.
- Policy changes are recorded.
- Retention enforcement can be tested.

**SR-RET-002: Secure Data Archival**

- Priority: High
- Actor: Platform System

Description:

The system shall support secure archival of records that are no longer part of routine operational activity but must be retained.

Business Rules:

1. Archived data must remain protected.
2. Archived records must be accessible only to authorized users.
3. Archived records must retain necessary ownership and tenant context.
4. Archival must preserve relevant metadata and integrity.
5. Archived records must follow applicable retention and legal hold policies.

Acceptance Criteria:

- Eligible records can be archived.
- Archived records remain protected.
- Authorized users can retrieve permitted records.
- Archival activity is audited.

**SR-RET-003: Customer Data Export and Portability**

- Priority: Critical
- Actor: Firm Admin and Authorized Platform Administrator

Description:

The system shall support secure export of firm and customer data for legitimate portability and business continuity purposes.

Business Rules:

1. Export scope must be configurable.
2. Exports must respect tenant and customer authorization.
3. Export formats may include CSV, Excel, JSON and approved document formats.
4. Sensitive exports must require appropriate authorization.
5. Export files must be encrypted or otherwise securely protected.
6. Export links must be time-limited.
7. Export activity must be audited.
8. A firm must not be able to export another firm's records.

Acceptance Criteria:

- Authorized users can export permitted information.
- Export scope is enforced.
- Cross-tenant export is prevented.
- Export events are recorded.

**SR-RET-004: Controlled Data Deletion and Erasure**

- Priority: Critical
- Actor: Authorized Privacy Administrator

Description:

The system shall support controlled deletion or erasure of personal information where permitted and required.

Business Rules:

1. Deletion requests must be verified.
2. Deletion must consider statutory retention requirements, legal holds and contractual obligations.
3. Shared customer identity and firm-specific relationships must be handled separately.
4. Deleting one firm's relationship must not automatically delete another firm's independent records.
5. Financial and audit records subject to retention obligations must be handled according to applicable policy.
6. Deletion must cover relevant primary storage and follow defined backup expiry procedures.
7. Deletion decisions and execution must be auditable.

Acceptance Criteria:

- Eligible information can be deleted.
- Records subject to legal holds are protected.
- Other firms' records remain unaffected.
- Deletion activity is recorded.

**SR-RET-005: Legal Hold Management**

- Priority: Critical
- Actor: Authorized Platform Super Admin and Privacy Administrator

Description:

The system shall support placing relevant records under a legal or regulatory hold.

Business Rules:

1. Legal holds must identify the affected records or data scope.
2. Holds must include a reason, authorizing person and applicable dates.
3. Records under active hold must not be deleted through ordinary retention processes.
4. Hold removal must require authorization.
5. All hold actions must be audited.

Acceptance Criteria:

- Authorized personnel can create legal holds.
- Held information is protected from ordinary deletion.
- Hold release is controlled and traceable.

**SR-RET-006: Backup Retention and Data Lifecycle**

- Priority: High
- Actor: Platform System

Description:

The system shall align backup retention and expiry with approved data lifecycle policies.

Business Rules:

1. Backups must remain encrypted and access-controlled.
2. Backup retention periods must be defined.
3. Expired backups must be securely disposed of according to policy.
4. Restoration must not unintentionally reintroduce data that must remain deleted without appropriate reconciliation.
5. Backup and deletion processes must be documented and tested.

Acceptance Criteria:

- Backup retention is configured.
- Backup access is restricted.
- Expiry and restoration procedures are tested.

## 6.11 API Security and Integration Protection

### 6.11.1 Overview

The system shall protect internal and external APIs against unauthorized access, abuse, data exposure and common application security threats.

APIs shall follow secure design and implementation practices.

### 6.11.2 Functional and Technical Requirements

**SR-API-001: API Authentication and Authorization**

- Priority: Critical
- Actor: Platform System

Description:

All protected APIs shall require appropriate authentication and authorization.

Business Rules:

1. APIs must validate authentication tokens or credentials.
2. Token issuer, audience, signature and expiry must be verified where applicable.
3. Authorization must be evaluated for every protected operation.
4. Tenant and customer assignment restrictions must apply to APIs.
5. Public endpoints must be explicitly identified and protected against abuse.
6. Authentication and authorization failures must not expose sensitive information.

Acceptance Criteria:

- Protected endpoints reject unauthenticated requests.
- Unauthorized operations are rejected.
- Tenant context is validated.
- Invalid or expired tokens are rejected.

**SR-API-002: API Input Validation**

- Priority: Critical
- Actor: Platform System

Description:

The system shall validate all incoming API data using strict server-side validation rules.

Business Rules:

1. Request parameters, headers, query values and payloads must be validated.
2. Data types, lengths, ranges and allowed values must be enforced.
3. Unexpected fields must be handled according to approved validation policy.
4. Database queries must use safe parameterization or equivalent protections.
5. Validation failures must return safe and consistent errors.

Acceptance Criteria:

- Invalid requests are rejected.
- Malformed data cannot bypass validation.
- Injection vulnerabilities are tested.
- Error responses do not disclose internal details.

**SR-API-003: Rate Limiting and Abuse Prevention**

- Priority: Critical
- Actor: Platform System

Description:

The system shall enforce rate limits and abuse-prevention controls on APIs.

Business Rules:

1. Rate limits must be configurable by endpoint and user or client category.
2. Login, OTP, password recovery and other sensitive endpoints must have stricter limits.
3. Suspicious request patterns must be monitored.
4. Rate-limit enforcement must not permit one tenant to exhaust shared resources without detection.
5. Temporary restrictions must follow documented recovery procedures.

Acceptance Criteria:

- Configured rate limits are enforced.
- Sensitive endpoints receive additional protection.
- Abuse attempts are detectable.
- Rate-limit events are logged safely.

**SR-API-004: API Versioning and Compatibility**

- Priority: High
- Actor: Platform System

Description:

The system shall support controlled API versioning and backward-compatible changes where practical.

Business Rules:

1. API versions must be identifiable.
2. Breaking changes must follow a documented release and deprecation process.
3. Deprecated versions must have defined support periods.
4. API documentation must reflect released functionality.
5. Version changes must be tested before deployment.

Acceptance Criteria:

- API versions are identifiable.
- Changes follow the approved compatibility process.
- Deprecated endpoints are managed.
- Documentation remains current.

**SR-API-005: API Documentation and Schema Validation**

- Priority: High
- Actor: Development Team and Platform System

Description:

The system shall maintain machine-readable API documentation using OpenAPI.

Business Rules:

1. Protected API operations must document authentication requirements.
2. Request and response schemas must be defined.
3. Error responses must be documented.
4. Tenant and permission requirements must be described.
5. API specifications must be validated during CI/CD.

Acceptance Criteria:

- API specifications are available.
- Schemas are validated.
- Documentation matches implemented endpoints.
- Invalid specification changes fail the relevant pipeline checks.

**SR-API-006: Webhook Security**

- Priority: High
- Actor: Platform System

Description:

The system shall support secure webhook delivery for approved integrations.

Business Rules:

1. Webhook destinations must be validated.
2. Webhook messages must be authenticated using signatures or equivalent controls.
3. Delivery attempts must be logged.
4. Retries must use controlled backoff and limits.
5. Duplicate delivery must be handled safely through idempotency.
6. Sensitive data must not be included unnecessarily.
7. Webhook endpoints must be protected against server-side request forgery and unsafe destinations.

Acceptance Criteria:

- Webhook messages can be authenticated.
- Invalid destinations are rejected.
- Retries and failures are traceable.
- Duplicate events do not cause unintended repeated actions.

**SR-API-007: Integration Provider Abstraction**

- Priority: High
- Actor: Platform System

Description:

The system shall use replaceable integration interfaces for external providers such as email, SMS, WhatsApp, payment and identity services.

Business Rules:

1. Provider-specific logic must be isolated from core business modules.
2. Provider credentials must be securely managed.
3. Provider failures must be handled without bypassing security controls.
4. Integration changes must be tested.
5. Provider-specific data must be mapped to internal standardized formats.

Acceptance Criteria:

- Providers can be replaced through controlled configuration or implementation changes.
- Core business workflows remain independent of provider-specific code.
- Provider errors are handled safely.

## 6.12 Session and Device Security

### 6.12.1 Overview

The system shall securely manage authenticated sessions, device information, session expiry and account access revocation.

### 6.12.2 Functional Requirements

**SR-SESSION-001: Secure Session Management**

- Priority: Critical
- Actor: Platform System

Description:

The system shall maintain authenticated sessions using secure session-management mechanisms.

Business Rules:

1. Session identifiers must be generated securely.
2. Authentication cookies must use appropriate Secure, HttpOnly and SameSite attributes.
3. Session identifiers must be rotated after authentication and privilege changes where applicable.
4. Sessions must not expose sensitive authentication information.
5. Session validation must be performed server-side.

Acceptance Criteria:

- Sessions use secure identifiers.
- Cookie security attributes are enforced where cookies are used.
- Session rotation works as specified.
- Invalid sessions are rejected.

**SR-SESSION-002: Session Expiry and Inactivity Timeout**

- Priority: Critical
- Actor: Platform System

Description:

The system shall support configurable session expiry and inactivity timeouts.

Business Rules:

1. Privileged users must have stricter session policies.
2. Absolute session lifetime must be enforced.
3. Inactivity timeout must be configurable.
4. Expired sessions must require re-authentication.
5. Sensitive operations may require additional verification.

Acceptance Criteria:

- Sessions expire according to policy.
- Inactive sessions are terminated.
- Privileged sessions follow stricter limits.

**SR-SESSION-003: Active Session and Device Management**

- Priority: High
- Actor: All Users

Description:

The system shall allow users to review and manage their active sessions where appropriate.

Business Rules:

1. Users may view relevant session information.
2. Users may revoke individual sessions or all other sessions.
3. Session information must not expose sensitive device identifiers unnecessarily.
4. Suspicious session activity must be flagged.
5. Session revocation must take effect within the defined security policy.

Acceptance Criteria:

- Users can review permitted session information.
- Session revocation works.
- Revoked sessions cannot continue accessing protected services.

**SR-SESSION-004: Suspicious Login and Risk-Based Controls**

- Priority: High
- Actor: Platform System

Description:

The system shall identify potentially suspicious login activity and apply additional security controls.

Business Rules:

1. Unusual login patterns may trigger additional verification.
2. Repeated failed attempts may trigger temporary restrictions.
3. Privileged accounts must receive enhanced monitoring.
4. Users may receive security notifications about significant account events.
5. Risk-based decisions must not rely on a single unreliable indicator.
6. Security events must be recorded without unnecessarily retaining sensitive tracking information.

Acceptance Criteria:

- Defined suspicious patterns can trigger additional controls.
- Privileged accounts receive enhanced monitoring.
- Security notifications are generated where configured.
- Events are auditable.

**SR-SESSION-005: Immediate Access Revocation**

- Priority: Critical
- Actor: Platform Super Admin and Authorized Firm Admin

Description:

The system shall support immediate or policy-bounded revocation of user access when accounts are compromised, deactivated or otherwise unauthorized.

Business Rules:

1. Authorized administrators may deactivate accounts.
2. Active sessions and relevant refresh tokens must be revoked.
3. Revoked users must not continue accessing protected services through existing sessions.
4. Revocation must apply to relevant tenant memberships.
5. Access revocation must be audited.
6. Revoking one firm's membership must not automatically revoke unrelated memberships unless the account itself is globally disabled.

Acceptance Criteria:

- Authorized administrators can revoke access.
- Existing sessions are invalidated according to policy.
- Tenant-specific membership revocation preserves unrelated memberships.
- Revocation history is retained.

### 6.13 AI Security, Privacy and Governance

- **SEC-AI-001 — Approved AI Providers:** The platform shall permit AI processing only through AI providers and configurations approved by the Platform Super Admin.
- **SEC-AI-002 — Data Minimization:** Only the minimum information necessary for a specific AI operation shall be shared with an AI provider. Sensitive information shall be masked or excluded wherever technically feasible.
- **SEC-AI-003 — Tenant and Access Isolation:** AI requests, processing jobs, temporary files, outputs, and stored results shall enforce tenant, firm, customer, and role-based access restrictions.
- **SEC-AI-004 — Provider Data Usage:** AI provider configurations and contractual arrangements shall prohibit using customer data for model training unless explicitly permitted by the platform's approved data-governance policy and applicable consent.
- **SEC-AI-005 — Secure Processing:** AI inputs and outputs shall be protected in transit and at rest, and temporary processing data shall follow approved retention and deletion policies.
- **SEC-AI-006 — Human Oversight:** AI outputs shall not bypass existing authorization, verification, approval, or workflow controls. Consequential actions shall require explicit action by an authorized human user.
- **SEC-AI-007 — Audit Trail:** AI operations shall produce auditable records without unnecessarily copying sensitive document contents or personal information into logs.
- **SEC-AI-008 — Prompt and Output Security:** AI integrations shall apply safeguards against prompt injection, malicious document instructions, unauthorized data disclosure, and unsafe output handling.
- **SEC-AI-009 — Provider Failure and Fallback:** AI provider failures, timeouts, and degraded service shall not compromise stored records or prevent users from completing supported workflows manually.
- **SEC-AI-010 — AI Security Testing:** AI-assisted features shall undergo security, privacy, tenant-isolation, output-validation, and abuse-case testing before production release.

---

# 7. Infrastructure, Performance and Reliability Requirements

## 7.1 Cloud Infrastructure and Deployment Architecture

### 7.1.1 Overview

The system shall be deployed using a secure, scalable and maintainable cloud infrastructure suitable for a commercial multi-tenant SaaS platform.

The architecture shall support independent development, testing, staging and production environments.

### 7.1.2 Functional and Technical Requirements

**NFR-INFRA-001: Cloud Deployment**

- Priority: Critical
- Actor: Platform Operations Team

Description:

The system shall support deployment on a reputable cloud infrastructure provider.

Business Rules:

1. The architecture shall support deployment on AWS, Microsoft Azure or an equivalent provider.
2. Production infrastructure must use approved secure configurations.
3. Application, database, storage and network resources must be appropriately separated.
4. Infrastructure changes must be controlled and documented.
5. Production resources must not depend on developer workstations.

Acceptance Criteria:

- The application can be deployed to the selected cloud provider.
- Infrastructure components are documented.
- Production resources follow approved security configurations.

**NFR-INFRA-002: Environment Separation**

- Priority: Critical
- Actor: Development and Operations Team

Description:

The system shall maintain separate environments for development, testing, staging, pre-production and production.

Business Rules:

1. Environment configurations must be separated.
2. Production credentials must never be reused in development.
3. Production customer data must not be copied into lower environments without approved masking or anonymization.
4. Environment-specific secrets must be managed securely.
5. Deployment permissions must be restricted.

Acceptance Criteria:

- Environments are independently configurable.
- Credentials and secrets are separated.
- Production data is protected.
- Deployment access is controlled.

**NFR-INFRA-003: Containerized Application Deployment**

- Priority: High
- Actor: Development and Operations Team

Description:

The system shall support containerized deployment using Docker and compatible orchestration or managed container services.

Business Rules:

1. Application containers must use controlled and reproducible builds.
2. Production containers must not run with unnecessary privileges.
3. Container images must be scanned for known vulnerabilities.
4. Image versions must be traceable.
5. Deployment configurations must be version-controlled.

Acceptance Criteria:

- Application services can be built and deployed using containers.
- Container images are scanned.
- Builds are reproducible.
- Deployment versions are traceable.

**NFR-INFRA-004: Infrastructure as Code**

- Priority: High
- Actor: Operations Team

Description:

The system shall support infrastructure provisioning and configuration through version-controlled Infrastructure as Code where practical.

Business Rules:

1. Infrastructure configuration must be version-controlled.
2. Changes must undergo review.
3. Production changes must follow approval procedures.
4. Sensitive values must not be embedded in infrastructure source files.
5. Infrastructure changes must support controlled rollback.

Acceptance Criteria:

- Infrastructure configuration is documented and version-controlled.
- Changes can be reviewed before deployment.
- Secrets are protected.
- Rollback procedures are documented.

**NFR-INFRA-005: Network Security and Segmentation**

- Priority: Critical
- Actor: Platform Operations Team

Description:

The system shall use secure network architecture and access restrictions.

Business Rules:

1. Databases must not be exposed directly to the public internet.
2. Private services must use restricted network access.
3. Inbound and outbound traffic must follow approved firewall rules.
4. Administrative access must use secure channels.
5. Network changes must be audited.

Acceptance Criteria:

- Database access is restricted.
- Public endpoints are explicitly controlled.
- Network security rules are documented.
- Unauthorized network access is prevented.

## 7.2 Scalability and Performance

### 7.2.1 Overview

The platform shall support increasing numbers of CA firms, staff members, customers, assessment-year records, documents and concurrent users without requiring a fundamental architectural redesign.

Initial planning shall account for approximately 650–1,200 or more customer records, with capacity to scale beyond the initial deployment.

### 7.2.2 Non-Functional Requirements

**NFR-PERF-001: Application Response Time**

- Priority: Critical
- Actor: Platform System

Description:

The system shall provide responsive user interactions under expected operating conditions.

Initial performance targets:

1. Common read operations should achieve a 95th-percentile response time of 2 seconds or less.
2. Common write operations should achieve a 95th-percentile response time of 3 seconds or less.
3. Dashboard and reporting operations may use separately defined performance targets.
4. Large imports, exports and document-processing operations shall run asynchronously where appropriate.
5. Performance measurements must exclude or separately identify external provider delays.

Acceptance Criteria:

- Performance tests measure response times.
- Common operations meet approved targets under the defined test load.
- Slow operations can be identified through monitoring.

**NFR-PERF-002: Concurrent User Support**

- Priority: Critical
- Actor: Platform System

Description:

The platform shall support concurrent access by customers, staff and administrators.

Business Rules:

1. Capacity targets must be defined during performance planning.
2. Database connections must be managed efficiently.
3. Tenant context must remain isolated under concurrent workloads.
4. Shared resources must be protected against excessive consumption by one tenant.
5. Load testing must cover concurrent authentication and business operations.

Acceptance Criteria:

- Concurrent-user tests are completed.
- No cross-tenant context leakage occurs.
- Performance remains within approved thresholds.

**NFR-PERF-003: Horizontal Scalability**

- Priority: High
- Actor: Platform Operations Team

Description:

Application services shall support horizontal scaling where appropriate.

Business Rules:

1. Application instances should avoid relying on local persistent state.
2. Shared session and job state must use suitable shared services.
3. Scaling must preserve tenant isolation.
4. Scaling policies must consider CPU, memory, request load and queue depth.
5. Database scaling limitations must be documented.

Acceptance Criteria:

- Additional application instances can be deployed.
- Scaling does not break authentication or tenant context.
- Application state remains consistent.

**NFR-PERF-004: Database Performance**

- Priority: Critical
- Actor: Platform System

Description:

The database architecture shall support efficient queries and transactions as the platform grows.

Business Rules:

1. Appropriate indexes must be created for frequently queried fields.
2. Tenant identifiers must be considered in tenant-scoped query design.
3. Slow queries must be monitored.
4. Database connection pooling must be configured.
5. Query performance must be tested with realistic data volumes.
6. Database optimizations must not weaken RLS or authorization controls.

Acceptance Criteria:

- Critical database queries meet approved performance targets.
- Slow queries can be identified.
- Connection pooling is tested.
- Tenant security remains intact.

**NFR-PERF-005: Background Processing**

- Priority: High
- Actor: Platform System

Description:

The system shall support asynchronous processing for long-running or resource-intensive tasks.

Applicable tasks may include:

- Excel and CSV imports.
- Bulk exports.
- Email and SMS notifications.
- Document scanning.
- Report generation.
- Data migration.
- Scheduled maintenance tasks.

Business Rules:

1. Background jobs must carry validated tenant context.
2. Job execution must be retryable where appropriate.
3. Duplicate job execution must not cause unintended repeated business operations.
4. Failed jobs must be identifiable.
5. Job status must be available to authorized users.
6. Background workers must follow least-privilege access.

Acceptance Criteria:

- Long-running operations execute asynchronously.
- Job failures can be identified.
- Retry behavior is controlled.
- Tenant isolation is preserved.

**NFR-PERF-006: Caching**

- Priority: High
- Actor: Platform System

Description:

The system shall support caching to improve performance where appropriate.

Business Rules:

1. Cache entries containing tenant data must be tenant-scoped.
2. Sensitive information must not be cached unnecessarily.
3. Authorization-sensitive information must be invalidated when permissions change.
4. Cache expiry must be configurable.
5. Cache failure must not cause unauthorized access or incorrect financial transactions.

Acceptance Criteria:

- Appropriate data is cached.
- Tenant-specific cache isolation is verified.
- Expired or invalidated entries are not incorrectly reused.
- Cache failures are handled safely.

## 7.3 Availability, Reliability and Fault Tolerance

### 7.3.1 Overview

The platform shall be designed to provide reliable access to critical business operations.

The initial production availability target shall be 99.9% monthly uptime, excluding only explicitly defined and contractually permitted exclusions.

### 7.3.2 Non-Functional Requirements

**NFR-AVAIL-001: Service Availability**

- Priority: Critical
- Actor: Platform Operations Team

Description:

The production platform shall target 99.9% monthly availability.

Business Rules:

1. Availability must be measured using defined monitoring endpoints and service-level indicators.
2. Planned maintenance must follow an approved communication policy.
3. Availability reports must be maintained.
4. Critical service outages must generate alerts.
5. Availability must be reviewed periodically.

Acceptance Criteria:

- Availability is continuously monitored.
- Monthly availability can be calculated.
- Outages are recorded.
- Availability reports are available.

**NFR-AVAIL-002: Fault Tolerance**

- Priority: Critical
- Actor: Platform System

Description:

The system shall minimize service disruption caused by individual infrastructure or application component failures.

Business Rules:

1. Critical components must have documented failure-handling procedures.
2. Temporary service failures must not corrupt committed business records.
3. External provider failures must be handled safely.
4. Background jobs must support controlled retry and recovery.
5. Critical failures must generate operational alerts.

Acceptance Criteria:

- Defined component failures are tested.
- Committed data remains consistent.
- Recoverable operations can resume safely.
- Failure alerts are generated.

**NFR-AVAIL-003: Graceful Degradation**

- Priority: High
- Actor: Platform System

Description:

The platform shall preserve essential functionality when non-critical services are unavailable.

Business Rules:

1. Failure of a notification provider must not corrupt customer or ITR records.
2. Failure of reporting services must not prevent unrelated core transactions.
3. Temporary external integration failures must be reported clearly.
4. Security controls must never be disabled to maintain availability.
5. Failed asynchronous operations must be recoverable.

Acceptance Criteria:

- Non-critical service failures do not unnecessarily disable core functions.
- Users receive appropriate error messages.
- Failed tasks can be recovered.

**NFR-AVAIL-004: Transaction Integrity and Idempotency**

- Priority: Critical
- Actor: Platform System

Description:

The system shall preserve data consistency and prevent unintended duplicate operations.

Business Rules:

1. Financial operations must use appropriate database transactions.
2. Critical operations must support idempotency where applicable.
3. Retry operations must not create duplicate invoices, payments or other financial entries.
4. Partial failures must be handled through rollback or controlled recovery.
5. Transaction failures must be traceable.

Acceptance Criteria:

- Critical operations preserve consistency.
- Duplicate submissions are handled safely.
- Failed transactions do not leave invalid partial records.

## 7.4 Backup, Disaster Recovery and Business Continuity

### 7.4.1 Overview

The system shall provide backup, restoration and disaster recovery capabilities to protect customer data and maintain business continuity.

Recovery objectives shall be finalized according to the selected cloud architecture, business impact assessment and infrastructure budget.

### 7.4.2 Functional and Non-Functional Requirements

**NFR-DR-001: Automated Backup**

- Priority: Critical
- Actor: Platform Operations Team

Description:

The system shall perform automated backups of critical application data and configuration.

Business Rules:

1. Database backups must be performed according to an approved schedule.
2. Point-in-time recovery must be supported where available.
3. Document storage must have appropriate backup, replication or recovery protection.
4. Backups must be encrypted.
5. Backup access must be restricted.
6. Backup completion and failure must be monitored.
7. Backup retention must follow approved policies.

Acceptance Criteria:

- Automated backups execute successfully.
- Backup failures generate alerts.
- Backup integrity can be verified.
- Backup restoration is tested.

**NFR-DR-002: Recovery Point Objective**

- Priority: Critical
- Actor: Platform Operations Team

Description:

The system shall define and meet an approved Recovery Point Objective (RPO).

Initial planning target:

- Critical database records: RPO of 15 minutes or less, subject to the selected database backup and replication architecture.
- Document storage: recovery protection and replication objectives must be defined separately.

Business Rules:

1. RPO targets must be documented for critical services.
2. Backup and replication configuration must support approved targets.
3. Actual recovery performance must be measured.
4. Any deviation must be documented and reviewed.

Acceptance Criteria:

- RPO targets are documented.
- Recovery tests measure actual data loss windows.
- Deviations are recorded.

**NFR-DR-003: Recovery Time Objective**

- Priority: Critical
- Actor: Platform Operations Team

Description:

The system shall define and meet an approved Recovery Time Objective (RTO).

Initial planning target:

- Critical platform services: RTO of 4 hours or less, subject to infrastructure design and recovery testing.

Business Rules:

1. Recovery procedures must identify service dependencies.
2. Recovery roles and responsibilities must be documented.
3. Recovery procedures must include database, storage, application and identity services.
4. Recovery performance must be measured.
5. RTO targets must be reviewed periodically.

Acceptance Criteria:

- RTO targets are documented.
- Recovery exercises measure actual restoration time.
- Recovery failures and delays are documented.

**NFR-DR-004: Backup Restoration Testing**

- Priority: Critical
- Actor: Platform Operations and QA Team

Description:

The system shall undergo periodic restoration testing to verify backup integrity and recovery procedures.

Business Rules:

1. Restoration tests must be performed in an isolated environment.
2. Restored data must be checked for integrity and consistency.
3. Tenant isolation must be verified after restoration.
4. Restoration tests must not overwrite production data.
5. Test results and corrective actions must be recorded.

Acceptance Criteria:

- Restoration tests are performed periodically.
- Restored records are validated.
- Tenant isolation remains intact.
- Test evidence is retained.

**NFR-DR-005: Disaster Recovery Plan**

- Priority: Critical
- Actor: Platform Operations Team

Description:

The system shall maintain a documented disaster recovery plan for major infrastructure or service failures.

Business Rules:

1. The plan must identify critical services and dependencies.
2. Recovery responsibilities must be assigned.
3. Alternative infrastructure or recovery procedures must be documented.
4. Recovery communication procedures must be defined.
5. Disaster recovery exercises must be conducted periodically.
6. The plan must be reviewed after major infrastructure changes or incidents.

Acceptance Criteria:

- A documented disaster recovery plan exists.
- Recovery responsibilities are defined.
- Disaster recovery exercises are conducted.
- Corrective actions are tracked.

**NFR-DR-006: Business Continuity**

- Priority: High
- Actor: Platform Operations Team and Platform Super Admin

Description:

The system shall support continued or restored business operations during major service disruptions.

Business Rules:

1. Critical customer and firm services must be prioritized for restoration.
2. Communication procedures must exist for affected firms.
3. Business continuity procedures must address service-provider failures.
4. Recovery must preserve financial and tax-record integrity.
5. Business continuity procedures must be reviewed periodically.

Acceptance Criteria:

- Business continuity procedures are documented.
- Critical services are prioritized.
- Communication and recovery responsibilities are defined.

---

## 7.5 Usability and User Experience

### 7.5.1 Overview

The system shall provide a consistent, intuitive and efficient user experience for Platform Super Admins, Firm Admins, Firm Staff and Customers.

The interface shall be responsive and suitable for professional desktop use as well as mobile and tablet access.

### 7.5.2 Non-Functional Requirements

**NFR-UX-001: Consistent User Interface**

- Priority: High
- Actor: All Users

Description:

The system shall provide consistent navigation, layout, controls and interaction patterns across application modules.

Business Rules:

1. Common actions must follow consistent interaction patterns.
2. Navigation must reflect the user's role and permissions.
3. Forms must provide clear labels and validation feedback.
4. Destructive operations must require appropriate confirmation.
5. Error and success messages must be understandable.
6. Interface terminology must remain consistent.

Acceptance Criteria:

- Common interface components follow a documented design system.
- Users see only permitted navigation options.
- Forms provide clear feedback.
- Destructive actions are appropriately protected.

**NFR-UX-002: Responsive Design**

- Priority: Critical
- Actor: All Users

Description:

The system shall support responsive layouts across desktop, tablet and mobile devices.

Business Rules:

1. The initial product shall be a responsive web application.
2. Core customer portal functions must be usable on mobile devices.
3. Staff and administrator workflows must adapt to supported screen sizes.
4. Critical content must not require horizontal scrolling unnecessarily.
5. Responsive behavior must be tested across representative devices and screen sizes.

Acceptance Criteria:

- Supported screens render correctly.
- Core workflows remain usable on mobile.
- Content and controls are not unintentionally clipped.

**NFR-UX-003: Accessibility**

- Priority: Critical
- Actor: All Users

Description:

The system shall target WCAG 2.2 Level AA accessibility compliance.

Business Rules:

1. Keyboard navigation must be supported.
2. Form controls must have accessible labels.
3. Colour must not be the only means of communicating information.
4. Text and interactive controls must meet applicable contrast requirements.
5. Focus indicators must be visible.
6. Screen-reader compatibility must be considered.
7. Error messages must be accessible.
8. Accessibility must be tested during development and release validation.

Acceptance Criteria:

- Accessibility testing covers major workflows.
- Keyboard-only operation is supported.
- Critical accessibility defects are resolved before production release.

**NFR-UX-004: User Guidance and Help**

- Priority: Medium
- Actor: All Users

Description:

The system shall provide appropriate guidance to help users complete common tasks.

Business Rules:

1. Important workflows must provide contextual instructions where useful.
2. Validation errors must explain how to correct the issue.
3. Help content must be role-appropriate.
4. Knowledge-base content must be maintained.
5. User guidance must not expose sensitive internal system information.

Acceptance Criteria:

- Major workflows provide understandable guidance.
- Users can access permitted help resources.
- Error messages support corrective action.

**NFR-UX-005: Language and Localization Readiness**

- Priority: Medium
- Actor: Platform System

Description:

The system shall support localization-ready architecture.

Business Rules:

1. English shall be the initial launch language.
2. User-facing text must be structured for future translation.
3. Date, time, number and currency formatting must support localization.
4. Tax and financial calculations must not depend on display formatting.
5. Translation changes must not alter underlying stored values.

Acceptance Criteria:

- Initial interface is available in English.
- User-facing text is localization-ready.
- Formatting can be adapted without changing stored data.

## 7.6 Browser and Client Compatibility

### 7.6.1 Overview

The system shall support commonly used modern browsers and devices.

### 7.6.2 Non-Functional Requirements

**NFR-COMP-001: Browser Compatibility**

- Priority: High
- Actor: All Users

Description:

The system shall support current stable versions of major browsers.

Supported browsers shall include:

1. Google Chrome.
2. Microsoft Edge.
3. Mozilla Firefox.
4. Apple Safari.

Business Rules:

1. Browser compatibility must be tested periodically.
2. Unsupported or outdated browsers may receive a clear compatibility warning.
3. Security-sensitive functions must behave consistently across supported browsers.
4. Browser-specific defects must be documented and prioritized.

Acceptance Criteria:

- Core workflows function in supported browsers.
- Authentication and document access work consistently.
- Major browser compatibility defects are addressed.

**NFR-COMP-002: Device Compatibility**

- Priority: High
- Actor: All Users

Description:

The system shall support commonly used desktop, laptop, tablet and mobile devices.

Business Rules:

1. The system shall use responsive web technology.
2. Supported device classes must be documented.
3. Upload and download workflows must be tested on supported devices.
4. Touch interactions must be usable on mobile devices.

Acceptance Criteria:

- Core workflows function on supported device classes.
- Mobile interactions are usable.
- Device-specific defects are documented.

**NFR-COMP-003: File Format Compatibility**

- Priority: High
- Actor: Platform System

Description:

The system shall support approved file formats required for tax-document handling and data exchange.

Business Rules:

1. Supported formats must be explicitly documented.
2. Common document formats such as PDF and approved image formats must be supported.
3. Excel and CSV formats must be supported for data import and export.
4. Unsupported formats must be rejected safely.
5. File compatibility must not weaken malware scanning or validation.

Acceptance Criteria:

- Supported formats can be processed.
- Unsupported files are rejected.
- File processing preserves integrity.

# 8. Quality Assurance and Testing Requirements

## 8.1 Testing Strategy

### 8.1.1 Overview

The system shall follow a structured testing strategy to verify functional correctness, security, performance, usability, tenant isolation and operational reliability.

Testing shall be integrated throughout the software development lifecycle.

### 8.1.2 Testing Requirements

**NFR-TEST-001: Comprehensive Test Strategy**

- Priority: Critical
- Actor: Development and QA Team

Description:

The project shall maintain a documented test strategy covering all major application modules and quality attributes.

Required testing categories shall include:

1. Unit testing.
2. Integration testing.
3. API testing.
4. End-to-end testing.
5. Database and migration testing.
6. Tenant isolation testing.
7. Security testing.
8. Performance and load testing.
9. Accessibility testing.
10. Backup and recovery testing.
11. Regression testing.
12. User acceptance testing.

Acceptance Criteria:

- A documented test strategy exists.
- Test coverage is maintained for critical functionality.
- Test results are recorded.
- Critical defects block release.

**NFR-TEST-002: Unit Testing**

- Priority: Critical
- Actor: Development Team

Description:

Individual business modules, services, utilities and validation rules shall be covered by automated unit tests.

Business Rules:

1. Critical business logic must have appropriate unit tests.
2. Financial calculations must have precision and rounding tests.
3. Permission and workflow logic must be tested.
4. Boundary conditions and error conditions must be covered.
5. Tests must be repeatable.

Acceptance Criteria:

- Automated unit tests execute in CI.
- Critical business logic is covered.
- Failed tests prevent successful validation.

**NFR-TEST-003: Integration Testing**

- Priority: Critical
- Actor: Development and QA Team

Description:

The system shall undergo integration testing to verify interactions between application modules and external services.

Testing shall include:

- Backend and database integration.
- Authentication provider integration.
- Object storage integration.
- Notification provider integration.
- Redis and background job processing.
- Financial transaction workflows.
- Import and export workflows.

Acceptance Criteria:

- Critical integrations are tested.
- Failure scenarios are included.
- Integration test results are recorded.

**NFR-TEST-004: End-to-End Testing**

- Priority: Critical
- Actor: QA Team

Description:

The system shall provide automated end-to-end tests covering complete business workflows.

Representative workflows shall include:

1. Firm registration and approval.
2. Firm Admin invitation and activation.
3. Staff invitation and permission assignment.
4. Customer invitation and onboarding.
5. Customer relationship with multiple firms.
6. Assessment-year creation and ITR processing.
7. Document upload and review.
8. Customer messaging.
9. Invoice creation and payment recording.
10. Excel import and reconciliation.
11. Subscription activation and renewal.
12. Customer portal access and data export.

Acceptance Criteria:

- Critical user journeys are automated.
- Role-specific workflows are validated.
- Cross-tenant access attempts are included.
- End-to-end test failures block production release.

**NFR-TEST-005: Database and Migration Testing**

- Priority: Critical
- Actor: Development and QA Team

Description:

Database migrations and schema changes shall undergo controlled testing.

Business Rules:

1. Migrations must be version-controlled.
2. Migration scripts must be tested before production execution.
3. Existing data must be preserved unless a documented transformation is approved.
4. Rollback or forward-recovery procedures must be documented.
5. RLS policies and database constraints must be tested after migrations.
6. Production migration execution must be controlled.

Acceptance Criteria:

- Migrations are tested in non-production environments.
- Data integrity is verified.
- Tenant isolation is revalidated.
- Migration results are recorded.

## 8.2 Security and Tenant Isolation Testing

**NFR-TEST-006: Security Testing**

- Priority: Critical
- Actor: QA and Security Team

Description:

The system shall undergo regular security testing against relevant application and infrastructure threats.

Testing shall include:

1. Authentication bypass attempts.
2. Authorization and privilege escalation.
3. Cross-tenant access attempts.
4. SQL injection and input manipulation.
5. Cross-site scripting.
6. Cross-site request forgery where applicable.
7. File upload vulnerabilities.
8. Session and token security.
9. API abuse and rate-limit enforcement.
10. Secrets and sensitive data exposure.
11. Dependency and container vulnerabilities.
12. Infrastructure configuration weaknesses.

Acceptance Criteria:

- Security tests are performed before major production releases.
- Critical and high-severity vulnerabilities are resolved or formally risk-accepted under approved policy.
- Test evidence is retained.

**NFR-TEST-007: Dedicated Tenant Isolation Testing**

- Priority: Critical
- Actor: QA and Security Team

Description:

The system shall maintain dedicated automated tests for tenant isolation.

Business Rules:

1. Tests must use multiple independent test tenants.
2. Attempts to access another tenant's records must be rejected.
3. Tests must cover database, API, cache, search, files, reports and background jobs.
4. Connection pooling and transaction context leakage must be tested.
5. Tenant isolation defects must be treated as release-blocking security defects.

Acceptance Criteria:

- Cross-tenant tests execute automatically.
- Unauthorized access is consistently rejected.
- Isolation tests pass before production release.

## 8.3 Performance, Accessibility and Recovery Testing

**NFR-TEST-008: Performance and Load Testing**

- Priority: Critical
- Actor: QA and Operations Team

Description:

The system shall undergo load and stress testing to verify performance and scalability.

Business Rules:

1. Tests must represent expected user behavior.
2. Tests must include concurrent tenant activity.
3. Database and background job performance must be monitored.
4. Peak and sustained loads must be evaluated.
5. Performance regressions must be investigated.

Acceptance Criteria:

- Performance tests are conducted before major releases.
- Defined response-time and capacity targets are evaluated.
- Bottlenecks are documented.

**NFR-TEST-009: Accessibility Testing**

- Priority: High
- Actor: QA Team

Description:

The system shall undergo automated and manual accessibility testing.

Business Rules:

1. Testing must cover major user workflows.
2. Keyboard navigation must be checked.
3. Screen-reader behavior must be evaluated.
4. Colour contrast and form accessibility must be tested.
5. Critical accessibility issues must block release.

Acceptance Criteria:

- Accessibility testing results are documented.
- Critical accessibility defects are resolved.
- WCAG 2.2 AA target compliance is evaluated.

**NFR-TEST-010: Backup and Recovery Testing**

- Priority: Critical
- Actor: QA and Operations Team

Description:

Backup restoration and disaster recovery procedures shall undergo periodic testing.

Business Rules:

1. Restoration must be tested in an isolated environment.
2. Recovered data must be checked for completeness and consistency.
3. Tenant isolation must be validated after restoration.
4. Recovery objectives must be measured.
5. Corrective actions must be tracked.

Acceptance Criteria:

- Recovery tests are completed periodically.
- Restoration results are documented.
- Recovery objective deviations are reviewed.

## 8.4 Regression Testing and Defect Management

**NFR-TEST-011: Regression Testing**

- Priority: Critical
- Actor: QA Team

Description:

The system shall maintain regression tests to ensure that new changes do not break previously working functionality.

Business Rules:

1. Critical workflows must be included in regression suites.
2. Security-sensitive workflows must be included.
3. Financial and tax-record operations must be included.
4. Regression tests must run before production releases.
5. Failed critical regression tests must block release.

Acceptance Criteria:

- Regression suites are maintained.
- Critical tests execute before release.
- Failures are investigated and resolved.

**NFR-TEST-012: Defect Tracking and Resolution**

- Priority: High
- Actor: Development and QA Team

Description:

The project shall maintain a structured defect-management process.

Business Rules:

1. Defects must have severity and priority classifications.
2. Security and tenant isolation defects must receive special treatment.
3. Defect status and resolution history must be maintained.
4. Critical defects must block release.
5. Accepted risks must be documented and approved.

Acceptance Criteria:

- Defects can be tracked.
- Severity and priority are recorded.
- Critical unresolved defects block release.

### 8.5 AI-Assisted Feature Testing

- **NFR-TEST-013 — AI Extraction Accuracy:** **NFR-TEST-013 — AI Extraction Accuracy** AI document extraction shall be tested against representative documents and verified reference data. During the pilot phase, accuracy shall be measured separately for each supported document type and relevant extracted field. Minimum acceptable accuracy thresholds, test sample sizes and validation methods shall be established and formally approved before production release. No unvalidated accuracy threshold shall be treated as an approved production acceptance target.
- **NFR-TEST-014 — Human Verification:** Tests shall verify that extracted information cannot become verified business data without the required human confirmation.
- **NFR-TEST-015 — AI Output Safety:** Tests shall verify that AI-generated drafts and suggestions are clearly identified, editable, and unable to trigger unauthorized consequential actions.
- **NFR-TEST-016 — Prompt Injection Resistance:** Tests shall include malicious instructions embedded in uploaded documents and other AI inputs to verify that security boundaries and system instructions are not bypassed.
- **NFR-TEST-017 — Tenant Isolation:** Tests shall verify that AI requests, results, temporary processing data, and provider interactions cannot expose information across firms, customers, or unauthorized roles.
- **NFR-TEST-018 — Privacy and Data Retention:** Tests shall verify approved data minimization, masking where applicable, provider data-use restrictions, and deletion or retention requirements.
- **NFR-TEST-019 — Provider Failure:** Tests shall cover AI provider timeouts, errors, unavailable services, incomplete responses, and safe manual fallback.
- **NFR-TEST-020 — AI Regression:** Changes to AI providers, models, prompts, or processing logic shall undergo regression testing before production deployment.

# 9. Release Management and Acceptance Governance

## 9.1 Release Governance

### 9.1.1 Overview

The project shall use formal release governance to ensure that software releases satisfy functional, security, performance, accessibility, migration and operational requirements.

A successful software build alone shall not constitute production release approval.

### 9.1.2 Release Requirements

**NFR-REL-001: Release Readiness Criteria**

- Priority: Critical
- Actor: Development, QA and Operations Team

Description:

Each production release shall satisfy documented release-readiness criteria.

Business Rules:

1. Required functional tests must pass.
2. Critical security tests must pass.
3. Tenant isolation tests must pass.
4. Performance requirements must be evaluated.
5. Accessibility requirements must be evaluated.
6. Database migration validation must be completed.
7. Backup and recovery readiness must be confirmed.
8. Operational monitoring must be available.
9. Release documentation must be prepared.
10. Outstanding risks must be documented and approved by authorized stakeholders.

Acceptance Criteria:

- A release-readiness checklist exists.
- Required test evidence is available.
- Unresolved release blockers prevent approval.
- Release decisions are recorded.

**NFR-REL-002: Requirements-to-Test Traceability**

- Priority: Critical
- Actor: Project Management and QA Team

Description:

The project shall maintain traceability between SRS requirements, implementation components, test cases and release evidence.

Business Rules:

1. Functional requirements must have corresponding verification methods.
2. Critical non-functional requirements must have measurable verification criteria.
3. Changes to requirements must trigger an impact review.
4. Missing test coverage must be identifiable.
5. Traceability records must be updated throughout development.

Acceptance Criteria:

- Requirements can be mapped to test cases.
- Critical requirements have verification evidence.
- Requirement changes are traceable.

**NFR-REL-003: Change and Release Approval**

- Priority: Critical
- Actor: Authorized Project and Release Approvers

Description:

Production releases shall follow a controlled approval process.

Business Rules:

1. Releases must have an identified version.
2. Changes must be reviewed and tested.
3. Production deployment must require appropriate authorization.
4. High-risk changes must receive additional review.
5. Approval and deployment history must be recorded.

Acceptance Criteria:

- Releases have documented approval.
- Deployment history is maintained.
- Unauthorized production deployments are prevented.

**NFR-REL-004: Database Migration Verification**

- Priority: Critical
- Actor: Development and Operations Team

Description:

Every production database migration shall undergo verification before and after execution.

Business Rules:

1. Migration scripts must be reviewed.
2. Backup readiness must be confirmed before high-risk migrations.
3. Data integrity must be checked after migration.
4. RLS policies and tenant constraints must be verified.
5. Migration failures must trigger documented recovery procedures.
6. Production migration outcomes must be recorded.

Acceptance Criteria:

- Migration review is completed.
- Pre-migration safeguards are verified.
- Post-migration integrity checks pass.
- Failures are documented and handled.

**NFR-REL-005: Rollback and Recovery Readiness**

- Priority: Critical
- Actor: Development and Operations Team

Description:

The release process shall provide a documented rollback or recovery strategy.

Business Rules:

1. Application rollback procedures must be defined.
2. Database rollback must account for data compatibility and migration reversibility.
3. Destructive migrations must have additional safeguards.
4. Rollback readiness must be reviewed before deployment.
5. Rollback decisions and execution must be recorded.

Acceptance Criteria:

- Rollback procedures are documented.
- High-risk releases have a recovery plan.
- Rollback or forward-recovery procedures are tested where applicable.

**NFR-REL-006: Post-Release Monitoring**

- Priority: Critical
- Actor: Operations Team

Description:

Every production release shall undergo post-deployment monitoring.

Business Rules:

1. Application health must be checked after deployment.
2. Error rates and response times must be monitored.
3. Database and background job health must be checked.
4. Security alerts must be reviewed.
5. Customer-impacting issues must follow incident procedures.
6. Release monitoring outcomes must be recorded.

Acceptance Criteria:

- Post-release checks are completed.
- Critical service failures generate alerts.
- Release health is documented.
- Rollback or remediation is initiated when required.

**NFR-REL-007: Formal Release Sign-Off**

- Priority: Critical
- Actor: Authorized Business, QA, Security and Operations Approvers

Description:

Production release approval shall require documented acceptance from the designated responsible stakeholders.

Business Rules:

1. Functional acceptance must be recorded.
2. QA acceptance must be recorded.
3. Security acceptance must be recorded.
4. Operations readiness must be confirmed.
5. Known limitations and accepted risks must be documented.
6. Approval records must identify the approver and release version.

Acceptance Criteria:

- Required sign-offs are recorded.
- Release evidence is available.
- Production deployment cannot proceed without required approvals.

---

# 10. Business Rules and Data Governance

## 10.1 Core Business Rules

**BR-001: Tenant Data Ownership**

Every CA firm shall own and control its firm-specific business records. Firm-specific data must remain isolated from other firms.

**BR-002: Customer Identity and Firm Relationships**

A customer may be associated with multiple CA firms through separate firm-specific relationships. Each relationship shall maintain its own permissions, assignments, ITR records, documents, invoices and communication history.

**BR-003: PAN Uniqueness**

PAN uniqueness shall be enforced within the appropriate firm-specific customer relationship and applicable business rules. PAN shall not be treated as a global customer login credential.

**BR-004: Assessment Year Management**

Assessment years shall be configurable. The system shall not require code changes to introduce a new assessment year.

**BR-005: ITR Workflow Control**

ITR status transitions shall be controlled by role permissions and defined workflow rules. Ready for Filing, Filed and Completed shall remain distinct statuses.

**BR-006: Manual Filing Representation**

The platform shall record filing information supplied by authorized users. It shall not represent an ITR as government-filed merely because an internal workflow reaches a preparation or review stage.

**BR-007: Financial Record Integrity**

Posted financial transactions shall not be silently overwritten or deleted. Corrections shall use documented reversals, adjustments, refunds or credit notes, as applicable.

**BR-008: Document Version Integrity**

Replacing a document shall preserve the previous version and its relevant history. Unauthorized users shall not be able to access prior versions.

**BR-009: Subscription Separation**

A CA firm's SaaS subscription shall remain separate from customer invoices and tax-service payments.

**BR-010: Subscription Limit Protection**

Reaching a subscription limit shall not cause existing customer records, ITR records, documents, invoices or historical information to be deleted or become inaccessible.

**BR-011: Audit Accountability**

Sensitive administrative, financial, security and tax-workflow operations shall generate appropriate audit events.

**BR-012: Authorized Data Access**

Users shall access only records and operations permitted by their active role, permissions, firm relationship and applicable security policies.

## 10.2 Data Governance Rules

**BR-013: Data Accuracy**

The system shall preserve data accuracy through input validation, database constraints, controlled updates and reconciliation procedures.

**BR-014: Sensitive Data Protection**

Sensitive information, including PAN and financial records, shall receive protection appropriate to its classification.

**BR-015: Data Export**

Authorized firms and users shall be able to export permitted business data in documented formats, subject to security and privacy controls.

**BR-016: Retention and Deletion**

Data retention and deletion shall follow configured retention policies, applicable legal requirements and documented legal holds.

**BR-017: Data Ownership and Portability**

Customer and firm data shall remain logically attributable to the appropriate data owner. The platform shall support authorized data portability and controlled transfer procedures.

**BR-018: Time and Financial Precision**

System timestamps shall be stored consistently using UTC. Financial values shall use appropriate precise decimal representations and documented rounding rules.

# 11. External Systems and Integration Requirements

## 11.1 Integration Architecture

The system shall use modular integration interfaces so that external service providers can be replaced without requiring major changes to core business modules.

External integrations shall be authenticated, monitored, logged and protected against unauthorized access.

## 11.2 Identity and Authentication Integration

**INT-001: Identity Provider Integration**

- Priority: Critical

Description:

The system shall support integration with a secure identity provider using appropriate standards such as OpenID Connect and OAuth 2.0.

Requirements:

1. Staff and administrator authentication shall support mandatory MFA.
2. Customer authentication shall support password-based and verified OTP-based access.
3. Identity provider configuration shall be environment-specific.
4. Authentication failures shall be handled securely.
5. Session and identity events shall be auditable.

## 11.3 Email Integration

**INT-002: Email Service Integration**

- Priority: High

Description:

The system shall integrate with an external email service for transactional and notification messages.

Requirements:

1. Support invitation emails.
2. Support OTP and verification messages.
3. Support customer notifications.
4. Support invoice and payment notifications.
5. Support password and account security notifications.
6. Track delivery status and provider errors where available.
7. Protect email content from unnecessary exposure of sensitive information.

## 11.4 SMS and WhatsApp Integration

**INT-003: SMS Service Integration**

- Priority: High

Description:

The system shall support an external SMS provider for OTP and authorized notifications.

Requirements:

1. OTP messages shall have expiry and attempt limits.
2. Delivery failures shall be recorded.
3. Provider credentials shall be securely stored.
4. Provider changes shall not require changes to core business logic.

**INT-004: WhatsApp Business Integration**

- Priority: Medium

Description:

The system shall support an approved WhatsApp Business integration for permitted customer communications.

Requirements:

1. Integration shall use an authorized provider and applicable platform rules.
2. Customer consent and communication preferences shall be respected.
3. Message templates shall follow provider requirements.
4. Delivery and failure events shall be recorded where available.
5. Sensitive tax documents shall not be exposed through unsecured message content.

## 11.5 File and Object Storage Integration

**INT-005: Private Object Storage**

- Priority: Critical

Description:

The system shall integrate with private cloud object storage for customer documents and generated files.

Requirements:

1. Public access shall be disabled by default.
2. Authorized access shall use short-lived access mechanisms.
3. File metadata shall remain associated with the appropriate tenant and business record.
4. Upload and download operations shall be authorized.
5. Malware scanning and quarantine workflows shall be supported.
6. Storage provider configuration shall be replaceable through an abstraction layer.

## 11.6 Background Processing and Caching

**INT-006: Background Job Processing**

- Priority: High

Description:

The system shall use a background processing mechanism for long-running and asynchronous tasks.

Applicable tasks include:

- Bulk imports.
- Report generation.
- Notification delivery.
- File scanning.
- Scheduled reminders.
- Data exports.
- Other approved asynchronous operations.

Requirements:

1. Jobs shall have traceable status.
2. Failed jobs shall support controlled retry.
3. Duplicate execution shall not create duplicate financial or business transactions.
4. Tenant context shall be preserved and verified.
5. Job failures shall be logged and monitored.

**INT-007: Caching and Distributed State**

- Priority: High

Description:

The system may use Redis or an equivalent service for caching, queues and distributed coordination.

Requirements:

1. Cached data shall respect tenant boundaries.
2. Sensitive data shall not be cached unnecessarily.
3. Cache invalidation shall be defined for critical business records.
4. Authentication and authorization shall not depend solely on stale cached permissions.
5. Distributed state shall be protected against unauthorized access.

## 11.7 Payment and Financial Integrations

**INT-008: Payment Provider Readiness**

- Priority: Medium

Description:

The system shall be designed to support future integration with payment gateways.

Initial release requirements:

1. Customer payment recording shall support manual entry.
2. Payment receipts shall be maintained.
3. Partial payments, refunds and adjustments shall be supported.
4. SaaS subscription payment processing shall remain separate from customer billing.
5. Payment gateway integration may be introduced in a later phase.

## 11.8 API and Webhook Integration

**INT-009: External API and Webhook Support**

- Priority: Medium

Description:

The system shall support controlled external integration through versioned APIs and secure webhooks where required.

Requirements:

1. APIs shall use documented authentication and authorization.
2. API versions shall be managed.
3. Rate limiting shall be applied.
4. Webhook requests shall be authenticated and verified.
5. Retry and duplicate-event handling shall be supported.
6. Integration events shall be logged.
7. Sensitive information shall not be unnecessarily exposed.

## 11.9 Government and Tax Platform Integration Boundary

**INT-010: Government Portal Integration Boundary**

- Priority: High

Description:

The initial release shall not depend on direct government tax portal integration.

Requirements:

1. Actual ITR filing shall remain outside the platform.
2. Government acknowledgements and filing details may be recorded by authorized users.
3. Automated filing, government refund APIs and automatic AIS/26AS retrieval are excluded from the initial release.
4. Future integrations shall require separate legal, security and technical assessment.

# 12. Assumptions, Dependencies and Constraints

## 12.1 Project Assumptions

**ASM-001: Internet Connectivity**

The system assumes that users have suitable internet connectivity to access the cloud-hosted application.

**ASM-002: Supported Devices**

Users are expected to use supported modern browsers on desktop, laptop, tablet or mobile devices.

**ASM-003: CA Firm Responsibilities**

Each CA firm is responsible for maintaining accurate customer information, assigning appropriate staff permissions and reviewing tax-related work.

**ASM-004: Customer Contact Information**

Customer invitations and OTP authentication depend on valid, verified email addresses or mobile numbers.

**ASM-005: Tax Professional Review**

Assisted tax calculations and validations are support features. They do not replace professional review and approval by authorized CA personnel.

**ASM-006: External Provider Availability**

Email, SMS, WhatsApp, identity and storage integrations depend on the availability and service terms of their respective providers.

**ASM-007: Historical Data Quality**

Imported historical data may contain duplicates, missing values, inconsistent formats or other quality issues. The import process shall identify and report such issues rather than silently accepting them.

## 12.2 External Dependencies

**DEP-001: Cloud Infrastructure**

Production operation depends on the selected cloud infrastructure provider and its managed services.

**DEP-002: Identity Provider**

Authentication and MFA functionality depend on the configured identity provider.

**DEP-003: Notification Providers**

Email, SMS and WhatsApp notifications depend on configured external providers.

**DEP-004: Object Storage**

Document storage, retrieval and retention depend on the selected object storage service.

**DEP-005: Domain and Certificates**

Production access depends on valid domain configuration, DNS and TLS certificates.

**DEP-006: Monitoring and Backup Services**

Operational monitoring, backup and recovery depend on configured infrastructure and supporting services.

## 12.3 Technical Constraints

**CON-001: Database Technology**

PostgreSQL shall be the primary relational database.

**CON-002: Application Architecture**

The initial backend shall follow a modular monolith architecture rather than a distributed microservices architecture.

**CON-003: Frontend Technology**

The frontend shall use Next.js, React and TypeScript.

**CON-004: Backend Technology**

The backend shall use NestJS and TypeScript.

**CON-005: Database Access**

The system shall use Prisma with controlled SQL where necessary, particularly for PostgreSQL Row-Level Security and advanced database constraints.

**CON-006: Deployment**

The application shall use container-compatible deployment practices and maintain separate development, staging, pre-production and production environments.

**CON-007: Cloud Portability**

The architecture shall avoid unnecessary dependence on a single cloud provider where practical.

**CON-008: Initial Language**

English shall be the initial product language, with localization readiness.

## 12.4 Business and Scope Constraints

**CON-009: Initial Product Scope**

The first release shall focus on CA firm operations, customer management, ITR workflow management, document management, communication, billing and reporting.

**CON-010: Manual ITR Filing**

The platform shall not perform automatic government portal filing in the initial release.

**CON-011: Manual Refund Updates**

Refund tracking shall initially depend on information entered or updated by authorized users.

**CON-012: No Native Mobile Application**

The initial release shall provide a responsive web application rather than separate native Android and iOS applications.

**CON-013: Subscription Activation**

SaaS subscription activation and payment may initially be managed manually by Platform Super Admins.

**CON-014: Tax Calculation Scope**

Initial assisted tax calculations shall remain limited and subject to professional review. A comprehensive tax computation engine is outside the initial release scope.

---

# 13. Project Risks and Mitigation

## 13.1 Risk Management Approach

The project shall maintain a risk register to identify, assess, monitor and mitigate risks that may affect system security, reliability, delivery, compliance or business operations.

Risks shall be reviewed periodically and whenever significant architectural, operational or business changes occur.

Risk levels shall be classified as:

- **Critical:** May cause severe security, financial, legal or operational impact.
- **High:** May significantly affect system functionality, delivery or business operations.
- **Medium:** May cause manageable operational or delivery impact.
- **Low:** Limited impact that can be managed through routine controls.

## 13.2 Risk Register

**RISK-001: Cross-Tenant Data Exposure**

- Risk Level: Critical

Description:

Incorrect tenant authorization, database configuration or application logic could expose one CA firm's information to another firm.

Mitigation:

1. Implement PostgreSQL Row-Level Security.
2. Use restricted database runtime roles.
3. Apply tenant-aware authorization throughout the application.
4. Use tenant-aware database constraints and relationships.
5. Conduct automated cross-tenant security testing.
6. Monitor suspicious cross-tenant access attempts.

**RISK-002: Unauthorized Account Access**

- Risk Level: Critical

Description:

Compromised credentials, weak authentication or session vulnerabilities could allow unauthorized access.

Mitigation:

1. Use secure authentication and MFA.
2. Apply rate limiting and account protection.
3. Monitor suspicious login activity.
4. Implement secure session and token management.
5. Conduct regular authentication security testing.

**RISK-003: Loss or Corruption of Customer Data**

- Risk Level: Critical

Description:

Infrastructure failures, software defects or operational mistakes could cause data loss or corruption.

Mitigation:

1. Maintain encrypted backups.
2. Implement point-in-time recovery.
3. Test restoration procedures.
4. Maintain database integrity constraints.
5. Monitor database health and replication where applicable.

**RISK-004: Sensitive Document Exposure**

- Risk Level: Critical

Description:

Unauthorized access, insecure links or malicious file uploads could expose customer tax documents.

Mitigation:

1. Use private object storage.
2. Apply authorization checks before file access.
3. Use short-lived access links.
4. Scan uploaded files for malware.
5. Maintain document access audit logs.
6. Apply secure file validation and quarantine procedures.

**RISK-005: Incorrect Tax or Financial Calculations**

- Risk Level: High

Description:

Incorrect formulas, rounding or business rules could produce inaccurate tax or financial information.

Mitigation:

1. Use precise decimal arithmetic.
2. Maintain documented calculation rules.
3. Create automated calculation test cases.
4. Include boundary and rounding tests.
5. Require authorized professional review where applicable.
6. Maintain calculation and adjustment history.

**RISK-006: Poor Historical Data Quality**

- Risk Level: High

Description:

Existing Excel files and historical records may contain duplicate, incomplete or inconsistent information.

Mitigation:

1. Provide import previews.
2. Validate records before import.
3. Detect duplicates.
4. Generate detailed error reports.
5. Support controlled rollback and correction.
6. Reconcile imported totals and record counts.

**RISK-007: External Provider Failure**

- Risk Level: Medium

Description:

Email, SMS, WhatsApp, identity or cloud storage provider outages may interrupt dependent functionality.

Mitigation:

1. Use provider abstraction layers.
2. Implement retries and failure handling.
3. Monitor provider availability.
4. Maintain operational procedures for provider outages.
5. Evaluate alternative providers where practical.

**RISK-008: Performance Degradation**

- Risk Level: High

Description:

Increasing customer volumes, concurrent requests, large files or background jobs may degrade application performance.

Mitigation:

1. Conduct load testing.
2. Monitor database queries and application response times.
3. Use controlled caching.
4. Optimize database indexes.
5. Use asynchronous processing for long-running operations.
6. Scale infrastructure based on measured demand.

**RISK-009: Unauthorized Privileged Operations**

- Risk Level: Critical

Description:

Misuse or compromise of Platform Super Admin or Firm Admin accounts could cause widespread operational or data-access impact.

Mitigation:

1. Enforce mandatory MFA.
2. Apply least-privilege access.
3. Record privileged operations.
4. Apply additional verification to high-risk actions.
5. Review privileged access periodically.
6. Monitor suspicious administrative activity.

**RISK-010: Regulatory or Legal Changes**

- Risk Level: High

Description:

Changes in applicable tax, privacy, security or other legal requirements may require changes to the platform.

Mitigation:

1. Monitor applicable regulatory changes.
2. Maintain configurable assessment years and business rules.
3. Review legal and compliance requirements periodically.
4. Document affected system components.
5. Plan controlled updates and testing.

**RISK-011: Deployment or Migration Failure**

- Risk Level: High

Description:

A faulty deployment or database migration could interrupt services or affect existing records.

Mitigation:

1. Maintain separate deployment environments.
2. Test migrations before production.
3. Verify backup readiness.
4. Maintain rollback or forward-recovery procedures.
5. Require release approvals.
6. Monitor the system after deployment.

**RISK-012: Inadequate Capacity or Resource Availability**

- Risk Level: Medium

Description:

Insufficient infrastructure resources or development capacity may delay delivery or reduce service quality.

Mitigation:

1. Plan infrastructure capacity based on expected usage.
2. Monitor resource utilization.
3. Prioritize critical requirements.
4. Maintain a phased implementation roadmap.
5. Review delivery estimates as scope and complexity become clearer.

## 13.3 Risk Review and Ownership

1. Each critical and high risk shall have an assigned owner.
2. Risk assessments shall be reviewed periodically.
3. Mitigation actions shall have documented status.
4. Material new risks shall be added to the register.
5. Accepted risks shall require documented approval from an authorized stakeholder.
6. Security and tenant isolation risks shall receive priority attention.

# 14. Overall System Acceptance Criteria

## 14.1 General Acceptance Principles

The system shall be considered ready for production only when the agreed functional, security, performance, usability, operational and business acceptance criteria have been satisfied.

Acceptance shall be based on documented test results, stakeholder review and release approval.

## 14.2 Functional Acceptance

**AC-001: User and Role Management**

The system shall support the approved user roles, account lifecycle, invitations, authentication and role-based permissions.

Verification:

- Platform Super Admin workflows pass.
- Firm Admin workflows pass.
- Staff workflows pass.
- Customer portal workflows pass.
- Unauthorized operations are rejected.

**AC-002: Multi-Tenant Firm Management**

The system shall support multiple independent CA firms.

Verification:

- Firm onboarding and approval work correctly.
- Firm-specific records remain isolated.
- Customers can be associated with multiple firms.
- Firm-specific customer records remain independent.

**AC-003: Customer and ITR Management**

The system shall support customer management, dynamic assessment years and controlled ITR workflows.

Verification:

- Customer onboarding works.
- Assessment years can be configured.
- ITR records can be created and managed.
- Workflow transitions follow defined permissions.
- Filing and completion states are correctly represented.

**AC-004: Document and Checklist Management**

The system shall support secure document uploads, review, versioning and checklists.

Verification:

- Authorized users can upload and access permitted documents.
- Unauthorized access is rejected.
- Document review and replacement history is maintained.
- Required document checklists function correctly.

**AC-005: Communication and Notifications**

The system shall support customer communication and configured notifications.

Verification:

- Authorized messages can be exchanged.
- Notification preferences are respected.
- Delivery failures are recorded where supported.
- Communication history is maintained.

**AC-006: Financial Management**

The system shall support customer invoicing, payment recording, partial payments, refunds and reconciliation.

Verification:

- Financial records are accurate.
- Adjustments preserve transaction history.
- Customer financial records remain separate from SaaS subscription billing.
- Financial reports reconcile with underlying records.

**AC-007: Import and Migration**

The system shall support controlled Excel and CSV import and historical migration.

Verification:

- Import preview and validation work.
- Duplicates and errors are identified.
- Import results are traceable.
- Failed imports can be handled safely.
- Reconciliation results are available.

**AC-008: Reporting and Administration**

The system shall support authorized reporting and platform administration.

Verification:

- Reports respect tenant and role permissions.
- Platform administration functions work.
- Firm branding and configuration work.
- Audit information is available to authorized users.

**AC-009: SaaS Subscription Management**

The system shall support firm subscription plans, activation, usage tracking and configurable limits.

Verification:

- Plans can be configured.
- Subscription status is maintained.
- Usage limits are enforced as specified.
- Existing business data remains accessible when limits are reached.

## 14.3 Security Acceptance

**AC-010: Authentication and Authorization**

Authentication and authorization controls shall pass the required security tests.

Verification:

- Mandatory MFA is enforced for privileged users.
- Unauthorized access is rejected.
- Role and permission boundaries are respected.
- Session security tests pass.

**AC-011: Tenant Isolation**

Tenant isolation shall be verified across application and infrastructure layers.

Verification:

- Cross-tenant API access is rejected.
- Database RLS policies work correctly.
- Cache and search isolation is validated.
- File and background-job isolation is validated.
- No unresolved critical tenant isolation defects remain.

**AC-012: Data and Document Protection**

Sensitive information and customer documents shall be protected.

Verification:

- Encryption controls are verified.
- File access is authorized.
- Audit records are generated for required events.
- Security testing identifies no unresolved release-blocking defects.

## 14.4 Non-Functional Acceptance

**AC-013: Performance and Scalability**

The system shall satisfy the approved performance targets under representative workloads.

Verification:

- Common read and write response-time targets are evaluated.
- Expected concurrent usage is load-tested.
- Performance bottlenecks are documented.
- Capacity limits and scaling procedures are reviewed.

**AC-014: Availability and Recovery**

The system shall satisfy approved availability and recovery objectives.

Verification:

- Availability monitoring is active.
- Backup restoration is tested.
- Recovery procedures are documented.
- RPO and RTO results are measured.

**AC-015: Usability and Accessibility**

The system shall meet the approved usability and accessibility requirements.

Verification:

- Responsive layouts are tested.
- Supported browser workflows pass.
- Keyboard navigation is tested.
- Accessibility evaluation is documented against the WCAG 2.2 AA target.

**AC-016: Testing and Regression**

The system shall pass the required automated and manual testing.

Verification:

- Critical unit and integration tests pass.
- Critical end-to-end workflows pass.
- Regression testing passes.
- Security and tenant isolation testing passes.
- Critical unresolved defects are not permitted for production release.

## 14.5 Operational and Release Acceptance

**AC-017: Deployment Readiness**

The system shall be deployable through the approved release process.

Verification:

- Production infrastructure is configured.
- CI/CD controls are verified.
- Environment configuration is secure.
- Monitoring and alerting are operational.
- Deployment and recovery procedures are documented.

**AC-018: Documentation and Handover**

The project shall maintain the documentation required for operation and maintenance.

Required documentation shall include:

1. Software Requirements Specification.
2. System architecture documentation.
3. Database schema and data dictionary.
4. API documentation.
5. Deployment and environment configuration guidance.
6. Backup and recovery procedures.
7. Security and access-control documentation.
8. Test strategy and test evidence.
9. Operational and incident procedures.
10. User and administrator guidance.

**AC-019: Formal Production Approval**

Production release shall require documented acceptance from designated business, QA, security and operations stakeholders.

Verification:

- Release checklist is complete.
- Required test evidence is available.
- Outstanding risks are documented.
- Required approvals are recorded.
- Post-release monitoring is ready.

**AC-020: AI Document Extraction**

AI-assisted document extraction shall be tested against approved reference documents, and extracted values shall require human verification before becoming verified business data.

Verification:

- Extraction results are tested against approved reference documents.
- Accuracy results are recorded against thresholds established through pilot validation and formally approved before production release.
- Extracted values remain unverified until an authorized human reviews and confirms them.
- Incorrect or uncertain extraction results can be corrected or rejected.
- Verification outcomes are recorded for audit purposes.

**AC-021: AI Human Oversight**

AI-generated suggestions and drafts shall remain identifiable and editable. No consequential workflow action shall be completed solely through an AI-generated output.

Verification:

- AI-generated suggestions and drafts are clearly identified as AI-assisted.
- Authorized users can review and edit AI-generated content.
- Human approval is required before consequential workflow actions.
- AI-generated output cannot independently mark an ITR as filed, completed or approved.
- Human review and approval decisions are recorded in the audit trail.

**AC-022: AI Tenant Isolation and Privacy**

AI processing shall pass tenant-isolation, access-control, data-minimization, provider data-use and retention tests.

Verification:

- AI processing cannot access another tenant's documents or business data.
- AI features enforce the requesting user's permissions.
- Only data necessary for the requested AI operation is processed.
- AI providers comply with approved data-use and retention policies.
- AI data handling and access controls are tested and documented.
- Privacy and tenant-isolation test results are retained as release evidence.

**AC-023: AI Security**

AI-assisted features shall pass approved prompt-injection, malicious-document, unauthorized-disclosure and output-safety tests.

Verification:

- Prompt-injection attempts are tested against AI-assisted workflows.
- Malicious or manipulated documents cannot bypass document security controls.
- AI processing does not disclose information to unauthorized users or tenants.
- AI-generated outputs are checked for unsafe or unauthorized instructions.
- Security test findings are documented and critical findings are resolved before release.

**AC-024: AI Auditability**

AI-assisted operations shall be auditable, including the initiating user, operation type, relevant references, review outcome and approval decision.

Verification:

- AI-assisted operations generate audit records.
- Audit records identify the initiating user and relevant tenant.
- Records capture the operation type, timestamp and relevant business-record references.
- Human review, correction and approval outcomes are recorded.
- Audit records are protected against unauthorized modification or deletion.
- Authorized reviewers can retrieve AI-related audit evidence.

**AC-025: AI Reliability**

AI provider failures shall not corrupt records or prevent supported manual workflows. Safe fallback behavior shall be demonstrated.

Verification:

- AI provider timeouts and service failures are tested.
- Failed AI operations do not corrupt or partially overwrite business records.
- Users receive clear feedback when AI services are unavailable.
- Supported manual workflows remain available during AI service outages.
- Failed or interrupted AI jobs can be safely retried without unintended duplicate actions.
- Recovery and fallback test results are documented.

# 15. Glossary and Abbreviations

| Term | Definition |
|---|---|
| CA | Chartered Accountant |
| SaaS | Software as a Service |
| ITR | Income Tax Return |
| PAN | Permanent Account Number |
| AY | Assessment Year |
| MFA | Multi-Factor Authentication |
| OTP | One-Time Password |
| RBAC | Role-Based Access Control |
| RLS | Row-Level Security |
| API | Application Programming Interface |
| REST | Representational State Transfer |
| OIDC | OpenID Connect |
| OAuth 2.0 | Authorization framework for delegated access |
| TLS | Transport Layer Security |
| KMS | Key Management Service |
| PII | Personally Identifiable Information |
| UTC | Coordinated Universal Time |
| RPO | Recovery Point Objective |
| RTO | Recovery Time Objective |
| SLA | Service Level Agreement |
| CI/CD | Continuous Integration and Continuous Delivery/Deployment |
| E2E | End-to-End |
| QA | Quality Assurance |
| UAT | User Acceptance Testing |
| WCAG | Web Content Accessibility Guidelines |
| DR | Disaster Recovery |
| PITR | Point-in-Time Recovery |
| PDF | Portable Document Format |
| CSV | Comma-Separated Values |
| XLSX | Microsoft Excel Open XML Spreadsheet Format |
| KYC | Know Your Customer |
| CRM | Customer Relationship Management |
| WAF | Web Application Firewall |
| SRS | Software Requirements Specification |
| NFR | Non-Functional Requirement |
| FR | Functional Requirement |
| BR | Business Rule |
| INT | Integration Requirement |
| ASM | Assumption |
| DEP | Dependency |
| CON | Constraint |
| RISK | Project Risk |
| AC | Acceptance Criterion |
