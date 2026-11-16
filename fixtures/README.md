# Test fixtures

Tiny npm packages used by the test suite. None of them is ever published or installed.

- `clean/` — the reference package.
- `attacked-*/` — the same package with one change an attacker might make. Every variant
  must be caught by the diff and rules; `clean` compared with itself must be silent.
- `attacked-lifecycle-script/` — adds only a harmless `prepare` script to `package.json`.
  The diff must flag that file's content change, and the rules must report
  `lifecycle-script-added` (`high`) without `install-script-added`.

The "attacked" files are harmless stand-ins: they print a line or export a string. They are
here so tests can assert on paths and diffs, not to exercise any behaviour.

## Variants and expected findings

Each row treats the variant as the release and names its rebuilt, verified source where it
differs from `clean`. The rule ids and severities come from the assertions in
[`rules.test.ts`](../packages/core/src/rules/rules.test.ts).
[`compare.test.ts`](../packages/core/src/diff/compare.test.ts) also checks the file and
manifest diffs for these variants except `attacked-obfuscated`, whose findings are
asserted in the rule tests.

| Variant | Change and rebuilt source | Rule ids it must trigger | Highest severity |
| --- | --- | --- | --- |
| `clean` | None; compared with itself. | None | None |
| `native-source` | Adds a four-byte native addon stand-in; compared with itself. | `native-code-present` | `info` |
| `attacked-native-swapped` | Replaces the four-byte `build/Release/native.node` stand-in from `native-source` with different bytes. | `native-code-added`, `file-changed` | `critical` |
| `native-identical` | Ships the same four-byte native addon as `native-source`. | `native-code-present` | `info` |
| `attacked-binding-gyp` | Adds the harmless text build definition `binding.gyp`; compared with `clean`. | `native-code-added`, `file-added` | `critical` |
| `attacked-binary` | Adds the 20-byte native addon stand-in `build/Release/native.node`; compared with `clean`. | `native-code-added`, `file-added` | `critical` |
| `attacked-dependency` | Adds `left-pad` at `^1.3.0` to `dependencies` in `package.json`. | `dependency-added` | `high` |
| `attacked-extra-file` | Adds `lib/telemetry.js`, which the source does not produce. | `file-added` | `high` |
| `attacked-modified` | Adds a beacon URL string to `lib/greet.js`. | `file-changed` | `high` |
| `attacked-obfuscated` | Adds `lib/loader.js` with hex-named identifiers and escaped strings. | `obfuscated-code-added`, `file-added` | `critical` |
| `attacked-postinstall` | Adds a `postinstall` script that runs the added `lib/setup.js`. | `install-script-added`, `file-added` | `critical` |

To add a variant, copy `clean`, change one thing, add assertions to both test files above,
and add a row to this table.
