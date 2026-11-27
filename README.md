# data-platform-architect

Skill Claude Code qui aide à trancher une décision d'architecture data (Snowflake, dbt, Airbyte, Airflow, Cube, Kubernetes/ArgoCD) : cadrage, 2-3 options dont « étendre l'existant », comparaison pondérée, affirmations sourcées, recommandation et ADR.

## Installation

```bash
git clone https://github.com/ikramzoua/data-platform-architect ~/.claude/skills/data-platform-architect
cp ~/.claude/skills/data-platform-architect/references/context{.example,}.md
```

Remplis `references/context.md` avec ta stack (il est ignoré par git), puis dans Claude Code : `/data-platform-architect <ta problématique>`.
