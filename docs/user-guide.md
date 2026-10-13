# Gateway Atlas user guide

This guide takes you from a first run to exploring your own configuration, reading a live cluster, and planning an ingress-nginx migration in the Migration Playground.

- [1. Run Gateway Atlas](#1-run-gateway-atlas)
- [2. A first look](#2-a-first-look)
- [3. Import options](#3-import-options)
- [4. Connect to a live cluster (read-only)](#4-connect-to-a-live-cluster-read-only)
- [5. The Canvas](#5-the-canvas)
- [6. Inspector, search and diagnostics](#6-inspector-search-and-diagnostics)
- [7. Migration Playground](#7-migration-playground)
- [8. Explore Supported CRDs](#8-explore-supported-crds)
- [9. Command line](#9-command-line)
- [10. Troubleshooting](#10-troubleshooting)
- [11. Privacy and safety](#11-privacy-and-safety)

## 1. Run Gateway Atlas

You need Docker: Docker Desktop on macOS, or Docker Engine on x86-64 Linux. The image is multi-architecture, so Docker picks the right variant automatically.

```bash
docker pull ghcr.io/tareqaminul/gateway-atlas-rpi-1:2.2.1
```

```bash
docker run --rm -p 127.0.0.1:8080:8080 ghcr.io/tareqaminul/gateway-atlas-rpi-1:2.2.1 serve --addr 0.0.0.0:8080
```

Open <http://127.0.0.1:8080>. Stop Atlas with `Ctrl+C` in that terminal.

- `-p 127.0.0.1:8080:8080` publishes the UI on your machine only. Keep the `127.0.0.1` part: Atlas has no login, so do not expose it on a shared network.
- To use another port, change the first number, for example `-p 127.0.0.1:9090:8080`, and open port 9090.
- **Help → About** shows the running release.

## 2. A first look

1. Choose **Import → Bundled example** and pick **Minimal Gateway**. The Canvas draws `GatewayClass → Gateway → HTTPRoute → Service → Deployment`.
2. Click any card to open the **inspector** on the right: facts, relationships, findings and YAML.
3. Try **Playground · shared hosts & snippets**, then open **Migration Playground** in the header.

Each successful source is added to **Recent sources** in the header, so you can reload it in one click.

The header holds:
- **Recent sources** and **Import**;
- **Diagnostics**, with a count of errors, warnings and notes;
- **Analyze**, which analyzes the pasted YAML;
- **Migration Playground**;
- **Help**;
- the light/dark theme toggle.

## 3. Import options

Open **Import** and choose one of these. The menu closes when you click elsewhere.

| Option | Use it for |
| --- | --- |
| **Bundled example** | Safe guided samples: Gateway API, NGF policies, NGINX Ingress Controller, cross-namespace grants, route conflicts, and four **Playground** migration scenarios. |
| **YAML files** | One or more local `.yaml` / `.yml` files. |
| **Local directory** | Every YAML file in a folder tree, for example a Helm-rendered or GitOps directory. |
| **Web / GitHub URL** | An HTTPS YAML document, or a public GitHub directory permalink of the form `https://github.com/<owner>/<repo>/tree/<commit>/<path>`. Pin a commit so the import is reproducible. |
| **Kubernetes cluster** | A read-only scan of a live cluster; see [section 4](#4-connect-to-a-live-cluster-read-only). It is disabled unless the server was started with `--enable-live`. |

You can also paste YAML into **Paste YAML** at the bottom of the left panel and select **Analyze**. The editor keeps all documents in one stream and folds `metadata`, `spec` and `status` so you can expand only what you need.

Multi-document YAML and `kind: List` are both supported. Unknown kinds are kept as generic resources. Imported files are analyzed, never modified.

> **Tip:** an Ingress-only or Gateway-only export is normal. Atlas notes Services that are not in the import as information, not as errors, unless you scanned a live cluster where they should exist.

## 4. Connect to a live cluster (read-only)

Atlas reads a cluster through a kubeconfig on the machine that runs the server, never from the browser. Live access is off until you start the server with `--enable-live`.

### 4.1 Prepare a kubeconfig for the container

Use a dedicated copy so the container never sees more than it needs. Run these commands as your normal user:

```bash
mkdir -p "$HOME/.config/gateway-atlas" && chmod 700 "$HOME/.config/gateway-atlas"
```

```bash
kubectl config view --raw --flatten > "$HOME/.config/gateway-atlas/config" && chmod 600 "$HOME/.config/gateway-atlas/config"
```

`--flatten` embeds referenced certificate files, so the copy works inside the container. Treat this file as a credential: never commit or share it.

**Docker Desktop's own Kubernetes:** its kubeconfig points at `https://127.0.0.1:6443`, which inside a container means the container itself. Point your copy at the host name Docker provides instead:

```bash
kubectl --kubeconfig "$HOME/.config/gateway-atlas/config" config set-cluster docker-desktop --server=https://kubernetes.docker.internal:6443
```

Clusters reached through a cloud CLI login, such as AKS, EKS or GKE `exec` plugins, need that CLI inside the container, which the image does not include. Use a kubeconfig with a static token or client certificate for a read-only service account instead.

### 4.2 Start Atlas with live access

```bash
docker run --rm -p 127.0.0.1:8080:8080 \
  --mount type=bind,source="$HOME/.config/gateway-atlas",target=/home/nonroot/.kube,readonly \
  ghcr.io/tareqaminul/gateway-atlas-rpi-1:2.2.1 serve --addr 0.0.0.0:8080 --enable-live
```

The directory must contain a file named `config`. `--mount` fails clearly if the source is missing, whereas `-v` would silently create an empty directory.

### 4.3 Scan

1. Choose **Import → Kubernetes cluster**.
2. Pick a **Context**. Atlas shows the **Target cluster** it will read.
3. Optionally pick a **Namespace**; the default is **All permitted namespaces**.
4. Confirm the scan.

Atlas lists:

- **Core and networking:** Namespaces, Services, EndpointSlices, Pods, Deployments, ConfigMaps, Secrets (metadata only), and Ingresses.
- **Gateway API:** GatewayClasses, Gateways, HTTPRoutes and ReferenceGrants.
- **Discovered API groups:** every installed resource that allows `list` in the NGINX, App Protect, F5 CIS/IPAM, BIG-IP Next and Inference Extension groups (`k8s.nginx.org`, `gateway.nginx.org`, `appprotect.f5.com`, `cis.f5.com`, `fic.f5.com`, `k8s.f5net.com`, `gateway.k8s.f5net.com`, `inference.networking.k8s.io`, `inference.networking.x-k8s.io`).

Other Gateway API kinds such as GRPCRoute or TLSRoute, IngressClasses and other workload kinds are not scanned yet; import them as files if you need them. If your RBAC denies something, the scan still completes and reports what it could not read.

**What Atlas never does with a cluster:** create, update, patch, delete, apply, exec or port-forward. A read-only `ClusterRole` with `list` on these resources is enough. Secret values are removed on the server before analysis.

## 5. The Canvas

The Canvas reads **left to right**: class or controller → traffic entry (Gateway, Ingress, VirtualServer) → routes → Services → workloads. Policies, TLS and configuration sit in nearby lanes. Namespaces are drawn as tiles.

### Layouts

The **Families/Classic** button in the bottom-left toolbar switches the layout; your choice is remembered.
- **Families** (default) gives each Gateway family one row with its policy, iRule and Secret chain, and one band per controller with its own class tile.
- **Classic** is the earlier namespace-tile layout.

### Reading lines

| Line | Meaning |
| --- | --- |
| Solid | Traffic: parent, backend, selector |
| Dashed | Configuration |
| Dotted | Policy attachment |
| Dash-dot | TLS |

Arrows point from the resource that declares a reference to its target. The legend at the bottom lists the types on screen. Type-only labels appear for the selected resource's relationships; path labels show route paths.

### Navigating

- **Zoom:** scroll over the Canvas, or use the zoom buttons. **Fit view** shows everything. Large imports open at a readable zoom with a **Fit all** shortcut.
- **Drag:** drag a card to move it, or drag a namespace tile to move the whole namespace.
- **Lock:** the padlock locks the Canvas so nothing moves by accident.
- **Minimap:** bottom right.
- **Topology view** (top bar): **Traffic paths** focuses on traffic entries; **All resources** shows everything. Choose a **Traffic entry** to follow one Gateway or Ingress.
- **Stop at Services** hides Pods, workload controllers and EndpointSlices, and shows their counts on each Service.
- **Focus** narrows the Canvas to the selected resource's neighbourhood; **Show all** returns.
- **Namespace** (left panel) limits the view to one namespace.

### Save Image

**Save Image** exports the whole topology as:
- **SVG:** vector, best quality;
- **JPG:** a high-resolution picture;
- **PDF:** a vector document.

## 6. Inspector, search and diagnostics

- **Inspector:** select a card. **Details** shows kubectl-style facts, identity, source and relationships; **YAML** shows the original YAML, with the large `kubectl.kubernetes.io/last-applied-configuration` annotation folded; **Warnings** shows the resource's findings. You can resize the inspector.
- **Search** (left panel) searches all text or one field: Kind, Namespace, Label (`key` or `key=value`), API group or Name. Suggestions come from your import.
- **Diagnostics** (header) lists every finding. Click one to jump to the resource.
  - **Error:** definite under the available evidence, for example an invalid Service port, a cross-namespace reference without a ReferenceGrant, or a route with no valid parent.
  - **Warning:** likely, needs review, for example an unmatched selector.
  - **Note:** context or a limitation of a partial import, for example a Service that is not in the files, or a shared hostname.

Static analysis does not prove controller acceptance, endpoint readiness, DNS or reachability.

## 7. Migration Playground

The Migration Playground helps you plan a move from community **ingress-nginx** Ingresses to **Gateway API / NGINX Gateway Fabric**. It is consultative: it explains first and drafts last. Every draft is **figurative**. It can contain errors, omit behaviour, or miss the bigger picture (cost, security, traffic ownership), so review it with platform and application owners and test it in staging. A banner reminds you, and copying or downloading a draft asks you to acknowledge this once per session.

Import your Ingresses (files, directory, URL or a live scan), then open **Migration Playground**. It uses every `networking.k8s.io/v1` Ingress in the current import.

### 7.1 Discover

Select **Run Discovery**. You get:

- **Summary:** Ingresses and namespaces, hosts (and how many are shared), **unique annotations** across the import, ingress classes in use, and findings by severity. Many Ingresses normally share one controller's data plane and load balancer. In NGF, **each Gateway gets its own data plane and LoadBalancer Service**, so the number of Gateways, not Ingresses, sets the number of load balancers.
- **Ingresses table:** one row per Ingress with tier (Simple / Moderate / Advanced), hosts, rules and paths, annotations, snippets, CORS, regex, rewrite, TLS and "shares host with". Sort by column, filter by text, or use the badges (**Shared host**, **Snippet**, **CORS**, **Regex**, **Rewrite**, **TLS**, **Backend TLS**).
  - Expand a row to see its settings, its parsed snippet (headers with their lengths, hidden headers, conditional logic) and its **annotations**, each with an outcome and a note.
  - Where Atlas knows the target, a chip such as **HTTPRoute** `spec.rules.filters.cors` opens that exact field in the CRD explorer.
- **Hosts table:** how ingress-nginx merges Ingresses that share a host today: contributors in precedence order (oldest first), distinct paths against the 16-rule HTTPRoute limit, which `server-snippet` is actually in effect, host-wide regex and duplicate paths.
- **Findings:** grouped by severity. Each one says what happens **today**, what **Gateway API / NGF** offers, and what you need to **decide**, with evidence. Click a resource name to jump to its row.
- **Research more:** links to the F5 NGINX ingress-nginx migration guide. External links always ask before opening a new tab.

### 7.2 Design

Open **2 · Design**. Atlas does **not** preselect a topology.

1. **Add Gateway** for each Gateway you intend to run. Give it a name and namespace. `default` is fine; a dedicated namespace such as `gateway-infra` keeps Gateways, RBAC and certificates apart from applications. Optionally add a purpose and the GatewayClass of your NGF installation.
2. Choose **Listener hostnames** for each Gateway:
   - **Per host:** an exact listener per host. Strict, but you edit the Gateway for every new host, and it counts towards the 64-listener limit.
   - **Wildcard:** listeners such as `*.example.com`, which also cover deeper subdomains but not the apex. Atlas suggests wildcards after a check, and hosts outside them get exact listeners.
   - **No hostname:** hostnames live only on HTTPRoutes. This is the simplest option, but any namespace the Gateway admits can claim any hostname.
3. **Attach hosts:** tick which Gateways each host uses. A host may sit on several Gateways (internal and external, or old and new during a cutover). Expand a host to move an individual Ingress elsewhere.
4. Select **Check Design**.

The check shows:
- the Gateway / load-balancer count against today;
- hosts attached, HTTPRoutes after splits, listeners and ReferenceGrants;
- a card per Gateway;
- design findings, for example a host whose paths are split across Gateways, certificate consolidation, hostname ownership or route splits.

Design produces three outcomes, all included in **Download Report**:
- **Design record:** **Export design (JSON)** and **Import design** let you keep it in a pull request or ticket and reload it later.
- **Decisions to sign off:** what Atlas cannot decide for you (DNS strategy, certificates, namespace admission labels, route split boundaries), each with a suggested owner.
- **Platform skeleton:** figurative YAML for the Gateways, listeners, `allowedRoutes`, namespace labels and ReferenceGrants. It contains no HTTPRoutes or policies.

The **architecture diagram** draws the design down to Services. Scroll over it to zoom, scroll above it to move the page, or select **Open full screen**; **Esc** returns. The sticky bar keeps **Check Design**, **Download Report** and the jump links (Top, Check, Decisions, Skeleton, Diagram) in reach.

### 7.3 Draft (legacy per-Ingress)

The earlier converter maps each Ingress to its own Gateway and HTTPRoutes, with reviewed NGF v2.7.2 policies for body size, timeouts, buffering and rate limits. It is kept for reference and warns when it would create more Gateways than you have ingress classes, or routes over 16 rules. A topology-aware generator that uses your design record is planned.

## 8. Explore Supported CRDs

**Help → Explore Supported CRDs** opens a floating window that stays open while you work. Move it by its title bar, resize it from the corner, or minimise it; only **✕** closes it.

1. **CRD:** choose a CRD, grouped by product and pinned release (Gateway API Standard and Experimental, NGF, NGINX Ingress Controller, App Protect, Inference Extension, F5 CIS, F5 IPAM). CRDs in your current import appear first.
2. **Version:** the storage version is preselected; deprecated versions are marked.
3. **Explain:** shows the full field tree, like `kubectl explain --recursive`. Each field shows its type, required, enum values, default and validation rules. Mutually exclusive alternatives are shown together and marked **one of N**.

Filter by name, path or description, or use **Expand 2 levels**, **Expand all** and **Collapse all**. Select a field to see its full description, its constraints and a copyable `kubectl explain` command.

The catalog schemas are bundled with Atlas, so this works offline.

## 9. Command line

The same binary has a command-line interface. With the container, mount your files read-only:

```bash
docker run --rm -v "$PWD:/work:ro" ghcr.io/tareqaminul/gateway-atlas-rpi-1:2.2.1 validate /work/examples/missing-service.yaml
```

| Command | Purpose |
| --- | --- |
| `analyze <files>` | Resources, relationships and findings (`--json` for JSON) |
| `validate <files>` | Findings only; exits non-zero on errors (useful in CI) |
| `explain <files>` | Static traffic paths with field provenance |
| `graph <files>` | The relationship graph (`--format json`) |
| `assess <files>` | Migration preflight assessment |
| `migrate <files>` | Legacy per-Ingress draft into `--output` (default `generated`) |
| `cluster scan --context NAME [--namespace NAME]` | Read-only scan from a kubeconfig (`--kubeconfig`, `--json`) |
| `serve [--addr ADDR] [--enable-live]` | The web UI |

## 10. Troubleshooting

| Symptom | What to do |
| --- | --- |
| Page does not load | Check the container is running and the port in the URL matches the first number in `-p`. |
| **Kubernetes cluster** is greyed out | Restart with `--enable-live` and a mounted kubeconfig ([section 4](#4-connect-to-a-live-cluster-read-only)). |
| No contexts / kubeconfig error | The mounted folder must contain a regular, readable file named `config` at `/home/nonroot/.kube`. Recreate the container after changing the mount. |
| Cluster connection refused on Docker Desktop | Point the copy at `kubernetes.docker.internal:6443` ([section 4.1](#41-prepare-a-kubeconfig-for-the-container)). |
| Cloud cluster authentication fails | `exec` credential plugins are not in the image; use a static token or certificate for a read-only service account. |
| A relationship line is missing | Check that the source really contains an explicit reference with the exact group, kind, namespace and name. Atlas does not invent links. |
| Unresolved reference | Import the target too, or check its exact API group, version, kind, namespace and name. |
| Canvas looks crowded | Use **Stop at Services**, **Traffic paths**, a single namespace, **Focus**, or switch Families/Classic, then **Fit view**. |
| An external link did nothing | Atlas asks before opening a new tab. If your browser blocks new tabs, use **Copy link**. |

## 11. Privacy and safety

Atlas runs locally and sends your manifests nowhere. Before sharing screenshots, reports, design records or exported YAML, review them for private names, hosts, IP addresses, annotations and internal URLs. **Never** include kubeconfig credentials, tokens, certificates or Secret values in issues or reports. Reduce problems to small synthetic examples, like the ones in [examples](../examples/).
