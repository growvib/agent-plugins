# Contributing

Thanks for helping improve the GrowVib plugins. This repository is a Claude Code plugin
marketplace: `.claude-plugin/marketplace.json` lists the plugins, and each plugin lives under
`plugins/<name>/`.

## Reporting problems

- **A bug in the plugin** (it fails to install, the skill gives wrong guidance, a tool is
  missing): open an issue with the bug report template.
- **A problem with an order or a payment**: contact GrowVib support at support@growvib.com. The
  plugin only connects Claude to GrowVib; it cannot see or change orders.
- **A security issue**: follow [SECURITY.md](SECURITY.md). Never in a public issue.

## Making a change

1. Fork the repository and create a branch.
2. Make the change. Keep a plugin's skill accurate to what its MCP tools actually do: if you
   describe a tool, an argument or a status, check it against the live server card at
   https://api.growvib.com/.well-known/mcp/server-card.json.
3. Validate both manifests in strict mode:

   ```bash
   claude plugin validate --strict .
   claude plugin validate --strict ./plugins/growvib
   ```

4. Install your branch locally and check the MCP server connects:

   ```bash
   claude plugin marketplace add ./
   claude plugin install growvib@growvib
   claude mcp list
   ```

5. Bump `version` in `plugins/<name>/.claude-plugin/plugin.json` when the plugin's behavior
   changes, so installed copies pick up the update.
6. Open a pull request. Every change to `main` goes through a reviewed pull request and is
   squash-merged.

## Rules

- No secrets, keys, tokens or personal data in any file.
- Plugins point only at GrowVib's documented public endpoints.
- Write plainly. Copy that users read must not promise anything the service does not do.
- By contributing you agree your work is released under the [MIT License](LICENSE) and that you
  follow the [Code of Conduct](CODE_OF_CONDUCT.md).
