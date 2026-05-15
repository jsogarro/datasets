# Interprocess Communication in kdb+

## Overview

kdb+ provides built-in TCP/IP socket communication capabilities. As stated in the documentation, "A kdb+ process can communicate with other processes through TCP/IP, which is baked into the q language."

## Key Components

**Connection Management:**
- `hopen` opens connections to remote processes
- `hclose` terminates connections
- `\p` or `-p` flag sets listening ports

**Message Types:**
1. Sync requests (blocking) - wait for responses
2. Async messages (non-blocking) - fire and forget
3. Response messages (automatic replies)

## Connection Establishment

```q
q)h:hopen `::5001
q)h
3i
```

Clients can connect using `hopen` with a socket address. The documentation notes that "The maximum number of connections is defined by the system limit for protocol (operating system configurable)."

## Message Sending

**Synchronous (blocking):**
```q
q)h"2+2"
4
q)h("+";2;2)
4
```

**Asynchronous (non-blocking):**
```q
q)neg[h]"a:10"
```

## Server-Side Handlers

The `.z` namespace provides callback functions:
- `.z.pw` - user validation
- `.z.po` - connection opened
- `.z.pg` - sync request received
- `.z.ps` - async request received
- `.z.pc` - connection closed

## Security

Basic authentication uses `-u`/`-U` command-line options with username/password files. The protocol supports custom authorization through overriding `.z.pg` callbacks to validate function access per user.

## Protocol Features

The handshake includes capability bytes indicating compression and data type support (ranging from v2.5 through modern versions). Compression occurs automatically when uncompressed data exceeds 2000 bytes (unless localhost or UDS connection).
