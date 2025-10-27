# List GitHub Enterprise and GitHub Copilot users from GitHub APIs

This is a simple project using just HTML, CSS, and pure JavaScript to list users from a GitHub organization, identifying who has a GitHub Enterprise license, who has a GitHub Copilot license, and the respective Cost Center for each user.

This project needs improvement and revision. GitHub provides an API to list Copilot users, but not a direct API to list all Enterprise users. I've implemented several workarounds to get this done. It's not 100% accurate, but it's very close, and ultimately, it's better than nothing. ;)

## Overview

This tool provides a comprehensive view of:
- All GitHub Enterprise users in your organization
- GitHub Copilot license assignments
- Cost Center assignments
- Active vs inactive users
- Detailed user activity and license history

## Features

- ✅ **No Installation Required** - Single HTML file, runs entirely in the browser
- 🔐 **Secure** - Your credentials stay in your browser (localStorage)
- 📊 **Comprehensive Data** - Combines data from multiple GitHub APIs
- 🔄 **Pagination Support** - Handles large organizations with 100+ users
- 📋 **Detailed Reporting** - Shows license status, activity dates, and cost centers
- 🐛 **Debug Logging** - Console logs help identify inactive users and data sources
- 💾 **Local Storage** - Saves your enterprise name and token for convenience

## Prerequisites

1. **GitHub Enterprise Account** with admin access
2. **Personal Access Token** with the following scopes:
   - `admin:enterprise` - Required for Cost Centers API
   - `read:org` - Required for organization member listings
   - `read:user` - Required for user details
   - If using SAML SSO, the token must be authorized for SSO

## How to Use

### Step 1: Get Your GitHub Token

1. Go to https://github.com/settings/tokens
2. Click "Generate new token (classic)"
3. Give it a descriptive name (e.g., "Cost Center Report")
4. Select scopes:
   - `admin:enterprise`
   - `read:org`
   - `read:user`
