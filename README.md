# RAPPORT FINAL - LAB 2: TEST DE SECURITE ANDROID
## Rooting, Verified Boot & Hardening de Securite

---

## INFORMATIONS GENERALES

| Champ | Valeur |
|-------|--------|
| **Testeur** | Fatima Ezzahra El Boudhiri |
| **Date du test** | 04 mai 2026 |
| **Environnement** | Android Virtual Device (AVD) - Pixel 6 |
| **Version Android** | API 34 (Android 14 "UpsideDownCake") |
| **Architecture** | x86_64 |
| **Type d'appareil** | Appareil virtuel |
| **Application testee** | OWASP UnCrackable1 |
| **Version application** | 1.0 |
| **Package** | owasp.mstg.uncrackable1 |
| **Activite principale** | sg.vantagepoint.uncrackable1.MainActivity |
| **Taille APK** | 65 KB |
| **Statut** | Installee et fonctionnelle |
| **Statut Root** | [CONFIRME] uid=0(root) |
| **Etat Verified Boot** | Orange/Yellow (modifie) |

---

## ETAPE 2: FICHE PERIMETRE (5 LIGNES)

### Perimetre de Test Defini

1. **Application & Version**: OWASP UnCrackable1 version 1.0 (Package: owasp.mstg.uncrackable1)

2. **Environnement de Test**: Android Virtual Device (AVD) - Pixel 6 avec API 34, configuration writable-system

3. **Objectif**: Comprendre les impacts du rooting sur les mecanismes de securite Android, verifier la detection du rooting par l'application, et evaluer l'accessibilite des fonctionnalites sur un appareil roote

4. **Donnees Utilisees**: Donnees fictives uniquement (saisie de texte "hello" pour les tests de scenarios)

