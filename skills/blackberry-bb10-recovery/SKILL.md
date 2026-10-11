---
name: blackberry-bb10-recovery
description: Diagnose and recover an owned BlackBerry 10 device through matching stock and AutoRoot images, QNX SSH, Android Runtime permissions, and PPS BAR installation on Windows or a separately verified Mac workflow.
---

# BlackBerry BB10 recovery

Use this skill for BB10 / QNX restoration, SSH and Runtime repair. Discover the host platform, connected device, firmware, tool versions and current reachable interface before mutation. A request to open SSH does not imply permission to wipe the phone. Apply the user's existing authorization without asking again.

## Route by symptom

- Flash/root: read [Windows field notes](https://github.com/EazyLee30/reberry/blob/main/docs/windows-field-notes.md), sections 01–02. Match stock to root build source. v3 userfs is not a full stock image. Respect exact downloaded script behavior and required stock → root → radio sequence for this version.
- SSH authentication: distinguish port listening, key authentication and password verification. For the legacy password branch read [password-ssh.md](references/password-ssh.md).
- Android stuck on progress: inspect `pidin ar`, `slog2info`, file modes and initialization queue; read section 04. Do not infer service health from UI progress alone.
- BAR installation: read sections 05–06. Use upd role for PPS; await success before deleting only known uploaded staging files.
- Mac: read [handoff](https://github.com/EazyLee30/reberry/blob/main/docs/mac-handoff.md); derive commands from the actual platform tools. Do not claim Windows executable compatibility or an unperformed flash.

## Non-obvious invariants from the observed device

1. This v3 btool reads `/var/rootdata/use.sshd`, not `use.ssh`. Inspect the installed/downloaded version instead of silently changing a marker by memory.
2. QNX `/bin/on -u upd /bin/sh -c '…'` works for installation from root SSH. Root may still fail direct PPS access due to roles.
3. Android rejected writable configuration files: four cfg files at 777 were changed to 644 with ownership retained. Discover `/apps/sys.android.*/native`; validate a unique directory before targeting files. Do not recursively modify `/apps`.
4. `/etc` contains immutable/overlay paths; `/bin/ln -sPf` is a QNX process-manager link operation. Do not substitute Linux assumptions or blindly rewrite accounts.
5. Several expected absolute Unix utility paths are absent. Shell echo and `/bin/cat` were available. `__root` needs an interactive shell and does not reliably execute an extra command argument.
6. Development bridge SSH may depend on the bridge process. Independent daemon startup needs separate configuration and boot-script inspection.
7. Old SHA-1 RSA / MAC compatibility options belong to the individual BB10 connection. Keep host verification; do not weaken global SSH policy.

## Evidence and publication

Track each distinct result: tool completion, root update, listening port, authentication, Runtime services, PPS success, application launch. Report what remains unknown. A BAR filename may disagree with package metadata. An APK login failure does not prove a Runtime failure.

Use the owner's newly generated key. Never expose private keys, device passwords, password hashes, personal addresses, serial/PIN, user paths or raw account databases. Sanitize examples before public Git commits; keep proprietary firmware and applications outside the repo. Do not delete root maintenance payloads with wildcard BAR cleanup.

Detailed walkthrough links point to the public repository so this folder can be installed on its own. Read those pages only for the selected workflow.
