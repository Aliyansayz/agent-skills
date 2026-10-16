---

name: contextual-code-quality
description: Improve TypeScript code quality by replacing unclear names, unsafe types, duplicated logic, mixed responsibilities, magic values, and fragile error handling with readable, maintainable, strongly typed code. Use when reviewing, refactoring, debugging, or extending existing TypeScript applications, especially sales CRM systems.
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Contextual Code Quality — TypeScript Agent Skill

## 1. Purpose

Act as a TypeScript code-quality agent that makes existing code easier to understand, maintain, test, and extend.

Do not refactor code merely to make it look sophisticated. Improve it so that developers can understand:

* What each function does.
* Why a business rule exists.
* What each variable represents.
* Which data a function expects and returns.
* What happens when data is missing or invalid.
* Which parts of the application are responsible for business logic, data access, and user interaction.
* How a change affects existing application behavior.

Use domain-specific language instead of vague variable and function names.

When working with Sales CRM code, use clear business concepts such as `Lead`, `Contact`, `Opportunity`, `SalesRepresentative`, `LeadStatus`, `PipelineStage`, and `FollowUpDate`.

The objective is understandable, strongly typed, reliable, testable, and maintainable software.

## 2. When to Activate

Use this skill when:

* TypeScript code contains variables such as `x`, `data`, `obj`, `temp`, `thing`, or `result` without sufficient context.
* Functions have vague names such as `process()`, `handle()`, `manage()`, or `doStuff()`.
* A function performs validation, business calculations, database updates, notifications, and logging all at once.
* The application relies on `any`, unsafe type assertions, unexplained strings, or magic numbers.
* Business rules are duplicated in multiple places.
* Errors are silently swallowed or converted into misleading success responses.
* Functions contain deeply nested conditions or excessive branching.
* CRM terminology is inconsistent across components, API endpoints, and database models.
* The code is difficult to test without connecting to a real database or external service.
* A developer wants to improve existing code without changing its intended behavior.

## 3. Core Principles

### 3.1 Prefer domain-specific names

Names should explain their meaning in the application.

Prefer:

```typescript
const leadId = "lead_123";
const salesRepresentativeId = "user_456";
const nextFollowUpDate = new Date();
const qualifiedLeadCount = 12;
```

Avoid:

```typescript
const id = "lead_123";
const user = "user_456";
const date = new Date();
const count = 12;
```

Short names are acceptable when their meaning is obvious within a small scope, such as `i` in a short loop. Do not expand every name mechanically; expand names when additional context improves understanding.

### 3.2 Use TypeScript to represent business concepts

Prefer explicit types, interfaces, discriminated unions, and named domain models over unstructured objects.

Avoid using `any` as an escape hatch. Use `unknown` for genuinely untrusted data and validate it before use.

### 3.3 Separate responsibilities

Each function or module should have a clear primary responsibility.

A CRM service should not have to construct HTTP responses, manipulate UI components, send emails, and execute database queries in one function.

### 3.4 Preserve intended behavior

Before refactoring, identify the existing business rules and expected results.

Do not silently change:

* Lead qualification criteria.
* Pipeline transition rules.
* Sales representative assignments.
* Follow-up scheduling rules.
* Duplicate detection behavior.
* Permissions and access controls.
* API response shapes.
* Database semantics.

If existing behavior appears incorrect or ambiguous, identify the concern explicitly instead of disguising a behavioral change as a refactor.

### 3.5 Prefer simple code over unnecessary abstractions

Do not introduce design patterns, classes, generics, factories, or additional dependencies unless they solve a real problem.

A straightforward function with a descriptive name is often better than an unnecessarily elaborate abstraction.

---

## 4. Poorly Written Sales CRM Example

The following code is intentionally problematic. It illustrates common code-quality issues in a lead-management application.

### 4.1 Before: Poorly written code

```typescript
// bad-crm.ts

export async function go(d: any, db: any, mail: any) {
    try {
        let x = d.name;
        let e = d.email;
        let p = d.phone || "";
        let s = d.status || "new";
        let v = Number(d.value || 0);
        let u = d.user || null;

        if (!x || !e) {
            return { ok: false, msg: "bad data" };
        }

        let old = await db.leads.findOne({ email: e });

        if (old && old.status != "lost") {
            old.v = old.v + v;
            old.s = v > 5000 ? "hot" : old.s;
            old.p = p || old.p;
            await db.leads.update(old.id, old);

            if (old.s == "hot") {
                await mail.send(
                    old.email,
                    "Hot lead",
                    "Lead is hot!"
                );
            }

            return { ok: true, data: old };
        }

        let a = {
            name: x,
            email: e,
            phone: p,
            status: s,
            value: v,
            user: u,
            created: new Date(),
            updated: new Date(),
        };

        let r = await db.leads.insert(a);

        if (v > 5000) {
            await mail.send(
                e,
                "Hot lead",
                "Lead is hot!"
            );
        }

        return { ok: true, data: r };
    } catch (e) {
        console.log(e);
        return { ok: false, msg: "error" };
    }
}
```

### 4.2 Problems the agent should identify

| Problem                      | Example                                                                            | Why it matters                                                                 | Improvement                                                                   |
| ---------------------------- | ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ | ----------------------------------------------------------------------------- |
| Unclear function name        | `go()`                                                                             | Does not communicate the business operation                                    | Use `createOrUpdateLead()` if that accurately describes the behavior          |
| Unclear variables            | `d`, `x`, `e`, `p`, `s`, `v`, `u`, `a`, `r`                                        | Developers must infer each variable's purpose                                  | Use domain-specific names                                                     |
| Unsafe types                 | `any` for input, database, and email service                                       | Removes TypeScript's ability to catch many mistakes                            | Define interfaces and explicit input types                                    |
| Hidden business rule         | `v > 5000`                                                                         | The qualification threshold is unexplained                                     | Define a named constant or documented business rule                           |
| Inconsistent terminology     | `status`, `hot`, `lost`                                                            | Lead status and qualification may represent different concepts                 | Define an explicit lead-status model                                          |
| Mixed responsibilities       | One function validates, queries, updates, creates, sends email, and handles errors | Individual behaviors are difficult to test                                     | Separate application, domain, and infrastructure responsibilities             |
| Potential data inconsistency | A database update can succeed before an email fails                                | The caller may receive an error even though the lead was saved                 | Define notification failure behavior explicitly                               |
| Weak validation              | Only checks `x` and `e`                                                            | Invalid email addresses, bad values, and malformed fields may enter the system | Validate input and business invariants                                        |
| Implicit status rules        | Existing leads are updated unless their status is `"lost"`                         | The meaning of this rule is hidden                                             | Name and document the update policy                                           |
| Unclear persistence contract | `db.leads.findOne()` and `.update()` use undocumented shapes                       | Database assumptions are difficult to verify                                   | Define a repository interface                                                 |
| Generic error response       | `"error"`                                                                          | Loses useful context and complicates troubleshooting                           | Handle expected failures explicitly and log unexpected failures appropriately |
| Inconsistent responses       | `{ ok, msg }` and `{ ok, data }`                                                   | Callers must infer which fields exist in each case                             | Use a discriminated result type or consistent application error model         |
| Mutable, loosely shaped data | `old.v`, `old.s`, `old.p`                                                          | Field names do not match the apparent persisted model                          | Define a consistent `Lead` type                                               |
| Unclear duplicate policy     | Existing records are merged by email                                               | The exact meaning of merging is hidden                                         | Document which fields can be updated and how value is aggregated              |
| Hard-to-test dependencies    | Database and email service are passed as `any`                                     | Contracts and failure cases are not checked by the compiler                    | Define narrow interfaces and inject dependencies                              |

**Important:** This is an illustrative example, not a complete production CRM implementation. A refactor must first confirm the existing CRM's actual behavior, data model, persistence semantics, and business rules.

---

## 5. Refactoring Workflow

Follow this workflow when improving existing TypeScript code.

### Step 1: Understand the context

Inspect the relevant code before modifying it.

Identify:

* The function's purpose.
* Its callers and dependencies.
* Its input and output formats.
* The database schema or repository contract.
* Domain entities and business terminology.
* Existing validation and authorization rules.
* Current tests and expected behavior.
* Error handling and side effects.

Read surrounding modules when needed. Do not infer a function's complete contract from its name alone.

### Step 2: Record the current behavior

Describe the existing logic in plain language.

For the example above, the observed behavior is:

1. Read lead details from the input.
2. Reject a request if the name or email is missing.
3. Find an existing lead by email.
4. If a matching lead exists and its status is not `"lost"`, add the submitted value to the existing value.
5. Mark that existing lead as `"hot"` when the resulting value exceeds the threshold.
6. Replace its phone number when a new nonempty phone number is provided.
7. Update the record and send a notification if the resulting status is `"hot"`.
8. Otherwise, create a new record with timestamps.
9. Notify the lead when its submitted value exceeds the threshold.
10. Return a success or failure object.

