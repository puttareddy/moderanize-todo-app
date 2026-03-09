# Modernization Tasks

## Infrastructure Setup (6 tasks)
- [ ] **Task-1** [P] Initialize FastAPI backend project structure in /Users/puttaiaharugunta/codebase/infi/moderanize-todo-app/backend
- [ ] **Task-2** [P] Create Python 3.11+ virtual environment with requirements.txt containing fastapi, uvicorn, pydantic, httpx, python-keycloak, pytest, pytest-asyncio
- [ ] **Task-3** [P] Set up code formatter (black) and linter (ruff or pylint) with configuration files
- [ ] **Task-4** Configure type checking using mypy or pyright with pyproject.toml settings
- [ ] **Task-5** Create Dockerfile for FastAPI backend using python:3.11-slim or python:3.12-slim base image with multi-stage build
- [ ] **Task-6** [P] Set up testing framework with pytest.ini configuration, pytest-asyncio, httpx testing client, and pytest-cov for coverage reporting

## In-Memory State Implementation (4 tasks)
- [ ] **Task-7** [P] Design Pydantic models: TodoResponse with fields userId (int), id (int), title (str), completed (bool), isLocal (bool); TodoCreateRequest with title (required str) and completed (optional bool defaulting to false); HealthResponse with status (str); ErrorResponse with message (str)
- [ ] **Task-8** Create StateManager class with thread-safe access patterns for cached_remote_todos (Optional[List[Todo]]), local_added_todos (List[Todo]), and deleted_todo_ids (Set[int])
- [ ] **Task-9** [P] Implement StateManager methods: load_remote_todos (fetch from JSONPlaceholder and cache first 25), add_local_todo (validate input, generate ID, append), delete_todo (check local vs remote, handle soft-delete), get_merged_todos (merge, filter soft-deleted, sort by id)
- [ ] **Task-10** Implement FastAPI lifespan events for state initialization on startup using threading.Lock or asyncio.Lock for thread safety

## Backend Migration (8 tasks)
- [ ] **Task-11** [P] Create FastAPI application instance in main.py with CORSMiddleware configured for all origins matching Express implementation at /Users/puttaiaharugunta/codebase/infi/todo-docker-app/backend/server.js
- [ ] **Task-12** Implement GET /health endpoint returning HealthResponse with status field set to 'ok' without authentication
- [ ] **Task-13** Implement GET /api/todos endpoint calling StateManager.get_merged_todos and returning List[TodoResponse] with error handling matching Express format
- [ ] **Task-14** Implement POST /api/todos endpoint accepting TodoCreateRequest, validating title field with 400 error for empty/whitespace titles, calling StateManager.add_local_todo, and returning created TodoResponse with 201 status
- [ ] **Task-15** Implement DELETE /api/todos/{id} endpoint accepting numeric id path parameter, validating id with 400 error for NaN, calling StateManager.delete_todo, and returning success message with id
- [ ] **Task-16** Migrate load_remote_todos logic using httpx async client to fetch from https://jsonplaceholder.typicode.com/todos and cache exactly first 25 todos
- [ ] **Task-17** [P] Configure global exception handlers for HTTPException, validation errors, and unexpected errors returning ErrorResponse with appropriate status codes and message field
- [ ] **Task-18** [P] Add request logging middleware to log method, path, and response status; configure error response format to match Express implementation at /Users/puttaiaharugunta/codebase/infi/todo-docker-app/backend/server.js

## Frontend Preservation (5 tasks)
- [ ] **Task-19** [P] Verify existing React 18.3.1 and Vite 5.4.11 frontend at /Users/puttaiaharugunta/codebase/infi/todo-docker-app/frontend remains unchanged during backend migration
- [ ] **Task-20** [P] Confirm nginx configuration at /Users/puttaiaharugunta/codebase/infi/todo-docker-app/frontend/nginx.conf still proxies /api/ requests to backend service
- [ ] **Task-21** Ensure frontend Dockerfile at /Users/puttaiaharugunta/codebase/infi/todo-docker-app/frontend/Dockerfile remains unchanged with node:20-alpine builder and nginx:1.27-alpine production
- [ ] **Task-22** Verify docker-compose.yml maintains frontend service with depends_on: backend relationship and correct port mappings
- [ ] **Task-23** [P] Test that existing React application at /Users/puttaiaharugunta/codebase/infi/todo-docker-app/frontend/src/App.jsx works with new FastAPI backend without any modifications

