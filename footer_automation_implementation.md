# 📋 Email Signature Generator & Automation Pipeline
**Project Overview:** Implementation design for a self-serve internal portal allowing employees to generate standardized company email footers and seamlessly install them into their email clients.

## 🏗️ Phase 1: The Generation Pipeline (Self-Serve Portal)
*A simple, lightweight internal web application to handle data input and live rendering.*

### Proposed Stack:
*   **Frontend**: React, Vue.js, or HTML/JS.
*   **Styling**: Tailwind CSS (for the portal UI) + Inline HTML (for the actual signature).
*   **Hosting**: Internal server, Vercel, Netlify, or Azure Static Web Apps.

### User Flow:
1.  **Authentication (Optional)**: User logs in via company SSO to pre-fill known data (Name, Email, Job Title).
2.  **Input Form**: User reviews and modifies data fields (Phone Number, Preferred Pronouns, Department).
3.  **Live Preview**: As data is entered, a DOM pane dynamically updates the HTML signature template to reflect changes in real-time.
4.  **Action / Delivery**: The user clicks the primary **“Copy to Clipboard”** button.
    *   *Technical Detail:* The site uses the `navigator.clipboard.write` API to copy the rendered DOM node as **Rich Text**, preserving the nested tables, inline CSS, and hosted image URLs exactly as required by email clients.

---

## ⚙️ Phase 2: Client Installation & Automation
*Evaluating the feasibility of zero-touch automated signature installation based on the email client in use.*

### 🟢 Fully Supported (Cloud-Based Automation)
For organizations using major cloud email providers, APIs can push signatures directly to user accounts without manual copy/pasting.

#### Option A: Google Workspace (Gmail)
*   **Method 1: User-Authorized App (OAuth)**
    *   Portal asks the user to "Sign in with Google."
    *   App requests `https://www.googleapis.com/auth/gmail.settings.basic` scope.
    *   App executes a `PATCH` request to the [Gmail SendAs API](https://developers.google.com/gmail/api/reference/rest/v1/users.settings.sendAs/patch) to inject the HTML signature string.
*   **Method 2: Centralized IT Automation (Zero User Interaction)**
    *   Service Account authorized with Domain-Wide Delegation.
    *   A backend CRON script iterates through the Google Workspace Directory, generates the HTML for every employee, and silently pushes it to their Gmail settings.

#### Option B: Microsoft 365 / Exchange (Outlook)
*   **Method 1: User-Authorized App (OAuth)**
    *   User logs into the portal using Microsoft SSO.
    *   App uses the [Microsoft Graph MailboxSettings API](https://learn.microsoft.com/en-us/graph/api/user-update-mailboxsettings?view=graph-rest-1.0&tabs=http) to update the cloud signature. This automatically syncs down to their Outlook Desktop and Web apps.
*   **Method 2: Centralized IT Automation**
    *   Admin grants tenant-wide admin consent to a server application.
    *   Backend script runs via Microsoft Graph API to modify `mailboxSettings` universally.
*   **Alternate Approach (Mail Flow Rules):** IT configures Exchange Mail Flow rules to forcefully append the HTML code to the bottom of all outbound messages. *(Drawback: Appends to the bottom of long email threads rather than immediately under the newly typed reply).*

### 🔴 Not Recommended (Local-Client Automation)
For traditional, non-cloud synced desktop clients, automated web-based installation is largely impossible.

#### Thunderbird & Apple Mail
These applications store signatures as local, physical files on the user's hard drive (`prefs.js` for Thunderbird, `.mailsignature` for Apple Mail). 
*   **Why it fails:** Web browsers (and therefore your Generator Portal) cannot securely execute scripts to read/write local filesystem configuration files.
*   **Workaround:** Requires heavy IT intervention (e.g., Windows Group Policy scripts, Jamf MDM profiles) to physically push the generated HTML files onto user machines and modify local app registries/preferences.

---

## 🎯 Implementation Recommendation
To achieve the best balance of engineering effort and user experience:

- [ ] **Step 1:** Build the core **React/HTML Generator Portal** with the live preview.
- [ ] **Step 2:** Implement a prominent **"Copy Signature"** button as the universal fallback that works across 100% of email clients (including local clients like Thunderbird). Provide simple visual instructions for pasting.
- [ ] **Step 3 (If using Google/MS365):** Implement an optional **"Deploy to Inbox"** button using the Google or Microsoft Graph APIs to completely eliminate the copy/paste friction for cloud users.
