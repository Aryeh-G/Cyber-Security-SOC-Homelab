# IAM Project – Identity Lifecycle & Access Control

## Objective

Build and test an identity and access management workflow in Active Directory using group-based access control.

In this project, I created a test user and simulated a Joiner, Mover, and Leaver lifecycle. I used Active Directory security groups to control access to Finance and HR resources, tested the resulting permissions from a domain-joined workstation, and reviewed the corresponding Windows Security events in Splunk.

## Environment

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

Domain Local groups were used to assign permissions to departmental resources:

- `DL_Finance_RW`
- `DL_HR_RW`

The Finance access path was:

```text
Finance User
    ↓
GG_Finance_Users
    ↓
DL_Finance_RW
    ↓
Finance Share
```

The HR access path was:

```text
HR User
    ↓
GG_HR_Users
    ↓
DL_HR_RW
    ↓
HR Share
```

This allowed access to be managed through group membership instead of assigning resource permissions directly to individual users.

### Finance Group Nesting

`GG_Finance_Users` was added as a member of `DL_Finance_RW`.

![Finance Group Nesting](1-%20Finance%20Users%20is%20a%20member%20of%20DL%20Finance%20RW.png)

### HR Group Nesting

`GG_HR_Users` was added as a member of `DL_HR_RW`.

![HR Group Nesting](2-%20GG%20HR%20Users%20is%20a%20member%20of%20DL%20HR%20RW.png)

## Joiner – Creating and Provisioning a User

I created a test user named **Jordan Rivera** in Active Directory and initially assigned the account to:

```text
GG_Finance_Users
```

Because `GG_Finance_Users` was nested inside `DL_Finance_RW`, Jordan received Finance access through group membership.

### Initial Finance Membership

![Jordan Finance Membership](3-%20New%20User%20Jordan%20Rivera%20assigned%20Finance%20Users%20group.png)

## Finance Resource Access

The Finance shared folder was configured so that `DL_Finance_RW` had Change and Read permissions.

### Finance Share Permissions

![Finance Share Permissions](4-%20share%20permissions%20for%20Finance%20folder%20.png)

I then logged into CLIENT as Jordan and accessed the Finance share hosted on DC01.

To validate write access, I created a test file inside the Finance share.

### Successful Finance Access

![Finance Access Test](5-%20Creating%20test%20file%20under%20Finance%20folder%20while%20logged%20in%20as%20the%20client%20jordan.png)

The successful file creation confirmed that Jordan received the expected Finance access through the group structure.

## Testing Unauthorized HR Access

Before moving Jordan to HR, I attempted to access the HR share using the same account.

Jordan was not a member of `GG_HR_Users`, so the account did not receive HR permissions through `DL_HR_RW`.

### HR Access Denied

![HR Access Denied](8-%20Tried%20to%20map%20HR%20share%20folder%20as%20user%20Jordan%20and%20did%20not%20have%20access.png)

Windows denied access to the HR share, confirming that the access restrictions were working as intended.

## Mover – Finance to HR

I simulated an employee transferring from the Finance department to HR.

Jordan was removed from:

```text
GG_Finance_Users
```

and added to:

```text
GG_HR_Users
```

### Updated Group Membership

![Jordan HR Membership](9-%20Removed%20GG_finance_users%20from%20Jordan%20Rivera%20User%20and%20added%20to%20GG_HR_Users.png)

The HR share was configured to grant Change and Read access through `DL_HR_RW`.

### HR Share Permissions

![HR Share Permissions](10-%20configured%20share%20permissions%20for%20HR%20shared%20folder%20and%20added%20DL_HR_RW%20group.png)

I then tested Jordan's access again.

Jordan could access the HR share while access to the previous Finance resource was denied.

### Access After Department Change

![Mover Access Validation](11-%20Can%20access%20HR%20Shared%20folder%20and%20Cannot%20access%20Finance%20Shared%20folder.png)

This demonstrated how changing group membership can modify resource access without assigning permissions directly to an individual account.

## Leaver – Removing Access and Disabling the Account

Finally, I simulated Jordan leaving the organization.

I removed the remaining departmental group membership, disabled the account, and moved the user into the `Disabled Users` organizational unit.

### Disabled User

![Disabled User](12-%20Disabled%20User%20Jordan%20Rivera%20and%20removed%20HR%20group%20and%20moved%20user%20to%20disabled%20users.png)

I then attempted to authenticate using the disabled account.

### Disabled Account Login Test

![Disabled Account Login](13%20-%20User%20Jordan%20Rivera%20has%20been%20disabled%20and%20this%20message%20appears%20when%20user%20trys%20to%20connect.png)

Windows rejected the login attempt and reported that the account had been disabled.

## Monitoring IAM Activity

I used Windows Security auditing and Splunk to review events generated during the identity lifecycle.

This provided an audit trail of actions such as:

- User creation
- Group membership changes
- Account changes
- Group membership removal
- Account disabling

## Event ID 4720 – User Account Created

Event ID `4720` recorded the creation of the Jordan Rivera account.

![Event ID 4720](14.2%20Joiner%20-%20Jordan%20Created%20lowkey%20might%20need%20to%20go%20first.png)

This provided an audit record showing when the identity was created.

## Event ID 4732 – Group Membership Addition

Event ID `4732` recorded a member being added to a security-enabled local group.

In this project, the event provided evidence of the global groups being nested into the Domain Local groups used for resource permissions.

![Event ID 4732](14.1%20First-%204732%20AGDLP%20nesting%20Demonstrates%20the%20global%20groups%20being%20nested%20into%20their%20domain%20loical%20groups%20for%20DGDLP.png)

## Event ID 4729 – Group Membership Removal

Event ID `4729` recorded a member being removed from a security-enabled global group.

These events provided an audit trail of access changes made during the Mover and Leaver portions of the project.

![Event ID 4729](14.3%20Mover%20-%20Error%20code%204729%20Jordan%20removed%20from%20the%20HR%20group%20during%20the%20move.png)

## Event ID 4725 – Account Disabled

Event ID `4725` recorded the disabling of Jordan Rivera's account.

![Event ID 4725](14.5%20-%204725%20acount%20disabled.png)

The event identified the target account and provided an audit record of the account being disabled during the Leaver process.

## Splunk Investigation

I searched the Windows Security logs in Splunk for activity associated with Jordan Rivera and the IAM security groups.

![Splunk IAM Investigation](14-%20Searching%20on%20Splunk%20jordans%20account%20creations%2C%20changes%2C%20group%20membership%20activity%20and%20disabling.png)

Using Splunk allowed me to review the historical activity associated with the account instead of relying only on the current state displayed in Active Directory.

This connected the administrative IAM actions performed in Active Directory with the security events generated by Windows.

## What I Learned

This project helped me understand how identity lifecycle management, Active Directory groups, resource permissions, and security monitoring work together.

I practiced the three major stages of an identity lifecycle:

**Joiner:** Create an identity and provision the appropriate access.

**Mover:** Modify access when a user's role or department changes.

**Leaver:** Remove access and disable the identity when it is no longer required.

I also gained hands-on experience implementing an AGDLP-style permission model, validating authorized and unauthorized resource access, and reviewing identity-related Windows Security events in Splunk.

The project showed how changes to an identity can affect actual resource access and how those administrative changes can later be investigated through security logs.
