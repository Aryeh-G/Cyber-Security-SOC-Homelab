# IAM Identity Lifecycle & Access Control

## Objective

Build and test an identity and access management workflow in Active Directory using group-based access control.

In this lab, I created a test user and simulated a Joiner, Mover, and Leaver identity lifecycle. I used Active Directory security groups to control access to Finance and HR resources, tested the resulting permissions from a domain-joined workstation, and reviewed the corresponding Windows Security events in Splunk.

## Lab Environment

- Windows Server Domain Controller: `DC01`
- Active Directory domain: `lab.local`
- Domain-joined Windows workstation: `CLIENT`
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

The groups were nested so that Finance access followed:

`User → GG_Finance_Users → DL_Finance_RW → Finance Share`

HR access followed:

`User → GG_HR_Users → DL_HR_RW → HR Share`

This allowed access to be managed through group membership instead of assigning permissions directly to individual users.

## Finance Group Nesting

`GG_Finance_Users` was added as a member of `DL_Finance_RW`.

![Finance Group Nesting](1-%20Finance%20Users%20is%20a%20member%20of%20DL%20Finance%20RW.png)

## HR Group Nesting

`GG_HR_Users` was added as a member of `DL_HR_RW`.

![HR Group Nesting](2-%20GG%20HR%20Users%20is%20a%20member%20of%20DL%20HR%20RW.png)

## Joiner – Creating and Provisioning a User

I created a test user named Jordan Rivera in Active Directory.

Jordan was initially assigned to:

- `GG_Finance_Users`

Because `GG_Finance_Users` was nested inside `DL_Finance_RW`, Jordan received access to the Finance resource through group membership.

![Jordan Finance Membership](3-%20New%20User%20Jordan%20Rivera%20assigned%20Finance%20Users%20group.png)

## Finance Share Permissions

The Finance shared folder was configured so that `DL_Finance_RW` had Change and Read permissions.

![Finance Share Permissions](4-%20share%20permissions%20for%20Finance%20folder%20.png)

This connected the Active Directory group structure to an actual network resource.

## Testing Finance Access

I logged into CLIENT as Jordan Rivera and accessed the Finance share hosted on DC01.

I created a test file inside the Finance share to verify that Jordan's group membership provided the expected write access.

![Finance Access Test](5-%20Creating%20test%20file%20under%20Finance%20folder%20while%20logged%20in%20as%20the%20client%20jordan.png)

The successful file creation confirmed that the Finance permissions were working as intended.

## HR Share Configuration

The HR shared folder was configured to grant access through `DL_HR_RW`.

![HR Share Permissions](6-%20Permission%20sharing%20settings%20for%20HR%20share%20folder.png)

The folder's Security permissions were also configured for the HR access group.

![HR Security Permissions](7-%20Security%20settings%20for%20HR%20share%20folder.png)

## Testing Unauthorized HR Access

Before assigning Jordan to HR, I attempted to access the HR share.

Jordan was not a member of `GG_HR_Users`, so the account did not receive permissions through `DL_HR_RW`.

![HR Access Denied](8-%20Tried%20to%20map%20HR%20share%20folder%20as%20user%20Jordan%20and%20did%20not%20have%20access.png)

Windows denied access to the HR share, confirming that the group-based access restrictions were working.

## Mover – Finance to HR

I then simulated an employee changing departments.

Jordan was removed from:

- `GG_Finance_Users`

and added to:

- `GG_HR_Users`

![Jordan HR Membership](9-%20Removed%20GG_finance_users%20from%20Jordan%20Rivera%20User%20and%20added%20to%20GG_HR_Users.png)

The HR share was configured to provide access through `DL_HR_RW`.

![HR Share Group Configuration](10-%20configured%20share%20permissions%20for%20HR%20shared%20folder%20and%20added%20DL_HR_RW%20group.png)

After the group membership change, Jordan could access the HR resource while the previous Finance access was no longer available.

![Mover Access Validation](11-%20Can%20access%20HR%20Shared%20folder%20and%20Cannot%20access%20Finance%20Shared%20folder.png)

This demonstrated how changing Active Directory group membership can modify a user's resource access without assigning permissions directly to the user.

## Leaver – Disabling the Account

Finally, I simulated the user leaving the organization.

Jordan's access group membership was removed, the account was disabled, and the user was moved into the Disabled Users organizational unit.

![Disabled User](12-%20Disabled%20User%20Jordan%20Rivera%20and%20removed%20HR%20group%20and%20moved%20user%20to%20disabled%20users.png)

I then attempted to authenticate using the disabled account.

![Disabled Account Login](13-%20User%20Jordan%20Rivera%20has%20been%20disabled%20and%20this%20message%20appears%20when%20user%20trys%20to%20connect.png)

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

![Event 4720](14.2%20Joiner%20-%20Jordan%20Created%20lowkey%20might%20need%20to%20go%20first.png)

This provided an audit record showing when the identity was created.

### Event ID 4732 – Group Nesting

Event ID `4732` recorded a member being added to a security-enabled local group.

![Event 4732](14.1%20First-%204732%20AGDLP%20nesting%20Demonstrates%20the%20global%20groups%20being%20nested%20into%20their%20domain%20loical%20groups%20for%20DGDLP.png)

This provided visibility into changes to the group structure used for resource access.

### Event ID 4729 – Group Membership Removal

Event ID `4729` recorded removal from a security-enabled global group during the department change.

![Event 4729](14.3%20Mover%20-%20Error%20code%204729%20Jordan%20removed%20from%20the%20HR%20group%20during%20the%20move.png)

Another Event ID `4729` captured removal of Jordan from the Finance group.

![Finance Group Removal](14-4-%20Leacer%20Access%20removal%20-%20Jordan%20removed%20from%20Finance%20Group.png)

These events provide an audit trail of group membership changes that affect a user's access.

### Event ID 4725 – Account Disabled

Event ID `4725` recorded the disabling of Jordan Rivera's account.

![Event 4725](14.5%20-%204725%20account%20disabled.png)

This provided an audit trail for the final account disablement step of the lifecycle.

### Event ID 4738 – Account Changed

Event ID `4738` recorded changes made to the Jordan Rivera account.

![Event 4738](14.6%20-%20Account%20Changed.png)

This provided additional visibility into changes made to the identity during the lifecycle process.

## Splunk Investigation

I searched the Windows Security logs in Splunk for activity associated with Jordan Rivera and the IAM security groups.

![Splunk IAM Investigation](14-%20Searching%20on%20Splunk%20jordans%20account%20careations%2C%20changes%2C%20group%20membership%20activity%20and%20disabling.png)

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
