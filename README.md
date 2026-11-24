# PR Graph

Review GitHub pull requests, fix blocking findings on their own branches, and ask a fresh reviewer to verify each change. A Python gate checks the exact commits, CI, labels and protected paths before it can squash-merge. Maintainers can run the full workflow or reuse just the gate.

Clone the trusted distribution and install the workflow files into a target checkout:

```sh
git clone https://github.com/arm-pz/pr-graph.git
python pr-graph/tools/pr-graph/install.py /path/to/target
```

The full loop needs an authenticated, configured agent runner. Codex and Claude launchers are included. The [setup guide](https://github.com/arm-pz/pr-graph/blob/main/tools/pr-graph/README.md) covers credentials, CI names and permissions.

Try “Review and fix the open PRs”, “Show which PRs are held and why”, or “Verify the fixes and give me the scorecard”. The [reviewer skill](https://github.com/arm-pz/consistent-code-review) produces the findings; PR Graph coordinates fixes, re-review and merging. The unrelated research repository with the same name studies graph-based posterior regularization.

## Run in another repository

Use the reusable [full workflow](.github/workflows/pr-graph-run.yml) or the [merge gate alone](.github/workflows/pr-graph.yml). Pin both the workflow reference and its `graph_revision` input to the same trusted commit of this repository. The caller fetches the distribution at run time. Skills Site and other consumers can adopt new revisions independently.

[Read the setup guide and caller examples](tools/pr-graph/README.md).

## Choose an agent

The workflow runs an executable selected by the runner operator. The adapter starts in the target checkout, receives the variables below, and writes its scorecard to standard output. It must return a nonzero status if the run cannot finish.

| Variable | Meaning |
| --- | --- |
| `PR_GRAPH_HOME` | Absolute path to the trusted graph distribution |
| `PR_GRAPH_TARGET` | Absolute path to the target checkout |
| `GITHUB_REPOSITORY` | Target GitHub owner/repository |
| `PR_GRAPH_AGENT` | Trusted adapter executable. Defaults to `pr-graph-agent` on PATH. The bundled names `codex` and `claude` select their respective adapters. |

Bundled launchers support Codex CLI and Claude Code. The runner supplies authentication, tool permissions, subagent support and the [consistent-code-review skill](https://github.com/arm-pz/consistent-code-review). Other agent runtimes can implement the same adapter contract. Keep PR test execution isolated from write credentials. Review and fix reports use the same JSON schemas regardless of the model.

## Label behavior

| Label | Review and fix | Merge |
| --- | --- | --- |
| `hold` | Yes | Blocked |
| `hands-off` | Skip | Blocked |
| `needs-owner` | Skip | Blocked |

PRs that change protected paths require a human merge. Configure required CI contexts in the target repository. The default required job is `check`.

## Development

Run `python -m unittest discover -s tools/agent-skills/tests` from this repository. This distribution originated in the MIT-licensed Skills Site project and retains its license.
