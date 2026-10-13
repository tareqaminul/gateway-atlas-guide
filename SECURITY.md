# Security policy

Gateway Atlas is designed to be read-only, local-first and Secret-redacting. If you find a way to make it change a cluster, leak Secret data or credentials, reach unexpected network destinations, or expose the UI beyond what you configured, please report it privately.

- Use **GitHub private vulnerability reporting** for this repository (**Security → Report a vulnerability**), if enabled. Otherwise, open a minimal public issue asking for a private contact, without details.
- Include the release (**Help → About**), your platform and the steps to reproduce, using synthetic data only.
- Do not include real kubeconfigs, tokens, certificates, Secret values or private manifests.

Please allow time for a fix before public disclosure.