## Docker Compose Update (3 tasks)
- [ ] **Task-24** Update docker-compose.yml to replace backend service from node:20-alpine to new FastAPI backend service with same container_name and port 3001
- [ ] **Task-25** [P] Configure environment variables in docker-compose.yml for PORT (default 3001), JSON_PLACEHOLDER_URL, and Keycloak settings
- [ ] **Task-26** Add .dockerignore file to backend directory to exclude unnecessary files from Docker build context

## Keycloak Integration (5 tasks)
- [ ] **Task-27** [P] Add Keycloak service to docker-compose.yml using quay.io/keycloak/keycloak:latest or stable image with environment variables for admin credentials, database vendor, and database connection
- [ ] **Task-28** Configure Keycloak service with database persistence using Docker volume, expose ports 8080 and 8443, and depends_on configuration ensuring Keycloak starts before backend
- [ ] **Task-29** [P] Configure Keycloak realm creation with descriptive name (e.g., todo-app), OpenID Connect client with confidential type, valid redirect URIs, and client ID/secret in backend environment variables
- [ ] **Task-30** Define user roles in Keycloak: todo:read, todo:write, todo:delete, todo:admin and create test user account with appropriate role assignments
- [ ] **Task-31** [P] Create Keycloak initialization script or use environment-based configuration to set up realm and client automatically on container startup

## Authentication Implementation (5 tasks)
- [ ] **Task-32** Implement JWT token validation function using python-keycloak library or direct jose library with Keycloak realm public key retrieval
- [ ] **Task-33** [P] Implement token validation logic: verify signature, check expiration time, validate issuer, validate audience, and extract user claims (sub, preferred_username, realm_access.roles, resource_access)
- [ ] **Task-34** Create FastAPI optional authentication dependency that extracts and verifies Authorization Bearer token from request headers, returning user claims or allowing anonymous access
- [ ] **Task-35** Create FastAPI required authentication dependency that rejects requests without valid tokens with 401 Unauthorized response and appropriate error message
- [ ] **Task-36** [P] Apply optional auth dependency to /api/todos endpoints initially for testing without breaking existing functionality, keep /health endpoint public

## Authorization Foundation (3 tasks)
- [ ] **Task-37** [P] Implement role-based authorization dependency that checks user roles from token claims against required permissions (todo:read, todo:write, todo:delete, todo:admin)
- [ ] **Task-38** Configure authorization error responses to return 403 Forbidden status with appropriate message when user lacks required role
- [ ] **Task-39** Document role definitions and permission mappings for future enforcement phase

## Testing & Verification (10 tasks)
- [ ] **Task-40** [P] Write unit tests for StateManager class methods: load_remote_todos, add_local_todo, delete_todo, get_merged_todos with thread-safe access verification
- [ ] **Task-41** [P] Write unit tests for Pydantic model validation: title validation (non-empty after trimming), default values (completed defaults to false), type conversion (userId, id as numbers)
- [ ] **Task-42** [P] Write integration tests for GET /health endpoint verifying 200 status with status ok
- [ ] **Task-43** [P] Write integration tests for GET /api/todos endpoint verifying 200 status with sorted merged todos array and correct filtering
- [ ] **Task-44** [P] Write integration tests for POST /api/todos with valid data verifying 201 status with created todo and sequential ID generation
- [ ] **Task-45** [P] Write integration tests for POST /api/todos with empty/whitespace title verifying 400 status with validation error message 'Title is required'
- [ ] **Task-46** [P] Write integration tests for DELETE /api/todos/{id} with valid id verifying 200 status with success message and correct soft-delete behavior
- [ ] **Task-47** [P] Write integration tests for DELETE /api/todos/{id} with invalid id (NaN) verifying 400 status with error message 'Invalid todo id'
- [ ] **Task-48** [P] Write integration tests for JWT token validation with valid tokens, expired tokens, and missing tokens
- [ ] **Task-49** Write integration tests for role-based authorization with users having different roles and verifying access control

