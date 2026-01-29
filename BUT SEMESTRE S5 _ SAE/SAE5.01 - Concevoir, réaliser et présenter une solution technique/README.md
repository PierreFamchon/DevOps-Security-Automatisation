<div align="center">

  <img src="https://cdn-icons-png.flaticon.com/512/8943/8943615.png" alt="Logo Remote Access" width="120" height="120">
  
  # 🖥️ Infrastructure VDI & Automatisation (SAE 5.01)

  **Conception, Déploiement et Automatisation d'une Infrastructure de Bureau Virtuel**

  ![Proxmox](https://img.shields.io/badge/Virtualization-Proxmox%20VE-orange?style=for-the-badge&logo=proxmox&logoColor=white)
  ![Python](https://img.shields.io/badge/Backend-Python%20Flask-blue?style=for-the-badge&logo=python&logoColor=white)
  ![Guacamole](https://img.shields.io/badge/Remote-Apache%20Guacamole-green?style=for-the-badge&logo=apache&logoColor=white)
  ![pfSense](https://img.shields.io/badge/Security-pfSense-darkblue?style=for-the-badge&logo=pfsense&logoColor=white)

  <br>

  [Description](#-description) •
  [Fonctionnalités](#-fonctionnalités-clés) •
  [Stack Technique](#-stack-technique) •
  [Structure](#-structure-du-projet) •
  [Installation](#-installation-et-configuration) •
  [Phases](#-phases-du-projet) •
  [Bilan](#-bilan) •
  [Auteurs](#-auteurs)

</div>

---

## 📝 Description

Ce projet vise à concevoir et déployer une **Infrastructure de Bureau Virtuel (VDI)** complète. L'objectif est de permettre aux étudiants et enseignants d'accéder à des environnements de Travaux Pratiques (Linux, Windows, Kali) à la demande, depuis n'importe quel navigateur web, sans installation de client lourd.

Le cœur du système repose sur un **portail d'automatisation** développé en Python/Flask qui orchestre l'hyperviseur et la passerelle d'accès.

---

## 🚀 Fonctionnalités Clés

* **Accès "Zero Client"** : Tout se passe dans le navigateur web via HTML5 (RDP/SSH via Guacamole).
* **Double Authentification Hybride** :
    * **LDAP (Active Directory)** : Authentification unique pour les étudiants et enseignants.
    * **MySQL (MariaDB)** : Gestion technique des connexions VDI.
* **Provisionnement Automatique** : Clonage instantané de "Golden Images" via l'API Proxmox.
* **Zero Touch Provisioning** : Les VMs rejoignent automatiquement le domaine AD au démarrage via un script embarqué (`join-ad.sh`).
* **Green IT** : Gestion dynamique des ressources pour éviter le "VM Sprawl" (machines zombies) et réduire l'empreinte énergétique.

---

## 🛠 Stack Technique

### Infrastructure & Virtualisation

![Proxmox](https://img.shields.io/badge/HYPERVISOR-PROXMOX%20VE%208-E57000?style=for-the-badge&logo=proxmox&logoColor=white)
![LXC](https://img.shields.io/badge/CONTAINER-LXC-E57000?style=for-the-badge&labelColor=404040)
![KVM](https://img.shields.io/badge/VIRTUALIZATION-KVM-404040?style=for-the-badge&labelColor=E57000)

### Sécurité & Réseau

![pfSense](https://img.shields.io/badge/FIREWALL-PFSENSE-2C3E50?style=for-the-badge&logo=pfsense&logoColor=white)
![FreeBSD](https://img.shields.io/badge/OS-FREEBSD-AB2B28?style=for-the-badge&labelColor=404040&logo=freebsd&logoColor=white)
![Guacamole](https://img.shields.io/badge/GATEWAY-APACHE%20GUACAMOLE-256627?style=for-the-badge&logo=apache&logoColor=white)

### Annuaire & Identité

![Windows Server](https://img.shields.io/badge/OS-WINDOWS%20SERVER%202016-0078D6?style=for-the-badge&logo=windows&logoColor=white)
![Active Directory](https://img.shields.io/badge/IAM-ACTIVE%20DIRECTORY-0078D6?style=for-the-badge&labelColor=404040&logo=microsoft&logoColor=white)
![DNS](https://img.shields.io/badge/SERVICE-DNS-404040?style=for-the-badge&labelColor=0078D6)

### Automatisation & Données

![Python](https://img.shields.io/badge/LANGUAGE-PYTHON%203-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Flask](https://img.shields.io/badge/FRAMEWORK-FLASK-000000?style=for-the-badge&logo=flask&logoColor=white)
![MariaDB](https://img.shields.io/badge/DATABASE-MARIADB-003545?style=for-the-badge&logo=mariadb&logoColor=white)

---

## 📂 Structure du Projet

L'application d'automatisation (Portail Web) est structurée comme suit :

```text
📂 SAE5.01 - Concevoir, réaliser et présenter une solution technique
├── 📄 README.md                # README du projet
├── 📄 Rapport SAE 5.01.pdf     # Rapport complet (PDF)
└── 📂 mon_portail_vm/
    │
    ├── 🐍 app.py               # Cœur de l'application (Logique métier, Routes Flask)
    ├── ⚙️ config.py            # Secrets (Tokens API Proxmox/Guac, URLs)
    │
    └── 📂 templates/           # Interface Utilisateur (Frontend HTML)
        ├── 📄 login.html       # Page d'authentification
        └── 📄 dashboard.html   # Tableau de bord de gestion des VMs
```
---

## ⚙ Installation et Configuration

### 1. Architecture Réseau

L'infrastructure repose sur une segmentation stricte via pfSense:

* Zone Publique (WAN) : 172.31.xx.xx (Connecté au réseau IUT).
* Zone Privée (LAN) : 192.168.1.0/24 (Héberge les VMs et l'AD, inaccessible de l'extérieur).
* Isolation : Utilisation d'un pont Linux (vmbr0) sans port physique pour isoler le LAN.

### 2. Automatisation (Le Défi du Proxy)

Un défi majeur a été le blocage des appels API locaux par le proxy de l'université. Nous avons implémenté un Bypass Proxy dans le script Python.

```python
# app.py - Solution Bypass Proxy
NO_PROXY = {
    "http": None,
    "https": None,
}
# Utilisation dans les appels API pour forcer le trafic local
requests.post(url, data=data, proxies=NO_PROXY)
```

### 3. Innovation : Workflow DNS Instantané

Au lieu d'attendre la remontée d'IP par l'agent QEMU (lent), nous utilisons une prédiction DNS.

* Le script génère le nom de la VM (ex: user-tp1).
* Il construit le FQDN (user-tp1.dom-famchon.rt.lan).
* Il configure immédiatement Guacamole avec ce nom de domaine.
* Résultat : L'accès est disponible quasi-instantanément.

---

## 📊 Bilan

Ce projet a permis de livrer une plateforme "Clef en main" répondant aux contraintes de sécurité et de performance.

* Interopérabilité : Réussite du dialogue entre des briques hétérogènes (Proxmox REST, Guacamole MySQL, AD LDAP).
* Résilience : L'infrastructure est documentée et prête pour la production.
* Compétences : Montée en compétence forte sur le routage complexe, le débogage API (Proxy) et l'administration système.

--- 

## 👤 Auteurs

Étudiants R&T 3ème Année (2025-2026)

| Nom | Rôle |
| :--- | :--- |
| **Pierre Famchon** | Lead Network / Automatisation / AD / Guacamole |
| **Nicolas Edouard** | Virtualisation / Réseau / Templates |
| **Yohann Piek** | Docummentation / Tests / Support |
