<p align="center"><img src="assets/banner.svg" alt="reberry — The keyboard remembers" width="100%"></p>

<p align="center">
<img alt="Device" src="https://img.shields.io/badge/BlackBerry-Classic_%2F_Q20-252522?style=flat-square">
<img alt="System" src="https://img.shields.io/badge/BB10-10.3.3.3216-75404b?style=flat-square">
<img alt="Host" src="https://img.shields.io/badge/Field_notes-Windows_%2B_macOS-b39360?style=flat-square">
<img alt="Skill" src="https://img.shields.io/badge/Agent_skill-included-494e42?style=flat-square">
</p>

<p align="center">English · <a href="README.zh-CN.md">简体中文</a> · <a href="README.zh-TW.md">繁體中文</a></p>

# reberry

**The keyboard remembers. Give the system a voice again.**

Field notes from restoring one BlackBerry Classic / Q20: a clean firmware baseline, root, independent SSH, and an Android Settings app that finally passed its loading screen. Practical observations from Windows, with a reusable agent skill and actual Mac SSH and BerryCore observations.

[Read the journal →](https://eazylee.xyz/reberry/) · [Windows walkthrough (Chinese)](docs/windows-field-notes.md) · [Reusable skill](skills/blackberry-bb10-recovery/SKILL.md) · [Mac field notes (Chinese)](docs/mac-field-notes.md) · [Mac handoff](docs/mac-handoff.md)

## v3 resources

The owner obtained the v3 tools and firmware resources through [BerryLife’s Pre-rooted Autoloaders v3 guide](https://bberrylife.github.io/Pages/V3root.html). That page links to the v3 tools and stock ROM resources, and credits original author Oleksandr. Follow its source links rather than downloading firmware from this repository.

## The path completed in this session

```text
Matching Stock 10.3.3.3216
  → AutoRoot v3 user filesystem built from that exact stock source
  → Matching radio
  → First boot and Development Mode
  → Term49 / root / an owner-controlled SSH key
  → Independent SSH service and optional password authentication
  → Android configuration permissions repaired
  → BAR installation through PPS as upd
  → Known uploaded installers removed after success
```

| Check | Evidence from this device |
| --- | --- |
| Stock, root v3, radio | All three flash operations completed; root update log completed |
| SSH | Independent port 22 executed remote commands; password login also succeeded |
| Android Runtime | System services started; the owner confirmed Settings opened |
| Batch BAR installation | Eight jobs reported `progress::100` and `result::success` |
| Game execution | Not individually checked; installation does not establish runtime compatibility |
| NetEase Cloud Music login | Old APK installed; its password-login server error remains unresolved |

## Three details worth carrying forward

1. This v3 script checks **`/var/rootdata/use.sshd`**. Read the downloaded startup script before copying a `use.ssh` instruction.
2. Root privileges do not automatically select the installer's role. **`/bin/on -u upd /bin/sh`** provided the required installation identity here.
3. Android's **`skipping insecure file`** messages pointed to configuration permissions. Changing four cfg files from 777 to 644, retaining their owners, restored initialization. Do not recursively change `/apps`.

## Copy this prompt into your agent

```text
Read https://github.com/EazyLee30/reberry . Start with README.md and
skills/blackberry-bb10-recovery/SKILL.md, then follow the references relevant to my task.
I own this BlackBerry 10 device. Help recover it using my host OS, device model,
firmware version and actual logs. Discover the current state and available tools first.
Do not assume reflashing is needed. Before flashing, confirm my authorization to erase
data and that the stock firmware matches the source used to build the root image.
Use my own newly generated SSH key. Keep credentials local; never publish passwords,
private keys, device identifiers, personal addresses or unsanitized logs.
Distinguish listening ports, authentication, root access, Android services,
installer success and actual application execution. Record unresolved issues.
For Mac work, read docs/mac-handoff.md and document only operations actually performed;
do not infer Apple Silicon flashing compatibility from a Windows success.
```

Copy `skills/blackberry-bb10-recovery` into your agent's skill directory (typically `~/.codex/skills/` for Codex), or ask the agent to read it directly. The installed skill uses repository links for its detailed walkthrough.

## Reading list

- [Windows: flashing, SSH, Runtime and PPS (Chinese)](docs/windows-field-notes.md)
- [Legacy password SSH branch (English)](skills/blackberry-bb10-recovery/references/password-ssh.md)
- [Mac contribution handoff and prompt (Chinese)](docs/mac-handoff.md)
- [Credits and sources](docs/credits.md)

The journal is available in [English](https://eazylee.xyz/reberry/), [简体中文](https://eazylee.xyz/reberry/zh-CN/) and [繁體中文](https://eazylee.xyz/reberry/zh-TW/). Technical source notes retain their stated language; an agent can translate them as needed.

## Privacy and scope

Includes newly authored text, original artwork, static web files and three owner-provided device photographs. No firmware, commercial apps, BAR/APK payloads, private keys, raw logs or account databases are published. Addresses and paths in examples are placeholders. This is evidence from one device, not a guarantee for every firmware or application. Reflashing erases data; the owner explicitly chose to proceed without backup in this session.

Prefer owner-controlled key authentication. The optional password branch documents a legacy DES compatibility limitation: only the first eight password bytes count, and protection is weak. Read its limitations and keep legacy root SSH off the public internet.

## Acknowledgements

Thank you to [BerryCore / sw7ft](https://github.com/sw7ft/BerryCore), [BerryLife](https://bberrylife.github.io/), [bb10root](https://github.com/bb10root), [BerryFarm / Term49](https://github.com/BerryFarm/Term49), Berry Much OS, Term49CX maintainers and the wider BB10 community. Thank you to OpenAI's ChatGPT and Codex for help with command analysis, troubleshooting, documentation and execution. Their work remains credited to its respective authors.

Original documentation and web design: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). BlackBerry trademarks belong to their respective owners. This is an independent community record.


BerryBrowser CJK: [project and compatibility notes](https://github.com/EazyLee30/berrybrowser-cjk) · [latest release](https://github.com/EazyLee30/berrybrowser-cjk/releases/latest). Chinese display font repack; upstream browser engine unchanged, Chinese input not supported.
