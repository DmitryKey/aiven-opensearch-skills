---
name: aiven-opensearch
description: Create and manage Aiven for OpenSearch services, connect via REST/Python/Node, manage indices and mappings, run search and Query DSL, configure backups and custom repositories. Covers Aiven API, avn CLI, and Terraform. Use when the user mentions Aiven OpenSearch, creating or connecting to an OpenSearch service on Aiven, index management, OpenSearch queries against Aiven, or Aiven CLI/API/Terraform for OpenSearch.
---

# Aiven for OpenSearch

Procedural guidance for Aiven for OpenSearch: service lifecycle, connection, indices, search, and backups. Aiven-specific only; assume general OpenSearch knowledge.

## Quick start

**Service URI** is the main connection string. It includes host, port, and credentials. Get it from:

- **Aiven Console**: Service → Overview → "Service URI" (copy).
- **CLI**: `avn service get <project> <service_name> --json | jq -r '.connection_uri'` or use the service details output.

**Auth:**

- **OpenSearch (REST/client)**: Use the Service URI as-is; credentials are embedded (e.g. `https://avnadmin:password@host:port`). Prefer TLS; Aiven provides CA when needed.
- **Aiven API** (create/update services, user_config): Create an [API token](https://console.aiven.io/account/tokens) and send `Authorization: aivenv1 <token>`.

## Creating a service

**CLI (avn):**

```bash
avn service create <service_name> \
  --service-type opensearch \
  --cloud <cloud_region> \
  --plan <plan_name> \
  -p <project_name>
```

Example: `--cloud google-europe-west1`, `--plan startup-4`. List plans with `avn service plans opensearch --cloud <region>`.

**API:** POST to `https://api.aiven.io/v1/project/<project>/service` with body:

```json
{
  "cloud": "google-europe-west1",
  "plan": "startup-4",
  "service_name": "my-opensearch",
  "service_type": "opensearch"
}
```

Header: `Authorization: aivenv1 <token>`.

**Terraform:** Use the Aiven provider and `aiven_opensearch` resource. Set `project`, `cloud`, `plan`, `service_name`. See [reference.md](reference.md) for the resource shape.

Service state moves from "Rebuilding" to "Running"; wait before connecting.

## Connecting

**cURL:** Use the Service URI directly. TLS is required; use `-k` only for quick local tests if needed.

```bash
curl -u "avnadmin:PASSWORD" "https://<host>:<port>/_cluster/health?pretty"
```

Replace with the actual user and password from the Service URI or Console.

**Python:** Use the official OpenSearch Python client. Build the client with the host, port, and auth from the Service URI (often `avnadmin` + password). Enable SSL and, if Aiven provides a CA, pass it in the client config. See [reference.md](reference.md) for a minimal snippet.

**Node.js:** Use the `@opensearch-project/opensearch` client with `node`, `node:port`, and auth from the Service URI; enable SSL. See [reference.md](reference.md).

## Indices

**Create (with mappings):**

```bash
curl -X PUT -H "Content-Type: application/json" \
  "<SERVICE_URI>/my-index" \
  -d '{"mappings":{"properties":{"title":{"type":"text"},"count":{"type":"integer"}}}}'
```

**List:** `GET <SERVICE_URI>/_cat/indices?v`

**Delete:** `DELETE <SERVICE_URI>/<index_name>`

**CLI helpers:** `avn service index-list -p <project> <service_name>` and `avn service index-delete -p <project> <service_name> <index_name>` for listing and deleting indices at the service level.

## Search

**All indices:** `GET <SERVICE_URI>/_search`

**Query string:** `GET <SERVICE_URI>/_search?q=field:value`

**Query DSL:** POST to `<SERVICE_URI>/_search` with a JSON body, e.g.:

```json
{
  "query": {
    "match": { "title": "search text" }
  },
  "size": 10
}
```

For advanced Query DSL, use the official OpenSearch documentation; behavior is the same as with self-managed OpenSearch.

## Backups and custom repositories

Snapshot repositories (e.g. S3, GCS, Azure Blob) are configured via **service user_config** in the Aiven API, not via OpenSearch REST API.

- **API:** PATCH/PUT the service and set the appropriate `user_config` keys for the repository type (see Aiven docs for exact key names for S3/GCS/Azure).
- **Terraform:** Set the corresponding arguments on the `aiven_opensearch` resource for the repository.

Documentation: [Manage custom repositories](https://aiven.io/docs/products/opensearch/howto/custom-repositories).

## Additional reference

For detailed API endpoints, full CLI command options, Terraform resource arguments, and official doc links, see [reference.md](reference.md).
