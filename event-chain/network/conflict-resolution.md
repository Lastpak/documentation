# Conflict resolution

In any decentralized system conflicts can arise. Multiple nodes may add an event at the same time or a node might miss an event causing it to branch the chain. Because an event represents something that happened off-chain, we can't simply drop events instead they need to be added in such a way that it's visible that a conflict occurred.

## Chain branches

If there is a conflict on an event chain, two \(or more\) branches of the chain exist. Conflict resolution means merging those branches back to a single event chain.

An identity can start a branch after the last event it has signed. This is a point up until we can prove that the identity has received the events. Trying to branch before this point MUST result in a [rejection](), resulting in a fork.

### Rebasing

To merge branches, events of one of the branches are rebased onto to other branch. A rebased event contains an `original` property with the properties of the original event, minus the body \(which doesn't change\).

### Rebuilding projections

If your branch is abandoned and you're switching to a new branch, your node needs to revert changes to projections up to the fork. For some projections like processes, it difficult to revert a state change.

Instead, the whole projection will be deleted and rebuild by replaying all events.

## Resolution methods

Every node is free to resolve conflicts any way it wants. Behaving in an unexpected way result in a dead lock or dead loop. Therefor there's a recommended way of resolving conflicts.

In some cases it's possible to benefit by abusing conflict and the recommended way for conflict resolution. This can be a reason to not adhere to this method.

### Recommended resolution

Each event contains a timestamp. This is the time according when event was signed, according to the signer. When determining which branch is used, a node SHOULD use the timestamp of the events that caused this fork. The newer event SHOULD be stitched upon the other branch.

This method does allow backdating. To reduce the period an event can be backdated to, a node SHOULD check the timestamp of the receipt. Depending on the blockchain and system used to anchor the event, this can take up to 15 minutes. Events that are less than 15 minutes old SHOULD be accepted without a receipt. Older events MAY be rejected.

### Dominant identity

An alternative is that one identity is dominant. In case of a conflict, the event chain hold by that identity is considered to be the valid one and other events are always stitched upon this. Such behavior is configured in the node. Note that having multiple identities claim dominance can lead to a dead-lock. An identity SHOULD NOT behave dominant if it did not initiate the event chain.

