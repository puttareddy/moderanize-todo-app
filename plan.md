# Modernization Plan

## Phase 1: Infrastructure Setup

### Task-1: Initialize FastAPI Backend Project Structure
- Create new FastAPI backend directory structure separate from the existing Node.js backend
- Set up Python virtual environment with Python 3.11+ as the target version
- Create requirements.txt file with core dependencies: fastapi, uvicorn, pydantic, httpx, python-keycloak
- Organize backend code into modular structure with separate modules for models, routes, and services
- Configure development environment with code formatter (black) and linter (ruff or pylint)
- Set up type checking configuration using mypyc or pyright for static type validation

### Task-2: Configure Build and Development Environment
- Create Dockerfile for FastAPI backend using python:3.11-slim or python:3.12-slim base image
- Configure multi-stage Docker build to optimize image size with production dependencies only
- Set up backend start script using uvicorn ASGI server with host 0.0.0.0 and port 3001
- Create .dockerignore file to exclude unnecessary files from Docker build context
- Configure environment variable handling for PORT, JSON_PLACEHOLDER_URL, and Keycloak settings
- Set up development hot-reload using uvicorn with reload flag for local development

### Task-3: Configure Database Connection (In-Memory Storage)
- Design in-memory state management module using Python module-level variables or class-based singleton pattern
- Implement thread-safe access patterns using threading.Lock or asyncio.Lock for concurrent request handling
- Create dependency injection pattern for state management to enable testing with isolated state instances
- Define state data structures: cached_remote_todos as Optional[List[Todo]], local_added_todos as List[Todo], deleted_todo_ids as Set[int]
- Configure state initialization on application startup using FastAPI lifespan events (startup and shutdown)

### Task-4: Set Up Testing Framework
- Install pytest as the testing framework with pytest-asyncio for async endpoint testing
- Install httpx testing client library for making test requests to FastAPI endpoints
- Create pytest.ini configuration file with test discovery patterns and async mode enabled
- Set up test directory structure with fixtures directory for shared test data and helper functions
- Configure test coverage reporting using pytest-cov or coverage.py for measuring code coverage
- Create test utilities for in-memory state reset between test cases to ensure test isolation

## Phase 2: Database Migration (In-Memory State Implementation)

### Task-5: Design In-Memory State Schema
- Define Python data classes using Pydantic models representing Todo entity with fields: userId (int), id (int), title (str), completed (bool), isLocal (bool)
- Design state container class to encapsulate all in-memory data structures with thread-safe access methods
- Define response models: TodoResponse with all Todo fields, TodoCreateRequest with title and optional completed fields
- Define error and health response models: HealthResponse with status field, ErrorResponse with message field
- Document state lifecycle: initialization on startup, mutation during runtime, persistence limited to memory only

### Task-6: Create State Management Module
- Implement StateManager class with methods for accessing and modifying cached_remote_todos, local_added_todos, and deleted_todo_ids
- Implement load_remote_todos method that fetches from JSONPlaceholder and caches result with thread-safe caching logic
- Implement add_local_todo method that validates input, generates next ID, and appends to local_added_todos list
- Implement delete_todo method that checks local_added_todos for removal or adds ID to deleted_todo_ids Set
- Implement get_merged_todos method that combines remote and local todos, filters soft-deleted IDs, and sorts by numeric id ascending
- Add locking mechanisms to all state modification methods to prevent race conditions in concurrent requests

### Task-7: Set Up ORM-Like Models (Pydantic Models)
- Create TodoResponse Pydantic BaseModel subclass with all fields: userId (int), id (int), title (str), completed (bool), isLocal (bool)
- Create TodoCreateRequest Pydantic BaseModel subclass with title field as required string and completed as optional boolean defaulting to false
- Create HealthResponse Pydantic BaseModel with status field as string
- Create ErrorResponse Pydantic BaseModel with message field as string
- Implement field validators using Pydantic validators for title field to ensure non-empty after trimming whitespace
- Configure JSON schema generation for all models to support FastAPI automatic API documentation

