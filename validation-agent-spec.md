# Validation Agent Specification

## Overview

The Validation Agent is an autonomous AI agent that enforces the constitution on every pull request. The agent loads the Modernization Constitution, runs five validation modules against proposed code changes, executes over 40 named checks with severity levels, produces a 0 to 100 scoring model, and automatically fails any pull request containing CRITICAL violations. The agent operates as a quality gate ensuring all PRs comply with project governance principles before merging.

The Validation Agent triggers on every pull request or commit, analyzes the diff between the proposed changes and the base branch, validates the changes against all constitution rules, generates a detailed validation report with per-check scoring, and posts a summary comment on the pull request. The agent identifies constitutional violations by severity level, provides specific remediation guidance for each failed check, calculates an overall compliance score, and blocks merging for CRITICAL failures.

The Validation Agent is designed to operate independently of human reviewers, running automatically on all pull requests to ensure consistent enforcement of project governance. The agent produces deterministic validation results, maintains historical validation data for trend analysis, and integrates with continuous integration pipelines to enforce quality gates before deployment.

## Validation Modules

### Module 1: Constitution Compliance

**Purpose:** Validate that all changes align with the Modernization Constitution governing principles and migration boundaries.

**Scope:** This module validates against the five governing principles defined in the constitution: Preserve Business Logic, Data Integrity, Contract Compliance, Incremental Verification, and No Regressions. It also ensures changes respect the MUST Preserve, MAY Change, and WILL Drop categories defined in migration boundaries.

**Input Analysis:** The module examines all changed files in the pull request, identifies which constitutional categories are affected, and runs targeted checks based on the nature of changes. For backend changes, it validates business logic preservation. For frontend changes, it verifies contract compliance. For deployment changes, it checks orchestration preservation.

**Output Format:** The module produces a validation report containing:
- Constitution principles evaluated with pass/fail status for each
- Migration boundary violations identified by category
- Specific governance rules violated with line references
- Severity levels assigned to each violation (CRITICAL, HIGH, MEDIUM, LOW)
- Remediation actions required to resolve violations
- Overall module score from 0 to 100

**Checks Performed:**
- Business logic preservation for todo CRUD operations
- Data structure integrity for in-memory state management
- API contract compliance for all endpoints
- Frontend backward compatibility verification
- Deployment orchestration preservation
- CORS configuration validation
- Migration boundary category adherence

**Remediation Guidance:** For each failed check, the module provides specific remediation steps referencing the constitution source lines. For CRITICAL failures, the module indicates that the PR cannot be merged until the violation is resolved and the fix is validated by re-running the agent.

---

### Module 2: Security and Authentication

**Purpose:** Validate that all security changes follow Keycloak integration patterns and do not introduce vulnerabilities during the modernization.

**Scope:** This module validates JWT token implementation, Keycloak client configuration, authentication flow implementation, authorization foundation setup, CORS security posture, and that no security regressions are introduced. It ensures the gradual addition of authentication does not break existing functionality prematurely.

**Input Analysis:** The module examines changes related to authentication middleware, JWT validation logic, Keycloak configuration, token extraction, role-based authorization, and CORS settings. It validates that security changes follow the phased approach defined in the implementation plan.

**Output Format:** The module produces a validation report containing:
- JWT validation implementation correctness
- Keycloak integration configuration validity
- Authentication flow security posture
- Authorization foundation completeness
- CORS policy security evaluation
- Security regression detection
- Severity levels assigned to each finding
- Overall module score from 0 to 100

**Checks Performed:**
- JWT signature verification implementation
- Token expiration validation presence
- Keycloak realm configuration correctness
- Client ID and secret security handling
- Optional vs required auth dependency usage
- Role-based authorization foundation setup
- CORS origin policy validation
- Authentication error response standards
- No premature authentication blocking

**Remediation Guidance:** For security violations, the module provides specific code changes required to meet security best practices. For CRITICAL security issues such as missing JWT validation or broken CORS, the PR must be rejected and fixed before resubmission.

---

### Module 3: Data and Privacy

**Purpose:** Validate that all data-related changes maintain integrity of in-memory state and follow proper data handling practices during migration.

**Scope:** This module validates in-memory state management implementation, thread-safety patterns for concurrent access, data structure preservation from Express to FastAPI migration, JSONPlaceholder integration correctness, soft-delete logic preservation, and that no data loss occurs during the modernization.

**Input Analysis:** The module examines changes to state management code, data model definitions, external API integration, ID generation logic, merging and sorting algorithms, and delete operations. It validates that data behavior remains identical to the original Express implementation.

**Output Format:** The module produces a validation report containing:
- In-memory state structure preservation validation
- Thread-safety implementation verification
- Data model field mapping correctness
- JSONPlaceholder caching behavior validation
- Soft-delete logic preservation check
- ID generation algorithm equivalence
- Merging and sorting logic verification
- Data loss detection
- Overall module score from 0 to 100

**Checks Performed:**
- cached_remote_todos structure matches Express implementation
- local_added_todos array preservation
- deleted_todo_ids Set preservation
- Thread-safe locking mechanisms presence
- JSONPlaceholder first-25 caching behavior
- Title validation logic equivalence
- ID generation algorithm correctness
- Soft-delete local vs remote distinction
- Merging and filtering logic preservation
- Sort by numeric ID ascending order

**Remediation Guidance:** For data integrity violations, the module provides specific code changes to align with the original Express implementation. The module compares line-by-line logic between the Express server.js and FastAPI implementation to ensure identical behavior.

---

### Module 4: Performance and Scalability

