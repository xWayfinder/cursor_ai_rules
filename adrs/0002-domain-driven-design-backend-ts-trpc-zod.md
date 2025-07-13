# 2. Domain Driven Design Backend Typescript, TRPC, Zod

Date: 2025-07-10

## Status

Accepted

## Context

This ADR describes decisions powering a cursor rule for how to implement Domain-Driven Design (DDD) principles in typescript.

## Why This Architecture Matters

[DDD Architecture Diagram](../images/ddd-diagram.drawio.png)

### The Problem We're Solving

As an application grows in complexity, it will encountered several challenges that traditional monolithic or unstructured approaches struggle to address:

1. **Business Logic Scattered Everywhere**: Core business rules were mixed with infrastructure concerns, making them hard to find, understand, and modify
2. **Tight Coupling**: Changes in one area would unexpectedly break functionality in seemingly unrelated parts of the system
3. **Testing Difficulties**: Business logic was entangled with database operations and external dependencies, making unit testing nearly impossible
4. **Knowledge Silos**: Only certain developers understood specific parts of the system, creating bottlenecks and knowledge gaps
5. **Refactoring Nightmares**: Making changes required understanding the entire system rather than focused domain areas

### Why Domain-Driven Design (DDD)

DDD provides a structured approach that addresses these challenges by:

- **Organizing code around business domains** rather than technical concerns
- **Creating clear boundaries** between different parts of the system
- **Separating business logic from infrastructure** concerns
- **Making the codebase self-documenting** through clear naming and structure
- **Enabling focused testing** of business rules in isolation

### Why We Have a Cursor Rule

The cursor rule for this architecture exists because:

1. **Consistency is Critical**: DDD requires consistent patterns across the entire codebase. Without enforcement, developers might revert to familiar patterns that break the architecture
2. **Learning Curve**: DDD concepts can be complex, and the rule serves as a guide for developers who are still learning the patterns
3. **Quality Assurance**: The rule ensures that new code follows established patterns, reducing technical debt and maintenance burden
4. **Team Alignment**: It helps maintain a shared understanding of how the system should be structured across the entire development team
5. **Future-Proofing**: Consistent architecture makes it easier to onboard new team members and maintain the system long-term

### The Investment Payoff

While this architecture requires more upfront planning and discipline, it pays dividends through:

- **Easier Intervention**: These patterns organize the code in a way that is easy for human readers to understand so when they need to intervene on with LLMs written code it's easy to comprehend the logic.
- **Faster Development**: Once patterns are established, new features can be implemented more quickly
- **Easier Maintenance**: Changes are isolated to specific domains, reducing the risk of breaking other parts of the system
- **Better Testing**: Business logic can be tested independently of infrastructure concerns
- **Improved Collaboration**: Clear boundaries make it easier for multiple developers to work on different parts of the system simultaneously
- **Scalability**: The architecture scales both technically and organizationally as the team and system grow

## Decision

### Folder Structure Overview

```
/src/server/
├── access-controls/
│   ├── auth/                  # Configuration for service like auth0
│   └── permissions/           # Permission logic
├── routers/                   # TRPC request handlers (API endpoints) + zod input validation
├── dto/                       # Output types - zod based dtos
├── controller/                # Request processing and authorization
├── app-services/              # Use case orchestration
├── domains/                   # Domain-specific business logic
│   └── <domain>/
│       ├── schema/            # Domain-specific database schema
│       │   └── tables.prisma  # Drizzle/Prisma table definitions
│       ├── mappers/           # Entity ↔ Schema conversions
│       ├── entities/          # Domain entities and aggregate roots
│       └── repository/        # Data persistence layer
└── database/                  # Database configuration and migrations
    ├── config.ts              # Database connection and settings
    ├── schema.prisma              # Combined schema exports
    └── migrations/            # All database migrations
        └── YYYYMMDDHHMMSS_migration_name.ts
```

---

**Request Handling**

---

### 1. 🌐 Request Handler TRPC

📂 **Folder**: `@server/routers`

🎯 **Purpose** Provide a secure and organized entry point for client-server communication, ensuring that only authenticated users can access protected resources while maintaining a clean separation between external requests and internal business logic.

📋 **Responsibilities**:

- Handle client-server communication by providing API endpoints
- Verify that users are authenticated before processing requests
- Validate input

### 2. Controller

📂 **Folder**: `@server/controller`
📄 File Name: `@server/controller/<action | domain>.controller.ts`

🎯 **Purpose** Bridge the gap between external API requests and internal business logic by transforming incoming data into formats that application and domain services can process, ensuring clean separation of concerns and maintainable code.

📋 **Responsibilities**:

