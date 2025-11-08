# SecureAPI-Py Quick Reference

## Import

```python
from secure_api import (
    SecureAPI,
    Router,
    log_request,
    auth_middleware,
    cors_middleware,
    rate_limit_middleware,
    Validator,
    EnhancedValidator,
    ValidationError,
    ApiError
)
```

## Basic Setup

```python
async def main(context):
    api = SecureAPI(context, database_id='YOUR_DB_ID')
    
    # Add middleware
    api.use_middleware(log_request)
    await api.execute_middleware()
    
    # Your logic here
    return await api.send_success(message='Success')
```

## Request Properties

```python
api.method              # HTTP method (GET, POST, etc.)
api.headers             # Dict of headers
api.query_params        # Dict of query parameters
api.body_text           # Request body as string
api.body_json           # Request body as dict
api.path                # Request path
api.url                 # Full URL
api.host                # Host name
api.user_id             # Authenticated user ID (or None)
api.user_jwt            # JWT token (or None)
api.is_authenticated    # Boolean
api.trigger_type        # 'http', 'schedule', or 'event'
api.trigger_event       # Event name (if event-triggered)
```

## Response Methods

```python
# Success response
await api.send_success(
    message='Success',
    data={'key': 'value'},
    status_code=200
)

# Error response
await api.send_error(
    message='Error occurred',
    status_code=500
)

# Other responses
await api.send_unauthorized(message='Not authorized')
await api.send_empty()  # 204 No Content
await api.send_text('Plain text')
await api.send_redirect('https://example.com')
await api.send_binary(bytes_data)
```

## Logging

```python
api.log('Simple message')
api.error('Error message')
api.log_json('Label', {'key': 'value'})
api.log({'auto': 'formatted'})  # Pretty-prints dicts/lists
```

## Database Operations

```python
# Create document
doc = await api.db.create_document(
    collection_id='tasks',
    data={'title': 'Task 1'},
    database_id='optional_db_id'  # Uses default if not provided
)

# List documents
docs = await api.db.list_documents(
    collection_id='tasks',
    queries=['limit(10)']
)

# Get document
doc = await api.db.get_document(
    collection_id='tasks',
    document_id='doc_id'
)

# Update document
doc = await api.db.update_document(
    collection_id='tasks',
    document_id='doc_id',
    data={'title': 'Updated'}
)

# Delete document
await api.db.delete_document(
    collection_id='tasks',
    document_id='doc_id'
)
```

## Validation

### Basic Validation
```python
from secure_api import Validator

Validator.validate_headers(api.headers, ['x-api-key'])
Validator.validate_query_params(api.query_params, ['type'])
Validator.validate_body(api.body_json, ['title', 'description'])
```

### Enhanced Validation
```python
api.validate_body({
    'title': 'required|string|min:3|max:100',
    'email': 'required|email',
    'age': 'required|int|min:18|max:120',
    'priority': 'in:low,medium,high',
    'url': 'url',
    'uuid': 'uuid'
})
```

### Custom Validation
```python
api.validate_custom({
    'custom_field': lambda v: v is not None and len(v) > 5
})
```

## Router

```python
router = Router()

# Define routes
router.get('/tasks', list_tasks)
router.post('/tasks', create_task)
router.get('/tasks/:id', get_task)
router.put('/tasks/:id', update_task)
router.delete('/tasks/:id', delete_task)

# Handle request
return await router.handle(api)

# Route handler
async def get_task(api, params):
    task_id = params['id']  # Access route parameter
    # Your logic
    return await api.send_success(data={'id': task_id})
```

## Middleware

### Built-in Middleware
```python
# CORS
api.use_middleware(cors_middleware(
    origin='*',
    methods='GET, POST, PUT, DELETE',
    headers='Content-Type, Authorization'
))

# Rate Limiting
api.use_middleware(rate_limit_middleware(
    max_requests=100,
    window_minutes=1
))

# Authentication
api.use_middleware(auth_middleware)

# Request Logging
api.use_middleware(log_request)
```

### Custom Middleware
```python
async def custom_middleware(api):
    api.log('Custom middleware executed')
    # Your logic here
    # Raise exception to stop request processing

api.use_middleware(custom_middleware)
```

## Context Storage

```python
# Set context data
api.set_context('user_role', 'admin')

# Get context data
role = api.get_context('user_role')

# Check if exists
if api.has_context('user_role'):
    # ...

# Clear all context
api.clear_context()
```

## Performance Monitoring

```python
# Start timer
api.start_timer('operation_name')

# Your code here

# End timer (logs duration automatically)
duration = api.end_timer('operation_name')

# Get timer result
duration = api.get_timer_result('operation_name')

# Log all timers
api.log_timer_summary()
```

## Error Handling

```python
from secure_api import ValidationError, ApiError

try:
    # Your code
    pass
except ValidationError as e:
    return await api.handle_error(e)
except ApiError as e:
    return await api.handle_error(e)
except Exception as e:
    api.error(f'Unexpected error: {e}')
    return await api.send_error(message='Internal server error')
```

## Environment Variables

```python
# Get with default
db_id = api.get_env('DATABASE_ID', default_value='default_db')

# Get without default
api_key = api.get_env('API_KEY')
```

## Complete Example

```python
from secure_api import SecureAPI, Router, log_request, ValidationError

async def main(context):
    api = SecureAPI(context, database_id='tasks_db')
    api.use_middleware(log_request)
    
    router = Router()
    router.post('/tasks', create_task)
    router.get('/tasks', list_tasks)
    
    try:
        await api.execute_middleware()
        return await router.handle(api)
    except ValidationError as e:
        return await api.handle_error(e)
    except Exception as e:
        api.error(f'Error: {e}')
        return await api.send_error(message=str(e))

async def create_task(api, params):
    api.validate_body({
        'title': 'required|string|min:3',
        'description': 'required|string'
    })
    
    task = await api.db.create_document(
        collection_id='tasks',
        data=api.body_json
    )
    
    return await api.send_success(
        message='Task created',
        data={'task': task},
        status_code=201
    )

async def list_tasks(api, params):
    tasks = await api.db.list_documents(collection_id='tasks')
    return await api.send_success(data={'tasks': tasks})
```

## Validation Rules Reference

| Rule | Description | Example |
|------|-------------|---------|
| `required` | Field must be present and non-empty | `'required'` |
| `string` | Must be a string | `'string'` |
| `int`, `integer`, `number` | Must be a number | `'int'` |
| `bool`, `boolean` | Must be a boolean | `'bool'` |
| `array`, `list` | Must be an array | `'array'` |
| `map`, `object`, `dict` | Must be an object | `'object'` |
| `email` | Must be valid email | `'email'` |
| `url` | Must be valid URL | `'url'` |
| `uuid` | Must be valid UUID | `'uuid'` |
| `min:n` | Minimum length/value | `'min:3'` |
| `max:n` | Maximum length/value | `'max:100'` |
| `in:a,b,c` | Must be one of values | `'in:low,medium,high'` |
| `regex:pattern` | Must match regex | `'regex:^[A-Z]'` |

Combine rules with pipe: `'required|string|min:3|max:100'`
