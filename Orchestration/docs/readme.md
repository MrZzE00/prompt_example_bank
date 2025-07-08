# Dossier de Spécifications Techniques : Cognitive Sprint Facilitator

| Statut   | Version | Date       | Auteur               |
| Finalisé | 1.0     | 2025-07-08 | PromptArchitectPrime |

## Table des Matières
1.  [Spécifications Fonctionnelles Détaillées](#1-spécifications-fonctionnelles-détaillées)
2.  [Schéma d'Architecture Technique](#2-schéma-darchitecture-technique)
3.  [Spécifications Détaillées des Nœuds (Agents)](#3-spécifications-détaillées-des-nœuds-agents)
4.  [Modèle de Données (State & Persistance)](#4-modèle-de-données-state--persistance)
5.  [Plan de Tests et Stratégie de QA](#5-plan-de-tests-et-stratégie-de-qa)
6.  [Guide de Déploiement et Monitoring](#6-guide-de-déploiement-et-monitoring)

---

## 1. Spécifications Fonctionnelles Détaillées

### 1.1. Vision du Produit

Cognitive Sprint Facilitator est une solution logicielle d'assistance par IA conçue pour augmenter la capacité cognitive des équipes lors d'ateliers d'innovation structurés. Via une interface conversationnelle, l'outil assiste le facilitateur humain en automatisant la capture d'idées, en enrichissant les discussions avec des données et des analyses pertinentes, et en consolidant les décisions dans un rapport final structuré.

### 1.2. Persona Utilisateur Principal

L'utilisateur principal du système est le **"Facilitateur Humain"**.

* **Rôle :** Il est responsable de guider l'équipe humaine à travers les exercices de l'atelier (ex: Foundation Sprint).
* **Objectifs :** Maximiser la créativité et la pertinence des décisions de l'équipe, tout en respectant un timing serré.
* **Défis (Pain Points) :**
    * La difficulté de prendre des notes exhaustives tout en animant activement.
    * Le risque de passer à côté d'angles morts (concurrents oubliés, biais cognitifs).
    * Le temps passé à synthétiser et à formater les livrables après l'atelier.

### 1.3. Exigences Fonctionnelles (User Stories)

#### Epic 1 : Gestion du Sprint

**FS-01 : Démarrage d'une Session de Sprint**
* **En tant que** Facilitateur,
* **Je veux** démarrer une nouvelle session de sprint,
* **Afin de** préparer un environnement de travail unique et persistant pour mon atelier.
* **Critères d'Acceptation :**
    1.  Quand j'initie la session, le système me salue et confirme le début du "Cognitive Sprint Facilitator".
    2.  Le système confirme la création d'une session unique qui sera maintenue pendant 48 heures.
    3.  Toutes les interactions suivantes sont enregistrées dans le contexte de cette session.

#### Epic 2 : Capture et Synthèse des Données

**FS-02 : Capture des Contributions de l'Équipe**
* **En tant que** Facilitateur,
* **Je veux** dicter les idées ou les votes de mon équipe à l'IA,
* **Afin de** m'assurer que toutes les contributions sont capturées fidèlement sans que j'aie à prendre des notes.
* **Critères d'Acceptation :**
    1.  Je peux soumettre une liste d'idées brutes (ex: "Voici les clients : A, B, C").
    2.  L'agent `ScribeNode` confirme la capture en restituant les idées sous forme de liste structurée.
    3.  Les données capturées sont stockées dans l'état de la session sans altération.

**FS-03 : Synthèse Thématique des Idées**
* **En tant que** Facilitateur,
* **Je veux** demander une synthèse d'une longue liste d'idées,
* **Afin de** rapidement identifier les grands thèmes et structurer la discussion avant un vote.
* **Critères d'Acceptation :**
    1.  Je peux demander : "Maestro, fais une synthèse des thèmes pour ces idées."
    2.  L'agent `AnalystNode` retourne une réponse qui regroupe les idées similaires sous des "clusters" thématiques.
    3.  Les idées originales ou "outliers" sont mises en évidence.

#### Epic 3 : Augmentation Cognitive

**FS-04 : Enrichissement par Recherche Externe**
* **En tant que** Facilitateur,
* **Je veux** demander des informations de marché sur un sujet précis (ex: un concurrent),
* **Afin d'**ancrer la discussion de l'équipe dans la réalité externe.
* **Critères d'Acceptation :**
    1.  Je peux demander : "Maestro, cherche des informations sur la stratégie de [Concurrent X]".
    2.  L'agent `MarketIntelNode` exécute une recherche web.
    3.  La réponse est un résumé factuel qui cite ses sources (URL).

**FS-05 : Évaluation par le Comité de "Critics"**
* **En tant que** Facilitateur,
* **Je veux** soumettre une proposition pour une évaluation multi-facettes,
* **Afin d'**obtenir une analyse à 360° (faisabilité, désirabilité, stratégie, etc.) pour éclairer une décision complexe.
* **Critères d'Acceptation :**
    1.  Je peux soumettre une idée claire à l'évaluation (ex: "Évalue cette approche : 'Créer une app mobile'").
    2.  Le système invoque le `CriticCommitteeNode`.
    3.  Le rapport final contient les évaluations structurées (score/risque, justification) de chaque "Critic" pertinent.

#### Epic 4 : Finalisation

**FS-06 : Génération du Rapport Final**
* **En tant que** Facilitateur,
* **Je veux** générer le document "Founding Hypothesis" à la fin du sprint,
* **Afin d'**obtenir un livrable propre et complet sans effort de mise en forme manuel.
* **Critères d'Acceptation :**
    1.  Je peux lancer la génération avec une commande simple ("Maestro, génère le rapport final").
    2.  L'agent `ChroniclerNode` récupère toutes les décisions finales archivées.
    3.  Le document final est produit au format Markdown, complété avec les bonnes informations.

---

## 2. Schéma d'Architecture Technique

### 2.1. Vue d'Ensemble & Paradigme Architectural

Le système est implémenté comme une **application multi-agents stateful**. L'orchestration est gérée par un graphe d'états dirigé, construit avec la librairie **LangGraph**, choisie pour sa capacité à gérer des workflows **cycliques** et **collaboratifs**.

### 2.2. Composants Clés

1.  **Frontend (UI) :** Interface de chat web servant de point d'interaction unique.
2.  **API Gateway :** Serveur backend qui gère les sessions et transmet les requêtes au moteur LangGraph.
3.  **LangGraph Engine :** Cœur du système, composé de :
    * **State Object (`SprintState`) :** Objet de données centralisé représentant la mémoire du sprint.
    * **Nodes (Nœuds / Agents) :** Fonctions encapsulant la logique de chaque agent spécialisé.
    * **Router (Arêtes Conditionnelles) :** Logique de décision qui dirige le flux entre les nœuds.
4.  **Services Externes :** API tierces (recherche web, génération de graphiques).
5.  **Stockage Persistent :** Base de données relationnelle (ex: PostgreSQL) pour la persistance de l'état et Vector Store pour la recherche sémantique.
6.  **Observabilité (LangSmith) :** Outil critique pour tracer, déboguer et monitorer les exécutions du graphe.

### 2.3. Schéma du Flux de Données Principal

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

## 3. Spécifications Détaillées des Nœuds (Agents)
Ce document fournit le contrat d'interface pour chaque nœud du graphe.

### 3.1. Agents de Production (Actors)
ScribeNode
Mission : Transcrire fidèlement les contributions brutes de l'équipe.

Inputs Requis : state['user_request'].

Logique d'Exécution : Invoquer le LLM avec le prompt prompts/scribe_prompt.yaml pour transformer le texte en JSON.

Output : Met à jour state['sprint_steps'][current_step].

AnalystNode
Mission : Générer des insights et des propositions à partir des données capturées.

Inputs Requis : state['sprint_steps'], state['user_request'].

Logique d'Exécution : Invoquer le LLM avec le prompt prompts/analyst_prompt.yaml et les données contextuelles.

Output : Met à jour state['sprint_steps'][current_step]['analysis_output'].

MarketIntelNode
Mission : Enrichir le State avec des données externes factuelles.

Inputs Requis : state['search_query'].

Logique d'Exécution : Appeler l'outil web_search, puis synthétiser les résultats avec le LLM via le prompt prompts/marketintel_prompt.yaml.

Output : Met à jour state['market_intel_results'].

ChroniclerNode
Mission : Compiler le rapport final "Founding Hypothesis".

Inputs Requis : L'objet State complet.

Logique d'Exécution : Extraire toutes les décisions finales de state['sprint_steps'] et remplir un template de rapport.

Output : Met à jour state['final_report'].

### 3.2. Comité d'Évaluation (Critics)
Chaque "Critic" reçoit une proposition et retourne une évaluation JSON structurée.

Input Commun : state['proposals_to_evaluate']

Output Commun : Ajoute son évaluation à state['critiques'][proposal_id]

FeasibilityCriticNode
Mission : Évaluer la réalisabilité technique.

Output (JSON) : { "critic": "Feasibility", "score": 1-10, "justification": "...", "key_risks": [...] }

DesirabilityCriticNode
Mission : Évaluer l'attrait pour le client.

Output (JSON) : { "critic": "Desirability", "score": 1-10, "justification": "...", "key_benefits": [...] }

StrategyCriticNode
Mission : Évaluer la force stratégique.

Output (JSON) : { "critic": "Strategy", "score": 1-10, "justification": "...", "competitive_advantage": "..." }

(Les autres agents Critics - Financial, Marketing, Legal, Operations, Ethics - suivent une structure similaire avec leur propre mission et format de sortie JSON.)

## 4. Modèle de Données (State & Persistance)

### 4.1. L'Objet d'État (SprintState)
Structure de données en mémoire passée entre les nœuds.

Python

# Pseudo-code pour la structure du SprintState
class SprintState(TypedDict):
    session_id: str
    user_request: str
    sprint_steps: Dict[str, Dict[str, Any]]
    proposals_to_evaluate: List[Dict[str, Any]]
    critiques: Dict[str, List[Dict[str, Any]]]
    final_report: str | None
    # ... autres champs

### 4.2. Schéma de la Base de Données (Persistance)
Structure relationnelle pour la sauvegarde et la traçabilité.

Table sprints : sprint_id (PK), creation_date, last_updated, full_state_json.

Table requests : request_id (PK), sprint_id (FK), timestamp, user_prompt.

Table proposals : proposal_id (PK), request_id (FK), actor_node, content.

Table critiques : critique_id (PK), proposal_id (FK), critic_node, evaluation (JSONB).

## 5. Plan de Tests et Stratégie de QA

### 5.1. Niveaux de Test
Tests Unitaires : Chaque nœud est testé de manière isolée en moquant les appels LLM et DB.

Tests d'Intégration : Des séquences de nœuds (ex: Actor -> Critic) sont testées pour valider les transitions et l'intégration avec la DB.

Tests End-to-End (E2E) : Des scénarios utilisateurs complets sont simulés via l'UI pour valider le "Golden Path".

### 5.2. Tests Non-Fonctionnels
Performance : Tests de charge sur les requêtes simples et complexes pour valider les cibles de latence.

Sécurité : Suite de tests de "prompt injection" pour vérifier la robustesse des garde-fous.

### 5.3. Outils
Frameworks : pytest (unitaire/intégration), Playwright (E2E).

Observabilité : L'utilisation de LangSmith est obligatoire pour le traçage et le débogage.

## 6. Guide de Déploiement et Monitoring

### 6.1. Prérequis
Stack : Python 3.11+, Docker, PostgreSQL 14+.

Clés d'API : OPENAI_API_KEY, LANGCHAIN_API_KEY, SEARCH_API_KEY.

### 6.2. Procédure de Déploiement
Build : Construire l'image Docker de l'application.

Configure : Appliquer les migrations de base de données.

Deploy : Déployer l'image sur une plateforme de conteneurs (ex: Kubernetes).

Verify : Vérifier le point de terminaison /health et exécuter un test de fumée.

### 6.3. Monitoring
Tableau de Bord : Un dashboard (ex: Grafana) doit suivre les métriques système (CPU/RAM), applicatives (erreurs, latence) et métier (coût, sessions actives).

Alertes : Des alertes doivent être configurées pour les seuils critiques (taux d'erreur > 5%, latence p95 > 25s, détection d'attaque de prompt).