### Task-8: Verify Data Integrity
- Create unit tests for StateManager class methods to verify all CRUD operations maintain data consistency
- Test ID generation logic to ensure sequential IDs without gaps across combined data sources
- Test soft-delete logic to verify remote todos are filtered out and local todos are permanently removed
- Test merging logic to verify combined list is properly sorted by numeric id in ascending order
- Verify thread-safe behavior by testing concurrent requests modify state without data corruption
- Compare state behavior with Express implementation at /Users/puttaiaharugunta/codebase/infi/todo-docker-app/backend/server.js

## Phase 3: Backend Migration

### Task-9: Rewrite API Endpoints in FastAPI
- Create FastAPI application instance in main module with CORS middleware configured for all origins matching existing Express implementation
- Implement GET /health endpoint that returns HealthResponse with status field set to 'ok' without authentication
- Implement GET /api/todos endpoint that calls StateManager.get_merged_todos and returns List[TodoResponse]
- Implement POST /api/todos endpoint that accepts TodoCreateRequest, validates title field, calls StateManager.add_local_todo, and returns created TodoResponse with 201 status
- Implement DELETE /api/todos/{id} endpoint that accepts numeric id path parameter, validates id, calls StateManager.delete_todo, and returns success message with id
- Configure automatic OpenAPI documentation generation with descriptions and example values for all endpoints

### Task-10: Migrate Business Logic
- Implement load_remote_todos logic using httpx async client to fetch from https://jsonplaceholder.typicode.com/todos
- Cache exactly first 25 todos from JSONPlaceholder response using slice operation on results list
- Implement ID generation logic that finds maximum id across both cached_remote_todos and local_added_todos then adds 1
- Implement soft-delete logic that checks local_added_todos for todo id and removes directly, otherwise adds to deleted_todo_ids Set
- Implement title validation that checks for null, empty, or whitespace-only title strings and returns 400 error with message 'Title is required'
- Implement default completed status logic that sets completed to false when not provided in request body
- Set isLocal flag to true for all newly created todos and ensure userId is set to 1

### Task-11: Implement Authentication in Target Stack (Foundation Phase)
- Implement JWT token validation dependency that extracts and verifies Authorization Bearer token from request headers
- Configure Keycloak connection settings using environment variables: KEYCLOAK_URL, KEYCLOAK_REALM, KEYCLOAK_CLIENT_ID, KEYCLOAK_CLIENT_SECRET
- Implement token validation logic that verifies signature using Keycloak realm public key, checks expiration time, and extracts user claims
- Create optional authentication dependency that can be applied to endpoints without breaking existing public access
- Keep /health endpoint public without authentication requirements
- Apply optional auth dependency to /api/todos endpoints with graceful degradation for missing tokens in initial phase

### Task-12: Set Up Middleware and Error Handling
- Configure CORSMiddleware in FastAPI with allow_origins set to all origins, allow_methods to all HTTP methods, allow_headers to all headers, allow_credentials to true
- Implement global exception handler for HTTPException that returns ErrorResponse with appropriate status code and message
- Implement global exception handler for validation errors that returns 400 status with field-specific error messages
- Implement global exception handler for unexpected errors that returns 500 status with generic error message
- Add request logging middleware to log incoming requests with method, path, and response status
- Configure error response format to match existing Express implementation with message field

## Phase 4: Frontend Migration

### Task-13: Set Up Target Frontend Framework (Preservation Phase)
- Verify existing React 18.3.1 and Vite 5.4.11 frontend remains unchanged during backend migration
- Confirm nginx configuration at /Users/puttaiaharugunta/codebase/infi/todo-docker-app/frontend/nginx.conf still proxies /api/ requests to backend
- Ensure frontend Dockerfile remains unchanged with node:20-alpine builder stage and nginx:1.27-alpine production stage
- Verify docker-compose.yml maintains frontend service with depends_on: backend relationship
- Confirm frontend package.json dependencies remain unchanged with react and vite versions preserved
- Test that existing React application at /Users/puttaiaharugunta/codebase/infi/todo-docker-app/frontend/src/App.jsx works with new FastAPI backend

### Task-14: Migrate UI Components (Preservation Phase)
- Keep App.jsx component structure unchanged with existing state management for todos, title, loading, submitting, and error
- Preserve existing loadTodos, handleAddTodo, and handleDeleteTodo functions that call /api/todos endpoints
- Maintain todo form with text input and submit button that posts to /api/todos with title and completed false
- Preserve todo list display that renders each todo with id prefix, title, completion status, and delete button
- Keep error display component that shows error messages from API responses
- Ensure delete button continues to call DELETE /api/todos/{id} endpoint and filters deleted todo from local state

