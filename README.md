# Snakebite

This is a tool designed to punch NAT holes and directly connect Wireguard peers. It draws inspiration from STUN, 
but everything is achieved using Wireguard.


# Requirements

The following dependencies must be available:
- awk
- curl
- socat
- wireguard (and wg-quick)

The system depends on a publicly accessible server. Given that most of the traffic does not pass through it, a small server would be sufficient.


# How it works

Example:

Peers `A` and `B` want to connect directly. Both are behind NATs. There is a "fang" server `F` that helps to punch holes.

Process:
- Both `A` and `B` connect to `F` using wireguard.
- Each client asks F for the list of peers, and updates its local copy. This list includes up-to-date public IPs and ports for each peer.
- If now `A` tries to reach `B` and `B` tries to reach `A`, they will eventually succeed.

To accomplish this, two scripts are provided:
- `server`: should run in the server `F`. It provides the updated list of peers.
- `client`: used by each peer to request and updated list of peers. The results will be automatically applied to the interface.


# Notes

The use of a low `PersistentKeepalive` value is recommended, as it helps maintaining the holes punched.

Though the server will return a full list of peers, only those specified in the local configuration file will function immediately, as routing was automatically setup when calling `wg-quick up`. To support peers added dynamically, you'll need to additionally configure their routes. The typical command is:

`ip -4 route add <endpoint> dev <interface>`
