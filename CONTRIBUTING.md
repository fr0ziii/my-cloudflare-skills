# Contributing

Thank you for improving these skills.

## Propose a change

Open an issue before a large change. Explain the user problem, the affected skill, and the expected agent behavior.

Keep each pull request focused on one concern. Use synthetic examples and remove credentials, account identifiers, and private data.

## Skill requirements

Each skill must:

- live in `skills/<skill-name>/`;
- include a `SKILL.md` with valid Agent Skills frontmatter;
- use a lowercase, hyphenated name that matches its directory;
- state specific invocation conditions in its description;
- keep the main workflow short and move branch-specific detail to one reference file;
- give each workflow phase a checkable exit condition;
- cite official documentation for platform facts;
- request Cloudflare documentation with `Accept: text/markdown` when possible.

Use simple, direct instructions. Keep each rule in one authoritative location.

## Checks

Before you submit a pull request:

1. Confirm that all relative links resolve.
2. Confirm that official documentation links return successfully.
3. Check that examples contain no secret or private values.
4. Check that the skill name and description satisfy the Agent Skills specification.
5. Run `git diff --check`.
6. Read the complete skill as an agent workflow and remove duplicated instructions.

By contributing, you agree that your contribution is licensed under the repository's [MIT License](LICENSE).