### Task-15: Implement Routing (Preservation Phase)
- Maintain existing single-page application structure without client-side routing
- Verify nginx proxy configuration routes all /api/ requests to backend service on port 3001
- Ensure Vite configuration at /Users/puttaiaharugunta/codebase/infi/todo-docker-app/frontend/vite.config.js remains unchanged
- Preserve SPA fallback routing in nginx.conf with try_files $uri /index.html directive
- Confirm no changes needed to routing since frontend is single page without client-side navigation

### Task-16: Migrate State Management (Preservation Phase)
- Preserve existing React state using useState hooks for todos, title, loading, submitting, and error
- Keep existing useEffect hook that calls loadTodos on component mount
- Maintain local state updates that directly modify React state without global state management
- Ensure no changes to state management pattern since frontend must work identically with new backend
- Verify all API calls continue to use fetch API with same request and response handling

### Task-17: Apply Target Styling Framework (Preservation Phase)
- Preserve existing vanilla CSS in styles.css without any framework migrations
- Confirm no CSS framework additions or changes during backend modernization
- Maintain existing class names: page, container, todo-form, todo-list, todo-item, meta, delete-button, error, status
- Keep existing styling approach with standard CSS without utility classes or preprocessors
- Ensure visual appearance remains identical since frontend code is unchanged

## Phase 5: Testing & Verification

### Task-18: Write Unit Tests for Migrated Code
- Create unit tests for StateManager class methods: load_remote_todos, add_local_todo, delete_todo, get_merged_todos
- Write tests for Pydantic model validation including title validation, default values, and type conversion
- Test ID generation logic to ensure sequential IDs without gaps in edge cases
- Test soft-delete logic to verify correct behavior for both local and remote todos
- Test merging logic to verify filtering and sorting behavior with various data combinations
- Test thread-safe state access with concurrent modification scenarios

### Task-19: Write Integration Tests for API Endpoints
- Create integration tests for GET /health endpoint to verify returns 200 status with status ok
- Create integration tests for GET /api/todos to verify returns 200 status with sorted merged todos array
- Create integration tests for POST /api/todos with valid data to verify returns 201 with created todo
- Create integration tests for POST /api/todos with empty title to verify returns 400 with validation error
- Create integration tests for DELETE /api/todos/{id} to verify returns 200 with success message
- Create integration tests for DELETE /api/todos/{id} with invalid id to verify returns 400 with error message

### Task-20: Verify All Business Rules Are Preserved
- Test that first 25 todos are fetched from JSONPlaceholder and cached for subsequent requests
- Verify that cached_remote_todos persists across multiple requests to avoid repeated external API calls
- Test that local_added_todos grows with POST requests and shrinks with DELETE requests for local todos
- Verify that deleted_todo_ids Set correctly tracks soft-deleted remote todo IDs
- Test that merged result filters out todos with IDs in deleted_todo_ids Set
- Verify that merged result is sorted by numeric id in ascending order
- Test that ID generation finds maximum across both data sources and increments by 1
- Verify that title validation rejects empty, null, or whitespace-only strings with 400 status
- Test that completed field defaults to false when not provided in POST request
- Verify that isLocal flag is set to true only on locally created todos

### Task-21: Performance Testing
- Measure response times for all endpoints to ensure performance matches or exceeds Express implementation
- Test concurrent request handling to verify thread-safe state management works under load
- Verify JSONPlaceholder fetch only happens once and subsequent requests use cached data
- Test memory usage to ensure in-memory state management does not leak memory
- Verify nginx proxy performance with new FastAPI backend maintains low latency

## Phase 6: Keycloak Integration

### Task-22: Add Keycloak to Docker Compose
- Add Keycloak service to docker-compose.yml using quay.io/keycloak/keycloak:latest image
- Configure Keycloak environment variables: KEYCLOAK_ADMIN, KEYCLOAK_ADMIN_PASSWORD, DB_VENDOR, DB_ADDR, DB_USER, DB_PASSWORD
- Set up Keycloak database persistence using Docker volume for data durability
- Expose Keycloak ports 8080 and 8443 for HTTP and HTTPS access within Docker network
- Create Keycloak initialization script or use environment-based configuration to set up realm and client
- Configure Keycloak service to start before backend service using depends_on directive

