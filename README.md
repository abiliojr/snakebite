# Snakebite

This is a minimalistic hack designed to punch NAT holes and directly connect WireGuard peers. All the exchange happens within the WireGuard network.

__NOTES__:
- This code is the result of a quick and dirty experiment. Use it at your own risk.
- It might not work with some NATs.

# Requirements

The following dependencies must be satisfied:
- socat
- WireGuard

This code depends on a publicly accessible server (aka _fang server_). Given that most of the traffic does not pass through it, a small server should be sufficient.


# How it works

Example:

Peers _A_ and _B_ want to connect directly. Both are behind NATs. There is a _fang server_ _F_ that helps to punch holes.

Mechanism:
- Both _A_ and _B_ connect to _F_ using WireGuard.
- Each client asks F for the list of known peers, and updates its local copy. This list includes up-to-date public IPs and ports for each peer.
- If now _A_ tries to reach _B_ and _B_ tries to reach _A_, they should succeed.

To accomplish this, two scripts are provided:
- `server`: should run in the publicly reachable machine _F_. It provides the updated list of peers.
- `client`: used by each peer to request and updated list of peers. The results will be automatically applied to the interface.


# Tips and tricks

- __Always__ use an IP address inside of the WireGuard network as the listening IP of the _server_.

- The use of a low `PersistentKeepalive` value is recommended, as it helps maintain the holes punched.

- Although the _client_ will add all the peers returned by the _server_, routing might not happen automatically. Here are some options:
    - Manually add the routes using: `ip -4 route add <endpoint> dev <wg interface>`
    - Include the peers and their `AllowedIPs` in the _client_.
    - On the _client_ set the `AllowedIPs` of the _fang server_ peer so it includes a range (e.g., `10.10.10.0/24`). This way a route for that range will be set on interface bring-up.

- Except for the _fang server_, other entries in the configuration file are not expected to contain an `Endpoint`.

- Fixing the `ListenPort` on the client interface allows to bring it down and up while keeping the holes punched.

- You can always automate running the client by using `PostUp` in the WireGuard configuration file.