This description intentionally distinguishes **observed implementation behavior** from **intended business behavior**. For example, the code tests the existing record's resulting value for one notification path but the submitted value for another. Do not silently treat these rules as equivalent.

### Step 3: Improve naming

Replace generic names with names that express their role.

| Poor name | Better name                                                        |
| --------- | ------------------------------------------------------------------ |
| `go`      | `createOrUpdateLead`                                               |
| `d`       | `leadInput`                                                        |
| `x`       | `leadName`                                                         |
| `e`       | `emailAddress`, or `error` in a catch block                        |
| `p`       | `phoneNumber`                                                      |
| `s`       | `leadStatus`                                                       |
| `v`       | `leadValue`                                                        |
| `u`       | `salesRepresentativeId`                                            |
| `old`     | `existingLead`                                                     |
| `a`       | `newLead`                                                          |
| `r`       | `createdLead`                                                      |
| `db`      | `leadRepository` or `databaseClient`, depending on its actual role |
| `mail`    | `emailNotificationService`                                         |

The improved name must accurately represent the value. Do not rename a value to `salesRepresentative` if it stores only a user ID.

### Step 4: Define the data model

Represent the CRM domain explicitly.

```typescript
export type LeadStatus =
    | "new"
    | "qualified"
    | "contacted"
    | "proposal"
    | "won"
    | "lost";

export interface Lead {
    id: string;
    name: string;
    email: string;
    phoneNumber: string | null;
    status: LeadStatus;
    estimatedValue: number;
    salesRepresentativeId: string | null;
    createdAt: Date;
    updatedAt: Date;
}

export interface CreateLeadInput {
    name: string;
    email: string;
    phoneNumber?: string | null;
    status?: LeadStatus;
    estimatedValue?: number;
    salesRepresentativeId?: string | null;
}
```

This model makes important assumptions explicit.

For example, this version represents `"hot"` as something other than a lead status. A production CRM might model lead temperature as a separate field, a calculated property, or part of a qualification system. Select the representation that matches the actual domain rather than forcing this example's choices onto an existing application.

### Step 5: Define dependency contracts

Do not expose a large, loosely typed database object when the business operation needs only a few specific actions.

```typescript
export interface LeadRepository {
    findByEmail(email: string): Promise<Lead | null>;

    create(
        input: Omit<Lead, "id">
    ): Promise<Lead>;

    update(
        leadId: string,
        changes: Partial<
            Pick<
                Lead,
                | "name"
                | "phoneNumber"
                | "status"
                | "estimatedValue"
                | "salesRepresentativeId"
                | "updatedAt"
            >
        >
    ): Promise<Lead>;
}

export interface EmailNotificationService {
    sendLeadQualificationNotification(
        recipientEmail: string,
        leadName: string
    ): Promise<void>;
}
```

The application now depends on explicit operations instead of arbitrary methods on `any`.

Adapt these interfaces to the actual persistence layer. A database implementation must also handle concurrency, unique constraints, and transactional behavior where required.

### Step 6: Extract business rules

Avoid unexplained thresholds and repeated expressions.

```typescript
const LEAD_QUALIFICATION_VALUE_THRESHOLD = 5_000;

function qualifiesByEstimatedValue(
    estimatedValue: number
): boolean {
    return estimatedValue > LEAD_QUALIFICATION_VALUE_THRESHOLD;
}
```

The function name describes the rule's meaning.

Do not introduce a constant simply to give every literal a name. Name values when they represent a business policy, configuration setting, meaningful unit, or repeated decision.

### Step 7: Separate responsibilities

A healthy CRM design often separates these concerns:

| Layer                | Responsibility                                                 | Typical example               |
| -------------------- | -------------------------------------------------------------- | ----------------------------- |
| Input validation     | Determine whether incoming data has the required shape         | `validateCreateLeadInput()`   |
| Domain rules         | Apply business policies                                        | `qualifiesByEstimatedValue()` |
| Application service  | Orchestrate the use case                                       | `createOrUpdateLead()`        |
| Repository           | Persist and retrieve lead data                                 | `LeadRepository`              |
| Notification service | Send notifications through the selected provider               | `EmailNotificationService`    |
| HTTP controller      | Translate HTTP requests and application results into responses | `createLeadController()`      |
| Tests                | Check business behavior and failure scenarios                  | `createOrUpdateLead.test.ts`  |

Keep the architecture proportional to the application. A small project may not need a separate module for every row.

---

## 6. After: A More Understandable TypeScript Implementation

The following implementation demonstrates improved names, explicit contracts, a domain rule, and separated responsibilities.

