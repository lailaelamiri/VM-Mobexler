# LAB — Analyse Statique Android — MobSF

> **Cours Sécurité Mobile** · Analyse Statique · MobSF · OWASP MASVS

---

## Objectif

Analyser statiquement une application Android vulnérable (**Allsafe**) à l'aide de **MobSF** dans un environnement contrôlé (VM Mobexler) afin d'identifier les vulnérabilités potentielles sans exécuter l'application, puis produire un rapport d'audit professionnel.

---

## Outils utilisés

| Outil | Rôle |
|---|---|
| VM Mobexler | Environnement d'analyse isolé |
| MobSF v4.0.6 | Framework d'analyse statique automatisée |
| Firefox | Interface web MobSF |
| Terminal (zsh) | Commandes et documentation |
| allsafe.apk (10.39 MB) | Application Android intentionnellement vulnérable |

---

## Partie 1 — Préparation de l'environnement

### Étape 1 — Création du dossier de travail

```bash
mkdir -p ~/lab && cd ~/lab
```

### Étape 2 — Téléchargement de l'APK

L'APK pédagogique utilisé est **Allsafe** — une application intentionnellement vulnérable conçue pour la formation en sécurité mobile.

```bash
wget https://github.com/t0thkr1s/allsafe/releases/latest/download/allsafe.apk
```

> **Capture 1 — Téléchargement de l'APK**

<img width="952" height="712" alt="Screenshot 2026-04-24 085844" src="https://github.com/user-attachments/assets/0a0a9f36-8810-46bb-ae01-9cb895db065e" />


---

### Étape 3 — Vérification de l'intégrité

```bash
sha256sum allsafe.apk > apk_hash.txt
ls -lh allsafe.apk
cat apk_hash.txt
```

```
d6792d6634a033f048f935f1269179d3c27b859c4c34b1e9e5b008a88375efd9  allsafe.apk
-rw-r--r-- 1 mobexler mobexler 10.39M Apr 24 2026 allsafe.apk
```

### Étape 4 — Fichier de traçabilité

```bash
echo "Date : $(date)" > analyse_info.txt
echo "Analyste : [Votre nom]" >> analyse_info.txt
echo "APK : allsafe.apk" >> analyse_info.txt
cat apk_hash.txt >> analyse_info.txt
```

---

## Partie 2 — Lancement de MobSF

### Étape 5 — Démarrage de MobSF via Docker

MobSF est préinstallé dans Mobexler sous forme de script Docker.

```bash
/home/mobexler/AndroidZone/MobSFdocker.sh
```

MobSF démarre et affiche :

```
[INFO] Mobile Security Framework v4.0.6
REST API Key: ac90ca6940ce1ec55bcbdfebc2ef18e691710f2017f4a46fe1bded2e28f7e15d
Default Credentials: mobsf/mobsf
```

> 📸 **Capture 2 — MobSF démarré dans le terminal**
<img width="794" height="526" alt="Screenshot 2026-04-24 090356" src="https://github.com/user-attachments/assets/f0cd6761-dc7f-4ea8-b531-5ffc0b8680a2" />


---

### Étape 6 — Accès à l'interface web

Ouvrir Firefox et naviguer vers :

```
http://127.0.0.1:8000
```

Identifiants : `mobsf` / `mobsf`


<img width="963" height="770" alt="Screenshot 2026-04-24 090518" src="https://github.com/user-attachments/assets/3674e720-83c1-4536-a4ff-06cdf7e8d58d" />

---

## Partie 3 — Import et analyse de l'APK

### Étape 7 — Upload de l'APK

Dans l'interface MobSF :
1. Cliquer sur **"Upload & Analyze"**
2. Sélectionner `~/lab/allsafe.apk`
3. Cliquer sur **"Analyze"**
4. Attendre la fin de l'analyse (2-5 minutes)

> 📸 **Capture 3 — Résultats MobSF : Score de sécurité 46/100**
<img width="976" height="829" alt="Screenshot 2026-04-24 092821" src="https://github.com/user-attachments/assets/206256c0-59e4-408a-8dcb-cd852a023550" />


### Résultats généraux

| Champ | Valeur |
|---|---|
| **App Name** | Allsafe |
| **Package** | infosecadventures.allsafe |
| **Version** | 1.5 (code 5) |
| **Target SDK** | 35 |
| **Min SDK** | 23 (Android 6.0) |
| **Taille** | 10.39 MB |
| **Score Sécurité** | **46/100** |
| **MD5** | 52a7bf23df56e39a26034304e41108f2 |
| **SHA256** | d6792d6634a033f048f935f1269179d3c27b859c4c34b1e9e5b008a88375efd9 |

---

## Partie 4 — Analyse du Manifeste et des Permissions

### Étape 8 — Manifest Analysis

