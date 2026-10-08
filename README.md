# Healer — GitHub Action

Runs MEITI's quality pipeline on your repo as a CI step: compile and test checks, an AI code
review, security audit (semgrep), the Pentágono score (lint, dead code, dependency audit), targeted
repairs, and optionally a real-use test where an AI actually uses your running app.

**Your app comes back repaired or unchanged, never worse.** Healer works on a clone of your code,
changes only the area of each error, and if the result is worse than your original (more
vulnerabilities or a lower Pentágono score) it stays locked in quarantine and nothing is written.

[Español más abajo](#español)

## Quick start

1. Get an API key at [meiti.dev/healer](https://meiti.dev/healer).
2. Save it as a repository secret named `HEALER_API_KEY` (Settings → Secrets and variables → Actions).
3. Add the workflow:

```yaml
# .github/workflows/healer.yml
name: Healer
on: [push, pull_request]

jobs:
  healer:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: meiti-dev/healer-action@v1
        with:
          api-key: ${{ secrets.HEALER_API_KEY }}
```

If the code ends `unresolved`, the step fails, like any other CI gate.

## Free or pro AI

```yaml
      - uses: meiti-dev/healer-action@v1
        with:
          api-key: ${{ secrets.HEALER_API_KEY }}
          ai-level: free
```

- `pro` (default): paid APIs (Claude, Gemini Pro). Most accurate on hard code. Your code is never
  used for training.
- `free`: a chain of free-tier providers (Groq, NVIDIA, Gemini Flash). No AI cost. On `free`, the
  repaired code may be reused to retrain MEITI's models, and those providers may use what they
  receive to improve theirs. For private code, use `pro`.

## Real-use test

```yaml
      - uses: meiti-dev/healer-action@v1
        with:
          api-key: ${{ secrets.HEALER_API_KEY }}
          test-uso-real: 'true'
          app-intent: 'A shop: add a product to the cart, open the cart, pay.'
          start-command: 'npm run dev'
          port: '5173'
```

`start-command` and `port` are needed only the first time for each `app-id`.

## What it does and does not do

- Writes back **only the files Healer repaired** (`changed-files` output), and only if the checkout
  is still the code that was sent (verified by SHA-256 of every file). Everything else stays
  byte-for-byte identical.
- Reports `status`, repairs, Pentágono score and pending improvements as outputs and as a summary
  in the Actions tab.
- **Never commits or pushes.** What to do with the result is up to your workflow: fail the build,
  just report, or open a PR with a step like
  [`peter-evans/create-pull-request`](https://github.com/peter-evans/create-pull-request), which
  will see the repaired files in the workspace.

## Inputs

| Input | Default | Description |
|---|---|---|
| `api-key` | *(required)* | Use a secret, never write it in the YAML. |
| `ai-level` | `pro` | `free` or `pro`, see above. |
| `path` | `.` | Folder to send, relative to the checkout. |
| `app-id` | `<owner>/<repo>` | Groups runs of the same app to build its history. |
| `app-intent` | — | Short description of what the app does (improves the real-use test). |
| `max-iterations` | `8` | Maximum repair rounds per run. |
| `test-uso-real` | `false` | `true` to also run the real-use test. |
| `start-command` / `port` | — | How your dev server starts, for the real-use test. |
| `timeout-minutes` | `60` free, `30` pro | How long to wait for the result. If it runs out, the job keeps going in Healer and its result stays in the panel. Give the workflow job more time than this. |
| `fail-on` | `unresolved` | `never` to only report, without failing the step. |
| `base-url` | `https://healer-api.meiti.dev` | Change only to point to your own instance. |

## Outputs

| Output | Description |
|---|---|
| `status` | `production_ready`, `unresolved` or `unsupported_stack`. |
| `repairs-count` | Number of repairs actually applied. |
| `changed-files` | Files repaired and written to the workspace, one per line. |
| `pentagono-score` | Pentágono score, 0 to 100. |
| `app-id` | The app id used. |

## Español

Corre el pipeline de calidad de MEITI sobre tu repo como paso de CI. Tu app vuelve **reparada o
igual, nunca peor**: Healer trabaja sobre un clon, cambia solo la zona de cada error, y si el
resultado empeora queda bloqueado en la cuarentena y no se escribe nada.

1. Consigue tu API key en [meiti.dev/healer](https://meiti.dev/healer).
2. Guárdala como secreto del repo con el nombre `HEALER_API_KEY`.
3. Agrega el workflow de arriba (`uses: meiti-dev/healer-action@v1`).

Con `ai-level: free` la IA no tiene costo. En gratis, el código reparado puede reusarse para
reentrenar los modelos de MEITI, y esos proveedores pueden usar lo que reciben; en `pro` tu código
nunca se usa para entrenar. Para código privado, usa `pro`.

La Action escribe de vuelta **solo los archivos reparados** y solo si tu checkout sigue siendo el
código que se mandó. Nunca hace commit ni push: lo que haces con el resultado lo decide tu workflow.

Documentación completa: [meiti.dev/healer](https://meiti.dev/healer).