**Scope note:** The example preserves the most important observed update rules from the poorly written code, including the `estimatedValue` accumulation, phone-number update, and existing-lead eligibility check. It makes the notification decision consistent by checking the resulting value. Because that changes one edge case from the original implementation, treat it as an intentional business-rule correction that must be reviewed—not as a strictly behavior-preserving refactor.

### 6.1 Domain model and contracts

```typescript
// lead.types.ts

export type LeadStatus =
    | "new"
    | "qualified"
    | "contacted"
    | "proposal"
    | "won"
    | "lost";

export interface Lead {
    id: string;
    name: string;
    email: string;
    phoneNumber: string | null;
    status: LeadStatus;
    estimatedValue: number;
    salesRepresentativeId: string | null;
    createdAt: Date;
    updatedAt: Date;
}

export interface CreateLeadInput {
    name: string;
    email: string;
    phoneNumber?: string | null;
    status?: LeadStatus;
    estimatedValue?: number;
    salesRepresentativeId?: string | null;
}

export interface LeadRepository {
    findByEmail(email: string): Promise<Lead | null>;

    create(
        input: Omit<Lead, "id">
    ): Promise<Lead>;

    update(
        leadId: string,
        changes: Partial<
            Pick<
                Lead,
                | "name"
                | "phoneNumber"
                | "status"
                | "estimatedValue"
                | "salesRepresentativeId"
                | "updatedAt"
            >
        >
    ): Promise<Lead>;
}

export interface EmailNotificationService {
    sendLeadQualificationNotification(
        recipientEmail: string,
        leadName: string
    ): Promise<void>;
}
```

### 6.2 Business rule

```typescript
// lead-qualification.ts

export const LEAD_QUALIFICATION_VALUE_THRESHOLD = 5_000;

export function qualifiesByEstimatedValue(
    estimatedValue: number
): boolean {
    return estimatedValue > LEAD_QUALIFICATION_VALUE_THRESHOLD;
}
```

This function can be tested independently from the database and email provider.

### 6.3 Input validation

```typescript
// lead-validation.ts

import type {
    CreateLeadInput,
    LeadStatus
} from "./lead.types.js";

const VALID_LEAD_STATUSES = new Set<LeadStatus>([
    "new",
    "qualified",
    "contacted",
    "proposal",
    "won",
    "lost"
]);

export function validateCreateLeadInput(
    input: CreateLeadInput
): void {
    if (!input.name?.trim()) {
        throw new Error("Lead name is required.");
    }

    if (!input.email?.trim()) {
        throw new Error("Lead email address is required.");
    }

    // This is a basic format check, not comprehensive email verification.
    const basicEmailPattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

    if (!basicEmailPattern.test(input.email.trim())) {
        throw new Error("Lead email address is invalid.");
    }

    if (
        input.estimatedValue !== undefined &&
        (
            !Number.isFinite(input.estimatedValue) ||
            input.estimatedValue < 0
        )
    ) {
        throw new Error(
            "Estimated lead value must be a finite, non-negative number."
        );
    }

    if (
        input.status !== undefined &&
        !VALID_LEAD_STATUSES.has(input.status)
    ) {
        throw new Error("Lead status is invalid.");
    }
}
```

In a real API, validate incoming JSON at the system boundary. TypeScript types are removed at runtime, so declaring an interface does not guarantee that a client sent valid data.

A schema library such as Zod can provide structured runtime validation when appropriate.

### 6.4 Application service

