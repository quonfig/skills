# Browser JavaScript, React, React Native

Frontend SDKs **evaluate on the server**: the client sends its context and gets
back already-evaluated values for that context. Use a **frontend (browser) SDK
key** (`qfg sdk-key create --environment <env> --type browser`), never the
backend key. Only flags/configs sent to client SDKs are visible here
(`qfg create --type boolean-flag` sets that by default).

Context is fixed per client: set it at init, then change it with
`updateContext` (JS) or the `contextAttributes` prop (React). There is no
per-call context argument.

## Plain JavaScript — `@quonfig/javascript`

```bash
npm install @quonfig/javascript
```

```js
import { quonfig } from "@quonfig/javascript";

await quonfig.init({
  sdkKey: "your-frontend-sdk-key",
  context: { user: { key: "user-123" } }, // required
});

if (quonfig.isEnabled("my-flag")) {
  // ...
}
const color = quonfig.get("button-color"); // undefined until loaded

await quonfig.updateContext({ user: { key: "user-456" } }); // re-fetches
await quonfig.poll({ frequencyInMs: 60000 }); // optional: keep values fresh
```

Besides `get` and `isEnabled` there are only `getDuration` and `getDetails` —
no typed getters, and no default argument on `get`. Supply your own fallback (`quonfig.get("greeting") ?? "Hello"`).

## React — `@quonfig/react`

```bash
npm install @quonfig/react
```

```jsx
import { QuonfigProvider, useQuonfig, useFlag } from "@quonfig/react";

const WrappedApp = () => (
  <QuonfigProvider
    sdkKey={process.env.NEXT_PUBLIC_QUONFIG_API_KEY}
    contextAttributes={{ user: { key: user.id, email: user.email } }}
    onError={(error) => console.error(error)}
  >
    <App />
  </QuonfigProvider>
);

const Logo = () => {
  const { isEnabled, get, loading } = useQuonfig();
  return isEnabled("new-logo") ? <NewLogo /> : <OldLogo />;
};

const Banner = () => {
  const showBanner = useFlag("new-banner");
  // ...
};
```

- Changing `contextAttributes` re-fetches. `pollInterval` (ms) opts into polling.
- `loading` is true until values arrive; `initialFlags` hydrates SSR without a
  flicker.
- Tests: wrap components in `QuonfigTestProvider config={{ "my-flag": true }}`.
- Typed hooks: `qfg generate --targets react-ts -o ./generated`, then
  `import { useQuonfig } from "./generated/quonfig-client"` and read
  `q.flagSmoke` style accessors.

## React Native — `@quonfig/react-native`

```bash
npm install @quonfig/react-native @quonfig/react @quonfig/javascript base-64 react-native-get-random-values
```

Import `"@quonfig/react-native"` first in `index.js` (it installs polyfills),
then use `QuonfigProvider` / `useQuonfig` from `@quonfig/react-native` exactly
as in React above.
