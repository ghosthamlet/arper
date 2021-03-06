# Arper

This tool was originally built for Istanbul Packathon 2016 (http://packathon.org)

**NOTE:** this tool is still under **HEAVY** development! **Use at your own risk**.

## Why

I want to see who is connecting to my WiFi network (or any other network for that matter) in real time.

Applications like [1] represent the state of the art in detecting nodes on a network. Their approach is based on periodic scanning of IP addresses using tools like Nmap. There are two problems with this approach:

1. It's not real time and it requires constant *polling* (and we all hate polling don't we?).
2. If there is a central entity in the network (e.g. a router) it might limit things like ICMP requests (ping), port scanning etc. Furthermore individual nodes can simply decide not to reply to any unauthenticated request, regardless of the protocol used (e.g. on OS X a user can set the machine in "stealth mode", i.e. the machine will stop replying to ICMP requests).
3. They work on top of the IP layer (OSI level 3) which means the packets they send have to be routed by an entity, likely a router.

> Why can't I just leave a RaspberryPi scanning IP addresses?

Because IP addresses can change on a regular basis, for example if you use DHCP. It's perfectly normal. MAC addresses, on the other hand, can only be spoofed and the average Joe most probably doesn't know how to do that.

## How

In every network based on the Ethernet protocol (like WiFi 802.11) once a node obtains an IP address (e.g. from a DHCP server) it advertises its pair (IP address, MAC address), that is, its virtual and physical addresses, to **every** node on the network in a peer-to-peer fashion using the ARP [2] protocol.

ARP packets **cannot be routed**. A central entity in the network has no control over what ARP packets are sent. Nodes can at most flood the network with packets (for example flooding with ARP responses is often used to carry on man-in-the-middle attacks).

This means that every single node on the network will receive ARP packets broadcasted by other nodes, in a completely passive manner. No polling required!

## What

Arper represents a new class of network detection tools. Arper can detect, in real time, when a new node in the network broadcasts its IP and MAC addresses. By monitoring a network device with Pcap it can passively listen for ARP packets. That *passively* is of extreme importance. It means there is virtually zero impact on performance and power usage while Arper is running.

## Issues

This and any other detection methods (including Nmap) do not work if the WiFi access point (which acts at the datalink layer) implements VLANs [3] \(a.k.a. "client isolation"). A VLAN is, in fact, a virtual channel established between the node and the AP, at the datalink layer, that prevents the node from communicating with any other node in the network, no matter what higher-level protocol is used. It's basically the equivalent of plugging a cable into a switch.

VLANs are usually implemented in places with a public network such as cafes, universities, companies etc.

## I want it

- `npm install --save arper` (as a local module in your project, assuming you created a `package.json`)
- `npm install -g arper` (as a global module, you might need to prepend `sudo`)

Arper's `monitor` function accepts three arguments:
- interface (String): the network interface to use.
- callback (Function): a callback for whenever a packet is received.
- pretty (Boolean): if true IP and MAC addresses will be passed as strings (e.g. `00:01:02:AA:BB:CC`) otherwise as an array of decimals (e.g. `[128, 100, 150, 80, 10, 35]`).

```
var arper = require("arper");
arper.monitor("en0", function(err, newNode) { // en0 is the default WiFi interface on Mac
  if (err)
    console.log("Something went wrong!");
  // Do something with newNode
}, true);
```

**Note:** according to how your OS handles permissions for network interfaces you might have to elevate `node`'s permissions by prepending `sudo`. For instance you might want to run the examples with `sudo npm start`.

## Middleware

Middleware for Arper can be easily attached. Do you want to receive a notification when a new node is connected? You can. Do you want to receive an SMS/Email/... when it happens? You can.

An example of dead simple logging middleware:

```
var arper = require("arper");

var loggingMiddleware = function(newNode) {
  console.log(newNode);
};

arper.addMiddleware(loggingMiddleware);

arper.monitor("en0", function(err, newNode) {
  ...
}, true);
```

Middleware functions are called in the order they are added and are **never** passed an error. Only the main callback does (see below).

## Examples

The `examples` directory includes some use cases of Arper by using specific middleware.

Each example can be run with `npm start`.

## Tests

Clone the repository or go to your `NODE_PATH` directory and `cd` to `arper`. Then run: 

`npm test`

## References

- [1] http://whoisonmywifi.com/
- [2] https://en.wikipedia.org/wiki/Address_Resolution_Protocol
- [3] https://en.wikipedia.org/wiki/Virtual_LAN

## F.A.Q.

- I something along the lines of `Error: (cannot open BPF device) /dev/bpf0: Permission denied`. How do I fix it?
    Make sure you are running `node` with enough permissions (i.e. use `sudo` on most systems).