**Purpose:** Validate that performance characteristics are maintained or improved during modernization, and that no performance regressions are introduced.

**Scope:** This module validates response times for API endpoints, concurrent request handling, in-memory state access patterns, external API caching efficiency, Docker service startup times, and resource usage profiles. It ensures the FastAPI implementation performs comparably to or better than the Express baseline.

**Input Analysis:** The module examines changes affecting async patterns, database access patterns (even though in-memory), external API calls, middleware overhead, and concurrency handling. It validates that performance optimization patterns are followed and that no anti-patterns are introduced.

**Output Format:** The module produces a validation report containing:
- Async pattern implementation correctness
- Concurrent request handling validation
- Caching efficiency verification
- Middleware overhead assessment
- Response time regression detection
- Memory usage pattern validation
- Thread-safety performance impact
- Overall module score from 0 to 100

**Checks Performed:**
- Async/await patterns used for I/O operations
- No blocking synchronous calls in async functions
- Thread-safe locking does not cause deadlocks
- JSONPlaceholder caching prevents redundant calls
- State access patterns minimize lock contention
- CORS middleware configured for performance
- FastAPI async route usage where appropriate
- Connection pooling for external API calls
- No memory leaks in state management
- Startup time comparable to Express implementation

**Remediation Guidance:** For performance violations, the module provides specific code optimization recommendations. The module identifies blocking patterns that should be converted to async, locking patterns that may cause contention, and caching opportunities that are missed.

---

### Module 5: Accessibility and User Experience

**Purpose:** Validate that accessibility standards are maintained and user experience is not degraded during modernization, particularly for any frontend changes.

**Scope:** This module validates WCAG 2.1 AA compliance for any frontend changes, semantic HTML preservation, keyboard navigation functionality, error handling user experience, loading states and feedback, and that the user experience remains consistent with the original application.

**Input Analysis:** The module examines changes to React components, CSS styling, error displays, loading indicators, form validation feedback, and any user-facing UI elements. Since the frontend is initially preserved, this module primarily validates that no inadvertent regressions occur.

**Output Format:** The module produces a validation report containing:
- WCAG 2.1 AA compliance verification
- Semantic HTML structure preservation
- Keyboard navigation functionality
- Screen reader compatibility
- Error message clarity and accessibility
- Loading state user experience
- Color contrast validation
- Focus indicator visibility
- Overall module score from 0 to 100

**Checks Performed:**
- Semantic HTML elements used appropriately
- Form labels properly associated with inputs
- Error messages accessible via ARIA
- Loading states provide feedback
- Focus indicators visible on keyboard navigation
- Color contrast meets 4.5:1 ratio for normal text
- No keyboard traps introduced
- ARIA attributes present for custom components
- Reduced motion preference respected
- Error states provide clear remediation guidance

**Remediation Guidance:** For accessibility violations, the module provides specific HTML structure changes and ARIA attribute additions needed to meet WCAG standards. The module also validates that any visual changes maintain sufficient color contrast.

## Named Checks

### Constitution Compliance Checks (12 checks)

**CC-001: Business Logic Preservation - Remote Todo Caching**
- **Severity:** CRITICAL
- **Pass Criteria:** The implementation fetches exactly the first 25 todos from JSONPlaceholder on the first request and caches them identically to Express implementation at server.js lines 15-26
- **Remediation:** Ensure the FastAPI load_remote_todos method uses an async HTTP client (httpx), stores only the first 25 items via slice operation, caches the result in a module-level variable or class property, returns the cached value on subsequent calls, and matches the exact JSONPlaceholder URL behavior

**CC-002: Business Logic Preservation - Todo Merging Logic**
- **Severity:** CRITICAL
- **Pass Criteria:** The implementation merges cached remote todos with local added todos, filters out soft-deleted IDs, and sorts by numeric ID ascending identically to Express implementation at server.js lines 28-32
- **Remediation:** Ensure the merging logic concatenates the two data sources, applies a filter that removes any todo where the numeric id exists in the deleted_todo_ids Set, sorts the final result using numeric comparison on the id field, and produces an array matching the Express output format

**CC-003: Business Logic Preservation - ID Generation Algorithm**
- **Severity:** CRITICAL
- **Pass Criteria:** The implementation generates new todo IDs by finding the maximum numeric id across both data sources and adding 1, identically to Express implementation at server.js lines 56-60
- **Remediation:** Ensure the ID generation logic finds the maximum value across both cached_remote_todos and local_added_todos lists, converts all ids to numbers for comparison, handles the edge case where no todos exist (starting at 1), and produces a unique sequential ID

**CC-004: Business Logic Preservation - Soft-Delete Logic**
- **Severity:** CRITICAL
- **Pass Criteria:** The implementation checks if a todo ID exists in local added todos and removes it directly, otherwise adds the ID to the soft-delete Set, identically to Express implementation at server.js lines 85-90
- **Remediation:** Ensure the delete operation searches local_added_todos for the todo id, removes it from the array if found, adds the id to the deleted_todo_ids Set if not found locally, and maintains the distinction between permanent removal of local todos and soft-delete of remote todos

**CC-005: Business Logic Preservation - Title Validation**
- **Severity:** CRITICAL
- **Pass Criteria:** The implementation validates that the title field is not null, undefined, empty, or whitespace-only, returning HTTP 400 with message "Title is required" identically to Express implementation at server.js lines 51-53
- **Remediation:** Ensure the title validation converts the input to string, trims whitespace, checks if the result is empty, returns HTTP 400 status code, and returns an error response with the exact message "Title is required" matching the Express implementation

