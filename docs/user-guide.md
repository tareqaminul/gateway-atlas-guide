# Gateway Atlas user guide

This guide covers every part of Gateway Atlas 2.3.1. It starts with running Atlas, then explores your configuration on the Canvas, reads a live cluster, plans a migration in the Migration Playground, and explains CRDs. You do not need to know how Atlas works inside. If you only want to get going, start with the [quick start](quick-start.md).

- [1. Run Gateway Atlas](#1-run-gateway-atlas)
- [2. The workspace at a glance](#2-the-workspace-at-a-glance)
- [3. Import options](#3-import-options)
- [4. Connect to a live cluster (read-only)](#4-connect-to-a-live-cluster-read-only) — including [snapshots](#save-a-snapshot) and [an OpenShift provisioner](#44-on-an-openshift-provisioner-docker-on-rhel)
- [5. The Canvas](#5-the-canvas)
- [6. Search and narrowing down](#6-search-and-narrowing-down)
- [7. Inspector and YAML viewer](#7-inspector-and-yaml-viewer)
- [8. Diagnostics](#8-diagnostics)
- [9. Example gallery](#9-example-gallery)
- [10. Migration Playground](#10-migration-playground)
- [11. Explore Supported CRDs](#11-explore-supported-crds)
- [12. Help, themes and recent sources](#12-help-themes-and-recent-sources)
- [13. Command line](#13-command-line)
- [14. Troubleshooting](#14-troubleshooting)
- [15. Privacy and safety](#15-privacy-and-safety)

## How Atlas works, in two pictures

![Analyze flow: import, parse, analyze, explore](../images/flow-analyze.svg)

![Migration Playground flow: import, Discover, Design, outcomes, with the legacy Draft branch](../images/flow-migration.svg)

## 1. Run Gateway Atlas

You need Docker: Docker Desktop on macOS, or Docker Engine on x86-64 Linux. The image is multi-architecture (`linux/arm64` and `linux/amd64`), so Docker picks the right variant automatically.

```bash
docker pull ghcr.io/tareqaminul/gateway-atlas:latest
```

```bash
docker run --rm -p 127.0.0.1:8080:8080 ghcr.io/tareqaminul/gateway-atlas:latest serve --addr 0.0.0.0:8080
```

Open <http://127.0.0.1:8080>. Stop Atlas with `Ctrl+C` in that terminal.

- **Local only:** `-p 127.0.0.1:8080:8080` publishes the UI on your machine only. Keep the `127.0.0.1` part: Atlas has no login, so do not expose it on a shared network.
- **Another port:** change the first number, for example `-p 127.0.0.1:9090:8080`, and open port 9090.
- **Tags:**
  - `:latest`, used throughout this guide, follows the newest release. Pin a version tag such as `:2.3.1` for a fixed, repeatable setup.
  - The older name `ghcr.io/tareqaminul/gateway-atlas-rpi-1` receives the same version tags for existing setups. Despite the name, it is the same image.
- **Version:** **Help → About** shows the running release.

## 2. The workspace at a glance

![The Gateway Atlas workspace before anything is loaded](../images/welcome.png)

| Area | What it holds |
| --- | --- |
| **Header** | **Recent sources**, **Import**, **Diagnostics** (counts of errors, warnings and notes), **Analyze** (for pasted YAML), **Migration Playground**, **Help**, and the light/dark theme toggle. |
| **Left panel** | **Namespace** filter, **Search**, the resource list grouped by kind, and **Paste YAML**. The **«** button hides the panel. |
| **Topology bar** | A summary of what is shown (for example "16/26 resources · 4 hidden below Services"), the **Topology view**, the **Traffic entry** picker, **Stop at Services**, **Hide Secrets** and **Focus**. |
| **Canvas** | The relationship map, with the toolbar (zoom, fit, lock, layout, **Save snapshot** for live scans, **Save Image**), the legend, the minimap and the **CRDs** shortcut. |
| **Inspector** | Opens on the right when you select a resource. |
| **Status bar** | What you are looking at, for example "Exploring Example: Migration In-Progress". |

On an empty start page, **Analyze example** loads a small sample in one click.

## 3. Import options

Open **Import** and choose one of these. The menu closes when you click elsewhere.

![The Import menu](../images/import-menu.png)

| Option | Use it for |
| --- | --- |
| **Bundled example** | Safe, synthetic samples, grouped into **Scenarios**, **Learning cases**, **Migration Playground** and **BIG-IP Next**. See [section 9](#9-example-gallery) and the quick start's [example table](quick-start.md#6-which-example-should-i-open). |
| **YAML files** | One or more local `.yaml` / `.yml` files. |
| **Local directory** | Every YAML file in a folder tree, for example a Helm-rendered or GitOps directory. |
| **Web / GitHub URL** | An HTTPS YAML document, or a public GitHub directory permalink of the form `https://github.com/<owner>/<repo>/tree/<commit>/<path>`. Pin a commit so the import is reproducible. |
| **Kubernetes cluster** | A read-only scan of a live cluster; see [section 4](#4-connect-to-a-live-cluster-read-only). It is unavailable unless the server was started with `--enable-live`. |

### Paste YAML

Expand **Paste YAML** at the bottom of the left panel and paste one or more documents. Select **Analyze** in the header.
- The editor highlights YAML, numbers lines and folds sections.
- The expand button gives the editor more room.

![Pasted YAML and its analysis side by side; the missing shop-api Service is drawn dashed](../images/paste-yaml.png)

Multi-document YAML and `kind: List` are both supported. Unknown kinds are kept as generic resources. Imported files are analyzed, never modified.

**Secret values are removed.** If you paste or import a Secret with `data` or `stringData`, Atlas removes the values before analysis. After **Analyze**, the editor and **Recent sources** keep only the text without them, and the status bar says "Secret values removed from the YAML input". Your files on disk are not changed. Imports may be up to 32 MiB.

> **Tip:** an Ingress-only or Gateway-only export is normal. A Service that is not in the import is drawn as a dashed placeholder and reported as a note, not as an error. The exception is a live-cluster scan, where the Service should exist.

## 4. Connect to a live cluster (read-only)

Atlas reads a cluster through a kubeconfig held by the server, never from the browser. Live access is off until you start the server with `--enable-live`. The kubeconfig can be mounted into the container ([4.1](#41-prepare-a-kubeconfig-for-the-container)) or loaded in the browser ([4.3](#43-scan)).

### 4.1 Prepare a kubeconfig for the container

Use a dedicated copy, so the container never sees more than it needs. Run these commands as your normal user:

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

Clusters reached through a cloud CLI login, such as the AKS, EKS or GKE `exec` plugins, need that CLI inside the container, and the image does not include it. Use a kubeconfig with a static token or client certificate for a read-only service account instead.

### 4.2 Start Atlas with live access

```bash
docker run --rm -p 127.0.0.1:8080:8080 \
  --mount type=bind,source="$HOME/.config/gateway-atlas",target=/home/nonroot/.kube,readonly \
  ghcr.io/tareqaminul/gateway-atlas:latest serve --addr 0.0.0.0:8080 --enable-live
```

The directory must contain a file named `config`. `--mount` fails clearly if the source is missing, whereas `-v` would silently create an empty directory.

### 4.3 Scan

1. Choose **Import → Kubernetes cluster**.
   - No kubeconfig mounted? Select **Load kubeconfig** and pick a file. Atlas keeps it in the server's memory only, until **Forget** or a restart. It is never saved or sent back. Use a flattened, read-only one (`kubectl config view --raw --flatten`); `exec` login plugins and certificate file paths are rejected with a message saying why.
2. Pick a **Context**. Atlas shows the **Target cluster** it will read.
3. Optionally pick a **Namespace**; the default is **All permitted namespaces**.
4. Select **Scan read-only**.

![The cluster import dialog](../images/live-cluster.png)

Atlas lists:

- **Core and networking:** Namespaces, Services, EndpointSlices, Pods, Deployments, ConfigMaps, Secrets (metadata only) and Ingresses.
- **Gateway API:** GatewayClasses, Gateways, HTTPRoutes and ReferenceGrants.
- **Discovered API groups:** every installed resource that allows `list` in these groups:
  - NGINX: `k8s.nginx.org`, `gateway.nginx.org`;
  - F5 WAF for NGINX (App Protect): `appprotect.f5.com`;
  - F5 CIS and IPAM: `cis.f5.com`, `fic.f5.com`;
  - BIG-IP Next: `k8s.f5net.com`, `gateway.k8s.f5net.com`, and for BIG-IP Next for Kubernetes 2.x `gateway.k8s.f5.com`, `k8s.f5.com`, `inference.k8s.f5.com`;
  - Inference Extension: `inference.networking.k8s.io`, `inference.networking.x-k8s.io`.

Other Gateway API kinds, such as GRPCRoute or TLSRoute, IngressClasses and other workload kinds are not scanned yet; import them as files if you need them. If your RBAC denies something, the scan still completes. The status bar counts the scan warnings, and **Diagnostics** lists what could not be read. If the scan fails, Atlas says why (the name could not be resolved, a timeout, a TLS problem, credentials or permissions) and offers **Try again**.

**What Atlas never does with a cluster:** create, update, patch, delete, apply, exec or port-forward. A read-only `ClusterRole` with `list` on these resources is enough. Secret values are removed on the server before analysis.

#### Save a snapshot

After a scan, **Save snapshot** in the Canvas toolbar saves what Atlas read as one YAML file in your downloads, such as `atlas-snapshot-<cluster>-all-<time>.yaml`. It makes no further call to the cluster.

![Save snapshot beside Save Image in the Canvas toolbar](../images/save-snapshot.png)

- **Import it later** with **Import → YAML files**, even without cluster access. The status bar reads "Exploring Snapshot: *cluster* · *scope* · taken *time*". A snapshot counts as cluster evidence, like a live scan: a missing Service is an error, and controller conditions are reported.
- **What is in it:** every scanned resource with its status, sorted and in kubectl's field order. `managedFields` is left out. The header records the context, cluster, scope, time and any scan warnings. If the scan reported warnings or found nothing, Atlas asks first and marks the snapshot incomplete.
- **Treat it as confidential.** Secret values are removed, but ConfigMap data, environment values, annotations, addresses and hostnames remain.
- **Not a backup:** it holds only what Atlas reads, and it cannot be applied with `kubectl`.

### 4.4 On an OpenShift provisioner (Docker on RHEL)

Atlas runs beside the cluster, in Docker on the provisioner host, not inside OpenShift. Four things differ from a laptop.

**1. Give Atlas its own read-only identity.** Rather than the installer's admin kubeconfig, create a ServiceAccount that can only read. Run these as a cluster admin:

```bash
oc create namespace gateway-atlas
```

```bash
oc create serviceaccount atlas-reader -n gateway-atlas
```

```bash
oc apply -f - <<'YAML'
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: gateway-atlas-readonly
rules:
- apiGroups: [""]
  resources: [namespaces, services, pods, configmaps]
  verbs: [get, list, watch]
- apiGroups: [apps]
  resources: [deployments]
  verbs: [get, list, watch]
- apiGroups: [networking.k8s.io]
  resources: [ingresses]
  verbs: [get, list, watch]
- apiGroups: [discovery.k8s.io]
  resources: [endpointslices]
  verbs: [get, list, watch]
- apiGroups: [gateway.networking.k8s.io]
  resources: [gatewayclasses, gateways, httproutes, referencegrants]
  verbs: [get, list, watch]
- apiGroups: [k8s.nginx.org, gateway.nginx.org, appprotect.f5.com, cis.f5.com, fic.f5.com, k8s.f5net.com, gateway.k8s.f5net.com, gateway.k8s.f5.com, k8s.f5.com, inference.k8s.f5.com, inference.networking.k8s.io, inference.networking.x-k8s.io]
  resources: ["*"]
  verbs: [get, list, watch]
YAML
```

```bash
oc create clusterrolebinding gateway-atlas-readonly --clusterrole=gateway-atlas-readonly --serviceaccount=gateway-atlas:atlas-reader
```

The role leaves out Secrets on purpose: a token that can list Secrets can read every Secret value in the cluster. The scan still completes and notes what it could not read.

**2. Build a kubeconfig for that identity**, as a normal (non-root) user:

```bash
mkdir -p "$HOME/.config/gateway-atlas" && chmod 700 "$HOME/.config/gateway-atlas"
```

```bash
oc config view --raw --minify --flatten -o jsonpath='{.clusters[0].cluster.certificate-authority-data}' | base64 -d > "$HOME/.config/gateway-atlas/ca.crt"
```

```bash
oc --kubeconfig "$HOME/.config/gateway-atlas/config" config set-cluster ocp --server="$(oc whoami --show-server)" --certificate-authority="$HOME/.config/gateway-atlas/ca.crt" --embed-certs=true
```

```bash
oc --kubeconfig "$HOME/.config/gateway-atlas/config" config set-credentials atlas-reader --token="$(oc create token atlas-reader -n gateway-atlas --duration=24h)"
```

```bash
oc --kubeconfig "$HOME/.config/gateway-atlas/config" config set-context ocp-readonly --cluster=ocp --user=atlas-reader && oc --kubeconfig "$HOME/.config/gateway-atlas/config" config use-context ocp-readonly && chmod 600 "$HOME/.config/gateway-atlas/config" && rm "$HOME/.config/gateway-atlas/ca.crt"
```

The token expires; run the `set-credentials` command again to renew it.

**3. Run Atlas with the host's network.** Lab and provisioner API names often come from `/etc/hosts` or a local DNS server, sometimes on IPv6 only, and a container on Docker's default network cannot see them: the scan then waits and fails with "The cluster could not be reached". Host networking avoids that; binding to `127.0.0.1` keeps Atlas local:

```bash
docker run -d --name gateway-atlas --restart unless-stopped --network host --user "$(id -u):$(id -g)" -v "$HOME/.config/gateway-atlas:/atlas-kube:ro,Z" -e KUBECONFIG=/atlas-kube/config ghcr.io/tareqaminul/gateway-atlas:latest serve --addr 127.0.0.1:8080 --enable-live
```

- `--user "$(id -u):$(id -g)"` lets the container read your owner-only kubeconfig; do not make the file readable by others.
- `:ro,Z` mounts the folder read-only and relabels it for SELinux, which can stay enforcing. Use it only on this dedicated folder.
- With Podman (`podman-docker`), use `--userns=keep-id` instead of `--user`.

To see the real cause of a connection problem, run one read-only scan from the command line; it prints the error:

```bash
docker run --rm --network host --user "$(id -u):$(id -g)" -v "$HOME/.config/gateway-atlas:/atlas-kube:ro,Z" -e KUBECONFIG=/atlas-kube/config ghcr.io/tareqaminul/gateway-atlas:latest cluster scan --context ocp-readonly --namespace default
```

**4. Open it from your laptop** through an SSH tunnel, then browse to <http://127.0.0.1:8080>:

```bash
ssh -L 8080:127.0.0.1:8080 <user>@<provisioner>
```

Atlas does not scan OpenShift Routes yet; it reads Ingresses, Gateway API resources and the NGINX and F5 resources listed above.

## 5. The Canvas

The Canvas reads **left to right**:

class or controller → traffic entry (Gateway, Ingress, VirtualServer, TransportServer) → routes → Services → workloads

Policies, TLS and configuration sit in nearby lanes, and namespaces are drawn as tiles.

![Gateway API - Various Route Types: HTTP, gRPC, TCP, TLS and UDP routes across ten namespaces](../images/canvas-big-picture.png)

### 5.1 Layouts

The layout button in the toolbar names the current layout (**Layout: Families** or **Layout: Classic**). Select it to switch; a short notice says what the other layout emphasises, and your choice is remembered.
- **Families** (default) answers *where does traffic go?* Every namespace is drawn as rows, one per Gateway, Ingress, VirtualServer or TransportServer, with its policies and configuration, and namespaces are grouped into one band per controller. Atlas places related cards together:
  - each GatewayClass tile sits beside and slightly above its Gateways, even when they share a namespace with another controller's Gateways;
  - a Service several routes share sits to their right, between them, and a route's own Service sits level with it;
  - a namespace reached from another one sits next to the card that links to it, or under it when a straight line would cross other cards.
- **Classic** answers *what does each namespace hold?* with the earlier namespace-tile layout.

Large imports stay readable. When a column would hold more than eight namespaces, Atlas wraps it into a grid shaped like the window.

### 5.2 Reading cards and lines

- **Cards:** each card shows its kind and name. Small badges add facts, such as the hostnames on a route or "1 deploy" on a Service. A dashed card is something that is referenced but not in your import.
- **Lines:** each line points from the resource that declares a reference to its target. The style tells you the type: traffic (parent, backend, selector) is solid, while configuration, policy, TLS and mirror lines have their own dash patterns and colours. The **legend** at the bottom names every type on screen.

**Traffic semantics.** Edge labels show what the traffic does:
- the path;
- weights such as `90%`;
- header, query and method matches such as `x-canary=always`;
- rule filters;
- `mirrored` backends;
- redirects such as `301 → https`;
- Ingress canaries and VirtualServer splits.

![One HTTPRoute with a 90% default, a query-matched experiment, a header-matched 10% canary and a mirrored shadow backend](../images/search-select.png)

**AI inference.** An InferencePool is drawn like a Service: it sits in the Service column as its route's backend, and **Stop at Services** collapses the model servers it selects. A note on the pool card names its **Endpoint Picker** (EPP), whose Service sits beside the pool on a dashed *endpoint-picker* line. With **Stop at Services** off, Pods fold into their Deployment ("2 pods"), and workloads that the same pools select share one **Model servers** card, so each pool draws one line instead of one per workload. The card is a grouping, not a Kubernetes resource; select it to see its members.

![Four InferencePools over shared llm-d simulator model servers, each with its own Endpoint Picker](../images/model-servers.png)

**Policy pills.** When more than two policies or filters attach to one resource, they collapse into **pills** above the card. Click the **+N** pill to see them all. Route policies sit in a lane above their route.

![Policy pills on an HTTPRoute: access, auth, observability, proxy settings, rate limit and snippets](../images/policy-pills.png)

### 5.3 Navigating

- **Zoom:** scroll over the Canvas, or use **Zoom in** and **Zoom out**. **Fit view** shows everything.
- **Opening view:** large imports open at a readable zoom, with a notice such as "Opened on *api* at a readable zoom". Select **Fit all** there to see everything.
- **Drag:** drag a card to move it, or drag a namespace tile to move the whole namespace.
- **Lock graph:** the padlock locks the Canvas, so nothing moves by accident.
- **Minimap:** bottom right.

### 5.4 Narrowing the Canvas

| Control | What it does |
| --- | --- |
| **Topology view** | **Traffic paths** focuses on traffic entries and what they reach; **All resources** shows everything. |
| **Traffic entry** | Follow one Gateway, Ingress, VirtualServer or route, such as `Gateway · api/api-gateway`. |
| **Stop at Services** | Hides Pods, workload controllers and EndpointSlices, and shows their counts on each Service (on by default). |
| **Hide Secrets** | Hides Secret cards and the lines to them. Names and types stay in the list and inspector (on by default). |
| **Focus** / **Show all** | Keeps only the selected resource's neighbourhood, and brings everything back. |
| **Namespace** (left panel) | Limits the view to one namespace. |

![One traffic entry selected: the api-gateway Gateway with its route, backends and policies](../images/traffic-entry.png)

### 5.5 Save Image

**Save Image** exports the whole topology as:
- **SVG:** vector, preferred, lossless;
- **JPG:** a high-resolution picture;
- **PDF:** a vector document.

![The Save Image menu](../images/save-image.png)

## 6. Search and narrowing down

**Search resources**, in the left panel, searches all text, or one field picked in the drop-down above it:
- **Kind**;
- **Namespace**;
- **Label** (`key` or `key=value`);
- **API group**;
- **Name**.

Suggestions come from your current import and show how many resources match.

![Search by Name: "checkout" suggests eight matching names with counts](../images/search-suggestions.png)

As you type, the resource list narrows (for example "1/86 resources"). Click a resource in the list, or pick a suggestion, to select it. The Canvas then moves as little as possible to show the resource with its relationships. Clicking a card near the edge of the Canvas brings its relationships into view in the same way.

Clear the search with **✕**.

## 7. Inspector and YAML viewer

Select any card or list entry to open the **inspector**. You can resize it, or close it with **✕**.

- **Details:**
  - **At a glance:** the key facts, such as hostnames, parent Gateways, rules, backends and status;
  - **Identity:** namespace, source document and API group;
  - **Relationships:** every link in and out, with the field that creates it, for example `spec.rules[0].backendRefs[0]`.
- **Yaml:** the original YAML, with line numbers and folding. The bulky `metadata.managedFields` and `kubectl.kubernetes.io/last-applied-configuration` are folded for you. **Copy YAML** copies it; the expand button opens a larger view.
- **Warnings:** this resource's findings, with the field, an explanation and the evidence.

![Inspector · Details for an HTTPRoute](../images/inspector-details.png)

![Inspector · Yaml: the original manifest](../images/inspector-yaml.png)

![Focus on one route, with its finding in the Warnings tab](../images/focus-and-warnings.png)

## 8. Diagnostics

**Diagnostics**, in the header, lists every finding, and its badge shows the counts (for example `7E · 1W · 9N`). Click a finding to jump to the resource.

| Severity | Meaning | Examples |
| --- | --- | --- |
| **Error** | Definite under the available evidence. | A route whose `sectionName`, `port`, kind, namespace or hostname matches no listener. A cross-namespace reference without a ReferenceGrant. A Service port that does not exist. |
| **Warning** | Likely; needs review. | A Service selector that matches no imported workload. |
| **Note** | Context, or a limitation of a partial import. | A Service that is not in the files. A hostname shared by several routes, kinds or namespaces. The IngressLink that NGINX Gateway Fabric creates for you. |

![Diagnostics for Gateway API - Broken References](../images/diagnostics.png)

After a live scan, or when you open a snapshot, **Diagnostics** also lists the **scan warnings**: resources Atlas could not read, for example because RBAC denied them. The view is then incomplete.

Static analysis does not prove controller acceptance, endpoint readiness, DNS or reachability.

## 9. Example gallery

Every bundled example is synthetic and safe to share. A few that show what the Canvas can do:

**NGF · WAF (Policy Source Type - PLM).** F5 WAF for NGINX on NGINX Gateway Fabric, with policies managed by Policy Lifecycle Management. One WAFPolicy protects the Gateway, and a stricter one overrides it on one route.

![WAF with PLM: WAFPolicies, APPolicy, APLogConf and a ReferenceGrant across namespaces](../images/example-waf-plm.png)

**NGF · AI Guardrails Integration.** An inference Gateway whose PayloadProcessors call an F5 AI Guardrails scanner, with BackendTLSPolicy and CA ConfigMap.

![AI Guardrails on NGF](../images/example-ai-guardrails.png)

**NGF · External LB Integration (BIG-IP CIS, IPAM, GatewayLink).** BIG-IP in front of NGF:
- NGF creates the IngressLink for each Gateway itself, linked through its owner reference;
- F5 IPAM assigns the addresses.

![External LB integration with BIG-IP CIS and IPAM](../images/example-external-lb.png)

**BIG-IP Next · BNK beside Envoy (data-plane comparison).** BIG-IP Next for Kubernetes and Envoy Gateway side by side, including AI-safety and LLM routing.

![BIG-IP Next beside Envoy Gateway](../images/example-bigip-next.png)

**Gateway API Inference Extension · Endpoint Picker and vLLM**, shown here in the light theme. An NGINX Gateway Fabric Gateway routes model traffic by header to two InferencePools, each with its own Endpoint Picker, in front of vLLM servers. It uses only what NGINX Gateway Fabric documents for the Inference Extension: InferencePool backends.

![Inference Extension in the light theme](../images/light-theme-inference.png)

**Gateway API Inference Extension - Shared Model Servers (llm-d simulator).** Istio and NGINX Gateway Fabric Gateways in one namespace, four InferencePools with their Endpoint Pickers, and llm-d simulator model servers they share; see the picture in [5.2](#52-reading-cards-and-lines).

## 10. Migration Playground

The Migration Playground helps you plan a move from **ingress-nginx** or **F5 NGINX Ingress Controller** Ingresses to **Gateway API / NGINX Gateway Fabric**. It is consultative: it explains first and drafts last.

Every draft is **figurative**. It can contain errors, omit behaviour, or miss the bigger picture (cost, security, traffic ownership). Review it with platform and application owners, and test it in staging. A banner reminds you, and copying or downloading a draft asks you to acknowledge this once per session.

To start:
1. Import your Ingresses (files, directory, URL or a live scan).
2. Select **Migration Playground**. It uses every `networking.k8s.io/v1` Ingress in the current import.
3. **Back to Topology** returns to the Canvas.

![The Migration Playground before discovery](../images/playground-start.png)

### The bigger picture: three Gateway patterns

Before you design anything, decide what a Gateway is *for*. In NGINX Gateway Fabric, each new Gateway gets its own data plane and Service (for example, a LoadBalancer). So the number of Gateways is a decision about boundaries and cost; it should not follow from how many Ingresses you happen to have.

![Three Gateway patterns, shared, boundary-specific and dedicated, and where Discover, Design and the legacy Draft fit](../images/gateway-patterns.svg)

| Pattern | Use it when | Watch out for |
| --- | --- | --- |
| **A · Shared Gateway** | Most apps share exposure, TLS and security posture, and teams own their own routes. | A shared blast radius. Govern hostname ownership, quotas and policy conflicts. |
| **B · Boundary-specific** | You need to separate public and internal traffic, prod and non-prod, or compliance zones. | More listeners, addresses and operations, and teams still share each Gateway. |
| **C · Dedicated Gateway** | A tenant or critical app needs its own TLS, load balancer, release cadence or isolation. | Higher cost and sprawl. Verify real data-plane isolation, not just separate objects. |

**Shared** and **Dedicated** are the two extremes, and most platforms land on A or B. Keep C for the tenants that truly need it. The rule of thumb: **add a Gateway only for a meaningful trust, exposure or operational boundary.**

How the Playground fits:

- **1 · Discover** shows how hosts, snippets and TLS are shared today. That tells you where the real boundaries are before you pick a pattern.
- **2 · Design** lets you build any of the three:
  - one Gateway for every host is pattern A;
  - a public and an internal Gateway is pattern B (the example in [10.2](#102-design));
  - a Gateway per team is pattern C.

  Its checks focus on the **ownership model** in the middle: who owns each host, route namespace, listener and certificate. It also shows the load balancers and decisions that each extra Gateway brings.
- **Draft (legacy per-Ingress)** maps every Ingress to its own Gateway. That is a lift-and-shift past the dedicated extreme: 40 Ingresses would become 40 data planes and load balancers, where one controller served them before. It is useful for exploring annotation mappings, but it is rarely the right design.

### 10.1 Discover

Select **Run Discovery**. **Download Report** saves everything below as one report.

**Summary.** The summary counts:
- Ingresses and namespaces;
- hosts, and how many are shared;
- **unique annotations**;
- ingress classes;
- findings by severity (Critical, High, Medium, Info).

Many Ingresses normally share one controller's data plane and load balancer. In NGINX Gateway Fabric, **each new Gateway gets its own data plane and Service (for example, a LoadBalancer)**, so the number of Gateways, not Ingresses, sets the number of load balancers.

![Discover summary and Ingress table](../images/discover-summary.png)

**Ingresses table.** There is one row per Ingress, with:
- its tier (Simple / Moderate / Advanced);
- hosts, rules and paths;
- annotations;
- snippets, CORS, regex, rewrite and TLS;
- which Ingresses it shares a host with.

Sort by any column, filter by text, or use the badges: **Shared host**, **Snippet**, **CORS**, **Regex**, **Rewrite**, **TLS** and **Backend TLS**.

Expand a row to see its settings and its parsed snippet: the headers it adds, with their lengths, the headers it hides, and any conditional logic.

![An expanded Ingress: body size, CORS and the parsed server-snippet](../images/discover-ingress-row.png)

Open **Annotations** in an expanded row. Each annotation gets:
- an outcome, such as *close equivalent* or *not mapped — decide manually*;
- a note.

Where Atlas knows the target, a chip such as **HTTPRoute** `spec.rules.filters.cors` opens that exact field in the CRD explorer.

![Annotations with their outcomes and NGF target fields](../images/discover-annotations.png)

**Hosts table.** It shows how ingress-nginx merges Ingresses that share a host today:
- the contributors, in precedence order (oldest first);
- distinct paths, checked against the 16-rule HTTPRoute limit;
- which `server-snippet` is actually in effect;
- host-wide regex and duplicate paths.

**Findings.** Findings are grouped by severity. Each one says:
- what happens **today**;
- what **Gateway API / NGF** offers;
- what you need to **decide**, with the resources and evidence.

Click a resource name to jump to its row.

![Hosts and the first findings](../images/discover-hosts.png)

**Research more** links to the F5 NGINX ingress-nginx migration guide. External links always ask before opening a new tab.

### 10.2 Design

Open **2 · Design**. Atlas does **not** preselect a topology; any of the [three patterns](#the-bigger-picture-three-gateway-patterns) can be built here.

1. **Add Gateway** for each Gateway you intend to run. Give it:
   - a **name** and **namespace**. `default` is fine, but a dedicated namespace such as `gateway-infra` keeps Gateways, RBAC and certificates apart from applications;
   - optionally, a **purpose** (for example *external* or *internal*);
   - the **GatewayClass** of your NGF installation.
2. Choose **Listener hostnames** for each Gateway:
   - **Per host:** an exact listener per host. This is strict, but you must edit the Gateway for every new host, and each host counts towards the 64-listener limit.
   - **Wildcard:** listeners such as `*.example.com`. They cover deeper subdomains but not the apex. Atlas suggests wildcards after a check, and hosts outside them get exact listeners.
   - **No hostname (on HTTPRoutes):** hostnames live only on the HTTPRoutes. This is the simplest option, but any namespace the Gateway admits can claim any hostname.
3. **Attach hosts:** tick the Gateways each host uses.
   - A host may sit on several Gateways, for example internal and external, or old and new during a cutover.
   - **Suggest by namespace** proposes a Gateway for each host, and **Apply N suggestions** accepts them.
   - **attach all** puts every host on one Gateway.
   - The **Route namespace** column decides where each host's routes live.
   - Expand a host to move an individual Ingress elsewhere.
4. Select **Check Design**.

![Two Gateways, public-edge and internal-tools, with hosts attached](../images/design-gateways.png)

This example, a `public-edge` and an `internal-tools` Gateway, is pattern **B · Boundary-specific**.

The **design check** shows:
- the Gateway and load-balancer count, compared with today;
- hosts attached, HTTPRoutes after splits, listeners and ReferenceGrants;
- a card per Gateway, with its listeners, `allowedRoutes` and routes;
- design findings, for example routes that need splitting, hostname ownership or cross-namespace admission.

![Design check: 2 Gateways, 9/9 hosts attached, 13 HTTPRoutes after splits, ready](../images/design-check.png)

Design produces four outcomes, all included in **Download Report**:

- **Design record:** **Export design (JSON)** and **Import design** let you keep the design in a pull request or ticket, and reload it later.
- **Decisions to sign off:** what Atlas cannot decide for you, each with a suggested owner. Examples are route splits, hostname ownership, namespace admission labels and the load-balancer count.

  ![Decisions to sign off, with owners](../images/design-decisions.png)

- **Platform skeleton:** figurative YAML for the Gateways and any ReferenceGrants that the design fully determines. Everything else is commented, in order: prerequisites, load-balancer and NginxProxy settings, certificates, namespace labels, draft HTTPRoutes and verification steps. Tick the acknowledgement, then **Download platform skeleton**, **Copy skeleton** or **Open full screen**. Replace every `CHANGE-ME` before uncommenting.

  ![The platform skeleton, with its summary and prerequisites](../images/design-skeleton.png)

- **Architecture diagram:** draws the design down to Services, with GatewayClass and certificates shown as pills on the Gateway cards.
  - Scroll over the diagram to zoom, or scroll above it to move the page.
  - **Open full screen** gives it the whole window; **Esc** returns.

  ![The architecture diagram in full screen](../images/design-diagram.png)

The sticky bar keeps **Check Design**, **Download Report** and the **Jump to** links (Top, Check, Decisions, Skeleton, Diagram) in reach.

### 10.3 Draft (legacy per-Ingress)

The legacy converter maps each Ingress to its own Gateway and HTTPRoutes, without a topology decision. This is the lift-and-shift extreme described in [the bigger picture](#the-bigger-picture-three-gateway-patterns): one data plane and load balancer per Ingress. Use it to explore mappings, not as a migration design. It understands both community ingress-nginx and **F5 NGINX Ingress Controller** annotations.

1. **Setup.**
   - **Source snapshot:** the Ingresses in the current import.
   - **Target:**
     - the profile: **NGF v2.8.0 · reviewed policy APIs**, or **Gateway API only · no NGF policies**;
     - the data plane: **NGINX Open Source** or **NGINX Plus**;
     - the **GatewayClass** name.
2. Select **Assess Migration**, or **Assess & Generate Draft**. Generation always includes the assessment.
3. **Migration preflight:**
   - the detected source controller, with its evidence;
   - Ingress and annotation counts;
   - how many Gateways the draft would create;
   - routes over 16 rules.

![Draft setup and preflight for an F5 NGINX Ingress Controller snapshot](../images/draft-setup.png)

**Annotations.** There is one row per annotation key, with:
- the provider (for example **F5 NIC**);
- the NGF target, with field chips that open the CRD explorer;
- the mapping: **exact**, **approximate** or **unsupported**;
- the edition it needs (for example *NGINX Plus + F5 WAF*);
- the draft result and guidance.

Filter by **Outcome** and **Mapping**, or find a key.
- **Exact** mappings are generated into the draft.
- **Approximate** ones become commented suggestions.
- **Unsupported** ones are reported; nothing is silently dropped.

![Annotation mappings for F5 NGINX Ingress Controller, with NGF target fields and editions](../images/draft-annotations.png)

**Files.** The source manifests sit beside the generated output, at equal height. The output has three tabs:
- **Draft YAML;**
- **Suggestions:** commented CHANGE-ME skeletons;
- **Review notes.**

After the acknowledgement you can **Copy Generated YAML**, **Download Draft YAML**, **Download Suggestions** or **Download Review Report**. **Decisions before cutover** lists what still needs an owner.

![Before: the source Ingress and its annotations. After: the drafted Gateway and HTTPRoute](../images/draft-files.png)

**Compare topology** draws the source snapshot and the proposed draft side by side, on two Canvases. Each opens on its whole topology, and **Open full screen** on either tile shows that side alone, with a switch to the other. A connected graph does not prove equivalent behaviour or a healthy deployment.

![Before and after topology](../images/draft-compare.png)

A topology-aware generator, which drafts from your Design record, is planned.

## 11. Explore Supported CRDs

Open **Help → Explore Supported CRDs**, or select the floating **CRDs** button at the bottom right. A window opens and stays open while you work:
- move it by its title bar;
- resize it from the corner;
- minimise it;
- close it with **✕**.

1. **CRD:** choose a CRD, grouped by product and pinned release. CRDs in your current import appear first. The catalog includes:
   - Gateway API Standard and Experimental;
   - NGINX Gateway Fabric 2.8.0;
   - NGINX Ingress Controller 5.7.0;
   - F5 WAF for NGINX, including Policy Lifecycle Management;
   - the Inference Extension (InferencePool);
   - F5 CIS and F5 IPAM;
   - common prerequisites such as cert-manager.
2. **Version:** the storage version is preselected; deprecated versions are marked.
3. **Explain:** shows the full field tree, like `kubectl explain --recursive`.

Each field shows its type, whether it is **required**, enum values, defaults and validation **rules**. Fields that exclude each other share a colour mark:
- **1:** exactly one;
- **≤1:** at most one;
- **≥1:** at least one.

![The CRD explorer: WAFPolicy, with policySource alternatives marked "1 of 3"](../images/crd-explorer.png)

Filter by name, path or description, or use **Expand 2 levels**, **Expand all** and **Collapse all**. Select a field to see its full description, its constraints and a copyable `kubectl explain` command. The **source** link shows where the schema came from.

The catalog schemas are bundled with Atlas, so this works offline. F5 BIG-IP Cloud-Native Edition 2.4 kinds appear as **reference cards** (identity, purpose and relationships), because their schemas cannot be redistributed.

## 12. Help, themes and recent sources

- **Help:**
  - **GitHub repository**;
  - **Explore Supported CRDs**;
  - **About**, which shows the release and the CRD and API versions Atlas knows.
- **Theme:** the sun/moon button switches between light and dark.
- **Recent sources:** reloads an example, file set or URL you used in this browser. Imports over 2 MiB, such as large snapshots, are not kept; import the file again.

![The Help menu](../images/help-menu.png)

## 13. Command line

The same binary has a command-line interface. With the container, mount your files read-only:

```bash
docker run --rm -v "$PWD:/work:ro" ghcr.io/tareqaminul/gateway-atlas:latest validate /work/examples/missing-service.yaml
```

| Command | Purpose |
| --- | --- |
| `analyze <files>` | Resources, relationships and findings (`--json` for JSON). |
| `validate <files>` | Findings only; exits non-zero on errors (useful in CI). |
| `explain <files>` | Static traffic paths, with field provenance. |
| `graph <files>` | The relationship graph (`--format json`). |
| `assess <files>` | Migration preflight assessment. |
| `migrate <files>` | Legacy per-Ingress draft into `--output` (default `generated`). |
| `cluster scan --context NAME [--namespace NAME]` | Read-only scan from a kubeconfig (`--kubeconfig`, `--json`). |
| `serve [--addr ADDR] [--enable-live]` | The web UI. |

## 14. Troubleshooting

| Symptom | What to do |
| --- | --- |
| The page does not load | Check that the container is running, and that the port in the URL matches the first number in `-p`. |
| **Kubernetes cluster** is unavailable | Restart with `--enable-live` ([section 4](#4-connect-to-a-live-cluster-read-only)). Then mount a kubeconfig, or use **Load kubeconfig** in the dialog. |
| The status bar counts scan warnings | Some resources could not be read, usually because of RBAC. **Diagnostics** lists them; grant `get, list, watch` on them, or accept a partial view. |
| A snapshot will not import | Imports are limited to 32 MiB. Scan one namespace at a time for very large clusters. |
| No contexts, or a kubeconfig error | The mounted folder must contain a regular, readable file named `config` at `/home/nonroot/.kube`. Recreate the container after changing the mount. |
| "The cluster could not be reached" | Check network access and credentials. On Docker Desktop, point the copy at `kubernetes.docker.internal:6443` ([section 4.1](#41-prepare-a-kubeconfig-for-the-container)). On a provisioner whose API name comes from `/etc/hosts` or a local DNS server, use `--network host` ([section 4.4](#44-on-an-openshift-provisioner-docker-on-rhel)). |
| Cloud cluster authentication fails | `exec` credential plugins are not in the image. Use a static token or certificate for a read-only service account. |
| A relationship line is missing | Check that the source really contains an explicit reference with the exact group, kind, namespace and name. Atlas does not invent links. |
| A card is dashed | It is referenced but not in your import. Import the target too, or check its exact API group, version, kind, namespace and name. |
| The Canvas looks crowded | Use **Stop at Services**, **Traffic paths**, one **Traffic entry**, one namespace, **Search** or **Focus**, then **Fit view**. |
| An external link did nothing | Atlas asks before opening a new tab. If your browser blocks new tabs, use **Copy link**. |

## 15. Privacy and safety

Atlas runs locally and sends your manifests nowhere.

Secret values (`data`, `stringData` and the copy kubectl keeps in an annotation) are removed from every import: live scans, files, pasted YAML and URLs.

**Snapshots are confidential.** They hold what Atlas read from a cluster, including ConfigMap data, environment values, annotations, addresses and hostnames. Store them like other cluster exports.

Before you share screenshots, reports, design records, snapshots or exported YAML, review them for:
- private names and hosts;
- IP addresses;
- annotations;
- internal URLs.

**Never** include kubeconfig credentials, tokens, certificates or Secret values in issues or reports. Reduce problems to small synthetic examples, like the ones in [examples](../examples/).
