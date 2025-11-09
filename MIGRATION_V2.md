# Migration Guide: v1.x to v2.0.0

## Overview

Version 2.0.0 introduces a **breaking change** by migrating from Appwrite's deprecated `Databases` service to the new `TablesDB` service. This migration aligns with Appwrite's latest API changes.

## What Changed?

### Service Migration
- **Old**: `from appwrite.services.databases import Databases`
- **New**: `from appwrite.services.tables_db import TablesDB`

### Terminology Changes

Appwrite has moved from "collections and documents" to "tables and rows":

| Old (v1.x) | New (v2.0.0) |
|------------|--------------|
| Collection | Table |
| Document | Row |
| `collection_id` | `table_id` |
| `document_id` | `row_id` |
| `document_security` | `row_security` |
| `attributes` | `columns` |

### API Method Changes

The `DatabaseManager` class maintains backward-compatible method names, but internally uses the new TablesDB API:

| Method | Old Internal Call | New Internal Call |
|--------|------------------|-------------------|
| `create_collection()` | `databases.create_collection()` | `tables_db.create_table()` |
| `list_collections()` | `databases.list_collections()` | `tables_db.list_tables()` |
| `get_collection()` | `databases.get_collection()` | `tables_db.get_table()` |
| `update_collection()` | `databases.update_collection()` | `tables_db.update_table()` |
| `delete_collection()` | `databases.delete_collection()` | `tables_db.delete_table()` |
| `create_document()` | `databases.create_document()` | `tables_db.create_row()` |
| `list_documents()` | `databases.list_documents()` | `tables_db.list_rows()` |
| `get_document()` | `databases.get_document()` | `tables_db.get_row()` |
| `update_document()` | `databases.update_document()` | `tables_db.update_row()` |
| `delete_document()` | `databases.delete_document()` | `tables_db.delete_row()` |

## Do I Need to Change My Code?

### ✅ **NO CHANGES REQUIRED** if you're using SecureAPI's DatabaseManager

The `DatabaseManager` class maintains the same method signatures. Your existing code will continue to work:

```python
# This still works in v2.0.0!
from secure_api import SecureAPI

async def main(context):
    api = SecureAPI(context, database_id='my_db')
    
    # Create a document (internally creates a row)
    doc = await api.db.create_document(
        collection_id='tasks',
        data={'title': 'My Task'}
    )
    
    # List documents (internally lists rows)
    docs = await api.db.list_documents(collection_id='tasks')
```

### ⚠️ **CHANGES REQUIRED** if you're using Appwrite SDK directly

If you were importing and using Appwrite's `Databases` service directly, you need to update:

**Before (v1.x):**
```python
from appwrite.services.databases import Databases

databases = Databases(client)
doc = databases.create_document(
    database_id='db',
    collection_id='tasks',
    document_id='123',
    data={'title': 'Task'}
)
```

**After (v2.0.0):**
```python
from appwrite.services.tables_db import TablesDB

tables_db = TablesDB(client)
row = tables_db.create_row(
    database_id='db',
    table_id='tasks',
    row_id='123',
    data={'title': 'Task'}
)
```

## Parameter Changes

### Collection/Table Creation

**Before:**
```python
api.db.create_collection(
    name='Tasks',
    collection_id='tasks',
    document_security=True  # Old parameter
)
```

**After:**
```python
api.db.create_collection(
    name='Tasks',
    collection_id='tasks',
    document_security=True  # Still works! Mapped to row_security internally
)
```

### Index Creation

**Before:**
```python
api.db.create_index(
    collection_id='tasks',
    key='title_idx',
    type='key',
    attributes=['title']  # Old parameter name
)
```

**After:**
```python
api.db.create_index(
    collection_id='tasks',
    key='title_idx',
    type='key',
    attributes=['title']  # Still works! Mapped to columns internally
)
```

## Response Object Changes

The response objects from TablesDB use the new terminology:

**Before (v1.x):**
```python
{
    "$id": "123",
    "$collectionId": "tasks",
    "$databaseId": "db",
    "title": "My Task"
}
```

**After (v2.0.0):**
```python
{
    "_id": "123",
    "_tableId": "tasks",
    "_databaseId": "db",
    "data": {
        "title": "My Task"
    }
}
```

**Note:** The data structure has changed - row data is now nested under a `data` property.

## Accessing Row Data

### Before (v1.x)
```python
doc = await api.db.get_document(
    collection_id='tasks',
    document_id='123'
)
title = doc['title']  # Direct access
```

### After (v2.0.0)
```python
row = await api.db.get_document(
    collection_id='tasks',
    document_id='123'
)
title = row['data']['title']  # Access via 'data' property
# Or access metadata
row_id = row['_id']
table_id = row['_tableId']
```

## Testing Your Migration

1. **Update the package:**
   ```bash
   pip install --upgrade secure-api-py
   ```

2. **Test your existing code:**
   - Method calls should work without changes
   - Update code that accesses response properties

3. **Check response handling:**
   - Update any code that directly accesses document fields
   - Use `row['data']['field']` instead of `row['field']`

## Common Issues

### Issue: "AttributeError: 'TablesDB' object has no attribute 'create_collection'"

**Cause:** You're trying to use old Databases methods directly on TablesDB.

**Solution:** Use SecureAPI's DatabaseManager which provides backward-compatible methods.

### Issue: "KeyError: 'title'" when accessing row data

**Cause:** Row data is now nested under the `data` property.

**Solution:** 
```python
# Old
title = row['title']

# New
title = row['data']['title']
```

### Issue: Response has `_id` instead of `$id`

**Cause:** TablesDB uses underscore-prefixed metadata fields.

**Solution:** Update your code to use the new field names:
- `$id` → `_id`
- `$collectionId` → `_tableId`
- `$databaseId` → `_databaseId`
- `$createdAt` → `_createdAt`
- `$updatedAt` → `_updatedAt`

## Rollback

If you need to rollback to v1.x:

```bash
pip install secure-api-py==1.0.1
```

## Support

If you encounter issues during migration:
1. Check the [CHANGELOG.md](CHANGELOG.md)
2. Review the [README.md](README.md) for updated examples
3. Open an issue on [GitHub](https://github.com/mj-963/secure_api_py/issues)

## Summary

✅ **Good News:** Most code continues to work without changes
⚠️ **Action Required:** Update response object access patterns
📚 **Benefit:** Using Appwrite's latest, non-deprecated API
