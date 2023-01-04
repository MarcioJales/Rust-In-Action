# Explaining the `get` method on `http.rs`

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

...
...

let mut sockets = SocketSet::new(vec![]);
let tcp_handle = sockets.add(tcp_socket);
```

These sentences are strightforward. 2 buffers are defined and provided during creation of the TCP socket as required. `TcpSocketBuffer` is a type definition to `RingBuffer`. In the end, a handle is created from the addition of the TCP socket to a `SocketSet`. I don't know why this concept of "a set of sockets" exists. Anyway, there isn't such Struct in the new versinon (0.8.2) in `smoltcp`.

---

```rust
  let ip_addrs = [IpCidr::new(IpAddress::v4(192, 168, 42, 1), 24)];

  let fd = tap.as_raw_fd();
  let mut routes = Routes::new(BTreeMap::new());
  let default_gateway = Ipv4Address::new(192, 168, 42, 100);
  routes.add_default_ipv4_route(default_gateway).unwrap();
  let mut iface = EthernetInterfaceBuilder::new(tap)
    .ethernet_addr(mac)
    .neighbor_cache(neighbor_cache)
    .ip_addrs(ip_addrs)
    .routes(routes)
    .finalize();
```

These lines are responsible for building the `EthernetInterface` entity, except for `let fd = tap.as_raw_fd();`, which is going to be used later on a call to `phy_wait()`. 

`192.168.42.1` is assigned to this interface. It is in the range configured on the kernel firewall by `sudo iptables -t nat -A POSTROUTING -s 192.168.42.0/24 -j MASQUERADE`.

`192.168.42.100` is the default gateway since it was the IP assigned to the virtual device by `sudo ip addr add 192.168.42.100/24 dev tap-rust`.

The `route` variable consists of a **route table**. We add `default_gateway` as the default route, that is, traffic `0.0.0.0/0` is handled by it. This addition might also be achieved by the command line:

```bash
ip route add 0.0.0.0/0 via 192.168.42.100
```

And what is the effect of all these methods on `EthernetInterfaceBuilder`? Let's check in order:
---

