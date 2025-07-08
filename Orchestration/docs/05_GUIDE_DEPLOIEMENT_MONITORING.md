# 06. Guide de Déploiement et Monitoring

| ID du Document | CSF-OPS-v1.0 |
| :--- | :--- |
| **Titre** | Guide de Déploiement & Monitoring - Cognitive Sprint Facilitator |
| **Version** | 1.0 |
| **Date** | 2025-07-08 |
| **Auteur** | PromptArchitectPrime |

## 1. Vue d'Ensemble

Ce document fournit les instructions techniques pour le déploiement, la configuration et la surveillance de l'application "Cognitive Sprint Facilitator". Il s'adresse aux ingénieurs responsables de l'infrastructure et des opérations.

## 2. Prérequis de l'Environnement

### 2.1. Stack Technique

* **Langage :** Python 3.11+
* **Frameworks :**
    * `langchain`, `langgraph`
    * `fastapi` (pour l'API Gateway)
    * `psycopg2-binary` ou équivalent (pour la connexion DB)
* **Base de Données :** PostgreSQL 14+ (recommandé pour le support JSONB)
* **Infrastructure :** Conteneurisation avec Docker est requise. Déploiement sur une plateforme d'orchestration comme Kubernetes est recommandé.
* **Dépendances :** Toutes les dépendances Python sont listées dans le fichier `requirements.txt`.

### 2.2. Services Externes

Un accès réseau sortant est requis pour les services suivants :
* **API OpenAI :** Pour les appels au Core LLM Engine.
* **API LangSmith :** Pour l'observabilité et le traçage.
* **API de Recherche Web :** (ex: Google Search API, Tavily API).

## 3. Configuration de l'Application

L'application est configurée via des variables d'environnement. Un fichier `.env.example` doit être présent à la racine du projet.

```env
# Fichier .env.example

# Clés d'API
OPENAI_API_KEY="sk-..."
LANGCHAIN_API_KEY="ls__..." # Pour LangSmith
SEARCH_API_KEY="..."

# Configuration de la Base de Données
DATABASE_URL="postgresql://user:password@host:port/dbname"

# Configuration de l'Application
LOG_LEVEL="INFO" # "DEBUG" pour le développement, "INFO" ou "WARNING" pour la production
APP_HOST="0.0.0.0"
APP_PORT="8000"

## 4. Procédure de Déploiement

### 4.1. Construction de l'Image Docker

Exécutez :
```bash
docker build -t cognitive-sprint-facilitator:v1.0 .
```

### 4.2. Migration de Base de Données (obligatoire)
Avant de déployer l'application, exécutez les migrations pour garantir que le schéma est à jour :
```bash
# Exemple avec Alembic
alembic upgrade head
```
Dans Kubernetes, vous pouvez créer un **Job init-migration** (voir manifest `job-db-migration.yaml`) qui s'exécute avant le `Deployment` principal.

### 4.3. Déploiement de l'Application
1. Poussez l'image vers votre registry (ex : Docker Hub, AWS ECR).
2. Appliquez le manifeste `deployment.yaml`. Celui-ci doit inclure :
   * **readinessProbe** :
     ```yaml
     readinessProbe:
       httpGet:
         path: /health
         port: 8000
       periodSeconds: 10
       failureThreshold: 3
     ```
   * **livenessProbe** identique ou sur `/live` pour redémarrer le pod en cas de blocage.
3. Assurez-vous que le champ `envFrom: .env` ou `env` contient `DATABASE_URL` et autres variables.

### 4.4. Vérification Post-Déploiement
1. Vérifiez que le Job de migration est **Succeeded**.
2. Attendez que le `Deployment` passe à l'état **Ready** (readinessProbe).
3. Exécutez un **smoke test** via `/health` puis création d'une session de sprint.

## 5. Stratégie de Logging et Monitoring

### 5.1. Logging
Format : Tous les logs doivent être au format JSON pour une ingestion facile par des outils comme Fluentd ou Logstash.

Événements à Journaliser :

INFO : Démarrage de l'application, création de nouvelles sessions.

WARNING : Erreur contrôlée (ex: API externe indisponible), tentative de prompt injection détectée.

ERROR : Erreur inattendue, échec d'un nœud dans le graphe, échec de connexion à la base de données.

DEBUG : Traces détaillées du cheminement dans le graphe (pour le développement).

### 5.2. Monitoring (Tableau de Bord)
Un tableau de bord (ex: Grafana, Datadog) doit être mis en place pour suivre les métriques clés :

Métriques Système :

Utilisation CPU et Mémoire du conteneur de l'application.

Nombre de connexions actives à la base de données.

Métriques Applicatives :

Taux d'Erreur API : Pourcentage de requêtes HTTP 5xx.

Latence API (p95) : Latence de bout en bout pour les requêtes simples et complexes.

Débit : Nombre de requêtes par minute.

Métriques Métier :

Nombre de sprints actifs.

Coût total des appels à l'API OpenAI par jour.

Nombre d'exécutions de chaque type de nœud (pour identifier les plus utilisés).

### 5.3. Alerting
Des alertes doivent être configurées pour les seuils critiques suivants :

Taux d'Erreur API > 5% sur une fenêtre de 5 minutes.

Latence p95 > 25 secondes sur les cycles complexes.

Log de Sécurité : Toute détection d'une tentative de prompt injection.

Utilisation CPU/Mémoire > 85% pendant plus de 10 minutes.

### 5.4. Observabilité avec LangSmith
LangSmith est un outil de monitoring de première ligne pour cette application. Il est non négociable pour l'analyse en production. L'équipe d'astreinte doit l'utiliser pour :

Déboguer les Échecs : Analyser la trace complète de chaque exécution de graphe qui a échoué.

Analyser la Latence : Identifier quel nœud ou quel appel LLM est un goulot d'étranglement.

Suivre les Coûts : Surveiller la consommation de tokens pour chaque étape du workflow.


---