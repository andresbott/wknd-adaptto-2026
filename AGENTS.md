# AGENTS.md

Agent-facing notes for the `wknd-adaptto-2026` AEM project (adaptTo() 2026
security demo).

## What this repo is

A full-stack AEM as a Cloud Service Sites project generated from the AEM Project
Archetype (`appId=wknd`, `appTitle=WKND Site`). Standard Cloud Manager module
layout: `all`, `core`, `ui.apps`, `ui.apps.structure`, `ui.config`,
`ui.content`, `ui.frontend`, `dispatcher`, plus `it.tests` / `ui.tests`.

## Deployment target (AEM as a Cloud Service)

This code is deployed via Cloud Manager to:

| Field          | Value                                            |
|----------------|--------------------------------------------------|
| Program        | The tiger and the crane                          |
| Environment    | bott-adaptto-2026                                |
| Tier           | DEV                                              |
| Program ID     | p15854                                           |
| Environment ID | e2240987                                         |
| Author URL     | https://author-p15854-e2240987.adobeaemcloud.com |
| Solutions      | aemassets, aemsites                              |

To pull live details for this environment, use the **aem** MCP server:
`list-aem-environments` → match `programTitle: "The tiger and the crane"` /
`environmentTitle: "bott-adaptto-2026"` → then use its `authorUrl` with
`list-aem-capabilities` / `lookup-api-spec` / `read-api`.

## Build & local validation

Requires **Java 21** and **Maven 3.9.4+**.

- Full build + Cloud Manager validation gate: `mvn clean install`
  (runs `aemanalyser-maven-plugin` and the FileVault package validators — this
  is what must be green before shipping).
- Classic AEM variant (also run in CI): `mvn clean install -Pclassic`
- Install to a local author instance: `mvn clean install -PautoInstallPackage`

## Shipping

Ship changes with the `git-flows:ship` skill (open PR → wait for green CI →
squash-merge with a Conventional Commits subject → fast-forward `main` → offer a
release tag). Never commit directly on `main`.

## Skills

- `fix-security` (`.claude/skills/fix-security/`) — end-to-end flow to identify
  the deployment program, pull environment details from the **aem** MCP server,
  apply the security fix, validate locally, and ship.
