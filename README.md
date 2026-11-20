# volt-runner

Runs a workspace's [Volt](https://voltspace.io) coding sessions on a server of your company.

When someone hands an issue to a coding agent in Volt, the runner starts the agent in a
container of the session's own, on the session's own clone of the repository, shows what it
does in Volt, passes on its questions and pushes the branch at the end of each turn. Volt
then opens a draft pull request.

A runner belongs to the workspace, not to a person: an admin connects it once, and everyone
who can edit issues has agents work on it. Nobody installs anything on their own computer.

The image is `ghcr.io/voltplanner/runner`.

## Start

In Volt, an admin opens **Settings › Runners** and presses **Connect runner**. Volt shows a
command like this one, with its own address and a code in it:

```bash
docker run -d --name volt-runner --restart unless-stopped \
  --hostname "$(hostname)" -v volt-runner:/data \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -e VOLT_URL=https://api.voltspace.io \
  -e VOLT_CONNECT_CODE=XXXX-XXXX-XXXX \
  -e OPENROUTER_API_KEY=your-key \
  -e VOLT_OPENCODE_MODELS=openrouter/deepseek/deepseek-v4-pro \
  ghcr.io/voltplanner/runner
```

Put your provider's key in place of `your-key` and run it on the server. The code works once
and for a few minutes; the runner then shows in Volt as connected.

The server needs Docker Engine 26 or newer. The runner gets the server's Docker socket,
because it starts a container for every session.

## Agents and keys

The image has two agents, and the runner offers each one it has a key for:

- **Claude Code** with `ANTHROPIC_API_KEY`.
- **OpenCode** with the key of a provider it knows, such as `OPENROUTER_API_KEY` or
  `DEEPSEEK_API_KEY`; the command Volt shows names the key of the provider you choose.
  People pick from the models named in `VOLT_OPENCODE_MODELS`, with commas between.
  `docker exec volt-runner opencode models` lists the name of every model the runner
  reaches.

The keys stay on your server. They are given to the runner's container alone and never go
to Volt; an agent reaches its model through the runner and never sees the key.

## A container per session

- Every session works in a container of its own, on its own clone of the repository, and
  sees nothing of other sessions.
- A session's container has no Docker socket and none of the runner's keys, and runs as an
  ordinary user.
- The runner has nobody's git access. For each session Volt gives it a token of its app on
  GitHub, good for that one repository and for an hour. The token stays in the runner's
  container.
- A session gets 2 cores, 4 GB of memory and 1024 processes; `VOLT_SESSION_CPUS`,
  `VOLT_SESSION_MEMORY` and `VOLT_SESSION_PIDS` on the runner change that.
- A runner runs 2 sessions at once, or what `VOLT_RUNNER_CAPACITY` says, up to 16. A turn
  it has no room for waits in Volt.

A session's container has the network the server has, and containers share the server's
kernel: put the runner on a server where that is acceptable.

## Updating and removing

Everything the runner keeps is in its volume, so a new container from the same volume is the
same runner. To update, or to change its keys and models:

```bash
docker pull ghcr.io/voltplanner/runner
docker rm -f volt-runner
# then the same docker run command again; the used code doesn't matter
```

To remove a runner, press **Disconnect** on it in **Settings › Runners**: its sessions end.
Then remove the container and its volume
(`docker rm -f volt-runner && docker volume rm volt-runner`).

## More

- The image has git, Node, Python and a compiler. A repository that needs more gets an image
  built `FROM ghcr.io/voltplanner/runner`, named to the runner as `VOLT_SESSION_IMAGE`.
- `docker exec volt-runner volt-runner doctor` says which agents the runner found and which
  of them have a key.
- How sessions work in Volt: [voltspace.io/docs/coding-sessions](https://voltspace.io/docs/coding-sessions).

Releases 0.7 and older on this page are of the earlier runner, which was installed on a
person's own computer. Volt no longer works with it.
