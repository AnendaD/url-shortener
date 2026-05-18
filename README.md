# URL Shortener

## Features

- **URL Shortening**: Convert long URLs into short, memorable aliases
- **Custom Aliases**: Support for user-defined aliases or auto-generated random strings
- **RESTful API**: Clean, well-structured HTTP endpoints
- **Structured Logging**: Beautiful, leveled logging with slog
- **SQLite Storage**: Lightweight, file-based database
- **Comprehensive Testing**: Unit tests and integration tests included

## Architecture

The project follows a clean architecture pattern with clear separation of concerns:

```
url-shortener/
├── cmd/url-shortener/          # Application entry point
├── internal/
│   ├── config/                 # Configuration management
│   ├── http-server/
│   │   ├── handlers/           # HTTP request handlers
│   │   │   ├── redirect/       # URL redirect handler
│   │   │   └── url/save/       # URL save handler
│   │   └── middleware/         # HTTP middleware (logging)
│   ├── lib/
│   │   ├── api/                # API utilities
│   │   ├── logger/             # Logging utilities
│   │   └── random/             # Random string generation
│   └── storage/
│       └── sqlite/             # SQLite storage implementation
├── storage/
└─ tests/                       # Integration tests
```

## Prerequisites

- Go 1.21+
- SQLite3

## Installation

1. Clone the repository:
```bash
git clone <repository-url>
cd url-shortener
```

2. Install dependencies:
```bash
go mod download
```

3. Set up the configuration file:
```bash
cp config/local.yaml config/local.yaml
```

4. Set the `CONFIG_PATH` environment variable:
```bash
export CONFIG_PATH=./config/local.yaml
```

## Configuration

The application uses YAML configuration files. Example configuration (`local.yaml`):

```yaml
env: "local"
storage_path: "./storage/storage.db"
http_server:
  address: "localhost:8082"
  timeout: 4s
  idle_timeout: 60s
  user: "myuser"
  password: "mypass"
```

### Configuration Parameters

- **env**: Environment (local/dev/prod) - affects logging format
- **storage_path**: Path to SQLite database file
- **http_server.address**: Server bind address
- **http_server.timeout**: Request timeout duration
- **http_server.idle_timeout**: Idle connection timeout
- **http_server.user**: Basic auth username
- **http_server.password**: Basic auth password

## Usage

### Running the Application

```bash
export CONFIG_PATH=./config/local.yaml
go run cmd/url-shortener/main.go
```

### API Endpoints

#### 1. Create Short URL

**Endpoint**: `POST /url`

**Authentication**: Basic Auth required

**Request Body**:
```json
{
  "url": "https://example.com/very/long/url",
  "alias": "myalias"  // Optional
}
```

**Response** (Success):
```json
{
  "status": "OK",
  "alias": "myalias"
}
```

**Example with curl**:
```bash
curl -X POST http://localhost:8082/url \
  -u myuser:mypass \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com", "alias": "example"}'
```

#### 2. Redirect to Original URL

**Endpoint**: `GET /{alias}`

**Authentication**: None required

**Response**: HTTP 302 redirect to the original URL

**Example**:
```bash
curl -L http://localhost:8082/example
# Redirects to https://example.com
```

### Error Responses

```json
{
  "status": "Error",
  "error": "error message"
}
```

Common errors:
- `"invalid request"` - Empty request body or missing alias
- `"field URL is a required field"` - URL not provided
- `"field URL is not a valid URL"` - Invalid URL format
- `"url already exists"` - Alias already in use
- `"not found"` - Alias doesn't exist

## Testing

### Run Unit Tests

```bash
go test ./internal/...
```

### Run Integration Tests

1. Ensure the server is running on `localhost:8082`
2. Run the tests:

```bash
go test ./tests/...
```