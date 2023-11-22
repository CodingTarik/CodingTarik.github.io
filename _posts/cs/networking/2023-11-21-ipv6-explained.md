---
Author: CodingTarik  
Date: 2023-11-21 20:55:00 +0800  
categories: [cs, networking]
tags: [ipv6, network]
title: "IPv6 Addresse explained"
---

# IPv6 Address Notation
Summary of:
https://de.wikipedia.org/wiki/IPv6, 
https://www.elektronik-kompendium.de/sites/net/2107111.htm
## Hexadecimal Notation

IPv6 addresses are typically written in hexadecimal (compared to decimal for IPv4). The address is divided into eight blocks, each containing 16 bits (4 hexadecimal digits). These blocks are separated by colons, as opposed to IPv4 addresses that use dots. An example of an IPv6 address in this notation is:

```plaintext
2001:0db8:85a3:08d3:1319:8a2e:0370:7344
```

## Omitting Leading Zeros

Leading zeros within a block can be omitted. For instance, the address:

```plaintext
2001:0db8:0000:08d3:0000:8a2e:0070:7344
```

is equivalent to:

```plaintext
2001:db8:0:8d3:0:8a2e:70:7344
```

## Omitting Consecutive Zero Blocks

One or more consecutive blocks with a value of 0 (or 0000) can be omitted by using two consecutive colons. For example:

```plaintext
2001:db8:0:0:0:0:1428:57ab
```

is equivalent to:

```plaintext
2001:db8::1428:57ab
```

## Reduction Rules

The reduction described in rule 3 should be performed only once. This means that at most one contiguous group of zero blocks in the address can be replaced. For instance, the address:

```plaintext
2001:0db8:0:0:8d3:0:0:0
```

can be reduced to either:

```plaintext
2001:db8:0:0:8d3::
```

or

```plaintext
2001:db8::8d3:0:0:0
```

Reducing it to `2001:db8::8d3::` is invalid, as it is ambiguous and could be mistakenly interpreted. It is recommended to reduce the group with the most zero blocks.

## Decimal Notation for the Last Two Blocks

For the last two blocks (four bytes or 32 bits) of the address, traditional decimal notation with dots as separators can be used. For example:

```plaintext
::ffff:127.0.0.1
```

is an alternative representation of:

```plaintext
::ffff:7f00:1
```

This notation is especially useful when embedding the IPv4 address space into the IPv6 address space.

# Network Notation in IPv6

IPv6 networks are represented in CIDR notation. This notation consists of the first address (or network address) and the prefix length in bits, separated by a forward slash. For example, `2001:0db8:1234::/48` represents the network with addresses ranging from `2001:0db8:1234:0000:0000:0000:0000:0000` to `2001:0db8:1234:ffff:ffff:ffff:ffff:ffff`. The size of an IPv6 network (or subnet) in terms of the number of assignable addresses in that network must be a power of two. Since a single host can also be considered a network with a 128-bit prefix, host addresses are sometimes written with an appended "/128".

## CIDR Notation Example

Consider the CIDR notation `2001:0db8:abcd::/64`. This represents a network with addresses from `2001:0db8:abcd:0000:0000:0000:0000:0000` to `2001:0db8:abcd:ffff:ffff:ffff:ffff:ffff`. The "/64" indicates that the first 64 bits represent the network prefix, leaving 64 bits for host addresses within the network.

## Subnet Size as a Power of Two

CIDR notation requires the size of the network or subnet to be a power of two. This ensures efficient allocation of address space. For instance, a "/48" network allows for 2^(128-48) = 2^80 assignable addresses.

## Host Address Notation

Since a single host can be treated as a network with a 128-bit prefix, host addresses may be written with a "/128" suffix. For example, `2001:0db8::1/128` denotes the host with the address `2001:0db8::1`.

In summary, CIDR notation provides a concise and standardized way to represent IPv6 networks, making it easier to manage and allocate address space efficiently.

# Special Addresses in IPv6

IPv6 introduces several special addresses that serve distinct purposes within the network architecture. Understanding these addresses is crucial for proper network management and communication. Let's delve into some of these notable addresses:

## Unspecified Address

- **Notation:** `::/128` or `0:0:0:0:0:0:0:0/128`
- **Description:** The unspecified address signifies the absence of a specific address. It should not be assigned to any host and is often utilized by initializing hosts as the source address in IPv6 packets until they receive their own address. Server programs can also use this address to listen on all addresses of the host, equivalent to `0.0.0.0/32` in IPv4.

## Default Route

- **Notation:** `::/0` or `0:0:0:0:0:0:0:0/0`
- **Description:** The default route is employed when no entry is found in the routing table. It is analogous to `0.0.0.0/0` in IPv4 and serves as the route of last resort.

