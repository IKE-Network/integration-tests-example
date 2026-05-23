# integration-tests-example

Integration test harness for the IKE Network release cascade. Pluggable
into `workspace-reactor-example` as an optional subproject (#343), but also
runnable standalone.

Formerly named `ike-example-its` (with subproject key `its` in the
workspace manifest); renamed under the canonical naming policy in
IKE-Network/ike-issues#467 so the artifact ID, git repo name, on-disk
directory, and workspace.yaml subproject key all match the spelled-out
role.

## First Steps

```bash
mvn validate    # unpack build standards into .claude/standards/
mvn verify      # run all IT cases
```

If `mvn validate` fails because `ike-build-standards` isn't in the local
repository, install it first via `ike-tooling`'s reactor.

## Build

```bash
mvn verify                       # all IT cases
mvn verify -Dinvoker.test=NAME   # one case
```

## Key Conventions

- Maven 4 with POM modelVersion 4.1.0
- Parent: `network.ike.platform:ike-parent` (from ike-platform) — gives
  this harness Java 25 build conventions, distributionManagement, and the
  same plugin matrix as the workspace it slots into.
- IT-target version pins live as literals inside the maven-invoker-plugin's
  `<filterProperties>` block, not as `<properties>` declarations. Maven 4's
  consumer-POM flattener bakes `<properties>` values into released artifacts;
  putting `1-SNAPSHOT` placeholders there causes the workspace
  `ws:release-publish` preflight to refuse the release. Filter properties
  are read by `maven-invoker-plugin` only and never reach the consumer POM.

## Prohibited Patterns

- **Never use `maven-antrun-plugin`** — use a proper Maven goal
- **Never use `build-helper-maven-plugin` for multi-execution property
  chaining** — write a proper Maven goal in `ike-maven-plugin`
- **Never embed shell commands inline in POM** — extract to a named script

## See also

- `README.adoc` for the planned IT inventory and the
  "would-have-been-caught" list.
- `IKE-Network/ike-issues#343` — the issue that motivated splitting this
  out of `workspace-reactor-example` (originally `ike-example-ws`, then
  `workspace-example`).
