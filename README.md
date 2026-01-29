# 🖥️ Infrastructure VDI & Automatisation (SAE 5.01)

![Proxmox](https://img.shields.io/badge/Virtualization-Proxmox%20VE-orange) ![Python](https://img.shields.io/badge/Backend-Python%20Flask-blue) ![Guacamole](https://img.shields.io/badge/Remote-Apache%20Guacamole-green) ![pfSense](https://img.shields.io/badge/Security-pfSense-darkblue)

Ce projet vise à concevoir et déployer une **Infrastructure de Bureau Virtuel (VDI)** complète. L'objectif est de permettre aux étudiants et enseignants d'accéder à des environnements de Travaux Pratiques (Linux, Windows, Kali) à la demande, depuis n'importe quel navigateur web, sans installation de client lourd.

Le projet inclut un **portail d'automatisation** développé en Python/Flask pour l'orchestration des VMs.

## 📋 Sommaire
- [Architecture Globale](#-architecture-globale)
- [Stack Technique](#-stack-technique)
- [Fonctionnalités Clés](#-fonctionnalités-clés)
- [Installation et Configuration](#-installation-et-configuration)
- [Le Portail d'Automatisation](#-le-portail-dautomatisation)
- [Innovation : Workflow DNS](#-innovation--workflow-dns-instantané)
- [Auteurs](#-auteurs)

---

## 🏗 Architecture Globale

[cite_start]L'infrastructure repose sur un serveur physique hébergeant un hyperviseur et une segmentation réseau stricte pour garantir la sécurité.

* **Zone Publique (WAN)** : Connectée au réseau de l'IUT (172.31.xx.xx).
* **Zone Privée (LAN)** : Réseau interne (192.168.1.0/24) hébergeant les VMs et services critiques.
* **Passerelle** : Un routeur virtuel (pfSense) assure la liaison et le filtrage entre ces zones, rendant le LAN inaccessible directement depuis l'extérieur.

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

## 🚀 Fonctionnalités Clés

* **Accès "Zero Client"** : Tout se passe dans le navigateur web via HTML5.
* **Double Authentification Hybride** :
    * **LDAP (AD)** : Pour l'authentification des utilisateurs (étudiants/profs).
    * **MySQL** : Pour stocker la configuration technique des connexions.
* **Provisionnement Automatique** : Clonage de "Golden Images" (Windows/Linux) via l'API Proxmox.
* **Intégration Active Directory** : Les VMs rejoignent automatiquement le domaine au démarrage via un script `join-ad.sh` (Zero Touch).
* **Green IT** : Gestion dynamique des ressources pour éviter le gaspillage énergétique et le "VM Sprawl".

---

## ⚙ Installation et Configuration

### 1. Hyperviseur & Réseau (Proxmox + pfSense)
* **Proxmox** : Création d'un pont Linux (`vmbr0`) isolé pour le LAN interne, sans port physique lié.
* **pfSense** :
    * **Interface WAN** : Configuration DHCP (IP en 172.31.x.x).
    * **Interface LAN** : IP statique `192.168.1.1`.
    * **NAT Outbound** : Mode automatique pour permettre aux VMs de sortir sur Internet.
    * **Port Forwarding** : Redirection du port 8080 (WAN) vers l'IP interne de Guacamole.

### 2. Services d'Annuaire (Windows AD)
* **Domaine** : `dom-famchon.rt.lan`.
* **DNS** : Création d'une zone inversée `16.31.172.in-addr.arpa` pour la résolution IP → Nom.
* **Redirecteurs** : Ajout de l'IP pfSense (`192.168.1.1`) pour résoudre les noms internet.

### 3. Passerelle Apache Guacamole
* **Installation** : Compilation de `guacd` et déploiement du `.war` sur Tomcat 9.
* **Liaison AD** : Configuration du fichier `guacamole.properties` avec `ldap-user-base-dn: DC=dom-famchon,DC=rt,DC=lan`.

---

## 🐍 Le Portail d'Automatisation (Python/Flask)

[cite_start]L'application agit comme un chef d'orchestre entre l'utilisateur, l'API Proxmox et l'API Guacamole[cite: 1183].

### Structure
* `app.py` : Cœur de l'application (Logique métier, Routes).
* `config.py` : Contient les secrets (Tokens API, URLs).
* `templates/` : Interfaces HTML (Login, Dashboard).

### Contournement du Proxy (Challenge Technique)
Le script Python passait par le proxy de l'université pour joindre `localhost`, causant des erreurs. Nous avons forcé le bypass du proxy pour les requêtes locales.

```python
# app.py - Solution Bypass Proxy
NO_PROXY = {
    "http": None,
    "https": None,
}
# Utilisation dans les appels API
requests.post(url, data=data, proxies=NO_PROXY)
