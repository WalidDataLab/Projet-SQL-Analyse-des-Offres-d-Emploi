# 🗄️ Projet SQL — Analyse des Offres d'Emploi

> Ce projet fait partie de ma série de projets SQL dans le cadre de l'apprentissage et de la maîtrise du langage SQL pour l'analyse de données.

---

## 📋 Description du Projet

Ce projet contient des solutions à deux problèmes pratiques portant sur l'analyse d'une base de données d'offres d'emploi. L'objectif est de maîtriser les **sous-requêtes (subqueries)**, les **jointures (JOINs)** et les **expressions conditionnelles (CASE)** en SQL.

---

## 🧩 Problème Pratique 1 — Top 5 des Compétences les Plus Demandées

### ❓ Question
Identifier les 5 compétences les plus fréquemment mentionnées dans les offres d'emploi. Utiliser une sous-requête pour trouver les IDs de compétences avec le plus grand nombre d'occurrences dans la table `skills_job_dim`, puis joindre ce résultat avec la table `skills_dim` pour obtenir les noms des compétences.

---

### 💡 Solution

#### Approche 1 — Avec `LEFT JOIN` direct et `GROUP BY`

```sql
SELECT
    COUNT(job_id) AS number_of_postings_of_skills,
    skills_job_dim.skill_id,
    skills_dim.skills AS name_of_skill
FROM skills_job_dim

LEFT JOIN skills_dim
    ON skills_job_dim.skill_id = skills_dim.skill_id

GROUP BY skills_job_dim.skill_id, skills_dim.skills
ORDER BY number_of_postings_of_skills DESC
LIMIT 5;
```

#### Approche 2 — Avec Sous-requête (Subquery)

```sql
SELECT
    number_postings_by_skill_id.skill_id,
    number_postings_by_skill_id.number_of_postings,
    skills_dim.skills AS name_of_skill

FROM (
    SELECT
        skill_id,
        COUNT(job_id) AS number_of_postings
    FROM skills_job_dim
    GROUP BY skill_id
) AS number_postings_by_skill_id

LEFT JOIN skills_dim
    ON number_postings_by_skill_id.skill_id = skills_dim.skill_id

ORDER BY number_postings_by_skill_id.number_of_postings DESC
LIMIT 5;
```

---

### 🔍 Pourquoi utiliser `LEFT JOIN` ?

Le `LEFT JOIN` est utilisé ici pour **conserver toutes les compétences présentes dans `skills_job_dim`**, même si certaines n'ont pas de correspondance dans la table `skills_dim`. Cela garantit qu'aucune compétence n'est exclue du comptage à cause d'une absence dans la table de référence. Dans ce contexte, l'intégrité des données de comptage est prioritaire.

### 🔍 Pourquoi utiliser une Sous-requête (Subquery) ?

La sous-requête permet de **séparer la logique de comptage de la logique de jointure**. On calcule d'abord le nombre d'offres par compétence (`skill_id`) dans une requête interne, puis on joint ce résultat avec `skills_dim` pour récupérer les noms. Cette approche améliore la lisibilité et la clarté du code, surtout quand les traitements sont complexes.

---

### 📊 Résultat

| number_of_postings_of_skills | skill_id | name_of_skill |
|------------------------------|----------|---------------|
| 385 750 | 0 | sql |
| 381 863 | 1 | python |
| 145 718 | 76 | aws |
| 132 851 | 74 | azure |
| 131 285 | 5 | r |

> **SQL** et **Python** dominent largement les offres d'emploi, ce qui confirme leur importance centrale dans les métiers de la data.

---

## 🧩 Problème Pratique 2 — Catégorisation des Entreprises par Taille

### ❓ Question
Déterminer la catégorie de taille (`Small`, `Normal` ou `Large`) pour chaque entreprise en calculant d'abord le nombre total d'offres d'emploi par entreprise. Une entreprise est considérée comme `Small` si elle a moins de 10 offres, `Normal` entre 10 et 50 offres, et `Large` si elle en a plus de 50.

---

### 💡 Solution

#### Étape 1 — Catégoriser chaque entreprise

```sql
SELECT
    total_of_job.*,

    CASE
        WHEN total_of_job.number_of_job <= 10 THEN 'small'
        WHEN total_of_job.number_of_job > 10 AND total_of_job.number_of_job <= 50 THEN 'normal'
        ELSE 'large'
    END AS size_of_company

FROM (
    SELECT
        COUNT(job_id) AS number_of_job,
        company_id AS id_of_company
    FROM job_postings_fact
    GROUP BY company_id
) AS total_of_job;
```

#### Étape 2 — Compter les entreprises par catégorie (Bonus)

```sql
SELECT
    COUNT(size_of_company) AS number_of_company_size,
    size_of_company

FROM (
    SELECT
        total_of_job.*,

        CASE
            WHEN total_of_job.number_of_job <= 10 THEN 'small'
            WHEN total_of_job.number_of_job > 10 AND total_of_job.number_of_job <= 50 THEN 'normal'
            ELSE 'large'
        END AS size_of_company

    FROM (
        SELECT
            COUNT(job_id) AS number_of_job,
            company_id AS id_of_company
        FROM job_postings_fact
        GROUP BY company_id
    ) AS total_of_job
)

GROUP BY size_of_company
ORDER BY number_of_company_size DESC;
```

---

### 🔍 Pourquoi utiliser `CASE` ?

L'expression `CASE` fonctionne comme une **structure conditionnelle** (similaire à un `if/else` en programmation). Elle évalue une condition et retourne une valeur selon le résultat. Ici, elle permet de transformer un nombre brut d'offres en une **catégorie lisible** (`small`, `normal`, `large`), ce qui facilite l'analyse et la visualisation.

### 🔍 Pourquoi utiliser une Sous-requête ici ?

La sous-requête est utilisée pour **pré-calculer le nombre d'offres par entreprise** avant d'appliquer la logique de classification. SQL ne permet pas d'utiliser directement un alias de colonne calculé (`COUNT(job_id)`) dans le même `SELECT` où il est défini. La sous-requête contourne cette limitation et permet une écriture propre et structurée.

---

### 📊 Résultats

**Catégorie par entreprise (extrait) :**

| number_of_job | id_of_company | size_of_company |
|---------------|---------------|-----------------|
| 2 | 164125 | small |
| 16 | 47216 | normal |
| 23 | 352914 | normal |
| ... | ... | ... |

**Nombre d'entreprises par catégorie :**

| number_of_company_size | size_of_company |
|------------------------|-----------------|
| 128 521 | small |
| 9 651 | normal |
| 1 861 | large |

> La grande majorité des entreprises (**128 521**) sont de petite taille, tandis que seulement **1 861** sont de grandes entreprises — ce qui reflète la réalité du marché du travail.

---

## 🛠️ Technologies Utilisées

- **PostgreSQL / SQL** — Langage de requête structuré
- **Concepts clés** : `LEFT JOIN`, `GROUP BY`, `ORDER BY`, Sous-requêtes (Subqueries), `CASE WHEN`

---

## 📁 Structure du Projet

```
📦 sql-job-analysis
 ┣ 📄 README.md
 ┣ 📄 Practice Problem.PNG
 ┣ 📄 1.PNG
 ┣ 📄 2.PNG
 ┗ 📄 3.PNG
```

---

## 👤 Auteur

Projet réalisé dans le cadre d'un parcours d'apprentissage et de maîtrise du **SQL pour l'analyse de données**.