Navigation : **Static Analyzer → Manifest Analysis**

> 📸 **Capture 4 — Manifest Analysis : 2 HIGH, 7 WARNING**
<img width="906" height="660" alt="Screenshot 2026-04-24 093209" src="https://github.com/user-attachments/assets/155b59ac-2b31-47c7-a64d-1cf8d322997e" />


| N° | Issue | Sévérité |
|---|---|---|
| 1 | App installable sur Android 6.0 vulnérable (minSdk=23) | 🔴 HIGH |
| 2 | Network Security Config présent | ℹ️ INFO |
| 3 | `android:debuggable=true` | 🔴 HIGH |
| 4 | `android:allowBackup=true` | 🟡 WARNING |
| 5 | Activity `ProxyActivity` exportée | 🟡 WARNING |
| 6 | Activity `DeepLinkTask` exportée | 🟡 WARNING |
| 7 | BroadcastReceiver `NoteReceiver` exporté | 🟡 WARNING |

---

### Étape 9 — Permissions

Navigation : **Static Analyzer → Permissions**

> 📸 **Capture 5 — Permissions dangereuses identifiées**
<img width="890" height="594" alt="Screenshot 2026-04-24 093507" src="https://github.com/user-attachments/assets/b8691c30-c8e1-42f1-ae6c-6bbb657e7fdf" />


| Permission | Statut | Usage déclaré |
|---|---|---|
| READ_EXTERNAL_STORAGE | 🔴 Dangereuse | Lecture stockage externe |
| WRITE_EXTERNAL_STORAGE | 🔴 Dangereuse | Écriture stockage externe |
| RECORD_AUDIO | 🔴 Dangereuse | Accès microphone |
| ACCESS_NETWORK_STATE | ✅ Normale | Statut réseau |
| INTERNET | ✅ Normale | Accès Internet |
| QUERY_ALL_PACKAGES | ✅ Normale | Lister les apps |

---

### Étape 10 — Composants exportés

> 📸 **Capture 6 — Composants : Services, Receivers, Providers**
<img width="921" height="641" alt="Screenshot 2026-04-24 093453" src="https://github.com/user-attachments/assets/e4a15942-ae48-4abe-a7a5-3b97c114398b" />

| Type | Composant | Exporté |
|---|---|---|
| Service | RecorderService | ⚠️ Oui |
| Service | ComponentDiscoveryService (Firebase) | — |
| Receiver | NoteReceiver | ⚠️ Oui |
| Receiver | ProfileInstallReceiver | — |
| Provider | DataProvider | ⚠️ Oui |
| Provider | FileProvider | — |

---

## Partie 5 — Analyse de la Configuration Réseau

### Étape 11 — Network Security

Navigation : **Static Analyzer → Network Security**

> 📸 **Capture 7 — Network Security : 1 HIGH**
<img width="865" height="260" alt="Screenshot 2026-04-24 094202" src="https://github.com/user-attachments/assets/389cd471-0d58-4a83-a4b5-fca3b29bc466" />

| Scope | Sévérité | Description |
|---|---|---|
| `infosecadventures.io` | 🔴 HIGH | Trafic en clair (HTTP) autorisé |

**Fichier `res/xml/network_security_config.xml`** — cleartext traffic explicitement autorisé pour `infosecadventures.io`, exposant les données transmises à des interceptions de type MITM.

---

## Partie 6 — Analyse du Code

### Étape 12 — Code Analysis

Navigation : **Static Analyzer → Code Analysis**

> 📸 **Capture 8 — Code Analysis : 1 HIGH, 6 WARNING, 2 INFO, 2 SECURE**
<img width="875" height="585" alt="Screenshot 2026-04-24 094250" src="https://github.com/user-attachments/assets/b7916ae4-c3c1-49f1-b5e7-8e191a8f910a" />


| N° | Issue | Sévérité | MASVS | Fichier |
|---|---|---|---|---|
| 11 | ECB mode — chiffrement faible | 🔴 HIGH | MSTG-CRYPTO-2 | WeakCryptography.java |
| 1 | Informations sensibles dans les logs | ℹ️ INFO | MSTG-STORAGE-3 | — |
| 6 | Données copiées dans le clipboard | ℹ️ INFO | MSTG-STORAGE-10 | ClipUtil.java |
| 5 | SSL Certificate Pinning | ✅ SECURE | MSTG-NETWORK-4 | CertificatePinning.java |

---

### Étape 13 — Hardcoded Secrets

Navigation : **Static Analyzer → Hardcoded Secrets**

> 📸 **Capture 9 — 29 secrets hardcodés détectés**
<img width="921" height="561" alt="Screenshot 2026-04-24 093430" src="https://github.com/user-attachments/assets/bcfa22b7-eeab-414a-bf0d-962e71d46e88" />


