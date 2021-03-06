# TypeScript React Native Starter

## Prerequisites

Because you might be developing on one of several different platforms, targeting several different types of devices, basic setup can be involved.
You should first ensure that you can run a plain React Native app without TypeScript.
Follow [the instructions on the React Native website to get started](https://facebook.github.io/react-native/docs/getting-started.html).
When you've managed to deploy to a device or emulator, you'll be ready to start a TypeScript React Native app.

You will also need [Node.js](https://nodejs.org/en/), [NPM](https://www.npmjs.com), and [Yarn](https://yarnpkg.com/lang/en).

## Initializing

Once you've tried scaffolding out an ordinary React Native project, you'll be ready to start adding TypeScript.
Let's go ahead and do that.

```sh
react-native init MyAwesomeProject
cd MyAwesomeProject
```

## Adding TypeScript

The next step is to add TypeScript to your project.
The following commands will:

* add TypeScript to your project
* add [React Native TypeScript Transformer](https://github.com/ds300/react-native-typescript-transformer) to your project
* initialize an empty TypeScript config file, which we'll configure next
* add an empty React Native TypeScript Transformer config file, which we'll configure next
* Adds [typings](https://github.com/DefinitelyTyped/DefinitelyTyped) for React and React Native

Okay let's go ahead and run these.

```sh
yarn add --dev typescript
yarn add --dev react-native-typescript-transformer
yarn tsc --init --pretty --jsx react
touch rn-cli.config.js
yarn add --dev @types/react @types/react-native
```

The `tsconfig.json` file contains all the settings for the TypeScript compile.
The defaults created by the command above are mostly fine, but open the file and uncomment the following line:

```js
{
  ...
  // "allowSyntheticDefaultImports": true,  /* Allow default imports from modules with no default export. This does not affect code emit, just typechecking. */
  ...
}
```

The `rn-cli.config.js` contains the settings for the React Native TypeScript Transformer.
Open it and add the following:

```js
module.exports = {
  getTransformModulePath() {
    return require.resolve("react-native-typescript-transformer");
  },
  getSourceExts() {
    return ["ts", "tsx"];
  }
};
```

## Migrating to TypeScript

Rename the generated `App.js` and `__tests__/App.js` files to `App.tsx`. `index.js` needs to use the `.js` extension.
All new files should use the `.tsx` extension (or `.ts` if the file doesn't contain any JSX).

If you try to run the app now, you'll get an error like `object prototype may only be an object or null`.
This is caused by a failure to import the default export from React as well as a named export on the same line.
Open `App.tsx` and modify the import at the top of the file:

```diff
-import React, { Component } from 'react';
+import React from 'react'
+import { Component } from 'react';
```

Some of this has to do with differences in how Babel and TypeScript interoperate with CommonJS modules.
In the future, the two will stabilize on the same behavior.

At this point, you should be able to run the React Native app.

## Adding TypeScript Testing Infrastructure

Since we're using [Jest](https://github.com/facebook/jest), we'll want to add [ts-jest](https://www.npmjs.com/package/ts-jest) to our devDependencies.

```sh
yarn add --dev ts-jest
```

Then, we'll open up our `package.json` and replace the `jest` field with the following:

```json
"jest": {
  "preset": "react-native",
  "moduleFileExtensions": [
    "ts",
    "tsx",
    "js"
  ],
  "transform": {
    "^.+\\.(js)$": "<rootDir>/node_modules/babel-jest",
    "\\.(ts|tsx)$": "<rootDir>/node_modules/ts-jest/preprocessor.js"
  },
  "testRegex": "(/__tests__/.*|\\.(test|spec))\\.(ts|tsx|js)$",
  "testPathIgnorePatterns": [
    "\\.snap$",
    "<rootDir>/node_modules/"
  ],
  "cacheDirectory": ".jest/cache"
}
```

This will configure Jest to run `.ts` and `.tsx` files with `ts-jest`.

## Installing Type Declaration Dependencies

To get the best experience in TypeScript, we want the type-checker to understand the shape and API of our dependencies.
Some libraries will publish their packages with `.d.ts` files (also called "typed declaration" or "type definition" files) which can describe the shape of the underlying JavaScript.
For other libraries, we'll need to explicitly install the appropriate package in the `@types/` npm scope.

For example, here we'll need types for Jest, React, and React Native, and React Test Renderer.

```ts
yarn add --dev @types/jest @types/react @types/react-native @types/react-test-renderer
```

We saved these declaration file packages to our _dev_ dependencies because we're not publishing this package as a library to npm.
If we were, we might have to add some of them as regular dependencies.

You can read more [here about getting `.d.ts` files](https://www.typescriptlang.org/docs/handbook/declaration-files/consumption.html).

## Ignoring More Files

For your source control, you'll want to start ignoring the `.jest` folder.
If you're using git, we can just add entries to our `.gitignore` file.

```config
# Jest
#
.jest/
```

As a checkpoint, consider committing your files into version control.

```sh
git init
git add .gitignore # import to do this first, to ignore our files
git add .
git commit -am "Initial commit."
```

## Adding a Component

We can now add a component to our app.
Let's go ahead and create a `Hello.tsx` component.
Create a `components` directory and add the following example.

```ts
// components/Hello.tsx
import React from "react"
import { Button, StyleSheet, Text, View } from "react-native"

export interface Props {
  name: string
  enthusiasmLevel?: number
  onIncrement?: () => void
  onDecrement?: () => void
}

interface State {
  enthusiasmLevel: number
}

export class Hello extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props)

    if ((props.enthusiasmLevel || 0) <= 0) {
      throw new Error("You could be a little more enthusiastic. :D")
    }

    this.state = {
      enthusiasmLevel: props.enthusiasmLevel || 1
    }
  }

  onIncrement = () => this.setState({ enthusiasmLevel: this.state.enthusiasmLevel + 1 });
  onDecrement = () => this.setState({ enthusiasmLevel: this.state.enthusiasmLevel - 1 });
  getExclamationMarks = (numChars: number) => Array(numChars + 1).join("!")

  render() {
    return (
      <View style={styles.root}>
        <Text style={styles.greeting}>
          Hello {this.props.name + this.getExclamationMarks(this.state.enthusiasmLevel)}
        </Text>

        <View style={styles.buttons}>
          <View style={styles.button}>
            <Button
              title="-"
              onPress={this.onDecrement}
              accessibilityLabel="decrement"
              color="red"
            />
          </View>

          <View style={styles.button}>
            <Button
              title="+"
              onPress={this.onIncrement}
              accessibilityLabel="increment"
              color="blue"
            />
          </View>
        </View>
      </View>
    )
  }
}

// styles

const styles = StyleSheet.create({
  root: {
    alignItems: "center",
    alignSelf: "center"
  },
  buttons: {
    flexDirection: "row",
    minHeight: 70,
    alignItems: "stretch",
    alignSelf: "center",
    borderWidth: 5
  },
  button: {
    flex: 1,
    paddingVertical: 0
  },
  greeting: {
    color: "#999",
    fontWeight: "bold"
  }
})
```

Whoa! That's a lot, but let's break it down:

* Instead of rendering HTML elements like `div`, `span`, `h1`, etc., we're rendering components like `View` and `Button`.
  These are native components that work across different platforms.
* Styling is specified using the `StyleSheet.create` function that React Native gives us.
  React's StyleSheets allow us to control our layout using Flexbox, and style using other constructs similar to those in CSS stylesheets.

## Adding a Component Test

Now that we've got a component, let's try testing it.

We already have Jest installed as a test runner.
We're going to write snapshot tests for our components, let's add the required add-on for snapshot tests:

```sh
yarn add --dev react-addons-test-utils
```

Now let's create a `__tests__` folder in the `components` directory and add a test for `Hello.tsx`:

```ts
// components/__tests__/Hello.tsx
import React from 'react'
import renderer from 'react-test-renderer'

import { Hello } from "../Hello"

it("renders correctly with defaults", () => {
  const button = renderer.create(<Hello name="World" enthusiasmLevel={1} />).toJSON()
  expect(button).toMatchSnapshot()
})
```

The first time the test is run, it will create a snapshot of the rendered component and store it in the `components/__tests__/__snapshots__/Hello.tsx.snap` file.
When you modify your component, you'll need to update the snapshots and review the update for inadvertent changes.
You can read more about testing React Native components [here](https://facebook.github.io/jest/docs/en/tutorial-react-native.html).

## Next Steps

Check out [our React TypeScript tutorial](https://github.com/Microsoft/TypeScript-React-Starter) where we also cover topics like state management with [Redux](http://redux.js.org).
These same subjects can be applied when writing React Native apps.

Additionally, you may want to look at the [ReactXP](https://microsoft.github.io/reactxp/) if you're looking for a component library written entirely in TypeScript that supports both React on the web as well as React Native.

## Helpful Resources

* [Create React Native TypeScript](https://github.com/mathieudutour/create-react-native-app-typescript) is a port of [Create React Native App](https://github.com/react-community/create-react-native-app) that uses TypeScript.
* [React Native Template TypeScript](https://github.com/emin93/react-native-template-typescript) is a clean and minimalist template for a quick start with TypeScript.
