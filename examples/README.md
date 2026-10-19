# Example manifests

Synthetic manifests to import into Gateway Atlas with **Import → YAML files**, **Import → Local directory** (this folder), or **Paste YAML**. All names, hosts and certificates are invented, using `example.com` and `example.test`.

Gateway Atlas also ships 30 **bundled examples**, including these, under **Import → Bundled example**. See the [quick start](../docs/quick-start.md#6-which-example-should-i-open).

| File | What it shows |
| --- | --- |
| [valid-gateway.yaml](valid-gateway.yaml) | The basics: a GatewayClass, a Gateway, an HTTPRoute and its Service. |
| [missing-service.yaml](missing-service.yaml) | A route whose backend Service is not in the import (a dashed card and a note). |
| [gateway-api-route-types.yaml](gateway-api-route-types.yaml) | HTTP, gRPC, TCP, TLS and UDP routes, with weights, header and query matches, mirrors, redirects, a BackendTLSPolicy and NGF policies, across many namespaces. |
| [gateway-api-broken-references.yaml](gateway-api-broken-references.yaml) | Typical mistakes, each explained in Diagnostics: wrong `sectionName` or port, a kind or namespace that the listener does not allow, a missing ReferenceGrant, a selector typo and a missing certificate. |
| [migration-in-progress.yaml](migration-in-progress.yaml) | Old and new controllers serving the same hosts: Gateway API beside a legacy Ingress and an NGINX VirtualServer, with a 50/50 split. |
| [playground-shared-hosts.yaml](playground-shared-hosts.yaml) | Migration Playground: two shared hosts behind one ingress-nginx controller, with snippets, CORS, regex and the 16-rule limit. |
| [playground-many-teams.yaml](playground-many-teams.yaml) | Migration Playground: ten Ingresses from six teams behind one public class, plus internal Ingresses. Compare shared, per-team and internal/external Gateway designs. |
| [playground-api-rewrites.yaml](playground-api-rewrites.yaml) | Migration Playground: capture-group rewrites, regex paths, CORS with credentials, a header snippet and a gRPC backend. |
| [playground-oversized-host.yaml](playground-oversized-host.yaml) | Migration Playground: 36 paths on one host (an HTTPRoute holds 16 rules) and a Content-Security-Policy header close to the 4,096-character limit. |

You can also import this folder straight from GitHub with **Import → Web / GitHub URL**, using a permalink of the form `https://github.com/<owner>/<repo>/tree/<commit>/examples`.