**CC-006: Data Integrity - In-Memory Structure Preservation**
- **Severity:** HIGH
- **Pass Criteria:** The implementation maintains the three in-memory data structures: cached remote todos list, local added todos array, and deleted todo IDs Set, with identical state management to Express implementation at server.js lines 11-13
- **Remediation:** Ensure the state management module defines three data structures that correspond to the Express implementation, initializes them correctly (null, empty list, empty Set), provides thread-safe access to these structures, and maintains the same data flow through the application lifecycle

**CC-007: Data Integrity - Thread-Safe State Access**
- **Severity:** HIGH
- **Pass Criteria:** The implementation provides thread-safe access patterns for concurrent requests to in-memory state, using threading.Lock or asyncio.Lock appropriately for Python async patterns
- **Remediation:** Ensure all state read operations use shared read locks or appropriate concurrency patterns, all state write operations use exclusive locks to prevent race conditions, lock acquisition is bounded to prevent deadlocks, and the lock scope is minimized to only protect critical sections

**CC-008: Contract Compliance - GET /health Endpoint**
- **Severity:** CRITICAL
- **Pass Criteria:** The GET /health endpoint returns HTTP 200 with JSON object containing status field set to "ok", identically to Express implementation at server.js lines 34-36
- **Remediation:** Ensure the FastAPI health endpoint returns exactly {"status": "ok"} with HTTP status code 200, does not require authentication, has the correct route path "/health", and produces the same JSON structure as the Express implementation

**CC-009: Contract Compliance - GET /api/todos Endpoint**
- **Severity:** CRITICAL
- **Pass Criteria:** The GET /api/todos endpoint returns HTTP 200 with an array of Todo objects sorted by numeric id ascending, with soft-deleted todos filtered out, identically to Express implementation at server.js lines 38-45
- **Remediation:** Ensure the FastAPI todos GET endpoint returns an array of todo objects, each object has the same field structure (userId, id, title, completed, isLocal for local todos), the array is sorted by numeric id in ascending order, todos with IDs in the deleted_todo_ids Set are excluded, and HTTP status code 200 is returned on success

**CC-010: Contract Compliance - POST /api/todos Endpoint**
- **Severity:** CRITICAL
- **Pass Criteria:** The POST /api/todos endpoint accepts title and optional completed fields, returns HTTP 201 with the created todo object including auto-generated id and isLocal flag set to true, identically to Express implementation at server.js lines 47-75
- **Remediation:** Ensure the FastAPI todos POST endpoint accepts a JSON body with title (required) and completed (optional, defaults to false), validates the title is non-empty after trimming, generates the next sequential ID, creates a todo object with userId set to 1, includes isLocal field set to true, appends to local_added_todos, returns HTTP status code 201, and returns the complete created todo object

**CC-011: Contract Compliance - DELETE /api/todos/:id Endpoint**
- **Severity:** CRITICAL
- **Pass Criteria:** The DELETE /api/todos/:id endpoint accepts a numeric id parameter, returns HTTP 200 with JSON object containing message "Todo deleted successfully" and the id field, identically to Express implementation at server.js lines 77-96
- **Remediation:** Ensure the FastAPI todos DELETE endpoint accepts an id path parameter, validates the id is a valid number, performs the appropriate delete operation (remove from local array or add to soft-delete Set), returns HTTP status code 200, and returns JSON object with message "Todo deleted successfully" and the id field matching the Express format

**CC-012: Contract Compliance - Error Response Format**
- **Severity:** HIGH
- **Pass Criteria:** All error responses return JSON object with a message field containing the error description, with HTTP status codes 400 for validation errors and 500 for server errors, identically to Express implementation
- **Remediation:** Ensure all error handlers return a JSON object with a message field, validation errors (title required, invalid id) return HTTP 400, server errors return HTTP 500, error messages match or are equivalent to Express implementation, and the error response format is consistent across all endpoints

---

### Security and Authentication Checks (10 checks)

**SA-001: JWT Signature Verification**
- **Severity:** CRITICAL
- **Pass Criteria:** JWT token validation verifies the signature using the Keycloak realm public key, rejects tokens with invalid signatures, and handles signature verification failures appropriately
- **Remediation:** Ensure JWT validation logic retrieves the Keycloak realm public key, uses the public key to verify the token signature, rejects tokens that fail signature verification with 401 Unauthorized, and logs signature verification failures for debugging

**SA-002: Token Expiration Validation**
- **Severity:** CRITICAL
- **Pass Criteria:** JWT token validation checks the expiration time (exp claim), rejects expired tokens with 401 Unauthorized, and handles clock skew appropriately
- **Remediation:** Ensure JWT validation logic extracts the exp claim from the token payload, compares the exp value to the current time, rejects expired tokens with 401 Unauthorized and appropriate error message, and implements reasonable clock skew tolerance (typically 30-60 seconds)

**SA-003: Keycloak Configuration Security**
- **Severity:** HIGH
- **Pass Criteria:** Keycloak client credentials (client ID and client secret) are stored securely in environment variables, not committed to code, and accessed through proper configuration mechanisms
- **Remediation:** Ensure client ID and client secret are read from environment variables, these values are not hardcoded in source code, environment variables are not logged in error messages, and Docker secrets or .env files are used appropriately for sensitive configuration

**SA-004: Optional Authentication Implementation**
- **Severity:** HIGH
- **Pass Criteria:** Authentication dependencies are implemented as optional during the initial phase, allowing public access to todo endpoints while establishing the auth foundation, following the implementation plan
- **Remediation:** Ensure optional auth dependency allows requests with or without tokens, missing tokens do not block access initially, the dependency can be switched to required in a later phase, and the health check endpoint remains publicly accessible

