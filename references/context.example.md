# Contexte de la data platform

> Copie ce fichier en `references/context.md` (ignoré par git) et remplis-le avec ta stack réelle. Les lignes **À compléter** changent souvent la recommandation.

## Métier

- Activité de l'entreprise et domaine des données : **À compléter**
- Données centrales (faits et dimensions clés) : **À compléter**
- Multi-tenant ? Qui ne doit voir que ses données, et où l'isolation est appliquée : **À compléter**
- Consommateurs : équipes internes, clients, BI, agents IA, apps : **À compléter**

## Stack

| Couche | Outil | Où (repo / chemin) |
|---|---|---|
| Ingestion | ex. Airbyte sur Kubernetes | |
| Sources | | |
| Entrepôt | ex. Snowflake, Cortex | |
| Migrations DDL | ex. schemachange | |
| Transformation | ex. dbt : `staging` → `intermediate` → `marts` | |
| Orchestration | ex. Airflow | |
| CI/CD | ex. GitHub Actions, preprod → prod | |
| Semantic layer | ex. Cube | |
| IA | ex. agent conversationnel sur l'entrepôt | |
| Infra | ex. Azure AKS, Terraform, ArgoCD, External Secrets | |
| BI / monitoring | ex. Power BI, Grafana | |

## Mesures connues

- Benchs, temps de requête, coûts mesurés (avec date et méthode) : **À compléter**

## Contraintes — À compléter

- Taille de l'équipe data et compétences : **À compléter**
- Budget Snowflake mensuel / crédits, édition, région : **À compléter**
- Volumétrie des plus grosses tables (lignes, Go) et croissance : **À compléter**
- SLA de fraîcheur par consommateur : **À compléter**
- Exigences contractuelles clients (RGPD, localisation des données, droit d'audit) : **À compléter**
- Outils interdits ou imposés : **À compléter**
