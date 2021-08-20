# Final Contribution Summary
Project: OpenCircuits

My primary contributions are for building a back-end that supports circuit sharing features and collaborative editing via operational transformation (OT).

##  Miscellaneous Contributions
### [Security bug fix](https://github.com/OpenCircuits/OpenCircuits/pull/670)
This back-end bug allowed malicious actors with 3rd party google JWTs to impersonate an OpenCircuits user.
### [Copy-on-save bug fix](https://github.com/OpenCircuits/OpenCircuits/pull/670)
This bug caused saving a circuit to sometimes create a duplicate rather than update the original

### [Feedback on boolean expression-parser](https://github.com/OpenCircuits/OpenCircuits/pull/630#pullrequestreview-669494926)
Early in the semester, I reviewed a PR and gave feedback for a feature that allows users to generate circuits from boolean expressions.

## Process Contributions
### [Top-level Issue Tracker](https://github.com/OpenCircuits/OpenCircuits/issues/677)
Living issue with high-level features and PRs

### [Back-end Refactor Draft PR](https://github.com/OpenCircuits/OpenCircuits/pull/674)
This PR was created to try and create incremental changes to review, but marked as draft since the requirements weren't clear enough at the time to be worth separating.

## Design Contributions
I have design docs in a separate repository:
### [Initial design docs](https://github.com/KevinMackenzie/OpenCircuitsStatus/blob/master/designs/DesignDoc.md)
A first-draft of the back-end architecture and considerations for operational transformation.

### [Initial OT discussion](https://github.com/KevinMackenzie/OpenCircuitsStatus/blob/master/designs/OT.md)
A high-level description of OT and considerations specific to the OpenCircuits document model and user interactions.

### [Formal Algorithm Description](https://github.com/KevinMackenzie/OpenCircuitsStatus/blob/master/designs/Algorithm.md)
A formal description of the OT algorithm components complete with client and server pseudocode.  

### [First-draft Protocol](https://github.com/KevinMackenzie/OpenCircuitsStatus/blob/master/designs/OTProtocol.md)
An initial draft of the protocol between the client and server, including how clients join/leave the algorithm and the messaging format.

