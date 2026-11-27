# Leviers et pièges par domaine

Ce fichier liste ce qui compte **sur cette stack**. Chaque point est un réflexe à vérifier, pas une vérité : confirme avec la doc ou une mesure avant de l'affirmer.

## Sommaire
- Ingestion
- Modélisation dbt
- Orchestration
- Semantic layer et multi-tenant
- Coût Snowflake
- IA sur l'entrepôt (agent, Cortex)
- Déploiement et exploitation
- Gouvernance et qualité

## Ingestion
- Avant d'ajouter une source : connecteur Airbyte existant ? Sinon, comparer connecteur custom (Airbyte CDK), Snowpipe / `COPY INTO` depuis Azure Blob, ou partage Snowflake si la source est déjà sur Snowflake.
- Si l'ingestion est éteinte la nuit (scale-to-zero, ex. KEDA) : un besoin de sync nocturne ou temps réel remet ce choix en cause.
- Suppressions côté source : vérifier le mode de sync (incremental append vs dedup + history) et la gestion des hard deletes.
- Vérifier : logs de sync Airbyte, `ACCOUNT_USAGE.COPY_HISTORY` / `LOAD_HISTORY`.

## Modélisation dbt
- Grain d'abord : la plupart des bugs de chiffres viennent d'un grain mal défini (ex. une fact = visite × produit vs semaine × magasin × produit).
- Marts par client : arbitrer duplication par client vs un mart commun filtré par `client_id`. Le commun réduit la maintenance mais oblige à isoler en aval (Cube, row access policies).
- Incrémental : `unique_key` + stratégie (`merge` vs `delete+insert`), late-arriving data, `is_incremental()` avec une fenêtre de rattrapage. Prévoir un full refresh testé.
- Snapshots dbt pour les dimensions SCD2 plutôt que du code maison.
- schemachange vs dbt : DDL infra (rôles, grants, objets non-dbt) dans schemachange ; tout ce qui est dérivé de données dans dbt. Éviter qu'un objet soit possédé par les deux.
- Vérifier : `dbt build --select state:modified+`, tests `unique`/`not_null`/`relationships`, comptes de lignes avant/après.

## Orchestration
- Si Airflow tourne en docker-compose : point de fragilité (HA, upgrade, secrets) par rapport à un reste sur Kubernetes + ArgoCD. Une question d'orchestration peut cacher « faut-il passer Airflow sur Kubernetes (chart officiel) ».
- Orchestrer dbt : DAG par couche vs Cosmos (un task par modèle) vs `dbt build` unique. Le grain des tasks change le retry et la visibilité, pas la vitesse.
- Alternatives natives Snowflake (Tasks, Dynamic Tables) pour des chaînes simples : moins d'infra, mais lineage et tests hors dbt sauf si dbt les crée (`materialized='dynamic_table'`).
- Déclencher sur événement (fin de sync Airbyte) plutôt que sur horaire quand la fraîcheur compte.

## Semantic layer et multi-tenant
- Cube est la couche d'isolation client et de cache : toute nouvelle consommation client (BI, export, agent IA, app) devrait passer par Cube ou par une isolation équivalente dans Snowflake (row access policies + rôle par client). Deux mécanismes d'isolation parallèles = deux endroits où fuir.
- Pré-agrégations : rentables sur les requêtes répétées ; coût = refresh × nombre de clients (`scheduledRefreshContexts`). Chiffrer.
- Mesures déclarées par client : la définition d'un KPI doit vivre à un seul endroit (dbt ou Cube, pas les deux).
- Manquant ≠ zéro : toute agrégation doit distinguer absence de donnée et valeur nulle.
- Vérifier : test de parité Cube vs SQL direct , `npm test` de la politique d'accès.

## Coût Snowflake
- Mesurer avant d'optimiser : `ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY`, `QUERY_ATTRIBUTION_HISTORY`, `QUERY_HISTORY` (bytes scanned, spilling, partitions scanned/total).
- Leviers dans l'ordre d'effort : auto-suspend court, taille de warehouse adaptée (un warehouse par usage : ingestion / dbt / BI / Cube / IA pour attribuer les coûts), incrémental, clustering seulement sur grosses tables avec pruning mauvais, resource monitors.
- Cube et un agent IA peuvent générer beaucoup de petites requêtes : vérifier le cache et le warehouse dédié.
- Cortex : coût facturé au token/crédit selon le modèle ; chiffrer sur le volume réel.

## IA sur l'entrepôt (agent, Cortex)
- Principe recommandé : le chiffre vient du moteur, jamais du LLM. Toute option qui fait générer du SQL libre par un LLM sur l'entrepôt va contre ce principe ; la justifier ou la rejeter.
- Lire les mesures via Cube garantit la même définition et la même isolation que la BI.
- Embeddings : Cortex (dans Snowflake, pas de sortie de données) vs API externe (contrat, RGPD, coût).

## Déploiement et exploitation
- Tout nouveau service : Helm chart + ArgoCD Application + External Secrets + scale-to-zero si non-prod. Compter ce coût dans l'option.
- Managé (SaaS) vs self-hosted sur AKS : le self-hosted économise la licence mais coûte en astreinte et upgrades. Avec une petite équipe, le pencher vers le managé est souvent justifié ; le montrer avec des chiffres.
- Terraform pour l'infra cloud, jamais de ressources cliquées.

## Gouvernance et qualité
- Tests dbt et contrats (`contract: enforced`) sur les marts consommés par des clients.
- Accès : rôles Snowflake par fonction et par client ; grants versionnés (schemachange / orchestration).
- Lineage et documentation : dbt docs d'abord, catalogue dédié seulement si plusieurs équipes non-data consomment.
- RGPD : données de staff (SIRH, onboarding) ; masquage (masking policies) avant d'exposer.
