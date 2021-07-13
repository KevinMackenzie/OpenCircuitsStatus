# Status Report, July 07, 2021
Project: Open Circuits

## This Week
This past week, I have been writing down the changes required to the back-end of OpenCircuits to support multi-client editing.  This includes an Access / permission system for managing user authorization and a document management service based on Operational Transformation.

I defined a prototype model structure and set of orthagonal document operations to try and get a fuller understanding of the requirements for Operational Transformation on OpenCircuits.  It seems like, being a graph-based editor, OpenCircuits can work well with a simpler set of rules for merging conflicting user's intents than JSON and plain-text documents.

I also have been prototyping the Access system I designed to get a better feel for if Go is an appropriate language to continue using, and currently I see no reason to change.

Finally, I have made two small pull requests.  One fixes a security flaw in the google authentication that would allow a malicious actor with access to 3rd party JWT's for the user to access their OpenCircuits data.  The other fixes an issue causing user documents to duplicate if saved after page reload

## Next Week
This coming week I plan to collaborate with the project owner to define the new document model that can work well with OT and discuss my findings on how to re-use non-trivial model logic on the front-end and back-end.

I plan to make a pull requests on the back-end: Refactor existing back-end to be better organized for the Access system.  The work is done, it just needs to be separated from my working branch.

I will also be organizing the work I outlined in the designed docs into github issues to create incremental units of work.

Finally, I will take the current work I have done prototyping the Access system and rework it to fit the incremental units just mentioned.

## Blockers

The current tooling for building the project is oriented towards front-end development and the production environment.  Testing certain back-end features (i.e. google authentication) requires hacks and work-arounds.

## Links
- See [designs/DesignDoc.md](../designs/DesignDoc.md) for the architecture design
- See [designs/OT.md](../designs/OT.md) for the Operational Transformation design ideas/considerations
- PR 1: https://github.com/OpenCircuits/OpenCircuits/pull/670
- PR 2: https://github.com/OpenCircuits/OpenCircuits/pull/672
- Access Prototyping: https://github.com/OpenCircuits/OpenCircuits/tree/backend_access