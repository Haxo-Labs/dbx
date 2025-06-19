# DBX

![DBX Banner](banner.png)

A minimal API layer for all types of databases, portable across Workers, Raspberry Pi, and RISC-V boards. Written in Rust with bindings for TypeScript and other languages.

## Features

- 🚀 Fast and lightweight database abstraction layer
- 🔄 **Multi-database support**: select database type and URL at runtime
- 🔢 Robust Redis primitives with support for pipeline, transaction, and Lua scripts
- 🌐 **Full REST API** for Redis operations with comprehensive endpoints
- 🧰 Well-documented API with comprehensive examples
- 🛠️ Modern Rust implementation with configurable features
- 🧩 Modular architecture for easy extension
- ⚙️ Environment-based configuration with `.env` support

## Project Structure

```
dbx/
├── crates/            # Main crate containing all modules
│   ├── adapter/       # Database adapters
│   │   └── redis/     # Redis adapter implementation
│   │       ├── client.rs          # Redis client functionality
│   │       └── primitives/        # Redis primitive data types
│   │           └── string.rs      # Redis string operations
├── api/               # REST API server
│   ├── src/           # API source code
│   │   ├── main.rs    # CLI entry point
│   │   ├── config.rs  # Configuration management
│   │   ├── models.rs  # Request/response models
│   │   ├── server.rs  # Axum server setup
│   │   ├── handlers/  # API endpoint handlers (per database)
│   │   │   ├── redis.rs
│   │   │   └── ...
│   │   ├── routes/    # API routes (per database)
│   │   │   ├── redis.rs
│   │   │   └── ...
│   │   └── middleware.rs # Error handling
│   ├── examples/      # Usage examples
│   ├── tests/         # Integration tests
│   └── .env           # Environment configuration
├── Cargo.toml         # Workspace configuration
└── Cargo.lock         # Dependency lock file
```

## Multi-Database Support

DBX supports running API servers for different databases. You select the database type and connection URL at startup using CLI flags. Only the relevant routes and handlers for the selected database are enabled.

- **Current support:** Redis
- **Planned:** Postgres, MongoDB, MySQL (add your own handlers/routes for these)

### Extending

To add a new database:

- Add a new variant to the `DatabaseType` enum in `api/src/config.rs`
- Add new handler and route modules in `api/src/handlers/` and `api/src/routes/`
- Extend the CLI/server logic to support the new type

## Getting Started

### Prerequisites

- Rust 1.75 or later
- Cargo
- Redis server (for API testing)

### Building

```bash
# Clone the repository
git clone https://github.com/effortlesslabs/dbx.git
cd dbx

# Build all crates
cargo build

# Run tests
cargo test

# Run doctests
cargo test --doc
```

### Running the REST API (with CLI)

```bash
# Set up environment (optional - defaults provided)
cp api/.env.example .env
# Edit .env with your Redis connection details

# Start the API server for Redis (customize as needed)
cargo run --bin dbx-api -- --database-type redis --database-url redis://default:redispw@localhost:55000

# The server will start on http://localhost:3000 by default
```

#### CLI Options

- `--database-type` (or `-d`): Database type to serve (e.g. `redis`). Default: `redis`.
- `--database-url` (or `-u`): Database connection URL (e.g. `redis://default:redispw@localhost:55000`).
- `--host`: API server host (default: `127.0.0.1`)
- `--port` (or `-p`): API server port (default: `3000`)
- `--pool-size`: Connection pool size (default: `10`)
- `--log-level`: Log verbosity (e.g. `info`, `debug`)

#### Example

```bash
cargo run --bin dbx-api -- --database-type redis --database-url redis://default:redispw@localhost:55000 --host 0.0.0.0 --port 8080
```

### API Usage Examples

```bash
# Health check
curl http://localhost:3000/health

# Set a key
curl -X POST http://localhost:3000/api/v1/redis/strings/mykey \
  -H "Content-Type: application/json" \
  -d '{"value": "hello world", "ttl": 3600}'

# Get a key
curl http://localhost:3000/api/v1/redis/strings/mykey

# Batch operations
curl -X POST http://localhost:3000/api/v1/redis/strings/batch/set \
  -H "Content-Type: application/json" \
  -d '{"key_values": {"key1": "value1", "key2": "value2"}}'
```

## API Endpoints

### Health & Info

- `GET /health` - Health check with Redis connection status
- `GET /info` - Server information and configuration

### String Operations

- `GET /api/v1/redis/strings/:key` - Get a string value
- `POST /api/v1/redis/strings/:key` - Set a string value (with optional TTL)
- `DELETE /api/v1/redis/strings/:key` - Delete a string key
- `GET /api/v1/redis/strings/:key/exists` - Check if a key exists
- `GET /api/v1/redis/strings/:key/ttl` - Get the TTL of a key
- `POST /api/v1/redis/strings/:key/incr` - Increment a numeric value
- `POST /api/v1/redis/strings/:key/incrby` - Increment by specific amount

### Advanced Operations

- `POST /api/v1/redis/strings/:key/setnx` - Set only if key doesn't exist
- `POST /api/v1/redis/strings/:key/cas` - Compare and set atomically

### Batch Operations

- `POST /api/v1/redis/strings/batch/set` - Set multiple keys at once
- `POST /api/v1/redis/strings/batch/get` - Get multiple keys at once
- `POST /api/v1/redis/strings/batch/delete` - Delete multiple keys at once
- `POST /api/v1/redis/strings/batch/incr` - Increment multiple counters
- `POST /api/v1/redis/strings/batch/incrby` - Increment multiple counters by amounts

### Lua Script Operations

- `POST /api/v1/redis/scripts/rate-limiter` - Implement rate limiting
- `POST /api/v1/redis/scripts/multi-counter` - Increment multiple counters atomically
- `POST /api/v1/redis/scripts/multi-set-ttl` - Set multiple keys with TTL atomically

## Configuration

The API can be configured via environment variables or CLI arguments:

```bash
# Environment variables
DATABASE_TYPE=redis
DATABASE_URL=redis://default:redispw@localhost:55000
HOST=127.0.0.1
PORT=3000
POOL_SIZE=10
LOG_LEVEL=info

# CLI arguments
cargo run --bin dbx-api -- --database-type redis --database-url redis://default:redispw@localhost:55000 --port 3000
```

## Development Status

This project is currently in early development. See [ROADMAP.md](ROADMAP.md) for the detailed development plan and future goals.

## License

This project is licensed under either of

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
- MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.
