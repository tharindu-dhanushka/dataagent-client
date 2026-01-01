# VE Data Agent - Teams App

App de Teams para consultar el Data Agent de Fabric con lenguaje natural.

## Requisitos

- Node.js 18+
- Azure AD App Registration
- Acceso al Data Agent de Fabric

## Setup

### 1. Crear App Registration en Azure AD

1. Ir a [Azure Portal](https://portal.azure.com) > Azure Active Directory > App registrations
2. New registration:
   - Name: `VE Data Agent`
   - Supported account types: `Accounts in this organizational directory only`
   - Redirect URI: `Single-page application (SPA)` > `http://localhost:3000`
3. Copiar el **Application (client) ID**

### 2. Configurar permisos API

En la App Registration:
1. API permissions > Add a permission
2. APIs my organization uses > buscar `Power BI Service`
3. Delegated permissions:
   - `Dataset.Read.All`
   - `Workspace.Read.All`
4. Grant admin consent

### 3. Configurar la app

Editar `src/authConfig.js`:

```js
clientId: "TU_CLIENT_ID_AQUI",
```

### 4. Instalar y ejecutar

```bash
cd "Copilot/Teams App"
npm install
npm run dev
```

Abrir http://localhost:3000

## Deploy a Teams

### Opción A: Teams Toolkit CLI (Recomendado)

1. **Instalar Teams Toolkit CLI**:
```bash
npm install -g @microsoft/teamsapp-cli
```

2. **Login a Azure y M365**:
```bash
teamsapp auth login azure
teamsapp auth login m365
```

3. **Provisionar recursos**:
```bash
teamsapp provision --env dev
```

4. **Deployar**:
```bash
teamsapp deploy --env dev
```

5. **Publicar a Teams**:
```bash
teamsapp publish --env dev
```

### Opción B: Manual

#### 1. Hosting

Subir el build a Azure Static Web Apps, Vercel, o similar:

```bash
npm run build
```

#### 2. Actualizar manifest

Editar `appPackage/manifest.json`:
- `{{APP_ID}}`: Un GUID único para la app
- `{{APP_DOMAIN}}`: Tu dominio (ej: `ve-data-agent.azurestaticapps.net`)
- `{{AAD_APP_ID}}`: El Client ID de Azure AD (`cb26c94e-580b-4828-8c9d-20a1d395d2b7`)

#### 3. Configurar Azure AD para Teams SSO

```bash
# Ya configurado:
az ad app update --id cb26c94e-580b-4828-8c9d-20a1d395d2b7 \
  --identifier-uris "api://cb26c94e-580b-4828-8c9d-20a1d395d2b7"
```

#### 4. Crear iconos

Crear en `appPackage/`:
- `color.png`: 192x192 px
- `outline.png`: 32x32 px (transparente)

#### 5. Empaquetar

```bash
cd appPackage
zip -r ../ve-data-agent.zip manifest.json color.png outline.png
```

#### 6. Subir a Teams

**Como usuario:**
1. Teams > Apps > Manage your apps
2. Upload an app > Upload a custom app
3. Seleccionar `ve-data-agent.zip`

**Como admin (para toda la organización):**
1. Ir a [Teams Admin Center](https://admin.teams.microsoft.com)
2. Teams apps > Manage apps
3. Upload new app > Seleccionar `ve-data-agent.zip`
4. Publish

## Endpoint del Data Agent

```
https://api.fabric.microsoft.com/v1/workspaces/69de573b-8d13-4e01-8492-11d91eb5ab15/dataagents/7a4af8af-66c4-4282-b061-21f6f965a172/aiassistant/openai
```

## Estructura

```
Teams App/
├── src/
│   ├── main.jsx        # Entry point
│   ├── App.jsx         # Componente principal (chat)
│   └── authConfig.js   # Config MSAL + endpoint
├── appPackage/
│   └── manifest.json   # Manifest de Teams
├── index.html
├── package.json
└── vite.config.js
```
