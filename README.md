# Clicker Application

A microkernel-based clicker game with plugin system, event-driven architecture, and real-time history tracking.

## Architecture

```
┌─────────────┐
│  Frontend   │ (React on port 80)
│  (React)    │
└──────┬──────┘
       │ HTTP REST
       ↓
┌─────────────┐      gRPC        ┌──────────────┐
│   Backend   │ ←────────────────→│   Plugins    │
│  (Express)  │                   │   (gRPC)     │
└──────┬──────┘                   └──────────────┘
       │                          port 50051
       │ publishes events
       ↓
┌─────────────┐
│  RabbitMQ   │
│   Queue     │
└──────┬──────┘
       │ consumes events
       ↓
┌─────────────┐      HTTP         ┌──────────────┐
│    Logs     │ ←────────────────→│   Frontend   │
│  Service    │   /api/history    │  /history    │
└──────┬──────┘                   └──────────────┘
       │
       ↓
┌─────────────┐      ┌──────────────┐
│   MySQL     │ ←────│ phpMyAdmin   │ (Web UI :8080)
│  Database   │      │  (Web GUI)   │
└─────────────┘      └──────────────┘
```

## Services

1. **Frontend** (port 80) - React application with clicker game UI and history page
2. **Backend** (port 3001) - Express + Prisma microkernel core, manages game state
3. **Plugins** (port 50051) - gRPC services that define game mechanics (P1, P2)
4. **Logs Service** (port 3002) - Event-driven history tracking via RabbitMQ
5. **RabbitMQ** (ports 5672, 15672) - Message queue for event streaming
6. **MySQL** (port 3306) - Database for game state and history
7. **phpMyAdmin** (port 8080) - Web-based MySQL management interface

## Quick Start

### Start all services:
```bash
cd clicker-app
docker-compose up -d
```

### Access:
- **Game**: http://localhost
- **History**: http://localhost/history
- **Backend API**: http://localhost:3001
- **Logs API**: http://localhost:3002
- **phpMyAdmin (MySQL)**: http://localhost:8080 (root/password)
- **RabbitMQ Management**: http://localhost:15672 (admin/admin)
- **MySQL Direct**: localhost:3306 (root/password)

### Stop all services:
```bash
docker-compose down
```

## Features

### Game Mechanics
- Manual clicking to earn points
- Purchase upgrades to increase click value
- Auto-click upgrades for passive income
- Real-time counter updates

### Plugin System
Swap between different game mechanics without changing core code:
- **P1 (Double Click)**: Multiplies manual click value
- **P2 (Auto Clicker)**: Generates passive clicks per second

### History Tracking
- Real-time event logging via message queue
- View all click history with timestamps
- Statistics dashboard (increases, decreases, net change)
- Paginated history table
- Filter by action type and reason

## Plugin Switching Guide

## Available Plugins

### P1 - Double Click Plugin
- **Tag**: `komenpetch/clicker-plugin:p1`
- **Features**:
  - Double Click upgrade: Each click gives 2 instead of 1
  - Cost: 10 clicks

### P2 - Auto Clicker Plugin
- **Tag**: `komenpetch/clicker-plugin:p2`
- **Features**:
  - Auto Clicker: +1 click/second, Cost: 15

## How to Switch Plugins

### On VM or Local Machine

1. **Stop the current services**:
   ```bash
   docker-compose down
   ```

2. **Edit docker-compose.yml**:
   - Line 24: Change plugin image tag
   - For P1: `image: komenpetch/clicker-plugin:p1`
   - For P2: `image: komenpetch/clicker-plugin:p2`

3. **Pull the new image**:
   ```bash
   docker-compose pull plugin
   ```

4. **Start services**:
   ```bash
   docker-compose up -d
   ```

5. **Verify the switch**:
   ```bash
   curl http://localhost:3001/api/plugin/info
   ```

### Quick Switch Commands

