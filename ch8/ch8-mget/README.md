# Explaining "http.rs"

```rust
let neighbor_cache = NeighborCache::new(BTreeMap::new());
```

The [neighbor cache](https://learning.oreilly.com/library/view/tcp-ip-illustrated-volume/9780132808200/ch08.xhtml#:-:text=8.5.4.%20Neighbor%20Unreachability%20Detection%20(NUD)) is concept from IPv6 used when running the neighbor discovery mechanism. It is backed by a data structure and holds IPv6-to-link-layer-address mapping information required to perform direct delivery of IPv6 datagrams to on-link neighbors as well as information regarding the state of the mapping.

It will be passed to the Ethernet interface later on the moment of build.

Documentation says that if `storage.len() == 0` (the BTreeMap in this case), the `new` method panics. On a sandbox environemnt, I've instantiated a `BTreeMap` and the length returned was 0. So, I don't know how simply providing `BTreeMap::new()` is supposed to work.

I assume that the use of `BTreeMap` is due to its (good) performance as a cache data structure.

---

```rust
let tcp_rx_buffer = TcpSocketBuffer::new(vec![0; 1024]);
let tcp_tx_buffer = TcpSocketBuffer::new(vec![0; 1024]);
let tcp_socket = TcpSocket::new(tcp_rx_buffer, tcp_tx_buffer);
```

These sentences are strightforward. 2 buffers are defined and provided during creation of the TCP socket as required. `TcpSocketBuffer` is a type definition to `RingBuffer`.