5. Click "Generate token"
6. **Copy the token immediately** (you won't see it again)
7. If your organization uses SAML SSO, click "Configure SSO" and authorize the token

### Step 2: Open the HTML File

1. Download `github-list-users-en.html`
2. Open it in a modern web browser (Chrome, Firefox, Edge, Safari)

### Step 3: Configure

1. Enter your **Enterprise Name** (e.g., `your-company-name`)
2. Enter your **GitHub Token** (starts with `ghp_`)
3. Credentials are automatically saved to localStorage

### Step 4: Generate Report

1. Click the **"📋 List Users"** button
2. Wait for the data to load (may take a few minutes for large organizations)
3. Review the results in the table

## Understanding the Report

### Summary Statistics

- **Cost Centers** - Total number of cost centers configured
- **Total Users** - All unique users found
- **Enterprise + Copilot** - Users with both licenses
- **Enterprise Only** - Users with only Enterprise license
- **Inactive/Deleted** - Users in Cost Centers but not in active organizations
- **Organizations** - Number of organizations processed

### Table Columns

| Column | Description |
|--------|-------------|
| **Name** | User's display name from GitHub profile |
| **Email** | Public email from GitHub profile (if available) |
| **Username** | GitHub username/login |
| **Cost Center** | Assigned cost center name |
| **Enterprise** | ✓ = Active member, ✗ = Not a member |
| **Copilot** | ✓ = Has Copilot license, ✗ = No Copilot |
| **Created** | When Copilot license was created |
| **Cancellation** | Pending cancellation date (if any) |
| **Last Activity** | Last time Copilot was used |
| **Status** | ACTIVE or INACTIVE |

### Status Indicators

- **ACTIVE** (green) - User is an active member of an organization
- **INACTIVE** (red) - User is listed in Cost Centers but not found in any organization (possibly deleted)

## Debug Information

The tool provides detailed console logging:

### Phase 1: Cost Centers
Logs all users found in Cost Centers

### Phase 2: Organizations
- Lists all active organization members
- Lists all Copilot license holders
- Cross-references data sources

### Phase 3: User Details
- Fetches individual user profiles
- Identifies deleted/404 users
- Flags inconsistencies (e.g., Copilot without Enterprise)

### Open Browser Console
- **Chrome/Edge**: Press F12
- **Firefox**: Press F12
- **Safari**: Enable Develop menu, then press Cmd+Option+I

## API Endpoints Used

The tool calls the following GitHub APIs:

### 1. Cost Centers API

**Endpoint:**
```
GET /enterprises/{enterprise}/settings/billing/cost-centers
```

**Purpose:** Retrieves all cost centers and their assigned resources (users and organizations)

**Curl Example:**
```bash
curl -L \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer ghp_YOUR_TOKEN_HERE" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/enterprises/your-company-name/settings/billing/cost-centers
```

**Example Response:**
```json
{
  "costCenters": [
    {
      "id": "cc_123abc",
      "name": "Engineering Team",
      "resources": [
        {
          "type": "User",
          "name": "john-doe"
        },
        {
          "type": "Org",
          "name": "acme-engineering"
        }
      ]
    }
  ]
}
```

---

### 2. Organization Members API (Paginated)

**Endpoint:**
```
GET /orgs/{org}/members?per_page=100&page={page}
```

**Purpose:** Lists all active members of an organization (Enterprise license holders)

**Curl Example (Page 1):**
```bash
curl -L \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer ghp_YOUR_TOKEN_HERE" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  "https://api.github.com/orgs/acme-engineering/members?per_page=100&page=1"
```

**Curl Example (Page 2):**
```bash
curl -L \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer ghp_YOUR_TOKEN_HERE" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  "https://api.github.com/orgs/acme-engineering/members?per_page=100&page=2"
```

**Example Response:**
```json
[
  {
    "login": "john-doe",
    "id": 12345,
    "type": "User",
    "site_admin": false
  },
  {
    "login": "jane-smith",
    "id": 67890,
    "type": "User",
    "site_admin": false
  }
]
```

**Response Headers (for pagination):**
```
Link: <https://api.github.com/orgs/acme-engineering/members?per_page=100&page=2>; rel="next",
      <https://api.github.com/orgs/acme-engineering/members?per_page=100&page=5>; rel="last"
```

---

### 3. Copilot Billing API (Paginated)

**Endpoint:**
```
GET /orgs/{org}/copilot/billing/seats?per_page=100&page={page}
```

**Purpose:** Lists all Copilot license assignments with activity data

**Curl Example (Page 1):**
```bash
curl -L \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer ghp_YOUR_TOKEN_HERE" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  "https://api.github.com/orgs/acme-engineering/copilot/billing/seats?per_page=100&page=1"
```

**Curl Example (Page 2):**
```bash
curl -L \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer ghp_YOUR_TOKEN_HERE" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  "https://api.github.com/orgs/acme-engineering/copilot/billing/seats?per_page=100&page=2"
```

**Example Response:**
```json
{
  "total_seats": 150,
  "seats": [
    {
      "created_at": "2024-01-15T10:30:00Z",
      "updated_at": "2024-01-20T14:22:00Z",
      "pending_cancellation_date": null,
      "last_activity_at": "2024-01-20T09:15:00Z",
      "last_activity_editor": "vscode/1.85.0",
      "assignee": {
        "login": "john-doe",
        "id": 12345,
        "type": "User"
      }
    },
    {
      "created_at": "2024-01-10T08:00:00Z",
      "updated_at": "2024-01-18T16:45:00Z",
      "pending_cancellation_date": "2024-02-01",
      "last_activity_at": "2024-01-18T11:30:00Z",
      "last_activity_editor": "jetbrains/2023.3",
      "assignee": {
        "login": "jane-smith",
        "id": 67890,
        "type": "User"
      }
    }
  ]
}
```

**Response Headers (for pagination):**
```
Link: <https://api.github.com/orgs/acme-engineering/copilot/billing/seats?per_page=100&page=2>; rel="next",
      <https://api.github.com/orgs/acme-engineering/copilot/billing/seats?per_page=100&page=3>; rel="last"
```

---

### 4. Users API

**Endpoint:**
```
GET /users/{username}
```

**Purpose:** Fetches public profile information for individual users

**Curl Example:**
```bash
curl -L \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer ghp_YOUR_TOKEN_HERE" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/users/john-doe
```

**Example Response:**
```json
{
  "login": "john-doe",
  "id": 12345,
  "name": "John Doe",
  "email": "john.doe@example.com",
  "company": "Acme Corp",
  "location": "San Francisco, CA",
  "bio": "Software Engineer",
  "type": "User",
  "site_admin": false,
  "created_at": "2020-05-10T08:30:00Z",
  "updated_at": "2024-01-15T12:45:00Z"
}
```

**Note:** Email is only included if the user has made it public in their profile settings.

---

### Testing Individual Endpoints

You can test each endpoint independently before running the full report:

**1. Test Cost Centers Access:**
```bash
export GITHUB_TOKEN="ghp_YOUR_TOKEN_HERE"
export ENTERPRISE_NAME="your-company-name"

curl -L \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  "https://api.github.com/enterprises/$ENTERPRISE_NAME/settings/billing/cost-centers"
```

**2. Test Organization Access:**
```bash
export ORG_NAME="your-org-name"

curl -L \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  "https://api.github.com/orgs/$ORG_NAME/members?per_page=10"
```

**3. Test Copilot Access:**
```bash
curl -L \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  "https://api.github.com/orgs/$ORG_NAME/copilot/billing/seats?per_page=10"
```

**4. Test User Profile Access:**
```bash
export USERNAME="john-doe"

curl -L \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  "https://api.github.com/users/$USERNAME"
```

---

## Rate Limiting

- The tool includes automatic delays between API calls (200-300ms)
- Processes up to 100 items per page
- For large organizations, the report may take several minutes

## Troubleshooting

### Error: "Please enter your GitHub Enterprise name"
- Make sure you've filled in the Enterprise Name field

### Error: "Please enter your GitHub Personal Access Token"
- Make sure you've filled in the GitHub Token field

### Error: "HTTP 401"
- Your token may be invalid or expired
- Generate a new token and try again

### Error: "HTTP 403"
- Token doesn't have required permissions
- Ensure token has `admin:enterprise` scope
- If using SAML SSO, authorize the token

### Error: "HTTP 404"
- Enterprise name may be incorrect
- Cost Centers may not be configured
- You may not have access to the enterprise

### Users showing as INACTIVE
- These users exist in Cost Centers but are not active members
- They may have been removed from the organization
- Consider cleaning up Cost Centers to remove them

### Missing Copilot data
- Organization may not have Copilot enabled
- Check console logs for specific error messages

## Privacy & Security

- **All processing happens in your browser** - No data is sent to external servers
- **Credentials stored locally** - Uses browser's localStorage
- **HTTPS only** - All GitHub API calls use HTTPS
- **No tracking** - No analytics or external scripts

## Data Persistence

The tool stores:
- Enterprise name
- GitHub token (encrypted by browser)

Stored in: `localStorage`

To clear stored data:
1. Open browser console (F12)
2. Run:
   ```javascript
   localStorage.removeItem('github_enterprise');
   localStorage.removeItem('github_token');
   ```

## Supported Browsers

- ✅ Chrome/Chromium (recommended)
- ✅ Firefox
- ✅ Edge
- ✅ Safari
- ❌ Internet Explorer (not supported)

## Limitations

- Cannot fetch private email addresses (only public emails)
- Requires admin access to enterprise and organizations
- Rate limited by GitHub API (5000 requests/hour)
- Browser must support ES6+ JavaScript features

## Example Use Cases

1. **License Audit** - Identify who has Copilot licenses
2. **Cost Analysis** - See license distribution across cost centers
3. **Cleanup** - Find inactive users to remove from Cost Centers
4. **Compliance** - Verify all users are properly assigned
5. **Activity Monitoring** - Track Copilot usage and last activity dates

## Support

For issues with:
- **This tool** - Check browser console for errors
- **GitHub API** - Contact GitHub Support
- **Permissions** - Contact your GitHub Enterprise admin

## Version

Current version: 1.0.0
Last updated: 2024

## License

This is a utility tool. Use at your own discretion.
