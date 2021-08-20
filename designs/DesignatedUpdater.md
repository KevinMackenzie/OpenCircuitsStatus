# Milestone Circuits and new Client Role: Designated Updater
One issue I bring up in the [original design doc](./DesignDoc.md) is that the server cannot accumulate the document unless it has access to the client logic.  I describe a system where there is an internal service that gets called periodically to perform these operations.  Here, I propose an alternative solution that can be used without this additional service, but does not exclude such a service from being used in the future.

## Overview
In this OT algorithm there are clients and a single server.  The server is the trusted source of the log, but cannot produce a full document.  The clients are not trusted, but can produce a full document, render the thumbnail, etc.  Clients with edit permissions _can_ make significant operations to the document, but the log will preserve the history and make all such operations reversible.

The concept of milestone circuits are necessary for trimming the log, so one client of the algorithm must be trusted to create milestone circuits.  I propose such a client role, called the Designated Updater, which is the sole client that can send milestone circuits.  Similarly, this client also can render the thumbnail and submit updated metadata.  This role can be assigned based on connection latency, or some other metric, to any user with at least edit permissions.

## Metadata
The circuit can use OT for updating metadata components and it does not require any out-of-band communication to update other clients.  Metadata needs to be queryable so that circuits can appear in the user's circuit list, or be searchable once public circuits are a feature.  Since the server does not have the document accumulate logic, it cannot extract metadata from log entries.

A new message can be added to the protocol, which is only accpeted from the designated updater, for metadata updates (name, description, thumbnail).  These values get set in the database.  A byzantine designated updater could make the metadata nonsense, but that does not break the content of the circuit.

## Content
Milestone circuits are important for being able to view old versions after logs have been trimmed, and to reduce the burden on the client to accumulate an entire document upon load.

The designated updater submits a milestone circuit for its current log clock to the server.  The server adds this milestone to a list and may delete past entries based on timing.  This keeps multiple milestones available.  As long as a milestone exists _after_ a log trim point, the log can be trimmed.

## Concerns
Byzantine designated updaters can submit broken milestone circuits.  If the client is running on an untrusted computer, such as a user's computer, the designated updater can be malicious as well.  In principle this is a problem, and the solution is to use an internal service for the designated updater, but for now it is a compromise to assume that anyone with edit permissions is trusted enough to be the designated updater.

![Creative Commons License](https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png) This work is licensed under a http://creativecommons.org/licenses/by-nc-sa/4.0/ Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License.