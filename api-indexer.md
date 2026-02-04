## Table of Contents

- [Quick check](#quick-check)
- [Wazuh 4.14.1 templates](#wazuh-4141-templates)
- [🌐 curl (client URL)](#-curl-client-url)
  - [Usage](#1-usage)
  - [Examples](#2-examples)
    - [username:password GET](#21-usernamepassword-get)
    - [username:password PUT](#22-usernamepassword-put)
    - [Certificates](#23-certificates)
  - [Common API calls](#3-common-api-calls)
    - [Cluster allocation (unassigned shards)](#31-cluster-allocation-unassigned-shards)
    - [Cluster health](#32-cluster-health)
    - [Delete index](#33-delete-index)
    - [List indices](#34-list-indices)
    - [List shards](#35-list-shards)
    - [List templates](#36-list-templates)
    - [Pipeline](#37-pipeline)
    - [Query VD index and obtain x documents](#38-query-vd-index-and-obtain-x-documents)
    - [Retrieve Wazuh template definition](#39-retrieve-wazuh-template-definition-fields-mappings)
    - [List Indexer plugins](#310-list-indexer-plugins)
- [_cat vs _search (VD index)](#4-_cat-vs-search-vd-index)
  - [Check index status (using _cat)](#41-check-index-status-using-_cat)
  - [Retrieve all documents (basic _search)](#42-retrieve-all-documents-basic-_search)
  - [Filter search results by field value](#43-filter-search-results-by-field-value-vd-high)
  - [Limit returned fields using _source](#44-limit-returned-fields-using-_source-vd-high)
  - [Pretty output](#45-use-pretty-printed-output-vd-critical)
- [Replicas to 0](#5-replicas-to-0)

## Quick check

```
GET _cluster/health
GET _cluster/allocation/explain
GET _cat/nodes?v 
GET _cat/indices?v
GET _cat/shards?v 
GET _cat/templates?v&s=name
```

---
## Wazuh 4.14.1 templates

```
GET _cat/templates?v&s=name
```

| name                                                               | index_patterns                                 | order |
| ------------------------------------------------------------------ | ---------------------------------------------- | ----- |
| `wazuh`                                                            | `[wazuh-alerts-4.x-*, wazuh-archives-4.x-*]`   | 0     |
| `wazuh-agent`                                                      | `[wazuh-monitoring-*]`                         | 0     |
| `wazuh-states-inventory-browser-extensions-<SERVER_NAME>_template` | `[wazuh-states-inventory-browser-extensions*]` | 1     |
| `wazuh-states-inventory-groups-<SERVER_NAME>_template`             | `[wazuh-states-inventory-groups*]`             | 1     |
| `wazuh-states-inventory-hardware-<SERVER_NAME>_template`           | `[wazuh-states-inventory-hardware*]`           | 1     |
| `wazuh-states-inventory-hotfixes-<SERVER_NAME>_template`           | `[wazuh-states-inventory-hotfixes*]`           | 1     |
| `wazuh-states-inventory-interfaces-<SERVER_NAME>_template`         | `[wazuh-states-inventory-interfaces*]`         | 1     |
| `wazuh-states-inventory-networks-<SERVER_NAME>_template`           | `[wazuh-states-inventory-networks*]`           | 1     |
| `wazuh-states-inventory-packages-<SERVER_NAME>_template`           | `[wazuh-states-inventory-packages*]`           | 1     |
| `wazuh-states-inventory-ports-<SERVER_NAME>_template`              | `[wazuh-states-inventory-ports*]`              | 1     |
| `wazuh-states-inventory-processes-<SERVER_NAME>_template`          | `[wazuh-states-inventory-processes*]`          | 1     |
| `wazuh-states-inventory-protocols-<SERVER_NAME>_template`          | `[wazuh-states-inventory-protocols*]`          | 1     |
| `wazuh-states-inventory-services-<SERVER_NAME>_template`           | `[wazuh-states-inventory-services*]`           | 1     |
| `wazuh-states-inventory-system-<SERVER_NAME>_template`             | `[wazuh-states-inventory-system*]`             | 1     |
| `wazuh-states-inventory-users-<SERVER_NAME>_template`              | `[wazuh-states-inventory-users*]`              | 1     |
| `wazuh-states-vulnerabilities-<SERVER_NAME>_template`              | `[wazuh-states-vulnerabilities-*]`             | 1     |
| `wazuh-statistics`                                                 | `[wazuh-statistics-*]`                         | 0     |

```
wazuh
wazuh-agent
wazuh-states-inventory-browser-extensions-ubuntu24043_template
wazuh-states-inventory-groups-ubuntu24043_template
wazuh-states-inventory-hardware-ubuntu24043_template
wazuh-states-inventory-hotfixes-ubuntu24043_template
wazuh-states-inventory-interfaces-ubuntu24043_template
wazuh-states-inventory-networks-ubuntu24043_template
wazuh-states-inventory-packages-ubuntu24043_template
wazuh-states-inventory-ports-ubuntu24043_template
wazuh-states-inventory-processes-ubuntu24043_template
wazuh-states-inventory-protocols-ubuntu24043_template
wazuh-states-inventory-services-ubuntu24043_template
wazuh-states-inventory-system-ubuntu24043_template
wazuh-states-inventory-users-ubuntu24043_template
wazuh-states-vulnerabilities-ubuntu24043_template
wazuh-statistics
```

---
## 🌐 curl (client URL)

Command-line tool used to **send HTTP(S) requests to servers**.

You use it to:
- Test or interact with APIs,
- Change configurations,
- Download/upload data.  
all without a browser.

| Methods | Description        |
| ------- | ------------------ |
| GET     | Retrieve (default) |
| POST    | Create             |
| PUT     | Update             |
| DELETE  | Remove             |

> [!info] REST API
> **Representational State Transfer Application Programming Interface** is an architectural style for designing networked applications, particularly web services. It's a set of principles and constraints that guide how an API should be structured to enable communication between different software systems over the internet.

#### Conceptual Flow

1. curl connects via HTTPS to the API port (9200).
2. Authenticates (password or certificate).
3. Sends a JSON payload to an endpoint.
4. The **Wazuh Indexer** applies the change and replies in JSON.

---
### (1) Usage

```bash
curl <MODIFIERS> <AUTHENTICATION> <METHOD> <ENDPOINT> <HEADER> <PAYLOAD>
```

### (2) Examples
#### (2.1) `username:password` GET  

```bash
curl -k -u admin:password "https://localhost:9200/_cat/indices/wazuh*?v&s=i"
```

| Parameters | Description                                                                                                                                                                          |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `-k`       | Ignores SSL (Secure Sockets Layer) certificate validation errors                                                                                                                     |
| `-u`       | Performs HTTP Basic Authentication using `admin:password`                                                                                                                           |
| `_cat`     | It’s short for "catalog", a special namespace in Elasticsearch/OpenSearch for human-readable, table-style output. <br>The `_cat` APIs are mainly for quick inspection and debugging. |
| `?`        | First parameter starting character.                                                                                                                                                  |
| `v`        | **Verbose**: includes headers or column names.                                                                                                                                       |
| `&`        | More than one parameter are separated with an ampersand.                                                                                                                             |
| `s=`       | Sort by.                                                                                                                                                                             |
| `i`        | Index name.                                                                                                                                                                          |

---
#### (2.2) `username:password` PUT

```bash
curl -k -u username:password -X PUT "https://localhost:9200/.opendistro-alerting-alert-history*/_settings" \
     -H 'Content-Type: application/json' \
     -d '{
       "index": {
         "number_of_replicas": 0,
         "auto_expand_replicas": false
       }
     }'
```

| Parameters                                                               | Description                                                                    |
| ------------------------------------------------------------------------ | ------------------------------------------------------------------------------ |
| `-k`                                                                     | Ignores SSL (Secure Sockets Layer) certificate validation errors.              |
| `-u`                                                                     | Performs HTTP Basic Authentication using `username:password`                   |
| `-X PUT`                                                                 | Overrides default GET. Uses PUT to perform a modification.                     |
| `"https://localhost:9200/.opendistro-alerting-alert-history*/_settings"` | API endpoint (Wazuh Indexer).                                                  |
| `-H 'Content-Type: application/json'`                                    | Adds an **HTTP header** to tell the server the request body is in JSON format. |
| `-d '{...}'`                                                             | **Payload** (body) sent with the request.                                      |
 
> [!info] HTTP Basic Authentication
> It adds the following header automatically:
> `Authorization: Basic YWRtaW46YWRtaW4=`
> (That’s the base64-encoded string of `admin:admin`)

---
#### (2.3) Certificates

```bash
curl -sSk --cert /etc/wazuh-indexer/certs/admin.pem \
          --key /etc/wazuh-indexer/certs/admin-key.pem \
          -X PUT "https://localhost:9200/.opendistro-alerting-alert-history*/_settings" \
          -H 'Content-Type: application/json' \
          -d '{
            "index": {
              "number_of_replicas": 0,
              "auto_expand_replicas": false
            }
          }'
```

| Parameters | Description                                                               |
| ---------- | ------------------------------------------------------------------------- |
| `-s`       | Silent mode (no progress bar).                                            |
| `-S`       | Shows errors if they occur.                                               |
| `--cert`   | Specifies public certificate file (admin.pem).                            |
| `--key`    | Specifies private key (admin-key.pem) associated with public certificate. |
This is required when the server demands client authentication, not just HTTPS.
1. The server presents its certificate ✅
2. You also present yours ✅
3. Both sides are verified before the connection proceeds ✅

---
### (3) Common API calls


> [!info] Enhancing readability
>When executing API calls from the CLI, add `| jq` at the end of the command to obtain a properly formatted, readable JSON output instead of a single compressed line. If you prefer using the `?pretty` parameter (`?pretty=true`) for formatting, use it alone without jq, as combining both provides no additional benefit—choose either jq or ?pretty, but not both.

```bash
root@<SERVER_NAME>:~# curl -k -u admin:password -X DELETE https://localhost:9200/wazuh-states-vulnerabilities-<SERVER_NAME>
{"acknowledged":true}root@<SERVER_NAME>:~#
```

```bash
root@<SERVER_NAME>:~# curl -k -u admin:password -X DELETE https://localhost:9200/wazuh-states-vulnerabilities-<SERVER_NAME> | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    21  100    21    0     0    514      0 --:--:-- --:--:-- --:--:--   525
{
  "acknowledged": true
}
root@<SERVER_NAME>:~#
```

```bash
root@<SERVER_NAME>:~# curl -k -u admin:password -X DELETE https://localhost:9200/wazuh-states-vulnerabilities-<SERVER_NAME>?pretty=true
{
  "acknowledged" : true
}
```

---
#### 3.1 Cluster allocation (unassigned shards)

##### Dev tools

```
GET _cluster/allocation/explain
```
##### CLI `username:password`

```bash
curl -k -u admin:password "https://localhost:9200/_cluster/allocation/explain"
```
##### CLI certificates (`/etc/wazuh-indexer/certs`)

```bash
curl -sSk --cert /etc/wazuh-indexer/certs/admin.pem --key /etc/wazuh-indexer/certs/admin-key.pem "https://localhost:9200/_cluster/allocation/explain"
```

---
#### 3.2 Cluster health

##### Dev tools

```
GET _cluster/health
```
##### CLI `username:password`

```bash
curl -k -u admin:password "https://localhost:9200/_cluster/health"
```
##### CLI certificates (`/etc/wazuh-indexer/certs`)

```bash
curl -sSk --cert /etc/wazuh-indexer/certs/admin.pem --key /etc/wazuh-indexer/certs/admin-key.pem "https://localhost:9200/_cluster/health"
```

---
#### 3.3 Delete index

##### Dev tools

```
DELETE wazuh-states-vulnerabilities-<SERVER_NAME>
```
##### CLI `username:password`

```bash
curl -k -u admin:password -X DELETE "https://localhost:9200/wazuh-states-vulnerabilities-<SERVER_NAME>"
```
##### CLI certificates (`/etc/wazuh-indexer/certs`)

```bash
curl -sSk --cert /etc/wazuh-indexer/certs/admin.pem --key /etc/wazuh-indexer/certs/admin-key.pem -X DELETE "https://localhost:9200/wazuh-states-vulnerabilities-<SERVER_NAME>"
```

---
#### 3.4 List indices

##### Dev tools

```
GET _cat/indices?v&s=i 
```
##### CLI `username:password`

```bash
curl -k -u admin:password "https://localhost:9200/_cat/indices?v&s=i"
```
##### CLI certificates (`/etc/wazuh-indexer/certs`)

```bash
curl -sSk --cert /etc/wazuh-indexer/certs/admin.pem --key /etc/wazuh-indexer/certs/admin-key.pem "https://localhost:9200/_cat/indices?v&s=i"
```
##### Dev tools

```
GET _cat/indices/wazuh-*?v&s=i 
```
##### CLI `username:password`

```bash
curl -k -u admin:password "https://localhost:9200/_cat/indices/wazuh-*?v&s=i" 
```
##### CLI certificates (`/etc/wazuh-indexer/certs`)

```bash
curl -sSk --cert /etc/wazuh-indexer/certs/admin.pem --key /etc/wazuh-indexer/certs/admin-key.pem "https://localhost:9200/_cat/indices/wazuh-*?v&s=i"
```

---
#### 3.5 List shards

##### Dev tools

```
GET _cat/shards?v&s=i
```
##### CLI `username:password`

```bash
curl -k -u admin:password "https://localhost:9200/_cat/shards?v&s=i"
```
##### CLI certificates (`/etc/wazuh-indexer/certs`)

```bash
curl -sSk --cert /etc/wazuh-indexer/certs/admin.pem --key /etc/wazuh-indexer/certs/admin-key.pem "https://localhost:9200/_cat/shards?v&s=i"
```

---
#### 3.6 List templates

##### Dev tools

```
GET _cat/templates?v
```
##### CLI `username:password`

```bash
curl -k -u admin:password "https://localhost:9200/_cat/templates?v"
```
##### CLI certificates (`/etc/wazuh-indexer/certs`)

```bash
curl -sSk --cert /etc/wazuh-indexer/certs/admin.pem --key /etc/wazuh-indexer/certs/admin-key.pem "https://localhost:9200/_cat/templates?v"
```

---
#### 3.7 Pipeline

##### Dev tools

```
GET _ingest/pipeline
```
##### CLI `username:password`

```bash
curl -k -u admin:password "https://localhost:9200/_ingest/pipeline?pretty=true"
```
##### CLI certificates (`/etc/wazuh-indexer/certs`)

```bash
curl -sSk --cert /etc/wazuh-indexer/certs/admin.pem --key /etc/wazuh-indexer/certs/admin-key.pem "https://localhost:9200/_ingest/pipeline?pretty=true"
```

> [!info] Filebeat pipeline
> Equivalent to the pipeline defined in `/usr/share/filebeat/module/wazuh/alerts/ingest/pipeline.json`.

---
#### 3.8 Query VD index and obtain x documents

##### Dev tools

```
GET wazuh-states-vulnerabilities-<SERVER_NAME>/_search?size=100 
```
##### CLI `username:password`

```bash
curl -k -u admin:password "https://localhost:9200/wazuh-states-vulnerabilities-<SERVER_NAME>/_search?size=100&pretty=true"
```
##### CLI certificates (`/etc/wazuh-indexer/certs`)

```bash
curl -sSk --cert /etc/wazuh-indexer/certs/admin.pem --key /etc/wazuh-indexer/certs/admin-key.pem "https://localhost:9200/wazuh-states-vulnerabilities-<SERVER_NAME>/_search?size=100&pretty=true"
```

---
#### 3.9 Retrieve Wazuh template definition (fields, mappings)

##### Dev tools

```
GET _template/wazuh
```
##### CLI `username:password`

```bash
curl -k -u admin:password "https://localhost:9200/_template/wazuh?pretty=true"
```
##### CLI certificates (`/etc/wazuh-indexer/certs`)

```bash
curl -sSk --cert /etc/wazuh-indexer/certs/admin.pem --key /etc/wazuh-indexer/certs/admin-key.pem "https://localhost:9200/_template/wazuh?pretty=true"
```

---
#### 3.10 List Indexer plugins

##### Dev tools

```
GET _cat/plugins?v
```
##### CLI `username:password`

```bash
curl -k -u admin:password "https://localhost:9200/_cat/plugins?v"
```

```bash
curl -sSk -u admin:password "https://localhost:9200/_cat/plugins?v"
```

> [!info] `-k` vs. `-sSk`
> The key difference between `curl -k` ... and `curl -sSk` ... lies in output verbosity. 
> The first shows curl's default progress bar, while the second uses -s (silent, hides progress/output) combined with -S (show errors if they occur) and -k (insecure SSL). On successful requests, both return identical results, but -sSk provides cleaner output with no progress meter, making it ideal for scripts or repeated use where only the response (or errors) should appear.
##### CLI certificates (`/etc/wazuh-indexer/certs`)

```bash
curl -sSk --cert /etc/wazuh-indexer/certs/admin.pem --key /etc/wazuh-indexer/certs/admin-key.pem "https://localhost:9200/_cat/plugins?v"
```
##### Result

```
node-1 opensearch-alerting                  2.19.2.0
node-1 opensearch-anomaly-detection         2.19.2.0
node-1 opensearch-asynchronous-search       2.19.2.0
node-1 opensearch-cross-cluster-replication 2.19.2.0
node-1 opensearch-geospatial                2.19.2.0
node-1 opensearch-index-management          2.19.2.0
node-1 opensearch-job-scheduler             2.19.2.0
node-1 opensearch-knn                       2.19.2.0
node-1 opensearch-ml                        2.19.2.0
node-1 opensearch-neural-search             2.19.2.0
node-1 opensearch-notifications             2.19.2.0
node-1 opensearch-notifications-core        2.19.2.0
node-1 opensearch-observability             2.19.2.0
node-1 opensearch-performance-analyzer      2.19.2.0
node-1 opensearch-reports-scheduler         2.19.2.0
node-1 opensearch-security                  2.19.2.0
node-1 opensearch-sql                       2.19.2.0
```

---
### (4) `_cat` vs `search_` (VD index)

> [!NOTE] OpenSearch API
> The **`_cat`** API provides **human-readable, tabular views** of cluster and index metadata. It's ideal for quick diagnostics or summaries and typically supports flags like `?v` for verbose output.
> The **`_search`** API is used to **query documents stored in an index**, returning results that match your query criteria. It supports pagination (`?size=<n>`), filtering (`_source`), and root-level formatting (`?pretty=true`).

---
#### 4.1 Check index status (using `_cat`)

```
GET _cat/indices/wazuh-states-vulnerabilities-<SERVER_NAME>?v
```

---
#### 4.2 Retrieve all documents (basic `_search`)

```
GET <INDEX_NAME>/_search
{
  "query": {
    "match_all": {}
  }
}
```
##### Example:

```
GET wazuh-states-vulnerabilities-<SERVER_NAME>/_search
{
  "query": {
    "match_all": {}
  }
}
```

---
#### 4.3 Filter search results by field value (VD High)

```
GET <INDEX_NAME>/_search?size=<MAX_LIMIT>
{
  "query": {
    "match": {
      "<FIELD_NAME>": "<VALUE>"
    }
  }
}
```
##### Example - Get up to 100 documents with severity High:

```
GET wazuh-states-vulnerabilities-<SERVER_NAME>/_search?size=100
{
  "query": {
    "match": {
      "vulnerability.severity": "High"
    }
  }
}
```

---
#### 4.4 Limit returned fields using `_source` (VD High)

```
GET <INDEX_NAME>/_search?size=<MAX_LIMIT>
{
  "_source": [
    "<FIELD_1>",
    "<FIELD_2>",
    "<FIELD_3>"
  ],
  "query": {
    "match": {
      "<FIELD_NAME>": "<VALUE>"
    }
  }
}
```
##### Example - Return only selected fields for improved readability:

```
GET wazuh-states-vulnerabilities-<SERVER_NAME>/_search?size=100
{
  "_source": [
    "agent.id",
    "agent.name",
    "vulnerability.id",
    "vulnerability.description"
  ],
  "query": {
    "match": {
      "vulnerability.severity": "High"
    }
  }
}
```

---
#### 4.5 Use pretty-printed output (VD Critical)

```
GET <INDEX_NAME>/_search?size=<MAX_LIMIT>&pretty=true
{
  "_source": [
    "<FIELD_1>",
    "<FIELD_2>",
    "<FIELD_3>"
  ],
  "query": {
    "match": {
      "<FIELD_NAME>": "<VALUE>"
    }
  }
}
```
##### Example - Return up to 100 critical vulnerabilities, formatted for readability:

```
GET wazuh-states-vulnerabilities-<SERVER_NAME>/_search?size=100&pretty=true
{
  "_source": [
    "agent.id",
    "agent.name",
    "vulnerability.id",
    "vulnerability.description"
  ],
  "query": {
    "match": {
      "vulnerability.severity": "Critical"
    }
  }
}
```

---
### (5) Replicas to 0

---
#### CLI `username:password`

##### 5.1 Set the replica count to 0 for all `.opendistro-*` indices

```bash
curl -k -u admin:password \
  -X PUT "https://<INDEXER_IP>:9200/.opendistro-*/_settings" \
  -H 'Content-Type: application/json' \
  -d '{
        "index": {
          "number_of_replicas": 0,
          "auto_expand_replicas": false
        }
      }'
```

This applies to:

- `.opendistro-alerting-alert-history*`
- `.opendistro-ism-managed-*`
- `.opendistro-ism-config`
- `.opendistro-job-scheduler-lock`
- any other `.opendistro-*` index created by Alerting or ISM.

---
##### 5.2 Apply persistent ISM history settings at the cluster level

```bash
curl -k -u admin:password \
  -X PUT "https://<INDEXER_IP>:9200/_cluster/settings" \
  -H 'Content-Type: application/json' \
  -d '{
        "persistent": {
          "opendistro": {
            "index_state_management": {
              "history": {
                "number_of_replicas": "0"
              }
            }
          }
        }
      }'
```

---
#### CLI certificates (`/etc/wazuh-indexer/certs`)

##### 5.3 Set the replica count to 0 for all `.opendistro-*`, `.opensearch-*` indices

```bash
curl -sSk \
  --cert /etc/wazuh-indexer/certs/admin.pem \
  --key  /etc/wazuh-indexer/certs/admin-key.pem \
  -X PUT "https://<INDEXER_IP>:9200/.opendistro-*,.opensearch-*/_settings" \
  -H 'Content-Type: application/json' \
  -d '{
        "index": {
          "number_of_replicas": 0,
          "auto_expand_replicas": false
        }
      }'
```

```bash
curl -sSk \
  --cert /etc/wazuh-indexer/certs/admin.pem \
  --key  /etc/wazuh-indexer/certs/admin-key.pem \
  -X PUT "https://<INDEXER_IP>:9200/.open*/_settings" \
  -H 'Content-Type: application/json' \
  -d '{
        "index": {
          "number_of_replicas": 0,
          "auto_expand_replicas": false
        }
      }'
```

---
##### 5.4 Apply persistent ISM history settings at the cluster level

```bash
curl -sSk \
  --cert /etc/wazuh-indexer/certs/admin.pem \
  --key  /etc/wazuh-indexer/certs/admin-key.pem \
  -X PUT "https://<INDEXER_IP>:9200/_cluster/settings" \
  -H 'Content-Type: application/json' \
  -d '{
        "persistent": {
          "opendistro": {
            "index_state_management": {
              "history": {
                "number_of_replicas": "0"
              }
            }
          }
        }
      }'
```