## Loopback Address

- **Notation:** `::1/128` or `0:0:0:0:0:0:0:1/128`
- **Description:** The loopback address represents the local host, commonly associated with the term "localhost." In IPv4, `127.0.0.1/32` from the address range `127.0.0.0–127.255.255.255` fulfills a similar role, though IPv6 simplifies this with a dedicated `::1/128` address. This address is used for internal testing, diagnostics, and communication with the local system.

Understanding these special IPv6 addresses is essential for network administrators and developers, ensuring the proper functioning of network communication and routing.

# Link-Local Unicast Addresses in IPv6

IPv6 Link-Local addresses, defined within the `fe80::/10` address space, serve specific purposes within local network segments. These addresses are essential for communication and automatic configuration within closed network environments, eliminating the need for a DHCP server. Let's explore the characteristics and applications of Link-Local Unicast addresses.

## Characteristics of Link-Local Addresses

- **Address Range:** `fe80::/10`
- **Structure:** The first 10 bits are set to `1111111010`, followed by 54 bits with a value of `0`, resulting in a prefix of `fe80::/64`.
- **Usage:** Link-Local addresses are valid only within closed network segments, typically formed using switches or hubs, extending up to the first router.

### Address Components

- **10 Bits:** `1111111010`
- **54 Bits:** `0`
- **64 Bits:** Interface ID

## Applications and Usage

- **Segment-Specific Communication:** Link-Local addresses are utilized for addressing nodes within closed network segments. Communication using these addresses is restricted to the local network.
  
- **Autoconfiguration:** Link-Local addresses support autoconfiguration, enabling devices within a network segment to obtain addresses without relying on a DHCP server.

- **Neighbour Discovery:** These addresses play a crucial role in Neighbor Discovery, facilitating the identification and communication with devices within the same local network.

## Zone ID Specification

To communicate using a Link-Local address, the Zone ID must be specified. Since a Link-Local address can exist on a device multiple times, especially with multiple network interfaces, indicating the Zone ID becomes necessary.

Example Link-Local Address with Zone ID:

- `fe80::7645:6de2:ff:1%1`
- `fe80::7645:6de2:ff:1%eth0`

In the above examples, `%1` or `%eth0` represents the Zone ID, ensuring proper identification of the specific instance of the Link-Local address on the device.

Link-Local Unicast addresses provide a valuable mechanism for local network communication and configuration, contributing to the efficiency and simplicity of IPv6 networks.

# Understanding Site Local Unicast Addresses (Deprecated)

In the evolving landscape of networking protocols, IPv6 introduced Site Local Unicast Addresses as successors to private IPv4 addresses like those within the 192.168.x.x range. Operating within the fec0::/10 address space (from fec0... to feff...), these addresses, also known as site-local addresses, were designed to be routed only within the boundaries of the same organization.

## Characteristics of Site Local Unicast Addresses:

1. **Limited Routing Scope:**
   Site Local Unicast Addresses were intended for communication within a specific organization. They were not meant to be routed beyond the confines of the organization's network, ensuring a controlled and internal address space.

2. **Flexible Address Space Assignment:**
   Organizations had the flexibility to choose their preferred address space within the fec0::/10 range. This autonomy allowed for tailored network configurations, especially during mergers or the establishment of VPN connections between initially separate networks using Site Local Addresses.

3. **Deprecation and Transition:**
   Despite their initial utility, Site Local Addresses faced challenges such as address space conflicts when organizations merged or established VPN connections. In response to these issues and others, RFC 3879, released in September 2004, deprecated Site Local Unicast Addresses. The deprecation signaled that these addresses would phase out from future standards.

## Transition to Unique Local Addresses:

As successors to Site Local Unicast Addresses, Unique Local Addresses (ULA) emerged. These addresses, defined in subsequent standards, addressed the shortcomings of Site Local Addresses and aimed to provide a more robust and globally applicable solution for private addressing.

## Conclusion:

The deprecation of Site Local Unicast Addresses marked a transition towards more standardized and globally compatible addressing schemes. Organizations are encouraged to adopt the newer Unique Local Addresses for their internal networking needs. Understanding this shift is crucial for network administrators and engineers to align their systems with evolving best practices and standards in the ever-changing realm of networking technologies.

# Understanding Unique Local Unicast Addresses (ULA) in IPv6

In the realm of IPv6 addressing, Unique Local Addresses (ULA) provide a solution for private addressing similar to IPv4's private address ranges (e.g., 192.168.x.x). Defined in RFC 4193, ULAs operate within the fc00::/7 address space, specifically ranging from fc00... to fdff....

## Key Characteristics of Unique Local Addresses:

