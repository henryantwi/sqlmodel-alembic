# Getting Started with Alembic and SQLModel

This README provides a comprehensive guide to setting up and using Alembic with SQLModel for efficient database management and migrations in Python applications.

## Overview

**SQLModel** is a modern Python library that combines the power of SQLAlchemy with Pydantic, providing type hints and data validation for database models. **Alembic** is SQLAlchemy's database migration tool that allows you to manage database schema changes over time.

Together, they offer a streamlined approach to:
- Define database models with type safety
- Handle database migrations automatically  
- Maintain database schema version control

## Prerequisites

- Python 3.7+
- Basic understanding of Python and databases
- Familiarity with SQLAlchemy concepts (helpful but not required)

## Installation and Setup

### Step 1: Environment Setup

Create and activate a virtual environment:

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

Install required packages:

```bash
pip install sqlmodel alembic
```

### Step 2: Define SQLModel Models

Create your database models using SQLModel classes. Here's an example model:

```python
# models.py
from sqlmodel import SQLModel, Field

class Book(SQLModel, table=True):
    id: int = Field(default=None, primary_key=True)
    title: str
    author: str
```

Key features of SQLModel:
- **Type hints**: Full Python type annotation support
- **Automatic validation**: Built-in data validation via Pydantic
- **Table definition**: `table=True` parameter marks it as a database table
- **Field configuration**: `Field()` allows primary keys, defaults, and constraints

## Alembic Configuration

### Step 3: Initialize Alembic

Initialize Alembic in your project directory:

```bash
alembic init migrations
```

This creates:
- `alembic.ini` - Main configuration file
- `migrations/` directory with Alembic files
- `migrations/env.py` - Migration environment configuration

### Step 4: Configure Database Connection

Edit `alembic.ini` to set your database URL:

```ini
# alembic.ini
sqlalchemy.url = sqlite:///database.db
```

For other databases:
```ini
# PostgreSQL
sqlalchemy.url = postgresql://user:password@localhost/dbname

# MySQL  
sqlalchemy.url = mysql://user:password@localhost/dbname
```

### Step 5: Configure Migration Environment

Modify `migrations/env.py` to include your SQLModel models:

```python
# migrations/env.py
from sqlmodel import SQLModel
from alembic import context
from models import *  # Import your SQLModel models here

target_metadata = SQLModel.metadata

# Rest of the file remains unchanged
```

### Step 6: Update Migration Template

Edit `migrations/script.py.mako` to include SQLModel import:

```python
# migrations/script.py.mako
import sqlmodel
# ... rest of the template
```

## Working with Migrations

### Creating Migrations

Generate a new migration after modifying your models:

```bash
alembic revision --autogenerate -m "Add book table"
```

This command:
- Compares your current models with the database schema
- Generates a migration script with necessary changes
- Creates a new file in `migrations/versions/`

### Applying Migrations

Apply migrations to update your database:

```bash
# Upgrade to latest migration
alembic upgrade head

# Upgrade to specific revision
alembic upgrade <revision_id>

# Downgrade one revision
alembic downgrade -1
```

### Migration Commands Reference

```bash
# Show current revision
alembic current

# Show migration history
alembic history

# Show pending migrations
alembic show <revision_id>

# Create empty migration (for manual changes)
alembic revision -m "Custom migration"
```

## Testing Your Setup

Create a test script to verify everything works:

```python
# test_setup.py
from sqlmodel import Session, create_engine
from models import Book

# Create engine and tables
engine = create_engine('sqlite:///database.db')
SQLModel.metadata.create_all(engine)

# Test database operations
with Session(engine) as session:
    # Create a new book
    book = Book(title="Sample Book", author="Author Name")
    session.add(book)
    session.commit()
    
    # Query the book
    retrieved_book = session.get(Book, book.id)
    print(f"Book: {retrieved_book.title} by {retrieved_book.author}")
```

## Best Practices

### Model Design
- Use type hints consistently for better IDE support and validation
- Define relationships using `Relationship()` for complex models
- Use `Field()` for advanced column configurations

### Migration Management
- Always review auto-generated migrations before applying
- Use descriptive migration messages
- Test migrations on development data before production
- Keep migrations small and focused on single changes

### Project Structure
```
project/
├── models.py          # SQLModel definitions
├── database.py        # Database connection and session management
├── alembic.ini       # Alembic configuration
├── migrations/       # Migration files
│   ├── env.py
│   └── versions/
└── main.py          # Application entry point
```

## Advanced Features

### Complex Models with Relationships

```python
from sqlmodel import SQLModel, Field, Relationship
from typing import Optional, List

class Author(SQLModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    name: str
    books: List["Book"] = Relationship(back_populates="author")

class Book(SQLModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    title: str
    author_id: Optional[int] = Field(default=None, foreign_key="author.id")
    author: Optional[Author] = Relationship(back_populates="books")
```

### Environment-Specific Configurations

Use environment variables for different deployment stages:

```python
# database.py
import os
from sqlmodel import create_engine

DATABASE_URL = os.getenv("DATABASE_URL", "sqlite:///./database.db")
engine = create_engine(DATABASE_URL)
```

## Troubleshooting

### Common Issues

1. **Import errors in migrations**: Ensure all models are imported in `env.py`
2. **Alembic can't detect changes**: Verify `target_metadata = SQLModel.metadata`
3. **Database connection issues**: Check database URL format and credentials
4. **Migration conflicts**: Use `alembic merge` for conflicting revisions

### Debug Mode

Enable SQL logging to debug issues:

```python
engine = create_engine("sqlite:///database.db", echo=True)
```

## Resources

- [SQLModel Documentation](https://sqlmodel.tiangolo.com/)
- [Alembic Documentation](https://alembic.sqlalchemy.org/)
- [SQLAlchemy Documentation](https://docs.sqlalchemy.org/)
- [Pydantic Documentation](https://pydantic-docs.helpmanual.io/)

## License

This guide is based on the original article by Kasper Juunge and is intended for educational purposes.
