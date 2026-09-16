# Screenshots

Drop your proof screenshots here using the filenames referenced in the phase docs. This is the
"evidence" half of the write-up — the verification shots matter more than the setup shots.

| Filename | What it should show | Referenced in |
|---|---|---|
| `01-get-addomain.png` | `Get-ADDomain` output (DNSRoot `ad.boydlab.net`, NetBIOSName `BOYDLAB`) | Phase 1 |
| `02-ou-tree.png` | Completed OU tree in ADUC (Boyd Industries + departments + sub-OUs) | Phase 2 |
| `03-agdlp-nesting.png` | `Finance-Share-Read` Members tab showing nested `Finance Staff` group | Phase 3 |
| `04-effective-access.png` | Effective Access — Fiona allowed, Jim denied | Phase 4 |
| `05-partofdomain-true.png` | `PartOfDomain : True` on the client | Phase 5 |
| `06-gpo-applied-finance.png` | Policy wallpaper on Fiona's (Finance) desktop | Phase 6 |
| `06-gpo-not-applied-sales.png` | Default wallpaper on Jim's (Sales) desktop | Phase 6 |

**Tips**
- PNG keeps text crisp; crop to just the relevant window.
- You can black out anything you'd rather not show, though lab-internal names/IPs are harmless.
- If you rename a file, update the matching reference in the phase doc so the image renders on
  GitHub.
