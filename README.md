# Prompt Exemple Bank

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

> **Prompt Exemple Bank** est une collection structurée de prompts, d'agents et de guides d'orchestration illustrant les meilleures pratiques que j'applique au quotidien pour concevoir des solutions IA robustes et auditables.

---

## 🗂️ Table des Matières
1. [Contexte](#contexte)
2. [Fonctionnalités Clés](#fonctionnalités-clés)
3. [Structure du Dépôt](#structure-du-dépôt)
4. [Prise en Main Rapide](#prise-en-main-rapide)
5. [Exemples Remarquables](#exemples-remarquables)
6. [Contribuer](#contribuer)
7. [Licence](#licence)

---

## Contexte
La banque d'exemples couvre l'intégralité d'un **workflow d'orchestration multi-agents** allant de la capture d'information (actors) à l'évaluation croisée (critics) jusqu'à la génération d'un livrable final. Elle démontre :

* La rédaction de prompts **défensifs** (SecurityDirective, guardrails).
* La **séparation** claire entre configuration YAML et documentation Markdown.
* L'usage de **LangGraph** pour modéliser des flux cycliques et stateful.
* Des guides de **déploiement** (Docker/Kubernetes) et de **test QA** complets.

## Fonctionnalités Clés
| Catégorie | Description |
|-----------|-------------|
| Orchestration | Exemple pas-à-pas d'un graphe LangGraph avec router, actors, critics et state object. |
| Prompts | 16 fichiers commentés illustrant la rédaction de prompts sûrs et réutilisables. |
| Documentation | Spécifications fonctionnelles & techniques (FS, architecture, QA, déploiement). |
| Sécurité | Directive ABSOLUTE, filtrage de contenus toxiques, stratégie d'évitement des fuites de prompt. |
| CI/QA | Plan de tests unitaire, intégration, E2E et non-fonctionnels incluant scénarios d'abuse-mode. |

## Structure du Dépôt
```
prompt_exemple_bank/
├── Orchestration/
│   ├── 00_orchestrateur.md                 # Agent maître (router)
│   ├── actors/                             # Agents de production (scribe, analyst, …)
│   ├── critics/                            # Comité d'évaluation (feasibility, ethics, …)
│   └── docs/                               # Spécifications, QA, déploiement
├── example00_MVP_TestAgent.md              # Exemple de stratégie de test de portefeuille dans un Workflow linéaire
├── example01_ConseillerBudgetPersonnel.md  # Agent d'analyse budgétaire StandAlone
└── README.md                               # Vous y êtes 😊
```

## Prise en Main Rapide
1. **Cloner** le dépôt :
   ```bash
   git clone https://github.com/MrZzE00/prompt_exemple_bank.git
   cd prompt_exemple_bank
   ```
2. **Explorer** les spécifications :
   ```bash
   open Orchestration/docs/00_SPECIFICATIONS_FONCTIONNELLES.md
   ```
3. **Visualiser** le graphe LangGraph (fichier mermaid) dans `Orchestration/readme.md`.

> Remarque : Le dépôt est **documentaire**. Aucun code exécutable n'est fourni ; l'objectif est de partager des modèles et des bonnes pratiques.

## Exemples Remarquables
| Fichier | Pourquoi c'est intéressant ? |
|---------|-----------------------------|
| `Orchestration/actors/02_marketintel_agent.md` | Montre un prompt avec exigence de citation de sources + schéma JSON strict. |
| `Orchestration/critics/04_EthicsCritic_Agent.md` | Évalue les risques éthiques avec niveaux de risque typés. |
| `Orchestration/docs/05_GUIDE_DEPLOIEMENT_MONITORING.md` | Décrit un pipeline complet Docker → Kubernetes avec migrations DB & probes. |

> Avant de soumettre une PR, merci de créer une branche et de vérifier que vos fichiers respectent le style YAML/Markdown du projet.

## Licence
Ce projet est distribué sous licence **MIT** – voir le fichier [LICENSE](LICENSE) pour plus de détails.