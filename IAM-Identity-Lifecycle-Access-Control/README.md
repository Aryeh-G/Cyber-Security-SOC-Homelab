# Lab 11 – IAM Identity Lifecycle & Access Control

## Objective

Build and test an identity and access management workflow in Active Directory using group-based access control.

In this lab, I created a test user and simulated a Joiner, Mover, and Leaver lifecycle. I used Active Directory security groups to control access to Finance and HR resources, tested the resulting permissions from a domain-joined workstation, and reviewed the corresponding Windows Security events in Splunk.

## Lab Environment

- Windows Server Domain Controller: DC01
- Active Directory domain: `lab.local`
- Domain-joined Windows CLIENT
- Active Directory Users and Computers
- Windows file shares
- Windows Security auditing
- Splunk Enterprise

## Access Control Design

I used an AGDLP-style group structure to manage access to departmental resources.

Global groups represented users by department:

- `GG_Finance_Users`
- `GG_HR_Users`

Domain Local groups represented permissions to resources:

- `DL_Finance_RW`
- `DL_HR_RW`

The groups were nested so that:

```text
Finance User
    ↓
GG_Finance_Users
    ↓
DL_Finance_RW
    ↓
Finance Share
```

and:

```text
HR User
    ↓
GG_HR_Users
    ↓
DL_HR_RW
    ↓
HR Share
```

This allowed access to be managed through group membership instead of assigning permissions directly to individual users.

### Finance Group Nesting

![Finance Group Nesting](gg%20finance%20member%20of%20DL%20finance.png)

`GG_Finance_Users` was added as a member of `DL_Finance_RW`.

## Joiner – Creating and Provisioning a User

I created a test user named **Jordan Rivera** in Active Directory.

Jordan was initially assigned to:

```text
GG_Finance_Users
```

Because `GG_Finance_Users` was nested inside `DL_Finance_RW`, Jordan inherited access to the Finance resource through group membership.

### Finance Membership

![Jordan Finance Membership](3-%20New%20User%20Jordan%20Rivera%20assigned%20Finance%20Users%20group(1).png)

## Finance Share Permissions

The Finance shared folder was configured so that the `DL_Finance_RW` group had Change and Read permissions.

### Finance Share Configuration

![Finance Share Permissions](4-%20share%20permissions%20for%20Finance%20folder%20(1).png)

This connected the Active Directory group structure to an actual network resource.

## Testing Finance Access

I logged into CLIENT as Jordan Rivera and accessed the Finance share hosted on DC01.

I created a test file inside the Finance share to verify that Jordan's group membership provided the expected write access.

### Successful Finance Access

![Finance Access Test](5-%20Creating%20test%20file%20under%20Finance%20folder%20while%20logged%20in%20as%20the%20client%20jordan(1).png)

The successful file creation confirmed that the Finance permissions were working as intended.

## Testing Unauthorized HR Access

Before assigning Jordan to HR, I attempted to access the HR share.

Jordan was not a member of `GG_HR_Users`, so the account did not receive permissions through `DL_HR_RW`.

### HR Access Denied

![HR Access Denied](8-%20Tried%20to%20map%20HR%20share%20folder%20as%20user%20Jordan%20and%20did%20not%20have%20access(1).png)

Windows denied access to the HR share, confirming that the group-based access restrictions were working.

## Mover – Finance to HR

I then simulated an employee changing departments.

Jordan was removed from:

```text
GG_Finance_Users
```

and added to:

```text
GG_HR_Users
```

### Updated Group Membership

![Jordan HR Membership](9-%20Removed%20GG_finance_users%20from%20Jordan%20Rivera%20User%20and%20added%20to%20GG_HR_Users(1).png)

The HR share was configured to grant access through `DL_HR_RW`.

### HR Share Permissions

![HR Share Permissions](10-%20configured%20share%20permissions%20for%20HR%20shared%20folder%20and%20added%20DL_HR_RW%20group(1).png)

After the group membership change, Jordan could access the HR resource while the previous Finance access was no longer available.

### Access After Department Change

![Mover Access Validation](11-%20Can%20access%20HR%20Shared%20folder%20and%20Cannot%20access%20Finance%20Shared%20folder(1).png)

This demonstrated how changing Active Directory group membership can modify a user's resource access without assigning permissions directly to the user.

## Leaver – Disabling the Account

Finally, I simulated the user leaving the organization.

Jordan's access groups were removed, the account was disabled, and the user was moved into the `Disabled Users` organizational unit.

### Disabled User

![Disabled User](12-%20%20Disabled%20User%20Jordan%20Rivera%20and%20removed%20HR%20group%20and%20moved%20user%20to%20disabled%20users(1).png)

I then attempted to authenticate using the disabled account.

### Disabled Account Login Test

![Disabled Account Login](13%20-%20User%20Jordan%20Rivera%20has%20been%20disabled%20and%20this%20message%20appears%20when%20user%20trys%20to%20connect(1).png)

Windows rejected the authentication attempt and reported that the account had been disabled.

## Monitoring IAM Activity

I used Windows Security auditing and Splunk to review identity-related events generated during the lifecycle changes.

The investigation included events associated with:

- User creation
- Group membership changes
- Account changes
- Account disabling

### Event ID 4720 – User Account Created

Event ID `4720` recorded the creation of the Jordan Rivera account.

![Event 4720](14.2%20Joiner%20-%20Jordan%20Created%20lowkey%20might%20need%20to%20go%20first(1).png)

This provides an audit record showing when the identity was created.

### Event ID 4732 – Group Nesting

Event ID `4732` captured a member being added to a security-enabled local group.

![Event 4732](14.1%20First-%204732%20AGDLP%20nesting%20Demonstrates%20the%20global%20groups%20being%20nested%20into%20their%20domain%20loical%20groups%20for%20DGDLP(1).png)

This provided visibility into changes to the group structure used for resource access.

### Event ID 4729 – Group Membership Removal

Event ID `4729` recorded removal from a security-enabled global group during the access changes.

![Event 4729](14.3%20Mover%20-%20Error%20code%204729%20Jordan%20removed%20from%20the%20HR%20group%20during%20the%20move(1).png)

Group membership events can be used to track when users gain or lose access associated with departmental security groups.

### Event ID 4725 – Account Disabled

Event ID `4725` recorded the disabling of Jordan Rivera's account.

![Event 4725](14.5%20-%204725%20acount%20disabled.png)

This provided an audit trail for the final account disablement step of the lifecycle.

## Splunk Investigation

I searched the Windows Security logs in Splunk for activity associated with Jordan Rivera and the IAM security groups.

This allowed me to correlate administrative changes made in Active Directory with the security events generated by Windows.

![Splunk IAM Investigation](14-%20Searching%20on%20Splunk%20jordans%20account%20careations,%20changes,%20group%20membership%20activity%20and%20disabling(1).png)

Instead of relying only on the current state shown in Active Directory, the logs provided a historical record of changes made to the account and its group memberships.

## What I Learned

This lab helped me understand how identity lifecycle management, Active Directory groups, file permissions, and security monitoring work together.

I practiced the three major stages of an identity lifecycle:

**Joiner:** Create the identity and provision the appropriate access.

**Mover:** Modify access when the user's role or department changes.

**Leaver:** Remove access and disable the identity when it is no longer required.

I also gained hands-on experience using an AGDLP-style permission model rather than assigning permissions directly to individual users.

Finally, reviewing the activity in Windows Security logs and Splunk showed how identity changes can be investigated and audited after they occur.
