# Day 3–4: Windows Log Auditing – Enable & Collect

## Objective

Enable Windows log auditing for:
- Successful and failed logon events (Event IDs 4624, 4625)
- Process creation events (Event ID 4688)

---

## Environment

- OS: Windows 10 Home Edition (no gpedit.msc)
- Tool used: `auditpol` (Command Line Audit Policy Editor)
- Purpose: Configure log auditing without access to Group Policy Editor

---

## Audit Policy Configuration via `auditpol`

### Problem
`gpedit.msc` is unavailable on Windows Home Edition.

### Solution
Used `auditpol` to manually enable necessary auditing policies.

### Commands Executed:

```cmd
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Process Creation" /success:enable
