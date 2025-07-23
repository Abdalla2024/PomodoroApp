# Pomodoro Timer App - Data Models & Database Schema

## 1. Database Architecture

### 1.1 Database Choice
**SQLite** - Local-first approach for offline functionality and privacy
- Embedded database requiring no server setup
- ACID compliance for data integrity
- Cross-platform compatibility
- Efficient for read-heavy analytics queries

### 1.2 Schema Design Principles
- **Normalization**: Proper relationships to avoid data duplication
- **Indexing**: Optimized queries for analytics and session retrieval
- **Constraints**: Data integrity through foreign keys and check constraints
- **Versioning**: Migration system for schema updates

## 2. Core Data Models

### 2.1 User Entity
```sql
CREATE TABLE users (
    id TEXT PRIMARY KEY DEFAULT (hex(randomblob(16))),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    version INTEGER DEFAULT 1
);

-- User settings stored separately for easier updates
CREATE TABLE user_settings (
    user_id TEXT PRIMARY KEY,
    notification_sound BOOLEAN DEFAULT true,
    vibration BOOLEAN DEFAULT true,
    visual_alerts BOOLEAN DEFAULT true,
    default_focus_time INTEGER DEFAULT 25, -- minutes
    default_break_time INTEGER DEFAULT 5,  -- minutes
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

### 2.2 Task Management System

#### 2.2.1 Tasks Table
```sql
CREATE TABLE tasks (
    id TEXT PRIMARY KEY DEFAULT (hex(randomblob(16))),
    user_id TEXT NOT NULL,
    title TEXT NOT NULL CHECK (length(title) > 0 AND length(title) <= 200),
    description TEXT CHECK (length(description) <= 1000),
    status TEXT DEFAULT 'todo' CHECK (status IN ('todo', 'in_progress', 'done')),
    priority INTEGER DEFAULT 0 CHECK (priority BETWEEN 0 AND 3), -- 0=low, 3=high
    tag TEXT CHECK (length(tag) <= 50), -- e.g. 'Work', 'Personal'
    total_focus_time INTEGER DEFAULT 0, -- total minutes focused on this task
    estimated_time INTEGER, -- estimated completion time in minutes
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    completed_at DATETIME,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Index for efficient task retrieval by user and status
CREATE INDEX idx_tasks_user_status ON tasks(user_id, status);
CREATE INDEX idx_tasks_completed_at ON tasks(completed_at) WHERE completed_at IS NOT NULL;
```

#### 2.2.2 Tags System
```sql
CREATE TABLE tags (
    id TEXT PRIMARY KEY DEFAULT (hex(randomblob(16))),
    user_id TEXT NOT NULL,
    name TEXT NOT NULL CHECK (length(name) > 0 AND length(name) <= 50),
   -- color TEXT DEFAULT '#6366f1' CHECK (length(color) = 7 AND color LIKE '#%'),
    is_custom BOOLEAN DEFAULT true,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    UNIQUE(user_id, name) -- Prevent duplicate tag names per user
);

-- Predefined tags for new users
INSERT INTO tags (id, user_id, name, color, is_custom) VALUES
    ('work_tag', 'default', 'Work', '#ef4444', false),
    ('study_tag', 'default', 'Study', '#3b82f6', false),
    ('personal_tag', 'default', 'Personal', '#10b981', false),
    ('health_tag', 'default', 'Health', '#f59e0b', false),
    ('creative_tag', 'default', 'Creative', '#8b5cf6', false);
```

### 2.3 Timer and Session Management

#### 2.3.1 Sessions Table
```sql
CREATE TABLE sessions (
    id TEXT PRIMARY KEY DEFAULT (hex(randomblob(16))),
    user_id TEXT NOT NULL,
    task_id TEXT,
    session_type TEXT DEFAULT 'focus' CHECK (session_type IN ('focus', 'break')),
    planned_duration INTEGER NOT NULL CHECK (planned_duration > 0), -- minutes
    actual_duration INTEGER, -- actual completed minutes
    start_time DATETIME NOT NULL,
    end_time DATETIME,
    completed BOOLEAN DEFAULT false,
    bird_unlocked_id TEXT, -- reference to unlocked bird, if any
    interruptions INTEGER DEFAULT 0, -- count of pauses during session
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (task_id) REFERENCES tasks(id) ON DELETE SET NULL,
    FOREIGN KEY (bird_unlocked_id) REFERENCES birds(id)
);

-- Indexes for analytics queries
CREATE INDEX idx_sessions_user_date ON sessions(user_id, start_time);
CREATE INDEX idx_sessions_task ON sessions(task_id) WHERE task_id IS NOT NULL;
CREATE INDEX idx_sessions_completed ON sessions(user_id, completed, start_time);
```

### 2.4 Bird Collection System

#### 2.4.1 Birds Master Data
```sql
CREATE TABLE birds (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL UNIQUE CHECK (length(name) > 0 AND length(name) <= 50),
    description TEXT CHECK (length(description) <= 500),
    rarity TEXT NOT NULL CHECK (rarity IN ('common', 'rare', 'legendary')),
    image_path TEXT NOT NULL,
    unlock_probability REAL NOT NULL CHECK (unlock_probability > 0 AND unlock_probability <= 1),
    sort_order INTEGER NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Predefined birds data
INSERT INTO birds (id, name, description, rarity, image_path, unlock_probability, sort_order) VALUES
    ('robin', 'Robin', 'A cheerful red-breasted bird that loves early mornings.', 'common', 'birds/robin.png', 0.35, 1),
    ('sparrow', 'Sparrow', 'A social bird that represents consistency and community.', 'common', 'birds/sparrow.png', 0.25, 2),
    ('bluebird', 'Bluebird', 'A symbol of happiness and prosperity.', 'rare', 'birds/bluebird.png', 0.20, 3),
    ('cardinal', 'Cardinal', 'A vibrant red bird representing focus and determination.', 'rare', 'birds/cardinal.png', 0.10, 4),
    ('owl', 'Wise Owl', 'A nocturnal scholar representing wisdom and deep thinking.', 'rare', 'birds/owl.png', 0.08, 5),
    ('phoenix', 'Phoenix', 'A legendary bird of rebirth and transformation.', 'legendary', 'birds/phoenix.png', 0.02, 6),
    ('peacock', 'Peacock', 'A magnificent bird displaying beauty and pride.', 'legendary', 'birds/peacock.png', 0.005, 7),
    ('dragon', 'Sky Dragon', 'A mythical creature representing ultimate achievement.', 'legendary', 'birds/dragon.png', 0.002, 8);
```

#### 2.4.2 User Bird Collection
```sql
CREATE TABLE user_birds (
    user_id TEXT NOT NULL,
    bird_id TEXT NOT NULL,
    unlocked_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    unlock_session_id TEXT, -- which session unlocked this bird
    PRIMARY KEY (user_id, bird_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (bird_id) REFERENCES birds(id),
    FOREIGN KEY (unlock_session_id) REFERENCES sessions(id)
);

-- Index for collection analytics
CREATE INDEX idx_user_birds_unlocked ON user_birds(user_id, unlocked_at);
```

### 2.5 Analytics and Statistics

#### 2.5.1 Daily Statistics (Aggregated Data)
```sql
CREATE TABLE daily_stats (
    user_id TEXT NOT NULL,
    date DATE NOT NULL,
    total_focus_time INTEGER DEFAULT 0, -- minutes
    total_break_time INTEGER DEFAULT 0, -- minutes
    sessions_completed INTEGER DEFAULT 0,
    sessions_started INTEGER DEFAULT 0,
    tasks_completed INTEGER DEFAULT 0,
    birds_unlocked INTEGER DEFAULT 0,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, date),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Index for time-series queries
CREATE INDEX idx_daily_stats_date ON daily_stats(user_id, date);
```

#### 2.5.2 Tag Statistics (Aggregated Data)
```sql
CREATE TABLE tag_stats (
    user_id TEXT NOT NULL,
    tag_id TEXT NOT NULL,
    total_time INTEGER DEFAULT 0, -- total minutes spent on this tag
    session_count INTEGER DEFAULT 0, -- number of sessions with this tag
    last_used DATETIME,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, tag_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (tag_id) REFERENCES tags(id) ON DELETE CASCADE
);
```

## 3. Database Relationships

### 3.1 Entity Relationship Diagram
```
Users (1) ←→ (1) UserSettings
  ↓ (1:N)
Tasks ←→ (M:N) Tags (through task_tags)
  ↓ (1:N)
Sessions (N:1) → Birds (through bird_unlocked_id)
  ↓ (1:N)
UserBirds (M:N between Users and Birds)

Users (1) ←→ (N) DailyStats
Users (1) ←→ (N) TagStats
```

### 3.2 Key Relationships
- **User → Tasks**: One user can have many tasks
- **Task → Tags**: Many-to-many relationship for flexible tagging
- **User → Sessions**: One user can have many timer sessions
- **Session → Task**: Each session can be associated with one task (optional)
- **Session → Bird**: Each session can unlock at most one bird
- **User ← → Birds**: Many-to-many through user_birds (collection tracking)

## 4. Data Validation Rules

### 4.1 Business Logic Constraints
```sql
-- Ensure session duration is realistic (1 minute to 3 hours)
ALTER TABLE sessions ADD CONSTRAINT check_duration 
    CHECK (planned_duration BETWEEN 1 AND 180);

-- Ensure completion time doesn't exceed planned time by more than 10%
CREATE TRIGGER validate_session_completion
    BEFORE UPDATE ON sessions
    WHEN NEW.actual_duration > (NEW.planned_duration * 1.1)
BEGIN
    SELECT RAISE(ABORT, 'Actual duration cannot exceed planned duration by more than 10%');
END;

-- Update task total focus time when session is completed
CREATE TRIGGER update_task_focus_time
    AFTER UPDATE ON sessions
    WHEN NEW.completed = 1 AND OLD.completed = 0 AND NEW.task_id IS NOT NULL
BEGIN
    UPDATE tasks 
    SET total_focus_time = total_focus_time + NEW.actual_duration,
        updated_at = CURRENT_TIMESTAMP
    WHERE id = NEW.task_id;
END;

-- Auto-complete task when marked as done
CREATE TRIGGER auto_complete_task
    BEFORE UPDATE ON tasks
    WHEN NEW.status = 'done' AND OLD.status != 'done'
BEGIN
    UPDATE tasks SET completed_at = CURRENT_TIMESTAMP WHERE id = NEW.id;
END;
```

### 4.2 Data Integrity Rules
- **Referential Integrity**: Foreign keys ensure valid relationships
- **Check Constraints**: Validate enum values and data ranges
- **Unique Constraints**: Prevent duplicate tag names per user
- **Not Null Constraints**: Ensure required fields are populated

## 5. Database Operations

### 5.1 Common Queries

#### 5.1.1 Timer Operations
```sql
-- Start a new focus session
INSERT INTO sessions (user_id, task_id, session_type, planned_duration, start_time)
VALUES (?, ?, 'focus', ?, CURRENT_TIMESTAMP);

-- Complete a session with bird unlock check
UPDATE sessions 
SET end_time = CURRENT_TIMESTAMP, 
    actual_duration = ?,
    completed = 1,
    bird_unlocked_id = ?
WHERE id = ?;

-- Get active session for user
SELECT s.*, t.title as task_title 
FROM sessions s
LEFT JOIN tasks t ON s.task_id = t.id
WHERE s.user_id = ? AND s.end_time IS NULL;
```

#### 5.1.2 Task Management
```sql
-- Get tasks by status with tags
SELECT t.*, GROUP_CONCAT(tag.name) as tag_names
FROM tasks t
LEFT JOIN task_tags tt ON t.id = tt.task_id
LEFT JOIN tags tag ON tt.tag_id = tag.id
WHERE t.user_id = ? AND t.status = ?
GROUP BY t.id
ORDER BY t.created_at DESC;

-- Update task status
UPDATE tasks 
SET status = ?, updated_at = CURRENT_TIMESTAMP
WHERE id = ? AND user_id = ?;
```

#### 5.1.3 Analytics Queries
```sql
-- Daily focus time for last 30 days
SELECT date, total_focus_time
FROM daily_stats
WHERE user_id = ? 
  AND date >= date('now', '-30 days')
ORDER BY date;

-- Tag time distribution
SELECT t.name, ts.total_time, ts.session_count
FROM tag_stats ts
JOIN tags t ON ts.tag_id = t.id
WHERE ts.user_id = ?
ORDER BY ts.total_time DESC;

-- Bird collection progress
SELECT b.*, ub.unlocked_at
FROM birds b
LEFT JOIN user_birds ub ON b.id = ub.bird_id AND ub.user_id = ?
ORDER BY b.sort_order;
```

### 5.2 Performance Optimization

#### 5.2.1 Indexing Strategy
```sql
-- Composite indexes for common query patterns
CREATE INDEX idx_sessions_user_type_date ON sessions(user_id, session_type, start_time);
CREATE INDEX idx_tasks_user_status_updated ON tasks(user_id, status, updated_at);
CREATE INDEX idx_user_birds_user_unlocked ON user_birds(user_id, unlocked_at DESC);

-- Partial indexes for specific conditions
CREATE INDEX idx_active_sessions ON sessions(user_id, start_time) 
WHERE end_time IS NULL;

CREATE INDEX idx_completed_tasks ON tasks(user_id, completed_at) 
WHERE status = 'done';
```

#### 5.2.2 Query Optimization
- **Limit Result Sets**: Use pagination for large datasets
- **Selective Columns**: Only query needed columns
- **Prepared Statements**: Improve query compilation performance
- **Transaction Batching**: Group related operations

## 6. Data Migration and Versioning

### 6.1 Schema Versioning
```sql
CREATE TABLE schema_migrations (
    version INTEGER PRIMARY KEY,
    applied_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    description TEXT
);

-- Track current schema version
INSERT INTO schema_migrations (version, description) 
VALUES (1, 'Initial schema creation');
```

### 6.2 Migration Scripts
- **Version 1**: Initial schema setup
- **Version 2**: Add analytics tables (daily_stats, tag_stats)
- **Version 3**: Add bird rarity and unlock probability
- **Version 4**: Add user preferences and themes

### 6.3 Backup and Export
```sql
-- Export user data for backup
SELECT 
    'tasks' as table_name,
    json_group_array(
        json_object(
            'id', id,
            'title', title,
            'description', description,
            'status', status,
            'created_at', created_at
        )
    ) as data
FROM tasks WHERE user_id = ?
UNION ALL
SELECT 'sessions', json_group_array(
    json_object(
        'id', id,
        'planned_duration', planned_duration,
        'actual_duration', actual_duration,
        'start_time', start_time,
        'completed', completed
    )
) FROM sessions WHERE user_id = ?;
```

## 7. Database Configuration

### 7.1 SQLite Configuration
```sql
-- Performance optimizations
PRAGMA journal_mode = WAL;  -- Write-Ahead Logging for better concurrency
PRAGMA synchronous = NORMAL; -- Balance between performance and durability
PRAGMA cache_size = 10000;   -- Larger cache for better performance
PRAGMA temp_store = MEMORY;  -- Store temporary tables in memory
PRAGMA mmap_size = 268435456; -- 256MB memory-mapped I/O

-- Foreign key enforcement
PRAGMA foreign_keys = ON;
```

### 7.2 Connection Management
- **Connection Pooling**: Single connection per app instance
- **WAL Mode**: Enable concurrent reads during writes
- **Backup Strategy**: Regular exports and WAL checkpoint operations
- **Vacuum Schedule**: Periodic database optimization

This data model provides a robust foundation for the Pomodoro Timer App, ensuring data integrity, performance, and scalability while maintaining the simplicity needed for a mobile application.