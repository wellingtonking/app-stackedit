# Architecture

This document is intended to help new contributors get oriented in the StackEdit codebase and quickly find the right place to implement new features.

## High-level overview

StackEdit is primarily a **Vue 2 single-page application** with:

- a Vue application entry point in `src/index.js`
- a root UI shell in `src/components/App.vue`
- centralized state in `src/store/*`
- most business logic in `src/services/*`
- Markdown feature registration in `src/extensions/*`
- a small server-side layer in `server/`
- a Webpack-based build pipeline in `build/` and `config/`

The application is also designed as a PWA. At startup, `src/index.js` installs the offline runtime, loads extensions and optional services, creates the Vue app, and mounts `App` with the Vuex store.

## Runtime architecture

At a high level, the runtime flow looks like this:

1. `src/index.js` bootstraps the app
2. `src/components/App.vue` initializes sync and network services
3. `src/components/Layout.vue` renders the main workspace
4. UI components read and update state through Vuex modules in `src/store/`
5. Services in `src/services/` perform editing, conversion, persistence, sync, and provider integration work

In practice, most features are built across these layers:

```text
Vue components -> Vuex store -> services -> local storage / IndexedDB / remote providers
```

That separation is important when making changes:

- **components** mostly render UI and react to user events
- **Vuex modules** manage application state and shared coordination
- **services** hold most of the real feature logic

## Main source areas

### `src/components/`

This is the main UI layer.

Key files:

- `src/components/App.vue`: root shell, startup orchestration, global notifications/modals/context menu
- `src/components/Layout.vue`: main workspace composition
- `src/components/Editor.vue`: Markdown editing surface
- `src/components/Preview.vue`: rendered preview surface

`Layout.vue` is the best file to inspect when you need to understand how the workspace is assembled. It wires together:

- explorer
- navigation bar
- editor
- preview
- side bar
- status bar
- find/replace
- discussion gutters

Use `src/components/` first when a change is visibly about layout, interaction, or presentation. Then trace down into the store and services that the component relies on.

### `src/store/`

This is the Vuex layer. `src/store/index.js` composes the application from multiple domain-focused modules, including:

- `content` and `contentState`
- `file`, `folder`, and `explorer`
- `layout`
- `modal`
- `notification`
- `discussion`
- `findReplace`
- `workspace`
- `syncLocation`
- `publishLocation`
- `userInfo`
- `queue`
- `data`
- `syncedContent`

If a feature affects more than one component, or needs persistent/shared state, there is usually a Vuex module involved.

Typical reasons to change store code:

- adding or changing shared feature state
- updating getters that drive multiple views
- introducing new actions that coordinate UI and services
- changing how files, workspace state, or sync metadata are represented

### `src/services/`

This is where most of the application logic lives.

Important services include:

- `src/services/editorSvc.js`: editor lifecycle and selection behavior
- `src/services/markdownConversionSvc.js`: Markdown parsing, sectioning, and HTML conversion
- `src/services/markdownGrammarSvc.js`: grammar setup used by the editor
- `src/services/localDbSvc.js`: local persistence in IndexedDB and localStorage synchronization
- `src/services/syncSvc.js`: workspace/file sync orchestration
- `src/services/networkSvc.js`: online/offline state handling
- `src/services/workspaceSvc.js`: workspace-level coordination
- `src/services/explorerSvc.js`: file tree/explorer support
- `src/services/publishSvc.js`: publishing-related workflows
- `src/services/extensionSvc.js`: extension option/registration plumbing

Two especially important boundaries are:

- **local persistence**: `localDbSvc.js`
- **remote synchronization**: `syncSvc.js`

If a feature saves data only on the client, start with `localDbSvc.js`.  
If it affects remote sync behavior, provider state, or workspace reconciliation, start with `syncSvc.js`.

### `src/services/providers/`

This folder contains provider integrations and related helpers for external systems such as:

- GitHub
- GitLab
- Dropbox
- Google Drive
- WordPress
- Zendesk
- Blogger
- workspace providers such as CouchDB/Git-backed workspaces

If a feature is specific to an external integration, or changes how StackEdit talks to a remote platform, this folder is likely the right place.

Also inspect:

- `src/services/providers/common/Provider.js`
- `src/services/providers/common/providerRegistry.js`

These files are the core abstractions for provider registration and content serialization.

### `src/extensions/`

