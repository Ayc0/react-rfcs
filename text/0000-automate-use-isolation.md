- Start Date: 2024-11-18
- RFC PR: https://github.com/reactjs/rfcs/pull/260
- React Issue: (leave this empty)

# Summary

This RFC proposes to automatically apply `useIsolation(() => …)` (see https://github.com/reactjs/rfcs/pull/257) to all custom hooks using the [React compiler](https://react.dev/learn/react-compiler).

# Basic example

Let’s say someone write this custom hook in their code:

```jsx
function useLongPoll(url, delay) {
  const [id, setId] = React.useState(0);

  React.useEffect(() => {
    const intervalId = setInterval(() => setId((id) => id + 1), delay);
    return () => clearInterval(intervalId);
  }, [delay]);

  const [status, setStatus] = React.useState();
  React.useEffect(() => {
    const controller = new AbortController();
    fetchStatus(url, { signal: controller.signal }).then((result) =>
      setStatus(result)
    );
    return () => controller.abort();
    // Trigger a re-fetch every so often
  }, [id]);

  return status;
}
```

The React compiler could automatically _patch_ it to look like the following:

```jsx
function useLongPoll(url, delay) {
  return useIsolation(() => _useLongPoll(url, delay), [url, delay]);
}

function _useLongPoll(url, delay) {
  const [id, setId] = React.useState(0);

  React.useEffect(() => {
    const intervalId = setInterval(() => setId((id) => id + 1), delay);
    return () => clearInterval(intervalId);
  }, [delay]);

  const [status, setStatus] = React.useState();
  React.useEffect(() => {
    const controller = new AbortController();
    fetchStatus(url, { signal: controller.signal }).then((result) =>
      setStatus(result)
    );
    return () => controller.abort();
    // Trigger a re-fetch every so often
  }, [id]);

  return status;
}
```

# Motivation

This topic is mostly for performance reasons, see [comment in the original RFC](https://github.com/Ayc0/react-rfcs/blob/main/text/0000-use-isolation.md#wrapping-existing-hooks-for-perf-optimizations-only).

One of the main problem when creating custom hooks / calling custom from 3rd party libraries, is that they can depend on their own update cycles, and they can create a lot of renders that may not be used in the end to drive UI changes. `useIsolation` introduced in another RFC can fix that, but developers still have to manually apply it when needed.

With the introduction of the React compiler, the compiler automatically applies the equivalent of `useMemo`, it seems fitting to also extend it to `useIsolation`.

# Detailed design

TO FILL

# Drawbacks

If the written code properly follows the rule of hooks, no code should depend on render cycles. So automatically applying `useIsolation()` should be fine.

For the code paths not strictly following them, we can follow the current behavior of the compiler: not doing anything.

# Alternatives

TO FILL

# Adoption strategy

TO FILL

# How we teach this

TO FILL

# Unresolved questions

TO FILL
