# devenv: nested input overrides are dropped

The native backend honors the first-level follows override `a/direct`, but silently drops the second-level `a/b/c` override.

## Reproduction

```sh
devenv version
devenv --offline update
jq '.nodes.a.inputs, .nodes.b.inputs' devenv.lock
```

All inputs are tiny local path flakes. No network access or builds are needed.
`nixpkgs` is only a dummy input name; `devenv` follows it to avoid default remote inputs. This reproduces input locking, not shell evaluation.

## Expected

`a/direct` and `a/b/c` both follow root `nixpkgs`:

```json
{ "b": "b", "direct": ["nixpkgs"] }
{ "c": ["nixpkgs"] }
```

## Actual

`devenv --offline update` succeeds (exit 0), without an ignored-override warning:

```json
{ "b": "b", "direct": ["nixpkgs"] }
{ "c": "c" }
```

The checked-in `devenv.lock` is the observed output, including the unwanted separate `c` node.

## Versions

- `devenv 2.4.0+b904dcb (x86_64-linux)`
- `nix (Determinate Nix 3.24.0) 2.35.3`
- Linux x86_64

## Related issue

https://github.com/cachix/devenv/issues/3378
