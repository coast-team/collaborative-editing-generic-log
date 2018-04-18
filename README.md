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
    ...
    {
        "type": "localInsertion",
        "timestamp": 1523531787342,
        "siteId": 0,
        "clock": 0,
        "index": 0,
        "content": "Hellox",
        "length": 6,
        "context": {},
        "collaborators": [0, 1],
        "neighbours": [0, 1]
    },
    {
        "type": "localInsertion",
        "timestamp": 1523531787343,
        "siteId": 1,
        "clock": 0,
        "index": 0,
        "content": " world",
        "length": 6,
        "context": {},
        "collaborators": [0, 1, 2],
        "neighbours": [0, 1, 2]
    },
    ...
    {
        "type": "remoteInsertion",
        "timestamp": 1523531960000,
        "siteId": 2,
        "sender": 1,
        "operation": { ... },
        "context": {},
        "collaborators": [0, 1, 2],
        "neighbours": [0, 1, 2]
    },
    {
        "type": "remoteInsertion",
        "timestamp": 1523531960300,
        "siteId": 2,
        "sender": 1,
        "operation": { ... },
        "context": {
            "1": 0
        },
        "collaborators": [0, 1, 2],
        "neighbours": [0, 1, 2]
    },
    {
        "type": "localDeletion",
        "timestamp": 1523531960875,
        "siteId": 2,
        "clock": 0,
        "index": 5,
        "content": "x",
        "length": 1,
        "context": {
            "0": 0,
            "1": 0
        },
        "collaborators": [0, 1, 2],
        "neighbours": [0, 1, 2]
    },
    {
        "type": "disconnection",
        "timestamp": 1524032256605,
        "siteId": 2
    },
    ...
]
```

## Documentation

### General fields

The following fields are shared by all log entries.

- type: The type of event. According to its value, the schema of the entry will change. Several values are accepted :
    - _creation_: The creation of the document.
    - _connection_: A site joining the collaborative session.
    - _disconnection_: A site leaving the collaborative session.
    - _localDeletion_: A site deleting some text.
    - _localInsertion_: A site inserting some text.
    - _remoteDeletion_: A site receiving a deletion from another site.
    - _remoteInsertion_: A site receiving an insertion from another site.
- siteId: The unique identifier of the site which generate this entry of the log.
- timestamp: The timestamp of the entry.

We now describe the fields specific to each kind of event.

### creation

- documentId: The identifier of the document.

###  localDeletion and localInsertion

- clock: The logical clock of the site at the moment of the update.
- index: The index of the operation.
- content: The content inserted or deleted by the operation.
- length: The number of characters inserted or deleted by the operation.

### remoteDeletion and remoteInsertion

- sender: The collaborator providing the remote operation. May differ from the operation's author.
- operation: The remote operation received. Its schema depending of the conflict resolution mechanism used, it won't be described here.

### localDeletion, localInsertion, remoteDeletion and remoteInsertion

- context: The context of generation or reception of the operation. It represents the state of the document using a _state vector_, which keeps track of the operations observed by the site.
- collaborators: The global list of current collaborators.
- neighbours: The list of collaborators to which the site is directly connected to at the time. May differ from the field _collaborators_ because of network characteristics.

## Open questions

- Do we need the content of the document or can we anonymize it ?

## Limits

Unfortunately, we won't be able to replicate accurately a collaborative editing using such a log.
Indeed, some conflict resolution mechanisms are not determinist.
For example, when using LogootSplit, the resulting order of concurrent insertions at the same offset is random.
Thus, given one log, we are not able to ensure the same result for each repeated execution.
