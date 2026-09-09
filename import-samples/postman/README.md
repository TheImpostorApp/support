# Postman import samples

Sample **Postman v2.1** exports for trying out Impostor's **Import** flow — a collection
and two environments, the same files you'd get from Postman's own *Export* command.

The requests target the local [mock API stack](../../mock-server/README.md)
(`cd mock-server && docker compose up -d`), so an imported collection is immediately
runnable.

## Try it

1. In Impostor, open (or create) a workspace.
2. **Import…** from the sidebar's **⋮** menu.
3. Add all three files at once — each one's type is auto-detected — and import.
4. Pick the imported **Local (Postman import test)** environment and send a request.

| File | What it is |
|------|------------|
| `mock-api.postman_collection.json` | A **v2.1 collection** covering the importer's whole surface (see below). |
| `local.postman_environment.json` | An environment pointing at the mock stack. Includes `type: "secret"` variables, a disabled variable, and an empty value. |
| `production.postman_environment.json` | A second environment, for trying multi-environment import and switching. Includes a non-string value (`retryCount: 3`). |

## What the collection covers

- **Nested folders**, including a folder inside a folder.
- **Auth inheritance** — collection bearer → folder basic → request-level bearer /
  API-key / an explicit `noauth` that stops inheritance.
- **Every body mode** — raw JSON and XML, urlencoded, form-data with a file field,
  a binary `file` body, and GraphQL.
- **Both URL shapes** — Postman's object form (`host`/`path`/`query`) and the bare
  string shorthand.
- **Disabled** headers, query params and variables, which import as disabled rows.
- **Pre-request and test scripts** — these run as-is, since Impostor's `im.*` script API
  is `pm.*`-compatible.
- **Duplicate request names**, which must not overwrite each other.
- **A deliberately unsupported `oauth1` auth**, so you can see how unmappable pieces are
  surfaced as import warnings rather than silently dropped.

The form-data and binary-file paths (`/tmp/example.txt`, `/tmp/payload.bin`) are
placeholders — point them at a real file before sending those two requests.

## Expected import results

Importing the collection reports **5 folders, 19 requests, 3 variables** and **1 warning**
(the unsupported `oauth1` auth type). The two "Duplicate name" requests don't overwrite
each other — they land as `duplicate-name.request.yaml` and
`duplicate-name-2.request.yaml`.

Importing an environment reports **7 variables** (Local) / **6** (Production). The
`secret`-typed values go into the OS keychain, with only references written to the
`.env/*.env.yaml` file.

See the [Importing docs](https://docs.impostor.uk/import) for the full mapping.
