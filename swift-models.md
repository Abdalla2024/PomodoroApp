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

## 2. Core Data Models (SwiftUI)

### 2.1 User Entity

```swift
struct User: Identifiable, Codable {
    var id: String = UUID().uuidString
    var createdAt: Date = Date()
    var updatedAt: Date = Date()
    var version: Int = 1
}

struct UserSettings: Codable {
    var userId: String
    var notificationSound: Bool = true
    var vibration: Bool = true
    var visualAlerts: Bool = true
    var defaultFocusTime: Int = 25
    var defaultBreakTime: Int = 5
    var createdAt: Date = Date()
    var updatedAt: Date = Date()
}
```

### 2.2 Task Management System

#### 2.2.1 Task Model

```swift
struct Task: Identifiable, Codable {
    var id: String = UUID().uuidString
    var userId: String
    var tagId: String?
    var title: String
    var description: String?
    var status: TaskStatus = .todo
    var priority: Int = 0
    var totalFocusTime: Int = 0
    var estimatedTime: Int?
    var createdAt: Date = Date()
    var updatedAt: Date = Date()
    var completedAt: Date?
}

enum TaskStatus: String, Codable {
    case todo, inProgress = "in_progress", done
}
```

#### 2.2.2 Tag Model

```swift
struct Tag: Identifiable, Codable {
    var id: String = UUID().uuidString
    var userId: String
    var name: String
    var color: String = "#6366f1"
    var isCustom: Bool = true
    var createdAt: Date = Date()
}
```

### 2.3 Timer and Session Management

```swift
struct Session: Identifiable, Codable {
    var id: String = UUID().uuidString
    var userId: String
    var taskId: String?
    var sessionType: SessionType = .focus
    var plannedDuration: Int
    var actualDuration: Int?
    var startTime: Date
    var endTime: Date?
    var completed: Bool = false
    var birdUnlockedId: String?
    var interruptions: Int = 0
    var createdAt: Date = Date()
}

enum SessionType: String, Codable {
    case focus, breakSession = "break"
}
```

### 2.4 Bird Collection System

#### 2.4.1 Bird Model

```swift
struct Bird: Identifiable, Codable {
    var id: String
    var name: String
    var description: String?
    var rarity: BirdRarity
    var imagePath: String
    var unlockProbability: Double
    var sortOrder: Int
    var createdAt: Date = Date()
}

enum BirdRarity: String, Codable {
    case common, rare, legendary
}
```

#### 2.4.2 UserBird Model

```swift
struct UserBird: Codable {
    var userId: String
    var birdId: String
    var unlockedAt: Date = Date()
    var unlockSessionId: String?
}
```

### 2.5 Analytics and Statistics

#### 2.5.1 DailyStats Model

```swift
struct DailyStats: Codable {
    var userId: String
    var date: Date
    var totalFocusTime: Int = 0
    var totalBreakTime: Int = 0
    var sessionsCompleted: Int = 0
    var sessionsStarted: Int = 0
    var tasksCompleted: Int = 0
    var birdsUnlocked: Int = 0
    var createdAt: Date = Date()
    var updatedAt: Date = Date()
}
```

#### 2.5.2 TagStats Model

```swift
struct TagStats: Codable {
    var userId: String
    var tagId: String
    var totalTime: Int = 0
    var sessionCount: Int = 0
    var lastUsed: Date?
    var createdAt: Date = Date()
    var updatedAt: Date = Date()
}
```

## 3. Database Relationships

### 3.1 Entity Relationship Diagram

```
Users (1) ←→ (1) UserSettings
  ↓ (1:N)
Tasks ←→ (N:1) Tags
  ↓ (1:N)
Sessions (N:1) → Birds (through bird_unlocked_id)
  ↓ (1:N)
UserBirds (M:N between Users and Birds)

Users (1) ←→ (N) DailyStats
Users (1) ←→ (N) TagStats
```

### 3.2 Key Relationships

- **User → Tasks**: One user can have many tasks
- **Task → Tag**: One-to-one relationship. Each task has one tag
- **User → Sessions**: One user can have many timer sessions
- **Session → Task**: Each session can be associated with one task (optional)
- **Session → Bird**: Each session can unlock at most one bird
- **User ← → Birds**: Many-to-many through user\_birds (collection tracking)

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