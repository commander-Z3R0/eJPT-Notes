# Windows File System Vulnerabilities

Cette leçon couvre les **Alternate Data Streams (ADS)**, une fonctionnalité du système de fichiers NTFS que les attaquants exploitent pour cacher des charges malveillantes à l'intérieur de fichiers légitimes — contournant ainsi la détection antivirus de base.

> ⚠️ **Note** : Toutes les commandes de cette leçon sont exécutées sur le **système Windows cible** via une session shell ou invite de commandes. Des privilèges administrateur sont requis pour l'étape du lien symbolique.

---

## Alternate Data Streams (ADS)

### Qu'est-ce que les ADS ?

**Alternate Data Streams (ADS)** est un attribut du **système de fichiers NTFS** (utilisé par Windows). Il a été conçu à l'origine pour assurer la compatibilité avec le **Apple HFS (Hierarchical File System)** lors du transfert de fichiers entre macOS et Windows.

Chaque fichier créé sur un lecteur formaté NTFS contient **deux streams** :

| Stream | Utilité | Visible par l'utilisateur ? |
|--------|---------|----------------------------|
| **Data Stream** | Stream par défaut — contient le contenu réel du fichier | Oui |
| **Resource Stream** | Contient les métadonnées du fichier (auteur, horodatages, etc.) | Non |

Le **Resource Stream** est la clé — il peut stocker des données arbitraires, y compris du code exécutable, et reste **complètement caché** dans les listings de répertoire standard et dans l'Explorateur Windows.

### Pourquoi les Attaquants Utilisent les ADS

Les ADS constituent une technique d'évasion puissante car :

- Le fichier hôte semble **normal** — même nom, même taille visible.
- Le stream caché **n'apparaît pas** dans les listings `dir` ni dans l'Explorateur.
- Les **antivirus basiques à signatures** et les **outils de scan statique** n'inspectent pas les ADS.
- Les données cachées persistent tant que le fichier hôte existe.

---

## Les ADS en Pratique

### Étape 1 : Cacher des Données dans un Stream de Fichier

La commande suivante crée un stream caché appelé `secret.txt` à l'intérieur de `test.txt`. Même si `test.txt` n'a pas de contenu visible, le stream secret stocke des données indépendamment :

```cmd
notepad test.txt:secret.txt
```

Quand le Bloc-notes s'ouvre, tapez le contenu caché et sauvegardez. Résultat :
- `test.txt` affiche **0 octets** dans l'Explorateur et dans `dir` — semble vide.
- Le contenu est stocké invisiblement dans le stream `secret.txt` de `test.txt`.

### Étape 2 : Cacher une Charge Malveillante dans un Fichier Légitime

Copiez votre payload dans le resource stream d'un fichier journal à l'apparence innocente :

```cmd
type payload.exe > C:\Temp\windowslog.txt:payload.exe
```

Ce qui se passe :
- `windowslog.txt` apparaît comme un fichier journal normal et vide (0 octet).
- `payload.exe` est maintenant caché dans son resource stream.
- Vous pouvez **supprimer le `payload.exe` original** — la copie persiste dans `windowslog.txt`.

```cmd
del payload.exe
```

### Étape 3 : Créer un Lien Symbolique pour Exécuter la Charge Cachée

Pour exécuter le payload caché sans l'extraire, créez un **lien symbolique** qui mappe un nom de commande directement vers le stream caché. Cela doit être fait avec des **privilèges administrateur** :

```cmd
cd C:\Windows\System32
mklink winsvclog.exe C:\Temp\windowslog.txt:payload.exe
```

Ce que cela fait :
- Crée un lien symbolique nommé `winsvclog.exe` dans `System32`.
- Quand `winsvclog.exe` est tapé dans n'importe quelle invite de commandes, Windows exécute le `payload.exe` caché depuis `windowslog.txt`.
- Le payload s'exécute sans aucun fichier visible sur le disque — seul un fichier journal vide et un lien symbolique existent.

### Étape 4 : Exécuter la Charge Cachée

```cmd
winsvclog.exe
```

Le payload s'exécute silencieusement depuis le stream ADS.

---

## Vérifier les ADS (Détection)

La commande `dir` standard **ne révèle pas** les streams ADS. Pour lister tous les streams d'un fichier :

```cmd
dir /r C:\Temp\windowslog.txt
```

Le flag `/r` révèle les resource streams. Exemple de sortie :

```
windowslog.txt          0 octets
windowslog.txt:payload.exe:$DATA
```

Vous pouvez également utiliser l'outil **Sysinternals Streams** :

```cmd
streams.exe C:\Temp\windowslog.txt
```

---

## Résumé de l'Attaque ADS

| Étape | Commande | Objectif |
|-------|---------|---------|
| **1. Cacher des données** | `notepad file.txt:hidden.txt` | Stocker des données invisiblement dans un stream |
| **2. Cacher le payload** | `type payload.exe > log.txt:payload.exe` | Copier l'exécutable dans le resource stream du fichier |
| **3. Supprimer l'original** | `del payload.exe` | Effacer toute preuve visible |
| **4. Créer un lien symbolique** | `mklink winsvclog.exe C:\Temp\log.txt:payload.exe` | Mapper une commande vers le stream caché |
| **5. Exécuter** | `winsvclog.exe` | Lancer le payload caché sans fichier sur le disque |
| **6. Détecter** | `dir /r file.txt` | Révéler les streams cachés |

---

## Points Clés

- **ADS (Alternate Data Streams)** est une fonctionnalité NTFS permettant aux fichiers de contenir plusieurs streams de données — seul le stream par défaut est visible.
- Conçu à l'origine pour la **compatibilité macOS HFS**, les ADS sont détournés par les attaquants pour cacher des charges malveillantes dans des fichiers légitimes.
- Chaque fichier NTFS possède un **Data Stream** (contenu visible) et un **Resource Stream** (métadonnées cachées — là où les attaquants cachent les payloads).
- Un fichier avec un stream ADS caché affiche **0 octet** dans l'Explorateur et dans `dir` standard — il semble complètement vide.
- Utilisez `type payload.exe > file.txt:payload.exe` pour copier un exécutable dans le stream caché d'un fichier.
- Utilisez `mklink` dans `System32` pour créer un lien symbolique exécutant le payload caché par son nom — nécessite des **privilèges administrateur**.
- Utilisez `dir /r` ou **Sysinternals Streams** pour détecter et révéler les streams ADS cachés.
- Les ADS contournent l'**AV basique à signatures** et les **scanners statiques** — mais les solutions EDR modernes et les AV avancés peuvent les détecter.
