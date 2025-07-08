## Table des Matières

- [01. Introduction](#01-introduction)
- [02. L'Objet d'État (`SprintState`)](#02-lobjet-dtat-sprintstate)
- [03. Schéma de la Base de Données (Persistance)](#03-schma-de-la-base-de-donnes-persistance)
- [04. Diagramme des Relations entre Entités](#04-diagramme-des-relations-entre-entits)

---
source_file: 03_MODELE_DE_DONNEES.md
---


# 04. Modèle de Données (State & Persistance)

| ID du Document | CSF-DM-v1.0                                      |
| :---           | :---                                             |
| **Titre**      | Modèle de Données - Cognitive Sprint Facilitator |
| **Version**    | 1.0                                              |
| **Date**       | 2025-07-08                                       |
| **Auteur**     | PromptArchitectPrime                             |

## 01. Introduction

Ce document définit les structures de données principales du système. Il distingue deux concepts clés :
1.  **L'Objet d'État (`SprintState`) :** La structure de données en mémoire, volatile, qui est passée entre les nœuds du graphe LangGraph à chaque étape d'une session.
2.  **Le Schéma de Persistance :** La structure des tables de la base de données qui assure la sauvegarde de l'état entre les sessions et la traçabilité des opérations.

## 02. L'Objet d'État (`SprintState`)

Le `SprintState` est le "sang" du système. Il est représenté ici sous une forme similaire à un `TypedDict` Python pour plus de clarté.

```python
# Pseudo-code pour la structure du SprintState
from typing import TypedDict, List, Dict, Any

class Critique(TypedDict):
    critic: str  # ex: "Feasibility", "Desirability"
    score: int | None
    risk_level: str | None # "Low", "Medium", "High"
    justification: str
    # ... autres champs spécifiques au critic

class SprintState(TypedDict):
    # Métadonnées de session
    session_id: str
    user_request: str
    history: List[str]

    # Données du sprint
    sprint_steps: Dict[str, Dict[str, Any]] # ex: {'customer': {'ideas': [...], 'decision': '...'}}
    
    # Données en cours de traitement
    search_query: str | None
    proposals_to_evaluate: List[Dict[str, Any]]
    critiques: Dict[str, List[Critique]] # ex: {'proposal_id_1': [critique_1, ...]}
    
    # Livrables finaux
    final_report: str | None
    error_message: str | None

## 03. Schéma de la Base de Données (Persistance)
Une base de données relationnelle (type PostgreSQL) est recommandée pour garantir l'intégrité et la traçabilité.

Table: sprints
Stocke les informations générales de chaque session d'atelier.
| Nom de Colonne  | Type      | Contraintes | Description                                                 |
| :---            | :---      | :---        | :---                                                        |
| sprint_id       | UUID      | PRIMARY KEY | Identifiant unique de la session.                           |
| creation_date   | TIMESTAMP | NOT NULL    | Date de début du sprint.                                    |
| last_updated    | TIMESTAMP | NOT NULL    | Date de la dernière interaction.                            |
| decider_name    | TEXT      |             | Nom du "Decider" humain.                                    |
| full_state_json | JSONB     |             | Sauvegarde complète de l'objet SprintState pour la reprise. |

Table: requests
Trace chaque requête utilisateur pour un audit complet.
| Nom de Colonne | Type      | Contraintes | Description                                     |
| :---           | :---      | :---        | :---                                            |
| request_id     | UUID      | PRIMARY KEY | Identifiant unique de la requête.               |
| sprint_id      | UUID      | FOREIGN KEY | Lie la requête à un sprint.                     |
| timestamp      | TIMESTAMP | NOT NULL    | Horodatage de la requête.                       |
| user_prompt    | TEXT      |             | Le contenu exact de la requête du facilitateur. |

Table: proposals
Trace chaque proposition générée par un agent "Actor".
| Nom de Colonne | Type  | Contraintes | Description                                                |
| :---           | :---  | :---        | :---                                                       |
| proposal_id    | UUID  | PRIMARY KEY | Identifiant unique de la proposition.                      |
| request_id     | UUID  | FOREIGN KEY | Lie la proposition à la requête qui l'a initiée.           |
| actor_node     | TEXT  |             | Nom du nœud qui a généré la proposition (ex: AnalystNode). |
| content        | JSONB |             | Le contenu de la proposition.                              |

Table: critiques
Trace chaque évaluation produite par un agent "Critic". Ceci est la table clé pour votre exigence de traçabilité.
| Nom de Colonne | Type  | Contraintes | Description                                       |
| :---           | :---  | :---        | :---                                              |
| critique_id    | UUID  | PRIMARY KEY | Identifiant unique de la critique.                |
| proposal_id    | UUID  | FOREIGN KEY | Lie la critique à la proposition qu'elle évalue.  |
| critic_node    | TEXT  |             | Nom du nœud "Critic" (ex: FeasibilityCriticNode). |
| evaluation     | JSONB |             | L'objet JSON complet retourné par le "Critic".    |

## 04. Diagramme des Relations entre Entités
Extrait de code

erDiagram
    SPRINTS ||--o{ REQUESTS : "contient"
    REQUESTS ||--o{ PROPOSALS : "génère"
    PROPOSALS ||--o{ CRITIQUES : "reçoit"

    SPRINTS {
        UUID sprint_id PK
        TIMESTAMP creation_date
        TIMESTAMP last_updated
        TEXT decider_name
        JSONB full_state_json
    }
    REQUESTS {
        UUID request_id PK
        UUID sprint_id FK
        TIMESTAMP timestamp
        TEXT user_prompt
    }
    PROPOSALS {
        UUID proposal_id PK
        UUID request_id FK
        TEXT actor_node
        JSONB content
    }
    CRITIQUES {
        UUID critique_id PK
        UUID proposal_id FK
        TEXT critic_node
        JSONB evaluation
    }
