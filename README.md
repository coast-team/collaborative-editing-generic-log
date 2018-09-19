# Generic log from and for collaborative editing session

## Goals

The goal of this project is to propose a generic log format for collaborative editing tools.
Such logs are needed to replicate collaborative editing sessions using different conflict resolution mechanisms to compare theirs performances.

## Mock-up

```JSON
[
    {
        "type": "creation",
        "timestamp": 152353168720,
        "siteId": 0,
        "documentId": "log-mockup",
    },
    {
        "type": "connection",
        "timestamp": 152353168720,
        "siteId": 0,
    },
    {
        "type": "peerConnection",
        "timestamp": 152353169720,
        "siteId": 0,
        "remoteSiteId": 1,
    },
    {
        "type": "collaboratorJoin",
        "timestamp": 152353169724,
        "siteId": 0,
        "remoteSiteId": 1,
    }
    ...
    {
        "type": "localInsertion",
        "timestamp": 1523531787342,
        "siteId": 0,
        "clock": 0,
        "position": 0,
        "content": "Hellox",
        "logootsOperation": { ... },
        "context": {},
        "collaborators": [0, 1],
        "neighbours": {
            "downstream": [0, 1],
            "upstream": [0, 1],
        }
    },
    {
        "type": "localInsertion",
        "timestamp": 1523531787343,
        "siteId": 1,
        "clock": 0,
        "position": 0,
        "content": " world",
        "logootsOperation": { ... },
        "context": {},
        "collaborators": [0, 1, 2],
        "neighbours": {
            "downstream": [0, 1, 2],
            "upstream": [0, 1, 2],
        }
    },
    ...
    {
        "type": "remoteInsertion",
        "timestamp": 1523531960000,
        "siteId": 2,
        "remoteSiteId": 1,
        "remoteClock": 0,
        "textOperation": [ ... ],
        "logootsOperation": { ... },
        "context": {},
        "collaborators": [0, 1, 2],
        "neighbours": {
            "downstream": [0, 1, 2],
            "upstream": [0, 1, 2],
        }
    },
    {
        "type": "remoteDeletion",
        "timestamp": 1523531960300,
        "siteId": 2,
        "remoteSiteId": 1,
        "remoteClock": 1,
        "textOperation": [ ... ],
        "logootsOperation": { ... },
        "context": {
            "1": 0
        },
        "collaborators": [0, 1, 2],
        "neighbours": {
            "downstream": [0, 1, 2],
            "upstream": [0, 1, 2],
        }
    },
    {
        "type": "localDeletion",
        "timestamp": 1523531960875,
        "siteId": 2,
        "clock": 0,
        "position": 5,
        "length": 1,
        "logootsOperation": { ... },
        "context": {
            "0": 0,
            "1": 0
        },
        "collaborators": [0, 1, 2],
        "neighbours": [0, 1, 2]
    },
    {
        "type": "peerDisconnection",
        "timestamp": 1524032252505,
        "siteId": 0,
        "remoteSiteId": 1,
    },
    {
        "type": "collaboratorLeave",
        "timestamp": 1524032252509,
        "siteId": 0,
        "remoteSiteId": 1,
    }
    {
        "type": "disconnection",
        "timestamp": 1524032256605,
        "siteId": 0
    },
    ...
]
```

## Documentation

### General fields

The following fields are shared by all log entries.

- type: The type of event. According to its value, the schema of the entry will change. Several values are accepted :
  - _opening_: The opening of the document.
  - _creation_: The creation of the document. (The first time the document is opened)
  - _connection_: Your site join the collaborative session.
  - _disconnection_: Your site leave the collaborative session.
  - _peerConnection_: A site establishing a connection with yours.
  - _peerDisconnection_: A site removing a connection with yours.
  - _collaboratorJoin_: A site joining the collaborative session.
  - _collaboratorLeave_: A site leaving the collaborative session.
  - _localDeletion_: A site deleting some text.
  - _localInsertion_: A site inserting some text.
  - _remoteDeletion_: A site receiving a deletion from another site.
  - _remoteInsertion_: A site receiving an insertion from another site.
- siteId: The unique identifier of the site which generate this entry of the log.
- timestamp: The timestamp of the entry.

We now describe the fields specific to each kind of event.

### creation and opening

- documentId: The identifier of the document.

### connection

- neighbours: The list of collaborators to which the site is directly connected to at the time. May differ from the field _collaborators_ because of network characteristics.
  - downstream: The list of site we are able to receive directly message.
  - upstream: The list of site we are able to send directly message.

### peerConnection and peerDisconnection

- remoteNetworkId: The unique id of the node we established or removed a connection with.

### collaboratorJoin and collaboratorLeave

- remoteSiteId: The siteId of the collaborator which join or leave the session.
- remoteNetworkId: The unique id of the node which the collaborator uses.

### localDeletion and localInsertion

- clock: The logical clock of the site at the moment of the update.
- position: The index of the operation.
- content: The content inserted by the operation. (only for localInsertion)
- length: The number of characters inserted or deleted by the operation.
- logootsOperation: The operation corresponding to the TextOperation using LogootSplit
- context: The context of generation or reception of the operation. It represents the state of the document using a _state vector_, which keeps track of the operations observed by the site.
- collaborators: The global list of current collaborators.
- neighbours: The list of collaborators to which the site is directly connected to at the time. May differ from the field _collaborators_ because of network characteristics.
  - downstream: The list of site we are able to receive directly message.
  - upstream: The list of site we are able to send directly message.

### remoteDeletion and remoteInsertion

- remoteSiteId: The author of the operation.
- remoteClock: The logical clock of the remote site at the moment of the update.
- textOperation: The list of operations resulting from the remote operation.
- logootsOperation: The operation corresponding to the TextOperation using LogootSplit.
- context: The context of generation or reception of the operation. It represents the state of the document using a _state vector_, which keeps track of the operations observed by the site.
- collaborators: The global list of current collaborators.
- neighbours: The list of collaborators to which the site is directly connected to at the time. May differ from the field _collaborators_ because of network characteristics.
  - downstream: The list of site we are able to receive directly message.
  - upstream: The list of site we are able to send directly message.

## Open questions

- Do we need the content of the document or can we anonymize it ?

## Limits

Unfortunately, we won't be able to replicate accurately a collaborative editing using such a log.
Indeed, some conflict resolution mechanisms are not determinist.
For example, when using LogootSplit, the resulting order of concurrent insertions at the same offset is random.
Thus, given one log, we are not able to ensure the same result for each repeated execution.
