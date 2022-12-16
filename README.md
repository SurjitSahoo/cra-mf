# CRA-MF

Module Federation support for Create-React-App without using "eject" or creating a fork of "react-scripts", just by adding a single configuration `moduleFederation.config.js` file at the root of your application

## Usage

1. Install the latest version of the package form npm as a dependency

   ```sh
   npm i cra-mf
   ```

2. Create a MF configuration file in your project's root directory

   ```diff
   my-app
     ├── node_modules
   + ├── moduleFederation.config.js
     └── package.json
   ```

3. Update the existing calls to `react-scripts` in the `scripts` section of your `package.json` to use the `cra-mf` CLI:

   ```diff title="package.json"
   "scripts": {
   -  "start": "react-scripts start",
   +  "start": "cra-mf start",
   -  "build": "react-scripts build",
   +  "build": "cra-mf build",
      "test": "react-scripts test",
      "eject": "react-scripts eject"
   }
   ```

### Sample `moduleFederation.config.js` file

```js
const { dependencies } = require('./package.json');

module.exports = {
  name: 'app_name', // change me
  filename: 'remoteEntry.js',
  exposes: {
    './hello': './src/hello.tsx',
  },
  remotes: {
    remote1: 'remote@http://route.to.remote/remoteEntry.js',
  },
  shared: {
    ...dependencies,
    react: {
      singleton: true,
      import: 'react',
      shareScope: 'default',
      requiredVersion: dependencies.react,
    },
    'react-dom': {
      singleton: true,
      requiredVersion: dependencies['react-dom'],
    },
    'react-router-dom': {
      singleton: true,
      requiredVersion: dependencies['react-router-dom'],
    },
  },
};
```

## Dynamic Module Federation

You can use dynamic module federation feature to define remotes at run-time and load modules from them dynamically.

### Example:

```jsx
import useFederatedComponent from 'cra-mf';

export default function DynamicRemoteComponent() {
  const { Component, isError } = useFederatedComponent({
    remoteUrl: 'http://localhost:3001/remoteEntry.js',
    moduleToLoad: './AppRoutes',
    remoteName: 'remote_app',
  });

  if (isError) return <div>Error loading remote component</div>;

  return Component ? <Component /> : null;
}
```

### Usage with typescript

```tsx
import useFederatedComponent, { IFederatedComponent } from 'cra-mf';

interface Props {
  remote: IFederatedComponent;
}

export default function DynamicRemoteComponent({ remote }: Props) {
  const { Component, isError } = useFederatedComponent(remote);

  if (isError) return <div>Error loading remote component</div>;

  return Component ? <Component /> : null;
}
```