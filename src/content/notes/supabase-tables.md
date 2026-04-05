---
title: "Supabase Tables"
description: "A quick reference for creating tables, relationships, and constraints in Supabase/PostgreSQL"
tags: ["supabase", "sql", "postgres", "database"]
updated: "2026-04-02"
coAuthor: "opencode"
---

# Supabase Tables

This guide covers table creation, relationship types, and constraints for Supabase projects using PostgreSQL.

---

## Creating Tables

```sql
-- Basic table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT UNIQUE NOT NULL,
  name TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

## Relationships

### One-to-Many (has many)

```sql
-- users -> posts (one user has many posts)
CREATE TABLE posts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  title TEXT NOT NULL,
  content TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### Many-to-Many (belongs to many)

```sql
-- posts <-> tags (many-to-many via junction table)
CREATE TABLE tags (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT UNIQUE NOT NULL
);

CREATE TABLE post_tags (
  post_id UUID REFERENCES posts(id) ON DELETE CASCADE,
  tag_id UUID REFERENCES tags(id) ON DELETE CASCADE,
  PRIMARY KEY (post_id, tag_id)
);
```

### One-to-One

```sql
-- users -> profiles (one-to-one)
CREATE TABLE profiles (
  user_id UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
  bio TEXT,
  avatar_url TEXT,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

## Constraints

### Primary Key

```sql
-- Single column
id UUID PRIMARY KEY

-- Composite primary key
PRIMARY KEY (user_id, role_id)
```

### Foreign Key

```sql
-- Basic
user_id UUID REFERENCES users(id)

-- With actions
user_id UUID REFERENCES users(id) ON DELETE CASCADE
user_id UUID REFERENCES users(id) ON DELETE SET NULL
user_id UUID REFERENCES users(id) ON UPDATE CASCADE
```

### Unique

```sql
-- Column level
email TEXT UNIQUE

-- Table level
UNIQUE (email, organization_id)
```

### Check

```sql
-- Ensure value is positive
price NUMERIC CHECK (price > 0)

-- Multiple conditions
status TEXT CHECK (status IN ('active', 'inactive', 'pending'))
```

### Not Null

```sql
name TEXT NOT NULL
email TEXT NOT NULL
```

### Default

```sql
created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
is_active BOOLEAN DEFAULT true
priority INTEGER DEFAULT 1
```

### Additional Examples

```sql
-- RLS (Row Level Security) - Supabase specific
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view their own data"
  ON users FOR SELECT
  USING (auth.uid() = id);

-- Soft delete
ALTER TABLE posts ADD COLUMN deleted_at TIMESTAMP WITH TIME ZONE;

-- Enums
CREATE TYPE post_status AS ENUM ('draft', 'published', 'archived');

ALTER TABLE posts ADD COLUMN status post_status DEFAULT 'draft';
```

## Quick Reference

| Relationship | Implementation                   |
| ------------ | -------------------------------- |
| One-to-Many  | `FK` on child table              |
| Many-to-Many | Junction table with composite PK |
| One-to-One   | `FK` with unique constraint      |

---

_Last updated: 2026-04-02_
