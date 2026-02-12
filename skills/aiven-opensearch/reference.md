# Aiven OpenSearch — API, CLI, Terraform reference

## Aiven API

- **Base URL:** `https://api.aiven.io/v1/`
- **Auth:** `Authorization: aivenv1 <token>` (create tokens at [console.aiven.io/account/tokens](https://console.aiven.io/account/tokens))

**Create service:**  
`POST /project/<project_name>/service`  
Body: `service_name`, `service_type` (e.g. `opensearch`), `cloud`, `plan`.

**Get service (connection info):**  
`GET /project/<project_name>/service/<service_name>`  
Response includes connection URIs and credentials.

**Update service (e.g. user_config for backup repos):**  
`PATCH /project/<project_name>/service/<service_name>`  
Set `user_config` per [Aiven OpenSearch user config](https://aiven.io/docs/products/opensearch/reference/user-config-options).

Full API: [api.aiven.io/doc](https://api.aiven.io/doc/)

---

## avn CLI

Install: [aiven.io/docs/tools/cli](https://aiven.io/docs/tools/cli)

| Command | Purpose |
|--------|---------|
| `avn service create <name> --service-type opensearch --cloud <region> --plan <plan> -p <project>` | Create OpenSearch service |
| `avn service get -p <project> <service_name>` | Service details and connection info |
| `avn service list -p <project>` | List services in project |
| `avn service index-list -p <project> <service_name>` | List OpenSearch indices |
| `avn service index-delete -p <project> <service_name> <index_name>` | Delete an index |
| `avn service plans opensearch --cloud <region>` | List plans for a cloud |

---

## Terraform

Provider: [registry.terraform.io/providers/aiven/aiven](https://registry.terraform.io/providers/aiven/aiven/latest)

**Resource:** `aiven_opensearch`

Main arguments: `project`, `service_name`, `cloud`, `plan`. Optional: `disk_space`, `user_config` (for backup repos and other OpenSearch/Aiven options). Use the Aiven provider docs for exact attribute names and repository-related `user_config` keys.

Example shape (check provider docs for current schema):

```hcl
resource "aiven_opensearch" "os1" {
  project      = aiven_project.myproject.project
  service_name = "my-opensearch"
  cloud_name   = "google-europe-west1"
  plan         = "startup-4"
}
```

---

## Connection snippets

**Python (opensearch-py):** Build client with `host`, `port`, `http_auth=('avnadmin', password)`, and `use_ssl=True`. If Aiven provides a CA path, set `verify_certs=True` and `ssl_show_warn=False` and pass `ca_certs` if needed. Use the Service URI from the Console or API to get host, port, and password.

**Node (@opensearch-project/opensearch):** `new Client({ node: 'https://...', auth: { username: 'avnadmin', password: '...' }, ssl: { rejectUnauthorized: true } })`. Use Service URI components for the node URL and auth.

---

## Official documentation

- [Get started with Aiven for OpenSearch](https://aiven.io/docs/products/opensearch/get-started)
- [Connect with cURL](https://aiven.io/docs/products/opensearch/howto/opensearch-with-curl)
- [Connect with Python](https://aiven.io/docs/products/opensearch/howto/connect-with-python)
- [Connect with Node.js](https://aiven.io/docs/products/opensearch/howto/connect-with-nodejs)
- [Custom repositories (snapshots)](https://aiven.io/docs/products/opensearch/howto/custom-repositories)
- [Aiven API (v1)](https://api.aiven.io/doc/)