- Check if the user has permission to perform the requested action
- Convert incoming request data into a format that services can understand
- Forward the request to the correct service to handle the business logic
- Convert the service response back into a format that can be sent to the client

📒 **Notes**

- For simple actions in a domain controllers may combine the actions into a single file
- For a simple action spanning only a single domain a controller my call directly to a domain service
- For actions spanning multiple domains the controller will call a application service which will orchestrate amongst multiple domain services

### 3. Schemas DTOs

📂 **Folder**: `@server/schema`

🎯 **Purpose** Define standardized data contracts between the server and client to ensure type safety, validation, and proper data encapsulation. DTOs prevent exposing internal domain models directly to clients, allowing the internal and external representations to evolve independently while maintaining a stable API contract.

📋 **Responsibilities**:

- Define standardized data structures for input/output validation using Zod schemas
- Provide a clear contract for data exchange between client and server that:
  - Ensures only necessary and properly formatted data is transferred
  - Hides internal implementation details
  - Enables independent evolution of internal/external models
  - Enforces type safety and validation at API boundaries
  - Maintains backwards compatibility

### 4. To DTO and From DTO Mappers

🎯 **Purpose** Provide a clean separation between internal domain models and external API contracts through simple data transformation. This separation allows domain models and DTOs to evolve independently while maintaining a stable API interface. The mapping process should be thin and focused solely on data transformation, with the only exception being logic required for backwards compatibility with older API versions.

📋 **Responsibilities**: Convert data between different layers of the application:
<Entity> ToDto Mapper: Transforms domain entities into DTOs for external consumption
File Name: `@server/<domain>/mappers/toDto.mapper.ts`

- Performs straightforward data transformation between domain entities and DTOs
- Handles data type conversions and basic transformations
- Manages backwards compatibility transformations for older API versions
- Flattens or restructures data as needed for the DTO format

FromDto <Entity> Mapper: Transforms external DTOs into domain entities
File Name: `@server/<domain>/mappers/fromDto.mapper.ts`

- Performs straightforward data transformation from DTOs to domain entities
- Handles basic data type conversions
- Reconstructs domain object structures from DTO format
- Manages backwards compatibility transformations from older API versions

> **Note**: Business rules and validations should be handled in the domain layer or input validators, not in mappers. Mappers should remain thin and focused solely on data transformation.

---

**Access Control**

---

### 5. Authentication

📂 **Folder**: `@server/access-controls/auth`

Typical files:
- `auth0.config.ts` - Auth0 configuration and setup
- `auth0.client.ts` - Auth0 client initialization and management
- `auth0.middleware.ts` - Express/TRPC middleware for Auth0 integration
- `auth0.types.ts` - Type definitions for Auth0 user profiles and tokens

🎯 **Purpose** Authentication is critical for protecting sensitive data and operations by verifying user identities before granting access. Without robust authentication, malicious actors could impersonate legitimate users and compromise system security. The authentication layer serves as the first line of defense in maintaining the confidentiality and integrity of the application.

📋 **Responsibilities**:
- Verify user identity by checking provided credentials against stored data
- Create and handle session tokens (like JWTs) to track authenticated users
- Securely hash and store passwords using industry best practices
- Enable users to reset passwords and recover locked accounts
- Support third-party login options like OAuth and Single Sign-On (SSO)
- Track login activity and block repeated failed attempts to prevent attacks
- Support two-factor authentication (2FA) for extra account security
- Protect authentication data in transit using encryption



### 6. Permissions

📂 **Folder**: `@server/access-controls/permissions`

Typical files:
- `<use-case>.permissions.ts` - based around use cases for a role, action, domain or all

🎯 **Purpose** A robust permissions system is critical for protecting sensitive data and operations by ensuring users can only access resources they are authorized to use. Without proper authorization controls, users could access or modify data beyond their intended privileges, potentially compromising data security and privacy. Separating authorization from core business logic allows the application to evolve its permission model independently while maintaining clean, focused domain code.

📋 **Responsibilities**:
- Store all permission rules in one central place for easy management
- Check if users have permission to access specific features or data
- Make it simple for other code to verify permissions without needing to know the details
- Connect with external permission systems when needed
- Support different types of permissions like user roles and custom rules
- Keep track of when permissions are checked and when rules change
- Make sure permission checks stay fast even with many users
- Allow permission rules to be updated without changing other code
- Save commonly used permission results to improve speed

> 💡 Good to know:
> - Permissions are a special type of business logic that is best kept separate from the core business logic. This separation allows for more flexible and maintainable code, as changes to permission rules don't require modifications to the core business logic, and vice versa.
> - Keeping permissions separate makes it easier to migrate to external authorization systems like OPA (Open Policy Agent) or SpiceDB in the future, as the permission logic is already isolated from the core business logic.

