# StackEdit

[![Build Status](https://img.shields.io/travis/benweet/stackedit.svg?style=flat)](https://travis-ci.org/benweet/stackedit) [![NPM version](https://img.shields.io/npm/v/stackedit.svg?style=flat)](https://www.npmjs.org/package/stackedit)

> Full-featured, open-source Markdown editor based on PageDown, the Markdown library used by Stack Overflow and the other Stack Exchange sites.

https://stackedit.io/

### Ecosystem

- [Chrome app](https://chrome.google.com/webstore/detail/iiooodelglhkcpgbajoejffhijaclcdg)
- NEW! Embed StackEdit in any website with [stackedit.js](https://github.com/benweet/stackedit.js)
- NEW! [Chrome extension](https://chrome.google.com/webstore/detail/ajehldoplanpchfokmeempkekhnhmoha) that uses stackedit.js
- [Community](https://community.stackedit.io/)

### Build

```bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm start

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report
```

### Project architecture

For a dedicated contributor guide, see [ARCHITECTURE.md](./ARCHITECTURE.md).

StackEdit is a Vue 2 single-page application with a small Node/Express backend.
As a contributor, the quickest way to get oriented is to think of the project in 5 layers:

1. **App bootstrap and shell**: `/src/index.js`, `/src/components/App.vue`
2. **Main UI layout**: `/src/components/Layout.vue` and the components it composes
3. **Application state**: `/src/store/*`
4. **Business logic and integrations**: `/src/services/*`
5. **Markdown rendering and extensions**: `/src/extensions/*`

At runtime, `src/index.js` registers extensions and optional services, installs the offline runtime,
creates the Vue app, and mounts the root `App` component with the Vuex store. `App.vue` is the startup
orchestrator: it loads global styles, initializes sync and network services, and then renders the main
`Layout` once the application is ready.

### Codebase map for new contributors

#### `src/components/`: the user interface

- `App.vue` is the root shell.
- `Layout.vue` is the main workspace and wires together the explorer, editor, preview, side bar, and status areas.
- `Editor.vue` is the editable Markdown surface.
- `Preview.vue` is the rendered HTML surface and handles preview-specific interactions.
- `components/modals/` contains modal workflows.
- `components/common/` contains shared UI helpers and globals.

When you are asked to add or change a visible feature, this is usually the first place to inspect.
However, most non-trivial behavior is delegated to Vuex modules or services, so UI changes often span more than one layer.

#### `src/store/`: state and app-wide coordination

The store is split into many domain modules in `src/store/index.js`, including:

- `content` / `contentState` for document content and editing state
- `file`, `folder`, and `explorer` for the file tree and current document selection
- `layout` for panel sizes, visibility, and responsive behavior
- `modal` and `notification` for transient UI state
- `discussion`, `findReplace`, and `workspace` for feature-specific state
- `syncLocation` and `publishLocation` for external locations

If a feature changes application behavior across multiple components, there is usually a Vuex module involved.
This is the right place to look for state shape, getters, mutations, and actions before wiring new UI.

#### `src/services/`: where most logic lives

This project keeps a large amount of behavior in services rather than in Vue components.
Some important entry points are:

- `editorSvc.js`: editor integration and selection lifecycle
- `markdownConversionSvc.js`: Markdown parsing, sectioning, and HTML conversion
- `markdownGrammarSvc.js`: syntax grammar setup for the editor
- `localDbSvc.js`: IndexedDB/localStorage persistence
- `syncSvc.js`: remote synchronization orchestration
- `networkSvc.js`: online/offline handling
- `workspaceSvc.js` and `explorerSvc.js`: workspace and tree management
- `providers/`: integrations for GitHub, GitLab, Dropbox, Google Drive, WordPress, Zendesk, Blogger, and workspace backends

As a rule of thumb:

- if the change is about **how the editor behaves**, start with `editorSvc.js`
- if it is about **how Markdown is rendered**, start with `markdownConversionSvc.js` and `src/extensions/`
- if it is about **local persistence**, start with `localDbSvc.js`
- if it is about **cloud sync or import/export flows**, start with `syncSvc.js`, `publishSvc.js`, and `src/services/providers/`

#### `src/extensions/`: Markdown feature surface

Markdown features are registered from `src/extensions/index.js`.
This folder contains extension modules such as emoji, KaTeX, Mermaid, ABC notation, and the core Markdown extension.

If you need to add a new Markdown capability or tweak an existing one, inspect:

- `src/extensions/*`
- `src/services/extensionSvc.js`
- `src/services/markdownConversionSvc.js`

#### `src/styles/`: global styling

Global SCSS lives in `src/styles/` and is imported by `App.vue`.
Component-specific styles are typically colocated in each `.vue` file.

Use `src/styles/` when the change affects theme variables, typography, shared layout rules, or Markdown presentation.
Use component styles when the change is local to a single screen or widget.

#### Backend and static assets

- `server/` contains the Express server entry points and server-side endpoints
- `static/` contains static assets and helper pages
- `build/` and `config/` contain the Webpack build pipeline and environment configuration

If a feature touches OAuth callbacks, server configuration, deployment behavior, or bundling, these folders are the next places to inspect.

### Typical feature entry points

- **UI feature**: `src/components/` + matching Vuex module in `src/store/`
- **Editor behavior**: `src/components/Editor.vue` + `src/services/editorSvc.js`
- **Preview/rendering**: `src/components/Preview.vue` + `src/services/markdownConversionSvc.js`
- **Markdown extension**: `src/extensions/` + `src/services/extensionSvc.js`
- **Persistence or offline behavior**: `src/services/localDbSvc.js`, `src/services/networkSvc.js`
- **Sync/provider integration**: `src/services/syncSvc.js`, `src/services/publishSvc.js`, `src/services/providers/`
- **Styling/theme work**: `src/styles/` and component `<style>` blocks
- **Build/configuration work**: `build/`, `config/`, `.eslintrc.js`, `.stylelintrc`, `package.json`

### Developer workflow

Useful existing commands from `package.json`:

```bash
npm start               # run the dev server
npm run lint           # lint JS and Vue files in src/ and server/
npm run unit           # run Jest unit tests
npm run unit-with-coverage
npm run test           # lint + unit tests
npm run build          # production build
```

If you are new to the codebase, a good first debugging flow is:

1. find the visible component in `src/components/`
2. trace its Vuex module in `src/store/`
3. inspect the service it calls in `src/services/`
4. check whether the behavior is rendered through `src/extensions/` or styled through `src/styles/`

That path mirrors how most features are assembled in StackEdit and will get you to the right files quickly.

### Deploy with Helm

StackEdit Helm chart allows easy StackEdit deployment to any Kubernetes cluster.
You can use it to configure deployment with your existing ingress controller and cert-manager.

```bash
# Add the StackEdit Helm repository
helm repo add stackedit https://benweet.github.io/stackedit-charts/

# Update your local Helm chart repository cache
helm repo update

# Deploy StackEdit chart to your cluster
helm install --name stackedit stackedit/stackedit \
  --set dropboxAppKey=$DROPBOX_API_KEY \
  --set dropboxAppKeyFull=$DROPBOX_FULL_ACCESS_API_KEY \
  --set googleClientId=$GOOGLE_CLIENT_ID \
  --set googleApiKey=$GOOGLE_API_KEY \
  --set githubClientId=$GITHUB_CLIENT_ID \
  --set githubClientSecret=$GITHUB_CLIENT_SECRET \
  --set wordpressClientId=\"$WORDPRESS_CLIENT_ID\" \
  --set wordpressSecret=$WORDPRESS_CLIENT_SECRET
```

Later, to upgrade StackEdit to the latest version:

```bash
helm repo update
helm upgrade stackedit stackedit/stackedit
```

If you want to uninstall StackEdit:

```bash
helm delete --purge stackedit
```

If you want to use your existing ingress controller and cert-manager issuer:

```bash
# See https://docs.cert-manager.io/en/latest/tutorials/acme/quick-start/index.html
helm install --name stackedit stackedit/stackedit \
  --set dropboxAppKey=$DROPBOX_API_KEY \
  --set dropboxAppKeyFull=$DROPBOX_FULL_ACCESS_API_KEY \
  --set googleClientId=$GOOGLE_CLIENT_ID \
  --set googleApiKey=$GOOGLE_API_KEY \
  --set githubClientId=$GITHUB_CLIENT_ID \
  --set githubClientSecret=$GITHUB_CLIENT_SECRET \
  --set wordpressClientId=\"$WORDPRESS_CLIENT_ID\" \
  --set wordpressSecret=$WORDPRESS_CLIENT_SECRET \
  --set ingress.enabled=true \
  --set ingress.annotations."kubernetes\.io/ingress\.class"=nginx \
  --set ingress.annotations."cert-manager\.io/cluster-issuer"=letsencrypt-prod \
  --set ingress.hosts[0].host=stackedit.example.com \
  --set ingress.hosts[0].paths[0]=/ \
  --set ingress.tls[0].secretName=stackedit-tls \
  --set ingress.tls[0].hosts[0]=stackedit.example.com
```
