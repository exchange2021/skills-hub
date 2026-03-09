# skills-hub

`skills-hub` is a centralized repository for Codex Skills. It currently includes the `chainup-spot` skill for ChainUp/OpenAPI V2 spot and margin trading workflows.

## Purpose

- Consolidate reusable skill templates and execution workflows
- Manage skill versions and collaboration through Git
- Enable fast and stable onboarding for new skills

## Repository Structure

```text
skills-hub/
├── README.md
└── chainup-spot/
    ├── SKILL.md
    ├── agents/
    │   └── openai.yaml
    └── references/
        ├── authentication.md
        ├── spot-endpoints.md
        └── margin-endpoints.md
```

## Included Skill

### `chainup-spot`

Covers ChainUp/OpenAPI V2 spot and margin trading operations, including:

- Account balance query (`/sapi/v1/account`)
- Spot order placement/cancel/query (`/sapi/v2/*`)
- Margin order placement/cancel/trade query (`/sapi/v1/margin/*`)
- `X-CH-SIGN` authentication rules and request examples

Entry document: `chainup-spot/SKILL.md`

## Using in Codex

Copy the skill directory from this repository into your local Codex skills directory (for example, `~/.codex/skills`).

Example:

```bash
cp -a chainup-spot ~/.codex/skills/
```

## Contribution Guidelines

- Every skill must include a `SKILL.md`
- When auth rules, endpoints, or parameters change, update `references/` accordingly
- Never commit production secrets, tokens, or private credentials
- For new/updated skills, include usage examples and change notes

## Versioning Recommendations

- Use feature branches and merge via pull requests
- Recommended commit prefixes: `feat:` / `fix:` / `docs:` / `refactor:`
- If external behavior changes (auth, request format, field semantics), describe it clearly in the commit message
