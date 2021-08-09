# Status Report, August 07, 2021
Project: Open Circuits

## This Week
This week, I have been working with the project owner on a major refactor of the front-end model logic to improve code organization and meet the needs of the operational transformation algorithm.  The goals are:
1. Very simple serialization
1. Feature parity with existing code
1. Maximize shared code for current Digital circuits and future Analog circuits
1. Easy to expand / maintain

We achieve (1.) by completely isolating the editable circuit model data from its simulation state and by using GUIDs to reference objects to avoid cycles in the object graph.  The tentitive plans for (3.) are to use parametric polymorphism extensively to avoid unsafe downcasts in the current subclass-polymorphic model.  The other points should follow from careful implementation.  Work up to this point has been pseudocode in google docs to address both broad-strokes and the motivations of normal or exceptional individual components.

For code-related work, I have implemented a proof-of-concept for the front-end operational transformation logic.  It's split into a few layers with ease of testing in mind, which is mostly for regression because this code should rarely need changing after complete:
1. A strongly-typed connection layer, with a websocket / mock implementation.
1. An connection wrapper, which translates message events to document logic calls.
1. A Changelog component for (future) log trimming and clock management.
1. A core OTDocument component that implements the high-level logic for client-side operational transformations.

In the process of implementing the front-end logic, I have made the protocol between the client and server separated explicitly on the back-end so future changes to the protocol are easier to accomplish (i.e. viewing what other users have "selected").

I have begun writing tests for the front-end logic.  In the effort of having end-to-end testing of the OT logic without the complexity of the full model, I have implemented two non-trivial models.  One is just an array of numbers, with insert index / delete index operations.  The other is more complex: plain-text editing.  I intend to use these to test the back-end and the generic parts of the front-end before the front-end model refactor is complete.

## Next Week
Next week, I will continue working with the project owner on the refactor and will continue to write unit tests for the operational transformation logic on the front-end.

I will also work on getting an end-to-end testing environment set up for the generic operational transformation logic on the front-end and the back-end with multiple connections and concurrency control.  The exceptional cases cannot be reliably tested without specific tooling like this.

As a stretch-goal, I will have a working bare-bones front end with plain-text editing to use as an open demo during the presentation that the audience can connect to.


## Blockers
Implementation of the operational transformations on the real model logic is waiting on the model refactor.

## Links
Commits on [Operational transformation](https://github.com/OpenCircuits/OpenCircuits/commits/operational_transformation) code (most recent first)
Note: These may be rebased and split into branches for review-ability when complete.

https://github.com/OpenCircuits/OpenCircuits/commit/ed1ef2df27596aa3c5b892f22a454a6e2715e804

    Added success flag to client log
    
    This will be used to determine if an entry can be "undone"

---

https://github.com/OpenCircuits/OpenCircuits/commit/gaee93bdee352a9abd59c761218e85cd657c0499f

    Tabs to spaces

---
https://github.com/OpenCircuits/OpenCircuits/commit/g4823e86948b4520514ad1c2a0753983dd8ac77e8

    Added tests for ProposedCache

---
https://github.com/OpenCircuits/OpenCircuits/commit/g634e1557a113c6d6af6dafbb35080f50570e115a

    Removed unnecessary abstraction from connection

---
https://github.com/OpenCircuits/OpenCircuits/commit/ge37af0176142bd4d7b419e7c3e87f009e011d6ea

    Implemented connection for WS
    
    This includes a wrapper that abstracts the unreliable nature of the WS connection into a reliable interface.

---
https://github.com/OpenCircuits/OpenCircuits/commit/gd0d94cefe5e3a543b421b66ac08d7a767cc82089

    Made logic more testable
    
    I separated the different layers of the document class so they can be independently tested and still only expose the necessary features to the rest of the app.

---
https://github.com/OpenCircuits/OpenCircuits/commit/g8c2396c377b0d7feba358117f785ebfb3625bc0a

    Moved transformation as external function
    
    To avoid a complete graph of dependencies among the actions, I moved the transformation function to be external to the individual actions.

---
https://github.com/OpenCircuits/OpenCircuits/commit/g8501837507393cb0fbe80ba6be0b97b62cb05b73

    Fixed Changelog generic

---
https://github.com/OpenCircuits/OpenCircuits/commit/g1a1884e55b514b6f1f65cb62421aec71224187cd

    Added simple Text/HashMap OT test models

---
https://github.com/OpenCircuits/OpenCircuits/commit/gb2d3e8ff7bfc7384f757d735a7192a0a7ffd6ab8

    Improved generic Action generics

---
https://github.com/OpenCircuits/OpenCircuits/commit/gc0b1506645adc9583b84ac93b791b3a8e30dc567

    More tests and Serialize returns error
    
    serializing protocol types from server->client was a soft error case, but now it correctly returns an error, which propagates to the call site

---
https://github.com/OpenCircuits/OpenCircuits/commit/g80acd733060a8d5615f215a3c3e48049f037a9d6

    Tests & bugfixes in client/session protocol

---
https://github.com/OpenCircuits/OpenCircuits/commit/g395fcf0007880232d4c82bb589a168643a99999b

    Removed unnecessary casts

---
https://github.com/OpenCircuits/OpenCircuits/commit/gb08549fd21e9850b1fa5e65ef4065d369229e773

    Revised connection / protocol organization

---
https://github.com/OpenCircuits/OpenCircuits/commit/gf8baade665472def69bbb2efcd75a419b42bd47b

    Fixed document_test.go

---
https://github.com/OpenCircuits/OpenCircuits/commit/gd9094cc59e4a409294889038dbdff561ad079591

    Action has apply rather than doc
    
    This better represents how they will actually be implemented.
