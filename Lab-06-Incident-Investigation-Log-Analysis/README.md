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

## What I Learned

- How to investigate authentication activity using Splunk.
- How to narrow an investigation to a specific user account.
- How to build a timeline of security events using SPL.
- How to count and group authentication events to identify patterns.
- How to identify the source network address associated with failed authentication attempts.
- How to correlate Event ID 4625 failed logons with Event ID 4740 account lockouts.
- How multiple Windows Security events can be combined to reconstruct a simulated authentication incident.
