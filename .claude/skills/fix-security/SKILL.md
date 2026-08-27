---
name: fix-security
description: Use when asked to fix a security issue or vulnerability in this AEM project (e.g. "fix security", a vulnerable/flagged dependency, an embedded vulnerable jar, or a failed security scan). Identifies the deployed AEM program from AGENTS.md, pulls environment details from the aem MCP server, applies the fix (dispatching specialist agents when domain judgement is needed), runs mvn validation locally, and ships via git-flows:ship once green.
---

# Fix Security

End-to-end workflow to remediate **all** reported security findings in this AEM
project and ship the fix automatically. Follow the steps in order and create a
todo per step. Do not skip ahead — each step gates the next. This workflow is
autonomous: it never asks which findings to fix or whether to ship.

## When to use

The user asks to fix a security issue, vulnerability, failed security scan, or a
flagged/vulnerable dependency in this repo (e.g. "fix security").

## Prerequisites

- **Java 21** and **Maven 3.9.4+** on PATH (see `README.md` /
  `.cloudmanager/java-version`).
- The **aem** MCP server is connected — the user authenticates it via `/mcp`.
  Its tools are *deferred*: load a tool's schema with
  `ToolSearch("select:<tool-name>")` **before** calling it, or the call fails
  with an input-validation error.

## Step 1 — Identify the deployment program & environment

1. Read `AGENTS.md` at the repo root. Use its **Deployment target** section as
   the source of truth for Program / Environment / Program ID / Environment ID /
   Author URL.
2. Fallback, only if `AGENTS.md` is missing or has no deployment target:
   - `ToolSearch("select:mcp__aem__list-aem-environments")`, then call
     `mcp__aem__list-aem-environments`.
   - Match this repo to an environment using the app title
     (`archetype.properties` `appTitle`), the repo/dir name, and the program
     title.
   - **Confirm the match with the user** before proceeding, then offer to write
     the mapping back into `AGENTS.md` so future runs skip discovery.

## Step 2 — Pull environment details from the aem MCP server

Using the environment's `authorUrl` from Step 1 (load each tool's schema with
`ToolSearch("select:<tool-name>")` first):

1. `mcp__aem__list-aem-capabilities` — what the environment supports.
2. `mcp__aem__lookup-api-spec` — discover available API specs; look for
   security / vulnerability / SBOM / dependency-related endpoints.
3. `mcp__aem__read-api` — read the relevant current state (reported
   vulnerabilities, package/bundle versions, etc.) for the environment.

Capture the concrete finding(s): the affected artifact, its version, the CVE (if
any), and **where in this repo it is introduced** (a `pom.xml` dependency, an
embedded jar, a config value, …).

Remediate **every** finding the scan reports, in this one run. Do **not** ask the
user which vulnerabilities to fix, or for permission to fix them — "fix security"
always means fix all of them.

## Step 3 — Apply the fix

1. Locate where the issue lives in the repo.
2. Make the **minimal** change to remediate it — upgrade or remove the
   vulnerable dependency, drop an embedded vulnerable jar, correct the
   misconfiguration, etc.
3. **When domain judgement is needed, dispatch a specialist subagent** and act
   on its recommendation. Give the subagent the finding from Step 2 and ask for
   the exact fix:
   - Security analysis / SBOM / CVE triage → a security agent (e.g.
     `starfish-team:Sage`).
   - Java / OSGi bundle changes → a Java agent (e.g. `starfish-team:Grace`).
   - Anything else, or if those agents are unavailable → `general-purpose`.

## Step 4 — Validate locally (must be green)

Run the Cloud Manager validation gate from the repo root:

```
mvn clean install
```

This runs `aemanalyser-maven-plugin` and the FileVault validators. **Do not
proceed until you see `BUILD SUCCESS` with no validation errors.** If it fails,
read the actual error output, fix it, and re-run — never claim green without the
output in hand. If the change could affect the classic profile, also run the CI
variant `mvn clean install -Pclassic`.

## Step 5 — Ship it

As soon as Step 4 is green, **immediately invoke the `git-flows:ship` skill** —
do **not** ask the user whether, how, or on which branch to ship, and do **not**
stop to summarise and wait for approval first. `git-flows:ship` opens a PR, waits
for CI to go green, squash-merges with a Conventional Commits subject,
fast-forwards `main`, and offers a release tag. Never commit directly on `main`
— `git-flows:ship` handles the branch and PR.

## Red flags — stop and fix, don't skip

| Thought | Reality |
|---|---|
| "I'll just guess the program" | Read `AGENTS.md`, or confirm via `list-aem-environments`, first. |
| "The MCP call errored, I'll assume the finding" | The aem tools are deferred — load the schema with `ToolSearch` first, then retry. |
| "The build probably passes" | Run `mvn clean install` and read the output before shipping. |
| "I'll ship straight from main" | Never commit on `main`; let `git-flows:ship` create the branch/PR. |
| "Which of these findings should I fix?" | Fix **all** reported findings — never ask the user to choose. |
| "It's green — let me confirm how to ship" | Don't ask. Green → invoke `git-flows:ship` immediately. |
