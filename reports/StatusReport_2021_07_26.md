# Status Report, July 26, 2021
Project: Open Circuits

## This Week
This week I continued to refine the design documents for the collaborative editing feature and added a more formal, specific outline of an implementation of the Operational Transformation algorithm.  I have come up with a way to avoid code duplication of non-trivial logic between the client and server implementation by modifying the Operational Transformation algorithm.  This has implications for how document changelogs can be truncated / trimmed.

I also detail how messages between the client and server will be structured and how clients formally join the document to boot-strap invariants of the algorithm.

## Next Week
The next week, I will be diving into implementing the messaging layer, which will run on WebSockets.  This is composed of initializing and cleaning-up connections, authorizing messages, and forwarding them to the main algorithm logic.  I will also be implementing a subsystem that goes hand-in-hand with the messaging layer: the skeleton of the Operational Transformation logic, which keeps track of which documents are live and routes messages to/from client sessions and the document state thread.


## Blockers
None

## Links
New Files:
- See [designs/Algorithm.md](https://github.com/KevinMackenzie/OpenCircuitsStatus/blob/345dc28622e8ec4184f5c9f7343c5662cc4da57f/designs/Algorithm.md) for the system model and algorithm pseudocode
- See [designs/OTProtocol.md](https://github.com/KevinMackenzie/OpenCircuitsStatus/blob/345dc28622e8ec4184f5c9f7343c5662cc4da57f/designs/OTProtocol.md) for the protocol and some messaging-layer considerations

Individual commits that update existing files:
- https://github.com/KevinMackenzie/OpenCircuitsStatus/commit/f8214afd9a46bac5b9ae3c1073cda35e43cf2911
- https://github.com/KevinMackenzie/OpenCircuitsStatus/commit/1bf5886577d1fa5d1d879c9f5399d32051965483
- https://github.com/KevinMackenzie/OpenCircuitsStatus/commit/132cf46cf056056f14af9b3fbc6e3ac5468d4c2a
- https://github.com/KevinMackenzie/OpenCircuitsStatus/commit/345dc28622e8ec4184f5c9f7343c5662cc4da57f
