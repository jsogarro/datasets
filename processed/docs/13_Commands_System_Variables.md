# Chapter 13: Commands and System Variables - Q for Mortals

## Overview

This chapter covers three main categories of q environment controls:

1. **Commands** - Interactive directives starting with backslash (`\`)
2. **Command-line parameters** - Startup options prefixed with `-`
3. **System variables** - Environmental information in the `.z` namespace

## Key Command Categories

### Workspace Inspection
- `\a` - List tables in current/specified context
- `\b` - View aliases and dependencies
- `\f` - Display functions
- `\v` - Show variables
- `\w` - Report workspace resource usage

### Configuration Commands
- `\c` - Set console display size (default: 25 80)
- `\d` - Change working context/directory
- `\o` - Adjust GMT offset (hours if <24, minutes if ≥24)
- `\p` - Open/close listening port
- `\P` - Set float display precision (0-17 digits)
- `\t` - Configure timer interval or profile expression duration
- `\T` - Set remote execution timeout in seconds
- `\z` - Toggle date parsing format (0=mm/dd/yyyy, 1=dd/mm/yyyy)

### File/Database Operations
- `\l` - Load scripts, serialized entities, or database directories
- `\cd` - Change OS working directory

### Advanced Features
- `\e` - Toggle error trap behavior for client requests
- `\g` - Control garbage collection (memory ≥64MB returned to OS if enabled)
- `\S` - Set random seed for reproducible results
- `\s` - Display available slave count (read-only)
- `\u` - Reload password file
- `\W` - Set week offset (0=Saturday default=2 Monday)
- `\_` - Lock/scramble script files
- `\1`, `\2` - Redirect stdout/stderr
- `\\` - Exit q process

### Session Control
- `Ctl-c` - Interrupt long-running functions
- `Ctl-z` - Terminate session with prejudice
- `\` - Resume after error/toggle k interpreter
- `\x` - Restore default behavior (expunge handler)

## Command-Line Parameters

Common startup options:
- `-b` - Block client database modifications
- `-c` _r c_ - Console size (rows, columns)
- `-e` 0|1 - Enable/disable client error trapping
- `-g` 0|1 - Garbage collection mode
- `-p` _port_ - Listen on specified port
- `-P` _digits_ - Float display precision
- `-q` - Quiet startup (no banner)
- `-s` _N_ - Configure N slave processes
- `-t` _ms_ - Timer tick interval
- `-T` _secs_ - Remote execution timeout
- `-u` _file_ - Load password file (restricted access)
- `-U` _file_ - Load password file (unrestricted access)
- `-w` _bytes_ - Maximum workspace size
- `-W` _offset_ - Week start offset
- `-z` 0|1 - Date format parsing

## Critical System Variables (.z namespace)

### Time/Date Information
- `.z.p` - Current GMT timestamp (nanosecond precision)
- `.z.P` - Current local timestamp
- `.z.t` - GMT time component
- `.z.T` - Local time component
- `.z.d` - GMT date component
- `.z.D` - Local date component
- `.z.n` - GMT timespan
- `.z.N` - Local timespan

### System Information
- `.z.a` - IP address (encoded int)
- `.z.c` - Core count
- `.z.h` - Hostname
- `.z.i` - Process ID
- `.z.k` - Release date
- `.z.K` - Version number (float)
- `.z.l` - License information
- `.z.o` - Operating system identifier
- `.z.u` - Current user ID
- `.z.w` - Connected handle (0 at console)
- `.z.f` - Startup file path

### Event Handlers (Assignable Functions)
- `.z.exit` - Called on process shutdown
- `.z.pc` - Connection closed
- `.z.pd` - Peach distribution handles
- `.z.pg` - Synchronous remote message
- `.z.ph` - HTTP GET request
- `.z.pi` - Console input echo
- `.z.pm` - HTTP OPTIONS method
- `.z.po` - Connection opened
- `.z.pp` - HTTP POST request
- `.z.ps` - Asynchronous remote message
- `.z.pw` - Password validation
- `.z.ts` - Timer tick
- `.z.vs` - Global variable assignment
- `.z.wo` - WebSocket opened
- `.z.wc` - WebSocket closed
- `.z.ws` - WebSocket message

### Command-Line Access
- `.z.x` - Argument list (strings after filename)
- `.z.X` - Raw unprocessed command line

### Metadata
- `.z.ac` - HTTP SSO cookie processing
- `.z.b` - Direct dependencies dictionary
- `.z.q` - Quiet mode flag (read-only)
- `.z.s` - Current function (for recursion)
- `.z.W` - Output queue information (handles)
- `.z.zd` - Compression settings (block size, algorithm, level)

## Important Distinctions

**Never** use `value "\..."` for system commands in production—it creates security vulnerabilities. Use the `system` function instead for validated command execution.

Memory management notes: Q maintains thread-local heaps with reference counting (no garbage collection). Objects ≥64MB are returned to OS if `\g` is enabled; smaller objects return to heap for reuse.
