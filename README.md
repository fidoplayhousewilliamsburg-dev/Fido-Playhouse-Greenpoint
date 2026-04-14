# Fido Playhouse — Greenpoint 🐶

Internal dashboard for Fido Playhouse Greenpoint. Displays dogs currently checked in, grooming schedule, and drop off routes. Built with vanilla HTML/JS + Google Sheets as the database.

---

## Files

| File | Description |
|------|-------------|
| `index.html` | Main dashboard app — runs entirely in the browser |
| `Code.gs` | Google Apps Script — serves as the API between the app and Google Sheets |

---

## How it works

```
Gingr (PMS) → Zapier Webhook → Google Sheets → Google Apps Script API → index.html
```

1. When a dog checks in on **Gingr**, a Zapier webhook fires
2. Zapier writes the dog's data into **Google Sheets** (Sheet1)
3. A second Zapier step calls the **Apps Script API** to convert the birthday timestamp and calculate age
4. The `index.html` dashboard fetches data from the API every 10 seconds and updates the display

---

## Google Sheets structure

### Sheet1 — Dogs (check-ins)
| Col | Field |
|-----|-------|
| A | Name |
| B | Photo URL |
| C | Breed |
| D | Type |
| E | Checkout Date |
| F | ID |
| G | Address |
| H | Apt |
| I | Owner |
| J | Gender |
| K | Birthday (converted from Unix timestamp) |
| L | Age (calculated automatically) |
| M | Fixed / Castrado |

### Sheet3 — Grooming
| Col | Field |
|-----|-------|
| A | Name |
| B | Breed |
| C | Service |
| D | Details |
| E | Time |
| F | Value |
| G | Done (✓) |
| H | Groomer Value (60%) |
| I | Date |
| J | Tip |

### Sheet4 — Extras (service add-ons per dog, synced across devices)

---

## Apps Script API

**Base URL:**
```
https://script.google.com/macros/s/AKfycbyzCoEicj4hUBPMpftpxoRVyxcvjH_ivM0TRAj6p6kck28wGY1DChEvhB6R_Mc8zqOSog/exec
```

| Action | Description |
|--------|-------------|
| _(no action)_ | Returns all active dogs from Sheet1 |
| `?action=grooming` | Returns grooming list from Sheet3 |
| `?action=updateGrooming&row=N&done=true` | Marks a grooming row as done |
| `?action=addGrooming` | Adds a new grooming entry |
| `?action=editGrooming&row=N` | Edits an existing grooming entry |
| `?action=setExtras` | Saves service extras for a dog |
| `?action=getExtras` | Returns all service extras |
| `?action=calcBirthday` | Converts birthday timestamps and calculates ages |

---

## Zapier setup

The Zap fires on every Gingr check-in and has 4 steps:

1. **Trigger** — Webhook from Gingr (check_in event)
2. **Filter** — Only proceed if animal species is dog
3. **Google Sheets** — Create/update row in Sheet1
4. **Webhooks by Zapier (GET)** — Call `?action=calcBirthday` to convert birthday

---

## Updating the app

**To update the dashboard UI** — edit `index.html` and push to GitHub. If hosted via GitHub Pages, changes go live automatically.

**To update the API logic** — edit `Code.gs` in Google Apps Script, then click **Deploy → Manage deployments → New version**.

> ⚠️ Always create a new deployment version after changing `Code.gs` — otherwise the live URL keeps running the old code.

---

## Apps Script triggers

| Function | Trigger |
|----------|---------|
| `calcBirthdayAndAge` | Called via Zapier after each check-in |
| `deleteCheckedOutDogs` | Time-driven — runs daily to clean up checked-out dogs |

---

## Notes

- The dashboard auto-refreshes every **10 seconds**
- Slides auto-advance every **7 seconds**
- On a dog's birthday, the slide shows a special animated frame 🎂
- Groomer cut is always **60%** of the service value, calculated automatically
- Tips are tracked separately and added to the groomer's total
