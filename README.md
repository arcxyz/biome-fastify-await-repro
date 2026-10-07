# Fastify `useAwaitThenable` regression

Reproduction for https://github.com/biomejs/biome/issues/12176.

The entire failing input is `repro.ts`:

```ts
import Fastify from "fastify";
await Fastify().listen({ port: 0 });
```

## Reproduce

Tested with Node.js 24.21.0. Install the locked dependencies:

```sh
npm ci --ignore-scripts --no-audit --no-fund
```

Compare the same file with the two installed Biome versions:

```sh
npm run lint:baseline
npm run lint
```

| Command | Biome | Observed result |
| --- | --- | --- |
| `npm run lint:baseline` | 2.5.12 | Exit 0; no diagnostics |
| `npm run lint` | 2.5.15 | Exit 1; `lint/nursery/useAwaitThenable` at `repro.ts:2:1` |

The failing diagnostic is:

```text
`await` used on a non-Promise value.
```

Expected: no diagnostic on `repro.ts`. Fastify 5.12.5 declares the no-callback
overload as `listen(opts?: FastifyListenOptions): Promise<string>` in
`node_modules/fastify/types/instance.d.ts`. This call returns a Promise, so the
`await` is valid.

The commands only lint the files; they do not execute them or start a server.

## Control

```sh
npm run lint:controls
```

This intentionally exits 1 with exactly one `useAwaitThenable` diagnostic on
`controls.ts:2`, the awaited string. The `Promise.resolve(...)` on line 1 is
accepted. The rule is active and still distinguishes these two simple cases.

Both Biome versions share one Fastify installation and one configuration. Only
`nursery/useAwaitThenable` is enabled; there are no suppressions or application
dependencies beyond Fastify. `package-lock.json` fixes the dependency tree.