**29 secrets détectés**, dont :

```
firebase_database_url         : https://allsafe-8cef0.firebaseio.com
google_crash_reporting_api_key: AlzaSyDjteCQ0-ElkfBxVZlZmBfCSPNEYUYcK1g
google_api_key                : AlzaSyDjteCQ0-ElkfBxVZlZmBfCSPNEYUYcK1g
```

---

## Partie 7 — Corrélation OWASP MASVS

### Étape 14 — Mapping des vulnérabilités

| Vulnérabilité | Référence MASVS | Statut |
|---|---|---|
| Secrets hardcodés | MSTG-STORAGE-14 | ❌ Non conforme |
| Debug activé | MSTG-CODE-2 | ❌ Non conforme |
| Chiffrement ECB | MSTG-CRYPTO-2 | ❌ Non conforme |
| Trafic HTTP clair | MSTG-NETWORK-2 | ❌ Non conforme |
| Composants exportés | MSTG-PLATFORM-1 | ❌ Non conforme |
| Certificate Pinning | MSTG-NETWORK-4 | ✅ Conforme |

**Tests MASTG complémentaires identifiés :**
- `MASTG-TEST-0001` — Testing Local Storage for Sensitive Data
- `MASTG-TEST-0011` — Testing for Sensitive Data in Network Traffic

---

## Partie 8 — Export du Rapport MobSF

### Étape 15 — Génération du PDF

Dans l'interface MobSF, cliquer sur **"Generate PDF Report"** en haut à droite.


```bash
mv ~/Downloads/MobSF_Report*.pdf ~/lab/
echo "Rapport exporté : MobSF_Report_$(date +%Y%m%d).pdf" >> analyse_info.txt
```

> 📎 Le rapport complet généré par MobSF est disponible en annexe : **`MobSF_Report_20260424.pdf`**

---

## Rapport d'Audit Final

### Informations générales

| Champ | Valeur |
|---|---|
| **Application** | Allsafe |
| **Package** | infosecadventures.allsafe |
| **Version** | 1.5 (code 5) |
| **Fichier** | allsafe.apk (10.39 MB) |
| **SHA-256** | d6792d6634a033f048f935f1269179d3c27b859c4c34b1e9e5b008a88375efd9 |
| **Date d'analyse** | 2026-04-24 |
| **Analyste** | [Votre nom] |
| **Outil** | MobSF v4.0.6 dans VM Mobexler |
| **Score de sécurité** | **46/100** |

---

### Résumé exécutif

L'analyse statique de l'application **Allsafe** révèle un niveau de risque **ÉLEVÉ** (score MobSF : 46/100). Les principales vulnérabilités concernent la présence de **29 secrets hardcodés** dans le code source (clés Firebase, API Google), un **chiffrement cryptographique faible** (mode ECB), des **composants Android exportés sans protection**, et des **communications réseau non chiffrées** pour le domaine `infosecadventures.io`. L'application demande 3 permissions dangereuses dont certaines paraissent excessives au regard de sa fonction principale.

---

### Top 5 Vulnérabilités

#### 🔴 V1 — Secrets Hardcodés dans le Code Source
- **Sévérité :** Élevée
- **MASVS :** MSTG-STORAGE-14
- **Description :** 29 secrets détectés en clair dans le code, incluant clés Firebase et API Google.
- **Preuve :** `firebase_database_url`, `google_api_key` visibles dans les ressources de l'APK
- **Impact :** Vol de clés API, accès non autorisé aux services Firebase
- **Remédiation :** Utiliser **Android Keystore** ou variables d'environnement CI/CD. Révoquer immédiatement les clés exposées.

#### 🔴 V2 — Mode Debug Activé en Production
- **Sévérité :** Élevée
- **MASVS :** MSTG-CODE-2
- **Description :** `android:debuggable=true` présent dans le manifeste Android.
- **Preuve :** `AndroidManifest.xml` — `android:debuggable=true`
- **Impact :** Facilite le reverse engineering, permet l'attachement d'un debugger via ADB
- **Remédiation :** Désactiver en production. Utiliser les **build flavors** Android.

#### 🔴 V3 — Algorithme de Chiffrement Faible (ECB)
- **Sévérité :** Élevée
- **MASVS :** MSTG-CRYPTO-2 · CWE-327
- **Description :** Mode ECB utilisé pour le chiffrement AES — blocs identiques = chiffrés identiques.
- **Preuve :** `infosecadventures/allsafe/challenges/WeakCryptography.java`
- **Impact :** Données chiffrées vulnérables à l'analyse de patterns
- **Remédiation :** Remplacer par **AES-GCM** ou **ChaCha20-Poly1305**.

