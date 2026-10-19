# Gateway Atlas quick start

Get Gateway Atlas running, explore a bundled example, and then bring your own YAML or a live cluster. It takes about five minutes. For every feature in detail, see the [user guide](user-guide.md).

- [1. Before you start](#1-before-you-start)
- [2. Start Atlas](#2-start-atlas)
- [3. A five-minute tour](#3-a-five-minute-tour)
- [4. Optional: connect a live cluster (read-only)](#4-optional-connect-a-live-cluster-read-only)
- [5. Bring your own YAML](#5-bring-your-own-yaml)
- [6. Which example should I open?](#6-which-example-should-i-open)
- [7. Stop and next steps](#7-stop-and-next-steps)

## 1. Before you start

You need **Docker**:
- **macOS (Apple Silicon):** Docker Desktop.
- **Linux (x86-64):** Docker Engine.

The image is multi-architecture, so Docker picks the right variant for you.

You do **not** need a cluster, an account or internet access after the image is pulled. Atlas runs on your machine and sends your manifests nowhere.

## 2. Start Atlas

```bash
docker run --rm -p 127.0.0.1:8080:8080 ghcr.io/tareqaminul/gateway-atlas:2.3.0 serve --addr 0.0.0.0:8080
```

Open <http://127.0.0.1:8080>.

![The Gateway Atlas start page](../images/welcome.png)

> **Keep `127.0.0.1` in the `-p` option.** Atlas has no login, so publish it on your own machine only. To use another port, change the first number, for example `-p 127.0.0.1:9090:8080`, and open port 9090.

`ghcr.io/tareqaminul/gateway-atlas:latest` always points at the newest release. Use a version tag such as `2.3.0` when you want a fixed, repeatable setup.

## 3. A five-minute tour

### 3.1 Load an example

Open **Import** and choose a **Bundled example**. The examples are grouped as:
- **Scenarios:** realistic mixed setups;
- **Learning cases:** one idea each;
- **Migration Playground:** Ingress sets to migrate;
- **BIG-IP Next.**

Start with **Migration In-Progress**.

![The Import menu: bundled examples, files, a directory, a URL or a cluster](../images/import-menu.png)

The **Canvas** reads left to right:

class → traffic entry (Gateway, Ingress, VirtualServer) → routes → Services

Namespaces are drawn as tiles. Edge labels tell you what the traffic does, for example `/api · 50%`.

![A migration in progress: the new Gateway API route splits /api 50/50 while the legacy Ingress and VirtualServer still serve the same hosts](../images/canvas-hero.png)

### 3.2 Select a card

Click any card to open the **inspector** on the right. It has three tabs:
- **Details:** the key facts at a glance, the resource's identity and its relationships;
- **Yaml:** the original manifest, with folding and **Copy YAML**;
- **Warnings:** the findings for this resource.

Select **Focus** to keep only the selected resource's neighbourhood on the Canvas, and **Show all** to return.

![An HTTPRoute selected, with its relationships and facts in the inspector](../images/inspector-details.png)

### 3.3 Search and narrow down

Type in **Search resources**, on the left. You can search all fields, or pick one field first: **Kind**, **Namespace**, **Label**, **API group** or **Name**. Suggestions come from your import, with counts.

The resource list narrows as you type. Click a resource to select it, and the Canvas moves just enough to show it with its relationships.

![Searching by name: suggestions with counts from the current import](../images/search-suggestions.png)

You can also narrow the Canvas from the top bar:
- **Traffic paths** or **All resources**;
- a single **traffic entry**, such as one Gateway or Ingress;
- **Stop at Services**, which hides Pods and workloads behind each Service;
- **Hide Secrets**.

### 3.4 Check diagnostics

Load **Gateway API - Broken References**, then open **Diagnostics** in the header. Each finding names the resource, the exact field and the evidence. Click a finding to jump to its resource.

![Diagnostics: routes that no listener accepts, a missing ReferenceGrant and a wrong Service port](../images/diagnostics.png)

### 3.5 Try the Migration Playground

Load **Playground · retail platform (real-world case)**, select **Migration Playground**, then **Run Discovery**. You get:
- a summary of the Ingresses;
- an Ingress table with tiers and badges;
- how shared hosts merge today;
- findings that say what happens **today**, what **Gateway API / NGF** offers, and what you need to **decide**.

Not sure how many Gateways you need? Read [three Gateway patterns](user-guide.md#the-bigger-picture-three-gateway-patterns) first.

Then open **2 · Design**:
1. **Add Gateway** for each Gateway you want.
2. Attach hosts to the Gateways.
3. Select **Check Design**.

![Discover: 11 Ingresses across 8 namespaces, with findings by severity](../images/discover-summary.png)

### 3.6 Explore a CRD

Select **CRDs**, bottom right, or open **Help → Explore Supported CRDs**. Pick a CRD, such as **WAFPolicy · gateway.nginx.org**, and select **Explain**. It works offline.

![The CRD explorer: the WAFPolicy field tree with "one of" marks](../images/crd-explorer.png)

## 4. Optional: connect a live cluster (read-only)

Atlas reads a cluster through a kubeconfig on your machine. It never reads it from the browser. Live access is **off** unless you start Atlas with `--enable-live`. Atlas only **lists** resources; it has no code to change a cluster.

1. Make a dedicated copy of your kubeconfig for Atlas:

   ```bash
   mkdir -p "$HOME/.config/gateway-atlas" && chmod 700 "$HOME/.config/gateway-atlas"
   ```

   ```bash
   kubectl config view --raw --flatten > "$HOME/.config/gateway-atlas/config" && chmod 600 "$HOME/.config/gateway-atlas/config"
   ```

   Treat this file as a credential: never commit or share it.

2. **Docker Desktop Kubernetes only.** Its kubeconfig points at `127.0.0.1`, which inside a container means the container itself. Point your copy at the Docker host instead:

   ```bash
   kubectl --kubeconfig "$HOME/.config/gateway-atlas/config" config set-cluster docker-desktop --server=https://kubernetes.docker.internal:6443
   ```

3. Start Atlas with the folder mounted read-only:

   ```bash
   docker run --rm -p 127.0.0.1:8080:8080 --mount type=bind,source="$HOME/.config/gateway-atlas",target=/home/nonroot/.kube,readonly ghcr.io/tareqaminul/gateway-atlas:2.3.0 serve --addr 0.0.0.0:8080 --enable-live
   ```

4. Choose **Import → Kubernetes cluster**, pick a **Context** and optionally a **Namespace**, and select **Scan read-only**.

![Import from cluster: choose a context and a namespace, then scan read-only](../images/live-cluster.png)

Some cloud logins (AKS, EKS, GKE) use a credential plugin that is not in the image. For those, use a kubeconfig with a static token or certificate for a read-only service account. See the [user guide](user-guide.md#4-connect-to-a-live-cluster-read-only).

## 5. Bring your own YAML

| Option | How |
| --- | --- |
| **Paste YAML** | Expand **Paste YAML** at the bottom of the left panel, paste one or more documents, and select **Analyze**. |
| **YAML files** | **Import → YAML files**, and select one or more `.yaml` or `.yml` files. |
| **Local directory** | **Import → Local directory**, for example a Helm-rendered or GitOps folder. |
| **Web / GitHub URL** | **Import → Web / GitHub URL**. Enter an HTTPS YAML document, or a public GitHub folder in the form `https://github.com/<owner>/<repo>/tree/<commit>/<path>`. |

![Pasted YAML analyzed: an Ingress with a regex path, and a backend Service that is not in the paste shown dashed](../images/paste-yaml.png)

Imports are analyzed, never modified. Partial imports are normal. For example, a Service that is missing from an Ingress-only export is shown dashed and reported as a note, not as an error.

Try the synthetic files in this repository's [examples](../examples/) folder.

## 6. Which example should I open?

| To see… | Open |
| --- | --- |
| The basics | **Minimal Gateway** |
| Every Gateway API route type, weights, mirrors and redirects | **Gateway API - Various Route Types** |
| What broken configuration looks like | **Gateway API - Broken References** |
| Old and new controllers serving the same hosts | **Migration In-Progress** |
| NGINX Ingress Controller Ingresses and CRDs | **NGINX Ingress Controller with Ingresses** / **with CRDs** |
| NGF policies as pills | **NGF · rate, body, proxy, upstream, access & tracing** |
| F5 WAF for NGINX with Policy Lifecycle Management | **NGF · WAF (Policy Source Type - PLM)** |
| AI traffic | **Gateway API Inference Extension · Endpoint Picker and vLLM** and **NGF · AI Guardrails Integration** |
| BIG-IP in front of NGF | **NGF · External LB Integration (BIG-IP CIS, IPAM, GatewayLink)** |
| A migration to plan | **Playground · retail platform (real-world case)** or any other **Playground ·** example |
| BIG-IP Next for Kubernetes | **BIG-IP Next · BNK beside Envoy (data-plane comparison)** |

## 7. Stop and next steps

Stop Atlas with `Ctrl+C` in its terminal. **Recent sources**, in the header, reloads anything you imported in this browser.

Next, read the [user guide](user-guide.md) for every feature.

Found something wrong or confusing? Open an issue with the **Bug report** or **Feedback** form. Never include credentials, Secret values or private manifests.
