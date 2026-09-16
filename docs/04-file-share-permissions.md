# Phase 4 — File Share with Layered Permissions

**Goal:** Create an SMB file share whose access is governed by the AGDLP chain from Phase 3,
then verify the restriction actually holds — an in-scope user gets in, an out-of-scope user
doesn't. This is the **P** (Permissions) in AGDLP made real.

---

## Key concepts — two permission layers

A shared folder has **two independent permission layers**, and both apply:

- **NTFS permissions** — live on the folder itself (filesystem). Apply **however** the folder
  is reached (network, local, RDP). Granular; the "real" permission layer.
- **Share permissions** — live on the network share. Apply **only** over the network via
  `\\server\share`. A network-only gate.

**When both apply (network access), the *most restrictive* of the two wins.** If Share = Full
Control but NTFS = Read, the effective result is Read. Accessed *locally*, share permissions
don't apply at all — NTFS alone governs.

**Best practice (used here):** set Share permissions wide (Authenticated Users → Full Control)
and control actual access with **NTFS**. This keeps a single authoritative layer (NTFS) and
avoids the confusion of debugging two restrictive layers.

---

## Build

1. Created `C:\FinanceShare` on DC01.
2. **Advanced Sharing** → shared as `FinanceShare` → share permission: removed `Everyone`,
   added **Authenticated Users → Full Control** (the wide-open door).
3. **Security tab (NTFS):** added `Finance-Share-Read` with **Read / Read & execute / List
   folder contents**. Left Write/Modify unchecked (it's a read group).

---

## Locking it down — breaking inheritance

Newly created folders **inherit** NTFS permissions from the parent (`C:\`), which included
broad entries (e.g. `Users`) granting read to *any* authenticated user. That would make the
carefully-added `Finance-Share-Read` group redundant and leave the folder effectively open.

Fix — **disable inheritance**, converting inherited entries to explicit ones (the safe option),
then remove the broad entries:

- **Advanced → Disable inheritance → "Convert inherited permissions into explicit
  permissions."**
- Removed the broad `Users` entry.
- Kept `SYSTEM`, `Administrators`, `CREATOR OWNER`, and `Finance-Share-Read`.

Final NTFS entries: `SYSTEM`, `Administrators`, `CREATOR OWNER`, `Finance-Share-Read (Read)` —
nothing granting broad domain-wide access. *Now* the folder is genuinely restricted to the
AGDLP chain.

> This "I added a group but everyone can still get in" scenario — leftover inherited broad
> access — is a common real-world file-permission ticket. Checking inheritance is the fix.

---

## Verification

Used **Advanced → Effective Access** to prove the model, testing two users:

- **`ffinance` (Fiona, in Finance)** → green checks on Read / List / Read & execute. She gets
  in **through the chain** (Fiona → Finance Staff → Finance-Share-Read → NTFS Read).
- **`jjales` (Jim, in Sales)** → all denied. He's outside the chain.

The in-scope-reads / out-of-scope-blocked contrast confirms the group nesting, the NTFS grant,
and the inheritance break all work together.

Later (Phase 5), Fiona also accessed the share **over the network** as a domain user from the
client via `\\DC01\FinanceShare`, confirming real end-to-end access — read allowed, write
denied, exactly as configured.

> **Screenshot:** `screenshots/04-effective-access.png` — Effective Access showing Fiona
> allowed and Jim denied.
