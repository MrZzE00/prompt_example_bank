## Table des Matières

- [01. Vue d'Ensemble & Paradigme Architectural](#01-vue-densemble--paradigme-architectural)
- [02. Composants Clés](#02-composants-cls)
- [03. Schéma du Flux de Données Principal](#03-schma-du-flux-de-donnes-principal)
- [04. Description d'un Cycle "Actor-Critic"](#04-description-dun-cycle-actor-critic)

---
source_file: 01_ARCHITECTURE_TECHNIQUE.md
---


# 02. Schéma d'Architecture Technique

| ID du Document | CSF-TA-v1.0                                           |
| :---           | :---                                                  |
| **Titre**      | Architecture Technique - Cognitive Sprint Facilitator |
| **Version**    | 1.0                                                   |
| **Date**       | 2025-07-08                                            |
| **Auteur**     | PromptArchitectPrime                                  |

## 01. Vue d'Ensemble & Paradigme Architectural

Le système "Cognitive Sprint Facilitator" est implémenté comme une **application multi-agents stateful**. L'orchestration est gérée par un graphe d'états dirigé, construit avec la librairie **LangGraph**.

Ce paradigme est choisi pour sa capacité à gérer nativement des workflows **cycliques** (ex: évaluation -> révision -> réévaluation) et **collaboratifs** (plusieurs agents agissant sur un état partagé), ce qui est essentiel pour la nature interactive d'un atelier.

## 02. Composants Clés

L'architecture est décomposée en plusieurs services et composants logiques distincts :

1.  **Frontend (UI) :** Une interface de chat web (ex: React, Vue.js) qui sert de point d'interaction unique pour le facilitateur. Elle ne contient aucune logique métier.

2.  **API Gateway :** Un serveur backend (ex: FastAPI, Express.js) qui expose une API REST simple. Il gère l'authentification, les sessions utilisateur et transmet les requêtes au moteur LangGraph.

3.  **LangGraph Engine :** Le cœur du système, qui orchestre le workflow. Il est composé de :
    * **State Object (`SprintState`) :** Un objet de données centralisé (dictionnaire Python ou modèle Pydantic) qui représente l'état complet du sprint (historique, décisions, etc.). Il est passé à chaque nœud à chaque étape.
    * **Nodes (Nœuds / Agents) :** Des fonctions Python qui encapsulent la logique de chaque agent spécialisé (Scribe, Analyst, Critics...). Chaque nœud reçoit le `State`, exécute une tâche, et retourne une mise à jour du `State`.
    * **Router (Arêtes Conditionnelles) :** La logique de décision qui, après l'exécution de chaque nœud, analyse le `State` pour déterminer quel nœud appeler ensuite. C'est l'implémentation du `SprintMaestro_Agent`.

4.  **Services Externes :** Des API tierces appelées par des outils spécifiques au sein des nœuds.
    * API de recherche web (ex: Google Search, Tavily).
    * API de génération de graphiques (ex: Matplotlib/Seaborn via une API, ou un service tiers).

5.  **Stockage Persistent (`Persistent Storage`) :**
    * **Base de Données d'État (ex: PostgreSQL, Redis) :** Pour sauvegarder l'objet `SprintState` entre les sessions et garantir la continuité de l'atelier sur plusieurs jours.
    * **Base de Données Vectorielle (ex: ChromaDB, Pinecone) :** Utilisée spécifiquement par le `Chronicler_Agent` pour la recherche sémantique dans les archives des sprints passés (si cette fonctionnalité est activée).

6.  **Observabilité (LangSmith) :** L'intégration avec LangSmith est une exigence non-fonctionnelle critique pour tracer, déboguer et monitorer chaque exécution du graphe, offrant une visibilité complète sur le comportement des agents.

## 03. Schéma du Flux de Données Principal

Le diagramme suivant illustre comment une requête de l'utilisateur traverse le système LangGraph.

```mermaid
graph TD
    subgraph "Application"
        A[User Input via API] --> B{Router};

        subgraph "LangGraph Engine"
            B -- "Route en fonction du State" --> C[Node (Agent Spécialisé)];
            C -- "Met à jour le State" --> D[(Sprint State)];
            D -- "Retourne le State mis à jour" --> B;
        end

        B -- "Cycle terminé" --> E[Response Formatter];
        E --> F[Output via API];
    end

    subgraph "Infrastructure Externe"
        C -.-> G[Outils / API Externes];
        D <--> H[Base de Données (Persistance)];
    end

    style D fill:#dbf,stroke:#333,stroke-width:2px
    style B fill:#ffe4c4,stroke:#333,stroke-width:2px

## 04. Description d'un Cycle "Actor-Critic"
Un flux de travail complexe comme l'évaluation d'une proposition se déroule comme suit :

Requête : L'utilisateur soumet une proposition à évaluer.

Routage Initial : Le Router reçoit la requête. Il voit dans le State une proposition non évaluée et dirige le flux vers le nœud CriticCommitteeNode.

Exécution Parallèle : Le CriticCommitteeNode invoque plusieurs sous-logiques "Critic" (Feasibility, Desirability, etc.), potentiellement en parallèle.

Mise à Jour du State : Chaque "Critic" exécute son analyse et retourne son évaluation. Le CriticCommitteeNode agrège toutes ces évaluations et les ajoute à state.critiques.

Routage Final : Le Router reçoit le State mis à jour. Constatant que le cycle de critique est terminé, il dirige le flux vers un nœd ResponseFormatterNode.

Réponse : Ce dernier nœud met en forme le rapport consolidé, qui est renvoyé à l'utilisateur. L'état final du SprintState est sauvegardé dans la base de données.
