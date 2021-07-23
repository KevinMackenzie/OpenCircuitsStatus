# Operational Transformation and the new Model
This is a high-level document with considerations for the design of the OT subsystem.

The principle of OT (see [This Article](https://medium.com/coinmonks/operational-transformations-as-an-algorithm-for-automatic-conflict-resolution-3bf8920ea447) for a good primer) is to model changes to a document as applying a set of operations, or *entries*, to a state thread, or *log*.  The whole document is recreated by applying the *log* in *total order*.  Editing documents across a distributed system entails conflicts in non-trivial scenarios, which are caused by unreliable network connections and connection latency.

OpenCircuits has a graph-based model, which is different from the classical plain-text or JSON-based document models of other work in OT.  Since graphs are not ordered pieces of text, I believe it will be easier to implement.

I propose to use an intent-preserving model, which dictates how to determine a *total order* from the *partial order* of the set of entries coming from all connected clients.  When two entries are *concurrent*, there is no way to determine which one *happened before* the other.  Network latency makes wall-clock time unreliable for creating a *total order* in an asynchronous system, but there is a centralized server that all clients communicate with, so a simple clock scheme will be used that just increments at every entry (Lamport Clocks).  The timestamp is the clock value paired with the client's `SessionID`.  These timestamps are partially ordered because two clients could submit an entry with the same clock value.

The entry received first for the log slot will be chosen first, then the second will be transformed against the first entry before being added immediately after.  This generalizes to any number of changes.  Luckly, most operations commute so this transformation will be simple.

## The New Model
### The Log
The current model is non-trivial and is already reasonably-well setup for integrating into OT.  The OT code should not be brittle to changes in the model schema.  Log entries should be generic and compose Actions.  Each log entry will have a schema version number attached to it, so when schemas change, full migrations are not required.

Actions cannot fail.  They must be very permissive.  All operations must serialize to primitive types (GUID's, not objects), so they can be transmitted uniformly.

_Actions_ are the operations applied to the document.  _Entries_ are the representation stored in the log.  _Entries_ have more information than _Actions_ do.

For inter-op, this would be represented something like:
```Go
type Action struct {
	actionType int
	data []byte
}

type LogEntry struct {
	clock uint32
	clientId uint32 // Assigned by the server
	schemaVersion uint32
	action Action
}
```

Actions here are singular, but GroupActions should be atomic and count as one Action in the log.


### The Model
The model will remain largely the same, but each distinguishable object gets GUID to identify it uniquely within the circuit in Log Entries.

Actions will become complete permissive.  When deserializing an Action from an incoming log entry or applying an action with objects, if any of the GUIDs or any of the object's GUIDs are no longer in the circuit, the action fails.  **TODO**: Should the entire action fail atomically (GroupActions) or should each part be attempted?

One thing to consider is whether ports get GUID's, or if ports are just indexes on Components' GUID's.  The former works well because it won't ever connect a wire to a port that was deleted, or changed.  For IC's, if additional ports are added the wires will never connect to the wrong ports.

## Server Logic
When two operations are concurrent, the rule described above is that the _first_ entry received is accepted.  The server is really just a relay and an authoritative clock source.  It _may_ require some logic for transformations.  The server should have as little information about the model as possible to avoid code duplication.

## Client Logic
The client has to do its best to preserve its own intent while also abiding by the authoritative server source.  The client has two lists: _Pending_ for changes which have not been sent to the server, and _Sent_ for changes which have been sent but not ACK'd.  The rest of the details can be found in [Algorithm.md](Algorithm.dm)

## Transformations
The _Transformation_ of Operational Transformation is because text-based operations can affect the indices of later and yet-unknown operations, so entries have to be transformed against accepted changes to preserve the users' intents.  OpenCircuits is a canvas-based editor without a substantial textual component, so the problem of index-based text operations is not important.  All text operations can be sets, as opposed to inserts and deletes.

The geometric transformations _should not_ compose.  It would be very confusing if two people moved the Component in the same direction and it shot off twice that distance.  So, similar to text operations, geometric transformations should be sets.

Changing the number of inputs to a mux, encoder, etc. is a simple property set operation.  It wouldn't be desirable for this to be the composition of two events.

The analog to insert/delete from the text-based editing example is adding/deleting wires / components.  These are operations that spawn independent objects into existance and delete them.  These operations are absolute because their intent is not affected by properties of other components and wires, so no transformation is required.

Based on these three examples, actions that are _absolute_ do not require transformations while actions that are _relative_ do require transformations.  Furthermore, only non-desetricture _relative_ actions require transformations. Relative actions are:
1. SplitWireAction: The user clicks on a wire and drags, which splits the wire into two and inserts a port in the middle.
	- Transformation: If the parameter on the wire curve for the second event is after that of the first event, then change the second event to use the newly created wire from the first event.
2. SnipAction: The user deletes a port and merges the two wires together.
	- Transformation: If the event is on the deleted wire, move it onto the other wire, adjusting parameter as requird.

## Conflicting State?
The model for digital circuits prohibits multiple inputs on a single port.  If two users connect an input to the same port, which one should win?  I propose a relaxation of the model to _allow_ these kinds of conflicting state, but perhaps prevent simulation using it.  The problematic state could be highlighted in red or something to indicate a problem until one user fixes it.  Initially, the slowest client can win, as described above, and this can be added later

## Invertability of Entries
Since multiple clients can be working on the same part of the document as eachother, the *undo* operation becomes more difficult.  Each user should get their own *undo* stack.  Rather, when a user performs "undo", it inverts the users' Actions and appends them to the log.  Actions in the _Pending_ list are just removed before the server gets them.

Removing a node would implicitly delete some edges.  This can be solved by having *tombstones* on entries.  Generalized, tombstones are on all destructive actions and can help numerical precision problems with geometric transformations.  These already exist on Actions.

Sometimes, undoing an operation may bring back zombie features, like wires connected to nothing.  I propose that Actions return a success/failure result of whether anything actually happened.  If nothing happens as a result of inverting the action, then it should not be sent to the server.  The next history item is undone / redone until it is empty or one is at least partially successful.

## Trimming Logs
The log gets longer as more edits are made to it.  The entire log is not necessary to keep indefinitely.  On each operation, or periodically, the client sends the server the most recent entry it has received and any it is missing.  Once all clients have a log entry it could be removed.  However, this is undecideable and we want to keep a certain amount of log so that users' undo can be used even if the local changes are deleted.  We don't need to be in a hurry to delete log entries.

For example, log entries that are over a week old are deleted and entries over 2000 are deleted -- Of course, only if they have been also applied to the most recent milestone document.

## In-Progress Actions
Some actions have an in-progress view on the client and are not in the history stack.  Transforming components is the most obvious of these.  It would be cool if these in-progress actions showed up on collaborator's screens before the action is completed.  If, with rate limiting, these progress actions are added to the log, this is possible.  

Each incremental action could be non-invertable and ignored in the history stack, but the final Action would have the tombstone used to undo/redo the entire rotation at once.

Two users transforming the same component would cause fighting, but that isn't a problem we need to solve.

## Offline editing
Offline editing will work perfectly fine.  The _Pending_ list will grow long as the user edits the circuit.  If this is stored in _LocalStorage_, or the modern equivalent, it can be loaded up and sent to the server when the client reconnects.

It is OK for the client to be very out-of-date.  It may be the case that most of their operations got destroyed in the mean-time, but independent changes won't be affected.  Just a thought: For drastically diverging files, it could be nice to group sets of actions geometrically and offer a "keep mine, keep remote, merge" option to the user when just merging the changes could create a mess.