```typescript
// lead-service.ts

import type {
    CreateLeadInput,
    Lead,
    LeadRepository
} from "./lead.types.js";

import type {
    EmailNotificationService
} from "./lead.types.js";

import {
    qualifiesByEstimatedValue
} from "./lead-qualification.js";

import {
    validateCreateLeadInput
} from "./lead-validation.js";

export class LeadService {
    constructor(
        private readonly leadRepository: LeadRepository,
        private readonly emailNotificationService: EmailNotificationService
    ) {}

    async createOrUpdateLead(
        leadInput: CreateLeadInput
    ): Promise<Lead> {
        validateCreateLeadInput(leadInput);

        const normalizedEmail = leadInput.email.trim().toLowerCase();

        const existingLead =
            await this.leadRepository.findByEmail(normalizedEmail);

        if (existingLead !== null) {
            return this.updateExistingLead(
                existingLead,
                leadInput,
                normalizedEmail
            );
        }

        return this.createNewLead(
            leadInput,
            normalizedEmail
        );
    }

    private async updateExistingLead(
        existingLead: Lead,
        leadInput: CreateLeadInput,
        normalizedEmail: string
    ): Promise<Lead> {
        if (existingLead.status === "lost") {
            // The original implementation does not update this record.
            // Whether a lost lead should instead be reopened is a business decision.
            return existingLead;
        }

        const updatedEstimatedValue =
            existingLead.estimatedValue +
            (leadInput.estimatedValue ?? 0);

        const updatedPhoneNumber =
            leadInput.phoneNumber || existingLead.phoneNumber;

        const updatedLead = await this.leadRepository.update(
            existingLead.id,
            {
                phoneNumber: updatedPhoneNumber,
                estimatedValue: updatedEstimatedValue,
                status: qualifiesByEstimatedValue(updatedEstimatedValue)
                    ? "qualified"
                    : existingLead.status,
                updatedAt: new Date()
            }
        );

        if (
            qualifiesByEstimatedValue(updatedLead.estimatedValue)
        ) {
            await this.emailNotificationService
                .sendLeadQualificationNotification(
                    normalizedEmail,
                    updatedLead.name
                );
        }

        return updatedLead;
    }

    private async createNewLead(
        leadInput: CreateLeadInput,
        normalizedEmail: string
    ): Promise<Lead> {
        const now = new Date();

        const newLead = await this.leadRepository.create({
            name: leadInput.name.trim(),
            email: normalizedEmail,
            phoneNumber: leadInput.phoneNumber ?? null,
            status: leadInput.status ?? "new",
            estimatedValue: leadInput.estimatedValue ?? 0,
            salesRepresentativeId:
                leadInput.salesRepresentativeId ?? null,
            createdAt: now,
            updatedAt: now
        });

        if (
            qualifiesByEstimatedValue(newLead.estimatedValue)
        ) {
            await this.emailNotificationService
                .sendLeadQualificationNotification(
                    newLead.email,
                    newLead.name
                );
        }

        return newLead;
    }
}
```

### 6.5 Business rules that still need a decision

The example above deliberately exposes issues that should be resolved in a real CRM.

**Qualification status:** It maps a qualifying lead to `"qualified"` rather than introducing `"hot"` into `LeadStatus`. A real application may instead need separate fields for qualification status and lead temperature.

**Lost leads:** The existing example does not update a lead whose status is `"lost"`. The replacement preserves that broad rule by returning the existing record, but the API should ideally communicate that no update occurred. A typed result such as `updated`, `created`, or `skipped` can make this explicit.

**Duplicate notifications:** The example sends a notification whenever the resulting value is above the threshold. This could repeatedly notify users for every update. A better rule may send a notification only when a lead first crosses the threshold or first becomes qualified.

**Side-effect failures:** The database may save the lead successfully even if the email service fails. The application needs a defined policy: fail the operation, return a warning, retry asynchronously, or record a notification job in an outbox. For business-critical notifications, an outbox pattern can help coordinate persistence and eventual delivery.

**Concurrent requests:** Two requests can find the same lead and both update its value, potentially losing a change. The repository must provide appropriate transactional or atomic-update behavior.

**Email normalization:** Lowercasing and trimming emails is a common CRM deduplication policy, but it should be consistent with the application's existing data and uniqueness rules.

These decisions belong to the business requirements and system architecture, not merely code formatting.

---

## 7. Return Explicit Results Instead of Ambiguous Objects

Avoid APIs that return differently shaped objects without a declared type.

```typescript
return { ok: false, msg: "bad data" };
return { ok: true, data: lead };
```

For an application service that needs to communicate several outcomes, define a discriminated union.

```typescript
import type { Lead } from "./lead.types.js";

export type LeadOperationResult =
    | {
          outcome: "created";
          lead: Lead;
      }
    | {
          outcome: "updated";
          lead: Lead;
      }
    | {
          outcome: "skipped";
          lead: Lead;
          reason: "lead_is_lost";
      };
```

A controller can then map these outcomes to the appropriate HTTP status and response body.

Use exceptions for exceptional failures and explicit results for ordinary business outcomes. Avoid catching every error and replacing it with an uninformative message.

---

## 8. Naming Conventions

Apply consistent naming throughout the project.

### 8.1 Variables and functions

Use `camelCase`.

```typescript
const leadName = "Example Company";
const estimatedLeadValue = 7_500;

function calculateLeadScore(): number {
    return 75;
}

async function findLeadByEmail(
    emailAddress: string
): Promise<Lead | null> {
    // Repository call goes here.
    throw new Error("Implementation required.");
}
```

The last example is a signature illustration only; it is not intended as a production implementation.