**SA-005: Role-Based Authorization Foundation**
- **Severity:** MEDIUM
- **Pass Criteria:** Authorization foundation extracts user roles from JWT tokens and defines the expected role structure (todo:read, todo:write, todo:delete, todo:admin) for future enforcement
- **Remediation:** Ensure JWT validation extracts realm_access.roles or resource_access claims, the authorization dependency checks for required roles in token claims, role names match the constitution-defined structure, and role checking logic is present but enforcement can be deferred

**SA-006: CORS Configuration Security**
- **Severity:** MEDIUM
- **Pass Criteria:** CORS middleware is configured to match the Express all-origins policy while considering security implications for production deployment
- **Remediation:** Ensure CORSMiddleware allows all origins during development to maintain compatibility, the configuration is documented with security warnings, the configuration can be tightened for production deployment, and credentials flag is set appropriately based on security requirements

**SA-007: Authentication Error Response Standards**
- **Severity:** MEDIUM
- **Pass Criteria:** Authentication failures return 401 Unauthorized with descriptive error messages, authorization failures return 403 Forbidden, and error responses follow consistent format
- **Remediation:** Ensure missing or invalid tokens return 401 Unauthorized with clear message, unauthorized access attempts return 403 Forbidden, authentication and authorization errors follow the standard error response format with message field, and error messages do not leak sensitive information

**SA-008: No Premature Authentication Blocking**
- **Severity:** CRITICAL
- **Pass Criteria:** Authentication is not enforced before existing functionality is verified working, following the implementation plan phased approach
- **Remediation:** Ensure optional authentication dependency is used initially, public access to endpoints is maintained, authentication enforcement is only added after all existing functionality passes tests, and the implementation follows the phased approach defined in tasks.md

**SA-009: Keycloak Realm Configuration**
- **Severity:** MEDIUM
- **Pass Criteria:** Keycloak realm is configured with correct client setup, valid redirect URIs, and appropriate role definitions
- **Remediation:** Ensure Keycloak realm name matches backend configuration, OpenID Connect client is configured with confidential type, valid redirect URIs include the frontend URL, and client ID and secret match backend environment variables

**SA-010: Token Claim Extraction**
- **Severity:** MEDIUM
- **Pass Criteria:** JWT validation correctly extracts user claims including subject, preferred username, and roles from the token payload for use in authorization
- **Remediation:** Ensure token payload is decoded after signature verification, standard claims (sub, preferred_username) are extracted, role claims are extracted from realm_access.roles or resource_access, claims are made available to downstream authorization logic, and claim extraction handles missing claims gracefully

---

### Data and Privacy Checks (12 checks)

**DP-001: cached_remote_todos Structure**
- **Severity:** CRITICAL
- **Pass Criteria:** The cached_remote_todos data structure stores a list of Todo objects from JSONPlaceholder, holds exactly the first 25 items, and is initialized as None and set after first fetch
- **Remediation:** Ensure the state manager defines a variable or property for cached remote todos, it is initialized to None, it is populated with exactly 25 todos from JSONPlaceholder on first access, it returns the cached value on subsequent accesses, and the structure matches the Express cachedRemoteTodos behavior

**DP-002: local_added_todos Array**
- **Severity:** CRITICAL
- **Pass Criteria:** The local_added_todos data structure stores locally created todos as an array, grows with POST requests, and shrinks with DELETE requests for local todos
- **Remediation:** Ensure the state manager defines a list variable for local added todos, it is initialized as an empty list, new todos are appended with isLocal flag set to true, local todos are removed from this array on delete, and the structure matches the Express localAddedTodos behavior

**DP-003: deleted_todo_ids Set**
- **Severity:** CRITICAL
- **Pass Criteria:** The deleted_todo_ids data structure is a Set of numeric IDs tracking soft-deleted remote todos, populated when deleting non-local todos, and used to filter merged results
- **Remediation:** Ensure the state manager defines a Set for deleted todo IDs, it is initialized as an empty Set, IDs are added to this Set when deleting remote todos, IDs in this Set are filtered from merged results, and the structure matches the Express deletedTodoIds behavior

**DP-004: Thread-Safe Locking Mechanisms**
- **Severity:** HIGH
- **Pass Criteria:** All state modification operations are protected by appropriate locking mechanisms to prevent race conditions in concurrent requests
- **Remediation:** Ensure each state modification method (add, delete, load) uses a lock, the lock is properly acquired and released, lock scope is minimized to only protect critical sections, and the locking pattern is appropriate for Python async (asyncio.Lock) or threading.Lock

**DP-005: JSONPlaceholder Caching Behavior**
- **Severity:** CRITICAL
- **Pass Criteria:** The first 25 todos from JSONPlaceholder are fetched only once and cached, subsequent requests use the cached data without external API calls
- **Remediation:** Ensure the load_remote_todos method checks if cached data exists before fetching, external API call to JSONPlaceholder happens only on first invocation, exactly 25 items are extracted using slice operation, and the cached result is returned for all subsequent calls

**DP-006: Title Validation Logic**
- **Severity:** CRITICAL
- **Pass Criteria:** Title validation converts input to string, trims whitespace, rejects null, empty, or whitespace-only values with 400 status and message "Title is required"
- **Remediation:** Ensure title validation uses Pydantic validator or manual check, the validation converts to string and trims, empty string after trim triggers 400 status, error message matches "Title is required" exactly, and validation happens before ID generation or todo creation