## Business Rules Verification (7 tasks)
- [ ] **Task-50** [P] Test that first 25 todos are fetched from JSONPlaceholder and cached for subsequent requests without additional external API calls
- [ ] **Task-51** Test that ID generation finds maximum id across both cached_remote_todos and local_added_todos then adds 1, handling edge cases with empty data sources
- [ ] **Task-52** [P] Test soft-delete logic: locally created todos removed from local_added_todos array, remote todos added to deleted_todo_ids Set
- [ ] **Task-53** Test merging logic verifies combined list filters out IDs in deleted_todo_ids Set and sorts by numeric id in ascending order
- [ ] **Task-54** [P] Test title validation rejects null, empty string, and whitespace-only strings with 400 status and error message matching Express implementation
- [ ] **Task-55** Test completed field defaults to false when not provided in POST request body, converting to boolean type
- [ ] **Task-56** Test isLocal flag is set to true only on newly created todos and not present on remote todos from JSONPlaceholder

## API Documentation (3 tasks)
- [ ] **Task-57** [P] Add route descriptions and example values to FastAPI endpoints for auto-generated OpenAPI documentation
- [ ] **Task-58** Configure FastAPI to expose Swagger UI at /docs endpoint with comprehensive API documentation
- [ ] **Task-59** [P] Configure FastAPI to expose ReDoc at /redoc endpoint with styled documentation

## Cleanup & Documentation (5 tasks)
- [ ] **Task-60** [P] Archive or remove existing Node.js backend directory at /Users/puttaiaharugunta/codebase/infi/todo-docker-app/backend after verification
- [ ] **Task-61** Create comprehensive README.md explaining new FastAPI + Keycloak architecture, API endpoints, and deployment instructions
- [ ] **Task-62** [P] Update project README to reflect new backend architecture, remove references to Express.js backend, document new environment variables
- [ ] **Task-63** Document authentication flow with Keycloak including token retrieval, validation process, and error handling
- [ ] **Task-64** Document authorization model with role definitions, permission mappings, and current enforcement status (foundation phase)

## Deployment Configuration (4 tasks)
- [ ] **Task-65** [P] Finalize Docker Compose configuration with all three services: backend (FastAPI), frontend (nginx), Keycloak with correct service dependencies
- [ ] **Task-66** Configure production-ready Docker build process with multi-stage builds, non-root user, security best practices, and optimized image sizes
- [ ] **Task-67** [P] Set up environment variable management using .env files or Docker secrets for sensitive data (Keycloak credentials)
- [ ] **Task-68** Configure health checks for all Docker services to ensure proper startup dependencies and container readiness

## Final Verification (7 tasks)
- [ ] **Task-69** Run complete test suite with pytest to ensure all tests pass with new FastAPI backend
- [ ] **Task-70** [P] Perform end-to-end testing using React frontend to verify full user workflow (load todos, add todo, delete todo)
- [ ] **Task-71** Test authentication flow with Keycloak using test user credentials and verify JWT token validation
- [ ] **Task-72** Test soft-delete functionality across multiple user sessions and verify consistent state management
- [ ] **Task-73** [P] Verify API documentation at /docs endpoint is complete, accurate, and accessible
- [ ] **Task-74** Test Docker Compose startup sequence to ensure all services initialize in correct order with proper networking
- [ ] **Task-75** Verify CORS configuration allows frontend to communicate with backend without errors and matches Express implementation

## Performance & Optimization (5 tasks)
- [ ] **Task-76** [P] Measure response times for all endpoints to ensure performance matches or exceeds Express implementation
- [ ] **Task-77** Test concurrent request handling to verify thread-safe state management works under load without data corruption
- [ ] **Task-78** Verify JSONPlaceholder fetch only happens once and subsequent requests use cached data efficiently
- [ ] **Task-79** [P] Test memory usage to ensure in-memory state management does not leak memory over time
- [ ] **Task-80** Verify nginx proxy performance with new FastAPI backend maintains low latency comparable to Express backend

## Total: 80 tasks

---
**Note:**
- Tasks marked with [P] can be executed in parallel. Others must run sequentially.
- Each task has a unique **Task-N** identifier for better tracking and reference.
- Reference existing codebase at /Users/puttaiaharugunta/codebase/infi/todo-docker-app for comparison and verification.
- All backend code will be generated at /Users/puttaiaharugunta/codebase/infi/moderanize-todo-app/backend.
