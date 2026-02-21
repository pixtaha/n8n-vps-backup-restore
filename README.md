# n8n Workflows Backup & Restore (Docker · Self-Hosted VPS)

## Scope

This document covers two operations:
- **Backup** — Export all workflows from a self-hosted n8n instance and download to local machine
- **Restore** — Upload and import workflows into a new n8n instance

> Covers **workflows only** (no credentials, no executions).

---

## Environment

| | |
|---|---|
| OS | Ubuntu 22.04 LTS (VPS) |
| Runtime | Docker |
| Access | SSH via PowerShell (Windows) |

---

## Preconditions

* SSH access to VPS as `root`
* Docker installed and running
* n8n running in a Docker container
* Port `22` open for SSH

---

# 📤 Part 1: Backup (Export)

## Step 1: SSH into the VPS

```bash
ssh root@<VPS_IP>
```

## Step 2: Find the n8n container name

```bash
docker ps
```

> Look for the container running `n8nio/n8n` image — note the name in the `NAMES` column.
> It may not always be `n8n` — e.g. `n8n-pw1p-n8n-1`

## Step 3: Enter the n8n container

> `bash` is not available in the official n8n image — use `sh`

```bash
docker exec -it <CONTAINER_NAME> sh
```

## Step 4: Export all workflows

```bash
n8n export:workflow --all --output=/home/node/workflows.json
```

### Expected output
```
Successfully exported <N> workflows.
```

## Step 5: Verify the file

```bash
ls -lh /home/node/workflows.json
```

## Step 6: Exit the container

```bash
exit
```

## Step 7: Copy file from container to VPS filesystem

```bash
docker cp <CONTAINER_NAME>:/home/node/workflows.json /root/workflows.json
```

## Step 8: Exit the VPS

```bash
exit
```

## Step 9: Download to local machine

> Run from **local PowerShell only** — never from inside SSH

```powershell
scp root@<VPS_IP>:/root/workflows.json C:\Users\<USERNAME>\workflows.json
```

---

# 📥 Part 2: Restore (Import)

## Step 1: Upload file to the new VPS

> Run from **local PowerShell**

```powershell
scp C:\Users\<USERNAME>\workflows.json root@<NEW_VPS_IP>:/root/workflows.json
```

## Step 2: SSH into the new VPS

```bash
ssh root@<NEW_VPS_IP>
```

> If you get a `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED` error:
> ```powershell
> ssh-keygen -R <NEW_VPS_IP>
> ```
> Then reconnect.

## Step 3: Find the n8n container name

```bash
docker ps
```

## Step 4: Copy file into the container

```bash
docker cp /root/workflows.json <CONTAINER_NAME>:/home/node/workflows.json
```

## Step 5: Enter the container

```bash
docker exec -it <CONTAINER_NAME> sh
```

## Step 6: Import all workflows

```bash
n8n import:workflow --input=/home/node/workflows.json
```

### Expected output
```
Importing workflows...
Done
```

## Step 7: Exit and verify

```bash
exit
```

Open n8n in the browser and confirm workflows are imported.

---

## File Characteristics

* `workflows.json` is **minified (single-line JSON)** — expected behavior
* Ready for: import, backup storage, Git versioning

## What This Backup Contains

* ✅ All workflows
* ✅ Nodes and connections
* ✅ Workflow settings

## What This Backup Does NOT Contain

* ❌ Credentials
* ❌ Executions history
* ❌ User accounts

---

## Prompt Reference

| Prompt | Location |
|--------|----------|
| `PS C:\Users\...>` | Local machine |
| `root@server:~#` | Inside VPS |
| `~ $` | Inside n8n container |

---

## Known Issues

### `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED`
```powershell
ssh-keygen -R <VPS_IP>
```
Then reconnect normally.

### `Connection timed out` during `scp`
Close PowerShell completely, open a new window, re-run the command.

### `No such file or directory` during `scp`
You're running `scp` from inside SSH — exit first, run from local PowerShell.

---

## ⚠️ Re-running Export

Running the export command multiple times **overwrites** the same file.
To keep versioned backups:

```bash
n8n export:workflow --all --output=/home/node/workflows_2026-02-21.json
```
---
## Related

- [n8n-vps-backup](https://github.com/pixtaha/n8n-vps-backup) — Export workflows from n8n Docker instance to local machine
