# AngularMf

[Learn more about this workspace setup and its capabilities](https://nx.dev/getting-started/intro#learn-nx?utm_source=nx_project&amp;utm_medium=readme&amp;utm_campaign=nx_projects) or run `npx nx graph` to visually explore what was created. Now, let's get you up to speed!

## Run tasks

To run tasks with Nx use:

```sh
npx nx <target> <project-name>
```

For example:

```sh
npx nx serve main-shell
npx nx serve retirement

```

These targets are either [inferred automatically](https://nx.dev/concepts/inferred-tasks?utm_source=nx_project&utm_medium=readme&utm_campaign=nx_projects) or defined in the `project.json` or `package.json` files.

[More about running tasks in the docs &raquo;](https://nx.dev/features/run-tasks?utm_source=nx_project&utm_medium=readme&utm_campaign=nx_projects)

## Add new projects

While you could add new projects to your workspace manually, you might want to leverage [Nx plugins](https://nx.dev/concepts/nx-plugins?utm_source=nx_project&utm_medium=readme&utm_campaign=nx_projects) and their [code generation](https://nx.dev/features/generate-code?utm_source=nx_project&utm_medium=readme&utm_campaign=nx_projects) feature.

To install a new plugin you can use the `nx add` command. Here's an example of adding the React plugin:
```sh
npx nx add @nx/react
```

Use the plugin's generator to create new projects. For example, to create a new React app or library:

```sh
# Generate an app
npx nx g @nx/react:app demo

# Generate a library
npx nx g @nx/react:lib some-lib
```

You can use `npx nx list` to get a list of installed plugins. Then, run `npx nx list <plugin-name>` to learn about more specific capabilities of a particular plugin. Alternatively, [install Nx Console](https://nx.dev/getting-started/editor-setup?utm_source=nx_project&utm_medium=readme&utm_campaign=nx_projects) to browse plugins and generators in your IDE.

[Learn more about Nx plugins &raquo;](https://nx.dev/concepts/nx-plugins?utm_source=nx_project&utm_medium=readme&utm_campaign=nx_projects) | [Browse the plugin registry &raquo;](https://nx.dev/plugin-registry?utm_source=nx_project&utm_medium=readme&utm_campaign=nx_projects)


## Install Nx Console

Nx Console is an editor extension that enriches your developer experience. It lets you run tasks, generate code, and improves code autocompletion in your IDE. It is available for VSCode and IntelliJ.

[Install Nx Console &raquo;](https://nx.dev/getting-started/editor-setup?utm_source=nx_project&utm_medium=readme&utm_campaign=nx_projects)

Implementing Dynamic Module Federation in an Nx workspace allows your Angular applications to load remote modules at runtime, offering flexibility in deploying micro-frontend architectures. Here's a step-by-step guide to achieve this:

## Steps to replicate workspace and projects setup


**1. Set Up a New Nx Workspace**

Begin by creating a new Nx workspace and adding the Angular plugin:


```bash
npx create-nx-workspace@latest ng-mf --preset=apps
cd ng-mf
npx nx add @nx/angular
```


This initializes a monorepo structure tailored for Angular development.

**2. Generate Host and Remote Applications**

Create a host application (e.g., `dashboard`) and a remote application (e.g., `profile`):


```bash
nx g @nx/angular:host apps/dashboard --prefix=ng-mf --dynamic
nx g @nx/angular:remote apps/profile --prefix=ng-mf --host=dashboard
```


The `--dynamic` flag configures the host for dynamic federation, allowing it to determine remote locations at runtime.

**3. Configure the Module Federation Manifest**

The host application uses a manifest file to locate remote applications dynamically. This file is typically located at `apps/dashboard/public/module-federation.manifest.json`. Ensure it includes entries for each remote:


```json
{
  "login": "http://localhost:4201"
}
```


This configuration tells the host where to find the `profile` remote application.

**4. Modify the Host's `main.ts` for Dynamic Loading**

Update the host application's `main.ts` to fetch and set remote definitions at runtime:


```typescript
import { setRemoteDefinitions } from '@nx/angular/mf';

fetch('/assets/module-federation.manifest.json')
  .then((res) => res.json())
  .then((definitions) => setRemoteDefinitions(definitions))
  .then(() => import('./bootstrap').catch((err) => console.error(err)));
```


This sequence ensures that the application initializes after loading the remote definitions.

**5. Update Routing to Load Remote Modules Dynamically**

Adjust the host application's routing to load remote modules as needed:


```typescript
import { Route } from '@angular/router';
import { loadRemoteModule } from '@nx/angular/mf';
import { AppComponent } from './app.component';

export const appRoutes: Route[] = [
  {
    path: 'profile',
    loadChildren: () =>
      loadRemoteModule('profile', './Routes').then((m) => m.remoteRoutes),
  },
  {
    path: '',
    component: AppComponent,
  },
];
```


This configuration enables the host to load the `profile` module dynamically when the `profile` route is accessed.

**6. Serve the Applications**

To run both the host and remote applications concurrently, use:


```bash
nx serve dashboard --devRemotes=profile
```


This command starts the `dashboard` application and ensures the `profile` remote is available for dynamic loading.

**7. Verify the Setup**

Navigate to `http://localhost:4200` in your browser. Accessing the `/profile` route should dynamically load and display the `profile` module, confirming that dynamic module federation is functioning correctly.

By following these steps, you enable your Nx-managed Angular applications to dynamically load remote modules, facilitating a flexible and modular micro-frontend architecture.
 