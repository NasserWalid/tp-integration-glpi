# Projet d'Intégration Logicielle - ITSM & Monitoring

**Membres du groupe : OUOBA Lionel
                      KABORE S Evelyne
                      ZOUNGRANA Abdoul Nasser

## 1. Présentation du projet
Ce projet déploie une infrastructure conteneurisée complète autour de GLPI (Gestion de parc et Helpdesk). Il intègre une base PostgreSQL, un système d'analyse avec Grafana, ainsi qu'une stack de monitoring temps réel (Prometheus + cAdvisor) mesurant la consommation des conteneurs.

## 2. Prérequis
- Docker (v20.10.x minimum)
- Docker Compose (v2.x)
- RAM recommandée : 4 Go minimum
- OS testé : Linux (Ubuntu/Debian) / macOS / WSL2

## 3. Instructions de démarrage
1. Clonez ce dépôt.
2. Copiez le fichier `.env.example` en `.env`.
3. Démarrez l'infrastructure avec la commande :
   ```bash
   docker compose up -d
   ```
4. Attendez ~1 minute le temps que la base de données s'initialise. Vous pouvez suivre les logs avec `docker compose logs -f glpi`.

## 4. URLs d'accès
- **GLPI :** http://localhost:8080
- **Grafana :** http://localhost:3000
- **Prometheus :** http://localhost:9090
- **cAdvisor :** http://localhost:8081

## 5. Identifiants par défaut
- **GLPI :** `glpi` / `glpi` (identifiants par défaut au premier lancement)
- **Grafana :** `admin` / `admin` (configurés via le fichier .env)

## 6. Arrêt et nettoyage
Pour stopper les conteneurs proprement tout en conservant les données :
```bash
docker compose down
```
Pour détruire l'infrastructure **et effacer toutes les bases de données** (nettoyage complet) :
```bash
docker compose down -v
```

## 7. Réponses aux questions Prometheus (Partie 4)

**Q : Quels targets sont configurés et quel est leur statut ?**
Les targets configurées sont `prometheus` (lui-même), `cadvisor`, et `glpi`. Après démarrage, en se rendant sur `http://localhost:9090/targets`, leur statut doit être **UP** (à l'exception de GLPI s'il n'expose pas nativement de endpoint /metrics sans plugin).

**Q : Différence entre scrape_interval et evaluation_interval ?**
- `scrape_interval` : Fréquence à laquelle Prometheus va "aspirer" (scraper) les métriques depuis les targets (ex: toutes les 15s).
- `evaluation_interval` : Fréquence à laquelle Prometheus va calculer et évaluer ses règles d'alertes (Alerting Rules) ou de pré-calcul (Recording Rules).

**Q : Que fait la requête `rate(container_cpu_usage_seconds_total{name!=""}[5m])` ?**
Elle calcule le **taux d'utilisation du CPU par seconde** pour chaque conteneur Docker (ignorant ceux sans nom, `name!=""`), lissé sur une fenêtre glissante des **5 dernières minutes**. C'est la requête standard pour obtenir le % de CPU utilisé.

## 8. Retour d'expérience
*Difficultés rencontrées :* L'orchestration du démarrage entre PostgreSQL et GLPI a nécessité l'implémentation d'un `healthcheck` strict dans Docker Compose, sinon GLPI tentait de se connecter à une base non prête. La gestion des timestamps Unix de GLPI (en INT) a également requis l'utilisation de `to_timestamp()` dans Grafana pour que les graphiques temporels (Time Series) fonctionnent correctement.

## ⚠️ Note d'Ingénierie sur l'Architecture (Important)
Lors du déploiement initial selon le cahier des charges, une incompatibilité critique a été identifiée : **GLPI ne supporte plus nativement PostgreSQL** (les pilotes PHP intégrés requièrent MySQL/MariaDB). Les tentatives de connexion avec l'image `postgres:15` entraînaient un crash (`MySQL server has gone away`). 

**Action corrective :** En tant que décision d'ingénierie, la base de données a été migrée vers `mariadb:10.11`. Les configurations de datasources Grafana et les requêtes SQL (Time series) ont été adaptées à la syntaxe MySQL (ex: utilisation de `DATE_FORMAT` et `DATE_SUB`). Cette modification garantit un environnement 100% stable et fonctionnel.
