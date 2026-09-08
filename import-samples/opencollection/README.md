# OpenCollection (Bruno) import samples

Sample [OpenCollection](https://spec.opencollection.com) documents for trying out
Impostor's **Import** flow. OpenCollection is the open YAML format Bruno 3.x writes
natively (the successor to its `.bru` DSL), and a collection comes in two shapes — both
are here, along with the two single-file shapes.

The requests target the local [mock API stack](../../mock-server/README.md)
(`cd mock-server && docker compose up -d`), so an imported collection is immediately
runnable.

## Try it

1. In Impostor, open (or create) a workspace.
2. **Import…** from the sidebar's **⋮** menu.
3. Add `import-samples/opencollection/mock-api/opencollection.yml` and import.
4. Pick the imported **Local (OpenCollection)** environment and send a request.

| Path | What it is |
|------|------------|
| `mock-api/` | An **on-disk collection tree**: `opencollection.yml` at the root, one `.yml` per request, a `folder.yml` per sub-folder, and `environments/*.yml`. Import it by picking `mock-api/opencollection.yml` — the whole tree comes with it. |
| `mock-api-bundled.yml` | The **bundled shape** — one file carrying the tree in nested `items`, with environments inline under `config`. Detected by content, so the file name doesn't matter. |
| `staging.env.yml` | A **standalone environment** file, the shape Bruno writes under `environments/`. Picked on its own it imports straight into the workspace's environments. |

Any single request file (e.g. `mock-api/Basics/headers.yml`) can also be picked on its
own — it imports as one request into the target folder, with no wrapper.

Only a picked `opencollection.yml` or `folder.yml` speaks for its directory. A bundled
file under any other name is read on its own, so dropping one into a folder of unrelated
YAML never sweeps that folder up.

## What `mock-api/` covers

- **Both metadata shapes** — folders with a `folder.yml` (name, `seq`, `request` defaults)
  and one without (`Nested/Inner`, which takes its directory name).
- **Auth inheritance** — collection bearer → folder basic (`Auth/`) → request-level
  overrides, plus an explicit folder-level `none` (`Bodies/`) that stops inheritance.
- **Every auth type** — `inherit`, `none`, basic, bearer, apikey (header & query), digest,
  awsv4, oauth2 (client credentials and authorization code + PKCE), and an `oauth1` that
  Impostor doesn't support, so it surfaces as a warning.
- **Every body type** — `json`, `xml`, `form-urlencoded`, `multipart-form` (with a file
  field), `graphql`, and a `file` body with several `selected` variants.
- **Params** — query params (including a disabled one) and a **path** param, which is
  substituted into the URL because Impostor has no path-parameter table.
- **Protocols** — HTTP, GraphQL, WebSocket, and gRPC (reflection, `grpc://` target).
- **Scripts and assertions** — `before-request` / `after-response` / `tests` hooks, and
  declarative `runtime.assertions`.
- **Ordering** — `info.seq` on every item, which becomes each directory's `.order.yaml`.
- **Edge cases** — disabled headers/params/variables, a typed variable value
  (`{type, data}`), a declared-but-empty `secret: true` variable, two requests sharing a
  name, and a non-YAML file the walker ignores.

The file-body and multipart-file paths (`./fixtures/hello.txt`) are placeholders — point
them at a real file before sending those two requests.

## Expected import results

Importing `mock-api/opencollection.yml` reports **25 requests, 6 folders, 3 variables,
2 environments** and **5 warnings**:

1. path parameters substituted into the URL
2. declarative assertions kept as a comment
3. scripts imported verbatim (Bruno's `bru`/`req`/`res` API needs porting to `im.*`)
4. `oauth1` auth not supported
5. WebSocket draft messages not imported

The two "Duplicate name" requests don't overwrite each other
(`duplicate-name.request.yaml` and `duplicate-name-2.request.yaml`), and the
`secret: true` variable lands in the encrypted vault with no value on disk.

`mock-api-bundled.yml` reports **3 requests, 1 folder, 1 variable, 1 environment** and
the scripts warning. `staging.env.yml` reports **4 variables**.

See the [Importing docs](https://docs.impostor.uk/import) for the full mapping.
