# integration-tests-example — IKE Release-Cascade IT Harness

[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![Documentation](https://img.shields.io/badge/docs-ike.network%2Fintegration--tests--example-blue)](https://ike.network/integration-tests-example/)
[![IKE Network](https://img.shields.io/badge/IKE-Network-green)](https://ike.network/)

> **Note (2026-05-20):** This project was previously named
> `ike-example-its` and slotted into the workspace under the
> 3-letter key `its`. It was renamed to `integration-tests-example`
> under the canonical naming policy in IKE-Network/ike-issues#467
> so the artifact ID, git repo name, on-disk directory, and
> workspace.yaml subproject key all match the spelled-out role.
> GitHub redirects keep old clone URLs working.

End-to-end smoke tests that verify the IKE Network release cascade
works from the perspective of an external consumer. Each IT is a
self-contained POM project under `src/it/` that
`maven-invoker-plugin` executes in a fresh Maven environment,
resolving `ike-tooling`, `ike-docs`, and `ike-platform` artifacts
from Nexus (or from the local repository if already installed).

## Layout

This is its own repository (split from `workspace-example`
in [`IKE-Network/ike-issues#343`](https://github.com/IKE-Network/ike-issues/issues/343)
so the IT harness can evolve on its own cadence and so a broken IT
POM never blocks the workspace release cascade).

It plugs into `workspace-example` as an **optional** subproject:

```bash
# Standalone:
git clone https://github.com/IKE-Network/integration-tests-example.git
cd integration-tests-example
mvn verify

# As a workspace-example subproject (cloned into
# ./integration-tests-example/, where the workspace's file-activated
# profile picks it up):
cd workspace-example
mvn ws:scaffold-init                               # or:
git clone https://github.com/IKE-Network/integration-tests-example.git
mvn verify -pl integration-tests-example
```

When the subproject directory is present inside `workspace-example`,
the workspace reactor walks it during full builds. When it's absent,
the workspace reactor skips it cleanly. The directory name matches
the artifact, the repo, and the workspace.yaml subproject key —
one name everywhere (IKE-Network/ike-issues#467).

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

### Authored (implemented)

| IT | Purpose |
|---|---|
| `src/it/scm-lint-multimodule-pass/` | Multi-module reactor whose root declares `<scm>` and whose subprojects inherit. Verifies the `ike-version-management-extension` MISSING_SCM lint correctly walks the git boundary and skips subproject POMs ([`IKE-Network/ike-issues#496`](https://github.com/IKE-Network/ike-issues/issues/496)). |
| `src/it/scm-lint-root-missing-fail/` | Reactor root sitting at the `.git` boundary with no `<scm>` block. Asserts the lint fires with a `MISSING-SCM` violation that cites issue #496. |
| `src/it/scm-lint-standalone-pass/` | Single-module project at the `.git` repo root with a local `<scm>`. Sanity check that the lint still permits the original well-formed shape. |

These ITs require `ike-version-management-extension` at version
`2-SNAPSHOT` (or later, once released) to be installed in the local
Maven repository — register it via the IT's `.mvn/extensions.xml`.
Run `mvn install` in
[`IKE-Network/ike-version-management-extension`](https://github.com/IKE-Network/ike-version-management-extension)
before invoking `mvn verify` here. Each IT's `setup.bsh` creates an
empty `.git/` directory in the cloned IT project so the extension's
git-boundary walk treats that POM as a repository root; BeanShell
is used rather than Groovy because the bundled Groovy 3.x in
`maven-invoker-plugin` 3.9.0 cannot decode the Java 25 class files
that the rest of the IKE stack compiles to.

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

- **Documentation:** [`https://ike.network/integration-tests-example/`](https://ike.network/integration-tests-example/)
- **Workspace:** [`IKE-Network/workspace-example`](https://ike.network/workspace-example/) — pluggable harness for the workspace's release cascade
- **Foundation sites:**
  [`ike-tooling`](https://ike.network/ike-tooling/) ·
  [`ike-docs`](https://ike.network/ike-docs/) ·
  [`ike-platform`](https://ike.network/ike-platform/)
- **Build standards:** [`ike-build-standards`](https://ike.network/ike-tooling/ike-build-standards/)
- **Issues:** [`IKE-Network/ike-issues`](https://github.com/IKE-Network/ike-issues) (cross-project tracker)
- **Source:** [`IKE-Network/integration-tests-example`](https://github.com/IKE-Network/integration-tests-example)

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
