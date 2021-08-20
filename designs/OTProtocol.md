# WebSocket Protocol
This document details the protocol for websocket connections.

The interface between clients and the server will be over WebSockets.  This interface will contain two main types of information:
1. Operations / Actions on the document: the collaboration algorithm
1. Updates on user activity: users can see what each other have in-focus (like in google docs with selected text or cursor position)

There are two candidates for the data format:
1. JSON: Easy to read, trivial to serialize, composes well
1. Protobufs; Smaller format, could be added later

JSON will be used to start for ease of implementation.

## Base Collaboration Algorithm Messages
see [Algorithm.md](./Algorithm.md) for the details on the algorithm

The OT algorithm defines a message from clients to servers that contain an action proposed for the next log slot.  By adding an ACK component to the message, explicit acknowledgements are not required in high throughput scenarios and filling missing log entries on clients can be done efficiently.

```C
// Message from the client to the server
struct ProposeAction {
    struct {...} Action; // External, just bytes
    uint64_t ProposedClock; 
    // uint64_t ClientID; // Injected
}

struct Entry {
    ProposeAction action;
    uint64_t AcceptedClock; // The clock in the log
    uint64_t ClientID;
    string   UserID;
}

struct AckProposeAction {
    uint64_t AcceptedClock; // The clock in the log
}

// Message to the client that they failed to follow protocol
struct NackProposeAction {
    // The latest clock ID in the log
    uint64_t LogClock;
}
```

## User Interface Synchronization
The user interface should synchronize between multiple clients so they can see what each other are doing.  This can be done using the current Action infrastructure for actions that do not affect the document itself.  This may require some changes so remote clients don't open dialog boxes, but elements like highlighting wires, Components, etc will show up.


Alternatively, the current _state_ of other's user interface can be transmitted as it changes.  This means that individual Actions are not required and no accumulate operation is used.  This would have information like:
- Selected Objects (could be empty)
- UserID or different anonymous users, but NOT the ClientID/SessionID which is secret
- Others?

## Additional Entry Information
The `ProposeAction` message should contain a `SchemaVersion` so the server can reject messages from clients running out-of-date code.

## Joining the Network
Clients join the network by establishing the WS connection as outlined in [DesignDoc.md](./DesignDoc.md).  Upon connecting, the client will send a message to the server with the latest clock it knows about and the server will respond with all entries that have been missed.  The server does not register the client as "live" and start sending it messages until the `JoinMessage` has been received to maintain the invariant that all live clients' logs are identical.

```C
// Registers connection / session as a live client with the server
struct JoinMessage {
    uint64_t LogClock;
}

// Server sends the client all new entries
struct WelcomeMessage {
    Entry entries[];
}
```

![Creative Commons License](https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png) This work is licensed under a http://creativecommons.org/licenses/by-nc-sa/4.0/ Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License.