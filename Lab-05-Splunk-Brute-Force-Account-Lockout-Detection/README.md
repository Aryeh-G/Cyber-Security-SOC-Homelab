# Lab 5 – Splunk Brute-Force & Account Lockout Detection

## Objective

Create and test Splunk detection rules for repeated failed authentication attempts and Active Directory account lockouts using Windows Security Event Logs.

This lab builds on the Windows log ingestion configured in Lab 4 by using the collected security events to create detection logic and scheduled Splunk alerts.

## Lab Environment

- Windows Server Domain Controller: `DC01`
- Active Directory domain: `lab.local`
- Domain-joined Windows client: `CLIENT`
- Splunk Enterprise
- Splunk Universal Forwarder
- Windows Security Event Logs

## Brute-Force Detection

I used Windows Event ID 4625 to identify failed authentication attempts and grouped the events by account name.

The detection search used was:

`index=main EventCode=4625`
`| stats count by Account_Name`
`| where count > 5`

This detection looks for accounts with more than five failed authentication events in the search window.

I also reviewed failed authentication activity by account and source network address to better understand where the authentication attempts were coming from.

## Brute-Force Alert

I converted the detection search into a scheduled Splunk alert named `Brute Force Detection`.

For lab testing, the alert was configured to:

- Search the previous 5 minutes of events
- Run every minute
- Trigger when the search returned results
- Add triggered events to Splunk's Triggered Alerts

I then generated failed authentication attempts and verified that the alert appeared in the trigger history.

## Account Lockout Detection

I also created a detection for Active Directory account lockouts using Windows Event ID 4740.

The search used was:

`index=main EventCode=4740`

This detects Windows Security events generated when an account becomes locked out.

I intentionally generated repeated failed logins against the `jdoe` account until the domain account lockout policy prevented further authentication.

## Account Lockout Alert

I converted the Event ID 4740 search into a second scheduled Splunk alert named `Account Lockout Detection`.

After generating the account lockout, I verified that the detection triggered and appeared in Splunk's alert trigger history.

This provided visibility into both repeated authentication failures and the resulting account lockout.

## What I Learned

- How to build Splunk detection logic using Windows Security Event IDs.
- How to use Event ID 4625 to identify repeated failed authentication attempts.
- How to use Event ID 4740 to detect Active Directory account lockouts.
- How to aggregate failed authentication events by account using Splunk SPL.
- How thresholds and time windows affect when a detection rule triggers.
- How to convert a Splunk search into a scheduled alert.
- How to verify that a detection is working using Splunk alert trigger history.