This is the Markdown feature layer.

`src/extensions/index.js` registers the built-in extensions, including support for:

- emoji
- ABC notation
- KaTeX
- Mermaid
- core Markdown behavior

If you need to add or change Markdown syntax handling, rendered output behavior, or feature-specific parsing, start here along with:

- `src/services/extensionSvc.js`
- `src/services/markdownConversionSvc.js`

### `src/styles/`

This folder contains global SCSS and shared visual rules. `App.vue` imports these global styles during startup.

Use `src/styles/` for:

- theme variables
- typography
- shared layout rules
- Markdown presentation rules
- application-wide visual changes

Use component-level `<style>` blocks inside `.vue` files when a styling change is isolated to a specific component.

### `server/`

This is the server-side layer. It contains Express-side entry points and supporting server code.

Look here when a feature touches:

- server endpoints
- auth callback handling
- server-provided configuration
- request/response behavior that is not purely client-side

### `build/` and `config/`

These folders contain the Webpack-based build pipeline and environment-specific configuration.

Look here when a feature touches:

- bundling
- build output
- loader configuration
- dev-server behavior
- environment variables
- CSS or asset pipeline setup

### `static/`

This folder contains static assets and helper pages outside the main SPA code path.

## Where to implement new feature types

Use this section as a quick routing guide when you receive a feature request.

### UI or workflow features

Start with:

- `src/components/`
- the matching Vuex module in `src/store/`

Use this path when the feature changes:

- visible controls
- modals
- navigation
- status indicators
- file explorer behavior
- layout or panel interactions

### Editor behavior

Start with:

- `src/components/Editor.vue`
- `src/services/editorSvc.js`
- potentially `src/store/content.js` or `src/store/contentState.js`

Use this area for:

- selection behavior
- editing interactions
- cursor-related behavior
- editor lifecycle changes
- discussion highlighting tied to editor content

### Preview and rendering changes

Start with:

- `src/components/Preview.vue`
- `src/services/markdownConversionSvc.js`
- `src/extensions/`

Use this area for:

- rendered HTML changes
- preview interaction behavior
- Markdown-to-HTML conversion changes
- table of contents or preview-side rendering improvements

### Markdown feature work

Start with:

- `src/extensions/`
- `src/services/extensionSvc.js`
- `src/services/markdownConversionSvc.js`
- `src/services/markdownGrammarSvc.js`

Use this area for:

- new Markdown syntaxes
- extension-specific rendering
- parsing/highlighting behavior
- syntax feature toggles

### Persistence and offline behavior

Start with:

- `src/services/localDbSvc.js`
- `src/services/networkSvc.js`
- `src/store/data.js` or other relevant store modules

Use this area for:

- IndexedDB persistence
- local cache behavior
- localStorage-backed settings
- offline mode changes

### Sync and provider integrations

Start with:

- `src/services/syncSvc.js`
- `src/services/publishSvc.js`
- `src/services/providers/`
- relevant `syncLocation` or `publishLocation` store modules

Use this area for:

- syncing files/workspaces
- conflict handling
- provider authorization flows
- remote import/export behavior
- platform-specific integrations

### Styling and theming

Start with:

- `src/styles/`
- component `<style>` blocks

Use this area for:

- shared theme changes
- component styling
- Markdown display styling
- responsive layout polish

### Build, environment, or deployment changes

Start with:

- `build/`
- `config/`
- `package.json`
- `Dockerfile`
- `chart/`

Use this area for:

- build configuration
- asset pipeline changes
- development server configuration
- deployment packaging
- Helm or container changes

## Suggested way to trace a feature

When you are new to a feature area, a good debugging and implementation path is:

1. find the visible entry point in `src/components/`
2. inspect the related Vuex module in `src/store/`
3. find the service called from the component or action
4. if Markdown is involved, inspect `src/extensions/`
5. if persistence or sync is involved, inspect `localDbSvc.js` or `syncSvc.js`

That path mirrors how most features are assembled in StackEdit.

## Validation and local workflow

The repository currently defines these developer commands in `package.json`:

```bash
npm start
npm run lint
npm run unit
npm run unit-with-coverage
npm run test
npm run build
```

In the current sandbox environment, `npm run lint` and `npm run unit` fail before execution because the local dev dependencies are not installed, so documentation changes should be validated by reviewing the diff and verifying the described paths against the codebase.
