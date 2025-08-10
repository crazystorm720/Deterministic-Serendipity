# Solo Travel Manicurist YAML System

A **zero-frills**, portable inventory and client tracker for manicurists on the go.  
Works offline, syncs anywhere, and requires no special apps.

## Files

| File            | Purpose                                                                 |
|-----------------|-------------------------------------------------------------------------|
| `inventory.yml` | Track **critical supplies** (stock levels, reorder points).            |
| `clients.yml`   | Store **essential client info** (preferences, allergies, contact).     |
| `kit.yml`       | Pre-appointment **gear checklist** (never forget portable tools).      |
| `services.yml`  | Quick-reference **pricing/duration** guide for consistency.            |

## How to Use

1. **Edit files** in any text app (VS Code, Notes, etc.).
2. **Sync** via Dropbox/Google Drive for phone/laptop access.
3. **Update on-the-fly** between clients.

### Key Features
- ✅ **Low-stock alerts**: Run `python check_stock.py` to flag items to reorder.
- ✅ **Client pre-check**: Review `clients.yml` before appointments.
- ✅ **Kit validation**: Use `kit.yml` as a packing checklist.

## Example Workflow

```bash
# 1. Check inventory before a booking
cat inventory.yml | grep "urgent: true"

# 2. Verify client preferences
cat clients.yml | grep "Maria Garcia"

# 3. Confirm gear is packed
cat kit.yml | grep "checked: false"