---

**Business Logic**

---

### 7. Application Service

📂 **Folder**: `@server/app-services/`

Typical files:
- `@server/app-services/<use-case>.app-service.ts`

🎯 **Purpose** Application services provide a dedicated orchestration layer that enables complex business operations spanning multiple domains to work together while keeping those domains decoupled and focused on their core responsibilities. Without this layer, domains would need to directly depend on each other, leading to tight coupling and making the system harder to maintain and evolve.

📋 **Responsibilities**:

**Process Orchestration**
- Coordinate business operations that span multiple domain services
- Ensure domain services remain decoupled while working together
- Maintain clean separation between orchestration and domain logic

**Data Management**
- Handle transactions across domain boundaries
- Ensure data consistency when multiple domains are involved
- Provide a unified interface for multi-domain operations

**Infrastructure Concerns**
- Manage cross-cutting aspects like logging and error handling
- Handle event publishing between domains
- Maintain clean separation from core domain logic

### 8. Domain Service

🎯 **Purpose** Domain services encapsulate complex business logic that operates across multiple entities or requires coordination between different parts of the domain, providing a home for operations that don't conceptually belong to any single entity. This separation helps maintain clean, focused entities while still enabling sophisticated domain operations that may involve multiple objects or complex calculations.

📋 **Responsibilities**:
- Execute complex business rules that span multiple entities
- Coordinate operations between different entities in the domain
- Implement domain-specific calculations and validations
- Handle business logic that doesn't naturally fit within a single entity
- Maintain domain invariants and business rules across entity boundaries

### 9. Root Aggregate Entity

🎯 **Purpose** Serve as the central model that represents a complete business concept (like Order or User) and enforces its business rules, ensuring that the object always remains in a valid state according to domain requirements.
📋 **Responsibilities**:
- Encapsulate business logic specific to the entity and child entities itself.
- Use an aggregate root entity to manage interactions within a domain.

---

**Persistence**

---

### 10. Repository

🎯 **Purpose** Provide a clean abstraction layer for persistence operations that hides implementation details and ensures data consistency by managing all storage interactions in a single place.
📋 **Responsibilities**:

**Core Operations**
- Handle CRUD operations for domain entities
- Convert domain entities to and from storage models
- Manage data persistence within transactions
- Support multiple storage targets (database, event stream, ReBAC system)

**Data Integrity**
- Ensure atomic changes within aggregate roots
- Maintain data consistency across related entities
- Validate entity relationships and constraints
- Coordinate writes across different storage systems

**Event Stream Integration**
- Publish domain events to event streams
- Maintain event ordering and causality
- Support event sourcing patterns when needed
- Enable event replay and system rebuilding

**Access Control Storage**
- Record and maintain relationship-based access control data
- Support fine-grained permission queries
- Keep access control data in sync with entity changes

**Upsert Workflow**
1. Receive aggregate root entity from domain service
2. Begin distributed transaction if multiple storage targets
3. Check if entity exists in primary storage
4. Perform update or insert as needed
5. Handle all related entity relationships
6. Update secondary storage systems (events, ReBAC)
7. Commit transaction (or rollback on failure)
8. Return updated/created entity

### 11. ORM

🎯 **Purpose** Provide a seamless translation layer between domain objects and relational databases, enabling developers to work with familiar object-oriented code while ensuring efficient and reliable data persistence.
📋 **Responsibilities**:

**Data Mapping**
- Translate domain entities to database tables and back
- Handle relationships between connected entities
- Validate data and convert between types

**Schema Management**
- Manage database schema definitions
- Handle database migrations
- Provide abstraction layer for database operations

**Performance**
- Optimize database queries
- Support lazy and eager loading of related data
- Handle transactions and concurrent access

### 12. Relational Database

🎯 **Purpose** Provide a reliable and organized way to store business data by using tables with defined relationships, ensuring data can be efficiently queried and remains consistent over time.

📋 **Responsibilities**:
- Store data safely and prevent corruption
- Allow searching and combining data in flexible ways
- Guarantee that database changes are complete and reliable
- Handle multiple users accessing data at the same time
- Keep related data consistent and up-to-date
- Recover data if system crashes occur

## Consequences

### Positive Consequences
- Improved maintainability through clear separation of concerns
- Better testability of business logic
- Easier onboarding of new team members
- Reduced technical debt through consistent patterns
- Easier to intervene when working with AI

### Negative Consequences
- Increased initial development time
- Steeper learning curve for developers new to DDD
- More boilerplate code
- Potential over-engineering for simple CRUD operations
