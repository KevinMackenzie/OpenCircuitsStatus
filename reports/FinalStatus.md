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
- [OT algorithm, reliable connection abstraction, protocol implementation, and document abstraction on client](https://github.com/OpenCircuits/OpenCircuits/tree/operational_transformation/src/app/core/ot)
- [Document layer for core OT on server, sending updates to sessions, and notifying clients when others join the algorithm.](https://github.com/OpenCircuits/OpenCircuits/tree/operational_transformation/src/server/ot/doc)
- [Protocol between client / server](https://github.com/OpenCircuits/OpenCircuits/tree/operational_transformation/src/server/ot/conn)
- [Session layer for facilitating client permission enforcement and connection lifecycle](https://github.com/OpenCircuits/OpenCircuits/tree/operational_transformation/src/server/ot/session)

## Future Work
Currently, OT documents do not persist across server restarts, so I am creating storage abstractions and moving some code around to add this.

### [Milestone Circuits](https://github.com/KevinMackenzie/OpenCircuitsStatus/blob/master/designs/DesignatedUpdater.md)
To avoid storing the entire history of the circuit, milestones can be created and the changelog gets trimmed.  This doc outlines a solution to this problem within the algorithm I implemented.

## Commit History
Listed below are all the commits I made this semester: