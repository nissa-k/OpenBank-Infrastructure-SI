# OpenBank-Infrastructure-SI

> Compte rendu technique — Raccordement SI & Sécurité bout en bout  
> **Auteure :** Karadag Nissa | **Formation :** Bachelor Réseau & Sécurité

---

## Description

Mise en œuvre d'une infrastructure réseau multi-sites pour la société fictive **OpenBank**, simulant l'interconnexion sécurisée de deux sites distants (Paris et Nantes). Le projet couvre le déploiement d'un Active Directory centralisé, d'un tunnel VPN IPsec inter-sites et de firewalls Stormshield SNS.

---

## Architecture

```
Site Paris                          Site Nantes
─────────────────────               ─────────────────────
SNS-PARIS (Stormshield EVA1)        SNS-NANTES (Stormshield EVA1)
  WAN : 192.36.253.10/24    ◄──────►  WAN : 192.36.253.20/24
  LAN : 10.0.1.1/24                    LAN : 10.0.2.1/24

SRV-PARIS-DC                        SRV-NANTES-RODC
  IP : 10.0.1.10/24                    IP : 10.0.2.10/24
  Rôle : DC Principal (GUI)            Rôle : RODC (Core)

Domaine AD : openbank.loc
```

---

## Technologies utilisées

| Technologie | Usage |
|---|---|
| VirtualBox | Virtualisation des machines |
| Windows Server 2022 | Contrôleurs de domaine (GUI + Core) |
| Active Directory DS | Annuaire centralisé, GPO, OU |
| Stormshield SNS 4.8.6 | Firewall, VPN IPsec, filtrage |
| PKI / Certificats RSA | Authentification mutuelle VPN |
| LDAP (port 389) | Intégration AD ↔ Stormshield |

---

## Contenu du projet

### Étape 1 — Environnement VirtualBox
- Création du réseau NAT `NAT_TELETRAVAIL` (192.36.253.0/24)
- Import et démarrage des firewalls SNS-PARIS et SNS-NANTES

### Étape 2 — Windows Server 2022
- VM SRV-PARIS-DC : 4 Go RAM, 3 vCPU, 50 Go, interface bureau
- Configuration IP statique (10.0.1.10/24) et désactivation IPv6

### Étape 3 — Active Directory Paris (DC Principal)
- Nouvelle forêt : `openbank.loc` (niveau fonctionnel WS 2016)
- Structure OU : `OU_Paris` et `OU_Nantes` (Utilisateurs / Ordinateurs / Groupes)
- 4 groupes de sécurité : `GRP_IT_Admins`, `GRP_Paris_Users`, `GRP_Nantes_Users`, `GRP_VPN_SS`
- 4 comptes utilisateurs créés et placés dans les OU correspondantes

### Étape 4 — RODC Nantes (Contrôleur en lecture seule)
- Installation Windows Server 2022 Core, gestion via SConfig
- Pré-création du compte RODC depuis Paris, promotion via PowerShell (`-UseExistingAccount`)
- Réplication AD confirmée

### Étape 5 — Firewalls Stormshield
- PKI inter-sites : CA-Paris et CA-Nantes (RSA 4096 bits), échange croisé des certificats
- Certificats serveurs : `paris.openbank.loc` et `nantes.openbank.loc` (RSA 2048 bits)
- VPN IPsec IKEv2 avec profil `StrongEncryption` et authentification par certificat
- Politiques de filtrage inter-sites configurées sur les deux firewalls
- Intégration LDAP/Active Directory sur SNS-PARIS

---

## Bilan des objectifs

| Objectif | Statut |
|---|---|
| VirtualBox + import firewalls |  OK |
| Windows Server 2022 Paris (GUI) |  OK |
| Active Directory + domaine openbank.loc |  OK |
| Structure OU + Groupes + Utilisateurs |  OK |
| Windows Server 2022 Nantes (Core) |  OK |
| RODC Nantes joint au domaine |  OK |
| PKI — CA Paris et CA Nantes |  OK |
| Certificats serveurs |  OK |
| Règles de filtrage inter-sites |  OK |
| Intégration LDAP / AD |  OK |
| VPN IPsec site-à-site |  Partiel — bug interne EVA |

---

## Problèmes rencontrés

- **Compatibilité navigateur** : Stormshield EVA 4.8.6 incompatible avec Chrome/Edge → résolu avec Firefox
- **RAM insuffisante** : augmentation à 4 Go RAM / 4 vCPU pour les VMs firewall
- **Promotion RODC** : nécessite l'installation préalable du rôle AD DS + paramètre `-UseExistingAccount`
- **Tunnel VPN non établi** : incohérence interne EVA dans la gestion des certificats RSA (hors erreur de configuration)

---

## Fichiers

| Fichier | Description |
|---|---|
| `compte_rendu_openbank.pdf` | Compte rendu technique complet avec captures d'écran |

---

*Formation Bachelor Réseau & Sécurité — Projet réalisé sous VirtualBox en environnement virtualisé.*

## Auteur
Karadag Nissa
