## Table des Matières

- [01. Vision du Produit](#01-vision-du-produit)
- [02. Persona Utilisateur Principal](#02-persona-utilisateur-principal)
- [03. Exigences Fonctionnelles (User Stories)](#03-exigences-fonctionnelles-user-stories)
  - [Epic 1 : Gestion du Sprint](#epic-1--gestion-du-sprint)
  - [Epic 2 : Capture et Synthèse des Données](#epic-2--capture-et-synthse-des-donnes)
  - [Epic 3 : Augmentation Cognitive](#epic-3--augmentation-cognitive)
  - [Epic 4 : Finalisation](#epic-4--finalisation)
- [04. Glossaire des Nœuds du Système](#04-glossaire-des-nuds-du-systme)

---
source_file: 00_SPECIFICATIONS_FONCTIONNELLES.md
---


# 01. Spécifications Fonctionnelles Détaillées

| ID du Document | CSF-FS-v1.0                                                  |
| :---           | :---                                                         |
| **Titre**      | Spécifications Fonctionnelles - Cognitive Sprint Facilitator |
| **Version**    | 1.0                                                          |
| **Date**       | 2025-07-08                                                   |
| **Auteur**     | PromptArchitectPrime                                         |

## 01. Vision du Produit

Cognitive Sprint Facilitator est une solution logicielle d'assistance par IA conçue pour augmenter la capacité cognitive des équipes lors d'ateliers d'innovation structurés. Via une interface conversationnelle, l'outil assiste le facilitateur humain en automatisant la capture d'idées, en enrichissant les discussions avec des données et des analyses pertinentes, et en consolidant les décisions dans un rapport final structuré.

## 02. Persona Utilisateur Principal

L'utilisateur principal du système est le **"Facilitateur Humain"**.

* **Rôle :** Il est responsable de guider l'équipe humaine à travers les exercices de l'atelier (ex: Foundation Sprint).
* **Objectifs :** Maximiser la créativité et la pertinence des décisions de l'équipe, tout en respectant un timing serré.
* **Défis (Pain Points) :**
    * La difficulté de prendre des notes exhaustives tout en animant activement.
    * Le risque de passer à côté d'angles morts (concurrents oubliés, biais cognitifs).
    * Le temps passé à synthétiser et à formater les livrables après l'atelier.

## 03. Exigences Fonctionnelles (User Stories)

### Epic 1 : Gestion du Sprint

#### FS-01 : Démarrage d'une Session de Sprint
* **En tant que** Facilitateur,
* **Je veux** démarrer une nouvelle session de sprint,
* **Afin de** préparer un environnement de travail unique et persistant pour mon atelier.
* **Critères d'Acceptation :**
    1.  Quand j'initie la session, le système me salue et confirme le début du "Cognitive Sprint Facilitator".
    2.  Le système confirme la création d'une session unique qui sera maintenue pendant 48 heures.
    3.  Toutes les interactions suivantes sont enregistrées dans le contexte de cette session.

### Epic 2 : Capture et Synthèse des Données

#### FS-02 : Capture des Contributions de l'Équipe
* **En tant que** Facilitateur,
* **Je veux** dicter les idées ou les votes de mon équipe à l'IA,
* **Afin de** m'assurer que toutes les contributions sont capturées fidèlement sans que j'aie à prendre des notes.
* **Critères d'Acceptation :**
    1.  Je peux soumettre une liste d'idées brutes (ex: "Voici les clients : A, B, C").
    2.  L'agent `ScribeNode` confirme la capture en restituant les idées sous forme de liste structurée.
    3.  Les données capturées sont stockées dans l'état de la session sans altération.

#### FS-03 : Synthèse Thématique des Idées
* **En tant que** Facilitateur,
* **Je veux** demander une synthèse d'une longue liste d'idées,
* **Afin de** rapidement identifier les grands thèmes et structurer la discussion avant un vote.
* **Critères d'Acceptation :**
    1.  Je peux demander : "Maestro, fais une synthèse des thèmes pour ces idées."
    2.  L'agent `AnalystNode` retourne une réponse qui regroupe les idées similaires sous des "clusters" thématiques.
    3.  Les idées originales ou "outliers" sont mises en évidence.

### Epic 3 : Augmentation Cognitive

#### FS-04 : Enrichissement par Recherche Externe
* **En tant que** Facilitateur,
* **Je veux** demander des informations de marché sur un sujet précis (ex: un concurrent),
* **Afin d'**ancrer la discussion de l'équipe dans la réalité externe.
* **Critères d'Acceptation :**
    1.  Je peux demander : "Maestro, cherche des informations sur la stratégie de [Concurrent X]".
    2.  L'agent `MarketIntelNode` exécute une recherche web.
    3.  La réponse est un résumé factuel qui cite ses sources (URL).

#### FS-05 : Évaluation par le Comité de "Critics"
* **En tant que** Facilitateur,
* **Je veux** soumettre une proposition pour une évaluation multi-facettes,
* **Afin d'**obtenir une analyse à 360° (faisabilité, désirabilité, stratégie, etc.) pour éclairer une décision complexe.
* **Critères d'Acceptation :**
    1.  Je peux soumettre une idée claire à l'évaluation (ex: "Évalue cette approche : 'Créer une app mobile'").
    2.  Le système invoque le `CriticCommitteeNode`.
    3.  La réponse finale est un rapport consolidé contenant les évaluations structurées (score, justification) de chaque "Critic" pertinent.

### Epic 4 : Finalisation

#### FS-06 : Génération du Rapport Final
* **En tant que** Facilitateur,
* **Je veux** générer le document "Founding Hypothesis" à la fin du sprint,
* **Afin d'**obtenir un livrable propre et complet sans effort de mise en forme manuel.
* **Critères d'Acceptation :**
    1.  Je peux lancer la génération avec une commande simple ("Maestro, génère le rapport final").
    2.  L'agent `ChroniclerNode` récupère toutes les décisions finales archivées dans l'état de la session.
    3.  Le document final est produit au format Markdown, complété avec les bonnes informations aux bons endroits.

## 04. Glossaire des Nœuds du Système

| Nœud                  | Rôle Principal                                                                                                                                                                              |
| ------                | ----------------                                                                                                                                                                            |
| `ScribeNode`          | Capture fidèlement les idées dictées ou les votes et les stocke de manière structurée dans l'état de la session.                                                                            |
| `AnalystNode`         | Regroupe les idées similaires en thèmes et met en évidence les outliers pour faciliter la discussion.                                                                                       |
| `MarketIntelNode`     | Réalise des recherches externes ciblées (web) et renvoie des résumés factuels avec sources citées.                                                                                          |
| `CriticCommitteeNode` | Orchestration parallèle d'une série d'"Agents Critics" (Faisabilité, Désirabilité, Stratégie, Finances, Opérations, etc.) ; agrège leurs évaluations individuelles en un rapport consolidé. |
| `ChroniclerNode`      | Compile les décisions et artefacts de la session pour générer le rapport final formaté.                                                                                                     |