## Code Contributions
My contributions / progress on code are:
- Access [model](https://github.com/OpenCircuits/OpenCircuits/blob/operational_transformation/src/server/core/model/access.go) and [api](https://github.com/OpenCircuits/OpenCircuits/blob/operational_transformation/src/server/api/routes/access/handlers.go) for user permissions
- [Document layer for core OT on server, sending updates to sessions, and notifying clients when others join the algorithm.](https://github.com/OpenCircuits/OpenCircuits/tree/operational_transformation/src/server/ot/doc)
- [Session layer for facilitating client permission enforcement and connection lifecycle](https://github.com/OpenCircuits/OpenCircuits/tree/operational_transformation/src/server/ot/session)
- [Protocol between client / server](https://github.com/OpenCircuits/OpenCircuits/tree/operational_transformation/src/server/ot/conn)
- [OT algorithm, reliable connection abstraction, protocol implementation, and document abstraction on client](https://github.com/OpenCircuits/OpenCircuits/tree/operational_transformation/src/app/core/ot)

## Future Work
- OT Document storage: do not currently persist across server restarts, so I am creating storage abstractions and moving some code around to support this.
- User permission storage: only has an in-memory driver, so dev/prod storage drivers need to be written.
- Log trimming: changelogs are never trimmed.  Rules are described in design docs, but needs implementing
- [Milestone Circuits](https://github.com/KevinMackenzie/OpenCircuitsStatus/blob/master/designs/DesignatedUpdater.md):
To avoid storing the entire history of the circuit, milestones can be created and the changelog gets trimmed.  This doc outlines a solution to this problem within the algorithm I implemented.
- Integration with refactored circuit model (waiting on refactor to commence)

Create a "staging" branch for this really big feature so individual contributions can be made via PR for easier reviewing without breaking master.

## Commit History
Listed below are all the code commits I made this semester:

https://github.com/OpenCircuits/OpenCircuits/commit/commit/be7fb34bd3d90f6f009c8a98b9c28b0f2316c3ab
Date:   Thu Aug 19 18:35:23 2021 -0400

    Updated go.mod and go.sum with uuid

---

https://github.com/OpenCircuits/OpenCircuits/commit/a057d1d9f60dc26e95f6bdf4b2e2289e67fcaf83
Date:   Thu Aug 19 18:34:11 2021 -0400

    Added recovery to session / document

---
https://github.com/OpenCircuits/OpenCircuits/commit/919921b05bdceefb4fd68ef54dccc229242cecd9
Date:   Wed Aug 18 22:33:30 2021 -0400

    Switched to UUIDs
    
    Circuit / Link / Session IDs now use type 4 UUIDs formally.

---
https://github.com/OpenCircuits/OpenCircuits/commit/00a69e00716b5b4d2614bbed2d8036210ab31361
Date:   Wed Aug 18 20:58:27 2021 -0400

    Switched access expiration to unix timestamps

---
https://github.com/OpenCircuits/OpenCircuits/commit/be232f2c9411eb10cb0e445fe2611c76c7911d44
Date:   Wed Aug 18 20:44:10 2021 -0400

    Improved access convention and fixed bug
    
    The default value for the permission struct is either: not valid because of user/circuit/link id being blank; no permission, because the access level is 0.  Hence, there is no reason to have a pointer return type to indicate the "not found" case.  I updated the AccessDriver to reflect this
    
    I also fixed a bug in the model logic that would allow a low-permission user to demote a high-permission user.

---
https://github.com/OpenCircuits/OpenCircuits/commit/da087a19cfd38ec8395bf044006be39313e0c77c
Date:   Wed Aug 18 17:34:32 2021 -0400

    Revamped session launch for access checks
    
    I reclassified the SessionManager as just the Launcher.  It launches sesssion for both circuit-id (user) sessions and link-id sessions.  This handles not-found errors at the http level and authentication / access errors at the WS connection level.  Since auth is done in the Join message, I added a preliminary session state that only listens for join messages, which gets promoted to a full session upon success.  The full session now no longer services join messages and checks for permissions before proposing entries or sending updates.

---
https://github.com/OpenCircuits/OpenCircuits/commit/50b97a024e9af8cb624cc09c46bae2c14fe25c52
Date:   Wed Aug 18 17:20:23 2021 -0400

    Add base type for access checks
    
    base type used for common access checks among link and user sharing types

---
https://github.com/OpenCircuits/OpenCircuits/commit/91508f8864312ab5d9c55e8735b07b763ed63d57
Date:   Wed Aug 18 17:19:20 2021 -0400

    Added ExtractIdentity auth helper function
    
    packages the two common methods into one.

---
https://github.com/OpenCircuits/OpenCircuits/commit/a008d705b1fb9c7a97d82c516662254a537b6344
Date:   Wed Aug 18 10:03:33 2021 -0400

    Update manager_test.go

---
https://github.com/OpenCircuits/OpenCircuits/commit/972cc0aaf3037c07a3a4de84a81319722874e5c4
Date:   Wed Aug 18 10:01:34 2021 -0400

    Removed locks from SessionManager
    
    It turns out the session manager does not need to store a map of session references for anything.

---
https://github.com/OpenCircuits/OpenCircuits/commit/092e237f8736bad5363b6592591d52d14b0ebe13
Date:   Wed Aug 18 10:00:27 2021 -0400

    Switched to lock-less DocumentManager
    
    This is managed using channels.  While a RW lock _may_ have higher availability, the channel method is simpler and can still support parallelism if needed.

---
https://github.com/OpenCircuits/OpenCircuits/commit/71edeb3fdb18c1b3f501d2e7194a2d611e92ea3d
Date:   Tue Aug 17 20:33:40 2021 -0400

    Moved auth out of API

---
https://github.com/OpenCircuits/OpenCircuits/commit/db64394397d5b2980f57a9f6fc2d75b18d45860d
Date:   Tue Aug 17 20:28:51 2021 -0400

    Wrapped doc response channel
    
    Before a raw channel was passed around for communicating from the doc to the session, but I updated it to wrap the channel so it isn't accidentally closed and so the type-safety functions are members

---
https://github.com/OpenCircuits/OpenCircuits/commit/bd940e8af121c7b5abdf93cdcd7c17011a50c4c2
Date:   Tue Aug 17 17:07:13 2021 -0400

    Removed unused imports

---
https://github.com/OpenCircuits/OpenCircuits/commit/6897f16cd715a81a9795f3b9af49d7269bb9fe33
Date:   Tue Aug 17 17:05:37 2021 -0400

    Added user join/leave and consolidated protocol
    
    Sessions joining/leaving a document are important events for updating the UI to see what collaborators are up to, or at least who else is editing.  Handling this in the protocol is important because the server is what keeps track of which sessions are joined.

---
https://github.com/OpenCircuits/OpenCircuits/commit/9a74bfd8328a81138f8d60228c47d69e24d2b6dc
Date:   Tue Aug 17 15:26:07 2021 -0400

    Update session.go

---
https://github.com/OpenCircuits/OpenCircuits/commit/e0cb3677e2099f00b80c3d6670e12def5f0a4382
Date:   Tue Aug 17 15:25:50 2021 -0400

    Removed BroadcastState type
    
    This is going to be implemented as actions

---
https://github.com/OpenCircuits/OpenCircuits/commit/6a78e868570153890373491e299a34ed6af9bf90
Date:   Tue Aug 17 15:25:34 2021 -0400

    Document uses type switch

---
https://github.com/OpenCircuits/OpenCircuits/commit/98f74d5dfedb82d56b98856bb3eebc91f6a0b30d
Date:   Tue Aug 17 12:45:54 2021 -0400

    Strict separation between client/session and session/doc messaging
    
    This also improves the skeleton for where authentication/authorization checks will be made.  I decided to make session IDs be public since a new session is spawned for each connection to the WS url instead of ahead of time.  These changes show how the session layer is not just a pass-through.

---
https://github.com/OpenCircuits/OpenCircuits/commit/46f4f62f8f0fe085d813167aabcd516526a2fece
Date:   Tue Aug 17 01:41:24 2021 -0400

    Made AcceptedEntry consistent with ProposeEntry

---
https://github.com/OpenCircuits/OpenCircuits/commit/66c7225fd9b6f1c670c14f24fe1936df798df2ad
Date:   Tue Aug 17 01:31:09 2021 -0400

    Decoupled the protocol format from the Action format
    
    This means that "Actions" can be encoded in whatever way works best for them, but that does not constrain the protocol format.

---
https://github.com/OpenCircuits/OpenCircuits/commit/87ab3bf37249e0a844d23a19871f8138a25e3c61
Date:   Mon Aug 16 16:37:19 2021 -0400

    Added client/server tests
    
    Fix the WebSocket client to work correctly with the event-driver WS library.

---
https://github.com/OpenCircuits/OpenCircuits/commit/64d581a2e622aba566d48eff21c226abea299be7
Date:   Mon Aug 16 16:35:52 2021 -0400

    Fixed channel double-close error

---
https://github.com/OpenCircuits/OpenCircuits/commit/0ea5986bb28a5a3912b3fe8c901161282cfd581f
Date:   Mon Aug 16 16:34:43 2021 -0400

    Removed SessionID from entries
    
    The changelog does not need to know about session information.

---
https://github.com/OpenCircuits/OpenCircuits/commit/33ed7d0be409c93ed3d48fbf8abfec1fbee2cd80
Date:   Mon Aug 16 16:18:09 2021 -0400

    Made Client/Server Protocol Consistent
    
    There are some on-going issues with how to serialize the message types so they can be read easily by the server and the client.  This commit makes the client and server compatible, but is not the best solution

---
https://github.com/OpenCircuits/OpenCircuits/commit/7909f43d10971cc9708d2ce2191632109ea34e8b
Date:   Thu Aug 12 17:18:50 2021 -0400

    Fixed assertions and added default values
    
    Updated from legacy assert function and fixed assertion failures caused by missing default values in proposed/accepted entries.

---
https://github.com/OpenCircuits/OpenCircuits/commit/8969d21f2b63f442c20cf3ed4382f72e681ee9e3
Date:   Thu Aug 12 17:02:51 2021 -0400

    Revert "Added GUIDs to wires/ports/components"
    
    This reverts commit 2c1c58dbd04a5e64eb83b784bc71785bff96800a.

---
https://github.com/OpenCircuits/OpenCircuits/commit/fe5047b4ebe0bf220d0c9f537487873f8b500705
Date:   Thu Aug 12 17:02:45 2021 -0400

    Revert "Hack to serialize references"
    
    This reverts commit 4b3193f14afb75f9cd24ffd092333eb7f9eaf441.

---
https://github.com/OpenCircuits/OpenCircuits/commit/b5770da6993d4f77a1fea949da44f17ba1a9e4d3
Date:   Thu Aug 12 16:49:10 2021 -0400

    Fixed import spacing style

---
https://github.com/OpenCircuits/OpenCircuits/commit/f2198f7ce73d233e7cd108b6e7e4284136a6cd9d
Date:   Tue Aug 10 20:10:46 2021 -0400

    OTDocument.SendNext tests

---
https://github.com/OpenCircuits/OpenCircuits/commit/cf005a3ccd7b757a5adf3a8b525d33cbd7799134
Date:   Tue Aug 10 18:05:46 2021 -0400

    Removed vestigial "joined" flag
    
    This behavior got absorbed into the `WebSocketConnection` class and the `Connection` interface has the responsibility of a reliable connection.

---
https://github.com/OpenCircuits/OpenCircuits/commit/1301f20e667ba5e179c0b17f45c803431e873df5
Date:   Tue Aug 10 17:50:23 2021 -0400

    Split Recv, made plural, and ProposedCache uses Actions
    
    Now Recv only handles remote new entries, since ack entries don't need to be transformed, just added to the log and removed from the ProposedCache.  The ProposedCache now uses just actions, since it doesn't actually use any of the ProposedEntry fields.
    
    Several bugs fixed in the ProposedCache:
    - Usage of Push is uniform with revert/xform
    - Keep track of the "success" of sent so its preserved in the log on ack

---
https://github.com/OpenCircuits/OpenCircuits/commit/ed1ef2df27596aa3c5b892f22a454a6e2715e804
Date:   Mon Aug 9 16:04:06 2021 -0400

    Added success flag to client log
    
    This will be used to determine if an entry can be "undone"

---
https://github.com/OpenCircuits/OpenCircuits/commit/aee93bdee352a9abd59c761218e85cd657c0499f
Date:   Mon Aug 9 15:50:52 2021 -0400

    Tabs to spaces

---
https://github.com/OpenCircuits/OpenCircuits/commit/4823e86948b4520514ad1c2a0753983dd8ac77e8
Date:   Mon Aug 9 15:48:50 2021 -0400

    Added tests for ProposedCache

---
https://github.com/OpenCircuits/OpenCircuits/commit/634e1557a113c6d6af6dafbb35080f50570e115a
Date:   Mon Aug 9 15:48:31 2021 -0400

    Removed unnecessary abstraction from connection

---
https://github.com/OpenCircuits/OpenCircuits/commit/e37af0176142bd4d7b419e7c3e87f009e011d6ea
Date:   Mon Aug 9 14:02:13 2021 -0400

    Implemented connection for WS
    
    This includes a wrapper that abstracts the unreliable nature of the WS connection into a reliable interface.

---
https://github.com/OpenCircuits/OpenCircuits/commit/d0d94cefe5e3a543b421b66ac08d7a767cc82089
Date:   Mon Aug 9 11:40:57 2021 -0400

    Made logic more testable
    
    I separated the different layers of the document class so they can be independently tested and still only expose the necessary features to the rest of the app.

---
https://github.com/OpenCircuits/OpenCircuits/commit/8c2396c377b0d7feba358117f785ebfb3625bc0a
Date:   Mon Aug 9 09:59:30 2021 -0400

    Moved transformation as external function
    
    To avoid a complete graph of dependencies among the actions, I moved the transformation function to be external to the individual actions.

---
https://github.com/OpenCircuits/OpenCircuits/commit/8501837507393cb0fbe80ba6be0b97b62cb05b73
Date:   Sun Aug 8 23:42:11 2021 -0400

    Fixed Changelog generic

---
https://github.com/OpenCircuits/OpenCircuits/commit/1a1884e55b514b6f1f65cb62421aec71224187cd
Date:   Sun Aug 8 23:32:28 2021 -0400

    Added simple Text/HashMap OT test models

---
https://github.com/OpenCircuits/OpenCircuits/commit/b2d3e8ff7bfc7384f757d735a7192a0a7ffd6ab8
Date:   Sun Aug 8 20:44:20 2021 -0400

    Improved generic Action generics

---
https://github.com/OpenCircuits/OpenCircuits/commit/c0b1506645adc9583b84ac93b791b3a8e30dc567
Date:   Sun Aug 8 20:28:41 2021 -0400

    More tests and Serialize returns error
    
    serializing protocol types from server->client was a soft error case, but now it correctly returns an error, which propagates to the call site

---
https://github.com/OpenCircuits/OpenCircuits/commit/80acd733060a8d5615f215a3c3e48049f037a9d6
Date:   Sun Aug 8 20:18:49 2021 -0400

    Tests & bugfixes in client/session protocol

---
https://github.com/OpenCircuits/OpenCircuits/commit/395fcf0007880232d4c82bb589a168643a99999b
Date:   Sun Aug 8 14:51:58 2021 -0400

    Removed unnecessary casts

---
https://github.com/OpenCircuits/OpenCircuits/commit/b08549fd21e9850b1fa5e65ef4065d369229e773
Date:   Sun Aug 8 14:51:27 2021 -0400

    Revised connection / protocol organization

---
https://github.com/OpenCircuits/OpenCircuits/commit/f8baade665472def69bbb2efcd75a419b42bd47b
Date:   Sun Aug 8 12:55:15 2021 -0400

    Fixed document_test.go

---
https://github.com/OpenCircuits/OpenCircuits/commit/d9094cc59e4a409294889038dbdff561ad079591
Date:   Sun Aug 8 12:51:48 2021 -0400

    Action has apply rather than doc
    
    This better represents how they will actually be implemented.

---
https://github.com/OpenCircuits/OpenCircuits/commit/dad7105bcb50c1be63598557901c5517f83bfa90
Date:   Fri Aug 6 20:33:51 2021 -0400

    First draft of client OT logic

---
https://github.com/OpenCircuits/OpenCircuits/commit/f621801d2fdef8713dc9753ff901f9d8b8ba879f
Date:   Fri Aug 6 20:33:31 2021 -0400

    internal/external interfaces more robust
    
    The interface between the document and session has more compile-time safety checks to ensure types follow the protocol.  Also defined the protocol between the client and the session.

---
https://github.com/OpenCircuits/OpenCircuits/commit/045d4ec626dac1702ebf9f90867fa8afa3a2e186
Date:   Fri Aug 6 20:31:29 2021 -0400

    Changed Action and SchemaVersion types
    
    Actions now won't be deserialized by JSON, and the schema version is a string

---
https://github.com/OpenCircuits/OpenCircuits/commit/4b3193f14afb75f9cd24ffd092333eb7f9eaf441
Date:   Mon Aug 2 17:00:07 2021 -0400

    Hack to serialize references

---
https://github.com/OpenCircuits/OpenCircuits/commit/2c1c58dbd04a5e64eb83b784bc71785bff96800a
Date:   Mon Aug 2 12:19:20 2021 -0400

    Added GUIDs to wires/ports/components

---
https://github.com/OpenCircuits/OpenCircuits/commit/d5c16d432293a5b2f7a43a320a2d468803ad0b28
Date:   Sun Aug 1 23:16:04 2021 -0400

    ConnectionAction creates wire just-in-time
    
    If the ConnectionAction is created before it is executed, the CircuitDesigner may fail to create the wire.  Now, the wire is initialized once, just-in-time.

---
https://github.com/OpenCircuits/OpenCircuits/commit/33f9700cab704b2db7c2ce67d1ea5659acfe66f7
Date:   Sun Aug 1 17:42:49 2021 -0400

    document manager tests

---
https://github.com/OpenCircuits/OpenCircuits/commit/9954bfae393a1ed515b7a4ea7e486a6ecb1ba426
Date:   Sun Aug 1 17:26:46 2021 -0400

    Added small session channel buffer
    
    This came up in testing.  Without a buffer, some contention may be caused if high-load scenarios

---
https://github.com/OpenCircuits/OpenCircuits/commit/9a0430ef9df1bc89c1b9835fc364b91e1e1c673a
Date:   Sun Aug 1 17:25:42 2021 -0400

    Added document tests

---
https://github.com/OpenCircuits/OpenCircuits/commit/5a1c6c60165d5dff2971f3be2815e939825bee10
Date:   Sun Aug 1 16:48:12 2021 -0400

    Moved OT out of API

---
https://github.com/OpenCircuits/OpenCircuits/commit/c2a0b42e81a57951a7e30bd03cd801e9b75a099a
Date:   Sun Aug 1 16:39:17 2021 -0400

    Tests and Connection layer

---
https://github.com/OpenCircuits/OpenCircuits/commit/38e28c0ba6102155642f17413f8138a46b9893de
Date:   Sun Aug 1 01:41:42 2021 -0400

    Removed unnecessary complexity

---
https://github.com/OpenCircuits/OpenCircuits/commit/7cefece681ff0f07f0ecf35c054be6aed389090a
Date:   Sun Aug 1 01:29:56 2021 -0400

    Updated stuff

---
https://github.com/OpenCircuits/OpenCircuits/commit/2014a646b08341c397d1c5b54a0db89b843478b4
Date:   Sun Aug 1 01:29:45 2021 -0400

    Reorganized

---
https://github.com/OpenCircuits/OpenCircuits/commit/efd1c159a9d57fb7ace83b82835c15b554524180
Date:   Sat Jul 31 13:29:36 2021 -0400

    Scattered work on OT (rebuild this)

---
https://github.com/OpenCircuits/OpenCircuits/commit/c17a6baa1a0169cd327f9d3790357dcf10faa83e
Date:   Wed Jul 14 19:46:52 2021 -0400

    Split special auth providers

---
https://github.com/OpenCircuits/OpenCircuits/commit/d7e75327dcdd91a685c4fa66b17c1ec377d53f88
Date:   Wed Jul 14 19:43:32 2021 -0400

    Moved handlers to routes dir

---
https://github.com/OpenCircuits/OpenCircuits/commit/c56721431223fadd36b9089ebf6ba121c298f114
Date:   Wed Jul 14 18:39:52 2021 -0400

    Consistent Directory Structure

---
https://github.com/OpenCircuits/OpenCircuits/commit/1098f4e58c311f564b8e5c64b4b4052ed81ca9b5
Merge: 93d4786d f8cc1631
Date:   Wed Jul 14 16:27:52 2021 -0400

    Merge remote-tracking branch 'origin/backend_improvements' into backend_access

---
https://github.com/OpenCircuits/OpenCircuits/commit/f8cc1631be1bd9d6cd09d752d8e8447a7f980f8c
Date:   Wed Jul 14 09:54:03 2021 -0400

    Better error handling in wrapper

---
https://github.com/OpenCircuits/OpenCircuits/commit/bcf31010021126b72dd264e618a55534f4b96977
Date:   Wed Jul 14 09:41:32 2021 -0400

    Removed manual circuit parsing

---
https://github.com/OpenCircuits/OpenCircuits/commit/96678011760af6e98b81c3f804bfbcb5c4b5d54f
Date:   Wed Jul 14 09:35:41 2021 -0400

    Simplify Route definitions with Middleware and Context

---
https://github.com/OpenCircuits/OpenCircuits/commit/93d4786d967032a8cc84ec6b60b462fbdf3571db
Date:   Tue Jul 13 09:49:33 2021 -0400

    ANON user cannot create circuits

---
https://github.com/OpenCircuits/OpenCircuits/commit/3bc50ff9d78c53dd7e39debc4dd39d0880a65381
Date:   Mon Jul 12 22:13:56 2021 -0400

    Set front-end Circuit ID to retrieved Circuit's ID

---
https://github.com/OpenCircuits/OpenCircuits/commit/5839f8d856db9d70042020d5b4b3a8c41923cdc1
Date:   Mon Jul 12 21:56:33 2021 -0400

    Enumerate circuits shows shared circuits

---
https://github.com/OpenCircuits/OpenCircuits/commit/41f8de0c4f9e16ce287278c6fc5ccf0b2badba82
Date:   Mon Jul 12 21:04:45 2021 -0400

    factored out error handling

---
https://github.com/OpenCircuits/OpenCircuits/commit/d3ba1e9ce72702d8903536643db2a79d7ac1be2f
Date:   Mon Jul 12 15:26:39 2021 -0400

    Using access in circuits routes
    
    Other various reorganizing, using middleware for identity  injection, etc

---
https://github.com/OpenCircuits/OpenCircuits/commit/c30e42af4fd19d6535e8a5b1195729f36b40e54d
Date:   Sun Jul 11 20:53:44 2021 -0400

    Added in-memory access data driver

---
https://github.com/OpenCircuits/OpenCircuits/commit/4a90bda6e72e295e1c00da214a9e41c890136438
Date:   Sun Jul 11 13:23:58 2021 -0400

    Added anonymous auth
    
    This gives an identity to the unauthorized user, for link-based sharing.

---
https://github.com/OpenCircuits/OpenCircuits/commit/2acb2cc9342e64a7301762e210ead4d483066515
Date:   Sun Jul 11 13:23:19 2021 -0400

    Separated access rules from handlers and better error prop

---
https://github.com/OpenCircuits/OpenCircuits/commit/2df6a973cc794258c53aac701a09fc485bc1c6d4
Date:   Fri Jul 9 13:58:02 2021 -0400

    Initial work on interfaces and handlers
