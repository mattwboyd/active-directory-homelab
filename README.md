# Active Directory Domain Services — Home Lab

A self-directed home lab in which I built a functioning on-premises Active Directory
environment from scratch: promoting a domain controller, designing an OU structure,
implementing a group-based access model, joining a client, and enforcing configuration
through Group Policy — all documented with the reasoning behind each decision and
verification that each step actually worked.

> **This is a self-directed learning project, not production experience.** I built it to
> develop hands-on on-premises AD skills to complement my cloud identity (Entra ID)
> background as I transition into IT support / MSP work.

---

## Why this lab exists

I hold the **AZ-900** certification, am actively studying for **AZ-104** and **CompTIA A+**,
and already have hands-on **Entra ID (cloud identity)** experience. What I *didn't* have was
on-premises Active Directory experience — the half of identity that most small/mid-size
businesses and MSPs still run every day.

This lab fills that gap. It was modeled on the "Active Directory Basics" blueprint from
Jake's Tech Labs, but I ran it locally in **VirtualBox** instead of the cloud to keep it
free — the AD skills transfer identically, and I already had the cloud-provisioning half
covered from my Azure work.

The end goal was two things at once: a *working* environment, and *portfolio-ready evidence*
of the skills — with the reasoning shown, not just the clicks.

---

## Environment / architecture

| Component | Detail |
|---|---|
| Hypervisor | Oracle VirtualBox |
| Domain Controller | Windows Server 2025 Standard — hostname `DC01` |
| Client | Windows 11 Pro — hostname `BOYDS2` |
| Domain (DNS name) | `ad.boydlab.net` |
| Domain (NetBIOS) | `BOYDLAB` |
| Networking | VirtualBox Internal Network (`intnet`), isolated |
| DC01 IP | `192.168.10.10` /24 — DNS points to itself |
| Client IP | `192.168.10.20` /24 — DNS points to `192.168.10.10` (DC01) |
| Tooling | ADUC / GPMC (GUI) + PowerShell (scripting) |

**Logical layout**

```
Forest: ad.boydlab.net
└── Domain: ad.boydlab.net  (NetBIOS: BOYDLAB)
    └── OU: Boyd Industries
        ├── Finance ── Users / Computers / Groups
        ├── HR      ── Users / Computers / Groups
        ├── IT      ── Users / Computers / Groups
        └── Sales   ── Users / Computers / Groups
```

---

## Skills demonstrated

- **AD DS deployment** — installing the role, promoting a domain controller, creating a new forest/domain, integrated DNS
- **DNS fundamentals** — SRV-record location of domain controllers; why the client's DNS *must* point at the DC
- **OU design** — hierarchical structure built for Group Policy targeting and delegation, not just tidiness
- **PowerShell automation** — idempotent scripting (`New-ADOrganizationalUnit`, `New-ADUser`, `New-ADGroup`, `Add-ADGroupMember`), loops, existence-checks, secure credential handling
- **Group strategy (AGDLP)** — global vs. domain local scope, group nesting, role-vs-resource separation
- **File-share security** — Share vs. NTFS permission layers, most-restrictive-wins, breaking inheritance, verification via Effective Access
- **Domain join & authentication** — client networking, cross-machine Kerberos authentication of a domain user
- **Group Policy** — creating/linking a GPO, User vs. Computer configuration, SYSVOL asset staging, scope verification
- **Troubleshooting** — diagnosing loop-terminating errors, DNS resolution issues, and Windows OOBE workarounds

---

## Build log (by phase)

Each phase links to a detailed write-up with the reasoning, the exact commands, and
verification screenshots.

| Phase | What was built | Doc |
|---|---|---|
| 1 | Domain controller promotion — new forest, domain, DNS | [01-dc-promotion.md](docs/01-dc-promotion.md) |
| 2 | OU structure — designed and scripted | [02-ou-structure.md](docs/02-ou-structure.md) |
| 3 | Users & the AGDLP access model | [03-users-and-agdlp.md](docs/03-users-and-agdlp.md) |
| 4 | File share with layered permissions | [04-file-share-permissions.md](docs/04-file-share-permissions.md) |
| 5 | Client domain join & authentication | [05-domain-join.md](docs/05-domain-join.md) |
| 6 | Group Policy — create, link, scope, verify | [06-group-policy.md](docs/06-group-policy.md) |

---

## Selected proof

A few verification points that show the environment actually works (full screenshots in
each phase doc):

- `Get-ADDomain` confirming the forest/domain built correctly
- Effective Access showing an in-scope user (Finance) can read a share while an out-of-scope
  user (Sales) cannot
- `PartOfDomain : True` confirming the client joined
- The same Group Policy applying to a Finance user and *not* to a Sales user — proving
  correct scope targeting

---

## What I'd build next

- **Entra ID Connect / hybrid identity** — the natural progression, tying this on-prem
  directory to the cloud identity work I already have experience with
- Additional GPOs targeting the Computer configuration half
- A second domain controller to demonstrate replication and FSMO roles

---

*Built and documented by Matthew as part of an IT support / MSP career transition.*