### 8.2 Classes, interfaces, and types

Use `PascalCase`.

```typescript
class LeadService {}

interface LeadRepository {}

type LeadStatus = "new" | "qualified" | "lost";
```

Do not add an `I` prefix to every interface unless the project's established conventions require it.

### 8.3 Constants

Use `UPPER_SNAKE_CASE` for module-level constants that represent fixed configuration or named business policies.

```typescript
const LEAD_QUALIFICATION_VALUE_THRESHOLD = 5_000;
const DEFAULT_LEAD_STATUS: LeadStatus = "new";
```

For values that are configurable by a tenant, user, environment, or deployment, use configuration rather than pretending the values are universal constants.

### 8.4 Boolean variables

Use names that express a condition.

```typescript
const isLeadQualified = true;
const hasOpenOpportunity = false;
const canAssignLead = true;
const shouldSendNotification = false;
```

Avoid ambiguous names such as `flag`, `check`, or `valid`.

### 8.5 Functions

Prefer verbs that communicate an action or result.

| Vague name   | More contextual name                  |
| ------------ | ------------------------------------- |
| `process()`  | `processLeadImport()`                 |
| `handle()`   | `handleLeadStatusChange()`            |
| `getData()`  | `getLeadsByAssignedRepresentative()`  |
| `check()`    | `canTransitionLeadToQualified()`      |
| `doUpdate()` | `updateLeadContactDetails()`          |
| `send()`     | `sendLeadFollowUpReminder()`          |
| `calc()`     | `calculateOpportunityWeightedValue()` |

Use the more specific name only when it accurately reflects the behavior.

### 8.6 Dates, times, and amounts

Names should reveal the meaning and, where necessary, the unit.

Prefer:

```typescript
const followUpDueAt = new Date();
const estimatedValueInUsd = 12_500;
const elapsedDaysSinceContact = 4;
```

Avoid vague values such as:

```typescript
const date = new Date();
const value = 12_500;
const days = 4;
```

Money should use an appropriate currency representation and precision policy. For production financial applications, consider storing monetary values in integer minor units or using a suitable decimal library rather than relying on binary floating-point arithmetic.

---

## 9. TypeScript Quality Rules

### Rule 1: Avoid `any`

Bad:

```typescript
function updateLead(lead: any) {
    return lead.status;
}
```

Better:

```typescript
function getLeadStatus(lead: Lead): LeadStatus {
    return lead.status;
}
```

Use a proper model when the data contract is known. Use `unknown` and runtime validation when data comes from an untrusted source and its shape is not yet established.

### Rule 2: Avoid unjustified type assertions

Bad:

```typescript
const lead = response.data as Lead;
```

This tells the compiler to trust the programmer; it does not validate the response.

Better: validate external data at runtime before treating it as a `Lead`.

### Rule 3: Model impossible states out of the type

Bad:

```typescript
interface LeadResult {
    success: boolean;
    lead?: Lead;
    error?: string;
}
```

This permits contradictory values, including a result that has both a lead and an error.

Better:

```typescript
type LeadResult =
    | { success: true; lead: Lead }
    | { success: false; error: string };
```

Use richer error types when callers need to distinguish validation, authorization, conflict, and infrastructure failures.

### Rule 4: Make optionality explicit

Differentiate among:

* A field that must exist.
* A field that may be omitted in an input.
* A persisted field that may be `null`.
* A field whose value may be `undefined`.

For example:

```typescript
interface CreateLeadInput {
    phoneNumber?: string | null;
}
```

This lets the input omit the phone number or explicitly provide `null`. Decide how those cases should behave during updates instead of relying on accidental JavaScript truthiness.

### Rule 5: Avoid excessive mutation

Use `const` by default.

```typescript
const leadLabel = `${lead.name} (${lead.status})`;
```

Use `let` when reassignment is genuinely needed. Avoid mutation when it makes object changes harder to trace, but do not introduce needless object copying merely to follow a style preference.

### Rule 6: Use `async` and `await` consistently

Avoid mixing nested promise chains with `async`/`await` in the same straightforward control flow.

```typescript
async function getLeadById(
    leadRepository: LeadRepository,
    leadId: string
): Promise<Lead | null> {
    return leadRepository.findById(leadId);
}
```

This signature assumes the repository exposes `findById`. Add the method to its interface and implementation before using this function in a real project.

### Rule 7: Handle errors at the correct boundary

A repository should report data-access failures. An application service should apply business-level error behavior where needed. An HTTP controller should map errors to HTTP responses.

