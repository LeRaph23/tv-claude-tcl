# tv-claude-tcl

La TCL des parents, sans pubs et plus rapide. Tout repose sur [CleanYourTV](https://github.com/Gathin23/cleanyourtv) de Gathin (licence MIT) : ses outils, son launcher et ses listes testées sur 32 modèles TCL. Ce repo ne contient que nos réglages : le dock, le fond et les boutons de la télécommande.

Rien n'est flashé, rooté ni supprimé : les apps sont **désactivées**, et une commande remet la TV comme avant.

## Comment ça marche

- **Un ordi, une seule fois**, sur le même Wi-Fi que la TV, pendant 20 à 30 minutes. Il pilote la TV par le réseau (ADB), sans câble.
- Ensuite l'ordi ne sert plus. La TV démarre directement sur le nouvel écran d'accueil.
- Une mise à jour de la TV peut réactiver des apps. Il suffit alors de reprendre l'étape 4.

## 0. Avant de commencer

- **Le modèle exact de la TV** : étiquette au dos, ou **Paramètres > Système > À propos**. Gathin a une liste testée sur une TCL 65C735 française (Google TV) ; si le modèle est différent, la liste à utiliser doit correspondre à la plateforme de la TV, pas seulement à la marque.
- Rien de branché en USB sur la TV pendant l'opération : certaines coupent l'alimentation du port quand le débogage est activé.

## 1. Sur la TV

1. **Paramètres > Système > À propos** : appuie 7 fois sur OK sur **« Build de l'OS Android TV »**.
2. Retour, puis **Options pour les développeurs** : active **Débogage USB** (et **Débogage réseau** s'il existe).
3. Note l'**adresse IP** de la TV : **Paramètres > Réseau**.

## 2. Sur l'ordi

Il faut Python 3, Git, et pour compiler le launcher : JDK 17 et le SDK Android 34.

```bash
# Mac
brew install python git openjdk@17 android-commandlinetools
sdkmanager "platforms;android-34" "build-tools;34.0.0"
```

Récupère les deux repos, CleanYourTV figé sur la version vérifiée :

```bash
git clone https://github.com/LeRaph23/tv-claude-tcl
git clone https://github.com/Gathin23/cleanyourtv
cd cleanyourtv
git checkout 9071ec8
python3 -m venv tools/venv && tools/venv/bin/pip install -r tools/requirements.txt
cp themes/aurora/wallpaper.jpg ../tv-claude-tcl/theme/
export TV_HOST=192.168.1.50          # l'IP de la TV
```

## 3. Vérifier la TV

```bash
tools/venv/bin/python -I tools/tv.py --probe
```

Accepte « Autoriser le débogage ? » sur la TV (coche « Toujours »). La commande affiche le modèle, la version d'Android et le firmware. Choisis la liste de `examples/` qui correspond (`tcl-65c735` pour une TCL C735 française en Google TV). **En cas de doute, ne va pas plus loin** : la [méthode avec Claude Code](#la-méthode-la-plus-sûre--claude-code) détecte les apps de la TV elle-même.

## 4. Nettoyer, puis changer l'écran d'accueil

D'abord, relis la liste (`examples/tcl-65c735/debloat.txt`) et mets un `#` devant ce que tes parents utilisent. Le lot 4 coupe l'**Assistant Google, donc le bouton micro** de la télécommande (`katniss`, `tv.assistant`) et AirPlay.

```bash
INPUT_KEY_SERVICE=1 launcher/build-and-install.sh                   # 1. installe le launcher et en fait l'accueil
tools/disable.sh examples/tcl-65c735/debloat.txt                    # 2. désactive pubs, télémétrie, apps TCL inutiles
tools/theme.sh ../tv-claude-tcl/theme                               # 3. notre dock et notre fond
```

- **Le launcher d'abord** : le dernier lot de la liste (`setupwraith`, l'assistant de 1re configuration) ne doit être désactivé qu'une fois le nouvel accueil en place.
- `build-and-install.sh` démarre le nouveau launcher **avant** de toucher à l'ancien, et revient en arrière si l'accueil ne bascule pas.
- `disable.sh` enregistre l'état de la TV avant tout changement. Il refuse de désactiver `setupwraith` si le Wi-Fi a été enregistré pendant la 1re configuration (Android effacerait le réseau) : il l'indique. Dans ce cas, oublie puis rejoins le Wi-Fi dans **Paramètres > Réseau**, et relance la commande 2.

## 5. Terminer

- Teste avec la télécommande : Accueil, Netflix, YouTube, la touche Source, une chaîne TNT, une entrée HDMI.
- **Désactive le débogage USB** dans les options pour les développeurs : laissé actif, n'importe quel appareil du Wi-Fi peut demander à piloter la TV.

## Tout annuler

```bash
tools/restore.sh examples/tcl-65c735/debloat.txt com.google.android.apps.tv.launcherx
```

Ça réactive tout, et l'accueil Google TV revient. Même sans ordi, **Paramètres > Applications** sur la TV permet de réactiver une app.

## Nos réglages (`theme/config.json`)

| Clé | Rôle |
|---|---|
| `dock_apps` | Les apps de la rangée du haut, dans l'ordre. Les absentes sont ignorées |
| `background_color`, `wallpaper` | Fond bleu nuit plus clair que le noir par défaut, avec le fond d'écran « aurora » de Gathin, pour que les tuiles ressortent |
| `app_keys` | Les touches Netflix, YouTube et Prime Video de la télécommande. Sans ça, elles ne font plus rien une fois l'accueil Google TV désactivé (codes relevés sur la 65C735, à vérifier sur un autre modèle) |

Un `config.json` invalide est ignoré en entier par le launcher : un réglage raté ne peut pas casser l'écran d'accueil. Pour modifier : édite le fichier, puis relance `tools/theme.sh ../tv-claude-tcl/theme`.

## La méthode la plus sûre : Claude Code

C'est la méthode prévue par Gathin. Sur l'ordi, dans le dossier `cleanyourtv`, lance Claude Code et colle le contenu de [`PROMPT.md`](https://github.com/Gathin23/cleanyourtv/blob/9071ec8/PROMPT.md) en y mettant le modèle de la TV et en précisant d'utiliser `../tv-claude-tcl/theme`. Il sauvegarde l'état de la TV, cherche les apps à désactiver sur ta TV précisément, te demande avant chaque choix et vérifie chaque étape avec une capture d'écran de la TV.
