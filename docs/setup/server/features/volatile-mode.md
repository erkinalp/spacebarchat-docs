# Volatile Mode

## Overview

Volatile Mode is an Anticensor feature that enables running the server with an in-memory SQLite database instead of a persistent database. This mode is designed for testing, development, and ephemeral deployments where data persistence is not required.

When Volatile Mode is enabled, **all data is stored in memory and will be lost when the server stops or restarts**.

## Use Cases

1. **Automated Testing**: Run integration tests without affecting production data or requiring database cleanup
2. **Development**: Quick server startup without database setup or migrations
3. **Demonstrations**: Temporary instances for demos or presentations
4. **CI/CD Pipelines**: Ephemeral test environments in continuous integration
5. **Load Testing**: Test server performance without database I/O overhead

## Enabling Volatile Mode

There are two ways to enable Volatile Mode:

### Method 1: Environment Variable

Set the `VOLATILE_MODE` environment variable to `true`:

```bash
VOLATILE_MODE=true npm start
```

### Method 2: Configuration

Set `general_volatileMode` to `true` in your configuration:

```json
{
	"general": {
		"volatileMode": true
	}
}
```

Or in the database configuration table:

| key                    | value  |
| ---------------------- | ------ |
| `general_volatileMode` | `true` |

**Note**: The environment variable takes precedence over the configuration setting.

## Behavior

### Database Configuration

When Volatile Mode is enabled:

1. **Database Type**: Forced to SQLite regardless of the `DATABASE` environment variable
2. **Database Location**: Uses `:memory:` instead of a file path
3. **Synchronization**: Database schema synchronization is automatically enabled (`DB_SYNC=true`)
4. **Migrations**: Migrations are skipped since the schema is synchronized directly

### Startup Process

During server startup in Volatile Mode:

1. An in-memory SQLite database is created
2. All entity schemas are synchronized to the database
3. A warning message is displayed:
    ```
    [Database] Running in VOLATILE MODE - all data will be stored in memory and lost on restart!
    ```
4. The server starts normally with an empty database

### Data Lifecycle

- **Creation**: All data (users, guilds, messages, etc.) is created in memory
- **Persistence**: Data exists only while the server is running
- **Loss**: All data is immediately lost when:
    - The server process stops
    - The server crashes
    - The server is restarted
    - The system runs out of memory

## Limitations

### Data Persistence

- **No Persistence**: All data is lost on restart
- **No Backups**: Cannot backup or restore data
- **No Migrations**: Database migrations are not used
- **No Replication**: Cannot replicate data to other instances

### Production Use

**Volatile Mode is NOT recommended for production use** because:

1. All user data, messages, and configuration will be lost on restart
2. Server crashes result in complete data loss
3. No way to recover from errors or rollback changes
4. Cannot scale horizontally (each instance has its own isolated data)

### Memory Constraints

- **Memory Usage**: All data is stored in RAM
- **Scalability**: Limited by available system memory
- **Performance**: May degrade as data grows in memory
- **OOM Risk**: Risk of out-of-memory errors with large datasets

## Advantages

### Development Benefits

1. **Fast Startup**: No need to wait for database connections or migrations
2. **Clean State**: Every restart starts with a fresh, empty database
3. **No Setup**: No need to install or configure PostgreSQL/MySQL
4. **Isolation**: Each developer can run their own isolated instance

### Testing Benefits

1. **Test Isolation**: Each test run starts with a clean database
2. **Speed**: In-memory operations are faster than disk-based databases
3. **Simplicity**: No need to manage test databases or cleanup
4. **Parallel Testing**: Multiple test suites can run simultaneously without conflicts

## Configuration Interaction

### Environment Variables

Volatile Mode interacts with other environment variables:

- `DATABASE`: Ignored when Volatile Mode is enabled
- `DB_SYNC`: Automatically set to `true` in Volatile Mode
- `DB_LOGGING`: Still respected for query logging

### Configuration Options

Most configuration options work normally in Volatile Mode:

- User registration settings
- Rate limits
- Security settings
- Feature flags

However, configuration stored in the database will be lost on restart.

## Comparison with Regular Mode

