# Active Directory Basics 🏢

---

## Table of Contents
- [OUs vs Security Groups](#ous-vs-security-groups)
- [Managing Users in AD](#managing-users-in-ad)
- [Managing Computers in AD](#managing-computers-in-ad)
- [Group Policies (GPOs)](#group-policies-gpos)
- [Authentication Methods](#authentication-methods)
- [Trees, Forests, and Trusts](#trees-forests-and-trusts)

---

## OUs vs Security Groups

| Feature | OUs | Security Groups |
|---|---|---|
| Purpose | Apply policies (GPOs) | Assign permissions |
| Membership | One OU only | Many groups allowed |
| Used for | Configuration and policy | Access control |

---

## Managing Users in AD

### Deleting OUs
OUs are protected from accidental deletion by default.

Steps to delete:
1. AD Users and Computers → View → enable **Advanced Features**.
2. Right-click OU → Properties → **Object tab**.
3. Uncheck **"Protect object from accidental deletion."**
4. Delete the OU.

> ⚠️ Deleting an OU removes all users and groups inside it.

### Organizing Users
- Create or delete users inside the correct OUs.
- Keep departments clean and organized.
- Make sure each user is placed in the correct OU to match the organizational chart.

### Delegation — Giving Limited Admin Powers
Used to give non-admin users specific permissions on an OU (e.g., password resets).

Steps:
1. Right-click the target OU → **Delegate Control**.
2. Add the user (e.g., `phillip`).
3. Choose the specific permission (e.g., "Reset user passwords and force password change at next logon").
4. Finish.

### Testing Delegation via PowerShell

Reset a user's password:
```powershell
Set-ADAccountPassword sophie -Reset -NewPassword (Read-Host -AsSecureString -Prompt 'NewPassword') -Verbose
```

Force password change at next login:
```powershell
Set-ADUser -ChangePasswordAtLogon $true -Identity sophie -Verbose
```

> If you get a complexity error, use a strong password like `Pass1234!`

---

## Managing Computers in AD

When computers join a domain they go into the default "Computers" container. Best practice is to organize them into separate OUs:

| OU | Contents |
|---|---|
| Workstations | Regular user PCs and laptops |
| Servers | Machines providing services |
| Domain Controllers | Most sensitive systems — already in their own default OU |

This organization makes it easier to apply different Group Policies to each device type.

---

## Group Policies (GPOs)

**GPOs (Group Policy Objects)** — Collections of settings applied to OUs. Can target users or computers.

### How GPOs Work
1. Create a GPO under "Group Policy Objects."
2. Link it to the target OU.
3. Policies apply to that OU and all sub-OUs.
4. Limit scope using **Security Filtering**.

### Default Domain Policy
- Applied at the domain root → affects all computers.
- Contains basic settings like password policies and account lockout policies.

### GPO Distribution
- Stored in the **SYSVOL** shared folder on Domain Controllers.
- Changes can take up to **2 hours** to propagate.
- Force update: `gpupdate /force`

---

## Authentication Methods

Windows domains use two main authentication protocols:

| Protocol | Type | Description |
|---|---|---|
| Kerberos | Modern, default | Uses tickets. Secure. |
| NetNTLM | Legacy | Challenge-response. Kept for compatibility. |

Both rely on the **Domain Controller** to verify credentials.

### Kerberos Authentication (Default)
Uses tickets instead of repeatedly sending passwords. Domain Controller acts as the **KDC (Key Distribution Center)**.

**How it works:**
1. User sends username + encrypted timestamp to the KDC.
2. KDC returns a **TGT (Ticket Granting Ticket)** — encrypted with the `krbtgt` account hash.
3. User presents TGT + service SPN to the KDC to request access to a specific service.
4. KDC returns a **TGS (Ticket Granting Service)** — encrypted with the service account's password hash.
5. User presents TGS to the service → access granted if valid.

> Key idea: Authenticate once, get a TGT, use it to access multiple services without re-entering credentials.

### NetNTLM Authentication (Legacy)
Challenge-response process. No tickets involved.

**How it works:**
1. Client requests authentication from the server.
2. Server sends a random **challenge**.
3. Client uses NTLM hash + challenge to create a **response**.
4. Server forwards challenge + response to the Domain Controller.
5. DC validates by recalculating the response.
6. If it matches → authentication successful.

> The password or hash is never sent directly over the network.

- **Domain account** → DC verifies the response.
- **Local account** → Server verifies using the local **SAM** database.

---

## Trees, Forests, and Trusts

### Single Domain
Basic AD setup. All users, computers, and policies managed in one place.

### Trees
When a company grows and needs separate management per region, subdomains are created under the same namespace.

Example:
- Root: `thm.local`
- Subdomains: `uk.thm.local`, `us.thm.local`

**Benefits:**
- Each subdomain has its own Domain Controllers.
- Domain Admins only control their own domain.
- **Enterprise Admins** have control over all domains in the tree.

### Forests
When companies with different domain namespaces merge, their AD trees are joined into a **Forest**.

Example:
- Tree 1: `thm.local`
- Tree 2: `mht.local`
- Together = one Forest

### Trust Relationships
Allow users in one domain to access resources in another.

| Trust Type | Description |
|---|---|
| One-way | Domain AAA trusts BBB → BBB users can access AAA, but not vice versa |
| Two-way | Both domains trust each other → users from both sides can access each other's resources |

> A trust **enables** access — it does not **automatically grant** it. Admins still control what users can access across domains.