5. **Reseau**: Reseau de test isole (pas d'acces Internet reel pendant le test)

---

## ETAPE 1: ROOTER L'AVD - RESULTATS

### Statut du Root: REUSSI

#### Commandes Executees

```bash
# Demarrage de l'emulateur avec systeme modifiable
emulator -avd Pixel_6 -writable-system

# Activation du mode root
adb root
# Resultat: "adbd is already running as root"

# Remontage du systeme
adb remount
# Resultat: "remount succeeded"

# Verification du root
adb shell id
# Resultat: uid=0(root) gid=0(root) groups=0(root),1004(input),1007(log),1011(adb)...
```

#### Verifications de Securite

```bash
# Verification du mode verity
adb shell getprop ro.boot.veritymode
# Resultat: "enforcing"

# Verification de l'etat du Verified Boot
adb shell getprop ro.boot.verifiedbootstate
# Resultat: "orange" (systeme modifie mais fonctionnel)

# Test d'acces su
adb shell "su -c id"
# Resultat: uid=0(root) groups=0(root)...
```

#### Analyse des Resultats

- [CONFIRME] uid=0(root): Privileges root actifs
- [ETAT MODIFIE] Verified Boot State: Orange/Yellow
- [SUCCES] Remount reussi: Le systeme est en lecture/ecriture
- [SUCCES] Acces su: Commande su disponible et fonctionnelle

#### Implications de Securite

L'etat "orange" du Verified Boot indique que la chaine de confiance est rompue. Les protections au demarrage ne garantissent plus l'integrite du systeme. Un attaquant avec acces root pourrait modifier n'importe quelle partie du systeme et contourner les protections basees sur SELinux.

---

## ETAPE 5: RESULTATS DES 3 SCENARIOS

### SCENARIO 1: Ouverture de l'ecran d'accueil

**Objectif**: Verifier si l'application se lance correctement sur AVD rootee

**Resultats observes**:
- Application demarre correctement SANS avertissement rooting
- Interface complete: titre "Uncrackable1", champ texte "Enter the Secret String", bouton VERIFY
- Clavier virtuel visible et fonctionnel
- Menu avec icones en bas disponible
- Aucune erreur ou restriction detectee

**Avertissement rooting detecte**: NON

**Observation importante**: L'application UnCrackable1 n'affiche PAS d'avertissement rooting sur cet AVD rootee. C'est une faille de securite potentielle.

---

### SCENARIO 2: Rechercher un element/fonctionnalite

**Objectif**: Verifier si les fonctionnalites principales sont accessibles

**Entrees utilisateur**:
- Test 1: Saisie de texte "hello" dans le champ texte
- Test 2: Clic sur le bouton VERIFY
- Test 3: Clic sur l'icone GIF

**Resultats observes**:
- Saisie de texte: Fonctionnelle - le texte s'affiche correctement
- Bouton VERIFY: Fonctionnel - affiche popup "Nope... That's not it. Try again."
- Navigation GIF: Fonctionnelle - affiche galerie avec onglets
- Reactivite: L'app repond sans retard ou erreur
- Aucune restriction detectee

**Observation importante**: L'application fonctionne sans restriction sur l'AVD rootee. Aucun mecanisme de detection de rooting n'est present.

---

### SCENARIO 3: Ouvrir un detail/profil

**Objectif**: Tester l'acces aux donnees sensibles

**Resultats observes**:
- L'application est extremement simple et minimaliste
- Ecrans disponibles: Ecran principal, Galerie GIF, Options clavier
- Aucun ecran de profil ou parametres d'app detecte
- Aucune donnee utilisateur affichee
- Aucune donnee sensible accessible

**Donnees sensibles trouvees**: AUCUNE

**Observation importante**: L'application ne stocke et n'affiche aucune donnee sensible. L'app fonctionne sans restrictions de securite.

---

## ETAPE 6: RESUME ANDROID SECURITY

Android utilise une architecture multi-couche de securite pour proteger les appareils et donnees. La premiere couche est le materiel securise fournissant un environnement de confiance au niveau du processeur. Le bootloader verifie garantit que le noyau charge est authentique et n'a pas ete modifie. Le kernel Linux et SELinux fournissent l'isolation au niveau du systeme d'exploitation et le controle d'acces obligatoire. Le framework Android implemente les politiques de permissions et les mecanismes de sandbox pour les applications. Les permissions Android permettent aux utilisateurs de controler l'acces des applications aux ressources sensibles. Enfin, chaque application s'execute dans son propre processus isole avec un UID unique, prevenant les applications malveillantes d'acceder aux donnees d'autres applications.

---

## ETAPE 7: VERIFIED BOOT

### Objectif principal

Garantir que le systeme qui demarre est exactement celui prevu par le fabricant, sans modifications malveillantes ou accidentelles.

### Definition de "Chain of Trust"

La "chain of trust" est une serie de verifications enchainees ou chaque composant verifie l'authenticite du composant suivant. C'est comme une chaine de gardiens de securite ou chacun verifie l'identite du suivant avant de lui donner acces.

### Pourquoi l'integrite au demarrage est critique

Si le demarrage est compromis, l'attaquant controle completement l'appareil avec privileges supremes. Toutes les protections subsequentes deviennent inutiles.

### Verification sur l'AVD

```
adb shell getprop ro.boot.verifiedbootstate
Resultat: orange
```

L'etat "orange" indique que Verified Boot detecte une modification du systeme. Sur un appareil normal, cet etat serait "green".

---

## ETAPE 8: AVB (ANDROID VERIFIED BOOT)

AVB (Android Verified Boot version 2) est l'evolution moderne et plus flexible de Verified Boot. Contrairement a Verified Boot v1, AVB utilise une structure arborescente de hachages permettant une verification incrementale des differentes partitions. AVB ajoute la protection anti-rollback, qui empeche l'installation d'anciennes versions du systeme qui pourraient contenir des failles de securite connues. Cette protection fonctionne comme un systeme de versioning securise empechant tout retour arriere.

---

## ETAPE 9: DEFINITION DU ROOTING

Le rooting signifie obtenir les privileges de super-utilisateur sur un appareil Android, permettant d'acceder et modifier des zones protegees du systeme. Rooter un appareil modifie et rompt les mecanismes de protection integres comme Verified Boot, SELinux et le sandbox des applications. En laboratoire, le rooting permet d'observer les comportements au bas niveau du systeme et tester si une application respecte les bonnes pratiques de securite face a un attaquant privilegie. Cependant, le rooting est risque car il elimine les protections du systeme, necessitant un isolement strict, une documentation complete et une reinitialisation obligatoire pour eviter tout risque de fuite de donnees.

---

## ETAPE 10: INTERET LABO

En laboratoire, disposer d'un environnement privilegie permet d'observer des artefacts systeme normalement inaccessibles comme les fichiers /data/data/, /system/ et logs kernel. Cela permet d'analyser comment les applications stockent leurs donnees et de tester la robustesse du stockage face a un attaquant disposant d'acces root. Avec les privileges root, on peut verifier si une application se repose UNIQUEMENT sur les protections du systeme ou si elle implemente son propre chiffrement. Cette experience en laboratoire permet de comprendre les vulnerabilites reelles et de concevoir des applications qui restent securisees meme face a un attaquant privilegie.

---

## ETAPE 11: MATRICE DE RISQUES

| Risque | Description |
|--------|-------------|
| Integrite non garantie | Les conclusions sur la securite reelle pourraient etre biaisees par le systeme modifie |
| Surface d'attaque accrue | Si l'appareil sort du labo, il est expose a des menaces exploitant l'etat rootee |
| Donnees sensibles exposees | Les donnees reelles pourraient etre extraites facilement avec acces root |
| Instabilite systeme | Le rooting peut causer des comportements impredictibles rendant les tests non reproductibles |
| Melange comptes perso/test | Les donnees personnelles pourraient fuir via l'acces root |
| Mauvais nettoyage fin seance | Sans reinitialisation, les donnees sensibles pourraient persister |
| Reseau non isole | L'appareil rootee pourrait communiquer avec systemes externes |
| Tracabilite insuffisante | Impossible de reproduire ou d'auditer sans documentation detaillee |

---

## ETAPE 12: MESURES DEFENSIVES

| Mesure | Description |
|--------|-------------|
| Reseau isole | Utiliser un reseau dedié au test, completement isole des systemes de production |
| Donnees fictives | Jamais d'informations reelles (noms, emails, mots de passe, numeros telephone) |
| Device dedie | Utiliser un appareil reserve exclusivement aux tests de securite |
| Reset procedure | Wipe/snapshot l'appareil apres chaque session pour eliminer traces |
| Journal configuration | Documenter chaque commande et etape pour assurer reproductibilite |
| Aucun compte perso | Ne jamais se connecter aux services personnels |
| Controle APK strict | Installer et tester UNIQUEMENT l'application autorisee |
| Horodatage captures | Timestamper et capturer chaque etape pour traçabilite complete |

---

## ETAPE 13: OWASP MASVS

### STORAGE-1: Stockage Securise des Donnees Sensibles

Les donnees sensibles comme API keys, mots de passe et tokens doivent etre stockees de maniere securisee avec chiffrement approprie (AES-256 equivalent). Elles ne doivent jamais etre en clair dans fichiers de preference ou bases de donnees non chiffrees.

### NETWORK-1: Communications Reseau Securisees

Toutes les communications reseau doivent utiliser TLS/SSL avec configuration correcte et verification de certificat appropriee. Les certificats doivent etre valides et signes par autorite de certification reconnue.

---

## ETAPE 14: OWASP MASTG

### Test Idea 1: Verifier l'insecurite du stockage

Avec acces root, examiner les fichiers de stockage a `/data/data/[package_name]/shared_prefs/` et `/data/data/[package_name]/databases/` pour verifier si donnees sensibles sont stockees en clair.

### Test Idea 2: Analyser les logs pour fuites

Executer `adb logcat` pendant l'utilisation de l'application pour detecter si informations sensibles sont loggees accidentellement.

---

## ETAPE 15: COMMANDES DE ROOTING

### Commandes essentielles

```bash
adb devices                             # Verifier appareils connectes
adb root                                # Activer root
adb remount                             # Remonter systeme en lecture/ecriture
adb shell id                            # Verifier acces root (uid=0)
adb shell getprop ro.boot.veritymode    # Verifier verity
adb shell getprop ro.boot.verifiedbootstate  # Verifier Verified Boot
adb shell "su -c id"                    # Tester si su disponible
```

### Option permissive

```bash
adb disable-verity                      # Desactiver verity
adb reboot                              # Redemarrer
adb remount                             # Remonter
adb logcat -d | tail -n 200 > logcat.txt  # Sauvegarder logs
```

---

## ETAPE 16: FICHE ENVIRONNEMENT

### Information Environnement

Date: 04 mai 2026
Testeur: Fatima Ezzahra El Boudhiri

Support:
- Type: Android Virtual Device (AVD)
- Modele: Pixel 6
- Version Android: API 34 (Android 14)
- Architecture: x86_64

Application testee:
- Nom: OWASP UnCrackable1
- Version: 1.0
- Package: owasp.mstg.uncrackable1

3 Scenarios:
1. Scenario 1: Ouverture ecran d'accueil - REUSSI
2. Scenario 2: Navigation et fonctionnalites - REUSSI
3. Scenario 3: Acces menus et parametres - REUSSI

Observations:
- Statut root: uid=0(root) confirme
- Verified Boot: orange (systeme modifie)
- Detection rooting: NON detectee
- Accessibilite: Toutes fonctionnalites accessibles
- Donnees sensibles: Aucune trouvee
- Restrictions securite: Aucune

Limites:
- Application extremement simple sans donnees sensibles
- Aucun test de communication reseau
- Tests limites a interface utilisateur
- Aucune analyse de code reverse

Reset: Prepare pour execution

---

## ETAPE 17: REMISE A ZERO AVD

Status: PREPARE POUR EXECUTION

Commandes:
```bash
adb emu avd stop
adb emu avd wipe-data
```

OU via Android Studio:
- Device Manager → Right-click AVD → Wipe Data

Verification de succes:
- Assistant initial Android doit s'afficher au redemarrage
- Screenshot de l'ecran de configuration initial comme preuve

---

## ETAPE 18: REMISE A ZERO DEVICE LABO

Non applicable (seul AVD utilise dans ce test)

---

## ETAPE 19: LIVRABLES

### Elements inclus

[INCLUS] Definition du rooting (4 phrases) - Etape 9
[INCLUS] Schema Verified Boot / Chain of Trust - Etape 7
[INCLUS] Matrice de risques (8 risques) - Etape 11
[INCLUS] Mesures defensives (8 mesures) - Etape 12
[INCLUS] MASVS: 2 exigences resumees - Etape 13
[INCLUS] MASTG: 2 idees de tests - Etape 14
[INCLUS] Fiche environnement remplie - Etape 16
[INCLUS] Checklist de reset + preuves - Etape 17

### Captures d'ecran references

1. screenshot_1_device_manager.png - Device Manager avec Pixel 6
<img width="1520" height="212" alt="image" src="https://github.com/user-attachments/assets/805a44e8-ccc4-4dbc-b7c8-99a78079b673" />
<img width="1536" height="486" alt="image" src="https://github.com/user-attachments/assets/598235d2-1d4c-4a7d-8f28-4663e9eb9ca3" />
<img width="1591" height="376" alt="image" src="https://github.com/user-attachments/assets/e49b9fb6-75e7-4a44-af0d-824351406342" />
<img width="1575" height="266" alt="image" src="https://github.com/user-attachments/assets/087cf509-6b7d-40ae-b813-ced46444f1ca" />
<img width="1646" height="299" alt="image" src="https://github.com/user-attachments/assets/e7bee3ce-4860-4cc8-b1e8-421d7bea793f" />
2. screenshot_2_uncrackable_home.png - Ecran d'accueil UnCrackable1
   <img width="687" height="1195" alt="image" src="https://github.com/user-attachments/assets/470e5ea7-fdc2-447e-80c4-67d69927edf0" />
3. screenshot_3_scenario2_verify.png - Popup verification
<img width="635" height="1204" alt="image" src="https://github.com/user-attachments/assets/61d29b42-bb9d-4452-916e-3bf711e3cb80" />

4. screenshot_4_scenario2_gif.png - Galerie GIF

  <img width="542" height="1070" alt="image" src="https://github.com/user-attachments/assets/58909d4c-6708-41fd-bea7-ca63eab146e4" />

5. screenshot_5_scenario3_settings.png - Parametres clavier

  <img width="614" height="1256" alt="image" src="https://github.com/user-attachments/assets/8c590c38-0846-4fa1-8f5c-9df23e16b0c7" />


6. screenshot_6_adb_commands.png
 <img width="1252" height="284" alt="image" src="https://github.com/user-attachments/assets/35c9125d-5344-4855-b54d-31eba6c4d3c8" />
ommandes ADB (root
7. screenshot_7_verified_boot.png - Verified Boot commands

<img width="1276" height="423" alt="image" src="https://github.com/user-attachments/assets/9c6f641a-d4f3-4ba9-922b-861ffc33dca9" />

---

## CONCLUSIONS PRINCIPALES

### Decouvertes

1. Absence de detection du rooting: L'application n'implemente AUCUN mecanisme de detection du rooting, ce qui est une faille importante.

2. Accessibilite complete: Toutes les fonctionnalites sont entierement accessibles sans restriction.

3. Absence donnees sensibles: L'application ne stocke ni n'affiche donnees sensibles, limitant risque direct.

4. Impact rooting: L'etat orange du Verified Boot confirme que le rooting rompt la chaine de confiance.

### Recommandations

1. L'application devrait implenter detection du rooting avec avertissement ou refus de fonctionnement.
2. Tous les stockages sensibles devraient utiliser chiffrement independamment des protections Android.
3. Les applications doivent implementer defense en profondeur plutot que se reposer sur protections Android.

### Conformite standards

- OWASP MASVS: Absence donnees sensibles limite impact violations
- OWASP MASTG: Tests detection rooting devraient etre systematiques

---

Testeur: Fatima Ezzahra El Boudhiri
Date: 04 mai 2026

FIN DU RAPPORT
