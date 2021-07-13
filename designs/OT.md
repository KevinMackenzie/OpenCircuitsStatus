# Operational Transformation and the new Model
The principle of OT (see [This Article](https://medium.com/coinmonks/operational-transformations-as-an-algorithm-for-automatic-conflict-resolution-3bf8920ea447) for a good primer) is to model changes to a document as applying a set of operations, or *entries*, to a state thread, or *log*.  The whole document is recreated by applying the *log* in *total order*.  Editing documents across a distributed system entails conflicts in non-trivial scenarios, which are caused by unreliable network connections and connection latency.

OpenCircuits has a graph-based model, which is different from the classical plain-text or JSON-based document models of other work in OT.  Since graphs are not ordered pieces of text, I believe it will be easier to implement.

I propose to use an intent-preserving model, which dictates how to determine a *total order* from the *partial order* of the set of entries coming from all connected clients.  When two entries are *concurrent*, there is no way to determine which one *happened before* the other.  Network latency makes wall-clock time unreliable for creating a *total order* in an asynchronous system, but there is a centralized server that all clients communicate with, so a simple clock scheme will be used that just increments at every entry (Lamport Clocks).  The timestamp is the clock value paired with the client's `SessionID`.  These timestamps are partially ordered because two clients could submit an entry with the same clock value.

The method for determining the total order could be in reverse-time-received order.  The server adds entries to the log as it receives them, but when an out-of-order entry is received, it gets added _before_ all other entries with the same clock.  This preserves the intent of the slowest client at the expense of changing what the faster clients see after-the-fact.  Not all entries will be commutative, so compromise must be made somewhere.  A maximum latency could be added so that grossly out-of-order entries are refused (i.e. >5 seconds).

## Organizing a New Model
To separate OT-based parts of the model from other parts of the model, I propose a new model broken down as follows:
1. `Schematic`: Includes all OT components
1. `Simulation`: The state of propagating signals in the system
1. `Metadata`: The non-OT components for auxilliary information

The primary goal is to have as little of the model be involved in the OT algorithm as possible, so frequent changes to the schema do not require changing the algorithm often.  Furthermore, the algorithm does not need to accumulate the document itself to successfully merge intent.

The `Schematic` a generic multi-graph where each node can have its own properties as key-value pairs.
```C
struct Schematic {
	CircuitId id;
	Set<Node> nodes;
	Set<Edge> edges;
	BidirMultiMap<(guid,fromPort), (guid,toPort)> edges;
	...otherProps
}
struct Edge {
	Guid guid;
	Guid fromNode;
	Guid fromPort;
	Guid toNode;
	Guid toPort;
	...otherProps
}
struct Port {
	Guid guid
	...otherProps
}
struct Node {
	Guid guid
	Port ports[]
	...otherProps { // All other stuff that isn't needed by OT
		Transformation transformation;
	}
}
```
The `Port`s used in wires for shaping them would be `Node`s in this model with 1 input/output.

## Log Entries
Log entry types should be *orthagonal* and *minimal* to simplify the logic used to apply them.  The application of log entries should be permissive, so failed or partially failed entries do not break the state.  **Is there an algorithm for this, or is a do-your-best approach fine?**

- AddNode
- RemoveNodes
- AddEdge
- RemoveEdges
- EditProps

To avoid the additional complexity of OT for text, `EditProps` entries are atomic.  Two people changing the number of inputs on a multiplexer, for instance, will follow the intent-preserving procedure explained above.  However, this could be mitigated if the UI for Node/Edge properties updated in real-time when `EditProps` entries are received by the client.  Text-based OT is not critical for the usability of OpenCircuits.

Additionally, `EditProps` may induce edge removals, like in the multiplexer input case, but these should not be added to the log explicitly.

A prototype schema for these transformations is
```C
struct Entry {
	union {
		AddNode,
		RemoveNodes,
		AddEdge,
		RemoveEdges,
		EditProps
	} entrySeq[]
	Size clock
	ClientID clientID
	...otherData
};
struct AddNode {
	Node node
}
struct RemoveNodes {
	Guid nodes[];
}
struct AddEdge {
	Edge edge;
}
struct RemoveEdges {
	Guid edges[];
}
struct EditProp {
	Guid object;
	Pair<String, Any> upsertProps[]
	String removeProps[]
}
```

Each entry may contain multiple entries in sequence that should be applied atomically.  We may find that this will only ever be size one.

## Managing Conflicting State
Sometimes, the intent of the clients are at odds.  Specifically, the model for digital circuits prohibits multiple inputs on a single port.  If two users connect an input to the same port, which one should win?  I propose a relaxation of the model to _allow_ these kinds of conflicting state, but perhaps prevent simulation using it.  The problematic state could be highlighted in red or something to indicate a problem until one user fixes it.  Initially, the slowest client can win, as described above, and this can be added later

## Invertability of Entries
Since multiple clients can be working on the same part of the document as eachother, the *undo* operation becomes more difficult.  Each user should get their own *undo* stack, but simply inverting their log entries and applying them may not yield good results; an even worse solution would be to replay the log with that entry removed.

Removing a node would implicitly delete some edges.  This can be solved by having *tombstones* on delete entries, but adding this to the OT algorithm could add significant complexity and make trimming the log more difficult.

I propose that the *undo*/*redo* stack lives exclusively on the client and separate from the log.  Operations are already invertable, so it may be possible for that logic to be adapted to produce a sequence of log entries instead.  If these log entries are applied atomically, the *undo*/*redo* operation is a single intent.

## Orphaned Props
I believe it possible to keep a consistent `Schematic` that does not have any orphaned `Node`s or `Edge`s, the complexity of `...otherProps` makes it possible for some entries to leave behind orphaned props.  It probably is not decideable when these props are OK to remove, but it *may not matter*.  For example, a `Port4/Color` prop may be orphaned after `Port4` is deleted and a slow client changes the color.  Since the handler to apply the `EditProp` entry to a `Node` object cannot find a `Port4`, it does not get added to the `Schematic` model.  Since log entries applied in a total order, the `Port4/Color` prop gets discarded.

## Trimming Logs
The log gets longer as more edits are made to it.  The entire log is not necessary to keep indefinitely.  On each operation, or periodically, the client sends the server the most recent entry it has received and any it is missing.  Once all clients have a log entry it can be removed.  Unfortunately, this is undecideable, so a compromise must be made.

Once all _active_ clients have a log entry, it can be removed.  An active client is one that has been participating in the algorithm recently (i.e. 5 minutes, 1 day).  This means the system model is _synchronous_.

The current `Schematic` is constructed by applying the current entries to the most recent landmark `Schematic`.  