**Switch to P1 (Double Click)**:
```bash
sed -i 's/clicker-plugin:p[12]/clicker-plugin:p1/' docker-compose.yml
docker-compose down
docker-compose pull plugin
docker-compose up -d
```

**Switch to P2 (Auto Clicker)**:
```bash
sed -i 's/clicker-plugin:p[12]/clicker-plugin:p2/' docker-compose.yml
docker-compose down
docker-compose pull plugin
docker-compose up -d
```

## Testing Different Algorithms

1. **Start with P1**: Test the double click upgrade mechanism
2. **Switch to P2**: Test the auto-clicker and click boost upgrades
3. **Compare**: Notice how different upgrade systems affect gameplay

## Note on Data Persistence

- The MySQL database persists data across plugin switches
- You may need to reset your progress when switching plugins since upgrades are plugin-specific
- Use the reset button in the UI or call: `curl -X POST http://localhost:3001/api/counter/reset`

---

## 🗄️ MySQL Database Inspection

### Access Methods

#### 1. **phpMyAdmin (Web Interface - Recommended)**

Open in browser: **http://localhost:8080**

**Login:**
- Username: `root`
- Password: `password`

**Features:**
- Visual table browsing
- SQL query editor with syntax highlighting
- Data export (CSV, JSON, SQL)
- Table structure viewer
- Search and filter data

**Quick Steps:**
1. Open http://localhost:8080
2. Login with root/password
3. Click `clicker_db` on the left sidebar
4. Click `click_history` to view all events
5. Use "SQL" tab to run custom queries

#### 2. **MySQL Command Line (Interactive)**

Enter MySQL shell:
```bash
docker exec -it clicker-mysql mysql -uroot -ppassword clicker_db
```

Common queries:
```sql
-- Show all tables
SHOW TABLES;

-- View recent history
SELECT * FROM click_history ORDER BY createdAt DESC LIMIT 10;

-- View statistics
SELECT
  COUNT(*) as total_events,
  SUM(CASE WHEN action='increase' THEN 1 ELSE 0 END) as increases,
  SUM(CASE WHEN action='decrease' THEN 1 ELSE 0 END) as decreases
FROM click_history;

-- View current game state
SELECT * FROM Counter;

-- Exit
EXIT;
```

#### 3. **One-Line Queries (from terminal)**

View recent history:
```bash
docker exec clicker-mysql mysql -uroot -ppassword clicker_db --table -e "SELECT id, action, amount, CONCAT(oldValue, ' → ', newValue) as Change, reason, DATE_FORMAT(createdAt, '%H:%i:%s') as Time FROM click_history ORDER BY id DESC LIMIT 10;" 2>&1 | grep -v "Using a password"
```

View statistics:
```bash
docker exec clicker-mysql mysql -uroot -ppassword clicker_db --table -e "SELECT COUNT(*) as 'Total Events', SUM(CASE WHEN action='increase' THEN 1 ELSE 0 END) as 'Increases', SUM(CASE WHEN action='decrease' THEN 1 ELSE 0 END) as 'Decreases' FROM click_history;" 2>&1 | grep -v "Using a password"
```

#### 4. **MySQL Workbench / DBeaver** (Desktop Tools)

**Connection Settings:**
- Host: `localhost`
- Port: `3306`
- Username: `root`
- Password: `password`
- Database: `clicker_db`

### Database Tables

- **`click_history`** - All click events (increases/decreases)
  - `id` - Event ID
  - `action` - "increase" or "decrease"
  - `amount` - Amount changed
  - `oldValue` - Value before change
  - `newValue` - Value after change
  - `reason` - "manual_click", "auto_click", or "purchase_upgrade"
  - `metadata` - JSON with additional info
  - `createdAt` - Timestamp

- **`Counter`** - Current game state
  - `id` - Counter ID
  - `value` - Current click count
  - `clicksPerClick` - Clicks per click
  - `upgrades` - Comma-separated upgrade IDs
