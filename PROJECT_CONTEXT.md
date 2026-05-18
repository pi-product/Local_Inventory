# Inventory Portal — Full Project Context

This document captures the complete context for the Local Inventory Portal project.
Upload this to any new Claude session to continue development from any device.

---

## Who I Am

- **Name:** Tianlin Zhao
- **Role:** Operations, logistics, and engineering lead at **Premium Invention**
- **Company:** Premium Invention — imports and distributes security cameras (S4-Pro, V3S, V4, P2 models) to North America/Europe/Oceania
- **Email:** product@premiuminvention.com
- **GitHub:** wyyzoo (tianlin.zhao@gmail.com)
- **Note:** I do not edit code directly — all code changes are made through Claude

---

## Project Overview

A Flask/SQLite web portal for managing the **Canada local warehouse inventory** of returned/refurbished security cameras. It runs on AWS EC2 and is accessible from any device via browser.

---

## Live Portal Access

| | |
|---|---|
| **URL** | http://35.162.133.146:5050 |
| **Read-only login** | `pi` / `inventory2026` |
| **Admin login** | `tzhao` / `P@55w0rd` |

---

## Infrastructure

| Component | Details |
|---|---|
| **EC2 instance** | `i-0194b27262756e62a`, t2.micro, us-west-2 (Oregon) |
| **Elastic IP** | `35.162.133.146` (permanent, won't change) |
| **OS** | Amazon Linux 2023 |
| **Service** | systemd `inventory-portal.service` (auto-starts on reboot, auto-restarts on crash) |
| **DB location on EC2** | `/home/ec2-user/inventory.db` (SQLite) |
| **Portal file on EC2** | `/home/ec2-user/inventory_portal.py` |
| **SSH key (on Mac)** | `~/.ssh/inventory-portal-key.pem` |
| **Security group** | `inventory-portal-sg` — port 5050 open to all, port 22 open to all |
| **AWS account** | `tzhao1980` (017040832260), free tier until Jan 2027 |
| **IAM user** | `tianlin-admin` (AdministratorAccess), access key configured in AWS CLI |
| **AWS CLI** | Configured on Mac, region us-west-2 |

---

## Code Repository

| | |
|---|---|
| **GitHub repo** | https://github.com/wyyzoo/Local_Inventory |
| **Main file** | `inventory_portal.py` |
| **Branch** | `main` |
| **Local copy (Mac only)** | `~/Projects/Local_Inventory/` |
| **Auto-deploy** | GitHub Action on push to `main` → SCP to EC2 → `systemctl restart inventory-portal` (~10 seconds) |

---

## Development Workflow

| Device | How to make code changes |
|--------|--------------------------|
| **MacBook** | Claude Code edits `~/Projects/Local_Inventory/inventory_portal.py` → git push → auto-deploys to EC2 |
| **iPad / iPhone / any browser** | Claude (claude.ai) writes the code change → paste into GitHub web editor at github.com/wyyzoo/Local_Inventory → commit to main → auto-deploys to EC2 |

**Key rule:** I never edit code directly. Claude writes all changes. On Mac, Claude Code handles git and deployment. On iPad, I paste Claude's changes into GitHub's web editor and commit.

---

## Database Schema

**File:** `inventory.db` (SQLite)

### `inventory` table
Columns: `id, sheet_row, date, model, version, category, rating, code, customer, ord, remark, status, booked_customer, booked_order, booked_purpose, booked_date, packaging`

- `status`: `available` or `booked`
- `category`: `Like New`, `Replacement`, `New`
- `packaging`: `Neutral` or `Customized`
- `rating`: float, range 5.0–10.0
- `model`: `S4`, `V3S`, `V4`

### `history` table
Columns: `id, date, model, version, category, rating, code, customer, ord, direction, reason`

- `direction`: `IN` or `OUT`
- `reason`: `Return`, `Sale`, `Swap`, `Replacement`, `Move`, etc.

**Current data:** ~80 inventory records (78 available, 2 booked); history has all inbound records

---

## Portal Features

### Header / Navigation
- Title: "Local Inventory — Canada"
- Tabs: 📦 Inventory | 📋 History
- Stat badges: Like New count, Replacement count, New count, Booked count
- Model counts: S4, V3S, V4
- Signed-in user shown top-right with Sign out button

### Inventory Tab
- Search bar (model, code, customer)
- Category filter tabs: All | Like New | Replacement | New | Booked
- **Columns:** Model | Version | Code | Category | Rtg | Inbound Date | Customer | Order | Packaging | Remark
- **Rating badge:** gradient colour — orange-ish at 5.0, green at 10.0
- **Packaging badge:** Neutral (grey) | Customized (blue)
- **Remark:** truncated with full tooltip on hover
- Checkbox selection for bulk actions

### Action Buttons (admin only — greyed out for read-only)
- **+ Check In** — add new camera to inventory (modal form)
- **Book Out** — book selected camera(s) out with purpose; includes "Other" free-text option
- **Edit** — edit any field on a selected record (shown when exactly 1 row selected)
- **Put Back** — revert booked cameras back to available
- **Complete** — move booked cameras to history as OUT records

### Check In Modal fields
Model, Version, Code, Category, Rating, Customer, Order, Inbound Date, Packaging, Remark

### Book Out Modal fields
Purpose (dropdown: Sale / Replacement / Swap / Move / Other), Customer, Order

### Edit Modal fields
All inventory fields editable: Model, Version, Code, Category, Rating, Customer, Order, Date, Packaging, Remark

### History Tab
- Columns: Date | Model | Version | Code | Category | Rtg | Customer | Order | Direction | Reason
- Direction badge: IN (green) | OUT (orange)

---

## Authentication System

Session-based login (Flask sessions). Login page at `/login`, logout at `/logout`.

| User | Password | Role |
|------|----------|------|
| `pi` | `inventory2026` | Read-only (Check In / Book Out / Edit / Put Back / Complete all disabled) |
| `tzhao` | `P@55w0rd` | Admin (full access) |

Server-side enforcement: all write API routes (`/api/checkin`, `/api/bookout`, `/api/putback`, `/api/complete`, `/api/update`) return 403 for read-only users.

---

## DB Backup (Mac only)

- **Schedule:** Daily at 8:00 AM via macOS launchd
- **Script:** `~/Projects/Local_Inventory/backup_db.sh`
- **Destination:** iCloud Drive → `Inventory Backup/inventory_YYYY-MM-DD.db`
- **Retention:** Last 14 days kept, older ones auto-deleted
- **launchd plist:** `~/Library/LaunchAgents/com.inventory.dbbackup.plist`

To run manually: `bash ~/Projects/Local_Inventory/backup_db.sh`

---

## Useful Commands (run from Mac terminal)

```bash
# SSH into EC2
ssh -i ~/.ssh/inventory-portal-key.pem ec2-user@35.162.133.146

# Check portal service status
ssh -i ~/.ssh/inventory-portal-key.pem ec2-user@35.162.133.146 "sudo systemctl status inventory-portal"

# Deploy code update manually (if not using GitHub auto-deploy)
scp -i ~/.ssh/inventory-portal-key.pem ~/Projects/Local_Inventory/inventory_portal.py ec2-user@35.162.133.146:~/
ssh -i ~/.ssh/inventory-portal-key.pem ec2-user@35.162.133.146 "sudo systemctl restart inventory-portal"

# Pull latest code from GitHub to local Mac
git -C ~/Projects/Local_Inventory pull

# Push local changes to GitHub (triggers auto-deploy)
git -C ~/Projects/Local_Inventory add inventory_portal.py
git -C ~/Projects/Local_Inventory commit -m "your message"
git -C ~/Projects/Local_Inventory push

# Run manual DB backup
bash ~/Projects/Local_Inventory/backup_db.sh

# Check EC2 instance
aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,State.Name,PublicIpAddress]' --output table
```

---

## Key File Locations

| File | Location |
|------|----------|
| Portal source (Mac) | `~/Projects/Local_Inventory/inventory_portal.py` |
| Portal source (EC2) | `/home/ec2-user/inventory_portal.py` |
| Database (EC2, live) | `/home/ec2-user/inventory.db` |
| DB backup (iCloud) | `iCloud Drive/Inventory Backup/inventory_YYYY-MM-DD.db` |
| SSH key | `~/.ssh/inventory-portal-key.pem` |
| GitHub Action | `~/Projects/Local_Inventory/.github/workflows/deploy.yml` |
| Backup script | `~/Projects/Local_Inventory/backup_db.sh` |
| Backup scheduler | `~/Library/LaunchAgents/com.inventory.dbbackup.plist` |

---

## GitHub Secrets

| Secret | Purpose |
|--------|---------|
| `EC2_SSH_KEY` | Private key for SSH into EC2 (used by GitHub Action for auto-deploy) |

---

## Known Issues / Notes

- The portal title still says "Local Inventory — Canada" — this is intentional
- Public IP `35.162.133.146` is an Elastic IP (permanent, free while instance is running)
- If the EC2 instance is **stopped** (not terminated), the Elastic IP will start incurring ~$0.005/hr charge — keep instance running
- The `inventory.db` on EC2 is the **only live database** — the Mac/Google Drive copies are stale
- Labels (Avery 6498) were previously generated for S4 inventory — script is not currently saved in the repo
- Google Drive copies of `inventory_portal.py` and `inventory.db` are outdated and can be archived
