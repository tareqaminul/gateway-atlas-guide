# Reviewer checklist

A guided tour for people testing Gateway Atlas. It takes about 30–45 minutes. Note anything confusing, wrong or slow, and open an issue with the **Feedback** or **Bug report** form.

Before you start, note the release shown in **Help → About**, and your browser, OS and screen size.

## 1. Start and orient (5 min)

- [ ] Start Atlas ([user guide §1](user-guide.md#1-run-gateway-atlas)) and open the UI.
- [ ] Load **Import → Bundled example → Minimal Gateway**. Is the left-to-right reading order clear?
- [ ] Switch light/dark theme. Is everything readable in both?

## 2. Canvas (10 min)

- [ ] Load **Playground · one controller, many teams**. Try **Families** and **Classic**. Which one explains the relationships faster?
- [ ] Select a Gateway. Are its relationships and the inspector facts what you expected?
- [ ] Use zoom, **Fit view**, the minimap, the padlock, **Stop at Services**, **Traffic paths / All resources** and **Focus**.
- [ ] **Save Image** as SVG, JPG and PDF. Do the files open, and do they look like the Canvas?
- [ ] Open the **Import** and **Save Image** menus and click elsewhere. Do they close?

## 3. Diagnostics and search (5 min)

- [ ] Load **Missing Service** and **Cross-namespace missing grant**. Are the findings understandable? Does clicking one take you to the resource?
- [ ] Search by Kind, Namespace and Label.

## 4. Migration Playground (15 min)

- [ ] Load **Playground · shared hosts & snippets** and open **Migration Playground → Run Discovery**.
- [ ] Are the summary and the **Ingresses** and **Hosts** tables useful at a glance? Is anything important missing?
- [ ] Expand **shop-web/storefront**. Do the annotation notes and the target-field chips make sense? Does a chip open the right field in the CRD explorer?
- [ ] Read three findings. Are *Today / Gateway API / Decide* clear and actionable?
- [ ] In **2 · Design**, add one Gateway, choose **Listener hostnames**, attach all hosts, and select **Check Design**. Then try two Gateways (internal/external) and an Ingress override.
- [ ] Are the decisions, the platform skeleton and the architecture diagram (including **Open full screen**) understandable?
- [ ] **Download Report** and **Export design (JSON)**, then **Import design** again.
- [ ] Is the "drafts are figurative" message clear without getting in the way?

## 5. CRD explorer (5 min)

- [ ] **Help → Explore Supported CRDs**: explain **HTTPRoute** (standard) and **NginxProxy**. Use the filter, expand and collapse, and select a field.
- [ ] Move, resize and minimise the window while using the page behind it.

## 6. Optional: your own data

Only use data you are allowed to use, and keep it on your machine.

- [ ] Import your own Ingress or Gateway manifests, or scan a test cluster read-only ([user guide §4](user-guide.md#4-connect-to-a-live-cluster-read-only)).
- [ ] What did Atlas get right? What did it miss or get wrong? Describe it with a small synthetic example, never the original manifests.

## Wrap-up

- What was the most useful part? What was the most confusing?
- Would you use it for a real migration review? What would stop you?