Do not turn every error into a generic successful response, and do not swallow errors just to keep the application running.

### Rule 8: Use explicit return types for public boundaries

Explicit return types make service contracts easier to understand and protect APIs from accidental changes.

```typescript
async function createLead(
    input: CreateLeadInput
): Promise<Lead> {
    // Implementation
    throw new Error("Implementation required.");
}
```

Inference is still useful for straightforward local variables and private expressions.

---

## 10. Sales CRM Domain Terminology

Use consistent definitions across code, documentation, API contracts, and the database.

| Term                       | Definition                                                                                              | Example identifier              |
| -------------------------- | ------------------------------------------------------------------------------------------------------- | ------------------------------- |
| Lead                       | A prospective customer or potential sales opportunity that has not necessarily been qualified           | `leadId`                        |
| Contact                    | A person or organization with whom the business maintains contact information                           | `contactId`                     |
| Account                    | A company or organization represented in the CRM                                                        | `accountId`                     |
| Opportunity                | A potential sale being tracked through the sales pipeline                                               | `opportunityId`                 |
| Sales representative       | A user responsible for developing or managing a lead or opportunity                                     | `salesRepresentativeId`         |
| Lead status                | The current operational state of a lead                                                                 | `leadStatus`                    |
| Pipeline stage             | The stage of a sales opportunity in the sales process                                                   | `pipelineStage`                 |
| Qualification              | The process of evaluating whether a lead meets defined business criteria                                | `qualificationStatus`           |
| Lead score                 | A numeric assessment derived from defined lead-scoring rules                                            | `leadScore`                     |
| Estimated value            | The anticipated monetary value associated with a lead or opportunity                                    | `estimatedValue`                |
| Weighted opportunity value | Estimated opportunity value multiplied by an explicitly defined probability                             | `weightedOpportunityValue`      |
| Follow-up                  | A planned next contact or action                                                                        | `followUpDueAt`                 |
| Activity                   | A recorded interaction, task, call, email, or meeting                                                   | `activityId`                    |
| Conversion                 | A defined transition, such as converting a lead to a contact and opportunity                            | `conversionResult`              |
| Duplicate detection        | A process for identifying records that may represent the same person or organization                    | `duplicateLeadCandidate`        |
| Assignment                 | Allocation of a record to a team or sales representative                                                | `assignedSalesRepresentativeId` |
| Repository                 | A component responsible for retrieving and persisting domain data                                       | `leadRepository`                |
| Application service        | A component that coordinates the steps of a business use case                                           | `leadService`                   |
| Domain rule                | A business policy that determines how the system should behave                                          | `qualifiesByEstimatedValue()`   |
| DTO                        | A data transfer object that defines data exchanged across a boundary                                    | `CreateLeadRequest`             |
| Idempotency                | A property that lets a repeated operation avoid unintentionally applying the same effect multiple times | `idempotencyKey`                |
| Outbox pattern             | A technique for recording messages alongside business changes so they can be delivered reliably later   | `notificationOutbox`            |

Do not use `LeadStatus`, `PipelineStage`, `LeadScore`, and `QualificationStatus` interchangeably. They represent different concepts even when a simple CRM initially combines some of them.

---

## 11. Testing Requirements

A refactor is not complete just because the new code looks cleaner.

### 11.1 Unit-test business rules

```typescript
import { describe, expect, it } from "vitest";

import {
    qualifiesByEstimatedValue,
    LEAD_QUALIFICATION_VALUE_THRESHOLD
} from "./lead-qualification.js";

describe("qualifiesByEstimatedValue", () => {
    it("returns false below the threshold", () => {
        expect(
            qualifiesByEstimatedValue(
                LEAD_QUALIFICATION_VALUE_THRESHOLD - 1
            )
        ).toBe(false);
    });

    it("returns false at the threshold", () => {
        expect(
            qualifiesByEstimatedValue(
                LEAD_QUALIFICATION_VALUE_THRESHOLD
            )
        ).toBe(false);
    });

    it("returns true above the threshold", () => {
        expect(
            qualifiesByEstimatedValue(
                LEAD_QUALIFICATION_VALUE_THRESHOLD + 1
            )
        ).toBe(true);
    });
});
```

The examples use Vitest. Use the test framework already adopted by the project whenever possible.

### 11.2 Test the CRM service

Use mock or in-memory dependencies to check these behaviors:

