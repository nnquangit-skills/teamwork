# Teamwork

Use Teamwork to coordinate several agents on one project. Each Task has one owner responsible for integration. The coordinator selects models, dispatches work through the project's runner, monitors each run, and reports results as soon as they pass review.

## Install

From the directory containing the `teamwork` skill's `SKILL.md`, run the following on macOS or Linux.

```sh
mkdir -p "$HOME/.agents/skills"
dest="$HOME/.agents/skills/teamwork"
if [ ! -e "$dest" ] && [ ! -L "$dest" ]; then
  ln -s "$PWD" "$dest"
fi
ls -ld "$dest"
```

The link follows your local checkout as you update it. If a destination already exists, inspect the path printed by `ls`. Start a new agent session and request `teamwork` by name. For an agent with its own skills directory, link the same checkout there.

## Use in a project

Open an agent at the root of the project. To request a plan:

> Use teamwork to plan cash-on-delivery checkout. Break the work into Epics, Features, Tasks, and Subtasks. Include IDs, priorities, dependencies, owners, and acceptance criteria. Stop after the plan.

Once you approve the plan:

> Use teamwork to deliver T1. Dispatch independent Subtasks through the repository's ACP preset. Review submissions as they arrive, notify me when each ID passes, and monitor the work through the integrated checkout test.

To run agents through ACP, configure a runner, adapter, and access to the selected models. If the project uses a fallback provider, configure credentials in the runtime environment and give the coordinator its preset name. You can use the planning workflow on its own.

## Use with Worktree

Teamwork manages agent runs. The companion `worktree` skill records progress and evidence by ID in a Markdown tree. Teams with an existing tracker can use Teamwork with that tracker.

See the [coordinator instructions](SKILL.md) for monitoring intervals, escalation, and acceptance criteria.
