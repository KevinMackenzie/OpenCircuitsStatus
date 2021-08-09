# Status Report, August 02, 2021
Project: Open Circuits

## This Week
This week, I have implemented a proof-of-concept back-end for operational transformation.  It includes:
1. A simple communication layer abstraction with an implementation via websockets
2. A session layer, which correspond 1:1 with connected clients, that parses/formats data for/from the document layer
3. A document layer, which is a state thread for each live document.
4. A simple in-memory document model for storing operational transformation changelog entries.

Missing in the proof of concept is: any form of access control, which will be managed in the session layer (2); document persistence, which will be managed in the document model (4); log trimming, which will also be managed in the document model (4) and via a separate internal service.  I have created tests for the document layer (3) and the model (4).

I have also begun addressing issue #678 to add GUIDs to certain parts of the model to make incremental document changes have consistent references when sent across the network.

Finally, I put in a small PR to address an issue caused by premature precondition enforcement.  There have been changes requested on the PR, but I foresee changing that code in other ways that would subsume the original issue and I will put off addressing it until I know where I'm going with it.

## Next Week
Next week I will continue to make the GUID changes required for serializing increment document changes.

Simultaneously, I will be working on implementing the client-side logic for operational transformation and communicating with the back-end.  Because of the lack of client-side concurrency, I think integrating into the client-side state management (Redux) will take longer than writing the algorithm logic.  I hope to have a proof-of-concept working with a subset of editing features by the end of next week.

## Blockers
None

## Links
Commits on [Operational transformation](https://github.com/OpenCircuits/OpenCircuits/commits/operational_transformation) code
- https://github.com/OpenCircuits/OpenCircuits/tree/efd1c159a9d57fb7ace83b82835c15b554524180
- https://github.com/OpenCircuits/OpenCircuits/tree/2014a646b08341c397d1c5b54a0db89b843478b4
- https://github.com/OpenCircuits/OpenCircuits/tree/7cefece681ff0f07f0ecf35c054be6aed389090a
- https://github.com/OpenCircuits/OpenCircuits/tree/38e28c0ba6102155642f17413f8138a46b9893de
- https://github.com/OpenCircuits/OpenCircuits/tree/c2a0b42e81a57951a7e30bd03cd801e9b75a099a
- https://github.com/OpenCircuits/OpenCircuits/tree/5a1c6c60165d5dff2971f3be2815e939825bee10
- https://github.com/OpenCircuits/OpenCircuits/tree/9a0430ef9df1bc89c1b9835fc364b91e1e1c673a
- https://github.com/OpenCircuits/OpenCircuits/tree/9954bfae393a1ed515b7a4ea7e486a6ecb1ba426
- https://github.com/OpenCircuits/OpenCircuits/tree/33f9700cab704b2db7c2ce67d1ea5659acfe66f7
- https://github.com/OpenCircuits/OpenCircuits/tree/2c1c58dbd04a5e64eb83b784bc71785bff96800a
- https://github.com/OpenCircuits/OpenCircuits/tree/4b3193f14afb75f9cd24ffd092333eb7f9eaf441

Small PR (WIP changes requested)
- https://github.com/OpenCircuits/OpenCircuits/pull/686