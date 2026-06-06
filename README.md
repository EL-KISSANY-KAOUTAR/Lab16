
# LAB 16 – Inspection du Trafic HTTPS Android et Contournement du SSL Pinning

## Présentation

Dans ce laboratoire, nous avons étudié les mécanismes de protection des communications HTTPS sur Android ainsi que les techniques utilisées lors des audits de sécurité mobile pour inspecter le trafic réseau chiffré.

L'objectif principal consiste à intercepter les communications HTTPS d'une application Android en utilisant **Burp Suite** comme proxy d'analyse et **Objection/Frida** pour contourner les mécanismes de SSL Pinning empêchant normalement cette interception.

---

## Objectifs du laboratoire

À l'issue de ce TP, nous sommes capables de :

- Installer et configurer l'environnement Frida et Objection.
- Configurer Burp Suite comme proxy d'interception HTTPS.
- Installer un certificat CA personnalisé sur Android.
- Comprendre le fonctionnement du SSL Pinning.
- Désactiver dynamiquement le SSL Pinning à l'aide d'Objection.
- Capturer et analyser les échanges HTTPS d'une application Android.

---

## Environnement Technique

| Composant | Version |
|---|---|
| Système d'exploitation | Windows 10 / Windows 11 |
| Python | 3.12+ |
| Android Platform Tools (ADB) | Dernière version |
| Objection | 1.12.4 |
| Frida | 17.9.10 |
| Burp Suite Community | v2026.4.3 |
| Émulateur Android | Pixel 4a - Android 11 (API 30) |

---

## Étape 1 : Installation des outils Frida et Objection

Afin de réaliser l'instrumentation dynamique de l'application Android, nous installons Frida et Objection à l'aide de Python.

**Installation**
```bash
pip install --upgrade objection frida frida-tools
```

**Vérification**
```bash
objection --version
frida --version
```

**Résultat attendu**
Objection 1.12.4
Frida 17.9.10

---

## Étape 2 : Déploiement de Frida Server sur l'émulateur Android

Frida nécessite l'exécution d'un serveur sur l'appareil Android afin de permettre l'instrumentation dynamique des applications.

**Copie du binaire**
```bash
adb -s emulator-5556 push frida-server /data/local/tmp/
```

**Attribution des permissions**
```bash
adb -s emulator-5556 shell chmod 755 /data/local/tmp/frida-server
```

**Démarrage du serveur**
```bash
adb -s emulator-5556 shell "/data/local/tmp/frida-server -l 0.0.0.0"
```

**Vérification**
```bash
frida-ps -Uai
```

Cette commande permet d'afficher les processus Android accessibles via Frida et confirme le bon fonctionnement du serveur.

---

## Étape 3 : Configuration du Proxy Burp Suite

Pour intercepter les communications HTTPS, Burp Suite est configuré comme proxy intermédiaire entre l'application Android et Internet.

**Configuration de Burp Suite**
1. Ouvrir `Proxy` → `Options`.
2. Ajouter un Proxy Listener sur : `0.0.0.0:8080`
3. Désactiver l'interception : `Intercept = OFF`

**Configuration de l'émulateur Android**
1. Accéder aux paramètres Wi-Fi.
2. Modifier le réseau connecté.
3. Sélectionner **Proxy Manuel**.
4. Configurer :
   - Adresse : `10.0.2.2`
   - Port : `8080`

![Configuration proxy Android](https://github.com/user-attachments/assets/d2d1b41e-69b8-4650-8422-5181beeacecc)

---

## Étape 4 : Installation du Certificat CA Burp

Pour que l'appareil Android fasse confiance au proxy Burp Suite, il est nécessaire d'installer son certificat d'autorité de certification (CA).

**Téléchargement**

Depuis le navigateur Android, accéder à :
http://10.0.2.2:8080
Puis télécharger le certificat proposé par Burp Suite.

![Téléchargement certificat Burp](https://github.com/user-attachments/assets/d8c817c4-09ee-4c1d-80fd-6f969a4424ee)

**Installation**

Chemin :
Paramètres → Sécurité → Certificats → Installer depuis le stockage

![Installation certificat CA](https://github.com/user-attachments/assets/bdf01aba-bf69-48f0-9f52-3ad02ce96065)

---

## Étape 5 : Désactivation du SSL Pinning

Certaines applications implémentent le SSL Pinning afin d'empêcher les attaques de type Man-In-The-Middle et l'interception du trafic.

Pour contourner cette protection dans un contexte de test de sécurité, nous utilisons Objection.

**Connexion à l'application cible**
```bash
objection -g com.android.chrome explore
```


**Désactivation du SSL Pinning**
android sslpinning disable

Cette commande injecte automatiquement un script Frida capable de neutraliser plusieurs mécanismes de SSL Pinning couramment utilisés :

- OkHttp
- TrustManager
- Conscrypt
- Android Network Security Config
- Bibliothèques SSL personnalisées

**Après la désactivation du SSL Pinning :**
- Les requêtes HTTPS deviennent visibles dans Burp Suite.
- Les réponses du serveur peuvent être analysées.
- Les en-têtes HTTP, cookies et tokens sont inspectables.
- Le comportement réseau de l'application peut être étudié.

Affichage dans Burp Suite après désactivation :

![Trafic HTTPS visible dans Burp Suite](https://github.com/user-attachments/assets/8c2af347-516b-47a8-9561-4514cde642ca)

---

## Conclusion

Ce laboratoire a permis de mettre en pratique les techniques d'analyse dynamique des applications Android en combinant **Frida**, **Objection** et **Burp Suite**.

Nous avons compris le rôle du **SSL Pinning** dans la sécurisation des communications HTTPS et appris à le contourner dans un environnement de test contrôlé afin d'observer et d'analyser le trafic réseau d'une application mobile.

Cette démarche constitue une étape essentielle dans les **audits de sécurité mobile**, les **tests d'intrusion Android** et l'évaluation de la robustesse des mécanismes de protection des applications.
