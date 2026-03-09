# Modernization Specification

## Overview

The Todo Demo application is a simple task management system that demonstrates a React frontend communicating with an Express.js backend. The application allows users to view, add, and delete todos from a combined data source consisting of pre-populated remote todos from JSONPlaceholder and locally created todos. The modernization project aims to rewrite the Express.js backend in Python using FastAPI, add Keycloak authentication and authorization capabilities, while maintaining complete backward compatibility with the existing React frontend and Docker Compose deployment infrastructure.

The current implementation uses an in-memory data store with a hybrid caching strategy: the first 25 todos are fetched from JSONPlaceholder and cached, while locally created todos are maintained separately. A soft-delete mechanism tracks deleted todo IDs to filter them from the merged result. The modernization will preserve all existing business logic, data structures, API contracts, and deployment orchestration while introducing type safety through Pydantic models, auto-generated API documentation, and a foundation for role-based access control through Keycloak integration.

## Current Architecture

The existing architecture follows a monolithic Express.js backend pattern with a single-file server implementation at `/backend/server.js`. The application uses a three-tier containerized deployment orchestrated by Docker Compose: an Express.js backend on port 3001, a React frontend built with Vite and served by nginx on port 8080, with nginx proxying all `/api/` requests to the backend service. The backend uses in-memory state management through three global variables: `cachedRemoteTodos` for caching the initial JSONPlaceholder response, `localAddedTodos` for storing locally created todos, and `deletedTodoIds` as a Set for tracking soft-deleted IDs.

Data flows from the frontend through nginx proxy to the Express backend, which on the first `/api/todos` GET request fetches 25 todos from JSONPlaceholder and caches them. Subsequent GET requests merge the cached remote todos with locally added todos, filter out any soft-deleted IDs using the Set, and sort by numeric ID in ascending order. POST requests to create new todos require a non-empty title, auto-generate the next sequential ID based on the maximum existing ID from both sources, default completed status to false, and mark the new todo as local. DELETE requests check if the todo ID exists in the local array and remove it directly, otherwise add the ID to the soft-delete Set.

The backend runs as a Node.js ES module application using Express 4.21.2 with CORS middleware enabled for all origins. The frontend is a React 18.3.1 application using Vite 5.4.11 as the build tool, with production builds served through nginx 1.27-alpine. The Docker Compose configuration defines two services with the frontend depending on the backend, maintaining service discoverability through Docker networking.

## Target Architecture

The target architecture will retain the three-tier containerized deployment structure while replacing the Express.js backend with a FastAPI application running on Python 3.11+. The backend will be organized into a modular structure with separate concerns: Pydantic models for request/response validation in a models module, route definitions for API endpoints in a routes module, and service layer functions for business logic in a services module. In-memory state management will be implemented using dependency injection patterns with thread-safe access to support concurrent requests.

A Keycloak service will be added to the Docker Compose orchestration, providing OpenID Connect authentication and JWT token validation. The FastAPI backend will implement middleware dependencies for JWT validation, extracting user claims and roles from tokens for authorization checks. CORS middleware will be configured to match the existing all-origins policy. The API documentation will leverage FastAPI's automatic OpenAPI generation, providing Swagger UI at `/docs` and ReDoc at `/redoc` endpoints.

The React frontend will initially remain unchanged to ensure backward compatibility, with future optional enhancements to include Keycloak login UI, token storage in browser storage, and Authorization header injection in API requests. The nginx proxy configuration will be preserved to maintain the `/api/` routing to the backend. Docker service names and networking will maintain the existing orchestration pattern with the addition of the Keycloak service.

## Data Models

### Todo Entity

The Todo entity represents a single task item with the following fields:
- userId: number, represents the user who owns the todo, always set to 1 in the current implementation
- id: number, unique identifier for the todo, numeric ascending order
- title: string, the task description text, required field, must be non-empty after trimming
- completed: boolean, task completion status, defaults to false if not provided
- isLocal: boolean, flag indicating whether the todo was created locally, only present for locally added todos

