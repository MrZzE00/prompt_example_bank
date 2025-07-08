# 03. Spécifications Détaillées des Nœuds (Agents)

| ID du Document | CSF-NA-v1.0 |
| :--- | :--- |
| **Titre** | Spécifications des Nœuds Agents |
| **Version** | 1.0 |
| **Date** | 2025-07-08 |
| **Auteur** | PromptArchitectPrime |

## 1. Introduction

Ce document fournit les spécifications techniques pour l'implémentation de chaque **Nœud** (Agent) au sein du graphe LangGraph. Chaque nœud est une fonction Python qui prend l'objet `SprintState` en entrée et retourne le `SprintState` mis à jour.

## 2. Agents de Production (Actors)

Ces nœuds sont responsables de la création et de la structuration de l'information.

### 2.1. ScribeNode

* **Mission :** Transcrire fidèlement les contributions brutes de l'équipe (idées, votes) dans l'état du sprint.
* **Inputs Requis (du `State`) :** `state['user_request']` (le dernier message du facilitateur).
* **Logique d'Exécution :**
    1.  Analyser le texte d'entrée pour identifier s'il s'agit d'une liste d'idées, d'un vote, ou d'une décision finale.
    2.  Invoquer le LLM avec le prompt `prompts/scribe_prompt.yaml` pour transformer le texte en JSON structuré.
    3.  Ne jamais interpréter ou corriger le contenu.
* **Prompt Associé :** `prompts/scribe_prompt.yaml`
* **Outils Externes :** Aucun.
* **Output (Mise à jour du `State`) :** Met à jour la section de l'étape en cours.
    * Exemple : `state['sprint_steps']['customer']['ideas'] = ["idée A", "idée B"]`

### 2.2. AnalystNode

* **Mission :** Générer des insights, des synthèses thématiques ou des propositions en se basant sur les données déjà capturées.
* **Inputs Requis (du `State`) :** `state['sprint_steps']` (pour accéder aux données des étapes précédentes), `state['user_request']` (pour comprendre la demande d'analyse).
* **Logique d'Exécution :**
    1.  Lire les données pertinentes des étapes précédentes (ex: avantages, problèmes).
    2.  Invoquer le LLM avec le prompt `prompts/analyst_prompt.yaml` et les données contextuelles.
    3.  Formuler une analyse concise ou une liste de propositions concrètes.
* **Prompt Associé :** `prompts/analyst_prompt.yaml`
* **Outils Externes :** Aucun.
* **Output (Mise à jour du `State`) :** Ajoute une analyse ou des propositions à l'étape en cours.
    * Exemple : `state['sprint_steps']['differentiation']['analysis_output'] = {"type": "proposal", "content": ["Axe 1", "Axe 2"]}`

### 2.3. MarketIntelNode

* **Mission :** Enrichir le `State` avec des données externes factuelles via la recherche web.
* **Inputs Requis (du `State`) :** `state['search_query']` (une requête de recherche claire définie par le `Router`).
* **Logique d'Exécution :**
    1.  Appeler l'outil `web_search` avec la requête fournie.
    2.  Invoquer le LLM avec le prompt `prompts/marketintel_prompt.yaml` et les résultats de la recherche pour synthétiser et extraire les informations clés.
    3.  Citer impérativement les sources (URL) pour chaque information.
* **Prompt Associé :** `prompts/marketintel_prompt.yaml`
* **Outils Externes :** `tools.web_search`
* **Output (Mise à jour du `State`) :** Ajoute les résultats de la recherche.
    * Exemple : `state['market_intel_results'] = [{"summary": "...", "source": "...", "snippet": "..."}]`

### 2.4. ChroniclerNode

* **Mission :** Compiler le rapport final "Founding Hypothesis" à la fin du sprint.
* **Inputs Requis (du `State`) :** L'objet `State` complet, en particulier `state['sprint_steps']` contenant toutes les décisions.
* **Logique d'Exécution :**
    1.  Parcourir `state['sprint_steps']` pour extraire chaque décision finale (`customer`, `problem`, `approach`, etc.).
    2.  Invoquer le LLM avec le prompt `prompts/chronicler_prompt.yaml` pour remplir un template de rapport avec les décisions extraites.
* **Prompt Associé :** `prompts/chronicler_prompt.yaml`
* **Outils Externes :** Aucun.
* **Output (Mise à jour du `State`) :** Ajoute le rapport final au `State`.
    * Exemple : `state['final_report'] = "## Founding Hypothesis\n**Customer:** Enfants 8-12\n..."`

## 3. Comité d'Évaluation (Critics)

Chaque "Critic" est un nœud qui reçoit une proposition à évaluer et retourne une évaluation JSON structurée.

* **Input Commun :** `state['proposals_to_evaluate']`
* **Output Commun :** Ajoute son évaluation à `state['critiques'][proposal_id]`

### 3.1. FeasibilityCriticNode

* **Mission :** Évaluer la réalisabilité technique et opérationnelle.
* **Prompt Associé :** `prompts/critics/feasibility_prompt.yaml`
* **Output (JSON) :** `{ "critic": "Feasibility", "score": 1-10, "justification": "...", "key_risks": [...] }`

### 3.2. DesirabilityCriticNode

* **Mission :** Évaluer l'attrait de la proposition pour le client final.
* **Prompt Associé :** `prompts/critics/desirability_prompt.yaml`
* **Output (JSON) :** `{ "critic": "Desirability", "score": 1-10, "justification": "...", "key_benefits": [...] }`

### 3.3. StrategyCriticNode

* **Mission :** Évaluer la force stratégique et l'avantage concurrentiel.
* **Prompt Associé :** `prompts/critics/strategy_prompt.yaml`
* **Output (JSON) :** `{ "critic": "Strategy", "score": 1-10, "justification": "...", "competitive_advantage": "..." }`

### 3.4. FinancialCriticNode

* **Mission :** Évaluer la viabilité économique de la proposition.
* **Prompt Associé :** `prompts/critics/financial_prompt.yaml`
* **Output (JSON) :** `{ "critic": "Financial", "score": 1-10, "justification": "...", "business_model_snapshot": "..." }`

### 3.5. MarketingCriticNode

* **Mission :** Évaluer le potentiel marketing et la facilité de mise sur le marché.
* **Prompt Associé :** `prompts/critics/marketing_prompt.yaml`
* **Output (JSON) :** `{ "critic": "Marketing", "score": 1-10, "justification": "...", "potential_channels": [...] }`

### 3.6. LegalCriticNode

* **Mission :** Identifier les risques légaux et réglementaires potentiels.
* **Prompt Associé :** `prompts/critics/legal_prompt.yaml`
* **Output (JSON) :** `{ "critic": "Legal", "risk_level": "Low/Medium/High", "identified_topics": [...], "recommendation": "..." }`

### 3.7. OperationsCriticNode

* **Mission :** Évaluer la complexité de l'exploitation à grande échelle.
* **Prompt Associé :** `prompts/critics/operations_prompt.yaml`
* **Output (JSON) :** `{ "critic": "Operations", "score": 1-10, "justification": "...", "key_operational_challenges": [...] }`

### 3.8. EthicsCriticNode

* **Mission :** Identifier les risques éthiques et les potentiels impacts négatifs.
* **Prompt Associé :** `prompts/critics/ethics_prompt.yaml`
* **Output (JSON) :** `{ "critic": "Ethics", "risk_level": "Low/Medium/High", "ethical_concerns": [...], "recommendation": "..." }`