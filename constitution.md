# Modernization Constitution

## Project Identity
- **Project Name:** Modernize Todo App
- **Source Path:** /Users/puttaiaharugunta/codebase/infi/todo-docker-app
- **Strategy:** hybrid

## Governing Principles
1. **Preserve Business Logic** — All existing business rules must be faithfully reproduced in the FastAPI backend
2. **Data Integrity** — No data loss during migration; in-memory data structures must maintain identical state management
3. **Contract Compliance** — External API contracts must be maintained; frontend must work without changes initially
4. **Incremental Verification** — Each migration step must be independently testable and verifiable
5. **No Regressions** — All existing functionality must work identically in the new stack before adding authentication

## Current State
- **Languages:** JavaScript (Node.js 20 ES modules), JavaScript (React)
- **Frontend:** React 18.3.1 + Vite 5.4.11, served by nginx:1.27-alpine on port 8080
- **Backend:** Express 4.21.2 on Node.js 20, running on port 3001
- **Database:** In-memory storage (no database)
- **Styling:** Vanilla CSS in styles.css

## Target State
- **Frontend:** React 18.3.1 + Vite 5.4.11 + nginx:1.27-alpine (preserved)
- **Backend:** Python 3.11+ + FastAPI
- **Database:** In-memory storage (preserved)
- **Styling:** Vanilla CSS (preserved)
- **Authentication:** Keycloak (new)

## Migration Boundaries

### MUST Preserve
- **API Contract Behavior:**
  - GET /health endpoint returning status: 'ok'
  - GET /api/todos returning merged todo list
  - POST /api/todos accepting title and optional completed, returning created todo
  - DELETE /api/todos/:id soft-deleting todos
  - Exact error message formats and HTTP status codes (400 for invalid input, 500 for server errors)

- **Data Structures and State Management:**
  - cachedRemoteTodos: List of first 25 todos from JSONPlaceholder, cached after first fetch
  - localAddedTodos: Array of locally created todos with isLocal flag
  - deletedTodoIds: Set tracking soft-deleted todo IDs (numeric)

- **Business Logic:**
  - Fetch exactly first 25 todos from https://jsonplaceholder.typicode.com/todos
  - Merge remote and local todos, filtering out soft-deleted IDs
  - Sort merged result by id ascending (numeric)
  - Generate new todo IDs as maximum of all existing IDs plus 1
  - Delete logic: remove from localAddedTodos if local, otherwise add to deletedTodoIds Set
  - Title validation: non-empty string after trimming
  - Default completed value to false if not provided

- **Frontend Compatibility:**
  - No changes to frontend React application initially
  - Nginx proxy configuration maintaining /api/ to backend routing
  - All API calls from App.jsx must work identically

- **Deployment Configuration:**
  - Docker Compose orchestration structure maintained
  - Backend port 3001, frontend port 8080
  - Service names and dependency relationships preserved

- **CORS Configuration:**
  - CORS enabled for all origins (as in current Express implementation)

### MAY Change
- **Internal Code Organization:**
  - Split monolithic server.js into multiple modules (models, routes, services)
  - Use dependency injection pattern for in-memory state
  - Organize by feature or layer separation

- **Implementation Details:**
  - Use async/await patterns with httpx for external API calls instead of fetch
  - Type definitions via Pydantic models instead of plain JavaScript objects
  - Error handling middleware patterns specific to FastAPI
  - Logging approach and format

- **Naming Conventions:**
  - Snake_case for Python variables and functions
  - Pydantic model names
  - File and module naming following Python conventions

- **Authentication Layer:**
  - Add Keycloak JWT validation as optional dependency
  - Authorization middleware structure
  - Role-based permission definitions

- **API Documentation:**
  - Auto-generated OpenAPI/Swagger docs at /docs endpoint
  - ReDoc documentation at /redoc endpoint

### WILL Drop
- **Legacy Node.js Implementation:**
  - Express framework and all Node.js backend code
  - server.js file and package.json dependencies (cors, express)
  - Current backend Dockerfile using node:20-alpine

- **Manual API Documentation:**
  - No separate API documentation files needed (replaced by auto-generated docs)

## Agent Guidelines
- Agents must read backend/server.js before rewriting any todo-related functionality
- Each FastAPI endpoint must be verified to produce identical JSON responses as Express equivalent
- Implement in-memory state management with thread-safe access patterns
- Test API endpoints with identical request payloads to ensure response parity
- Implement JSONPlaceholder integration with same caching behavior (first 25 todos only)
- Maintain soft-delete logic with identical ID filtering behavior
- Add Keycloak integration only after all existing functionality is verified working
- Frontend must remain unchanged until backend parity is confirmed
- All Docker Compose changes must maintain service discoverability and networking
- CORS middleware must allow same origin policy as current implementation 

