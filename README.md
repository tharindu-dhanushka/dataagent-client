# Data Agent Client - Teams App

![React](https://img.shields.io/badge/React-18.2-61DAFB?logo=react&logoColor=white)
![Vite](https://img.shields.io/badge/Vite-5.0-646CFF?logo=vite&logoColor=white)
![Microsoft Teams](https://img.shields.io/badge/Microsoft_Teams-6264A7?logo=microsoftteams&logoColor=white)
![Fluent UI](https://img.shields.io/badge/Fluent_UI-0078D4?logo=microsoft&logoColor=white)
![Azure AD](https://img.shields.io/badge/Azure_AD-0089D6?logo=microsoftazure&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)

A Microsoft Teams application that enables natural language queries to Fabric Data Agents, featuring real-time streaming responses, DAX query visualization, and multi-agent support.

## Features

- **Natural Language Queries**: Ask questions in plain language and get AI-powered answers from your semantic models
- **Multi-Agent Support**: Connect to multiple Fabric Data Agents and switch between them seamlessly
- **Real-Time Streaming**: See responses as they're generated with live progress indicators
- **Query Transparency**: View the generated DAX code and query results for each analysis step
- **Teams Integration**: Native Teams app experience with SSO authentication
- **Fluent UI Design**: Modern, responsive interface following Microsoft design guidelines

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Teams / Browser                          │
├─────────────────────────────────────────────────────────────────┤
│  React App (Vite)                                               │
│  ├── MSAL Authentication (Azure AD)                             │
│  ├── Fluent UI Components                                       │
│  └── Streaming SSE Handler                                      │
├─────────────────────────────────────────────────────────────────┤
│  Fabric REST API                                                │
│  ├── /workspaces - List available workspaces                    │
│  ├── /dataAgents - Discover Data Agents                         │
│  └── /aiassistant/openai - Query endpoint (streaming)           │
└─────────────────────────────────────────────────────────────────┘
```

## Requirements

- Node.js 18+
- Azure AD App Registration with appropriate permissions
- Access to Microsoft Fabric with Data Agent(s) configured

## Quick Start

### 1. Create App Registration in Azure AD

1. Go to [Azure Portal](https://portal.azure.com) > **Azure Active Directory** > **App registrations**
2. Click **New registration**:
   - **Name**: `Data Agent Client`
   - **Supported account types**: `Accounts in this organizational directory only`
   - **Redirect URI**: Select `Single-page application (SPA)` > `http://localhost:3000`
3. Copy the **Application (client) ID**

### 2. Configure API Permissions

In your App Registration:

1. Go to **API permissions** > **Add a permission**
2. Select **APIs my organization uses** > Search for `Power BI Service`
3. Add **Delegated permissions**:
   - `Dataset.Read.All`
   - `Workspace.Read.All`
4. Click **Grant admin consent** (requires admin privileges)

### 3. Configure the Application

Edit `src/authConfig.js` with your Client ID:

```js
clientId: "YOUR_CLIENT_ID_HERE",
```

### 4. Install and Run

```bash
npm install
npm run dev
```

Open http://localhost:3000

## Deploy to Microsoft Teams

### Option A: Teams Toolkit CLI (Recommended)

```bash
# Install Teams Toolkit CLI
npm install -g @microsoft/teamsapp-cli

# Login to Azure and Microsoft 365
teamsapp auth login azure
teamsapp auth login m365

# Provision cloud resources
teamsapp provision --env dev

# Deploy the application
teamsapp deploy --env dev

# Publish to Teams
teamsapp publish --env dev
```

### Option B: Manual Deployment

#### 1. Build and Host

Deploy the production build to Azure Static Web Apps, Vercel, or similar:

```bash
npm run build
# Upload the 'dist' folder to your hosting service
```

#### 2. Update the Manifest

Edit `appPackage/manifest.json` and replace placeholders:

| Placeholder | Description |
|-------------|-------------|
| `{{APP_ID}}` | A unique GUID for your Teams app |
| `{{APP_DOMAIN}}` | Your hosting domain (e.g., `your-app.azurestaticapps.net`) |
| `{{AAD_APP_ID}}` | Your Azure AD Application (client) ID |

#### 3. Configure Azure AD for Teams SSO

```bash
az ad app update --id YOUR_CLIENT_ID \
  --identifier-uris "api://YOUR_CLIENT_ID"
```

#### 4. Create App Icons

Create the following icons in `appPackage/`:
- `color.png`: 192x192 px (full color)
- `outline.png`: 32x32 px (transparent background, single color)

#### 5. Package the App

```bash
cd appPackage
zip -r ../dataagent-client.zip manifest.json color.png outline.png
```

#### 6. Upload to Teams

**For personal use:**
1. Open Teams > **Apps** > **Manage your apps**
2. Click **Upload an app** > **Upload a custom app**
3. Select `dataagent-client.zip`

**For organization-wide deployment:**
1. Go to [Teams Admin Center](https://admin.teams.microsoft.com)
2. Navigate to **Teams apps** > **Manage apps**
3. Click **Upload new app** > Select `dataagent-client.zip`
4. Approve and publish

## Project Structure

```
├── src/
│   ├── main.jsx          # Entry point with MSAL provider
│   ├── App.jsx           # Main chat component with streaming
│   └── authConfig.js     # MSAL configuration & endpoints
├── appPackage/
│   └── manifest.json     # Teams app manifest
├── index.html            # HTML entry point
├── package.json          # Dependencies and scripts
└── vite.config.js        # Vite build configuration
```

## Troubleshooting

### "AADSTS50011: Reply URL does not match"
Ensure your Azure AD app registration has the correct redirect URI:
- For local development: `http://localhost:3000`
- For production: Your deployed app URL

### "No Data Agents found"
- Verify you have access to at least one Fabric workspace with a Data Agent
- Check that the Data Agent is properly configured in Fabric
- Ensure your Azure AD app has the required permissions

### Token acquisition fails silently
- Clear browser cache and cookies
- Try using incognito/private browsing
- Check browser console for detailed error messages

### Streaming responses not working
- Verify network connectivity to `api.fabric.microsoft.com`
- Check browser DevTools Network tab for failed requests
- Ensure the Data Agent endpoint is accessible

## API Flow

1. **Authentication**: MSAL acquires token with Fabric scopes
2. **Discovery**: App fetches available workspaces and Data Agents
3. **Query Flow**:
   - Create Assistant instance
   - Create conversation Thread
   - Post user Message
   - Create Run with `stream: true`
   - Parse SSE events for real-time updates
   - Cleanup Thread on completion

## Technologies

- **React 18** - UI framework
- **Vite 5** - Build tool and dev server
- **MSAL React** - Azure AD authentication
- **Fluent UI React** - Microsoft design system components
- **Teams JS SDK** - Teams integration

## License

MIT
