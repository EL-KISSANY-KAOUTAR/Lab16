🔐 LAB 16 – Inspection du Trafic HTTPS Android et Contournement du SSL Pinning
📌 Présentation

Dans ce laboratoire, nous avons étudié les mécanismes de protection des communications HTTPS sur Android ainsi que les techniques utilisées lors des audits de sécurité mobile pour inspecter le trafic réseau chiffré.

L'objectif principal consiste à intercepter les communications HTTPS d'une application Android en utilisant Burp Suite comme proxy d'analyse et Objection/Frida pour contourner les mécanismes de SSL Pinning empêchant normalement cette interception.

🎯 Objectifs du laboratoire

À l'issue de ce TP, nous sommes capables de :

Installer et configurer l'environnement Frida et Objection.
Configurer Burp Suite comme proxy d'interception HTTPS.
Installer un certificat CA personnalisé sur Android.
Comprendre le fonctionnement du SSL Pinning.
Désactiver dynamiquement le SSL Pinning à l'aide d'Objection.
Capturer et analyser les échanges HTTPS d'une application Android.
🖥️ Environnement Technique
Composant	Version
Système d'exploitation	Windows 10 / Windows 11
Python	3.12+
Android Platform Tools (ADB)	Dernière version
Objection	1.12.4
Frida	17.9.10
Burp Suite Community	v2026.4.3
Émulateur Android	Pixel 4a - Android 11 (API 30)
🛠️ Étape 1 : Installation des outils Frida et Objection

Afin de réaliser l'instrumentation dynamique de l'application Android, nous installons Frida et Objection à l'aide de Python.

Installation
pip install --upgrade objection frida frida-tools
Vérification
objection --version
frida --version
Résultat attendu
Objection 1.12.4
Frida 17.9.10
📱 Étape 2 : Déploiement de Frida Server sur l'émulateur Android

Frida nécessite l'exécution d'un serveur sur l'appareil Android afin de permettre l'instrumentation dynamique des applications.

Copie du binaire
adb -s emulator-5556 push frida-server /data/local/tmp/
Attribution des permissions
adb -s emulator-5556 shell chmod 755 /data/local/tmp/frida-server
Démarrage du serveur
adb -s emulator-5556 shell "/data/local/tmp/frida-server -l 0.0.0.0"
Vérification
frida-ps -Uai

Cette commande permet d'afficher les processus Android accessibles via Frida et confirme le bon fonctionnement du serveur.

🌐 Étape 3 : Configuration du Proxy Burp Suite

Pour intercepter les communications HTTPS, Burp Suite est configuré comme proxy intermédiaire entre l'application Android et Internet.

Configuration de Burp Suite
Ouvrir Proxy → Options.
Ajouter un Proxy Listener sur :
0.0.0.0:8080
Désactiver l'interception :
Intercept = OFF
Configuration de l'émulateur Android
Accéder aux paramètres Wi-Fi.
Modifier le réseau connecté.
Sélectionner Proxy Manuel.
Configurer :
Adresse : 10.0.2.2
Port : 8080

📷 Capture : Configuration du proxy Android

🔒 Étape 4 : Installation du Certificat CA Burp

Pour que l'appareil Android fasse confiance au proxy Burp Suite, il est nécessaire d'installer son certificat d'autorité de certification (CA).

Téléchargement

Depuis le navigateur Android :

http://10.0.2.2:8080

Télécharger le certificat proposé par Burp Suite.

📷 Capture : Téléchargement du certificat CA

Installation

Chemin :

Paramètres
→ Sécurité
→ Certificats
→ Installer depuis le stockage

📷 Captures : Installation du certificat sur Android

🚨 Étape 5 : Désactivation du SSL Pinning

Certaines applications implémentent le SSL Pinning afin d'empêcher les attaques de type Man-In-The-Middle et l'interception du trafic.

Pour contourner cette protection dans un contexte de test de sécurité, nous utilisons Objection.

Connexion à l'application cible
objection -g com.android.chrome explore

📷 Capture : Connexion Objection

Désactivation du SSL Pinning
android sslpinning disable

Cette commande injecte automatiquement un script Frida capable de neutraliser plusieurs mécanismes de SSL Pinning couramment utilisés :

OkHttp
TrustManager
Conscrypt
Android Network Security Config
Bibliothèques SSL personnalisées
📊 Résultats Obtenus

Après la désactivation du SSL Pinning et la configuration du proxy Burp Suite :

✅ Les requêtes HTTPS deviennent visibles dans Burp Suite.

✅ Les réponses du serveur peuvent être analysées.

✅ Les en-têtes HTTP, cookies, tokens et paramètres applicatifs sont inspectables.

✅ Le comportement réseau de l'application peut être étudié dans un contexte d'audit de sécurité mobile.

📷 Capture : Trafic HTTPS intercepté dans Burp Suite

📝 Conclusion

Ce laboratoire a permis de mettre en pratique les techniques d'analyse dynamique des applications Android en combinant Frida, Objection et Burp Suite. Nous avons compris le rôle du SSL Pinning dans la sécurisation des communications HTTPS et appris comment le contourner dans un environnement de test contrôlé afin d'observer et d'analyser le trafic réseau d'une application mobile.

Cette démarche constitue une étape essentielle dans les audits de sécurité mobile, les tests d'intrusion Android et l'évaluation de la robustesse des mécanismes de protection des applications.
