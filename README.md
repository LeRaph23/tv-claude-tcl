# tv-claude-tcl

La TCL des parents, sans pubs et plus rapide. Tout repose sur [CleanYourTV](https://github.com/Gathin23/cleanyourtv) de Gathin (licence MIT) : ses outils, son launcher et ses listes testées sur 32 modèles TCL. Ce repo ne contient que nos réglages : le dock, le fond et les boutons de la télécommande.

**Le jour J, suis [JOUR-J.md](JOUR-J.md)** : toutes les commandes pour la 55P638, dans l'ordre, depuis un Mac.

Rien n'est flashé, rooté ni supprimé : les apps sont **désactivées**, et une commande remet la TV comme avant.

## Comment ça marche

- **Un ordi, une seule fois**, sur le même Wi-Fi que la TV, pendant 20 à 30 minutes. Il pilote la TV par le réseau (ADB), sans câble.
- Ensuite l'ordi ne sert plus. La TV démarre directement sur le nouvel écran d'accueil.
- Une mise à jour de la TV peut réactiver des apps. Il suffit alors de reprendre l'étape 4.

## 0. Avant de commencer

- **La TV : TCL 55P638 (2022)**. Google TV sur Android 11, plateforme TCL « G07 » (Realtek RTD2851A, 2 Go de RAM). Gathin a une liste pour cette plateforme, testée sur deux vraies TV, dont une en France : `examples/tcl-g07-google-tv`.
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
export TV_HOST=192.168.1.50          # l'IP de la TV
```

## 3. Vérifier la TV

```bash
tools/venv/bin/python -I tools/tv.py --probe
```

Accepte « Autoriser le débogage ? » sur la TV (coche « Toujours »). La commande affiche le modèle, la version d'Android et le firmware. La plateforme doit indiquer **G07** (ex. `G07_4K_GB_NF`) : c'est la liste `tcl-g07-google-tv`. **Si ce n'est pas G07, ne va pas plus loin** : la [méthode avec Claude Code](#la-méthode-la-plus-sûre--claude-code) détecte les apps de la TV elle-même.

## 4. Nettoyer, puis changer l'écran d'accueil

D'abord, ouvre `examples/tcl-g07-google-tv/debloat.txt` et mets un `#` au début de ces deux lignes :

- `com.amazon.amazonvideo.livingroom` : **c'est Prime Video**, que la liste désactive par défaut ;
- `com.tcl.ttvs` : c'est aussi le service de mise à jour de la TV. On garde les mises à jour.

Le reste convient à tes parents : la liste coupe l'Assistant Google (le micro) et le partage d'écran TCL, et garde la TV, les sources HDMI, Netflix, YouTube et Chromecast.

```bash
HOME_KEY_SERVICE=1 launcher/build-and-install.sh                    # 1. installe le launcher et en fait l'accueil
tools/disable.sh examples/tcl-g07-google-tv/debloat.txt             # 2. désactive pubs, télémétrie, apps TCL inutiles
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
tools/restore.sh examples/tcl-g07-google-tv/debloat.txt com.google.android.apps.tv.launcherx
```

Ça réactive tout, et l'accueil Google TV revient. Même sans ordi, **Paramètres > Applications** sur la TV permet de réactiver une app.

## Nos réglages (`theme/config.json`)

| Clé | Rôle |
|---|---|
| `dock_apps` | La rangée du haut, dans l'ordre : myCANAL, Netflix, Ligue 1+, YouTube, Prime Video. Les apps absentes sont ignorées. **Ligue 1+ est à ajouter** en 3e position une fois son identifiant lu sur la TV (`tools/tv.py 'pm list packages' \| grep -i ligue`) |
| `columns` | 6 tuiles par rangée : dock et icônes plus petits |
| `dock_title` | `false` : pas de nom d'app en grand au-dessus du dock |
| `advanced_effects` | Animations façon Apple TV : la tuile se soulève, l'icône zoome, la couleur d'ambiance glisse. Plus gourmand en mémoire : si ça saccade, passe-le à `false` |
| `sky_intensity` | Fond qui suit l'heure (nuit, jour, coucher de soleil), recalculé une fois par minute. 0 à 100 ; plus haut = plus visible |
| `background_color`, `wallpaper` | Bleu nuit sous le ciel, pas de fond d'écran (le ciel le remplace) |
| `app_keys`, `settings_key_keycodes` | Les touches Netflix, YouTube, Prime Video et la roue crantée (réglages) de la télécommande : sans ça, elles ne font plus rien une fois l'accueil Google TV désactivé. Codes vérifiés par Gathin sur une G07 française avec la télécommande Bluetooth ; ils demandent `HOME_KEY_SERVICE=1` à l'installation. La touche Source marche toute seule sur la G07 |

Un `config.json` invalide est ignoré en entier par le launcher : un réglage raté ne peut pas casser l'écran d'accueil. Pour modifier : édite le fichier, puis relance `tools/theme.sh ../tv-claude-tcl/theme`.

## La méthode la plus sûre : Claude Code

C'est la méthode prévue par Gathin. Sur l'ordi, dans le dossier `cleanyourtv`, lance `claude`, colle tout le contenu de [`PROMPT.md`](https://github.com/Gathin23/cleanyourtv/blob/9071ec8/PROMPT.md), puis ajoute à la fin :

```
IP de la TV : <IP>.
Utilise les outils de ce dossier (tools/, launcher/, examples/) plutôt que d'en écrire.
Modèle : TCL 55P638 (plateforme G07). Utilise examples/tcl-g07-google-tv/debloat.txt, sans désactiver Prime Video (com.amazon.amazonvideo.livingroom) ni com.tcl.ttvs (mises à jour).
Installe le launcher avec HOME_KEY_SERVICE=1.
Ce que mes parents utilisent :
- apps : myCANAL, Netflix, Ligue 1+, YouTube, Prime Video. Garde-les, avec le Play Store pour leurs mises à jour ;
- la TNT passe par un décodeur branché en HDMI : la touche Source, la liste des sources et l'entrée HDMI doivent marcher ;
- ni le micro ni l'Assistant Google : tu peux les désactiver ;
- pas de Chromecast : tu peux le désactiver.
Trouve le nom du paquet de Ligue 1+ sur la TV et ajoute-le en 3e position de dock_apps dans ../tv-claude-tcl/theme/config.json.
Vérifie avec la vraie télécommande les touches Netflix, YouTube, Prime Video et la roue crantée, et corrige app_keys / settings_key_keycodes si besoin (getevent -lt).
À la fin, applique le thème avec tools/theme.sh ../tv-claude-tcl/theme.
Ne désactive rien sans me demander.
```

Il sauvegarde l'état de la TV, cherche les apps à désactiver sur cette TV précisément, demande avant chaque choix et vérifie chaque étape avec une capture d'écran de la TV.
