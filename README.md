# Gateway Atlas

**Gateway Atlas maps Kubernetes traffic.** Load your manifests, or read a cluster read-only, and see how every Gateway, route, policy, certificate and Service relates to the others. It flags what looks wrong and helps you plan the move from ingress-nginx to Gateway API and NGINX Gateway Fabric.

This repository is for **users and reviewers**. It has the user guide, an architecture overview and example manifests, and it is where you report issues and feedback. Gateway Atlas is in an open **preview**: the current release is **2.2.2**.

## Why Gateway Atlas

Kubernetes networking configuration is spread across many objects and API groups. Each one is easy to read; how they connect is not. Typical questions are hard to answer from YAML alone:

- Which Gateway, listener and route serve this hostname, and which Service do they reach?
- Which policy, filter, certificate or iRule applies, and from which namespace?
- What is broken or ambiguous: a missing Service, a cross-namespace reference without a ReferenceGrant, a route with no valid parent?
- **Migration:** the community `kubernetes/ingress-nginx` controller was retired in March 2026. What does moving these Ingresses to Gateway API involve, and how many Gateways, and therefore load balancers, should there be?

Gateway Atlas answers these by building one relationship graph from your resources, showing it on an interactive Canvas, and explaining its findings in plain language.

## What it does

- **Explore:** a left-to-right Canvas of classes, Gateways, routes, policies, Services and workloads, grouped by namespace and controller. Select anything to see its facts, relationships, findings and YAML.
- **Diagnose:** errors, warnings and notes with the exact field and evidence, for example unresolved references, unauthorized cross-namespace references, bad ports or unmatched selectors.
- **Migration Playground** (ingress-nginx → Gateway API / NGF):
  - **Discover** shows what your Ingresses really do: hosts shared between Ingresses, snippets, CORS, regex, rewrites and limits.
  - **Design** lets you choose the Gateway topology and checks its load-balancer footprint, listeners, namespaces and certificates.
  - It ends with a design record, a decision list and a figurative platform skeleton.
- **Explore Supported CRDs:** a floating `kubectl explain`-style tree of 41 bundled CRD schemas (Gateway API, NGF, NGINX Ingress Controller, F5 CIS and more), available offline.
- **Covers:** Kubernetes and Gateway API, NGINX Gateway Fabric, NGINX Ingress Controller and App Protect, F5 BIG-IP Container Ingress Services and IPAM, BIG-IP Next for Kubernetes, and the Gateway API Inference Extension.

## Design principles

- **Local-first:** it runs on your machine; your manifests are not sent to any service.
- **Read-only:** live cluster access is opt-in and only lists resources. Atlas has no create, update, delete, apply or exec code.
- **No Secret data:** Kubernetes Secret `data` and `stringData` are removed before analysis; only names and types are shown.
- **Honest about limits:** static analysis is labelled as static, and approximate or unsupported migration behaviour is reported, never silently dropped. Migration drafts are figurative and must be reviewed.

## Quick start

You need Docker (Docker Desktop on macOS or Docker Engine on Linux). One image runs on Apple Silicon (`linux/arm64`) and x86-64 Linux (`linux/amd64`).

```bash
docker pull ghcr.io/tareqaminul/gateway-atlas-rpi-1:2.2.2
```

```bash
docker run --rm -p 127.0.0.1:8080:8080 ghcr.io/tareqaminul/gateway-atlas-rpi-1:2.2.2 serve --addr 0.0.0.0:8080
```

Open <http://127.0.0.1:8080>, choose **Import → Bundled example**, and explore. Then follow the [user guide](docs/user-guide.md).

## Documentation

- [User guide](docs/user-guide.md): running Atlas, imports, live clusters, the Canvas, diagnostics, the Migration Playground, the CRD explorer and troubleshooting.
- [Architecture](docs/architecture.md): how Atlas is built and where its trust boundaries are.
- [Reviewer checklist](docs/reviewer-checklist.md): a guided tour for testers.
- [Examples](examples/): synthetic manifests to try (all names use `example.test`).

## Feedback and issues

Please open an issue in this repository using the **Bug report** or **Feedback** form. Include the Atlas release (**Help → About**) and your browser and platform.

**Never paste kubeconfig credentials, tokens, certificates, Secret values or private manifests into an issue.** Reduce a problem to a small synthetic example instead. To report a security problem privately, see [SECURITY.md](SECURITY.md).

## License

Apache License 2.0. See [LICENSE](LICENSE).
