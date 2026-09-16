# Phase 2 — OU Structure

**Goal:** Design and build an Organizational Unit structure that supports Group Policy
targeting and administrative delegation — built once in the GUI, once in PowerShell.

---

## Key concepts

- **OUs exist for function, not tidiness.** The two real reasons to build them:
  1. **Group Policy targeting** — GPOs can be linked to OUs (and sites/domain), but **not to
     containers**. Objects left in the default `CN=Users` / `CN=Computers` containers cannot
     be targeted by a linked GPO except domain-wide.
  2. **Delegation** — you can grant limited admin rights over a single OU (e.g., let a
     helpdesk reset passwords for one department only).
- **Container vs. OU** — the default `Users` and `Computers` are *containers* (can't hold
  linked GPOs); `Domain Controllers` is an actual *OU*. This distinction is the whole reason
  to build a custom OU tree instead of using the defaults.
- **Separating Users and Computers** — GPOs have a User half and a Computer half. Keeping user
  and computer objects in separate OUs lets each policy target exactly the object type it's
  meant for, and avoids relying on filtering to keep policies off the wrong objects.

---

## Design

Department-first, object-type second — a hybrid that gives a delegation boundary at the
department level and clean GPO targeting at the object-type level:

```
Boyd Industries              (top-level OU — company-wide baseline GPOs link here)
├── Finance ── Users / Computers / Groups
├── HR      ── Users / Computers / Groups
├── IT      ── Users / Computers / Groups
└── Sales   ── Users / Computers / Groups
```

The default containers (`Users`, `Computers`, `Builtin`, etc.) are left intact — a custom OU
tree lives *alongside* the defaults, never replaces them. The `Domain Controllers` OU is left
untouched (it carries the Default Domain Controllers Policy the DC depends on).

> **Note on defaults:** New objects created without a specified OU land in the un-targetable
> default containers. In production, `redirusr` / `redircmp` can redirect those defaults into
> a chosen OU; day-to-day, objects are simply created directly in the correct OU.

---

## Build — GUI

Created the top-level `Boyd Industries` OU and the four department OUs in **Active Directory
Users and Computers** (ADUC), leaving *Protect container from accidental deletion* checked on
each.

## Build — PowerShell

Rather than 12 manual right-clicks for the sub-OUs, I scripted them. First the single-command
form to establish the pattern:

```powershell
New-ADOrganizationalUnit -Name "Users" -Path "OU=Sales,OU=Boyd Industries,DC=ad,DC=boydlab,DC=net"
```

Then a loop to create all object-type sub-OUs across every department. The final version is
**idempotent** — it checks whether each OU already exists before creating it, so it's safe to
re-run without erroring on duplicates:

```powershell
$departments = "Sales","Finance","HR","IT"
$objectTypes = "Users","Computers","Groups"

foreach ($dept in $departments) {
    foreach ($type in $objectTypes) {
        $path = "OU=$dept,OU=Boyd Industries,DC=ad,DC=boydlab,DC=net"
        if (-not (Get-ADOrganizationalUnit -Filter "Name -eq '$type'" -SearchBase $path -SearchScope OneLevel -ErrorAction SilentlyContinue)) {
            New-ADOrganizationalUnit -Name $type -Path $path
            Write-Host "Created $type in $dept"
        } else {
            Write-Host "Skipped $type in $dept (already exists)"
        }
    }
}
```

> **Screenshot:** `screenshots/02-ou-tree.png` — the completed OU tree in ADUC.

---

## Troubleshooting note

The first (non-idempotent) loop hit an "object already in use" error on the very first
iteration and **terminated the whole loop** — a good lesson that some PowerShell errors halt a
loop rather than skipping one item. Rewriting it with an existence-check (`Get-ADOrganizationalUnit`)
and `Write-Host` feedback made it idempotent and re-runnable — the professional pattern for any
provisioning script.