### Local Added Todo List

The localAddedTodos data structure is an array that stores all todos created through the POST endpoint. Each entry in this array has the isLocal flag set to true. This array grows as new todos are created and shrinks as locally created todos are deleted through the DELETE endpoint.

### Cached Remote Todos

The cachedRemoteTodos data structure is a list that holds the first 25 todos fetched from JSONPlaceholder. This list is populated on the first GET request to `/api/todos` and then cached for all subsequent requests. Each todo in this list does not have the isLocal flag.

### Deleted Todo IDs Set

The deletedTodoIds data structure is a Set containing numeric IDs of todos that have been soft-deleted. This Set is populated when deleting todos that originated from the JSONPlaceholder remote source. Locally created todos are removed from the localAddedTodos array directly and not added to this Set.

### Health Response

The Health response model contains a single field:
- status: string, always returns 'ok' for successful health checks

### Error Response

The Error response model contains a single field:
- message: string, human-readable error description

## API Endpoints

### GET /health

Returns health status of the backend service. No authentication required. Returns a JSON object with a status field set to 'ok'. On success, HTTP status code 200. This endpoint should remain public without Keycloak authentication requirements.

### GET /api/todos

Retrieves the merged list of all todos combining cached remote todos from JSONPlaceholder and locally added todos. No authentication required in the initial implementation. On success, returns an array of Todo objects sorted by numeric id in ascending order, with soft-deleted todos filtered out. HTTP status code 200 on success. The endpoint performs the following operations: checks if remote todos are cached and fetches from JSONPlaceholder if not, caches exactly the first 25 todos, merges with localAddedTodos array, filters out any IDs present in deletedTodoIds Set, and sorts by numeric id ascending. On failure, returns an error response with message field and HTTP status code 500.

### POST /api/todos

Creates a new todo item. No authentication required in the initial implementation. Accepts a JSON request body with the following fields:
- title: string, required, must be non-empty after trimming
- completed: boolean, optional, defaults to false if not provided

Returns the created Todo object on success with HTTP status code 201. The endpoint performs the following operations: ensures remote todos are loaded and cached, calculates the new ID by finding the maximum numeric id across both cached remote todos and local added todos then adding 1, trims the title string, creates a new todo object with userId set to 1, the calculated id, trimmed title, completed status as boolean, and isLocal flag set to true, appends the new todo to localAddedTodos array, and returns the created todo. If title validation fails, returns error response with message 'Title is required' and HTTP status code 400. On server errors, returns error response with message and HTTP status code 500.

### DELETE /api/todos/:id

Soft-deletes a todo by its numeric ID. No authentication required in the initial implementation. The id parameter is a numeric path parameter. Returns a JSON object on success with fields:
- message: string, always 'Todo deleted successfully'
- id: number, the ID of the deleted todo

HTTP status code 200 on success. The endpoint performs the following operations: parses and validates the id as a number, returns error 'Invalid todo id' with HTTP status code 400 if id is not a valid number, checks if the id exists in localAddedTodos array, removes the todo from localAddedTodos array if found locally, otherwise adds the id to the deletedTodoIds Set for remote todos. On server errors, returns error response with message and HTTP status code 500.

### GET /docs

FastAPI auto-generated Swagger UI documentation for all API endpoints. Returns an interactive HTML interface for exploring and testing the API. No authentication required. This is a new endpoint provided by FastAPI.

### GET /redoc

FastAPI auto-generated ReDoc documentation for all API endpoints. Returns a styled HTML documentation page for the API. No authentication required. This is a new endpoint provided by FastAPI.

## Business Rules

### Remote Todo Caching Rule

Source file: `/backend/server.js` lines 15-26

On the first request to retrieve todos, the system must fetch exactly the first 25 todos from JSONPlaceholder at https://jsonplaceholder.typicode.com/todos and cache them in memory. All subsequent requests must use the cached list without additional external API calls. The caching is implemented through the cachedRemoteTodos variable being null initially and set after the first successful fetch.

