# GraphOWL ingestion SDK for TypeScript

Push entities and relationships into [GraphOWL](https://github.com/graphowl/graphowl) from anything you can write TypeScript against: a script, a worker, a serverless function.

- **Batching handled.** The server accepts up to 1,000 items per request; the client splits larger pushes for you.
- **Safe to retry.** Every push carries an idempotency key and a retry reuses it. `429` and `5xx` responses are retried with capped exponential backoff and jitter; nothing else is.
- **Partial success is reported, not hidden.** You get a verdict per item.
- **Types from the API description.** They are generated from [`openapi.json`](openapi.json), so they cannot drift from what the service publishes.

Status: **early access**, version 0.1.0. The API may change before 1.0. Not yet published to npm.

## Use

```ts
import { GraphOwlClient, IngestBuilder } from "./src/ingest.js";

const client = new GraphOwlClient({ baseUrl: "https://your-graphowl-host", token });
const result = await client.push(
  new IngestBuilder()
    .entity({ kind: "service", name: "payments" })
    .entity({ kind: "database", name: "core", parentFqn: "payments" })
    .build(),
);
```

Entities are named by fully-qualified name, never by id: a parent is `parentFqn`, and the service derives the child's name from it.

## What your code should branch on

Errors are [RFC 9457](https://www.rfc-editor.org/rfc/rfc9457) problem documents.

| Status | Meaning | What to do |
|---|---|---|
| `207` | per-item outcomes | read `results`; some may have failed |
| `400` | the request is wrong | fix the code; do not retry |
| `409` | an idempotency key was reused for different content | fix the key generation |
| `429`, `5xx` | not now | retry with backoff; the SDK does |

## Develop

```bash
npm install
npm test            # generates types from openapi.json, then runs the tests (offline)
npm run typecheck
```

`openapi.json` is a copy of the API description published in the GraphOWL repository; one test checks that every route this SDK calls is still declared there.

## Licence

[Apache-2.0](LICENSE). The GraphOWL name and logos are not covered by it; see the [brand terms](https://github.com/graphowl/graphowl/blob/main/brand/README.md).
