# OpenJEV Support

This fork adds optional [OpenJEV](https://openjev.sh) support alongside the
original [TypeSafe](https://typesafe.ai) integration. TypeSafe remains the
default; OpenJEV is a community gateway to the same Jev model.

## What was added

| File | Change |
|---|---|
| `internal/config/config.go` | Added `provider` and `base_url` fields to the `[jev]` config section, with merge support. |
| `pkg/scrutus/run.go` | Added `ResolveProvider` and `APIKeyEnv` (exported). `Run()` adjusts the model id for OpenJEV before the pipeline starts. `newAssessor()` sets `WithBaseURL` to the OpenJEV endpoint when OpenJEV is selected. `openCache()` uses the provider-aware key env. |
| `cmd/scrutus/main.go` | `cache` subcommand uses `scrutus.APIKeyEnv` for provider-aware key resolution. |
| `README.md` | OpenJEV note after the intro, Quickstart alternative, and config examples. |

## Provider selection rule

1. **Explicit choice wins**: `jev.provider` in `.scrutus.toml` or the
   `JEV_PROVIDER` environment variable (`"openjev"` or `"typesafe"`).
2. **Otherwise**, if `TYPESAFE_API_KEY` is set → TypeSafe (unchanged default).
3. **Otherwise**, if only `OPENJEV_API_KEY` is set → OpenJEV.
4. **Default**: TypeSafe — anyone with a TypeSafe key sees zero behaviour change.

When OpenJEV is selected and `jev.model` is still the default `jev-latest`,
the model id is switched to `openjev` (the only model OpenJEV serves). An
explicitly pinned `jev.model` is always respected. An explicit
`jev.base_url` overrides the endpoint for either provider.

## How to configure

**Environment variables only** (simplest):

```sh
export OPENJEV_API_KEY=your-key
scrutus check ./...
```

**Force OpenJEV even with a TypeSafe key present:**

```sh
export JEV_PROVIDER=openjev
export OPENJEV_API_KEY=your-key
scrutus check ./...
```

**In `.scrutus.toml`:**

```toml
[jev]
provider = "openjev"
# base_url = "https://api.openjev.sh"  # optional, overrides the endpoint
# model = "openjev"                     # optional, defaults to "openjev" for OpenJEV
```

## How it was verified

- A live `POST https://api.openjev.sh/v1/systemone` request with model
  `openjev`, state `ping`, and one noul question returned HTTP 200.
- `grep -r "api.typesafe.ai"` confirms no hardcoded `api.typesafe.ai` default
  remains in the codebase; the TypeSafe endpoint comes from the SDK's own
  default, unchanged.

## Upstream

Original project: https://github.com/SergeAx/scrutus by @SergeAx.
