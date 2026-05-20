# IKE Example Workspace — Integration Test Harness

[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![Documentation](https://img.shields.io/badge/docs-ike.network%2Fike--example--its-blue)](https://ike.network/ike-example-its/)
[![IKE Network](https://img.shields.io/badge/IKE-Network-green)](https://ike.network/)

End-to-end smoke tests that verify the IKE Network release cascade
works from the perspective of an external consumer. Each IT is a
self-contained POM project under `src/it/` that
`maven-invoker-plugin` executes in a fresh Maven environment,
resolving `ike-tooling`, `ike-docs`, and `ike-platform` artifacts
from Nexus (or from the local repository if already installed).

## Layout

This is its own repository (split from
[`IKE-Network/ike-example-ws`](https://github.com/IKE-Network/ike-example-ws)
in [`IKE-Network/ike-issues#343`](https://github.com/IKE-Network/ike-issues/issues/343)
so the IT harness can evolve on its own cadence and so a broken IT
POM never blocks the workspace release cascade).

It plugs into `ike-example-ws` as an **optional** subproject:

```bash
# Standalone (the repo clones as ike-example-its by default):
git clone https://github.com/IKE-Network/ike-example-its.git
cd ike-example-its
mvn verify

# As an ike-example-ws subproject (cloned into ./its/, where the
# workspace's file-activated `with-its` profile picks it up):
cd ike-example-ws
mvn ws:scaffold-init                               # or:
git clone https://github.com/IKE-Network/ike-example-its.git its
mvn verify -pl its
```

When `its/` is present inside `ike-example-ws`, the workspace
reactor walks it during full builds. When it's absent, the
workspace reactor skips it cleanly. The repo's local directory
name `its` (not `ike-example-its`) preserves operator muscle
memory from the pre-#343 layout when the harness lived inside
`ike-example-ws`.

## What each IT must assert

1. **Artifact coordinates are correct** — `network.ike.platform:ike-parent:N`,
   `network.ike.docs:ike-doc-maven-plugin:N`, etc.
2. **adoc-classifier attachment is produced** — for any project
   with `src/docs/asciidoc/`, the doc-pipeline profile in
   `ike-parent` auto-activates and `maven-assembly-plugin` attaches
   a `<classifier>adoc</classifier><type>zip</type>` artifact.
   (Earlier revisions tested `<extensions>true</extensions>` plugin
   resolution for `<packaging>ike-doc</packaging>`; that machinery
   was retired in [`IKE-Network/ike-issues#321`](https://github.com/IKE-Network/ike-issues/issues/321).)
3. **Outputs are produced** — `target/generated-docs/html/`,
   `target/*.jar`, etc., depending on the IT.
4. **No unresolved property references** — the consumer POM does
   not rely on properties defined in parent contexts that weren't
   carried through.

## Test inventory

The harness is currently **structural-only**: `maven-invoker-plugin`
is configured in `pom.xml` and `src/it/settings.xml` is in place,
but no IT cases have been authored yet. The tables below are
authored backlog and "would-have-been-caught" lists.

### Authored backlog (planned)

| IT | Purpose |
|---|---|
| `src/it/doc-only/` | Minimal `<packaging>pom</packaging>` project with `src/docs/asciidoc/`. Asserts that `mvn verify` renders HTML and attaches the `adoc` source classifier without additional plugin config. |
| `src/it/java-plus-docs/` | `<packaging>jar</packaging>` project with `src/main/java/` and `src/docs/asciidoc/`. Asserts that both JAR and HTML are produced in a single `mvn verify`. |
| `src/it/bom-import/` | Consumer that imports `network.ike.platform:ike-bom` via `<scope>import</scope>` but does not inherit `ike-parent`. Asserts the BOM's managed dependency versions flatten correctly at consumer build time. |
| `src/it/workspace-create/` | Invokes `ws:scaffold-init` from a scratch directory and asserts the generated workspace POM references `network.ike.platform:ike-parent` at the released version. |

### "Would-have-been-caught" list

Cases worth adding as the cascade matures — each captures a real
post-mortem regression class:

| IT | Would have caught |
|---|---|
| `src/it/scaffold-extensions/` | `.mvn/extensions.xml` for `wagon-ssh-external` (#338) reaching external projects via `ike:scaffold-publish`. |
| `src/it/built-with-classifier/` | Curated narrative pulled from the platform-wide `supplement.yaml` unpacked from `ike-build-standards:built-with:zip` (#340). |
| `src/it/sbom-shape/` | `target/bom.json` validates against the CycloneDX 1.6 schema and contains expected components for a known-shape consumer (#333). |
| `src/it/release-cascade/` | Full `ike-tooling → ike-docs → ike-platform → consumer` cascade in isolated local repos with synthetic versions. Catches inter-repo coordination bugs of the kind that surfaced in #358 (staging unwrap), #380 (site URL realignment), and #381 (publish logic missed projectId-container layer). |
| `src/it/site-topology/` | Releases a multi-module consumer and asserts every subproject × {root, /N/, /latest/} URL on its gh-pages responds 200 — the missing-from-tree case #382 added to `verify-release-published`. |
| `src/it/inheritance-enforcer/` | External consumer that inherits `ike-parent` without overriding `<distributionManagement><site>` — asserts the build FAILS at validate with the #383 enforcer message. |

## Running the ITs

```bash
# Full IT suite:
mvn verify

# Single IT (once authored):
mvn verify -Dinvoker.test=doc-only
```

## Local vs Nexus mode

By default the IT settings consult the local repository first. To
force a pull from Nexus (simulating a clean consumer), delete the
relevant artifacts from `~/.m2/repository/network/ike/` before
running.

## Doc as Code + LLM-Friendly

This project follows the IKE Network's doc-as-code philosophy:
build conventions, documentation standards, and AI-assistant
guidance live as versioned Markdown files in
[`ike-build-standards`](https://github.com/IKE-Network/ike-tooling/tree/main/ike-build-standards#readme)
and are unpacked into every consumer's `.claude/standards/` at
the `validate` phase. When a developer — or Claude itself —
opens this project, the agent reads those standards and applies
them automatically; contributors don't have to memorize the
conventions.

The standards most directly relevant to an integration-test
harness are
[`TESTING.md`](https://github.com/IKE-Network/ike-tooling/blob/main/ike-build-standards/src/main/standards/TESTING.md)
(JUnit 5, AssertJ, `maven-invoker-plugin` for IT cases),
[`IKE-MAVEN.md`](https://github.com/IKE-Network/ike-tooling/blob/main/ike-build-standards/src/main/standards/IKE-MAVEN.md)
(IKE-specific Maven conventions), and
[`IKE-RELEASE.md`](https://github.com/IKE-Network/ike-tooling/blob/main/ike-build-standards/src/main/standards/IKE-RELEASE.md)
(release cascade, the workflow the ITs assert behavior against).
See the
[full inventory](https://github.com/IKE-Network/ike-tooling/tree/main/ike-build-standards#readme).

## Links

- **Documentation:** [`https://ike.network/ike-example-its/`](https://ike.network/ike-example-its/)
- **Workspace:** [`IKE-Network/ike-example-ws`](https://ike.network/ike-example-ws/) — pluggable harness for the workspace's release cascade
- **Foundation sites:**
  [`ike-tooling`](https://ike.network/ike-tooling/) ·
  [`ike-docs`](https://ike.network/ike-docs/) ·
  [`ike-platform`](https://ike.network/ike-platform/)
- **Build standards:** [`ike-build-standards`](https://ike.network/ike-tooling/ike-build-standards/)
- **Issues:** [`IKE-Network/ike-issues`](https://github.com/IKE-Network/ike-issues) (cross-project tracker)
- **Source:** [`IKE-Network/ike-example-its`](https://github.com/IKE-Network/ike-example-its)

## License

Apache License 2.0. See [LICENSE](LICENSE) or
[apache.org/licenses/LICENSE-2.0](https://www.apache.org/licenses/LICENSE-2.0).
<!-- BEGIN ike-managed: developer-setup -->

## Developer Setup

New to IKE development? The
[Developer Environment guide](https://ike.network/ike-tooling/ike-build-standards/developer-environment.html)
covers IDE configuration, JDK 25 setup, and the tooling conventions
every IKE workspace expects — start there before your first build.
<!-- END ike-managed: developer-setup -->
