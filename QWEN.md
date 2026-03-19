# Mattermost Autolink Plugin

## Project Overview

This is a **Mattermost plugin** that automatically transforms text matching regular expression patterns into Markdown links before messages are saved to the database. It allows Mattermost administrators to create custom auto-linking rules for their Mattermost instance.

**Key Features:**
- Pattern-based text replacement using Go's regexp library
- Supports named capture groups in patterns (`(?P<name>...)`)
- Template expansion with variables (`$name` or `${name}`)
- Optional scoping to specific teams/channels
- Word boundary matching support
- Bot post filtering
- Administration via `/autolink` slash command

**Primary Technologies:**
- **Language:** Go (1.22+)
- **Framework:** Mattermost Plugin SDK (server/public)
- **HTTP Router:** Gorilla Mux
- **Testing:** testify, gotestsum

## Project Structure

```
mattermost-plugin-autolink/
├── server/                    # Main plugin code
│   ├── main.go               # Plugin entry point
│   ├── autolink/             # Core autolink logic
│   │   ├── autolink.go       # Autolink struct and pattern matching
│   │   └── *_test.go         # Test files for various patterns
│   ├── autolinkplugin/       # Plugin implementation
│   │   └── plugin.go         # Main plugin struct and hooks
│   └── api/                  # HTTP API handlers
│       ├── api.go            # REST API for link management
│       └── api_test.go       # API tests
├── build/                     # Build scripts and manifests
├── assets/                    # Plugin assets (icon, etc.)
├── plugin.json               # Plugin manifest
├── go.mod                    # Go module dependencies
└── Makefile                  # Build automation
```

## Building and Running

### Prerequisites
- Go 1.22+
- Node.js (for webapp, if applicable)
- Mattermost server for testing/deployment

### Key Commands

```bash
# Install Go tools (golangci-lint, gotestsum)
make install-go-tools

# Run linters and tests
make check-style
make test

# Build the plugin for all architectures
make server

# Build and bundle the plugin (.tar.gz)
make dist

# Build, bundle, and deploy to a Mattermost server
make deploy

# Watch for changes and rebuild (development)
make watch

# Generate coverage report
make coverage

# Clean build artifacts
make clean
```

### Development Workflow

1. **Setup:** Run `make apply` to propagate plugin manifest settings
2. **Build:** Run `make server` to compile for all target architectures
3. **Test:** Run `make test` to execute unit tests
4. **Deploy:** Run `make deploy` to install on a Mattermost server
5. **Debug:** Use `make attach` to attach dlv debugger to running plugin

### Configuration

The plugin is configured via `config.json` under `PluginSettings.Plugins["mattermost-autolink"]`:

```json
{
  "links": [
    {
      "Pattern": "(?i)(MM)(-)(?P<jira_id>\\d+)",
      "Template": "[MM-${jira_id}](https://mattermost.atlassian.net/browse/MM-${jira_id})",
      "Scope": ["team/channel"],
      "WordMatch": false,
      "ProcessBotPosts": true
    }
  ]
}
```

**Configuration Options:**
- `Pattern`: Regular expression to match
- `Template`: Markdown link template with variable expansion
- `Scope`: Optional team or team/channel scoping
- `WordMatch`: Use word boundaries (`\b`)
- `ProcessBotPosts`: Apply to bot-generated posts

## Development Conventions

### Code Style
- **Linter:** golangci-lint with custom configuration in `.golangci.yml`
- **Enabled linters:** bodyclose, errcheck, gocritic, gofmt, goimports, gosec, gosimple, govet, ineffassign, misspell, revive, nakedret, staticcheck, stylecheck, unconvert, unused, whitespace
- **Go version:** 1.22 with `GO111MODULE=on`

### Testing Practices
- Tests use `github.com/stretchr/testify` for assertions
- Test files follow naming convention `*_test.go`
- Run tests with race detector: `go test -race ./...`
- Use `gotestsum` for formatted test output

### Architecture Patterns
- **Plugin Pattern:** Embeds `plugin.MattermostPlugin` base struct
- **Configuration:** Thread-safe config access with `sync.RWMutex`
- **HTTP Handlers:** Gorilla Mux router with middleware for authorization
- **Interface-based design:** `Store` and `Authorization` interfaces for testability

### Key Interfaces
```go
type Store interface {
    GetLinks() []autolink.Autolink
    SaveLinks([]autolink.Autolink) error
}

type Authorization interface {
    IsAuthorizedAdmin(userID string) (bool, error)
}
```

## Plugin Hooks

The plugin implements the following Mattermost plugin hooks:
- `OnActivate()` - Initialize plugin and API handler
- `MessageWillBePosted(c *plugin.Context, post *model.Post) (*model.Post, string)` - Process new messages
- `MessageWillBeUpdated(c *plugin.Context, newPost, oldPost *model.Post) (*model.Post, string)` - Process updated messages (if enabled)
- `ServeHTTP()` - Handle API requests

## API Endpoints

- `POST /api/v1/link` - Create or update an autolink rule
  - Requires admin or plugin authorization
  - Body: `autolink.Autolink` JSON

## Useful Links

- [Mattermost Plugin Developer Documentation](https://developers.mattermost.com/integrate/plugins/developer-workflow/)
- [Go regexp package](https://golang.org/pkg/regexp/)
- [Regex Testing Tools](https://regex101.com/)
- [GitHub Repository](https://github.com/mattermost-community/mattermost-plugin-autolink)