### Todo Merging Rule

Source file: `/backend/server.js` lines 28-32

The system must merge the cached remote todos list with the localAddedTodos array by concatenating them into a single collection, filtering out any todo whose numeric id exists in the deletedTodoIds Set, and sorting the final result by numeric id in ascending order. This ensures soft-deleted todos are hidden and the list maintains consistent ordering.

### ID Generation Rule

Source file: `/backend/server.js` lines 56-60

When creating a new todo, the system must generate a new unique numeric id by finding the maximum numeric id across all todos in both the cached remote todos list and the localAddedTodos array, then adding 1 to that maximum. If no todos exist, the new id starts at 1. This ensures sequential IDs without gaps in the combined dataset.

### Soft-Delete Rule

Source file: `/backend/server.js` lines 85-90

When deleting a todo, the system must check if the todo id exists in the localAddedTodos array. If found, the todo must be removed directly from the localAddedTodos array. If not found, the id must be added to the deletedTodoIds Set. This distinction ensures locally created todos are permanently removed while remote todos are tracked for filtering.

### Title Validation Rule

Source file: `/backend/server.js` lines 51-53

When creating a todo, the title field must be validated to ensure it is not null or empty. After converting to string and trimming whitespace, if the title is an empty string, the request must be rejected with HTTP status code 400 and error message 'Title is required'.

### Default Completed Status Rule

Source file: `/backend/server.js` line 49

When creating a todo, if the completed field is not provided in the request body, it must default to false. The completed value must always be converted to a boolean type.

### Local Todo Flagging Rule

Source file: `/backend/server.js` line 67

All todos created through the POST endpoint must have the isLocal field set to true. This flag is only present on locally created todos and not on remote todos from JSONPlaceholder.

### User ID Default Rule

Source file: `/backend/server.js` line 63

All todos created through the POST endpoint must have the userId field set to 1. This represents the default user ownership in the current implementation.

## UI Components

### App Component

The main application component located at `/frontend/src/App.jsx` manages the entire todo interface. It maintains state for the todos array, title input field, loading status, form submission status, and error messages. The component automatically loads todos on mount using the useEffect hook, presents a form for adding new todos, displays the list of todos with delete functionality, and shows error messages when they occur.

### Todo Form

A form component within the App that accepts user input for new todo titles. It consists of a text input field bound to the title state and a submit button that disables while the add operation is in progress. The form submission calls the handleAddTodo function which sends a POST request to create the todo.

### Todo List Display

A conditional display area that shows either a loading message, a list of todo items, or remains empty based on the loading state and todos array contents. When todos are loaded, it renders an unordered list with each todo item showing the numeric id, title, and completion status.

### Todo Item

Each individual todo item displays the todo information in a list element. The item includes the todo id prefixed with a hashtag, the todo title text, and a metadata line showing completion status as either 'Completed' or 'Pending'. A delete button is provided to remove the todo item.

### Error Display

A conditional error message area that appears when an error state contains a message. It displays the error text returned from API calls and persists until the next successful operation or explicit state clearing.

### Delete Button

A button component within each todo item that triggers the delete operation. When clicked, it calls the handleDeleteTodo function with the todo id, which sends a DELETE request to the backend and updates the local state to remove the deleted item.

## Authentication & Authorization

### Current State

The current implementation has no authentication or authorization mechanisms. All API endpoints are publicly accessible without any credential verification. There is no user identification, session management, or permission checks. Any client can perform read, write, and delete operations on todos without restriction.

### Target Authentication

The target authentication implementation will use Keycloak as the OpenID Connect identity provider. Keycloak will be deployed as a Docker Compose service alongside the backend and frontend. The FastAPI backend will implement JWT token validation middleware that verifies the Authorization header for bearer tokens on protected endpoints. The validation will include checking the token signature against the Keycloak realm public key, verifying token expiration, and extracting user claims from the token payload.

Authentication flow will follow the standard Authorization Code Flow pattern for web applications, though the initial phase may implement optional authentication to maintain backward compatibility. Valid tokens will be required for accessing protected todo endpoints, with the health check endpoint remaining public.