1. **Prefix Structure:**
   - **fc00:** Reserved for globally assigned, unique ULAs (assigned by provider).
   - **fd00:** Reserved for locally generated ULAs.

2. **Site-ID and Subnet-ID:**
   - Following the prefix, there are 40 bits allocated for the Site-ID. For ULAs with the fd prefix, this Site-ID is randomly generated, making it highly probable to be unique.
   - A subsequent 16-bit Subnet-ID designates a subnet within the Site.

3. **Example ULA Format:**
   - A ULA might look like fd9e:21a7:a92c:2323::1.
   - In this example, fd is the prefix for locally generated ULAs, 9e:21a7:a92c represents a randomly generated 40-bit Site-ID, and 2323 is an arbitrarily chosen Subnet-ID.

## Benefits of Probabilistic Site-IDs:

- **Address Collision Prevention:**
  - The use of highly probable unique Site-IDs reduces the likelihood of address collisions, especially when setting up tunnels between independently configured networks.

- **Packet Routing Behavior:**
  - Packets sent to an unreachable Site are more likely to be discarded, preventing unintended delivery to a local host with a coincidentally matching address.

## Local vs. Global Addresses:

- **Local Addresses:**
  - Local addresses, often associated with the fd prefix, are used for internal network communication within a specific organization or site.
  - Example: fd9e:21a7:a92c:2323::1

- **Global Addresses:**
  - Global addresses, identified by the fc prefix, are intended for broader use and can be routed globally.
  - Example: fc00:1234:5678:abcd::1

ULAs with the "fd" prefix are meant for locally generated addresses and are highly likely to be unique. However, when merging two networks using these addresses, address conflicts can occur. To mitigate this, it's advisable to algorithmically generate ULAs, reducing the risk of address space collisions when combining networks.

## Comparison with IPv4:

- **IPv4 Equivalent:**
  - ULAs with the fd prefix are comparable to private IPv4 addresses (e.g., 192.168.x.x) used within an organization.
  - Global ULAs with the fc prefix are akin to public IPv4 addresses, allowing for global routing.

Understanding the structure and usage of Unique Local Addresses is essential for network administrators to make informed decisions when configuring and managing IPv6 networks, ensuring efficient and secure communication within private network spaces.

# Multicast in IPv6

IPv6 reserves the address range **ff00::/8 (ff…)** for Multicast addresses. Following the Multicast prefix, there are 4 bits for flags and 4 bits for the scope.

## Flags:

- **0:** Permanent well-known Multicast addresses defined by IANA.
- **1:** (T-Bit set) Transient or dynamically assigned Multicast addresses.
- **3:** (P-Bit set, enforces T-Bit) Unicast-Prefix-based Multicast addresses (RFC 3306).
- **7:** (R-Bit set, enforces P- and T-Bit) Multicast addresses containing the Rendezvous-Point address (RFC 3956).

## Scopes:

1. **Interface-Local (1):** Packets never leave the interface (Loopback).
2. **Link-Local (2):** Not forwarded by routers and restricted to the subnet.
4. **Admin-Local (4):** The smallest range requiring specific administration in routers.
5. **Site-Local (5):** Routed but not by Border Routers.
8. **Organization-Local (8):** Routed by Border Routers but confined within the organization.
e. **Global (e):** Routed globally.
0, 3, f: Reserved ranges.
Remaining ranges are unassigned, left for administrators to define additional Multicast regions.

## Examples of well-known Multicast addresses:

- **ff01::1, ff02::1:** All Nodes Addresses, akin to Broadcast.
- **ff01::2, ff02::2, ff05::2:** All Routers Addresses, targeting all routers in a scope.

Multicast in IPv6 facilitates efficient communication to specific groups of hosts. It is a powerful mechanism for scenarios like broadcasting information to all nodes or routers within a network. Understanding the flags and scopes allows for precise control over the reach and behavior of Multicast traffic in an IPv6 environment.


## Understanding IPv6 Multicast Addresses: `ff02::1` vs. `ff02::2`

IPv6, the next-generation Internet Protocol, introduces multicast addresses to efficiently manage communication within a network. Two significant multicast addresses are `ff02::1` and `ff02::2`, serving specific roles in the IPv6 ecosystem.

### `ff02::1` - All Nodes on the Link

This multicast address, as defined in RFC 4861 (Neighbor Discovery for IP version 6), plays a crucial role in IPv6 networks. According to Section 4.3.2 of the RFC, `ff02::1` represents the "all-nodes multicast address." The specification mandates that every IPv6 node must subscribe to this multicast address on its connected links.

In practical terms, this means that every IPv6 device within a local network, regardless of whether it's a host or a router, listens to and communicates using the `ff02::1` multicast address. It facilitates a common communication channel for all nodes on the link.

