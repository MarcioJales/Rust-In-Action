# Explaining "http.rs"

```rust
let neighbor_cache = NeighborCache::new(BTreeMap::new());
```

The [neighbor cache](https://learning.oreilly.com/library/view/tcp-ip-illustrated-volume/9780132808200/ch08.xhtml#:-:text=8.5.4.%20Neighbor%20Unreachability%20Detection%20(NUD)) is concept from IPv6 used when running the neighbor discovery mechanism. It is backed by a data structure and holds IPv6-to-link-layer-address mapping information required to perform direct delivery of IPv6 datagrams to on-link neighbors as well as information regarding the state of the mapping.

It will be passed to the Ethernet interface later on the moment of build