**DP-007: ID Generation Algorithm**
- **Severity:** CRITICAL
- **Pass Criteria:** ID generation finds the maximum numeric ID across both cached_remote_todos and local_added_todos lists, adds 1 to the maximum, and handles edge cases correctly
- **Remediation:** Ensure the algorithm extracts all numeric IDs from both lists, finds the maximum value, adds 1 to generate the next ID, handles empty lists case (starts at 1), and produces sequential IDs without gaps

**DP-008: Soft-Delete Local vs Remote**
- **Severity:** CRITICAL
- **Pass Criteria:** Delete logic checks if todo ID exists in local_added_todos and removes it directly, otherwise adds ID to deleted_todo_ids Set for remote todos
- **Remediation:** Ensure the delete operation searches local_added_todos for the target ID, if found the todo is removed from the array, if not found the ID is added to deleted_todo_ids Set, and the distinction between local and remote todo handling is maintained

**DP-009: Merging and Filtering Logic**
- **Severity:** CRITICAL
- **Pass Criteria:** Merging logic concatenates cached_remote_todos and local_added_todos, filters out IDs present in deleted_todo_ids Set, and produces the combined list
- **Remediation:** Ensure the merge operation combines the two data sources, the filter operation removes todos with IDs in the deleted_todo_ids Set, filtering uses numeric ID comparison, and the result maintains the correct todo objects

**DP-010: Sort by Numeric ID Ascending**
- **Severity:** CRITICAL
- **Pass Criteria:** The merged todo list is sorted by the numeric id field in ascending order, matching the Express implementation sorting behavior
- **Remediation:** Ensure the sort operation uses the numeric id field for comparison, sorting is ascending (lowest to highest id), the sort is applied after merging and filtering, and the sort produces consistent ordering

**DP-011: No Data Loss During Migration**
- **Severity:** HIGH
- **Pass Criteria:** In-memory state management maintains all data correctly during operations, no data is lost between Express and FastAPI implementations
- **Remediation:** Ensure all CRUD operations correctly manipulate state structures, no race conditions cause data corruption, state persists correctly within the application lifecycle, and all operations produce deterministic results

**DP-012: Todo Object Field Mapping**
- **Severity:** HIGH
- **Pass Criteria:** Todo objects have fields userId, id, title, completed, and optionally isLocal, with correct types matching the Express implementation
- **Remediation:** Ensure Pydantic TodoResponse model defines all required fields, userId is int set to 1 for new todos, id is int and auto-generated, title is non-empty string, completed is bool defaulting to false, and isLocal is bool (present only for local todos)

---

### Performance and Scalability Checks (8 checks)

**PF-001: Async Pattern Implementation**
- **Severity:** MEDIUM
- **Pass Criteria:** External I/O operations (JSONPlaceholder fetch) use async/await patterns with appropriate async libraries (httpx)
- **Remediation:** Ensure JSONPlaceholder fetch uses httpx async client, the fetch method is async and awaited, no blocking synchronous calls are made, and async patterns follow Python best practices

**PF-002: No Blocking Calls in Async Functions**
- **Severity:** MEDIUM
- **Pass Criteria:** Async route handlers do not contain blocking synchronous operations that would negate the benefits of async execution
- **Remediation:** Ensure all async endpoint functions use await for I/O operations, CPU-intensive operations are not in the hot path, synchronous libraries are avoided in async contexts, and async patterns are consistently applied

**PF-003: Thread-Safe Performance**
- **Severity:** MEDIUM
- **Pass Criteria:** Thread-safe locking mechanisms do not introduce performance bottlenecks or deadlocks under concurrent load
- **Remediation:** Ensure lock acquisition time is minimized, lock scope covers only critical sections, no nested lock acquisition patterns that could cause deadlock, and lock contention is acceptable under expected concurrent load

**PF-004: JSONPlaceholder Caching Efficiency**
- **Severity:** MEDIUM
- **Pass Criteria:** Caching of JSONPlaceholder todos prevents redundant external API calls on every request, improving performance
- **Remediation:** Ensure cached data is returned on subsequent requests, external API call happens only once, cache hit rate is high for typical usage patterns, and caching logic is efficient

**PF-005: State Access Pattern Optimization**
- **Severity:** LOW
- **Pass Criteria:** In-memory state access patterns minimize lock contention and optimize for read-heavy workloads typical of todo applications
- **Remediation:** Ensure read operations use read locks where appropriate, write operations hold exclusive locks for minimal duration, lock-free patterns are used where safe, and the access pattern matches expected usage patterns

**PF-006: CORS Middleware Configuration**
- **Severity:** LOW
- **Pass Criteria:** CORSMiddleware is configured efficiently without unnecessary overhead that would impact response times
- **Remediation:** Ensure CORS middleware configuration is minimal but complete, unnecessary middleware is not added, middleware order is optimal, and configuration parameters are appropriate

**PF-007: FastAPI Async Route Usage**
- **Severity:** LOW
- **Pass Criteria:** FastAPI routes that perform I/O operations are defined as async functions to leverage the async server capabilities
- **Remediation:** Ensure routes performing external API calls are async, routes with I/O use async/await, the FastAPI app is served by uvicorn async server, and async route patterns are consistently applied

**PF-008: No Memory Leaks in State Management**
- **Severity:** MEDIUM
- **Pass Criteria:** In-memory state management does not leak memory over time, even with many create and delete operations
- **Remediation:** Ensure deleted items are properly removed from all references, no accumulation of unreachable objects, Set operations do not cause memory growth, and state structure sizes are bounded by application usage patterns

---

### Accessibility and User Experience Checks (8 checks)

