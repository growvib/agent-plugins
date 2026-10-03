# Security policy

## Reporting a vulnerability

Please report security issues privately. Do not open a public issue.

Use GitHub's private reporting form:
https://github.com/GrowVib/agent-plugins/security/advisories/new

This covers the plugin in this repository (its manifests, its skill and the MCP configuration it
installs) and the GrowVib MCP endpoints it connects to. Include what you found, how to reproduce
it, and what an attacker could do with it. Reports are reviewed by the GrowVib team.

## Scope notes

- The plugin holds no secrets and no keys. It points Claude Code at the public endpoint
  `https://api.growvib.com/mcp-public`, which needs no credentials.
- Payments are signed in the payer's own wallet, never by the plugin or the model. A report that
  an agent could move money without the payer signing is in scope and urgent.
- Problems with an order or a payment that are not security issues go to GrowVib support at
  support@growvib.com, not to this form.
