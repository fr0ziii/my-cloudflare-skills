# My Cloudflare Skills

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

Reusable [Agent Skills](https://agentskills.io/) for designing, building, migrating, securing, and releasing applications on the Cloudflare Developer Platform.

## Available skills

### `build-on-cloudflare`

A 12-phase workflow for Cloudflare applications. It covers product boundaries, runtime selection, configuration, data ownership, caching, security, background work, public interfaces, observability, billing, staging, and production release.

- [Skill instructions](skills/build-on-cloudflare/SKILL.md)
- [Cloudflare reference](skills/build-on-cloudflare/REFERENCE.md)

### `migrate-to-cloudflare`

An eight-phase workflow for moving applications, services, data, identity, and traffic to Cloudflare through reversible migration slices.

- [Skill instructions](skills/migrate-to-cloudflare/SKILL.md)
- [Migration reference](skills/migrate-to-cloudflare/REFERENCE.md)

## Install

Clone the repository:

```sh
git clone https://github.com/fr0ziii/my-cloudflare-skills.git
```

Copy one skill or all skills to the directory that your agent uses. For an Agent Skills-compatible global directory:

```sh
mkdir -p ~/.agents/skills
cp -R my-cloudflare-skills/skills/* ~/.agents/skills/
```

For one project:

```sh
mkdir -p .agents/skills
cp -R /path/to/my-cloudflare-skills/skills/<skill-name> .agents/skills/
```

Restart the agent if it discovers skills only at startup. If the agent exposes skills as commands, invoke the selected skill by its frontmatter name.

> [!IMPORTANT]
> Skills can tell an agent to run commands and change cloud resources. Review the instructions before use and keep deployment approval rules in effect.

## Repository layout

```text
skills/
├── build-on-cloudflare/
│   ├── SKILL.md
│   └── REFERENCE.md
└── migrate-to-cloudflare/
    ├── SKILL.md
    └── REFERENCE.md
```

Each skill follows the [Agent Skills specification](https://agentskills.io/specification).

## Contributing

Issues and pull requests are welcome. Read [CONTRIBUTING.md](CONTRIBUTING.md) before you propose a change. Follow the [security policy](SECURITY.md) for sensitive reports.

## License

[MIT](LICENSE) © 2026 David Iglesias Guerra
