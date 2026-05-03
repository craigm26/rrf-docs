# Federation Protocol

The Robot Registry is not a single database. It is a federated network of nodes that cross-verify, stay in sync, and ensure no single point of failure can disrupt robot identity resolution.

For full technical specification, see [rcan.dev §17 Distributed Registry Node Protocol](https://rcan.dev).

---

## How It Works

Each registry node maintains a signed, verifiable copy of the records it is authoritative for. Nodes propagate updates using a structured gossip protocol with cryptographic verification at every hop. A robot's RRN can be resolved by any node in the federation — the resolution chain returns the most-recently-confirmed authoritative record.

The root node is operated by the RRF and is the trust anchor. But because every node verifies signatures independently, the federation continues to function correctly even if the root node is temporarily unreachable. Cached records remain valid until their TTL expires.

This mirrors how DNS works for domain names: a hierarchy of authority, distributed resolution, and graceful degradation when parts of the network are unavailable.

---

## Node Types

### Root Node

**Operated by:** RRF only  
**Authority:** Full — issues new RRNs, delegates namespace, signs certificates  
**Count:** 1 (operated by RRF)

The authoritative root of the registry hierarchy. Operated by the RRF. Signs all canonical RRN assignments and issues namespace delegations to authoritative nodes.

---

### Authoritative Node

**Operated by:** Supporting Member manufacturers and regional federations  
**Authority:** Delegated — RRNs within assigned prefix only  
**Count:** Unlimited by design

Delegated authority over a specific manufacturer prefix or geographic namespace. Operates independently but syncs records with the root. Can issue RRNs within its delegated range.

---

### Resolver Node

**Operated by:** Any organization  
**Authority:** None — read-only, cannot assign RRNs  
**Count:** Unlimited

Full read-replica of the registry. Can resolve any RRN. Used for high-availability lookup endpoints. Cannot issue new RRNs. Common pattern for robot fleets needing local resolution.

---

### Cache Node

**Operated by:** Any organization — common in embedded and edge deployments  
**Authority:** None — cached read-only, may be stale  
**Count:** Unlimited

Partial, TTL-based cache of registry records. Trades completeness and freshness for speed. Used in bandwidth-constrained environments or offline-capable deployments.

---

## Run Your Own Registry Node

Any organization can run a resolver or cache node. This improves the resilience of robot identity resolution for your fleet, reduces latency in your deployment environment, and ensures your robots can resolve identities even when disconnected from the internet.

1. **Choose your node type:** Most organizations start with a Resolver node — a full read-replica that syncs from the root.
2. **Deploy the registry software:** The RRF registry software will be open source (OSI-approved license). Distribution packages (Docker / Helm) are in preparation — star the GitHub repo to be notified on release.
3. **Configure synchronization:** Point your node at the RRF root or an existing resolver. Sync interval is configurable; default is 5 minutes.
4. **Register your node (optional):** Register your resolver with the RRF to be listed in the public node directory and receive update notifications.

[Full technical documentation at rcan.dev §17](https://rcan.dev)

---

## Federated Resolution API

The RRF provides a federated RRN resolution endpoint:

```
GET /api/v1/resolve/:rrn
```

Where `:rrn` is a 12-digit canonical Robot Registration Number, e.g. `RRN-000000000001`.

Returns the authoritative record from the most-recently-synced node in the federation. See the [API docs](../api/index.md) for full reference.
