# Lab 7 – Active Directory Network Traffic Analysis with Wireshark

## Objective

Capture and analyze network traffic generated within an Active Directory environment to better understand the protocols involved in domain communication and authentication.

In this lab, I used Wireshark to examine DNS, Kerberos, and SMB traffic between a domain-joined Windows client and the Domain Controller.

## Lab Environment

- Windows Server Domain Controller: `DC01`
- Active Directory domain: `lab.local`
- Domain-joined Windows client: `CLIENT`
- VMware Workstation
- Wireshark
- Windows Command Prompt

## Traffic Capture

I captured traffic from the Windows client using Wireshark and applied display filters to isolate specific Active Directory-related protocols.

The main protocols investigated were:

- DNS
- Kerberos
- SMB2

## DNS Analysis

I applied the following Wireshark display filter:

```text
dns
```

I generated DNS traffic from the client and reviewed the resulting queries and responses.

The capture showed DNS communication involving the lab environment as well as external name resolution.

This demonstrated the role DNS plays in resolving hosts and supporting communication within an Active Directory environment.

## Kerberos Analysis

I filtered the capture for Kerberos authentication traffic.

```text
kerberos
```

The capture contained Kerberos authentication exchanges including:

- AS-REQ
- AS-REP

I also observed Kerberos error traffic during the capture, including pre-authentication failures.

Reviewing these packets provided visibility into communication between the client and Domain Controller during Kerberos authentication.

## SMB Analysis

I applied the following filter:

```text
smb2
```

The capture showed SMB2 session setup traffic between the Windows client and server.

The captured packets included:

- SMB2 Session Setup Request
- SMB2 Session Setup Response
- NTLMSSP Negotiate
- NTLMSSP Challenge
- NTLMSSP Authentication
- Logon failure response

This provided a network-level view of an SMB authentication attempt and its associated authentication exchange.

## Screenshots

### DNS Traffic

![DNS Traffic](1%20DNS%20Resolution_%20Internal%20DNS%20working_Forwarders%20working.png)

Wireshark was used to isolate DNS traffic and review internal and external name resolution activity.

### Kerberos Authentication Traffic

![Kerberos Authentication](2%20Kerberos%20Authentication%20.png)

Kerberos traffic captured between the client and Domain Controller showed authentication requests and responses, including AS-REQ and AS-REP messages.

### SMB Session Setup

![SMB Session Setup](3%20SMB%20Session%20Setup%20.png)

The SMB2 capture showed the session setup process and NTLMSSP authentication exchange between the client and server.

## What I Learned

- How to capture and filter network traffic using Wireshark.
- How DNS traffic appears during Windows network communication.
- How to identify Kerberos authentication requests and responses in packet captures.
- How to recognize SMB2 session setup traffic.
- How NTLMSSP authentication can appear within SMB2 communication.
- How packet analysis provides additional visibility into authentication activity beyond Windows event logs and SIEM searches.