### Target Authorization

The target authorization foundation will implement role-based access control using claims extracted from validated JWT tokens. Roles will be defined in Keycloak and assigned to users, with specific permissions mapped to todo operations. The permission structure will include roles for read access, write access, delete access, and administrative access.

Authorization checks will be implemented as FastAPI dependency injection middleware that can be applied to individual endpoints. The middleware will verify that the authenticated user's token contains the required role or permission claim for the requested operation. Initial implementation may focus on authentication-only with all authenticated users having full access, with role-based enforcement added in a subsequent phase.

## External Dependencies

### JSONPlaceholder API

External service endpoint at https://jsonplaceholder.typicode.com/todos used as the source of initial todo data. This is a free mock API that returns sample todo records. The backend fetches exactly 25 todos from this endpoint on the first GET request and caches them in memory. No authentication or API key is required. The service resets its data on server restart but provides consistent responses during a session.

### Keycloak

Open-source identity and access management solution to be deployed as a Docker Compose service. Keycloak will provide user authentication through OpenID Connect and JWT token issuance. The FastAPI backend will depend on Keycloak for token validation, requiring configuration of the realm URL, client ID, and client secret. For local development, Keycloak will run in the same Docker network as the backend service. Keycloak database persistence may be configured through volumes or init scripts for realm and client setup.

### python-keycloak

Python client library for interacting with Keycloak API. This dependency will be used by the FastAPI backend for token validation and potentially for realm and client management. The library provides methods for verifying JWT signatures, decoding token payloads, and checking token expiration. Alternative approaches include direct JWT validation using the jose library or cryptography for signature verification.

### httpx

Async HTTP client library for Python to replace the fetch API used in Express. This library will be used to make asynchronous requests to the JSONPlaceholder endpoint for fetching remote todos. httpx provides async/await support, connection pooling, and better error handling compared to synchronous requests. The library supports both HTTP/1.1 and HTTP/2 with automatic protocol negotiation.

### FastAPI

Modern Python web framework for building APIs with automatic type validation and OpenAPI documentation generation. This framework replaces Express.js as the backend web server. FastAPI provides built-in request validation, dependency injection for middleware, automatic API documentation, and high performance through asynchronous request handling. The framework will serve as the core dependency for all backend logic.

### Pydantic v2

Data validation library integrated with FastAPI for defining request and response models with type safety. Pydantic models will replace plain JavaScript objects for all API data structures. The library provides runtime type checking, automatic data conversion, validation error generation, and JSON schema generation. All API request bodies and response models will be defined as Pydantic BaseModel subclasses.

### FastAPI CORSMiddleware

Middleware component for configuring CORS policies in FastAPI applications. This will replace the Express cors middleware to maintain the existing all-origins CORS configuration. The middleware will be configured to allow all origins, all methods, all headers, and allow credentials to match the current Express implementation.

### nginx

Web server used to serve the React frontend and proxy API requests to the backend. The nginx configuration at `/frontend/nginx.conf` will remain unchanged, continuing to proxy `/api/` paths to the backend service on port 3001. nginx version 1.27-alpine will continue as the frontend production web server.

### Node.js 20 and npm

Runtime and package manager for the frontend build process only. The frontend development and build will continue using Node.js 20 with npm for dependency management. The Vite build tool will remain the frontend bundler. Node.js is not used in the backend after modernization.

### Docker and Docker Compose

Container orchestration tools for multi-service deployment. The existing Docker Compose configuration at `/docker-compose.yml` will be extended to include the Keycloak service. Docker will continue to be used for building images for the FastAPI backend, React frontend, and Keycloak. The backend Dockerfile will be rewritten from node:20-alpine to a Python base image.

### Vite

Build tool and development server for the React frontend. Vite will continue to be used for frontend development and production builds. The configuration at `/frontend/vite.config.js` will remain unchanged. Vite provides fast HMR for development and optimized builds for production.
