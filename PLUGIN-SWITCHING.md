# Plugin Switching Guide

## Available Plugins

### P1 - Double Click Plugin
- **Tag**: `komenpetch/clicker-plugin:p1`
- **Features**:
  - Double Click upgrade: Each click gives 2 instead of 1
  - Cost: 10 clicks

### P2 - Auto Clicker Plugin
- **Tag**: `komenpetch/clicker-plugin:p2`
- **Features**:
  - Auto Clicker I: +1 click/second, Cost: 20
  - Auto Clicker II: +5 clicks/second, Cost: 100
  - Auto Clicker III: +20 clicks/second, Cost: 500
  - Click Boost: Each manual click gives 3 instead of 1, Cost: 50

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
