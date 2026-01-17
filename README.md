# Hardening-ProxMox

Scripts **Shell** de durcissement pour systèmes **Debian**, conçus pour des hôtes **Proxmox VE** (standalone ou cluster).  
Les scripts appliquent un durcissement **SSH, système et réseau**, avec contrôle fin des accès administrateurs.

---

## Avertissement

Ces scripts modifient des composants critiques.

- Une mauvaise configuration peut **bloquer l’accès SSH**
- Exécution recommandée **en environnement de test**
- Accès console (IPMI / iLO / iDRAC / physique) requis
- **Snapshot / sauvegarde avant exécution**

---

## Vue d’ensemble

| Script | Rôle |
|------|-----|
| `create_user.sh` | Création d’un utilisateur SSH + compte admin lié |
| `hardening_ssh.sh` | Durcissement SSH strict (clé uniquement, restrictions) |
| `hard_sys.sh` | Durcissement système global (SSH, logs, Fail2ban, firewall Proxmox) |

---

## Détail des scripts

## `create_user.sh`

Script de **création d’identité SSH sécurisée** avec séparation des privilèges.

### Ce que fait le script

1. **Demande interactive**
   - Nom d’utilisateur standard (`user`)
   - Passphrase pour la clé privée SSH

2. **Gestion des groupes**
   - Crée le groupe `nossh` s’il n’existe pas
   - Ce groupe sera utilisé pour **interdire l’accès SSH**

3. **Utilisateur standard**
   - Crée l’utilisateur `user` (sans mot de passe)
   - Utilisateur destiné à la **connexion SSH**

4. **Compte administrateur lié**
   - Crée un compte `adm_user`
   - Ajoute ce compte :
     - au groupe `sudo`
     - au groupe `nossh` (→ **interdit de SSH**)
   - Usage : `sudo -u adm_user -i`

5. **Clé SSH**
   - Génère une clé RSA protégée par passphrase
   - Affiche :
     - l’empreinte
     - la clé publique
   - Ajoute la clé publique à `authorized_keys` de l’utilisateur standard

6. **Sécurisation SSH**
   - Ajoute automatiquement l’utilisateur standard à `AllowUsers`
   - Recharge le service SSH

### Résultat

- Accès SSH **uniquement** via l’utilisateur standard
- Administration **uniquement via sudo**
- Compte admin **impossible à utiliser en SSH**
- Séparation claire :
  - connexion
  - élévation
  - blocage

---

## `hardening_ssh.sh`

Script de **durcissement strict du service OpenSSH** basé sur des *drop-in files*.

### Actions réalisées

### 1. Préparation
- Installe `openssh-server` si absent
- Crée le groupe `nossh` (si absent)

---

### 2. Bannière légale
- Crée `/etc/issue.net` si absent
- Active la bannière SSH

---

### 3. Prévention du lockout
- Crée `/etc/ssh/sshd_config.d/10-allow-users.conf`
- Si aucun `AllowUsers` :
  - ajoute automatiquement l’utilisateur ayant lancé `sudo`

⚠️ **Effet important**  
Dès qu’un `AllowUsers` est présent, **tous les autres comptes sont refusés**.

---

### 4. Durcissement SSH (`60-hardening.conf`)

#### Authentification
- Clé publique **obligatoire**
- Désactivation :
  - mot de passe
  - keyboard-interactive
- Root SSH **interdit**
- PAM conservé (gestion de session)

#### Gestion des comptes
- `DenyGroups nossh`
- Tout compte membre de `nossh` est **bloqué en SSH**

#### Réduction de surface
- Désactivation :
  - X11 forwarding
  - TCP forwarding
  - Agent forwarding
  - tunnels

#### Sécurité connexion
- Limite tentatives (`MaxAuthTries`)
- Timeout court (`LoginGraceTime`)
- Déconnexion clients inactifs

#### Journalisation
- `LogLevel VERBOSE`

#### Cryptographie (optionnelle)
- KEX, ciphers, MACs modernes
- Compatible OpenSSH récent
- Désactivable pour clients anciens

---

### 5. Validation
- Test de configuration (`sshd -t`)
- Reload propre du service SSH

---

## `hard_sys.sh`

Script de **durcissement système orienté Proxmox**.

### Entrées requises
- Réseau d’administration autorisé (CIDR)
- Adresse email pour alertes SSH

---

### Actions réalisées

#### SSH
- Applique le durcissement SSH (mêmes principes que `hardening_ssh.sh`)
- Restreint l’accès SSH au réseau d’administration

#### Journalisation & alertes
- Script d’alerte email sur connexions SSH
- Intégration Rsyslog (compatible Fail2ban)

#### Fail2ban
- Installation et activation
- Protection du service SSH

#### Firewall Proxmox (PVE)
- Active le firewall PVE
- Politique par défaut : `DROP`
- Autorise uniquement :
  - SSH (22) depuis réseau admin
  - Interface Proxmox (8006) depuis réseau admin

---

### Résultat

- Accès réseau strictement contrôlé
- SSH surveillé et protégé
- Firewall Proxmox activé et cohérent
- Base sécurisée pour hôte Proxmox exposé

---

## Ordre d’exécution recommandé

1. `create_user.sh`
2. `hardening_ssh.sh`
3. `hard_sys.sh`

Toujours **tester l’accès SSH avant fermeture de session**.

---

## Philosophie du projet

- Séparation stricte des rôles
- Prévention du lockout
- Drop-in files (auditables, maintenables)
- Sécurité par défaut
- Compatible lab / prod / audit

---

## Statut

Projet en évolution.  
Scripts fournis **en l’état**.

Améliorations prévues :
- mode audit / dry-run
- rollback
- profils (standalone / cluster)
- mapping ANSSI / CIS

---

## Licence

Non définie.
