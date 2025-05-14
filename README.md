# ☁️ Déploiement et gestion d'application cloud avec Kubernetes (K3S)

Ce projet vise à maîtriser le déploiement, la sécurisation, et l’observabilité d’une application cloud native via un cluster Kubernetes, en utilisant la distribution légère **K3S**.

## 🎯 Objectifs pédagogiques

| Compétence | Description |
|------------|-------------|
| **Kubernetes** | Comprendre le fonctionnement d’un cluster Kubernetes via K3S |
| **Déploiement cloud** | Déployer une application complexe (WordPress + base de données) dans un namespace isolé |
| **Sécurité** | Sécuriser l’accès web avec HTTPS, gestion des secrets avec HashiCorp Vault |
| **Observabilité** | Mettre en place Prometheus, Grafana, Loki et Promtail pour suivre l’état du cluster et des services |

---

## 🏢 Cadre du projet

Votre entreprise a choisi K3S (Rancher) pour héberger ses applications. Vous devez configurer :

- Un site WordPress (avec MariaDB/MySQL)
- Un accès sécurisé HTTPS (via Traefik + Let's Encrypt)
- Une gestion avancée des secrets avec **HashiCorp Vault**
- Une plateforme d’observabilité complète (Prometheus, Grafana, Loki, Promtail)

---

## 🧱 Architecture du projet

### 🔹 Déploiement WordPress

- Déploiement dans un **Namespace dédié**
- Base de données persistante via **Persistent Volume Claims** (PVC)
- Utilisation de **Secrets Kubernetes** pour les identifiants
- Accessibilité en HTTP puis HTTPS via IngressRoute Traefik

### 🔹 Sécurisation HTTPS

- Utilisation de **Traefik 2.1** (Ingress Controller)
- Activation de **Let's Encrypt** via ConfigMap
- Déclaration de l’IngressRoute HTTPS avec certificat valide

### 🔹 Gestion avancée des secrets

- Déploiement de **HashiCorp Vault** via Helm
- Stockage dynamique des secrets, avec :
- **Rotation automatique** hebdomadaire
- **Génération aléatoire**
- Modification des objets `Deployment` pour consommer les secrets Vault

### 🔹 Observabilité

- Déploiement de :
- **Prometheus** pour les métriques
- **Grafana** pour les dashboards
- **Loki + Promtail** pour les logs
- Dashboards Grafana :
- **Cluster K3S** : CPU, RAM, connexions TCP, objets K8S, logs
- **Services** : Requêtes HTTP, codes réponse, logs WordPress
- Accès via l’onglet `Explore` de Grafana pour la recherche avancée de logs



