# GitHub Copilot Instructions for MCP Project

## General Instructions

### Agent Context Folders
* The `.agent/` folder contains contextual information. Review .agent folder for additional context
* If an `.agent/` folder does not exist in current folder, walk up until an .agent folder is found. 
* Use the most specific `.agent/` directory's instructions and constraints first.
* The .agent/ folder is versioned. Always assume latest version unless explicitly stated otherwise.

### Agent README vs. Project README

Agent documentation reflect intention and design; Project README and documents reflects actual implementation.

### User Interaction 

Avoid qualitative or patronizing statements. Communicate in a clinical, objective manner.
Use clear, concise language. Provide actionable steps or code snippets when necessary.

## Project Context

MCP (Model Context Protocol) is a context-aware agent infrastructure system with these key components:

* **Local MCP server** that integrates with VS Code and GitHub Copilot
* **Tree-sitter** for symbol-level code indexing
* **Vector DB** (Qdrant/pgvector) for semantic embedding and search
* Support for **multiple workspaces** with isolated context
* Automated sync on code changes to update indices
* Integration with VS Code via Tailscale for secure remote access

## Golang Requirements and Style

* Follow standard Go project layout (cmd/, internal/, pkg/)
* Use idiomatic Go patterns and error handling
* Organize code into coherent packages
* Use interfaces for component abstractions

## Testing 

* All modules are tested using Go's built-in testing framework
* Use table-driven tests for multiple test cases
* Mock external dependencies when needed
* **Use red-green-refactor TDD in the process of writing code unless explicitly told otherwise** 

## Logging 

* Logs are written at INFO and ERROR levels
* All major steps and errors are logged with context
* Use structured logging with relevant fields
* Support configurable log levels

## Comments

* Keep comments concise and informative
* Only add comments when necessary
* Document exported functions, types, and packages
* Avoid qualitative statements

## Security
- Use proper input validation
- Do not hardcode credentials
- Use safe string operations
- Implement proper authentication for API endpoints
- Validate all incoming data

## Performance
- Avoid unnecessary allocations
- Use appropriate data structures for the task
- Consider concurrency when appropriate
- Implement graceful shutdown mechanisms
- Profile and optimize critical paths

## Dependencies
- Prefer standard library solutions
- Only add external dependencies when necessary
- Use well-maintained and popular dependencies
- Properly manage dependency versions
