# Persistent Storage Integration Guide

## Overview

This guide explains how to integrate persistent storage functionality into your own applications, using the editable schedule system as a reference. We'll cover both client-side (localStorage) and server-side (database) approaches.

## Table of Contents

1. [Client-Side Storage (localStorage)](#client-side-storage-localstorage)
2. [Server-Side Storage (Database)](#server-side-storage-database)
3. [Hybrid Approach](#hybrid-approach)
4. [Implementation Examples](#implementation-examples)
5. [Best Practices](#best-practices)
6. [Error Handling](#error-handling)

---

## Client-Side Storage (localStorage)

### What is localStorage?

localStorage is a web storage API that allows you to store data in the user's browser. Data persists until explicitly cleared by the user or your application.

### Advantages
- ✅ No server required
- ✅ Instant read/write operations
- ✅ Works offline
- ✅ Simple implementation
- ✅ 5-10MB storage limit per domain

### Disadvantages
- ❌ Data is device-specific
- ❌ Can be cleared by user
- ❌ Not suitable for sensitive data
- ❌ Limited storage capacity

### Basic Implementation

```javascript
// Save data to localStorage
function saveData(key, data) {
    try {
        localStorage.setItem(key, JSON.stringify(data));
        return true;
    } catch (error) {
        console.error('Failed to save data:', error);
        return false;
    }
}

// Load data from localStorage
function loadData(key, defaultValue = null) {
    try {
        const data = localStorage.getItem(key);
        return data ? JSON.parse(data) : defaultValue;
    } catch (error) {
        console.error('Failed to load data:', error);
        return defaultValue;
    }
}

// Remove data from localStorage
function removeData(key) {
    try {
        localStorage.removeItem(key);
        return true;
    } catch (error) {
        console.error('Failed to remove data:', error);
        return false;
    }
}
```

### Schedule-Specific Implementation

```javascript
// Schedule storage functions
class ScheduleStorage {
    constructor(storageKey = 'userSchedule') {
        this.storageKey = storageKey;
    }

    // Save entire schedule
    saveSchedule(scheduleData) {
        const dataToSave = {
            schedule: scheduleData,
            lastModified: new Date().toISOString(),
            version: '1.0'
        };
        
        return saveData(this.storageKey, dataToSave);
    }

    // Load schedule with fallback
    loadSchedule() {
        const data = loadData(this.storageKey);
        return data ? data.schedule : this.getDefaultSchedule();
    }

    // Save individual cell
    saveCellData(subject, day, value) {
        const schedule = this.loadSchedule();
        const key = `${subject}-${day}`;
        schedule[key] = value;
        return this.saveSchedule(schedule);
    }

    // Get default empty schedule
    getDefaultSchedule() {
        const subjects = ['arabic', 'english', 'chemistry', 'physics', 'biology'];
        const days = ['mon', 'tue', 'wed', 'thu', 'fri', 'sat', 'sun'];
        const schedule = {};
        
        subjects.forEach(subject => {
            days.forEach(day => {
                schedule[`${subject}-${day}`] = '-';
            });
        });
        
        return schedule;
    }

    // Clear all schedule data
    clearSchedule() {
        return removeData(this.storageKey);
    }

    // Export schedule data
    exportSchedule() {
        const data = loadData(this.storageKey);
        return data ? JSON.stringify(data, null, 2) : null;
    }

    // Import schedule data
    importSchedule(jsonData) {
        try {
            const data = JSON.parse(jsonData);
            return saveData(this.storageKey, data);
        } catch (error) {
            console.error('Invalid JSON data:', error);
            return false;
        }
    }
}

// Usage example
const scheduleStorage = new ScheduleStorage();

// Save a time slot
scheduleStorage.saveCellData('arabic', 'mon', '<span class="time-slot">9:00</span>');

// Load the schedule
const currentSchedule = scheduleStorage.loadSchedule();

// Export for backup
const backupData = scheduleStorage.exportSchedule();
```

---

## Server-Side Storage (Database)

### Database Options

#### 1. SQLite (Lightweight)
```sql
-- Create schedule table
CREATE TABLE user_schedules (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id TEXT NOT NULL,
    subject TEXT NOT NULL,
    day_of_week TEXT NOT NULL,
    time_slot TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Create index for faster queries
CREATE INDEX idx_user_schedule ON user_schedules(user_id, subject, day_of_week);
```

#### 2. PostgreSQL/MySQL (Production)
```sql
-- Create users table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create schedules table
CREATE TABLE schedules (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    subject VARCHAR(50) NOT NULL,
    day_of_week VARCHAR(10) NOT NULL,
    time_slot TEXT,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create unique constraint
ALTER TABLE schedules ADD CONSTRAINT unique_user_subject_day 
UNIQUE (user_id, subject, day_of_week);
```

### Backend API Implementation (Node.js/Express)

```javascript
const express = require('express');
const sqlite3 = require('sqlite3').verbose();
const app = express();

app.use(express.json());

// Database connection
const db = new sqlite3.Database('schedule.db');

// Initialize database
db.serialize(() => {
    db.run(`CREATE TABLE IF NOT EXISTS user_schedules (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        user_id TEXT NOT NULL,
        subject TEXT NOT NULL,
        day_of_week TEXT NOT NULL,
        time_slot TEXT,
        created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
        updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    )`);
});

// API Routes

// Get user's complete schedule
app.get('/api/schedule/:userId', (req, res) => {
    const { userId } = req.params;
    
    db.all(
        'SELECT * FROM user_schedules WHERE user_id = ? ORDER BY subject, day_of_week',
        [userId],
        (err, rows) => {
            if (err) {
                return res.status(500).json({ error: err.message });
            }
            
            // Transform to frontend format
            const schedule = {};
            rows.forEach(row => {
                const key = `${row.subject}-${row.day_of_week}`;
                schedule[key] = row.time_slot || '-';
            });
            
            res.json({ schedule, lastModified: new Date().toISOString() });
        }
    );
});

// Save/update schedule cell
app.post('/api/schedule/:userId', (req, res) => {
    const { userId } = req.params;
    const { subject, day, timeSlot } = req.body;
    
    db.run(
        `INSERT OR REPLACE INTO user_schedules 
         (user_id, subject, day_of_week, time_slot, updated_at) 
         VALUES (?, ?, ?, ?, CURRENT_TIMESTAMP)`,
        [userId, subject, day, timeSlot],
        function(err) {
            if (err) {
                return res.status(500).json({ error: err.message });
            }
            
            res.json({ 
                success: true, 
                id: this.lastID,
                message: 'Schedule updated successfully' 
            });
        }
    );
});

// Save entire schedule
app.put('/api/schedule/:userId', (req, res) => {
    const { userId } = req.params;
    const { schedule } = req.body;
    
    db.serialize(() => {
        db.run('BEGIN TRANSACTION');
        
        // Clear existing schedule
        db.run('DELETE FROM user_schedules WHERE user_id = ?', [userId]);
        
        // Insert new schedule
        const stmt = db.prepare(`
            INSERT INTO user_schedules (user_id, subject, day_of_week, time_slot) 
            VALUES (?, ?, ?, ?)
        `);
        
        Object.keys(schedule).forEach(key => {
            const [subject, day] = key.split('-');
            const timeSlot = schedule[key] !== '-' ? schedule[key] : null;
            stmt.run([userId, subject, day, timeSlot]);
        });
        
        stmt.finalize();
        db.run('COMMIT');
        
        res.json({ success: true, message: 'Schedule saved successfully' });
    });
});

// Delete schedule
app.delete('/api/schedule/:userId', (req, res) => {
    const { userId } = req.params;
    
    db.run('DELETE FROM user_schedules WHERE user_id = ?', [userId], function(err) {
        if (err) {
            return res.status(500).json({ error: err.message });
        }
        
        res.json({ 
            success: true, 
            deletedRows: this.changes,
            message: 'Schedule deleted successfully' 
        });
    });
});

app.listen(3000, () => {
    console.log('Schedule API server running on port 3000');
});
```

### Frontend API Integration

```javascript
class ScheduleAPI {
    constructor(baseUrl = '/api', userId = null) {
        this.baseUrl = baseUrl;
        this.userId = userId;
    }

    // Set user ID
    setUserId(userId) {
        this.userId = userId;
    }

    // Load schedule from server
    async loadSchedule() {
        try {
            const response = await fetch(`${this.baseUrl}/schedule/${this.userId}`);
            const data = await response.json();
            
            if (!response.ok) {
                throw new Error(data.error || 'Failed to load schedule');
            }
            
            return data.schedule;
        } catch (error) {
            console.error('Error loading schedule:', error);
            throw error;
        }
    }

    // Save single cell to server
    async saveCellData(subject, day, timeSlot) {
        try {
            const response = await fetch(`${this.baseUrl}/schedule/${this.userId}`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({ subject, day, timeSlot })
            });
            
            const data = await response.json();
            
            if (!response.ok) {
                throw new Error(data.error || 'Failed to save cell data');
            }
            
            return data;
        } catch (error) {
            console.error('Error saving cell data:', error);
            throw error;
        }
    }

    // Save entire schedule to server
    async saveSchedule(schedule) {
        try {
            const response = await fetch(`${this.baseUrl}/schedule/${this.userId}`, {
                method: 'PUT',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({ schedule })
            });
            
            const data = await response.json();
            
            if (!response.ok) {
                throw new Error(data.error || 'Failed to save schedule');
            }
            
            return data;
        } catch (error) {
            console.error('Error saving schedule:', error);
            throw error;
        }
    }

    // Delete schedule from server
    async deleteSchedule() {
        try {
            const response = await fetch(`${this.baseUrl}/schedule/${this.userId}`, {
                method: 'DELETE'
            });
            
            const data = await response.json();
            
            if (!response.ok) {
                throw new Error(data.error || 'Failed to delete schedule');
            }
            
            return data;
        } catch (error) {
            console.error('Error deleting schedule:', error);
            throw error;
        }
    }
}

// Usage example
const api = new ScheduleAPI('/api', 'user123');

// Load schedule
api.loadSchedule()
    .then(schedule => {
        console.log('Loaded schedule:', schedule);
        // Update UI with loaded data
    })
    .catch(error => {
        console.error('Failed to load schedule:', error);
        // Handle error (show message, use cached data, etc.)
    });

// Save cell data
api.saveCellData('arabic', 'mon', '<span class="time-slot">9:00</span>')
    .then(result => {
        console.log('Cell saved:', result);
    })
    .catch(error => {
        console.error('Failed to save cell:', error);
    });
```

---

## Hybrid Approach

Combine localStorage and server storage for optimal user experience:

```javascript
class HybridScheduleStorage {
    constructor(userId) {
        this.localStorage = new ScheduleStorage(`schedule_${userId}`);
        this.api = new ScheduleAPI('/api', userId);
        this.syncInProgress = false;
    }

    // Load with fallback strategy
    async loadSchedule() {
        try {
            // Try to load from server first
            const serverSchedule = await this.api.loadSchedule();
            
            // Save to localStorage as cache
            this.localStorage.saveSchedule(serverSchedule);
            
            return serverSchedule;
        } catch (error) {
            console.warn('Server unavailable, using cached data:', error);
            
            // Fallback to localStorage
            return this.localStorage.loadSchedule();
        }
    }

    // Save with sync strategy
    async saveSchedule(schedule) {
        // Save to localStorage immediately (instant feedback)
        this.localStorage.saveSchedule(schedule);
        
        // Sync to server in background
        this.syncToServer(schedule);
        
        return true;
    }

    // Background sync to server
    async syncToServer(schedule) {
        if (this.syncInProgress) return;
        
        this.syncInProgress = true;
        
        try {
            await this.api.saveSchedule(schedule);
            console.log('Schedule synced to server');
        } catch (error) {
            console.error('Sync failed, will retry later:', error);
            // Could implement retry logic here
        } finally {
            this.syncInProgress = false;
        }
    }

    // Periodic sync (call this on app startup or periodically)
    async periodicSync() {
        try {
            const localSchedule = this.localStorage.loadSchedule();
            const serverSchedule = await this.api.loadSchedule();
            
            // Simple conflict resolution: server wins
            if (JSON.stringify(localSchedule) !== JSON.stringify(serverSchedule)) {
                this.localStorage.saveSchedule(serverSchedule);
                return serverSchedule;
            }
            
            return localSchedule;
        } catch (error) {
            console.error('Periodic sync failed:', error);
            return this.localStorage.loadSchedule();
        }
    }
}

// Usage
const hybridStorage = new HybridScheduleStorage('user123');

// Load with automatic fallback
hybridStorage.loadSchedule().then(schedule => {
    // Use schedule data
});

// Save with instant local storage + background sync
hybridStorage.saveSchedule(scheduleData);

// Periodic sync (e.g., every 5 minutes)
setInterval(() => {
    hybridStorage.periodicSync();
}, 5 * 60 * 1000);
```

---

## Best Practices

### 1. Data Validation

```javascript
function validateScheduleData(schedule) {
    const validSubjects = ['arabic', 'english', 'chemistry', 'physics', 'biology'];
    const validDays = ['mon', 'tue', 'wed', 'thu', 'fri', 'sat', 'sun'];
    
    for (const key in schedule) {
        const [subject, day] = key.split('-');
        
        if (!validSubjects.includes(subject) || !validDays.includes(day)) {
            throw new Error(`Invalid schedule key: ${key}`);
        }
        
        // Validate time format if not empty
        const value = schedule[key];
        if (value !== '-' && value && !value.includes('time-slot') && !value.includes('online-slot')) {
            // Could add more specific validation here
        }
    }
    
    return true;
}
```

### 2. Error Handling

```javascript
class ScheduleStorageWithErrorHandling {
    constructor() {
        this.errorCallbacks = [];
    }

    onError(callback) {
        this.errorCallbacks.push(callback);
    }

    handleError(error, operation) {
        console.error(`Storage error in ${operation}:`, error);
        
        this.errorCallbacks.forEach(callback => {
            try {
                callback(error, operation);
            } catch (callbackError) {
                console.error('Error in error callback:', callbackError);
            }
        });
    }

    async saveSchedule(schedule) {
        try {
            validateScheduleData(schedule);
            // ... save logic
        } catch (error) {
            this.handleError(error, 'saveSchedule');
            throw error;
        }
    }
}
```

### 3. Performance Optimization

```javascript
class OptimizedScheduleStorage {
    constructor() {
        this.saveTimeout = null;
        this.pendingChanges = {};
    }

    // Debounced save (wait for user to stop typing)
    debouncedSave(subject, day, value, delay = 1000) {
        const key = `${subject}-${day}`;
        this.pendingChanges[key] = value;
        
        clearTimeout(this.saveTimeout);
        this.saveTimeout = setTimeout(() => {
            this.flushPendingChanges();
        }, delay);
    }

    async flushPendingChanges() {
        if (Object.keys(this.pendingChanges).length === 0) return;
        
        const currentSchedule = this.loadSchedule();
        Object.assign(currentSchedule, this.pendingChanges);
        
        await this.saveSchedule(currentSchedule);
        this.pendingChanges = {};
    }
}
```

### 4. Data Migration

```javascript
class ScheduleStorageWithMigration {
    constructor() {
        this.currentVersion = '2.0';
        this.migrations = {
            '1.0': this.migrateFromV1,
            '1.5': this.migrateFromV15
        };
    }

    loadSchedule() {
        const data = loadData(this.storageKey);
        
        if (!data) return this.getDefaultSchedule();
        
        // Check if migration is needed
        if (data.version !== this.currentVersion) {
            return this.migrateData(data);
        }
        
        return data.schedule;
    }

    migrateData(data) {
        let currentData = data;
        
        // Apply migrations in sequence
        Object.keys(this.migrations).forEach(version => {
            if (this.shouldMigrate(currentData.version, version)) {
                currentData = this.migrations[version](currentData);
            }
        });
        
        // Save migrated data
        this.saveSchedule(currentData.schedule);
        
        return currentData.schedule;
    }

    migrateFromV1(data) {
        // Example: Convert old format to new format
        const newSchedule = {};
        
        // ... migration logic
        
        return {
            schedule: newSchedule,
            version: '1.5',
            lastModified: new Date().toISOString()
        };
    }
}
```

---

## Error Handling

### Client-Side Error Handling

```javascript
class RobustScheduleStorage {
    async saveWithRetry(data, maxRetries = 3) {
        for (let attempt = 1; attempt <= maxRetries; attempt++) {
            try {
                return await this.saveSchedule(data);
            } catch (error) {
                console.warn(`Save attempt ${attempt} failed:`, error);
                
                if (attempt === maxRetries) {
                    throw new Error(`Failed to save after ${maxRetries} attempts: ${error.message}`);
                }
                
                // Wait before retry (exponential backoff)
                await new Promise(resolve => setTimeout(resolve, Math.pow(2, attempt) * 1000));
            }
        }
    }

    async loadWithFallback() {
        try {
            // Try server first
            return await this.api.loadSchedule();
        } catch (serverError) {
            console.warn('Server load failed, trying localStorage:', serverError);
            
            try {
                return this.localStorage.loadSchedule();
            } catch (localError) {
                console.error('All storage methods failed:', { serverError, localError });
                
                // Return default schedule as last resort
                return this.getDefaultSchedule();
            }
        }
    }
}
```

### Server-Side Error Handling

```javascript
// Express error handling middleware
app.use((error, req, res, next) => {
    console.error('API Error:', error);
    
    // Database connection errors
    if (error.code === 'SQLITE_BUSY') {
        return res.status(503).json({
            error: 'Database temporarily unavailable',
            retryAfter: 5
        });
    }
    
    // Validation errors
    if (error.name === 'ValidationError') {
        return res.status(400).json({
            error: 'Invalid data format',
            details: error.message
        });
    }
    
    // Generic server error
    res.status(500).json({
        error: 'Internal server error',
        message: process.env.NODE_ENV === 'development' ? error.message : 'Something went wrong'
    });
});
```

---

## Integration Checklist

### ✅ Planning Phase
- [ ] Choose storage strategy (localStorage, database, or hybrid)
- [ ] Define data structure and validation rules
- [ ] Plan for offline functionality (if needed)
- [ ] Consider data migration strategy
- [ ] Design error handling approach

### ✅ Implementation Phase
- [ ] Set up storage layer (database tables, API endpoints)
- [ ] Implement basic CRUD operations
- [ ] Add data validation and sanitization
- [ ] Implement error handling and retry logic
- [ ] Add performance optimizations (debouncing, caching)

### ✅ Testing Phase
- [ ] Test with valid and invalid data
- [ ] Test offline scenarios (localStorage only)
- [ ] Test server failures and recovery
- [ ] Test data migration (if applicable)
- [ ] Performance testing with large datasets

### ✅ Deployment Phase
- [ ] Set up production database
- [ ] Configure backup and recovery
- [ ] Monitor error rates and performance
- [ ] Plan for scaling (if needed)

---

## Conclusion

The persistent storage system can be implemented at different levels of complexity:

1. **Simple**: localStorage only (good for prototypes, single-device use)
2. **Intermediate**: Database with API (good for multi-device, user accounts)
3. **Advanced**: Hybrid approach with offline support (best user experience)

Choose the approach that best fits your application's requirements, user base, and technical constraints. Start simple and evolve as your needs grow.

Remember to always validate data, handle errors gracefully, and provide good user feedback about the storage status.

