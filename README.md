# 🏢 Mini-Projet Kubernetes : Déploiement Sécurisé d'Odoo

## 🌐 Objectif

Déployer l'application **Odoo** dans le namespace `odoo` de manière **sécurisée** avec une gestion fine des droits d'accès. Ce mini-projet met en pratique les compétences essentielles en **Kubernetes**, **RBAC**, **certificats client**, **stockage persistant**, **ingress**, et **observabilité**.

---

## ⚖️ Prérequis

* Un cluster Kubernetes fonctionnel avec un utilisateur root/admin.
* Accès à la ligne de commande sur une VM ou machine Linux.

---

## ✅ Phases du Projet

### Phase A - Mise en place du réseau et du stockage (root)

1. **Installer MetalLB** avec un `IPAddressPool`
2. **Installer NGINX Ingress Controller** exposé via MetalLB
3. **Installer le NFS Provisioner et la StorageClass** pour le provisionning dynamique des volumes persistants

---

### Phase B - Gestion des utilisateurs

#### B1. Création de l'utilisateur Linux `aisfr`

#### B2. Création de l'utilisateur Kubernetes `<votre_prenom>` (connecté en tant que `aisfr`)

* Générer une clé privée
* Générer un CSR `/CN=<votre_prenom>/O=developer`
* Envoyer la requête au root
* **(avec le user root)** Signer le certificat avec la CA Kubernetes
* Créer un fichier `~/.kube/config` pour <votre_prenom> avec son certificat

#### B3. Attribution des droits sur le namespace `odoo` (root)

* Créer un `Role` avec tous les droits sur les ressources dans le namespace `odoo`
* Créer un `RoleBinding` pour le groupe `developer`

---

### Phase C - Déploiement d'Odoo (connecté en tant que `aisfr`)

Dans le namespace `odoo` :

* Namespace `odoo`
* Secret pour les identifiants PostgreSQL
* ConfigMap (si besoin pour la conf de Odoo)
* PVC pour Odoo (qui utilisera la storageClass vers le nfs)
* Deployment pour Odoo
* Service ClusterIP pour Odoo
* StatefulSet pour PostgreSQL (avec `volumeClaimTemplate` qui utilisera la storageClass vers le nfs)
* Service ClusterIP pour PostgreSQL
* Ingress Rule pour redigirer `odoo.m2iformation.fr` vers le service Odoo
* et tout autre fichier que vous jugez nécessaire...

---

### Phase D - Déploiement du Kubernetes Dashboard (root)

1. Installer le Dashboard dans le namespace `kubernetes-dashboard`
2. Créer une ingress rule : `dashboard.m2iformation.fr`
3. Créer deux ServiceAccounts :

   * `dashboard-admin` avec `cluster-admin`
   * `dashboard-read-only` avec `view`
4. Générer et tester les tokens de connexion - Utiliser les deux tokens pour accéder au dashboard:
   * `dashboard-admin` avec `kubectl proxy` pour l'administration des ressources
   * `dashboard-read-only` avec `dashboard.m2iformation.fr` pour la consultation des ressources de l'application odoo par un utilisateur tier

---

## 📊 Compétences évaluées

| Domaine          | Compétence                             |
| ---------------- | -------------------------------------- |
| ⚖️ Sécurité      | RBAC, certificats TLS, user management |
| ⚙️ Déploiement   | Deployments, StatefulSets, PVC         |
| 🔎 Networking    | MetalLB, Ingress Controller            |
| 📊 Monitoring    | Kubernetes Dashboard                   |
| 📄 Observabilité | Vue sur le cluster via UI              |
| 🚛 GitOps        | Organisation de manifestes             |
| 📊 **Rapport Visuel**| **Captures d'écran des Résultats**         |

---

## 📁 Structure attendue des fichiers (côté <votre_prenom>)

```
odoo-k8s/
├── captures-des-resultats/
├── manifests/
│   ├── odoo/
│   │   ├── 00-namespace.yaml
│   │   ├── 01-secret-postgres.yaml
│   │   ├── 02-configmap-odoo.yaml (optionnel)
│   │   ├── 03-pvc-odoo.yaml
│   │   ├── 04-deployment-odoo.yaml
│   │   ├── 05-service-odoo.yaml
│   │   ├── 06-statefulset-postgres.yaml
│   │   ├── 07-service-postgres.yaml
│   │   └── 08-ingress-odoo.yaml
│   └── dashboard/
│       ├── 01-dashboard-ingress.yaml
│       ├── 02-dashboard-admin-serviceaccount.yaml
│       └── 03-dashboard-readonly-serviceaccount.yaml
├── rbac/
│   ├── role-devops.yaml
│   └── rolebinding-<votre_prenom>.yaml

```

---

## 📊 Validation

* `kubectl get all -n odoo`
* Accès via navigateur : `http://odoo.m2iformation.fr` et faire la première configuration de Odoo
* Accès Dashboard : `kubectl proxy` - Lecture/Ecriture vérifiée avec le token admin
* Accès Dashboard : `http://dashboard.m2iformation.fr` - Lecture seule vérifiée avec le token read-only

---

🚀 Beaucoup de Courage dans votre montée en compétences dans le monde des Infrastructures Sécurisées !
