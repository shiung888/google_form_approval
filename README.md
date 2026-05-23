# Google Form Approval Workflow

A Google Apps Script solution that adds a **multi-step approval workflow** to any Google Form. When a form is submitted, it automatically routes the response through a configurable chain of approvers — each approver receives an email with approve/reject actions, and the original submitter is notified of the final decision.

---

## Features

- **Multi-step approval chain** — define as many approvers as needed per flow
- **Email notifications** — approvers receive action emails; submitters receive status updates
- **Real-time status tracking** — a web app UI shows live approval progress
- **Unique ID generation** — each submission gets a unique UID (e.g. `UID-00001`)
- **Comments support** — approvers can leave comments when approving or rejecting
- **Custom flows** — route different submissions to different approver chains based on a form field (e.g. `Department`)
- **Auto spreadsheet setup** — automatically creates and links a response spreadsheet if one doesn't exist

---

## How It Works

1. A user submits a Google Form
2. The script assigns a unique UID, sets status to `Pending`, and sends an approval email to the first approver
3. Each approver can **Approve** or **Reject** via the email link
4. On approval, the request moves to the next approver in the chain
5. Once all approvers approve (or any one rejects), the submitter is notified with the final status
6. A separate progress page lets anyone with the link track the current approval stage

---

## File Structure

| File | Description |
|---|---|
| `app.gs` | Main Apps Script — core logic for form submission, approval, rejection, email, and web app routing |
| `index.html` | Approval action page — shown to approvers when they click the email link |
| `approval_progress.html` | Progress tracking page — shows the full approval chain status |
| `approval_email.html` | HTML email template sent to approvers |
| `notification_email.html` | HTML email template sent to the form submitter |
| `css.html` | Shared CSS styles included across HTML templates |
| `js.html` | Shared JavaScript included across HTML templates |
| `404.html` | Fallback page for invalid or missing URL parameters |

---

## Setup

### Prerequisites

- A Google account
- A Google Form with **email collection enabled**
- A Google Spreadsheet to store approver configuration

### Step 1 — Configure Approvers

Create a Google Spreadsheet with your approver data. Each row represents one approver in the chain:

| Email | Name | Title |
|---|---|---|
| approver1@example.com | Alice | Manager |
| approver2@example.com | Bob | Director |

Update `app.gs` with your spreadsheet ID:

```js
const SHEETID = 'YOUR_SPREADSHEET_ID_HERE';
```

### Step 2 — Set Up the Script

1. Open your Google Form
2. Go to **⋮ Menu → Script editor**
3. Copy all files from this repository into the Apps Script editor (one file per tab)
4. Save the project

### Step 3 — Deploy as Web App

1. In the Apps Script editor, click **Deploy → New deployment**
2. Select type: **Web app**
3. Set **Execute as**: Me
4. Set **Who has access**: Anyone
5. Click **Deploy** and copy the Web App URL

### Step 4 — Update the Web App URL

Paste your Web App URL into `app.gs`:

```js
this.url = "YOUR_WEB_APP_URL_HERE";
```

### Step 5 — Install the Form Trigger

1. Open your Google Form
2. In the top menu, click **Approval → Install trigger**

This sets up the `onFormSubmit` trigger that fires the approval workflow automatically.

---

## Configuration

### Approval Flows

You can define multiple approval flows in `app.gs` and route submissions based on a form field:

```js
const FLOWS = {
  defaultFlow: [
    { email: "manager@example.com", name: "Alice", title: "Manager" },
    { email: "director@example.com", name: "Bob", title: "Director" },
  ],
  Engineering: [
    { email: "eng-lead@example.com", name: "Carol", title: "Eng Lead" },
  ],
};
```

Set `this.flowHeader` in `app.gs` to the form question that determines which flow to use:

```js
this.flowHeader = "Department"; // must match a form question title
```

If no matching flow is found, `defaultFlow` is used.

### Key Configuration Options in `app.gs`

| Property | Default | Description |
|---|---|---|
| `this.flowHeader` | `"Department"` | Form question used to select the approval flow |
| `this.emailHeader` | `"Email Address"` | Form question for the submitter's email |
| `this.sheetname` | `"Form Responses 1"` | Name of the responses sheet tab |
| `this.uidPrefix` | `"UID-"` | Prefix for generated unique IDs |
| `this.uidLength` | `5` | Number of digits in the UID |

---

## Status Values

| Status | Meaning |
|---|---|
| `Pending` | Awaiting action from the current approver |
| `Waiting` | Queued for a future approver (not yet their turn) |
| `Approved` | Approved by all approvers |
| `Rejected` | Rejected by an approver |

---

## Resetting the UID Counter

If you need to reset the UID sequence (e.g. during testing), open your Google Form and click **Approval → Reset UID**.

---

## License

MIT
