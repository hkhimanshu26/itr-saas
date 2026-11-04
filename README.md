# ITR Management SaaS

A secure, multi-tenant Income Tax Return (ITR) management platform designed for Chartered Accountant (CA) firms in India.

## 1. Project Overview

The platform will allow multiple CA firms to manage their customers, assessment years, income tax return records, documents, payments, and communications through a centralized web application.

Each CA firm will operate in an isolated tenant environment, ensuring that its customers and business data cannot be accessed by other firms.

## 2. User Roles

- **Platform Super Admin:** Manages the overall SaaS platform and CA firm accounts.
- **Firm Admin:** Manages a CA firm's users, customers, settings, and operations.
- **Firm Staff:** Handles assigned customers and ITR-related activities.
- **Customer:** Accesses personal ITR records, uploads documents, and tracks progress.

## 3. Core Features

- Multi-tenant CA firm management.
- Role-based access control.
- Customer management.
- Assessment-year-based ITR management.
- Secure document upload and management.
- Excel-based customer data import.
- ITR status and workflow tracking.
- Refund tracking.
- Invoices and payment records.
- Customer and staff communication.
- Audit logs and activity history.
- SaaS subscription architecture.

## 4. Planned Technology Stack

| Component | Technology |
|---|---|
| Frontend | Next.js, React, TypeScript |
| Backend | NestJS, TypeScript |
| Database | PostgreSQL |
| Database access | Prisma with carefully managed SQL |
| UI | Tailwind CSS, shadcn/ui |
| Background jobs | Redis, BullMQ |
| API documentation | OpenAPI |
| Testing | Jest, Playwright |
| Infrastructure | Docker |
| Version control | Git |

Technology choices will be validated during architecture planning.

## 5. Security Principles

- Strict tenant-level data isolation.
- PostgreSQL Row-Level Security (RLS).
- Role-based authorization.
- Secure authentication and session management.
- Multi-factor authentication for administrative users.
- Secure document storage.
- Audit logging.
- Input validation and rate limiting.
- Automated and manual security testing.

## 6. Initial Scope

The first release will focus on customer management, ITR records, document management, staff workflows, and customer access.

Government API integrations, automated ITR filing, automated tax calculations, and native mobile applications are outside the initial release scope.

## 7. Project Structure

```text
itr-saas/
├── apps/
├── docs/
├── infrastructure/
├── scripts/
└── README.md
```

## 8. Development Approach

Development will proceed in controlled phases:

1. Requirements and Software Requirements Specification (SRS).
2. System architecture and database design.
3. Security architecture and threat modelling.
4. Development environment setup.
5. Backend development.
6. Frontend development.
7. Integration and testing.
8. Security testing.
9. Deployment preparation.

Each phase will be reviewed and tested before proceeding to the next.

## 9. Project Status

**Current phase:** Project foundation and requirements planning.

**Development status:** Not yet started.