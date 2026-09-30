# Lab 4 – Splunk Forwarder & Log Ingestion

## Objective

Configure Splunk to receive Windows Security logs through the Splunk Universal Forwarder and verify that the events can be searched and analyzed from a centralized logging platform.

This lab builds on the Windows authentication monitoring performed in the previous labs by forwarding Windows Security events into Splunk for centralized analysis.

## Lab Environment

- Windows Server Domain Controller: `DC01`
- Active Directory domain: `lab.local`
- Domain-joined Windows client: `CLIENT`
- Splunk Enterprise
- Splunk Universal Forwarder
- Windows Security Event Logs

## Splunk Configuration

I configured Splunk Enterprise to receive forwarded data on TCP port `9997`.

The Splunk Universal Forwarder was configured to send Windows Security logs to the Splunk server. After configuring the receiving port and forwarder, I verified that Windows Security events were successfully being indexed in Splunk.

## Log Analysis

I used Splunk Search & Reporting to investigate failed Windows authentication events.

I searched for Event ID 4625 and filtered the results for the `jdoe` account using:

`index=main EventCode=4625 Account_Name=jdoe`

The search returned multiple failed authentication events associated with the account.

I then expanded an individual event to examine additional authentication information recorded in the Windows Security log.

## What I Learned

- How to configure Splunk to receive forwarded Windows event logs.
- How the Splunk Universal Forwarder sends Windows Security logs to a centralized Splunk server.
- How to verify that Windows Security events are being successfully indexed in Splunk.
- How to search for failed authentication attempts using Event ID 4625.
- How to filter authentication events for a specific user and examine individual event fields.
- How centralized logging can be used to investigate Windows security activity.