| Scenario                                  | Expected behavior to verify                                             |
| ----------------------------------------- | ----------------------------------------------------------------------- |
| Missing name                              | Input is rejected                                                       |
| Missing email                             | Input is rejected                                                       |
| Invalid email                             | Input is rejected                                                       |
| Negative estimated value                  | Input is rejected                                                       |
| No matching lead                          | A new lead is created                                                   |
| Existing eligible lead                    | The documented update policy is followed                                |
| Existing lost lead                        | The documented skip or reopening policy is followed                     |
| Value crosses the qualification threshold | The documented status transition occurs                                 |
| Notification provider fails               | The documented notification-failure policy is followed                  |
| Repository fails                          | The error is propagated or mapped according to the application contract |
| Two simultaneous updates                  | No persisted value is silently lost                                     |
| Duplicate lead creation request           | The unique-record policy is enforced                                    |

### 11.3 Add tests before changing uncertain behavior

If behavior is unclear, write a characterization test to record what the existing implementation actually does. Then decide, with explicit requirements, whether the refactor should preserve or change that behavior.

Tests are especially valuable when changing database code, asynchronous workflows, and business-critical side effects.

---

## 12. Refactoring Safety Checklist

Before completing a change, verify the following.

**Understanding**

* [ ] The relevant modules and dependencies were inspected.
* [ ] The current input and output contracts are understood.
* [ ] The existing business behavior was recorded.
* [ ] Ambiguous business rules were identified.

**Naming and types**

* [ ] Names reflect actual business concepts.
* [ ] Unnecessary `any` usage was removed.
* [ ] External input is validated at runtime.
* [ ] Optional, nullable, and required fields are distinguished.
* [ ] Public interfaces and return types are explicit.

**Architecture**

* [ ] Functions have focused responsibilities.
* [ ] Database access is behind a suitable interface where that adds value.
* [ ] Notifications and other side effects are handled deliberately.
* [ ] Duplicate business rules have been centralized where appropriate.
* [ ] New abstractions are justified by actual needs.

**Reliability**

* [ ] Error handling preserves meaningful information.
* [ ] Database consistency and concurrency have been considered.
* [ ] Notification retries and duplicate delivery are considered where relevant.
* [ ] Authorization and tenant boundaries have not been weakened.
* [ ] Existing API and persistence contracts are preserved unless a change was agreed.

**Validation**

* [ ] Relevant unit tests pass.
* [ ] Integration tests are updated where applicable.
* [ ] TypeScript compilation succeeds.
* [ ] Linting and formatting checks pass.
* [ ] A production build succeeds when required.
* [ ] Unverified checks are reported honestly.

---

## 13. Agent Operating Procedure

When a user asks you to improve code, follow this sequence.

1. **Inspect:** Read the relevant source files, types, tests, package configuration, and surrounding call sites.
2. **Diagnose:** Identify naming problems, type-safety risks, duplicated logic, excessive responsibilities, and unclear business rules.
3. **Prioritize:** Address correctness, security, type safety, and maintainability before cosmetic changes.
4. **Plan:** Explain the smallest coherent change that solves the actual problem when a plan would help the user.
5. **Refactor:** Make focused improvements while preserving intended behavior.
6. **Validate:** Run the available type checker, tests, linter, and build. Review the final diff.
7. **Report:** Summarize what changed, what behavior was preserved or intentionally changed, and which validation steps succeeded or could not be run.

Do not rewrite an entire project when a small, localized change is sufficient.

Do not claim that tests, builds, or runtime behavior were verified unless they were actually verified.

When using an incomplete example, label assumptions and distinguish illustrative code from code that has been tested in the user's project.

---

## 14. Definition of Done

A code-quality task is complete when:

* The code's purpose can be understood from its names and structure.
* Business concepts are represented consistently.
* TypeScript catches meaningful mistakes instead of being bypassed by `any`.
* Responsibilities are separated enough to support focused testing.
* Errors and side effects have explicit behavior.
* Existing requirements and contracts are preserved, except for documented and approved changes.
* Relevant tests and static checks pass, or their unverified status is clearly stated.
* The final implementation is no more complex than the problem requires.

**Primary rule:** Make the code easier for the next developer to understand without making the software harder to use, test, operate, or extend.

## 15. Recommended References

Use official documentation for the libraries and tools that the project actually uses:

* [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
* [TypeScript Strictness Options](https://www.typescriptlang.org/tsconfig/#strict)
* [TypeScript ESLint](https://typescript-eslint.io/)
* [Vitest Documentation](https://vitest.dev/guide/)
* [Zod Documentation](https://zod.dev/)

Follow the project's existing conventions where they are reasonable. Introduce new conventions incrementally rather than combining a refactor with an unnecessary wholesale rewrite.
