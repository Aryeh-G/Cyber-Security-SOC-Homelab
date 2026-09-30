# Lab 6 – Incident Investigation & Log Analysis in Splunk

## Objective

Investigate a simulated authentication incident in Splunk by analyzing Windows Security logs associated with a domain user.

The goal of this lab was to use the authentication data collected in previous labs to reconstruct activity, identify repeated failed logins, determine their source, and correlate the failures with account lockout events.

## Lab Environment

- Windows Server Domain Controller: `DC01`
- Active Directory domain: `lab.local`
- Domain-joined Windows client: `CLIENT`
- Test user: `jdoe`
- Splunk Enterprise
- Splunk Universal Forwarder
- Windows Security Event Logs

## Investigation

### Identify Relevant Authentication Events

I started by searching for failed authentication and account lockout events associated with `jdoe`.

```spl
index=main (EventCode=4625 OR EventCode=4740) jdoe
```

This allowed me to review both failed logon events and account lockouts associated with the user.

### Build an Authentication Timeline

I organized the relevant events into a table containing the timestamp, Event ID, account name, source network address, and Splunk host.

```spl
index=main (EventCode=4625 OR EventCode=4740) jdoe
| table _time, EventCode, Account_Name, Source_Network_Address, host
| sort _time
```

This made it easier to review the sequence of authentication activity.

### Count Failed Authentication Attempts

I counted the number of Event ID 4625 events associated with `jdoe`.

```spl
index=main EventCode=4625 jdoe
| stats count
```

The search returned multiple failed authentication events for the account.

### Identify the Source

I grouped the failed authentication events by source network address.

```spl
index=main EventCode=4625 jdoe
| stats count by Source_Network_Address
| sort -count
```

This allowed me to determine which source was associated with the failed authentication activity.

### Analyze the Authentication Pattern

I used a timechart to examine when the failed authentication events occurred.

```spl
index=main EventCode=4625 jdoe
| timechart count
```

This provided a timeline of failed authentication activity and helped show how the events were distributed over time.

### Correlate Account Lockouts

I searched for Event ID 4740 to identify account lockout events associated with `jdoe`.

```spl
index=main EventCode=4740 jdoe
```

I then compared the failed authentication activity with the account lockout events.

### Compare Event Counts

Finally, I grouped the authentication events by Event ID.

```spl
index=main (EventCode=4625 OR EventCode=4740) jdoe
| stats count by EventCode
```

This provided a summary of failed logon and account lockout activity observed during the investigation.

## Investigation Findings

The investigation identified repeated failed authentication activity associated with the `jdoe` account.

The Splunk searches showed:

- Multiple Event ID 4625 failed authentication events
- Failed authentication activity associated with the same source network address
- Event ID 4740 account lockout events associated with `jdoe`
- A pattern of failed authentication activity followed by account lockout

Because the activity was intentionally generated in the lab, these events represent a simulated brute-force scenario rather than an unknown real-world attack.

## Screenshots

### Identify Relevant Authentication Events

![Relevant Authentication Events](Starting%20from%20the%20Incident%20-%20What%20events%20exist%20for%20the%20user.png)

I began the investigation by searching for Event ID 4625 and Event ID 4740 activity associated with `jdoe`.

### Build the Investigation Timeline

![Investigation Timeline](Building%20Investigation%20table%20%5Dstep%202.png)

I organized the authentication events by time, Event ID, account name, source network address, and host to review the sequence of activity.

### Count Failed Authentication Attempts

![Failed Authentication Count](Count%20Failed%20Attempts%20for%20Specific%20User%20step%203.png)

I counted the Event ID 4625 events associated with `jdoe` to determine the amount of failed authentication activity.

### Identify the Source

![Authentication Source](Identify%20Attacker%20by%20I.P%20Step%204.png)

I grouped failed authentication events by `Source_Network_Address` to identify the source associated with the attempts.

### Analyze the Authentication Timeline

![Authentication Timeline](Checking%20Attack%20Pattern%20%28Timeline%29%20Step%205.png)

I used `timechart` to examine how the failed authentication events were distributed over time.

### Correlate Failed Logons and Account Lockouts

![Event Correlation](Correlating%20Everything--%20How%20many%20failures%20vs%20lockouts%20step%206.png)

I compared Event ID 4625 and Event ID 4740 activity to correlate failed authentication attempts with account lockout events.

## What I Learned

- How to investigate authentication activity using Splunk.
- How to narrow an investigation to a specific user account.
- How to build a timeline of security events using SPL.
- How to count and group authentication events to identify patterns.
- How to identify the source network address associated with failed authentication attempts.
- How to correlate Event ID 4625 failed logons with Event ID 4740 account lockouts.
- How multiple Windows Security events can be combined to reconstruct a simulated authentication incident.