### Task-23: Configure Keycloak Realm and Client
- Create Keycloak realm named todo-app or similar descriptive name for the application
- Configure OpenID Connect client with confidential client type for backend JWT validation
- Set up valid redirect URIs for potential future frontend authentication flow
- Configure client ID and client secret in backend environment variables for JWT validation
- Define user roles in Keycloak: todo:read, todo:write, todo:delete, todo:admin for authorization foundation
- Create test user account with appropriate role assignments for testing authentication flow

### Task-24: Implement JWT Token Validation
- Implement JWT validation function using python-keycloak library or direct jose library for token verification
- Configure Keycloak realm public key retrieval for signature verification
- Implement token header extraction from Authorization request header with Bearer token format
- Implement token validation checks: signature verification, expiration time check, issuer validation, audience validation
- Implement user claims extraction: sub (subject), preferred_username, realm_access.roles, resource_access
- Create FastAPI dependency function for authentication that returns user claims or raises 401 exception

### Task-25: Set Up Authentication Middleware
- Create optional authentication dependency that allows requests with or without tokens for backward compatibility
- Create required authentication dependency that rejects requests without valid tokens with 401 Unauthorized response
- Implement role-based authorization dependency that checks user roles against required permissions
- Apply optional auth to /api/todos endpoints initially to test token validation without breaking existing functionality
- Configure error responses for authentication failures to match RESTful standards with 401 status and appropriate message

## Phase 7: Cleanup & Documentation

### Task-26: Remove Legacy Code
- Remove or archive existing Node.js backend directory at /Users/puttaiaharugunta/codebase/infi/todo-docker-app/backend
- Delete Express backend dependencies from project documentation and deployment references
- Remove Node.js backend service from Docker Compose and replace with FastAPI backend service
- Archive original server.js file for reference during verification phase
- Clean up any temporary or experimental files created during migration process
- Update project README to reflect new FastAPI backend architecture

### Task-27: Update Documentation
- Create comprehensive README.md explaining new architecture with FastAPI and Keycloak
- Document API endpoints with examples using auto-generated /docs Swagger UI as reference
- Create API documentation section describing all endpoints, request formats, and response formats
- Document authentication flow with Keycloak including token retrieval and validation
- Document authorization model with role definitions and permission mappings
- Add deployment instructions for Docker Compose with all three services
- Document environment variables required for backend configuration and Keycloak integration

### Task-28: Configure Deployment
- Finalize Docker Compose configuration with all services: backend (FastAPI), frontend (nginx), Keycloak
- Configure production-ready Docker build process with multi-stage builds and security best practices
- Set up environment variable management using .env files or Docker secrets for sensitive data
- Configure health checks for all services to ensure proper startup dependencies
- Document port mappings for external access: backend on 3001, frontend on 8080, Keycloak on 8080/8443
- Create deployment scripts or instructions for local development setup
- Verify Docker networking configuration allows service discovery between containers

### Task-29: Final Verification
- Run complete test suite to ensure all tests pass with new architecture
- Perform end-to-end testing using React frontend to verify full user workflow
- Test authentication flow with Keycloak using test user credentials
- Test soft-delete functionality across multiple user sessions
- Verify API documentation at /docs endpoint is complete and accurate
- Test Docker Compose startup sequence to ensure all services initialize correctly
- Verify CORS configuration allows frontend to communicate with backend without errors
- Perform performance comparison with original Express implementation to ensure acceptable performance

## Summary

This migration plan transforms the Todo Demo application from an Express.js backend to a FastAPI backend with Keycloak authentication while preserving complete functional compatibility. The phased approach ensures each step is independently testable and verifiable, with the frontend remaining unchanged until backend parity is confirmed. The foundation for role-based authorization is established through Keycloak integration, providing a path for future permission-based access control.

Total Estimated Tasks: 29
Phases: 7
Critical Dependencies: FastAPI, Pydantic v2, httpx, python-keycloak, Keycloak, pytest
Preserved Components: React 18.3.1, Vite 5.4.11, nginx 1.27-alpine, Docker Compose orchestration
