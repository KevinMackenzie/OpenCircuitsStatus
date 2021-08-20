# Status Report, August 13, 2021
Project: Open Circuits

## This Week
This week I have continued to write tests for the front-end logic, and started writing end-to-end tests of the entire operational transformation framework.  This starts a new server process for each test and connects to the correct one based on port number since the typescript test framework is parallel.

One challenge was serializing/deserializing the messages between client and server.  Without a dedicated messaging layer like protobufs, explicit type information needed to be added to the serialization for the front-end to correctly reconstruct messages.  The back-end does not need to fully parse the messages.  My temporary solution is similar to the in-house serialization framework, but lighter weight, and compatable with the standard JSON deserialization used on the back-end.

On the back-end, I fixed a bug related to session/document messaging and a bug causing a double-close of a channel (which crashes the server).

## Next Week
Next week, I hope to continue working with the project owner on the refactor.

I will work on a finding a message format that is best suited to the technology used.

I will also be adding a new message type and back-end support for broadcasting per-client state to other clients so each user can see what the others are working on.

Finally, I would like to complete the integration between the back-end code that manages user sessions and the code that manages sharing permissions.

## Blockers
Implementation of the operational transformations on the real model logic is waiting on the model refactor.

## Links
Commits on [Operational transformation](https://github.com/OpenCircuits/OpenCircuits/commits/operational_transformation) code (most recent first)
Note: These may be rebased and split into branches for review-ability when complete.

---
https://github.com/OpenCircuits/OpenCircuits/commit/87ab3bf37249e0a844d23a19871f8138a25e3c61

    Added client/server tests
    
    Fix the WebSocket client to work correctly with the event-driver WS library.

---
https://github.com/OpenCircuits/OpenCircuits/commit/64d581a2e622aba566d48eff21c226abea299be7

    Fixed channel double-close error


https://github.com/OpenCircuits/OpenCircuits/commit/0ea5986bb28a5a3912b3fe8c901161282cfd581f

    Removed SessionID from entries
    
    The changelog does not need to know about session information.

---
https://github.com/OpenCircuits/OpenCircuits/commit/33ed7d0be409c93ed3d48fbf8abfec1fbee2cd80

    Made Client/Server Protocol Consistent
    
    There are some on-going issues with how to serialize the message types so they can be read easily by the server and the client.  This commit makes the client and server compatible, but is not the best solution

---
https://github.com/OpenCircuits/OpenCircuits/commit/7909f43d10971cc9708d2ce2191632109ea34e8b

    Fixed assertions and added default values
    
    Updated from legacy assert function and fixed assertion failures caused by missing default values in proposed/accepted entries.

---
https://github.com/OpenCircuits/OpenCircuits/commit/8969d21f2b63f442c20cf3ed4382f72e681ee9e3

    Revert "Added GUIDs to wires/ports/components"
    
    This reverts commit 2c1c58dbd04a5e64eb83b784bc71785bff96800a.

---
https://github.com/OpenCircuits/OpenCircuits/commit/fe5047b4ebe0bf220d0c9f537487873f8b500705

    Revert "Hack to serialize references"
    
    This reverts commit 4b3193f14afb75f9cd24ffd092333eb7f9eaf441.

---
https://github.com/OpenCircuits/OpenCircuits/commit/b5770da6993d4f77a1fea949da44f17ba1a9e4d3

    Fixed import spacing style

---
https://github.com/OpenCircuits/OpenCircuits/commit/f2198f7ce73d233e7cd108b6e7e4284136a6cd9d

    OTDocument.SendNext tests

---
https://github.com/OpenCircuits/OpenCircuits/commit/cf005a3ccd7b757a5adf3a8b525d33cbd7799134

    Removed vestigial "joined" flag
    
    This behavior got absorbed into the `WebSocketConnection` class and the `Connection` interface has the responsibility of a reliable connection.

---
https://github.com/OpenCircuits/OpenCircuits/commit/1301f20e667ba5e179c0b17f45c803431e873df5

    Split Recv, made plural, and ProposedCache uses Actions
    
    Now Recv only handles remote new entries, since ack entries don't need to be transformed, just added to the log and removed from the ProposedCache.  The ProposedCache now uses just actions, since it doesn't actually use any of the ProposedEntry fields.
    
    Several bugs fixed in the ProposedCache:
    - Usage of Push is uniform with revert/xform
    - Keep track of the "success" of sent so its preserved in the log on ack