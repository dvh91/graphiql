# Upgrading `graphiql` from `1.x` to `2.0.0`

Hello GraphiQL user and thanks for upgrading!

This migration guide walks you through all changes that come with `graphiql@2.0.0`, in particular the breaking ones, and will show you how to upgrade your `1.x` implementation.

## `GraphiQL` is now a function component

The `GraphiQL` component in `graphiql@1.x` was a class component. That allowed easy access to its props, state and methods by attaching a ref to it like so:

```jsx
import { GraphiQL } from 'graphiql';
import React from 'react';

class MyComponent extends React.Component {
  _graphiql: GraphiQL;

  componentDidMount() {
    const query = this._graphiql.getQueryEditor().getValue();
  }

  render() {
    return <GraphiQL ref={r => (this._graphiql = r)} />;
  }
}
```

With `graphiql@2` we refactored the codebase to more "modern" React. This also meant replacing all class components with function components. The code above no longer works in `graphiql@2` as attaching refs to function components is not possible in React.

All logic and state management now lives in multiple React contexts, provided by the `@graphiql/react` package. The example above can be refactored a such:

```jsx
import { useEditorContext } from '@graphiql/react';
import { GraphiQLInterface, GraphiQLProvider } from 'graphiql';
import { useEffect } from 'react';

function MyComponent() {
  /**
   * Instead of rendering the `GraphiQL` components we render its two parts
   * separately. The `useEditorContext` hook only works when being invoked in
   * a component that is already wrapped with the context providers.
   */
  return (
    <GraphiQLProvider>
      <InsideContext />
    </GraphiQLProvider>
  );
}

function InsideContext() {
  const { queryEditor } = useEditorContext();

  useEffect(() => {
    const query = queryEditor.getValue();
  }, []);

  return <GraphiQLInterface />;
}
```

Here is a list of all public class methods that existed in `graphiql@1` and its replacement in `graphiql@2`. All the contexts mentioned below can be accessed using a hook exported by `@graphiql/react`.

- `getQueryEditor`: Use the `queryEditor` property from the `EditorContext`.
- `getVariableEditor`: Use the `variableEditor` property from the `EditorContext`.
- `getHeaderEditor`: Use the `headerEditor` property from the `EditorContext`.
- `refresh`: Calling this method should no longer be necessary, all editors will refresh automatically after resizing. If you really need to refresh manually you have to call the `refresh` method on all editor instances individually.
- `autoCompleteLeafs`: Use the `useAutoCompleteLeafs` hook provided by `@graphiql/react` that returns this function.

There are a couple more class methods that were intended to be private and were already removed starting in `graphiql@1.9.0`. Since they were not actually marked with `private`, here's an extension to the above list for these methods:

- `handleClickReference`: This was a callback method triggered when clicking on a type or field. It would open the doc explorer for the clicked reference. If you want to manually mimic this behavior you can use the `push` method from the `ExplorerContext` to add an item to the navigation stack of the doc explorer, and you can use the `setVisiblePlugin` method of the `PluginContext` to show the doc explorer plugin (by passing the `DOC_EXPLORER_PLUGIN` object provided by `@graphiql/react`).
- `handleRunQuery`: To execute a query, use the `run` method of the `ExecutionContext`. If you want to explicitly set an operation name, call the `setOperationName` method of the `EditorContext` provider before that (passing in the operation name string as argument).
- `handleEditorRunQuery`: Use the `run` method of the `ExecutionContext`.
- `handleStopQuery`: Use the `stop` method from the `ExecutionContext`.
- `handlePrettifyQuery`: Use the `usePrettifyQuery` hook provided by `@graphiql/react` that returns this function.
- `handleMergeQuery`: Use the `useMergeQuery` hook provided by `@graphiql/react` that returns this function.
- `handleCopyQuery`: Use the `useCopyQuery` hook provided by `@graphiql/react` that returns this function.
- `handleToggleDocs` and `handleToggleHistory`: Use the `setVisiblePlugin` method of the `PluginContext`.

Some class methods were callbacks to modify state which are not intended to be called manually. All these methods don't have a successor: `handleEditQuery`, `handleEditVariables`, `handleEditHeaders`, `handleEditOperationName`, `handleSelectHistoryQuery`, `handleResetResize` and `handleHintInformationRender`

### `window.g` has been removed

In `graphiql@1.x` the `GraphiQL` class component stored a reference to itself on a global property named `g`. This property has been removed as refs don't exist for function components. (Also, the property was only intended for internal use like testing in the first place.)
