# Status Report, July 18, 2021
Project: Open Circuits

## This Week
I joined the course for the second half of the summer.

Over the past few weeks, I have been writing down the changes required to the back-end of OpenCircuits to support multi-client editing.  This includes an Access / permission system for managing user authorization and a document management service based on Operational Transformation (OT).

I defined a prototype model structure and set of orthagonal document operations to try and get a fuller understanding of the requirements for Operational Transformation on OpenCircuits.  It seems like, being a graph-based editor, OpenCircuits can work well with a simpler set of rules for merging conflicting user's intents than JSON and plain-text documents.

Based on my research findings, I have started an issue tracker for the feature, bringing together existing relevent issues and making a couple sub-issues for specific components.

I also have been prototyping the Access system I designed to get a better feel for if Go is an appropriate language to continue using, and currently I see no reason to change.

I have made two small pull requests.  One fixes a security flaw in the google authentication that would allow a malicious actor with access to 3rd party JWT's for the user to access their OpenCircuits data.  The other fixes an issue causing user documents to duplicate if saved after page reload

My third pull request refactors the existing Golang back-end to be better organized for the Access system and better uses Go and the Gin framework's features.

## Next Week
This coming week I plan to collaborate with the project owner on changes to the document model to make it fully compatable with OT and how to avoid code-duplication with the back-end and front-end sharing model logic.  The end result will be a breakdown of which technologies will be used for which parts of the architecture.

I will avoid continuing work with specific back-end technologies until an agreement has been made, so defining the websocket protocol for the collaboration algorithm is a good technology-agnostic component.  After that, based on the results of the discussion, would be to start making changes to the model as required and make the current user operations serializable over the network.


## Blockers

None

## Links
- See [designs/DesignDoc.md](https://github.com/KevinMackenzie/OpenCircuitsStatus/blob/master/designs/DesignDoc.md) for the architecture design
- See [designs/OT.md](../designs/OT.mdhttps://github.com/KevinMackenzie/OpenCircuitsStatus/blob/master/designs/OT.md) for the Operational Transformation design ideas/considerations
- PR 1: https://github.com/OpenCircuits/OpenCircuits/pull/670
- PR 2: https://github.com/OpenCircuits/OpenCircuits/pull/672
- PR 3: https://github.com/OpenCircuits/OpenCircuits/pull/674
- Issue Tracker: https://github.com/OpenCircuits/OpenCircuits/issues/677
- Other issues: [One](https://github.com/OpenCircuits/OpenCircuits/issues/678), [Another](https://github.com/OpenCircuits/OpenCircuits/issues/679)
- Access Prototyping: https://github.com/OpenCircuits/OpenCircuits/tree/backend_access
