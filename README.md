# volt-runner

Releases of volt-runner, which runs Volt coding sessions on your own machine.

Install it from Volt: **Settings → Runners**. It updates itself from the latest release here.

Each release holds the executables for every platform (gzipped), `stable.json`, the manifest
runners install and update from, and `stable.json.sig`, its ed25519 signature. A runner only
takes a release whose manifest is signed with the key built into it, and checks every file
against the sha256 the manifest names.