**AX-001: Semantic HTML Structure**
- **Severity:** MEDIUM
- **Pass Criteria:** Frontend changes (if any) use semantic HTML elements (header, main, nav, section, article) for proper document structure
- **Remediation:** Ensure HTML elements use semantic tags where available, divs are only used when no semantic alternative exists, document structure has proper landmark regions, and the structure is navigable via screen readers

**AX-002: Form Labels Association**
- **Severity:** MEDIUM
- **Pass Criteria:** All form inputs have visible labels properly associated with the input using htmlFor/id attribute matching
- **Remediation:** Ensure each form input has a corresponding label element, the label uses htmlFor attribute pointing to input id, label text describes the input clearly, and placeholder text does not replace labels

**AX-003: Error Message Accessibility**
- **Severity:** MEDIUM
- **Pass Criteria:** Error messages are accessible via ARIA, announced to screen readers, and provide clear guidance for remediation
- **Remediation:** Ensure error displays use ARIA attributes (aria-live, aria-describedby), error messages are announced to assistive technologies, error text is clear and actionable, and error states persist until resolved or dismissed

**AX-004: Loading State Feedback**
- **Severity:** LOW
- **Pass Criteria:** Loading states provide user feedback through visual indicators and descriptive text
- **Remediation:** Ensure loading states have visible indicators (spinner, skeleton, or text), loading text describes what is happening, loading state duration is reasonable, and feedback is consistent across the application

**AX-005: Focus Indicator Visibility**
- **Severity:** MEDIUM
- **Pass Criteria:** Focus indicators are visible on all interactive elements during keyboard navigation
- **Remediation:** Ensure focus outline is visible with sufficient contrast, focus style is consistent across browsers, focus indicators appear on buttons, links, and form inputs, and focus management works correctly

**AX-006: Keyboard Navigation Functionality**
- **Severity:** MEDIUM
- **Pass Criteria:** All functionality is accessible via keyboard, tab navigation follows logical order, and no keyboard traps exist
- **Remediation:** Ensure all interactive elements are keyboard accessible, tab order follows visual order, Escape key closes modals or cancels actions, Enter and Space activate buttons and toggles, and custom components handle keyboard events

**AX-007: Color Contrast Compliance**
- **Severity:** LOW
- **Pass Criteria:** Color contrast ratios meet WCAG 2.1 AA requirements for normal text (4.5:1) and large text (3:1)
- **Remediation:** Ensure primary text has 4.5:1 contrast against background, large text has 3:1 contrast ratio, text on colored backgrounds meets contrast requirements, and contrast is tested in both light and dark themes

**AX-008: ARIA Attributes for Custom Components**
- **Severity:** LOW
- **Pass Criteria:** Custom interactive components have appropriate ARIA attributes to ensure accessibility
- **Remediation:** Ensure custom components use appropriate ARIA roles, ARIA attributes describe component state (aria-expanded, aria-selected), ARIA labels describe purpose for screen readers, and dynamic content updates use ARIA live regions

---

## Scoring Model (0–100)

### Module Weighting

The overall validation score is calculated as a weighted sum of the five module scores:

- **Module 1: Constitution Compliance** - 30% weight (30 points maximum)
- **Module 2: Security and Authentication** - 25% weight (25 points maximum)
- **Module 3: Data and Privacy** - 25% weight (25 points maximum)
- **Module 4: Performance and Scalability** - 10% weight (10 points maximum)
- **Module 5: Accessibility and UX** - 10% weight (10 points maximum)

### Per-Check Scoring

Each check is scored individually based on severity level:

- **CRITICAL severity checks:** 10 points (pass) or 0 points (fail)
- **HIGH severity checks:** 5 points (pass) or 0 points (fail)
- **MEDIUM severity checks:** 2.5 points (pass) or 0 points (fail)
- **LOW severity checks:** 1 point (pass) or 0 points (fail)

### Module Score Calculation

Each module score is calculated as:

`Module Score = (Sum of passing check points) / (Maximum possible points for module) * 100`

For example, Module 1 has 12 checks (6 CRITICAL, 3 HIGH, 2 MEDIUM, 1 LOW):
- Maximum possible points: (6 * 10) + (3 * 5) + (2 * 2.5) + (1 * 1) = 60 + 15 + 5 + 1 = 81 points
- If 10 checks pass and 2 fail: Actual points depend on which checks failed
- Module score = Actual points / 81 * 100

### Overall Score Calculation

The overall validation score is:

`Overall Score = (Module 1 Score * 0.30) + (Module 2 Score * 0.25) + (Module 3 Score * 0.25) + (Module 4 Score * 0.10) + (Module 5 Score * 0.10)`

The overall score ranges from 0 to 100, with 100 representing full constitutional compliance and 0 representing complete failure.

### Score Interpretation

- **90-100:** Excellent compliance, recommended for merge
- **70-89:** Good compliance with minor issues, review before merge
- **50-69:** Marginal compliance with significant issues, requires fixes before merge
- **0-49:** Poor compliance, must be fixed before merge consideration
- **CRITICAL failure:** Any CRITICAL check fails results in automatic FAIL status regardless of overall score

## Auto-Fail on CRITICAL

### FAIL Status Trigger

The Validation Agent automatically assigns a FAIL status to any pull request that contains one or more CRITICAL check failures, regardless of the overall score calculation.

### CRITICAL Check List

