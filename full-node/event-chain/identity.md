---
description: 'An identity within the event chain represents a person, team or organization.'
---

# Identity

The identity is used for authentication and authorization. An identity is not a user, a user has an identity for each event chain it's participating in.

### Schemas

* [Identity](identity.md#identity-schema)
* [Privilege](identity.md#privilege-schema)

[JSON Schema](https://specs.livecontracts.io/v0.1.0/identity/schema.json) \| [changelog](https://github.com/legalthings/livecontracts-specs/tree/f138bf7777b31f535d6fa21c0ddad3a4aaea45d5/identity/changelog.md)

### Registration

The first event on the event chain MUST be an identity. This event enables the user or organization to add subsequent events on the chain. This is the initiator of the chain.

This initial identity can start adding other identities on the chain. If it knows the public key\(sa\) of, it can simply add them and notify the user about this new chain. If the public key is unknown a new key pair may be generated.

#### Authentication

Possibly the new identity might need to go to an authentication process before it can participate in a process, get write access to a contract, etc. In that case the system of the initiator is responsible of granting additional privileges at the end of the authentication process.

#### Removal

Any identity that can overwrite other identities of the event chain, may remove that identity by clearing the `node`, `keys` property. That way the user no longer receives new events and is unable to fetch the chain.

Note that it's not possible to force a node or user to delete an event chain that has already been received.

### Example

#### Initiating identity

```javascript
{
  "$schema": "https://specs.livecontracts.io/v0.1.0/identity/schema.json#",
  "id": "4a88f56a-5bb2-4fa7-8615-c75d5ec7b1d4",
  "info": {
    "name": "John Doe",
    "email": "john.doe@example.com"
  },
  "node": "amqps://app.legalthings.one",
  "signkeys": {
    "default": "8MeRTc26xZqPmQ3Q29RJBwtgtXDPwR7P9QNArymjPLVQ",
    "system": "FA2CiSAWUEANTxcffctxm8XQfTugZv7VX5C1Qb59vbxj"
  },
  "encryptkey": "26X04SH0o6JliuiwzT5kclnU1lN8fYaxaquANNIdbpNx"
}
```

#### Identity with specific privileges.

```javascript
{
  "$schema": "https://specs.livecontracts.io/v0.1.0/identity/schema.json#",
  "id": "75baa885-5862-4f81-80f4-60df746ff002",
  "info": {
    "name": "John Doe",
    "email": "john.doe@example.com"
  },
  "node": "amqps://app.legalthings.one",
  "privileges": [
    {
      "schema": "https://specs.livecontracts.io/v0.1.0/identity/schema.json#",
      "id": "lt:/identities/75baa885-5862-4f81-80f4-60df746ff002",
      "not": [ "privileges" ],
      "signkey": [ "user", "registration" ]
    },
    {
      "schema": "https://specs.livecontracts.io/v0.1.0/document/schema.json#",
      "id": "lt:/contracts/c9ddeeca-ad2f-4204-92da-eda18ed0bd21"
    },
    {
      "schema": "https://specs.livecontracts.io/v0.1.0/comment/schema.json#",
      "signkey": "user"
    }
  ],
  "signkeys": {
    "user": "8MeRTc26xZqPmQ3Q29RJBwtgtXDPwR7P9QNArymjPLVQ",
    "system": "FA2CiSAWUEANTxcffctxm8XQfTugZv7VX5C1Qb59vbxj"
  },
  "encryptkey": "26X04SH0o6JliuiwzT5kclnU1lN8fYaxaquANNIdbpNx"
}
```

#### Identity for invitation to the event chain

```javascript
{
  "$schema": "https://specs.livecontracts.io/v0.1.0/identity/schema.json#",
  "id": "3f9bf36c-2245-4fb7-9e0f-e55f1b7ace15",
  "info": {
    "name": "Arnold",
    "email": "arnold@example.com"
  },
  "privileges": [
    {
      "schema": "https://specs.livecontracts.io/v0.1.0/identity/schema.json#",
      "id": "lt:/identities/75baa885-5862-4f81-80f4-60df746ff002",
      "not": [ "privileges" ],
      "signkey": [ "user", "registration" ]
    },
    {
      "schema": "https://specs.livecontracts.io/v0.1.0/10-action/schema.json#",
      "id": "lt:/identities/cdab4c34-3fb6-41a6-aa58-921358b7677c",
      "signkey": "user"
    }
  ],
  "signkeys": {
    "registration": "8MeRTc26xZqPmQ3Q29RJBwtgtXDPwR7P9QNArymjPLVQ"
  }
}
```

## Identity schema

`https://specs.livecontracts.io/v0.1.0/identity/schema.json#`

### Properties

#### $schema

The Live Contracts Identity [JSON schema](http://json-schema.org) URI that describes the JSON structure of the identity.

#### id

A unique identifier for the identity within the event chain a resource id.

#### node

The uri of node that the identity is using to participate on this chain. This SHOULD be an [ampq URI](https://www.rabbitmq.com/uri-spec.html).

Note that additions to the chain are not broadcasted to all node, but only send to the nodes of the active identities.

#### privileges

A list of [privileges](identity.md#privilege).

#### signkeys

The X25519 public keys that the identity uses for signing events. It's an object where the key is the type.

Common types are `user` and `system`. The `user` key is kept client side, so you can be sure that the user takes that action. The `system` key doesn't belong to the user but to the system the user is running on. It can be specified for an automated step.

```javascript
{
  "user": "8MeRTc26xZqPmQ3Q29RJBwtgtXDPwR7P9QNArymjPLVQ",
  "system": "FA2CiSAWUEANTxcffctxm8XQfTugZv7VX5C1Qb59vbxj"
}
```

#### encryptkey

The `encryptkey` is an X25519 public key used for encrypting data for the identity. The private encrypt key SHOULD be hold and kept secret by the node which is responsible for encrypting and decrypting events.

This is key typically the same as the `system` signkey, but may differ is one node serves multiple systems.

## Privilege schema

`https://specs.livecontracts.io/v0.1.0/identity/schema.json#privilege`

A privilege grant the identity to add one type events to the event chain.

### Properties

#### schema

A live contracts schema URI. This determines the type of events that may be added to the event chain.

#### id

Set the `id` if the privilege only applies to a specific projection like a process or a contract.

#### only

When projecting an item from an event, filter the body so it only contains these properties. Properties listed here are ignored. This doesn't go more than one level deep.

#### not

When projecting an item from an event, filter the body so it does not contains these properties. Properties note listed here are ignored. This doesn't go more than one level deep.

#### signkey

If specified, only this type / these types of sign keys may be used to sign the event.

