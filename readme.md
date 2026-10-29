# Trimble Connect API Test Workbench

A lightweight HTML/JavaScript diagnostic utility for testing the Trimble Connect Workspace API and manually sending requests to Trimble Connect cloud APIs.

## Package contents

- `index.html` — responsive test interface and JavaScript implementation.
- `manifest.json` — extension manifest template.
- `image.svg` — app icon.
- `readme.md` — setup, usage, limitations and security notes.

## What it can test

### Workspace API
- Connect to the host using `TrimbleConnectWorkspace.connect(window.parent, callback)`.
- Get the current project using `project.getProject()` (preferred) or `project.getCurrentProject()` (legacy method).
- Read user settings with `user.getUserSettings()`.
- Read project members with `project.getMembers()`.
- Request permission to use the host-managed access token and listen for the `extension.accessToken` event.

### Cloud REST API request tester
The manual request tester supports the service families shown in the reference material:
- Core API
- Regions API
- Core Account API
- Model API
- Model Feature API
- Organizer API
- Property Set API
- Topics API

It lets you enter the exact official base URL, documented endpoint path, HTTP method, Bearer token and JSON body. It does not invent endpoint paths or payload schemas for you.

### Trimble Identity
The Authentication panel contains setup guidance and official documentation links. It does not implement an OAuth login flow or token exchange in the browser. This is intentional: production OAuth and client-secret handling should be implemented for the registered app type, and secrets must remain server-side.

## Install in Trimble Connect

1. Host `index.html`, `image.svg`, and `manifest.json` at stable **HTTPS** URLs accessible to the intended Trimble Connect users.
2. Edit `manifest.json`. Replace `https://YOUR_HOSTED_DOMAIN` in `icon` and `url` with the actual public HTTPS URLs. Keep the manifest and asset paths consistent.
3. Register an Extension App in the Trimble Developer Console and follow the current Trimble Connect extension setup guide.
4. In Trimble Connect for Browser, open the relevant project and go to **Project Settings → Apps & Capabilities → Add Custom** (labels can vary by product version).
5. Provide the hosted manifest URL.
6. Open the extension and choose **Connect workspace**.
7. Test the project and user settings calls before requesting access-token permission.
8. For REST tests, choose an API family, enter the service base URL and exact endpoint path from that API's current official OpenAPI definition, and provide a valid access token if required.

A manifest hosted on your computer's `file://` path is not a deployable extension manifest. For a quick local UI review, open `index.html` directly, but Workspace API calls will not work outside the embedded Trimble Connect context.

## API version and endpoint notes

Trimble's current developer documentation distinguishes newer Developer Console integrations from legacy access credentials. Core API `v2` and legacy Core API `2.0` are different versions. Regional endpoints and permissions also matter.

The Regions preset uses a documented example URL:
- Base URL: `https://us-east.api.trimble.com/connect/v2`
- Path: `/regions`
- Method: `GET`

This is only a sample for region discovery; it does not mean the target project is hosted in the US. Use the region and endpoint specified by the current official documentation for the target project.

The Core Account preset is only a template. Replace `{accountId}` with an actual account identifier and confirm the endpoint and permission requirements against the current API reference.

For Model, Model Feature, Organizer, Property Set and Topics, use their official OpenAPI definitions to determine the correct service URL, path, method, schema and required permissions. The REST tester is a generic HTTP client; it does not claim every endpoint can be called successfully from the browser.

## Browser CORS and backend proxy

Some cloud APIs may not allow direct browser requests due to CORS policy. If the browser reports a network/CORS error:
- verify the base URL, path, region, token and permissions;
- check whether the API supports browser-origin requests;
- if not, implement a secured backend proxy that validates the signed-in user, keeps credentials server-side, restricts allowed hosts and paths, and enforces authorization.

Do not work around CORS by disabling browser security.

## Security

- Do not put a client secret in `index.html`, a public repository or any browser-exposed configuration.
- The access token entered in the REST tester is held in page memory only; it is not written to local or session storage. It will be cleared when the page reloads.
- Treat tokens as credentials. Do not share screenshots or logs that expose them.
- Begin with `GET` requests on a test project. `POST`, `PUT`, `PATCH` and `DELETE` can modify real data.
- Use least-privilege scopes and permissions.
- Use HTTPS.
- Do not treat a project ID supplied by the browser as proof of authorization. Your production backend must enforce access checks.

## Troubleshooting

**Workspace API library unavailable**
- Open the page as a Trimble Connect extension, not just as a standalone browser page.
- Confirm that the Workspace API script is reachable from the hosted page.
- Confirm the manifest URL and extension setup.

**Workspace connects but a method fails**
- Confirm that the method exists in the version of the Workspace API loaded by the host.
- Inspect the displayed error and official API reference. `getCurrentProject()` is marked deprecated in favor of `getProject()` in the current reference.

**REST request fails**
- `401`: token missing, invalid or expired.
- `403`: user lacks permission or required scope.
- `404`: service URL or endpoint path may be wrong, or the resource may not exist.
- `429`: rate limit; respect server guidance before retrying.
- Browser network/CORS error: request may be blocked by browser cross-origin policy; consider a secured backend proxy.

## Official documentation

- Main documentation: https://developer.trimble.com/docs/connect
- Extend Trimble Connect: https://developer.trimble.com/docs/connect/guides/extend/
- Workspace API reference: https://components.connect.trimble.com/trimble-connect-workspace-api/index.html
- Core API: https://developer.trimble.com/docs/connect/tools/api/core
- Core Account API: https://developer.trimble.com/docs/connect/tools/api/core-account/
- Model API: https://developer.trimble.com/docs/connect/tools/api/model/
- Model Feature API: https://developer.trimble.com/docs/connect/tools/api/model-feature/
- Organizer API: https://developer.trimble.com/docs/connect/tools/api/organizer
- Property Set API: https://developer.trimble.com/docs/connect/tools/api/property-set/
- Topics API: https://developer.trimble.com/docs/connect/tools/api/topics/
- API integration guide: https://developer.trimble.com/docs/connect/guides/integrate/latest
- Authentication guide: https://developer.trimble.com/docs/authentication/guides/initial-api-transaction/

## Scope

This is a developer test harness, not an official Trimble application. Successful testing depends on the extension being correctly registered, user consent, app configuration, valid access tokens, the correct regional service, API version, endpoint schema and the user's permissions.
