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
# Standalone:
cd ike-example-its
mvn verify

# As an ike-example-ws subproject (file-activated profile picks it
# up automatically when the directory exists):
cd ike-example-ws
git clone https://github.com/IKE-Network/ike-example-its.git its
mvn ws:init    # or: mvn verify -pl its
```

When `its/` is present inside `ike-example-ws`, the workspace
reactor walks it during full builds. When it's absent, the
workspace reactor skips it cleanly.

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

### Authored

| IT | Purpose |
|---|---|
| `src/it/doc-only/` | Minimal `<packaging>pom</packaging>` project with `src/docs/asciidoc/`. Asserts that `mvn verify` renders HTML and attaches the `adoc` source classifier without additional plugin config. |
| `src/it/java-plus-docs/` | `<packaging>jar</packaging>` project with `src/main/java/` and `src/docs/asciidoc/`. Asserts that both JAR and HTML are produced in a single `mvn verify`. |
| `src/it/bom-import/` | Consumer that imports `network.ike.platform:ike-bom` via `<scope>import</scope>` but does not inherit `ike-parent`. Asserts the BOM's managed dependency versions flatten correctly at consumer build time. |
| `src/it/workspace-create/` | Invokes `ws:create` from a scratch directory and asserts the generated workspace POM references `network.ike.platform:ike-parent` at the released version. |

### "Would-have-been-caught" backlog

| IT | Would have caught |
|---|---|
| `src/it/scaffold-extensions/` | `.mvn/extensions.xml` for `wagon-ssh-external` (#338) reaching external projects via `ike:scaffold-publish`. |
| `src/it/built-with-classifier/` | Curated narrative pulled from the platform-wide `supplement.yaml` unpacked from `ike-build-standards:built-with:zip` (#340). |
| `src/it/sbom-shape/` | `target/bom.json` validates against the CycloneDX 1.6 schema and contains expected components for a known-shape consumer (#333). |
| `src/it/release-cascade/` | Full `ike-tooling → ike-docs → ike-platform → consumer` cascade in isolated local repos with synthetic versions. Catches inter-repo coordination bugs (the kind that hit v148/v149 and required intermediate fixes; the v150 workspace cascade also surfaced the parent-artifactId staging nesting in #342). |

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

## Links

- **Documentation:** [`https://ike.network/ike-example-its/`](https://ike.network/ike-example-its/)
- **Workspace:** [`IKE-Network/ike-example-ws`](https://ike.network/ike-example-ws/) — pluggable harness for the workspace's release cascade
- **Foundation sites:**
  [`ike-tooling`](https://ike.network/ike-tooling/) ·
  [`ike-docs`](https://ike.network/ike-docs/) ·
  [`ike-platform`](https://ike.network/ike-platform/)
- **Issues:** [`IKE-Network/ike-issues`](https://github.com/IKE-Network/ike-issues) (cross-project tracker)
- **Source:** [`IKE-Network/ike-example-its`](https://github.com/IKE-Network/ike-example-its)

## License

Apache License 2.0. See [LICENSE](LICENSE) or
[apache.org/licenses/LICENSE-2.0](https://www.apache.org/licenses/LICENSE-2.0).
