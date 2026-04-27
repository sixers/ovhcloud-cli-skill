# ovhcloud-cli-skill

Installable Codex skill bundle for the [OVHcloud CLI](https://github.com/ovh/ovhcloud-cli).

This repo packages one top-level `ovhcloud` skill, focused subskills, and read-only workflow recipes for working with `ovhcloud` safely from Codex.

## What You Get

- `skills/ovhcloud`: the installable bundle skill
- `skills/ovhcloud/subskills`: focused skills for core auth/config flows, Public Cloud areas, and non-cloud product families
- `skills/ovhcloud/recipes`: higher-level workflows such as account inventory, IAM audit, DNS audit, kube inspection, and storage inspection
- `docs/skills.md`: generated index of the bundle contents
- `scripts/generate_bundle.py`: generator that rebuilds the bundle from an upstream `ovhcloud-cli` clone

## Safety Model

These skills are intentionally conservative:

- Read-only `ovhcloud` commands are allowed by default
- Mutating commands are documented, but should only be run when the user explicitly asks for the state change
- The bundle is optimized for discovery, inspection, audits, and inventory tasks

## Prerequisites

- Codex with local skills support
- `ovhcloud` installed on your machine and available on `PATH`
- OVHcloud CLI authentication already configured, or ready to configure

Optional but recommended:

- A working `gh` login if you want to install directly from GitHub

## Install From GitHub

If you use the Codex skill installer, install the bundle from this repo path:

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo sixers/ovhcloud-cli-skill \
  --path skills/ovhcloud
```

This installs the bundle into:

```text
~/.codex/skills/ovhcloud
```

Restart Codex after installation so it picks up the new skill.

## Install Manually

Clone the repo anywhere you like:

```bash
git clone https://github.com/sixers/ovhcloud-cli-skill.git
cd ovhcloud-cli-skill
```

Then copy or symlink the bundle into your Codex skills directory:

```bash
mkdir -p ~/.codex/skills
ln -s "$(pwd)/skills/ovhcloud" ~/.codex/skills/ovhcloud
```

If you prefer copying instead of symlinking:

```bash
mkdir -p ~/.codex/skills
cp -R skills/ovhcloud ~/.codex/skills/ovhcloud
```

Restart Codex after installation.

## How To Use

Use the top-level bundle when you want Codex to choose the right subskill:

```text
Use $ovhcloud to inspect my OVHcloud estate and stay read-only.
```

You can also invoke narrower skills directly:

```text
Use $ovhcloud-shared to explain profile resolution and output flags.
Use $ovhcloud-cloud-kube to inspect my MKS clusters safely.
Use $ovhcloud-domain-zone to audit DNS records for a zone.
Use $recipe-audit-iam to review IAM users and policies without changing access.
```

The bundle entrypoint is here:

- [skills/ovhcloud/SKILL.md](skills/ovhcloud/SKILL.md)

The generated index is here:

- [docs/skills.md](docs/skills.md)

## Repo Layout

```text
.
├── docs/
│   └── skills.md
├── scripts/
│   └── generate_bundle.py
└── skills/
    └── ovhcloud/
        ├── SKILL.md
        ├── recipes/
        ├── references/
        └── subskills/
```

## Regenerate The Bundle

The bundle is generated from an upstream `ovhcloud-cli` checkout.

From this repo root:

```bash
python3 scripts/generate_bundle.py \
  --source-repo /path/to/ovhcloud-cli \
  --repo-root .
```

For example, if you cloned the upstream repo next to this one:

```bash
python3 scripts/generate_bundle.py \
  --source-repo ../ovhcloud-cli \
  --repo-root .
```

This regenerates:

- `skills/ovhcloud/SKILL.md`
- all subskills
- all recipe skills
- all generated references
- `docs/skills.md`

## Updating After Regeneration

After regenerating, it is a good idea to rerun the same local checks used while building the repo:

```bash
python3 - <<'PY'
from pathlib import Path
import subprocess

validator = Path.home() / ".codex/skills/.system/skill-creator/scripts/quick_validate.py"
for skill_dir in sorted({p.parent for p in Path("skills/ovhcloud").rglob("SKILL.md")}):
    subprocess.run(["python3", str(validator), str(skill_dir)], check=True)
PY
```

And spot-check the installed CLI help:

```bash
ovhcloud --help
ovhcloud cloud --help
ovhcloud account get -o json
ovhcloud vps list -o json
```

Keep all validation read-only unless you explicitly want to change live OVHcloud resources.
