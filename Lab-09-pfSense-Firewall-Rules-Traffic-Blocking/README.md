# Lab 9 – pfSense Firewall Rules & Traffic Blocking

## Objective

Configure and test a custom firewall rule in pfSense to block outbound traffic, generate firewall events, and investigate the resulting logs.

This lab builds on the pfSense deployment from Lab 8 by using the firewall to actively control and monitor traffic from the internal lab network.

## Lab Environment

- pfSense firewall/router
- Windows Server Domain Controller: `DC01`
- Domain-joined Windows client: `CLIENT`
- Internal network: `192.168.1.0/24`
- pfSense LAN gateway: `192.168.1.1`
- VMware Workstation

## Firewall Rule Configuration

I created a custom rule on the pfSense LAN interface with the following configuration:

- Action: **Block**
- Interface: **LAN**
- Address Family: **IPv4**
- Protocol: **Any**
- Source: **LAN subnets**
- Destination: `8.8.8.8`
- Logging: **Enabled**
- Description: `Block Google DNS`

Although the rule was named `Block Google DNS`, the protocol was configured as `Any`. This means the rule blocked all IPv4 traffic from the LAN to `8.8.8.8`, not only DNS traffic.

## Firewall Rule Order

pfSense evaluates firewall rules from top to bottom and applies the first matching rule.

I placed the custom block rule above the default `allow LAN to any` rule. If the allow rule were evaluated first, traffic to `8.8.8.8` would be permitted before reaching the block rule.

## Testing the Rule

After applying the rule, I generated traffic from the internal network to `8.8.8.8`.

I tested connectivity using:

```cmd
ping 8.8.8.8
```

The requests timed out, confirming that pfSense was blocking the traffic.

## Firewall Log Investigation

I enabled logging on the custom rule and reviewed the firewall logs under:

`Status → System Logs → Firewall`

The logs showed blocked connections from the internal network to `8.8.8.8`.

One of the observed patterns was:

- Source: `192.168.1.10`
- Destination: `8.8.8.8`
- Destination Port: `53`
- Protocol: `UDP`
- Action: **Blocked**

`192.168.1.10` is DC01, which also provides DNS services for the Active Directory environment. The traffic to UDP port 53 showed outbound DNS activity being blocked by the firewall rule.

## Screenshots

### Default LAN Firewall Rules

![Default LAN Firewall Rules](default%20firewall%20LAN%20rules.png)

The original LAN rules allowed traffic from the internal network before the custom block rule was added.

### Custom Block Rule Configuration

![Block Rule Configuration](Creating%20FIrewall%20Rule%20Blocking%20Google%20DNS%20.png)

The custom LAN rule was configured to block traffic destined for `8.8.8.8` and log matching packets.

### Firewall Rule Test

![Firewall Blocking Traffic](Firewall%20Blocking%20Google%20DNS%20Working_%20Tested%20by%20pinging%20google%20dns%208.8.8.8%20on%20Client%20and%20its%20not%20working.png)

The block rule was placed above the default allow rule. A ping test to `8.8.8.8` resulted in 100% packet loss, confirming that the rule was working.

### Firewall Logs

![Firewall Logs](Viewing%20Firewall%20System%20Logs%20to%20see%20the%20blocked%20traffic%20entries%20appear.png)

The pfSense firewall logs recorded blocked traffic, including UDP port 53 traffic from DC01 to `8.8.8.8`.

## What I Learned

- How to create and apply firewall rules in pfSense.
- Why firewall rule order matters when allow and block rules overlap.
- How to generate traffic to test whether a firewall rule is working.
- How to enable firewall rule logging and investigate blocked connections.
- How to identify source IPs, destination IPs, ports, protocols, and firewall actions in pfSense logs.
- How DNS traffic appears as UDP port 53 in firewall logs.