The following checks trigger automatic failure:
- **CC-001:** Business Logic Preservation - Remote Todo Caching
- **CC-002:** Business Logic Preservation - Todo Merging Logic
- **CC-003:** Business Logic Preservation - ID Generation Algorithm
- **CC-004:** Business Logic Preservation - Soft-Delete Logic
- **CC-005:** Business Logic Preservation - Title Validation
- **CC-008:** Contract Compliance - GET /health Endpoint
- **CC-009:** Contract Compliance - GET /api/todos Endpoint
- **CC-010:** Contract Compliance - POST /api/todos Endpoint
- **CC-011:** Contract Compliance - DELETE /api/todos/:id Endpoint
- **SA-001:** JWT Signature Verification
- **SA-002:** Token Expiration Validation
- **SA-008:** No Premature Authentication Blocking
- **DP-001:** cached_remote_todos Structure
- **DP-002:** local_added_todos Array
- **DP-003:** deleted_todo_ids Set
- **DP-004:** Thread-Safe Locking Mechanisms
- **DP-005:** JSONPlaceholder Caching Behavior
- **DP-006:** Title Validation Logic
- **DP-007:** ID Generation Algorithm
- **DP-008:** Soft-Delete Local vs Remote
- **DP-009:** Merging and Filtering Logic
- **DP-010:** Sort by Numeric ID Ascending

### PR Comment Format

When CRITICAL failures are detected, the agent posts a PR comment with the following structure:

```
## ❌ Validation Failed - CRITICAL Violations Detected

This pull request cannot be merged due to CRITICAL constitutional violations.

**Overall Score:** [calculated score]/100
**Status:** FAIL

### Critical Violations
[Each CRITICAL failure listed with check ID, name, and remediation]

### Required Actions
1. Address all CRITICAL violations listed above
2. Re-run validation after fixes
3. Ensure no CRITICAL failures remain

### Additional Issues
[Optional: List of non-critical failures for awareness]

---

**Validation Agent Report Generated:** [timestamp]
**Constitution Version:** [constitution version]
```

### No Merge Blocking

Pull requests with CRITICAL violations cannot be merged until:
1. All CRITICAL failures are addressed with code changes
2. The validation agent is re-run on the updated pull request
3. The updated validation shows no CRITICAL failures

Non-critical failures (HIGH, MEDIUM, LOW) do not block merge but should be reviewed and addressed at team discretion.

## Integration

### Trigger

The Validation Agent is triggered automatically on:
- New pull requests created
- Pull request updates (new commits pushed)
- Manual invocation via comment `/validate` (optional)

The agent runs synchronously during continuous integration pipeline execution, ensuring all PRs are validated before they can be merged.

### Inputs

The agent requires the following inputs:

1. **Pull Request Diff:** The full diff between the pull request branch and the target branch (typically main)
2. **Constitution Document:** The Modernization Constitution at `/specs/main/constitution.md` containing all governance principles and migration boundaries
3. **Base Reference Code:** The original Express backend at `/backend/server.js` for business logic comparison
4. **Target Implementation:** The proposed FastAPI backend changes for validation
5. **Configuration:** Agent configuration including severity thresholds, weight values, and integration settings

### Outputs

The agent produces the following outputs:

1. **Validation Report:** Detailed report containing module scores, check results, and overall score
2. **PR Comment:** Summary comment posted to the pull request with pass/fail status
3. **CI/CD Status:** Pass or fail status for continuous integration gate
4. **Historical Data:** Validation results stored for trend analysis and improvement tracking
5. **Remediation Guidance:** Specific guidance for each failed check

### Reporting

The validation report is structured as follows:

```markdown
# Validation Report

## Summary
- Overall Score: [score]/100
- Status: [PASS | FAIL]
- Modules Evaluated: 5
- Checks Executed: [total]
- Checks Passed: [count]
- Checks Failed: [count]

## Module Results

### Module 1: Constitution Compliance
- Score: [score]/100
- Status: [PASS | FAIL]
- Checks: [passed]/[total]
- Critical Failures: [count]

### Module 2: Security and Authentication
- Score: [score]/100
- Status: [PASS | FAIL]
- Checks: [passed]/[total]
- Critical Failures: [count]

### Module 3: Data and Privacy
- Score: [score]/100
- Status: [PASS | FAIL]
- Checks: [passed]/[total]
- Critical Failures: [count]

### Module 4: Performance and Scalability
- Score: [score]/100
- Status: [PASS | FAIL]
- Checks: [passed]/[total]

### Module 5: Accessibility and User Experience
- Score: [score]/100
- Status: [PASS | FAIL]
- Checks: [passed]/[total]

## Detailed Check Results

[Each check with pass/fail status, severity, and remediation if failed]

## Recommendations
[Optional recommendations for improvement not tied to specific checks]

---

Generated by Validation Agent v1.0
Constitution Reference: /specs/main/constitution.md
Execution Time: [timestamp]
```

### Continuous Integration Integration

The Validation Agent integrates with CI/CD pipelines through:

1. **Exit Code:** Returns non-zero exit code for FAIL status, blocking deployment
2. **Artifact Generation:** Produces validation report as CI artifact for review
3. **Status API:** Updates pull request status via commit status API
4. **Comment API:** Posts summary comment to pull request for visibility
5. **Retry Mechanism:** Allows re-validation on commit updates without new PR

### Configuration

The agent supports configuration through:

1. **Severity Thresholds:** Configure which severity levels trigger blocking (default: CRITICAL only)
2. **Module Weights:** Adjust weighting of modules in overall score calculation (default values specified above)
3. **Check Inclusions/Exclusions:** Enable or disable specific checks based on project needs
4. **Remediation Customization:** Provide project-specific remediation guidance templates
5. **Integration Settings:** Configure CI/CD platform-specific settings (GitHub Actions, GitLab CI, etc.)

---

## Appendix: Check Reference Table

