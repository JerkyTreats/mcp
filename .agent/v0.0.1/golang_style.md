# Golang Requirements and Style

Assume latest Go version. 

## Agent 

* The `.agent` folder contains contextual information. Review .agent folder for additional context
* If a `.agent` folder does not exist in current folder, walk up until an .agent folder is found. 


## Testing 

* All modules are tested using Go's built-in testing framework.
* Use red-green-refactor TDD in the process of writing code. 

## Logging 

* Logs are written at INFO and ERROR levels. 
* All major steps and errors are logged with context. 

## Comments

* Keep comments concise, and informative
* Only add comments when necessary
* Avoid qualitative statements.
