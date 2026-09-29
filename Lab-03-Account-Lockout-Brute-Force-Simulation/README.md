# Lab 3 – Account Lockout & Brute-Force Simulation

## Objective

Configure an Active Directory account lockout policy, intentionally trigger an account lockout, and investigate the resulting Windows Security event.

This lab builds on the failed logon monitoring performed in Lab 2 by showing what happens when repeated authentication failures reach the configured lockout threshold.

## Lab Environment

- Windows Server Domain Controller: `DC01`
- Active Directory domain: `lab.local`
- Domain-joined Windows client: `CLIENT`
- Test user: `jdoe`
- Group Policy Management
- Windows Event Viewer

## Account Lockout Policy

I configured the account lockout policy through the Default Domain Policy with the following settings:

- Account lockout threshold: **5 failed logon attempts**
- Account lockout duration: **15 minutes**
- Reset account lockout counter after: **15 minutes**

I also used the `net accounts` command on DC01 to verify that the policy was applied.

## Testing

From the domain-joined CLIENT machine, I intentionally entered an incorrect password for `jdoe` multiple times.

After reaching the configured threshold, Windows prevented the account from logging in and displayed:

> The referenced account is currently locked out and may not be logged on to.

I then reviewed the Security log on DC01 to investigate the lockout.

## Event ID 4740 Investigation

Event Viewer recorded **Event ID 4740 – A user account was locked out**.

The event showed:

- Locked account: `jdoe`
- Domain: `LAB`
- Caller Computer Name: `CLIENT`
- Domain Controller recording the event: `DC01`

This allowed me to identify both the affected account and the workstation associated with the lockout.

## Screenshots

### Account Lockout Policy

![Account Lockout Policy](Account%20Lockout%20Policy%20Configured.png)

The Default Domain Policy was configured to lock an account after five invalid logon attempts.

### Policy Verification

![Net Accounts Output](net%20accounts%20output%20cmd.png)

The `net accounts` command confirmed the five-attempt threshold and 15-minute lockout settings.

### Locked Account on CLIENT

![User jdoe Locked Out](User%20jdoe%20Locked%20out%20Message.png)

After repeated incorrect passwords, the `jdoe` account was locked and could no longer log in.

### Event ID 4740

![Event ID 4740](4740%20Event%20Log.png)

DC01 recorded Event ID 4740 after the account lockout occurred.

### Event ID 4740 Details

![Event ID 4740 Details](4740%20Event%20Log%20Details%20Tab.png)

The event details identify `jdoe` as the target account and `CLIENT` as the caller computer associated with the lockout.
## What I Learned

This lab showed me how an Active Directory account lockout policy affects authentication and how the resulting lockout can be investigated using Windows Security logs.

I also learned how Event ID 4740 can be used to identify the locked account and the computer associated with the lockout.