| Check ID | Module | Name | Severity | Max Points |
|----------|--------|------|-----------|------------|
| CC-001 | Constitution Compliance | Business Logic Preservation - Remote Todo Caching | CRITICAL | 10 |
| CC-002 | Constitution Compliance | Business Logic Preservation - Todo Merging Logic | CRITICAL | 10 |
| CC-003 | Constitution Compliance | Business Logic Preservation - ID Generation Algorithm | CRITICAL | 10 |
| CC-004 | Constitution Compliance | Business Logic Preservation - Soft-Delete Logic | CRITICAL | 10 |
| CC-005 | Constitution Compliance | Business Logic Preservation - Title Validation | CRITICAL | 10 |
| CC-006 | Constitution Compliance | Data Integrity - In-Memory Structure Preservation | HIGH | 5 |
| CC-007 | Constitution Compliance | Data Integrity - Thread-Safe State Access | HIGH | 5 |
| CC-008 | Constitution Compliance | Contract Compliance - GET /health Endpoint | CRITICAL | 10 |
| CC-009 | Constitution Compliance | Contract Compliance - GET /api/todos Endpoint | CRITICAL | 10 |
| CC-010 | Constitution Compliance | Contract Compliance - POST /api/todos Endpoint | CRITICAL | 10 |
| CC-011 | Constitution Compliance | Contract Compliance - DELETE /api/todos/:id Endpoint | CRITICAL | 10 |
| CC-012 | Constitution Compliance | Contract Compliance - Error Response Format | HIGH | 5 |
| SA-001 | Security & Authentication | JWT Signature Verification | CRITICAL | 10 |
| SA-002 | Security & Authentication | Token Expiration Validation | CRITICAL | 10 |
| SA-003 | Security & Authentication | Keycloak Configuration Security | HIGH | 5 |
| SA-004 | Security & Authentication | Optional Authentication Implementation | HIGH | 5 |
| SA-005 | Security & Authentication | Role-Based Authorization Foundation | MEDIUM | 2.5 |
| SA-006 | Security & Authentication | CORS Configuration Security | MEDIUM | 2.5 |
| SA-007 | Security & Authentication | Authentication Error Response Standards | MEDIUM | 2.5 |
| SA-008 | Security & Authentication | No Premature Authentication Blocking | CRITICAL | 10 |
| SA-009 | Security & Authentication | Keycloak Realm Configuration | MEDIUM | 2.5 |
| SA-010 | Security & Authentication | Token Claim Extraction | MEDIUM | 2.5 |
| DP-001 | Data & Privacy | cached_remote_todos Structure | CRITICAL | 10 |
| DP-002 | Data & Privacy | local_added_todos Array | CRITICAL | 10 |
| DP-003 | Data & Privacy | deleted_todo_ids Set | CRITICAL | 10 |
| DP-004 | Data & Privacy | Thread-Safe Locking Mechanisms | HIGH | 5 |
| DP-005 | Data & Privacy | JSONPlaceholder Caching Behavior | CRITICAL | 10 |
| DP-006 | Data & Privacy | Title Validation Logic | CRITICAL | 10 |
| DP-007 | Data & Privacy | ID Generation Algorithm | CRITICAL | 10 |
| DP-008 | Data & Privacy | Soft-Delete Local vs Remote | CRITICAL | 10 |
| DP-009 | Data & Privacy | Merging and Filtering Logic | CRITICAL | 10 |
| DP-010 | Data & Privacy | Sort by Numeric ID Ascending | CRITICAL | 10 |
| DP-011 | Data & Privacy | No Data Loss During Migration | HIGH | 5 |
| DP-012 | Data & Privacy | Todo Object Field Mapping | HIGH | 5 |
| PF-001 | Performance & Scalability | Async Pattern Implementation | MEDIUM | 2.5 |
| PF-002 | Performance & Scalability | No Blocking Calls in Async Functions | MEDIUM | 2.5 |
| PF-003 | Performance & Scalability | Thread-Safe Performance | MEDIUM | 2.5 |
| PF-004 | Performance & Scalability | JSONPlaceholder Caching Efficiency | MEDIUM | 2.5 |
| PF-005 | Performance & Scalability | State Access Pattern Optimization | LOW | 1 |
| PF-006 | Performance & Scalability | CORS Middleware Configuration | LOW | 1 |
| PF-007 | Performance & Scalability | FastAPI Async Route Usage | LOW | 1 |
| PF-008 | Performance & Scalability | No Memory Leaks in State Management | MEDIUM | 2.5 |
| AX-001 | Accessibility & UX | Semantic HTML Structure | MEDIUM | 2.5 |
| AX-002 | Accessibility & UX | Form Labels Association | MEDIUM | 2.5 |
| AX-003 | Accessibility & UX | Error Message Accessibility | MEDIUM | 2.5 |
| AX-004 | Accessibility & UX | Loading State Feedback | LOW | 1 |
| AX-005 | Accessibility & UX | Focus Indicator Visibility | MEDIUM | 2.5 |
| AX-006 | Accessibility & UX | Keyboard Navigation Functionality | MEDIUM | 2.5 |
| AX-007 | Accessibility & UX | Color Contrast Compliance | LOW | 1 |
| AX-008 | Accessibility & UX | ARIA Attributes for Custom Components | LOW | 1 |

**Total Checks:** 50
**Total Maximum Points:** 237.5
**Constitution Compliance Maximum:** 81 points (34.1% of total)
**Security & Authentication Maximum:** 52.5 points (22.1% of total)
**Data & Privacy Maximum:** 101 points (42.5% of total)
**Performance & Scalability Maximum:** 13 points (5.5% of total)
**Accessibility & UX Maximum:** 14 points (5.9% of total)
