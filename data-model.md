# Data Model

## Entities

### Todo
- **Fields:**
  - userId: number (represents the user who owns the todo, always set to 1 in the current implementation)
  - id: number (unique identifier for the todo, numeric ascending order, auto-generated)
  - title: string (task description text, required field, must be non-empty after trimming)
  - completed: boolean (task completion status, defaults to false if not provided)
  - isLocal: boolean (flag indicating whether the todo was created locally, only present for locally added todos)
- **Relationships:**
  - None (independent entity)
- **Source:** /Users/puttaiaharugunta/codebase/infi/todo-docker-app/backend/server.js (lines 62-68)

### HealthResponse
- **Fields:**
  - status: string (health status of the backend service, always returns 'ok')
- **Relationships:**
  - None
- **Source:** /Users/puttaiaharugunta/codebase/infi/todo-docker-app/backend/server.js (line 35)

### ErrorResponse
- **Fields:**
  - message: string (human-readable error description for API failures)
- **Relationships:**
  - None
- **Source:** /Users/puttaiaharugunta/codebase/infi/todo-docker-app/backend/server.js (lines 43, 52, 73, 82, 94)

### DeleteResponse
- **Fields:**
  - message: string (confirmation message, always 'Todo deleted successfully')
  - id: number (the ID of the deleted todo)
- **Relationships:**
  - None
- **Source:** /Users/puttaiaharugunta/codebase/infi/todo-docker-app/backend/server.js (line 92)

### TodoCreateRequest
- **Fields:**
  - title: string (required field for todo title, must be non-empty after trimming)
  - completed: boolean (optional field, defaults to false if not provided)
- **Relationships:**
  - Creates a new Todo entity
- **Source:** /Users/puttaiaharugunta/codebase/infi/todo-docker-app/backend/server.js (lines 47-49)

## Relationships Diagram

The data model follows a simple in-memory storage pattern with three primary collections and no persistent relational relationships:

1. **cachedRemoteTodos** holds the first 25 todos fetched from JSONPlaceholder (external API source)
2. **localAddedTodos** stores all Todo entities created through the POST endpoint
3. **deletedTodoIds** is a Set of numeric IDs tracking soft-deleted remote todos

**Data Flow:**
- Remote todos are fetched from JSONPlaceholder and cached in cachedRemoteTodos
- Local todos are created through POST /api/todos and stored in localAddedTodos
- DELETE operations either remove from localAddedTodos (if local) or add ID to deletedTodoIds Set (if remote)
- GET /api/todos merges cachedRemoteTodos and localAddedTodos, filters by deletedTodoIds, and sorts by id

**Soft-Delete Mechanism:**
- Local todos: permanent removal from localAddedTodos array
- Remote todos: ID added to deletedTodoIds Set for filtering in subsequent GET requests

## Migration Notes

### In-Memory Storage Preservation
- The new FastAPI implementation must maintain the exact same in-memory data structures
- No database migration is required as the original implementation uses in-memory storage
- State must be preserved across requests within the same server lifecycle
- State resets on server restart (same behavior as Express implementation)

### Data Structure Mapping (Express to FastAPI)

| Express (JavaScript) | FastAPI (Python) | Notes |
|---------------------|-------------------|-------|
| cachedRemoteTodos: null | cached_remote_todos: Optional[List[Todo]] | Initialize as None, type from typing module |
| localAddedTodos: [] | local_added_todos: List[Todo] | Empty list initialization |
| deletedTodoIds: new Set() | deleted_todo_ids: Set[int] | Set from builtins module |

### Field Type Conversions
- **userId**: JavaScript number to Python int
- **id**: JavaScript number to Python int (must handle numeric path parameters)
- **title**: JavaScript string to Python str (Pydantic validation for non-empty after trim)
- **completed**: JavaScript boolean to Python bool (default to False)
- **isLocal**: JavaScript boolean to Python bool (only for local todos)

### Business Logic Constraints to Preserve

1. **ID Generation Logic** (source: server.js lines 56-60)
   - Find maximum numeric id across both cached_remote_todos and local_added_todos
   - Add 1 to maximum to generate new unique id
   - Handle edge case where no todos exist (start at 1)

2. **Soft-Delete Behavior** (source: server.js lines 85-90)
   - Check if todo id exists in local_added_todos
   - If found: remove directly from local_added_todos list
   - If not found: add numeric id to deleted_todo_ids Set

3. **Title Validation** (source: server.js lines 51-53)
   - Reject null, undefined, empty string, or whitespace-only titles
   - Return HTTP 400 with message 'Title is required'
   - Trim whitespace from valid titles before storage

4. **Merging and Sorting** (source: server.js lines 28-32)
   - Concatenate cached_remote_todos and local_added_todos
   - Filter out any todo.id present in deleted_todo_ids Set
   - Sort final result by numeric id in ascending order

5. **JSONPlaceholder Caching** (source: server.js lines 15-26)
   - Fetch only once on first GET /api/todos request
   - Cache exactly first 25 todos from response
   - All subsequent requests use cached data without external API calls

### Pydantic Model Implementation Requirements

**TodoResponse Model:**
- All fields required for remote todos (userId, id, title, completed)
- isLocal field optional (only present on local todos)
- Field validators for title: non-empty after string trim
- Type validators: userId and id as integers, completed and isLocal as booleans

**TodoCreateRequest Model:**
- title: required string with validator for non-empty after trim
- completed: optional boolean with default value False
- No id field (auto-generated by backend)
- No userId field (defaults to 1)
- No isLocal field (automatically set to True)

**HealthResponse Model:**
- status: required string with fixed value 'ok'

**ErrorResponse Model:**
- message: required string for error description

**DeleteResponse Model:**
- message: required string with fixed value 'Todo deleted successfully'
- id: required integer matching deleted todo ID

### Thread Safety Considerations
- Express implementation uses single-threaded event loop (implicit thread safety)
- FastAPI async implementation must use threading.Lock or asyncio.Lock for concurrent requests
- All state modification operations must be protected by locks to prevent race conditions
- StateManager class should encapsulate locking logic for all state mutations

### API Response Format Compatibility
- GET /health must return JSON object with status field exactly as Express
- GET /api/todos must return array of Todo objects with identical field names (camelCase)
- POST /api/todos must return created Todo object with all fields including isLocal
- DELETE /api/todos/:id must return JSON object with message and id fields
- All error responses must return JSON object with message field

### External API Integration Constraints
- JSONPlaceholder URL: https://jsonplaceholder.typicode.com/todos
- Must fetch exactly first 25 todos (use slice operation on response array)
- Cache in memory after first successful fetch
- Handle network failures gracefully with 500 error and descriptive message

### No Database Migration Required
- Original implementation uses in-memory storage only
- No persistent data migration necessary
- New implementation will maintain same in-memory pattern
- Future enhancement may add database, but out of scope for this modernization

### CORS Preservation
- Express uses cors() middleware without parameters (allows all origins)
- FastAPI CORSMiddleware must be configured with:
  - allow_origins=['*'] or allow all origins
  - allow_methods=['*'] or allow all methods
  - allow_headers=['*'] or allow all headers
  - allow_credentials=True to match Express default behavior

### Environment Variable Configuration
- PORT: defaults to 3001 if not set (same as Express)
- JSON_PLACEHOLDER_URL: configurable but defaults to https://jsonplaceholder.typicode.com/todos
- Keycloak settings (new): KEYCLOAK_URL, KEYCLOAK_REALM, KEYCLOAK_CLIENT_ID, KEYCLOAK_CLIENT_SECRET
