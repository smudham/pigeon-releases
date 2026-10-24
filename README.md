# Pigeon

[![Latest release](https://img.shields.io/github/v/release/smudham/pigeon-releases?label=release)](https://github.com/smudham/pigeon-releases/releases/latest)
[![Downloads](https://img.shields.io/github/downloads/smudham/pigeon-releases/total)](https://github.com/smudham/pigeon-releases/releases)
![Platform: macOS | Windows](https://img.shields.io/badge/platform-macOS%20%7C%20Windows-lightgrey)

Pigeon is a desktop app for building, testing and load-testing HTTP APIs. It works on its own, with no account, and your collections, environments and history stay in a database on your computer.

This repo holds the downloads only. No source code lives here.

- [Download and install](#download-and-install)
- [Quick start](#quick-start)
- [Features](#features)
- [Import from Postman and export](#import-from-postman-and-export)
- [Keyboard shortcuts](#keyboard-shortcuts)
- [Your data](#your-data)
- [Updates](#updates)

## Download and install

**Latest version:** https://github.com/smudham/pigeon-releases/releases/latest

- **macOS (Apple Silicon, macOS 12+):** download the `.dmg` and drag Pigeon to Applications. The app is not notarized, so the first launch is blocked once: on macOS 15 or later open **System Settings → Privacy & Security** and click **Open Anyway**; on macOS 12–14 right-click Pigeon → **Open**.
- **Windows 11 (x64):** run the `-setup.exe`. It installs for your user and needs no admin rights. If SmartScreen warns, choose **More info → Run anyway**.

## Quick start

1. In the sidebar, click **+** next to **Collections** to create a collection.
2. Click **Add request** on the collection, pick a method and enter a URL, for example `https://httpbin.org/get`.
3. Click **Send** (or press Cmd/Ctrl+Enter). The response shows below the request.
4. Open **Environments** and click **New environment** (**New** if you already have one). Type a name and click **Create**. Add a variable such as `base_url`, click **Save**, then click **Use for requests**. Now use it in a URL as `{{base_url}}/items`.
5. Open the **Tests** tab and add a check:

   ```js
   pm.test('status is 200', () => pm.response.to.have.status(200))
   ```

6. Click **Run** on the collection to run every request and its tests in order.

Requests in a collection save automatically. The top bar shows **Saving…** and **Saved**.

Moving from Postman? Import your collections and environments first: see [Import from Postman and export](#import-from-postman-and-export).

## Features

### Requests

- Methods: GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS, TRACE.
- **Params** and **Headers** tables with an on/off checkbox per row. The Params table stays in sync with the URL.
- **Body** types:
  - **none**
  - **form-data**, with text and file fields
  - **x-www-form-urlencoded**
  - **raw**: JSON, XML, Text, HTML or JavaScript. The matching `Content-Type` is set unless you set it yourself.
    A JSON body can have `//` and `/* */` comments, as in Postman. Pigeon sends the body without them.
  - **binary**: send a file
  - **GraphQL**: query and variables
- **Settings** tab per request:
  - **Request timeout** in ms: default 60 s, 0 means no timeout, at most 24 h.
  - **Follow redirects**: up to 10.
  - **Verify TLS certificates**.
- **Cancel** a request that is running with the **Cancel** button, Esc, or Cmd/Ctrl+Period.
- Response panel:
  - Status, time and size, and the number of redirects with the final URL.
  - **Pretty** and **Raw** views with syntax highlighting.
  - **Wrap lines**, **Collapse all** and **Expand all**.
  - **Copy**, and **Save** the body to a file.
  - Images show as a preview. Other binary responses can be saved as a file.
  - Bodies over 5 MB are cut to their first 5 MB.
- HTTPS requests trust the certificates installed on your computer (macOS Keychain, Windows certificate store), so internal certificate authorities work.

### Auth

On the **Auth** tab, choose:

- **Bearer token**
- **Basic**: username and password.
- **API key**: as a header or a query parameter.
- **OAuth 2.0 — client credentials**: Token URL, Scope, Client ID, Client secret.
  - **Fetch new token** gets a token. On **Send**, Pigeon also fetches one if there is none.
  - A pill shows how long the token stays valid.
  - Tokens are kept in memory only.

All auth fields accept `{{variables}}`.

### Collections

- Collections hold requests directly and in folders, one folder level deep.
- Right-click for more actions. Double-click a name to rename it.
  - Collection: **Add request**, **New folder**, **Run**, **Variables and scripts**, **Export…**, **Rename**, **Duplicate**, **Delete**.
  - Folder: **Add request**, **Rename**, **Delete folder** (its requests move to the collection root).
  - Request: **Rename**, **Duplicate**, **Move to**, **Delete**.
- Drag requests to reorder them or move them into a folder or another collection. Drag a folder to move it to another collection.
- **Variables and scripts** (collection right-click menu): collection variables, plus a pre-request script and a test script that run before each request's own scripts.

### Environments and variables

- Open **Environments** in the sidebar, or click the **Env:** chip next to the request tabs.
- Create, rename, duplicate, export and delete environments. **Use for requests** makes one active.
- Mark a value **SECRET** to hide it on screen.
- Write `{{name}}` in the URL, params, headers, text body fields and auth fields. File paths are not filled in. Undefined variables are highlighted in the URL.
- A value can use other variables, for example a `base_url` of `{{scheme}}://{{host}}`. Pigeon fills them in up to 10 levels deep.
- Hover over a `{{variable}}` in the URL to see where its value comes from: an environment, the collection, or built in. Change the value in the box and press Enter to save it.
- The background colour of each variable in the URL shows its state:
  - teal: it has a value.
  - orange: it is defined, but it has no value yet.
  - red: it is not defined.
- To define a red one, hover over it and type a value. Then choose **Add to** (an environment, a new environment or the collection) and press Enter. If an environment that is not active defines it, click **Use** with that environment's name to make it active.
- When a name is defined in several places, the later one wins: collection variables → active environment → the runner's data file row → values set with `pm.variables.set`.
- Built-in values: `{{$uuid}}`, `{{$timestamp}}` (milliseconds) and `{{$isoDate}}`.

### Scripts and tests

- **Pre-request** tab: runs before the request is sent. It can change the URL, query, headers and body, and set variables.
- **Tests** tab: runs after the response arrives. Results show as **x / y PASSED**, with each test, its time and the **CONSOLE** output.
- Scripts use the Postman-style `pm` API:
  - `pm.test(name, fn)`, and `pm.expect(...)` with chai-style checks such as `.to.equal`, `.to.include` and `.to.have.property`.
  - `pm.response`: `.code`, `.json()`, `.text()`, `.headers.get()`, `.responseTime`, and checks such as `pm.response.to.have.status(200)`, `.to.be.ok` and `.to.have.jsonBody('data.id')`.
  - `pm.environment`, `pm.collectionVariables` and `pm.variables`: `.get`, `.set`, `.unset`, `.has`. Values set with `pm.environment` and `pm.collectionVariables` are saved. Values set with `pm.variables` last for that send or run only.
  - `pm.request` (editable in pre-request scripts, read-only in tests), `pm.iterationData`, `pm.info`, and `console.log`.
- Scripts run in a sandbox with no network or file access: at most 3 s CPU and 64 MB memory each.
- Not available in scripts: `pm.sendRequest`, `pm.globals`, `pm.cookies`, `pm.vault` and `require()`.

### Collection runner

- Open **Runner** in the sidebar, or click **Run** on a collection.
- Requests run in order: each folder's requests, then the requests at the collection root.
- Options:
  - **Iterations**: 1–5000.
  - **Delay**: 0–60000 ms between requests.
  - **Data file…**: a CSV file with a header row, or a JSON array of objects. Each row runs one iteration, and its columns become variables.
- Results show passed and failed counts, and for each request its status, assertions, time, **TESTS** and **CONSOLE**. **Stop** ends a run early.

### Load tests

- Open **Performance** in the sidebar.
- Pick the current request or any saved request, then set:
  - **VUs** (virtual users): 1–500.
  - **Ramp (s)**: 0–300.
  - **Duration (s)**: 1–600.
- A live view shows requests per second, failures and active users while the test runs.
- Results:
  - Requests per second, error rate and p95.
  - p50/p95/p99/avg/max for DNS, TCP connect, TLS handshake, wait time (TTFB) and download.
  - Status codes and errors, and `Server-Timing` when the server sends it.
- Export results as **JSON** or **CSV**. **PREVIOUS RUNS** keeps your last 100 runs.

### API docs and mock servers (OpenAPI)

- **Import OpenAPI / Swagger spec** (sidebar, **API Docs**): from a file (JSON or YAML), a URL or pasted text. OpenAPI 3.x and Swagger 2.0 are supported.
- The docs view lists endpoints by tag with parameters, request bodies, responses, schemas and examples. **Try in builder** opens an endpoint as a request.
- **Edit spec** opens an editor with a live preview. **Re-sync spec** fetches a URL-imported spec again.
- **Generate collection** creates a collection with one folder per tag.
- **Start mock server** runs a local mock on `127.0.0.1`, on the port shown in the **mock :port** pill.
  - It answers with each operation's example, or one built from the schema.
  - Send `Prefer: code=404` or `Prefer: example=<name>` to choose another response.
  - Spec edits apply without a restart.

### History

- Every request you send from the builder is recorded, including failed ones. Pigeon keeps the last 500.
- **Restore into builder** (history icon on each entry) opens the request with its response. Saved auth secrets are not stored in history, so enter them again.
- Tick two entries and click **Compare selected** to see a diff of the two response bodies.

### Code snippets

The **Export code snippet** button (code icon next to **Send**) turns the current request into cURL, Python (httpx), JavaScript (fetch) or Swift (URLSession). Variables and auth are filled in.

### Search and layout

- **Search** (Cmd/Ctrl+K) finds requests, collections, folders and API specs by name.
- The **sun/moon** button switches between dark and light themes.
- Drag the sidebar edge to resize it, or press Cmd/Ctrl+B to hide it.
- The response shows below the request. To show them side by side, click the layout button at the right of the **RESPONSE** bar. Click it again to go back. A narrow window always shows the response below.
- Drag the divider between the request and the response to resize them. Double-click it to reset it. With the divider selected, the arrow keys also move it. Pigeon keeps the layout and the divider position.
- To make all text larger or smaller, use **View → Zoom In**, **Zoom Out** and **Actual Size**. Pigeon keeps the zoom level after a restart. On Windows, press Alt to show the menu bar.

## Import from Postman and export

The import button (an arrow into a tray) next to **Collections** opens a menu with **Import from Postman…** and **Export all collections and environments…**. When you have no collections yet, the sidebar also shows **Import from Postman**.

### Import from Postman

1. Export from Postman:
   - **One collection:** open the **•••** menu next to the collection and choose **Export** (under **More**). Collection v2.1 and v2.0 files work. You can also export environments and globals.
   - **Everything:** open **Settings → Account settings → Export data** and ask for collections and environments. Postman emails you a link to a zip file. On a Postman team, only admins can export all data.
2. In Pigeon, choose **Import from Postman…**, then:
   - **Choose files…** for one or more exported files, or for the zip file from Postman's email.
   - **Choose folder…** for the folder you unzipped the zip file to.
3. Pigeon lists the collections and environments it found. Clear the check boxes of the ones you do not want, then click **Import**.

Pigeon adds the imports to the Local workspace. If a name is already in use, Pigeon adds " (2)" to it. If a collection uses `{{variables}}` that nothing defines, Pigeon adds them, without values, to a new environment named after the collection. Pigeon leaves out the variables that the collection's scripts set only as collection variables. Postman keeps such variables in environments, which a collection export does not include. The new environment becomes active when no other environment is. Click **Fill in values**, or hover over a variable in a request. Files that are not Postman collections or environments show under **files skipped**, with the reason.

Pigeon and Postman work differently in some places. After the import, Pigeon lists each difference it found and the requests it affects:

- **Folders:** Pigeon has one folder level. A nested folder becomes a folder with a name like "Parent / Child".
- **Folder scripts:** Pigeon has no folder scripts. Pigeon copies each folder script into the requests of that folder.
- **Auth set to inherit:** each request gets the auth of its folder or collection.
- **Path variables:** Pigeon writes the value of a path variable such as `:id` into the URL. A path variable without a value stays in the URL as `:id`.
- **Dynamic variables:** `{{$guid}}` and `{{$randomUUID}}` become `{{$uuid}}`, and `{{$isoTimestamp}}` becomes `{{$isoDate}}`. Pigeon does not supply values for other Postman dynamic variables, such as `{{$randomInt}}`. Pigeon's `{{$timestamp}}` is in milliseconds, and Postman's is in seconds.
- **Globals:** Postman globals become an environment named **Globals**. To use them, make it active.
- **Not imported:** saved example responses and descriptions.
- **Not supported:**
  - Auth types other than the four in [Auth](#auth), and OAuth 2.0 grants other than client credentials. Those requests get no auth.
  - Methods other than the eight in [Requests](#requests). Those requests use GET.
- **Scripts:** a script fails in Pigeon if it uses `pm.sendRequest`, `pm.globals`, `pm.cookies`, `pm.vault`, `require()` or Postman's old `postman.*` and `tests[...]` API. Postman's script libraries are also not in Pigeon: `Buffer`, `atob` and `btoa`, `CryptoJS`, lodash (`_`), `moment`, `cheerio`, `xml2Json` and `tv4`. See [Scripts and tests](#scripts-and-tests).
- **Files:** Pigeon does not keep the file paths of file bodies and file fields, so a shared collection cannot read files on your computer. Choose the files again.

If a variable is empty after the import, Postman did not put its value in the export file. Enter the value in Pigeon.

### Export

- **A collection:** right-click it and choose **Export…**. Pigeon saves a Postman Collection v2.1 file with the folders, requests, variables and scripts.
- **An environment:** open **Environments**, select it and click **Export**.
- **Everything:** choose **Export all collections and environments…**. Pigeon saves one zip file in the layout of Postman's data export. Pigeon can import this zip file. To import it into Postman, unzip it and import the folder.

Postman, Pigeon and other API clients can import these files.

Pigeon does not put secret values in the file unless you select **Include secret values**. Secret values are the secret environment values and the passwords, tokens, API keys and client secrets of request auth. A value that is only a `{{variable}}` always stays in the file. With **Include secret values**, the file holds these values as plain text. Headers, URLs, bodies and scripts are always saved as they are, so check them for passwords or tokens before you share the file. File paths are not saved.

## Keyboard shortcuts

| Shortcut | Action |
|---|---|
| Cmd/Ctrl+Enter | Send the request |
| Cmd/Ctrl+S | Save the request |
| Cmd/Ctrl+K | Search |
| Cmd/Ctrl+B | Show or hide the sidebar |
| Cmd/Ctrl+Period or Esc | Cancel a running request |
| Esc | Close a dialog or menu |
| F2 | Rename the selected item in the sidebar |
| Delete (⌫ on Mac) | Delete the selected item in the sidebar (asks first) |
| ↑ ↓ ← → | Move through the sidebar, open and close folders |

## Your data

- Everything is stored in `pigeon-local.sqlite` in Pigeon's data folder:
  - macOS: `~/Library/Application Support/Pigeon/`
  - Windows: `%APPDATA%\Pigeon\`
- The database holds your collections, environments, API specs, history and load-test runs. Environment secrets and saved auth values are stored in plain text in this file. OAuth tokens are kept in memory only.
- Your requests, OAuth token requests and spec downloads go only to the addresses you enter and to any redirects those servers send. Pigeon checks this repo on GitHub for updates. It has no usage tracking.
- **Export** saves copies of your collections and environments as files. See [Export](#export).
- If the database is damaged, Pigeon asks what to do. It can move the damaged file aside and start fresh. It never deletes it.

## Updates

Installed apps update themselves from this repo.

- **macOS:** Pigeon checks for updates automatically and asks before it installs one. To check now, choose **Pigeon → Check for Updates…**.
- **Windows:** Pigeon checks at launch and every 4 hours, downloads updates in the background, and installs them when you quit.
