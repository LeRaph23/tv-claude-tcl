# Jour J : TCL 55P638, depuis un MacBook Air

## La veille, chez toi (20 min)

1. Si `brew` n'existe pas : installe Homebrew depuis https://brew.sh.
2. Les outils :

```bash
brew install python git openjdk@17 android-commandlinetools
yes | sdkmanager --licenses
sdkmanager "platforms;android-34" "build-tools;34.0.0"
```

3. Les deux repos, CleanYourTV figé sur la version vérifiée :

```bash
mkdir -p ~/tv && cd ~/tv
git clone https://github.com/LeRaph23/tv-claude-tcl
git clone https://github.com/Gathin23/cleanyourtv
cd cleanyourtv
git checkout 9071ec8
python3 -m venv tools/venv && tools/venv/bin/pip install -r tools/requirements.txt
```

4. Garder Prime Video et les mises à jour de la TV (met un `#` devant ces deux lignes de la liste) :

```bash
sed -i '' -e 's/^com\.amazon\.amazonvideo\.livingroom/# &/' -e 's/^com\.tcl\.ttvs/# &/' examples/tcl-g07-google-tv/debloat.txt
grep -n "amazonvideo\|ttvs" examples/tcl-g07-google-tv/debloat.txt
```

Les deux lignes affichées doivent commencer par `#`.

5. Compiler une première fois le launcher, pour que les erreurs arrivent chez toi :

```bash
cd launcher && JAVA_HOME=/opt/homebrew/opt/openjdk@17 ANDROID_HOME=/opt/homebrew/share/android-commandlinetools ./gradlew assembleRelease && cd ..
```

Ça doit finir sans `FAILURE`.

## Chez tes parents

### Sur la TV (télécommande)

1. Si **Ligue 1+** n'est pas installée, installe-la depuis le Play Store.
2. **Paramètres > Système > À propos** : appuie 7 fois sur OK sur **« Build de l'OS Android TV »**.
3. Retour, **Options pour les développeurs** : active **Débogage USB**.
4. **Paramètres > Réseau et Internet** : note l'**adresse IP** de la TV.
5. Débranche ce qui est branché en USB sur la TV.

### Sur le Mac (même Wi-Fi que la TV)

Ouvre Terminal. Si macOS demande d'autoriser Terminal à accéder au **réseau local**, accepte.

```bash
cd ~/tv/cleanyourtv
export TV_HOST=192.168.1.XX        # l'IP de la TV
tools/venv/bin/python -I tools/tv.py --probe
```

Sur la TV : « Autoriser le débogage ? » → coche **Toujours**, puis **OK**. Relance la commande si besoin.
**Vérifie que la réponse contient `G07`.** Si ce n'est pas le cas, arrête-toi là.

**Ligue 1+ dans le dock** :

```bash
tools/venv/bin/python -I tools/tv.py 'pm list packages' | grep -i -e ligue -e lfp
open -e ../tv-claude-tcl/theme/config.json
```

Dans le fichier qui s'ouvre, ajoute l'identifiant affiché (sans `package:`) en 3e ligne de `dock_apps`, entre guillemets et suivi d'une virgule, puis enregistre.

**Les trois commandes, dans cet ordre** (attends la fin de chacune) :

```bash
HOME_KEY_SERVICE=1 launcher/build-and-install.sh
tools/disable.sh examples/tcl-g07-google-tv/debloat.txt
tools/theme.sh ../tv-claude-tcl/theme
```

1. Installe le launcher et en fait l'accueil. Il revient en arrière tout seul si ça ne marche pas.
2. Désactive pubs, télémétrie et apps TCL inutiles, après avoir enregistré l'état de la TV. S'il dit qu'il saute `setupwraith` à cause du Wi-Fi : sur la TV, oublie le réseau Wi-Fi puis reconnecte-toi, et relance la commande 2.
3. Applique le dock, le fond selon l'heure et les touches de la télécommande.

### Tester à la télécommande

- [ ] Touche **Accueil** → le nouvel écran
- [ ] myCANAL, Netflix, Ligue 1+, YouTube, Prime Video s'ouvrent depuis le dock
- [ ] Boutons **Netflix**, **YouTube**, **Prime Video** et **roue crantée** de la télécommande
- [ ] Touche **Source** → le décodeur TNT en HDMI, avec image et son
- [ ] Éteins et rallume la TV (vraiment, à la prise) : Wi-Fi connecté, nouvel accueil toujours là
- [ ] Si la navigation saccade : dans `config.json`, mets `"advanced_effects": false`, enregistre et relance `tools/theme.sh ../tv-claude-tcl/theme`

### Finir

- Sur la TV, **Options pour les développeurs** → désactive **Débogage USB**.
- Sur le Mac, ne supprime ni `~/tv` ni `~/.cleanyourtv` : l'état d'origine de la TV y est enregistré.

## Si quelque chose ne va pas

Réactive le débogage USB, puis :

```bash
cd ~/tv/cleanyourtv && export TV_HOST=192.168.1.XX
tools/restore.sh examples/tcl-g07-google-tv/debloat.txt com.google.android.apps.tv.launcherx
```

Tout revient comme avant, accueil Google TV compris. Sans le Mac : **Paramètres > Applications** sur la TV permet de réactiver une app.
