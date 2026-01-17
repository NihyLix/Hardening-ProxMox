# Hardening-ProxMox

Scripts **Shell** de durcissement pour systèmes **Debian**, utilisés notamment comme base pour **Proxmox VE**.  
Objectif : réduire la surface d’attaque et sécuriser l’accès système et SSH.

---

## Avertissement

Ces scripts modifient des paramètres critiques.

- Une mauvaise configuration peut **bloquer l’accès distant**
- Exécution recommandée **en VM / lab**
- Un accès console (IPMI / iLO / iDRAC / physique) doit être disponible
- **Snapshot / sauvegarde obligatoire** avant exécution

---

## Contenu du dépôt

| Script | Description |
|------|------------|
| `create_user.sh` | Création et préparation d’un utilisateur administrateur non-root |
| `hardening_ssh.sh` | Durcissement du service SSH via fichiers drop-in |
| `hard_sys.sh` | Durcissement global du système |

---

## Description des scripts

### `create_user.sh`

Prépare un **compte administrateur dédié**.

- Création d’un utilisateur non-root
- Attribution des droits d’administration
- Séparation des usages root / admin / applicatif

But : limiter l’usage direct du compte `root` et préparer un accès SSH sécurisé.

---

### `hardening_ssh.sh`

Durcissement avancé de **OpenSSH** via `/etc/ssh/sshd_config.d/`.

Fonctionnalités principales :
- Authentification **par clé publique uniquement**
- Désactivation :
  - mots de passe
  - authentification interactive
  - accès SSH root
- Gestion des accès :
  - `AllowUsers` pour éviter les lockouts
  - groupe `nossh` pour bloquer certains comptes
- Réduction de surface :
  - pas de forwarding, tunneling, X11
  - limitations de sessions et tentatives
- Journalisation renforcée (`LogLevel VERBOSE`)
- Bannière légale SSH
- Cryptographie moderne optionnelle
- Validation automatique de la configuration (`sshd -t`)

---

### `hard_sys.sh`

Durcissement général du système.

Fonctions typiques :
- Renforcement des paramètres OS
- Désactivation de services inutiles
- Réglages de sécurité globaux (permissions, réseau, comportements par défaut)

But : compléter le durcissement SSH par des mesures système.

---

## Bonnes pratiques

- Ordre recommandé :
  1. `create_user.sh`
  2. `hardening_ssh.sh`
  3. `hard_sys.sh`
- Vérifier l’accès SSH avant fermeture de session
- Tester chaque modification
- Documenter les changements (audit / rollback)

---

## Statut

Projet en évolution.  
Scripts fournis **en l’état**, sans garantie.

Améliorations possibles :
- mode audit / dry-run
- rollback
- profils (Debian, Proxmox, cluster)
- alignement formalisé (ANSSI / CIS)

---