### `ff02::2` - All Routers on the Link

Similarly, RFC 4861 designates `ff02::2` as the "all-routers multicast address." As outlined in Section 4.3.2 of the RFC, all IPv6 routers must join this multicast address on their connected links. This multicast address enables efficient communication specifically among routers within the network.

By having routers subscribe to `ff02::2`, they create a dedicated channel for exchanging routing information and other relevant data. This separation streamlines communication and ensures that routers can efficiently exchange necessary details without broadcasting to all nodes on the link.

### Practical Implications

- **Efficient Network Communication:** The use of multicast addresses enhances the efficiency of IPv6 networks by allowing nodes and routers to communicate selectively based on their roles.
  
- **Neighbor Discovery:** These multicast addresses are fundamental for Neighbor Discovery, a process vital for determining the presence and capabilities of neighboring devices on the same link.

- **Simplified Configuration:** IPv6 nodes and routers, by adhering to these multicast addresses, simplify the configuration process and ensure a standardized approach to communication.

Understanding the roles of `ff02::1` and `ff02::2` in IPv6 networks is essential for network administrators and engineers to optimize communication and streamline the exchange of critical information.

"
Reserved Multicast Addresses:   FF00:0:0:0:0:0:0:0
                                      FF01:0:0:0:0:0:0:0
                                      FF02:0:0:0:0:0:0:0
                                      FF03:0:0:0:0:0:0:0
                                      FF04:0:0:0:0:0:0:0
                                      FF05:0:0:0:0:0:0:0
                                      FF06:0:0:0:0:0:0:0
                                      FF07:0:0:0:0:0:0:0
                                      FF08:0:0:0:0:0:0:0
                                      FF09:0:0:0:0:0:0:0
                                      FF0A:0:0:0:0:0:0:0
                                      FF0B:0:0:0:0:0:0:0
                                      FF0C:0:0:0:0:0:0:0
                                      FF0D:0:0:0:0:0:0:0
                                      FF0E:0:0:0:0:0:0:0
                                      FF0F:0:0:0:0:0:0:0

   The above multicast addresses are reserved and shall never be
   assigned to any multicast group.

      All Nodes Addresses:    FF01:0:0:0:0:0:0:1
                              FF02:0:0:0:0:0:0:1

   The above multicast addresses identify the group of all IPv6 nodes,
   within scope 1 (interface-local) or 2 (link-local).

      All Routers Addresses:   FF01:0:0:0:0:0:0:2
                               FF02:0:0:0:0:0:0:2
                               FF05:0:0:0:0:0:0:2
                               " (Source: https://datatracker.ietf.org/doc/html/rfc4291)
# Global Unicast Addresses in IPv6

IPv6 Global Unicast Addresses are those not reserved for special purposes like multicast, unique local, or link-local addresses. These addresses are globally routable and unique. Let's explore the specific address ranges and their applications:

## 1. IPv4 Compatibility Addresses (Deprecated):

- **::/96:** Originally designated for IPv4 Compatibility addresses, using the last 32 bits for an embedded IPv4 address. Deprecated in RFC 4291 (February 2006) and no longer recommended for use.

## 2. IPv4-Mapped IPv6 Addresses:

- **0:0:0:0:0:ffff::/96:** Represents IPv4-mapped IPv6 addresses. The last 32 bits contain the IPv4 address. Routers can facilitate conversion between IPv4 and IPv6 using this mapping.

## 3. Globally Routable Addresses:

- **2000::/3:** This extensive range, from 2000... to 3fff..., is designated for globally routable Unicast addresses. These addresses are globally unique and assigned by IANA.

  - **2001- Addresses:** Allocated to providers for further distribution to their customers.
  - **2001::/32:** Used for the Teredo tunneling mechanism.
  - **2001:db8::/32:** Reserved for documentation purposes and should not represent actual network participants.

## 4. Tunneling Mechanisms:

- **2002- Addresses:** Indicate the 6to4 tunneling mechanism.

## 5. Regional Internet Registries (RIRs) Assignments:

- Addresses starting with **2003, 240, 260, 261, 262, 280, 2a0, 2b0, and 2c0:** Assigned by Regional Internet Registries (RIRs) for global Unicast purposes. Some of these ranges might not be fully allocated as of yet.

## 6. Test Network:

- **3ffe::/16:** Previously used for the 6Bone test network. This address range has been returned to IANA, following RFC 3701.

## 7. NAT64 Translation:

- **64:ff9b::/96:** Reserved for the NAT64 translation mechanism, as specified in RFC 6146.

Understanding these address ranges is crucial for network administrators and engineers to effectively manage and allocate IPv6 addresses in a global context. Each range serves specific purposes, from global internet routing to documentation and testing.