#### 🟡 V4 — Trafic Réseau Non Chiffré Autorisé
- **Sévérité :** Élevée
- **MASVS :** MSTG-NETWORK-2
- **Description :** Cleartext traffic (HTTP) autorisé pour `infosecadventures.io`.
- **Preuve :** `res/xml/network_security_config.xml`
- **Impact :** Attaque MITM possible, interception et modification des données en transit
- **Remédiation :** Supprimer l'exception cleartext. Forcer **HTTPS** pour tous les domaines.

#### 🟡 V5 — Composants Android Exportés Sans Protection
- **Sévérité :** Moyenne
- **MASVS :** MSTG-PLATFORM-1
- **Description :** Plusieurs composants exportés sans permission de protection.
- **Preuve :**
  - `infosecadventures.allsafe.ProxyActivity` — `android:exported=true`
  - `infosecadventures.allsafe.challenges.DeepLinkTask` — `android:exported=true`
  - `infosecadventures.allsafe.challenges.NoteReceiver` — `android:exported=true`
  - `infosecadventures.allsafe.challenges.DataProvider` — `android:exported=true`
- **Impact :** Une app malveillante peut invoquer ces composants depuis l'extérieur
- **Remédiation :** Ajouter `android:exported=false` ou protéger avec `android:permission`.

---

### Autres observations

- **android:allowBackup=true** (WARNING) : Données sauvegardables via ADB — MSTG-STORAGE-8
- **minSdk=23** (HIGH) : Android 6.0 obsolète avec vulnérabilités non patchées
- **Logs sensibles** (INFO) : CWE-532 — données potentiellement loguées — MSTG-STORAGE-3
- **Clipboard** (INFO) : Données copiables par d'autres apps — MSTG-STORAGE-10 (`ClipUtil.java`)
- **SSL Pinning** (SECURE) : Implémenté dans `CertificatePinning.java` 
- **Trackers** : 0/432 détectés 

---

### Recommandations prioritaires

1. 🔴 **Révoquer** toutes les clés API exposées (Firebase, Google) — action immédiate
2. 🔴 **Désactiver** `android:debuggable=true` avant tout déploiement en production
3. 🔴 **Remplacer** le chiffrement ECB par AES-GCM dans `WeakCryptography.java`
4. 🟡 **Forcer HTTPS** pour tous les domaines dans `network_security_config.xml`
5. 🟡 **Restreindre** les composants exportés ou les protéger par des permissions

---

## Schéma du flux d'analyse

```
allsafe.apk
    ├── AndroidManifest.xml
    │       ├── android:debuggable=true              ← 🔴 HIGH
    │       ├── android:allowBackup=true              ← 🟡 WARNING
    │       ├── ProxyActivity exported=true           ← 🟡 WARNING
    │       ├── DeepLinkTask exported=true            ← 🟡 WARNING
    │       └── NoteReceiver exported=true            ← 🟡 WARNING
    │
    ├── res/xml/network_security_config.xml
    │       └── cleartext traffic infosecadventures.io ← 🔴 HIGH
    │
    ├── res/values/strings.xml
    │       └── 29 secrets hardcodés                  ← 🔴 CRITIQUE
    │               ├── firebase_database_url
    │               ├── google_api_key
    │               └── google_crash_reporting_api_key
    │
    └── java/
            ├── WeakCryptography.java                 ← 🔴 HIGH (ECB)
            ├── ClipUtil.java                         ← ℹ️ INFO (clipboard)
            └── CertificatePinning.java               ← ✅ SECURE
```

---

## Conclusion

Ce lab démontre l'importance de l'analyse statique comme première étape d'un audit de sécurité mobile. MobSF a permis d'identifier automatiquement de nombreuses vulnérabilités sans exécuter l'application. Les découvertes les plus critiques — secrets hardcodés, debug activé, chiffrement faible — sont des erreurs de développement courantes qui peuvent avoir des conséquences graves en production.

Méthode appliquée :
1. Préparer l'environnement et documenter la traçabilité
2. Lancer MobSF et importer l'APK pour analyse statique
3. Analyser le manifeste, les permissions et les composants exportés
4. Examiner la configuration réseau et les secrets hardcodés
5. Corréler les findings avec OWASP MASVS
6. Produire un rapport d'audit structuré et actionnable

---

## Références

- [MobSF — GitHub](https://github.com/MobSF/Mobile-Security-Framework-MobSF)
- [OWASP MASVS](https://mas.owasp.org/MASVS/)
- [OWASP MASTG](https://mas.owasp.org/MASTG/)
- [Android Security — Documentation officielle](https://developer.android.com/topic/security/best-practices)
- [Network Security Configuration — Android](https://developer.android.com/training/articles/security-config)
- [Allsafe — GitHub](https://github.com/t0thkr1s/allsafe)
