# MCP Go API – Phase 1 Specification

## High-Level Requirements

## Recommended Directory Structure (Go Best Practices)

A well-structured Go project for the MCP server should follow established Go conventions to maximize maintainability, clarity, and extensibility. The following layout is recommended:

```
repo-root/
├── cmd/                  # Main applications (entrypoints) for the project
│   └── mcp-server/       # Main MCP server binary (main.go)
├── internal/             # Private application and library code (not importable by other projects)
│   ├── .agent/           # All subfolders may have their own agent documentation
│   ├── api/              # HTTP handlers, endpoint logic, request/response models
├── pkg/                  # Exportable library code (optional, for public APIs)
├── api/
│   └── openapi.yaml      # OpenAPI specification for the MCP API
├── workspaces/           # Per-workspace data, code, and context (repo-a/, repo-b/, ...)
├── .agent/               # Project documentation, specs, and design notes
├── scripts/              # Utility scripts (build, deploy, etc.)
├── go.mod, go.sum        # Go module files
└── README.md
```

### Directory Purpose & Notes
- **cmd/**: Each subdirectory is a CLI or service entrypoint; `mcp-server` is the main binary.
- **internal/**: All core logic, handlers, and business rules. Keeps code private to this repo.
- **pkg/**: (Optional) Place for code you intend to be used by other projects.
- **api/**: API definitions and OpenAPI specs; keep `openapi.yaml` here as the single source of truth.
- **workspaces/**: Persistent data and context for each managed workspace/repo.
- **.agent/**: Design docs, specs, and non-code artifacts.
- **scripts/**: Helper scripts for development and automation.

#### Golang Boilerplate Requiremetns

* `internal/config.go`
  * Assume `{home}/.mcp/config.json` exists
  * Read configuration from JSON file
  * Each feature/go file should register a config struct
    * Required config values should be implemented `init()`
  * Use `spf13/viper` 
* `internal/logging.go`
  * Use `uber.org/zap`
  * `EnableSilentLogging()`
    * May need a `resetLogger()` for test
  * Config option for logging options, INFO default. 

#### API Layer File Structure (internal/api)
A minimal, idiomatic structure for the API layer:

```
internal/api/
├── handlers.go      # HTTP handler functions for each endpoint
├── routes.go        # Route registration (mux/router setup)
├── models.go        # Request/response structs, shared API types
├── errors.go        # Standardized error response types/utilities
├── docs.go          # (Optional) Serve /openapi.yaml and /docs endpoints
```

- **handlers.go**: Contains the root logic for each REST endpoint (e.g., workspace, symbol, context handlers).
- **routes.go**: Sets up the HTTP router and core endpoint registration.
- **models.go**: Defines the Go structs used for API requests and responses, ensuring type safety and clarity.
- **errors.go**: Provides reusable error types and helpers for consistent error responses.
- **docs.go**: (Optional) Serves the OpenAPI YAML and documentation endpoints; useful for `/openapi.yaml` and `/docs`.

**All resources should have a dedicated file for their handlers, routes, etc. (e.g., `handlers_workspace.go`, `routes_workspace.go`, `models_workspace.go`) where appropriate.**

**They will be referenced by the root handler, route, etc.**

---

### Purpose
Establish a foundational, context-aware agent server (MCP) in Go, designed to support multi-repo/workspace development environments. The server will provide basic code intelligence and context endpoints, serving as the backbone for future semantic and agent-driven features.

### Goals
- **Local-first, extensible architecture:** The MCP server should run locally on a developer’s machine, with the ability to scale to multiple workspaces and projects.
- **Secure remote access:** Enable secure access (e.g., via Tailscale) from VS Code or other development tools.
- **Workspace isolation:** Support multiple, logically isolated workspaces (repos) within a single MCP instance.
- **Modular context sources:** Prepare for future integration with code symbol indexing, semantic search, and agent note ingestion.

### Major Deliverables (Phase 1)
- **JSON-RPC HTTP server implemented in Go**
- **Per-workspace directory structure:**
  ```
  workspaces/
    repo-a/
    repo-b/
  ```
- **Initial API endpoints:**
  - `getSymbols`: List available code symbols in a workspace
  - `getDefinition`: Retrieve the definition of a symbol
  - `getContext`: Fetch relevant context for a given code location or query
- **Documentation:** Clear API and deployment documentation for developers

## OpenAPI Specification

To ensure clear, maintainable, and tool-friendly API definitions, all HTTP endpoints for the MCP Go API will be specified using the [OpenAPI Specification](https://swagger.io/specification/).

### OpenAPI CodeGen

All code generation (codegen) for server and client stubs MUST be performed from the canonical OpenAPI YAML file (`api/openapi.yaml`). The OpenAPI file is the single source of truth for endpoint design, request/response schemas, error objects, and all API surface details. Codegen outputs (e.g., Go server stubs, client SDKs) MUST reflect the current design and structure as specified in the OpenAPI file and the documented API/spec section.

#### Codegen Workflow Expectations
- Codegen MUST always be run against the latest `openapi.yaml`. Generated code must never be manually edited; all changes must be made in the OpenAPI spec and then regenerated.
- Any changes to the API design (including endpoints, schemas, or error formats) MUST be first updated in `openapi.yaml` and reviewed for alignment with the design principles/specification in this document.
- The OpenAPI YAML file MUST include:
  - All endpoints described in the API/spec section
  - Complete request/response schemas with required and optional fields
  - Enumerations, error object formats, and example payloads
  - Use of OpenAPI features such as `oneOf`/`anyOf` for polymorphic responses, as needed
- The generated code MUST:
  - Match the endpoint definitions, types, and error handling described in the spec
  - Remain in sync with the OpenAPI file at all times
  - Be placed in a dedicated directory (e.g., `gen/`), never mixed with handwritten code

#### Scripts Directory Requirement
- The repository MUST contain a `scripts/` directory with a clearly documented script (e.g., `generate_openapi.sh`). This script is responsible for running the OpenAPI code generation tool (such as `openapi-generator-cli`) and producing server/client code from `openapi.yaml`.
- The script MUST:
  - Accept parameters for server/client and language (e.g., Go, TypeScript)
  - Output generated code to a dedicated, version-controlled directory (e.g., `gen/server/`, `gen/client/`)
  - Be referenced in the developer documentation and onboarding guides

#### Review and Compliance
- All generated code MUST be reviewed for compliance with the API/spec design before merging changes.
- The OpenAPI spec and codegen workflow MUST be included in the CI process to ensure generated code is up-to-date.


## Phase 1 Endpoints Overview

The following table summarizes the core endpoints for Phase 1 of the MCP Go API. Full details, schemas, and examples are defined in the OpenAPI spec (`api/openapi.yaml`).

| Endpoint                                 | Method | Purpose/Description                                                          | Key Design Notes                                                                                   |
|------------------------------------------|--------|------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------|
| `/`                                      | GET    | List top-level resources (workspaces, OpenAPI spec, docs)                    | HATEOAS links; always include `/workspaces`, `/openapi.yaml`, `/docs`                              |
| `/openapi.yaml` or `/openapi.json`       | GET    | Serve OpenAPI specification                                                  | Must be up-to-date and versioned; `/docs` for Swagger UI/Redoc                                     |
| `/workspaces`                            | GET    | List all available workspaces                                                | Plural nouns; support pagination; document allowed characters/case sensitivity                     |
| `/workspaces/{workspace}`                 | GET    | Retrieve metadata about a specific workspace                                 | Workspace names unique; consistent error objects if not found                                      |
| `/workspaces/{workspace}/symbols`         | GET    | List all code symbols in the workspace                                       | Plural nouns; pagination; globally unique symbol IDs; schema docs                                  |
| `/workspaces/{workspace}/symbols/{symbolId}` | GET | Retrieve definition and metadata for a specific symbol                       | URL-safe, unique IDs; standard error objects if not found; example responses in spec               |
| `/workspaces/{workspace}/files`           | GET    | List all files in the workspace                                              | Plural nouns; pagination; file paths URL-encoded                                                   |
| `/workspaces/{workspace}/files/{filepath}`| GET    | Retrieve content or metadata for a specific file                             | File paths URL-encoded; careful decoding; standard error objects                                   |
| `/workspaces/{workspace}/context`         | POST   | Fetch relevant context for a code location or query                          | Complex/extensible queries; documented request schema; structured response; idempotency supported  |

---

### Versioning
- Version the API from the start (e.g., `/v1/workspaces/...` or via OpenAPI version field) to prevent breaking clients as the API evolves.
