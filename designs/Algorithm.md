# Collaboration Algorithm
This document details the algorithm for OT to be used in OpenCicuits

The general idea is a distributed log with the following properties:
- Convergence: All copies of the log will be the same eventually
- Causality: The log has a total order
- Intent preservation: Users actions are commutative or can be transformed to preserve _intent_.

One instance of the algorithm runs per-document.  All inter-document connections (i.e. ICs) are proxied through Components in the embedding documents.

## System Model
The system is composed of a single server and any number of connected clients.  The connection between the clients is secured and authorized with a set of capabilities (view, edit, etc.?).  The connection is WebSockets, which are asynchronous, but do guarantee delivery and order.

Clients may go offline for any reason (crash, internet connection, closing) without prior warning and can engage in Byzantine behavior.  Malicious edits to the document are not Byzantine, but violations of the protocol are.

The Server will never engage in Byzantine behavior.  It is a trusted node on the network that is the authoritative source of the log.

## Log Model
The log is an ordered list of Entries, which are composed of Actions and and some metadata (clock, schema version, user ID, etc).  The behavior of the log is independent of the contents of the Actions.

The log also has a clock `logClock` which is the ID after the last log entry

## Document Model
The document is as it currently is in OpenCircuits.  Actions will be updated so they fail silently when the current document state cannot support them.

## Algorithm
This is based on the Operational Transformation algorithm, as outlined [here](https://medium.com/coinmonks/operational-transformations-as-an-algorithm-for-automatic-conflict-resolution-3bf8920ea447).

**TODO** Activity diagrams

### Invariants
- All processed entries will have been received or be en-route to all clients
- The log will not have holes
- All live clients' logs are identical (`sent` and `pending` lists will differ)

### Server
The server processes one entry at a time and the logic is simple.  It acts as a relay and establishes a total order.

```go
func serverRecv(entry ProposeAction) {
    if entry.proposedClock > logClock {
        // Didn't follow protocol (Byzantine)
        // Let the client know so it can crash & report it.
        nack(logClock)
        return
    }

    // In a concurrent request, acceptedClock != proposedClock
    entry.acceptedClock = logClock

    // Add the entry to the log
    log[lockClock] = entry

    // Acknowledge receiving the entry
    ack(lockClock)
    
    // Update the log clock
    logClock++

    // Send the entry to connected clients
    for each client except entry.clientId
        send(entry)
}
```

### Client
The client stores two additional lists separate from the Log.  The `sent` list contains entries that have been sent to the server, but haven't been ack'd yet.  The `pending` list contains entries that have not been sent yet.  The `sent` list has at most one element in it at a time.  The `pending` list can be any length.


```go
func clientAckHandler(aceptedClock) {
    // Remove from the "Sent"  list
    entry = sent.pop()

    // Apply entry to the document / log
    entry.acceptedClock = acceptedClock
    clientRecv(entry)

    // Next step
    clientSendNext()
}

// Called by app when actions are created
func clientSend(action) {
    pending.push(action)
    clientSendNext()
}

// Helper function to trigger sending the next available pending action
func clientSendNext() {
    if len(sent) > 0 {
        return
    }
    sent = pending.pop()

    // Increment the log clock when we SEND only
    send(sent, logClock++)
}

// The client will receive entries in real-time
func clientRecv(entry) { 
    // Transform the entry against everything it hasn't seen
    entry = transform(entry, log[entry.proposedClock:])

    // The below code should be atomic so the UI isn't weird

    // Revert document back to log-state
    doc.apply(reverse(invert(pending)))
    doc.apply(reverse(invert(sent)))

    // Add entry to the log
    log.append(entry)
    logClock = entry.acceptedClock

    // Apply new log entry
    doc.apply(entry)

    // Apply local changes again, transformed this time
    doc.apply(transform(sent, entry))
    doc.apply(transform(pending, entry))
}
```

### Connection Failure
While WS/TCP guarantees order / deliver of packets, the connection may time out or disconnect for some other reason.  In these cases, the WS connection is broken and the client must re-establish the connection, at which point it will receive all log entries that it has missed.

## Possible Optimizations
To start, the client will only have one copy of the document and the entire `pending` / `sent` list is undone, the new log entries are applied, and then then redone atomically (without updating the UI).  This could be slow or janky, so storing a second document that only contains the current Log state and re-building the working document from half-way in could help