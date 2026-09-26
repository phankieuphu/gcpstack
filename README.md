# GCPStack

**Your whole Google Cloud, on localhost.**

GCPStack is a planned single-binary emulator for Google Cloud services — Cloud Storage, Pub/Sub, Secret Manager and more — on one port, with one config file, and services that talk to each other. Like LocalStack, but for GCP.

🌐 **Repository:** https://github.com/phankieuphu/gcpstack

> Status: planning. The landing page in [`docs/`](docs/index.html) describes the architecture, roadmap and planned quickstart.

## Why

GCP developers run a separate emulator for each service (Pub/Sub, Firestore/Datastore, fake-gcs-server, bigquery-emulator…), each with its own start command, port and setup. GCPStack aims to replace that with one process, one port (`:4510`) and one internal event bus.

## Features (planned)

- **One binary, one port** — gRPC and REST share `:4510`; the gateway routes by content type.
- **Services that work together** — Storage events go to Pub/Sub, push subscriptions call your HTTP handler, Scheduler publishes to topics.
- **Real APIs, from protos** — servers generated from Google's official `googleapis` definitions. Unimplemented methods return `Unimplemented`, never a fake success.
- **Works with official SDKs** — Go, Java, Python and Node clients via the standard emulator variables or custom endpoints.
- **Terraform-ready** — point the Google provider's custom endpoints at localhost.
- **Seed, persist, reset** — seed from YAML, keep state in memory or SQLite, reset with one call.

## Quickstart (planned)

```yaml
# init.yaml
project: demo-project
services: [storage, pubsub]
storage:
  buckets:
    - name: uploads
      notifications:
        - topic: file-events
pubsub:
  topics:
    - name: file-events
      subscriptions:
        - name: file-worker
          push: http://host.docker.internal:8080/hooks/files
```

```sh
docker run -p 4510:4510 \
  -v ./init.yaml:/etc/gcpstack/init.yaml \
  gcpstack/gcpstack

export STORAGE_EMULATOR_HOST=http://localhost:4510
export PUBSUB_EMULATOR_HOST=localhost:4510
```

## Service coverage plan

| Service                   | Protocol  | Phase | Approach                   |
| ------------------------- | --------- | ----- | -------------------------- |
| Cloud Storage             | JSON REST | 1     | Build                      |
| Pub/Sub                   | gRPC      | 1     | Build from protos          |
| Secret Manager            | gRPC      | 3     | Build from protos          |
| Cloud Tasks               | gRPC      | 3     | Build from protos          |
| Cloud Scheduler           | gRPC      | 3     | Build from protos          |
| Firestore                 | gRPC      | 5     | Proxy official emulator    |
| BigQuery                  | REST      | 5     | Evaluate bigquery-emulator |
| Cloud Run, Functions, GKE | —         | —     | Out of scope               |

## Repository contents

```
.
├── docs/
│   ├── index.html      # landing page (self-contained bundle)
│   └── .nojekyll
├── .github/workflows/
│   └── pages.yml       # deploys docs/ to GitHub Pages
└── README.md
```

## GitHub Pages

The landing page is deployed by [`.github/workflows/pages.yml`](.github/workflows/pages.yml) on every push to `main` that touches `docs/` (or manually via **Actions → Run workflow**).

One-time setup: in **Settings → Pages**, set **Source** to **GitHub Actions**.

To preview locally:

```sh
python3 -m http.server -d docs 8000
# open http://localhost:8000
```

## License

MIT (planned). Independent open-source project; not affiliated with Google.