| Feature          | Regular Mode                 | Volatile Mode    |
| ---------------- | ---------------------------- | ---------------- |
| Database         | PostgreSQL/MySQL/SQLite file | In-memory SQLite |
| Data Persistence | Yes                          | No               |
| Migrations       | Required                     | Skipped          |
| Setup Complexity | High                         | Low              |
| Startup Time     | Slower                       | Faster           |
| Memory Usage     | Low                          | High             |
| Production Ready | Yes                          | No               |
| Data Recovery    | Possible                     | Impossible       |

## Best Practices

### For Development

1. **Use for Local Development**: Great for quick testing and experimentation
2. **Document Test Data**: Create scripts to populate test data on startup
3. **Seed Data**: Use database seeders to create initial data automatically
4. **Environment Separation**: Use different modes for development vs. staging

### For Testing

1. **Integration Tests**: Ideal for integration test suites
2. **Test Fixtures**: Create fixtures to populate test data
3. **Cleanup**: No cleanup needed - just restart the server
4. **Parallel Execution**: Run multiple test instances in parallel

### For Production

1. **Never Use in Production**: Always use a persistent database for production
2. **Staging Environments**: Use persistent databases even in staging
3. **Data Backup**: Implement proper backup strategies for production data
4. **High Availability**: Use database replication for production deployments

## Example: Development Workflow

### Starting a Development Server

```bash
# Start with volatile mode for quick testing
VOLATILE_MODE=true npm start
```

### Running Tests

```bash
# Run tests with in-memory database
VOLATILE_MODE=true npm test
```

### Docker Development

```dockerfile
FROM node:18

WORKDIR /app
COPY . .

RUN npm install
RUN npm run build

# Enable volatile mode for development container
ENV VOLATILE_MODE=true

CMD ["npm", "start"]
```

### CI/CD Pipeline

```yaml
# GitHub Actions example
jobs:
    test:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v2
            - uses: actions/setup-node@v2
            - run: npm install
            - run: npm run build
            - name: Run tests with volatile mode
              env:
                  VOLATILE_MODE: true
              run: npm test
```

## Troubleshooting

### Out of Memory Errors

If you encounter out-of-memory errors in Volatile Mode:

1. Reduce the amount of test data
2. Increase available memory for the Node.js process:
    ```bash
    NODE_OPTIONS="--max-old-space-size=4096" VOLATILE_MODE=true npm start
    ```
3. Consider using a persistent SQLite database for larger datasets

### Performance Issues

If performance degrades in Volatile Mode:

1. Check memory usage with `process.memoryUsage()`
2. Reduce the number of entities being created
3. Consider using a file-based SQLite database instead
4. Profile the application to identify memory leaks

### Data Loss

If you accidentally enabled Volatile Mode in production:

1. **Stop the server immediately** to prevent further data loss
2. Check if you have database backups
3. Disable Volatile Mode and restart with a persistent database
4. Restore from the most recent backup

## Related Configuration

- [Database Configuration](../database.md) - General database setup
- [Environment Variables](../configuration/env.md) - Environment variable reference
- [Configuration Options](../configuration/index.md) - All configuration options

## Technical Details

### Implementation

Volatile Mode is implemented in `src/util/util/Database.ts`:

```typescript
const isVolatileMode =
	process.env.VOLATILE_MODE === "true" ||
	(Config.get()?.general?.volatileMode ?? false);

const finalDbConnectionString = isVolatileMode
	? ":memory:"
	: dbConnectionString;
```

### SQLite In-Memory Database

The `:memory:` connection string is a special SQLite feature that creates a database entirely in RAM. This database:

- Exists only for the duration of the connection
- Is not shared between connections
- Provides full SQL functionality
- Offers excellent performance for small to medium datasets

## Security Considerations

1. **No Data Leakage**: Data cannot leak to disk since it's never written
2. **Clean Shutdown**: No sensitive data remains after shutdown
3. **Test Data**: Safe to use real-like test data without privacy concerns
4. **Temporary Secrets**: Secrets stored in the database are automatically cleared on restart

However, remember that:

- Memory dumps could still expose data
- Swap files might contain database contents
- Process memory is not encrypted
