# Cowork Skills

A collection of personal skills for Microsoft 365 Copilot Cowork.

Cowork skills extend Copilot with custom workflows, triggered by phrases you define. Drop a skill folder into your OneDrive and Copilot will use it in your next session.

## Available skills

| Skill | Purpose | Example triggers |
|-------|---------|------------------|
| [servicenow-knowledge-connector](./servicenow-knowledge-connector) | Deploy, configure, and troubleshoot the Microsoft 365 Copilot ServiceNow Knowledge connector | "deploy the ServiceNow connector", "index ServiceNow KB articles", "troubleshoot ServiceNow connector" |

## Install a single skill

1. Click the green **Code** button above → **Download ZIP**
2. Unzip, then copy the folder for the skill you want into your OneDrive at:
   `OneDrive/Cowork/skills/<skill-name>/`
3. The skill will be available in your next Cowork session

## Install everything

After unzipping, copy all skill folders into `OneDrive/Cowork/skills/`. Cowork supports up to 50 personal skills.

## Contributing

Open a pull request with a new top-level folder containing at minimum a `SKILL.md`. The folder name must match the `name:` field in the SKILL.md frontmatter. See existing skills for examples.

## License

[MIT](./LICENSE)
