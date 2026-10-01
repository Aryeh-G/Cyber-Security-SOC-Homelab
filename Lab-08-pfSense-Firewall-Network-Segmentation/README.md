# Lab 8 – pfSense Firewall Deployment & Network Segmentation

## Objective

Deploy a pfSense firewall in VMware Workstation and move the Active Directory lab environment behind a separate internal network.

The goal was to create a more realistic network architecture where pfSense acts as the gateway between the internal lab network and the upstream VMware NAT network.

## Lab Environment

- VMware Workstation
- pfSense Community Edition
- Windows Server Domain Controller: `DC01`
- Active Directory domain: `lab.local`
- Domain-joined Windows client: `CLIENT`
- Windows DNS

## Network Architecture

Before deploying pfSense, the Windows virtual machines were connected directly to the VMware NAT network.

After deploying pfSense, the lab network was configured as:

```text
                    Internet
                       |
                  VMware NAT
                 192.168.28.x
                       |
                 pfSense WAN
                192.168.28.133
                       |
                   pfSense
                       |
                 pfSense LAN
                  192.168.1.1
                       |
                    LABLAN
                   /      \
                DC01      CLIENT
          192.168.1.10    DHCP
```

This separated the internal `192.168.1.0/24` lab network from the upstream VMware NAT network.

## pfSense Virtual Machine Configuration

The pfSense VM was configured with two network interfaces:

- **WAN:** VMware NAT
- **LAN:** VMware LAN Segment `LABLAN`

During installation, the interfaces were assigned as:

- `em0` – WAN
- `em1` – LAN

This allows pfSense to route traffic between the isolated lab network and the upstream network.

## pfSense Interface Configuration

After installation, pfSense was configured with the following interfaces:

### WAN

- Interface: `em0`
- Address obtained through DHCP
- WAN address: `192.168.28.133/24`

### LAN

- Interface: `em1`
- LAN address: `192.168.1.1/24`

The LAN interface became the default gateway for systems inside the lab.

## Domain Controller Network Configuration

DC01 was moved onto the `LABLAN` network and configured with a static IP address:

```text
IP Address:       192.168.1.10
Subnet Mask:      255.255.255.0
Default Gateway:  192.168.1.1
Preferred DNS:    192.168.1.10
```

The Domain Controller continues to use itself for DNS because it hosts DNS for the `lab.local` Active Directory domain.

pfSense serves as the default gateway, while DC01 remains responsible for Active Directory DNS.

## Client Network Configuration

The CLIENT virtual machine was also moved from the VMware NAT network to `LABLAN`.

The client was configured to obtain its IP address automatically on the internal network while using DC01 as its DNS server.

The resulting design is:

```text
CLIENT
   |
   | DNS
   v
DC01 (192.168.1.10)
   |
   | Default Route
   v
pfSense (192.168.1.1)
   |
   v
Internet
```

This allows the client to use Active Directory DNS while pfSense handles routing between the internal and external networks.

## Connectivity Testing

After changing the network configuration, I tested connectivity from DC01.

First, I verified connectivity to the pfSense LAN interface:

```text
ping 192.168.1.1
```

The ping completed successfully with no packet loss.

I then tested external connectivity:

```text
ping google.com
```

The hostname resolved successfully and returned replies, confirming that DNS resolution and outbound routing were functioning.

## pfSense Web Interface

After configuring the LAN interface, I accessed the pfSense management interface from the internal network at:

```text
https://192.168.1.1
```

Successfully accessing the WebGUI confirmed communication between the internal network and the pfSense LAN interface.

## Troubleshooting

During deployment, the Windows virtual machines initially remained connected to the previous VMware NAT network.

I corrected the VMware network configuration by moving the internal network adapters to:

```text
LAN Segment: LABLAN
```

I then updated the Windows IP configuration for the new `192.168.1.0/24` network.

After making the changes, DC01 could successfully communicate with the pfSense gateway and external hosts.

## Screenshots

### VMware LABLAN Configuration

![VMware LABLAN Configuration](VMware%20Network%20Adapter%20Settings_%20Changing%20from%20NAT%20adapter%20to%20LABLAN%20adapter.png)

The Windows virtual machine network adapter was moved from VMware NAT to the isolated `LABLAN` segment.

### pfSense Interface Assignment

![pfSense Interface Assignment](Assigning%20Interfaces%20within%20Pfsense%20installation.png)

During pfSense installation, separate interfaces were assigned for WAN and LAN connectivity.

### pfSense Interface Configuration

![pfSense Console](Pfsense%20console%20scrfeen_%20WAN%20IP%20LAN%20IP%20and%20Interface%20Assignments.png)

The pfSense console shows `em0` operating as the WAN interface and `em1` as the LAN interface. The LAN interface is configured as `192.168.1.1/24`.

### DC01 Network Configuration and Connectivity

![DC01 Network Configuration](DC%20ipconfig%20after%20pfsense_tested%20with%20pinging%20pfsense%20firewall%20IP%20and%20pinging%20google.com%20.png)

DC01 was configured with the static address `192.168.1.10`, pfSense at `192.168.1.1` as its default gateway, and itself at `192.168.1.10` as its DNS server.

Successful ping tests confirmed connectivity to both the pfSense firewall and an external host.

### pfSense Web Interface

![pfSense Web Interface](Successful%20login%20to%20Firewall.png)

The pfSense WebGUI was successfully accessed from the internal `LABLAN` network.

## What I Learned

This lab helped me understand the difference between placing virtual machines directly on the same VMware NAT network and placing an internal network behind a dedicated firewall and router.

I learned how WAN and LAN interfaces are separated in pfSense, how pfSense can act as the default gateway for an internal network, and why Active Directory systems continue to use the Domain Controller for DNS.

I also gained experience configuring virtual network segments, assigning static addressing, validating routing and DNS connectivity, and troubleshooting network connectivity after changing the lab architecture.
