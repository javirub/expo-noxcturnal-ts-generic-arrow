# Noxcturnal + React Compiler parse a generic arrow in a `.ts` file as JSX

Minimal reproduction for Expo SDK 58's experimental Noxcturnal transform worker
(`experiments.noxcturnalTransformWorker`) together with the React Compiler
(`experiments.reactCompiler`).

`generic.ts` contains a single statement, valid TypeScript that `tsc` accepts:

```ts
export const identity = <T>(value: T) => value;
```

With both experiments on, the bundle fails:

```
 ERROR  SyntaxError: generic.ts: Unexpected token. Did you mean `{'>'}` or `&gt;`? (1:40)
> 1 | export const identity = <T>(value: T) => value;
    |                                        ^
```

The file is `.ts`, so `<T>` is a type parameter list and can never be a JSX tag, but the
React Compiler phase parses it as if JSX were allowed.

```sh
bun install            # or npm install
npx expo export --platform android   # fails (also --platform ios, and with --dev)
```

Controls (each one alone makes the export succeed):

- Remove `experiments.noxcturnalTransformWorker` from `app.json`.
- Set `experiments.reactCompiler` to `false`.
- Write the type parameter as `<T,>`.
- Use a function declaration: `export function identity<T>(value: T) { return value; }`.

Versions: `expo@58.0.16`, `@expo/metro-config@58.0.15`, `noxcturnal@0.1.5`,
`react-native@0.88.0`, `babel-plugin-react-compiler@1.0.0`, `typescript@6.0.3`.

Generic arrows like `get: <T>(path: string) => request<T>("GET", path)` are common in API
clients and typed storage helpers; four of our apps fail to bundle on this pattern.
