# AutoRPM for SimHub

Plugin C# .NET Framework 4.8 utilisant [Lovely Car Data](https://github.com/Lovely-Sim-Racing/lovely-car-data). Par défaut, AutoRPM privilégie le modèle reconnu par ATSR Hub et sa copie locale de Lovely Car Data. Sans ATSR, il utilise l'identifiant SimHub et ses correspondances intégrées, avec une couverture potentiellement plus limitée.
Il fournit les seuils RPM, états et couleurs de LED pour des barres de 8 à 15 LED (propriétés `N8_*` à `N15_*`). Le pilotage du matériel et la luminosité restent configurés dans SimHub.

## État au 28 septembre 2026

- **0.4.1** : stratégie ATSR pour tous les jeux, avec repli générique lorsque le profil spécifique est introuvable. La limite provient d'ATSR puis de SimHub ; en l'absence de limite valide, l'interface l'indique et les LED restent éteintes. Seuils par défaut ATSR à 88/93/98 %, interpolation et couleurs génériques AutoRPM vert/jaune/rouge. Ce repli ne reproduit pas les réglages personnalisés de chaque périphérique ATSR. Aucun profil constructeur Peugeot n'est inventé pour Assetto Corsa Rally.
- `AutoRPM.IsGenericProfile` distingue le repli d'un profil spécifique ; `AutoRPM.ProfileError` conserve la cause de l'échec. Les scripts de barre existants restent compatibles. 113 vérifications réussies ; essai réel tous jeux non effectué.

- **0.4.0** : priorité à `ATSRHubMain.Telemetry.CurrentCarId` pour reconnaître le modèle derrière une livrée. Les données sont lues en priorité dans le dossier Documents Windows `SimHub\ATSR\lovely-car-data\data`, sans le modifier. Un fichier absent ou invalide utilise le cache AutoRPM puis le téléchargement Lovely, avec conservation du cache valide en cas de panne réseau.
- La page et `AutoRPM.DataSource` affichent la provenance réelle des données ; `AutoRPM.IdentificationSource` indique séparément ATSR ou SimHub. ATSR Hub EVO utilise lui-même Lovely Car Data pour ces profils RPM.
- 89 vérifications réussies en 0.4.0, dont le profil Corvette local ATSR, les modèles LMP3 distincts, le repli et des trames SimHub simulées. Validation en jeu de la nouvelle priorité encore à effectuer. Les scripts LED existants restent utilisables.
- Les points ci-dessous documentent les validations des versions précédentes.

- Compilation réussie, sans avertissement, contre les DLL de l'installation locale **SimHub 9.12.8** située dans `C:\Program Files (x86)\SimHub2\SimHub`. Le document initial indiquait 9.8.1 ; cette ancienne version n'a pas été testée.
- Version 0.3.1 : 48 vérifications hors réseau réussies, dont une trame SimHub simulée avec identifiant voiture, chargement du profil, allumage des 12 LED et extinction au point mort. Construction et rendu WPF également vérifiés. Les essais réseau Ferrari/Porsche avaient réussi en 0.3.0.
- Chargement réseau réel réussi : Ferrari 296 LMGT3 Evo (6 LED source) et Hyper_Porsche 963 Custom Team 2023 (10 LED source).
- Assembly/classe/préfixe `AutoRPM`, cache `%LOCALAPPDATA%\AutoRPM\cache`.
- Menu et page confirmés dans SimHub par captures utilisateur. Version 0.3.1 installée après fermeture de SimHub ; copie vérifiée par SHA-256, ancienne DLL sauvegardée dans `backups`. Les aperçus des effets scriptés ont été observés par l'utilisateur ; la correction d'identification reste à valider dans une session LMU réelle.
- Distribution publique de la DLL obfusquée via les releases GitHub ; le code source reste privé.

## Installation manuelle

1. Télécharger `AutoRPM.dll` depuis la [dernière release](../../releases/latest).
2. Fermer SimHub.
3. Copier `AutoRPM.dll` dans `C:\Program Files (x86)\SimHub2\SimHub`, à côté de `SimHub.Plugins.dll`.
4. Relancer SimHub et activer **AutoRPM for SimHub** dans la liste des plugins.
5. Chercher `AutoRPM.` dans les propriétés disponibles.

La version 0.2.0 ajoute une page de réglages et une icône via `IWPFSettingsV2`. Dans la liste des plugins, activer **Afficher dans le menu de gauche**, puis ouvrir **AutoRPM**. La page affiche le profil, le régime, le rapport, les seuils et un aperçu des LED. Les boutons permettent de tester sans jeu et de recharger le profil. Les actions SimHub restent disponibles.
Pour retirer le plugin, fermer SimHub puis enlever uniquement `AutoRPM.dll` du dossier d'installation.

## Brancher les LED

### Une barre entière avec ses couleurs (version 0.3.0)

Dans **Devices**, ouvrir le périphérique voulu puis son éditeur de profil LED RGB. Ajouter l'effet **Special → Scripted content** (les libellés peuvent être traduits). Régler le nombre de LED sur **10** pour une barre de 10 LED, positionner l'effet sur la zone RPM et coller ceci dans le champ **JavaScript** de l'effet :

```javascript
var colors = [];
for (var i = 1; i <= 10; i++) {
  var led = ('0' + i).slice(-2);
  colors.push($prop('AutoRPM.N10_COLOR' + led) || '#FF000000');
}
return colors;
```

La page AutoRPM génère ce code selon le nombre choisi et propose un bouton de copie. Le script remplit la barre avec les couleurs calculées par AutoRPM, y compris extinction et redline. Vérifier que les anciens effets RPM du profil ne recouvrent pas cette même zone. Le périphérique et son profil doivent être actifs pour allumer le matériel.

### Propriétés individuelles

| Propriété (exemple 10 LED) | Signification | Syntaxe JavaScript |
|---|---|---|
| `AutoRPM.N10_LED01` | 0 éteinte, 1 seuil atteint ; ne représente pas le clignotement | `return $prop('AutoRPM.N10_LED01');` |
| `AutoRPM.N10_RPM01` | Seuil de la première LED en RPM | `return $prop('AutoRPM.N10_RPM01');` |
| `AutoRPM.N10_COLOR01` | Couleur ARGB affichée, noir si éteinte ; redline et clignotement inclus | `return $prop('AutoRPM.N10_COLOR01');` |

**Attention au mode de formule :** `[AutoRPM.N10_LED01]` est du **NCalc**, pas du JavaScript. Le journal utilisateur montrait « AutoRPM is not defined » parce que cette forme avait été collée dans un champ JS. En JavaScript, utiliser `$prop` avec `return`, comme ci-dessus. Les états ne se collent pas dans un champ couleur : utiliser `COLOR01` ou le script complet.

Chaque propriété `LED` vaut 0 ou 1. Pour un volant à 9 LED, utiliser dans un effet acceptant une condition NCalc :

| LED physique | Propriété d'état | Seuil RPM |
|---|---|---|
| 1 | `[AutoRPM.N9_LED01]` | `[AutoRPM.N9_RPM01]` |
| 2 | `[AutoRPM.N9_LED02]` | `[AutoRPM.N9_RPM02]` |
| … | … | … |
| 9 | `[AutoRPM.N9_LED09]` | `[AutoRPM.N9_RPM09]` |

Pour 12 LED, remplacer `N9` par `N12` et aller jusqu'à `LED12`.
Même logique pour `N8`, `N10` et `N11`. Plusieurs tailles sont disponibles simultanément.

Si le champ attend une couleur NCalc, exemple à **vérifier dans l'aperçu de l'éditeur du périphérique** :

```text
if([AutoRPM.N9_LED01] = 1, '#00FF00', '#000000')
```

Cette ancienne formule impose du vert. Pour les couleurs du profil, utiliser `[AutoRPM.N9_COLOR01]` dans un champ **couleur NCalc**, ou `return $prop('AutoRPM.N9_COLOR01');` dans un champ **couleur JavaScript**. Pour toute la barre, utiliser le script précédent.

Les couleurs de `ledColor[1..N]` sont adaptées au nombre de LED par sélection de la position source la plus proche, sans mélange de teintes. La redline utilise `ledColor[0]`. `redlineBlinkInterval` est interprété comme la durée en millisecondes de chaque phase allumée/éteinte ; zéro signifie une couleur fixe. En l'absence de palette, le repli est blanc (redline rouge), signalé dans la page et par `AutoRPM.HasSourceColors=0`. Le résultat suit les données Lovely disponibles ; avec un nombre différent de LED, il s'agit d'une adaptation, pas d'une reproduction pixel pour pixel du jeu.

## Test sans jeu

Dans les contrôles/actions de SimHub, associer un bouton ou une touche aux actions suivantes :

| Action complète | Effet |
|---|---|
| `AutoRPM.ToggleTestMode` | Active/désactive le profil fictif |
| `AutoRPM.TestRPMUp` | +100 RPM, arrête le balayage |
| `AutoRPM.TestRPMDown` | −100 RPM, arrête le balayage |
| `AutoRPM.TestRPMReset` | 5500 RPM, arrête le balayage |
| `AutoRPM.TestNextGear` | Rapport fictif suivant, de 1 à 6 |
| `AutoRPM.TestSweep` | Active le test et lance un cycle de 5 secondes |
| `AutoRPM.ReloadProfile` | Demande un nouveau chargement live au prochain échantillon |

Le balayage utilise une horloge indépendante des callbacks de télémétrie. Il va de 5000 à presque 8200 RPM, puis recommence, pour montrer aussi la redline.
Le profil Demo GT3 contient six seuils : 5600, 6000, 6400, 6800, 7200, 7500 ; redline à 7600.
Ses couleurs sont fictives : vert, jaune, rouge, puis bleu clignotant à la redline. Pour suivre LMU, cliquer **Revenir au mode live** et entrer en piste : choisir LMU dans l'accueil SimHub ne fournit pas à lui seul de télémétrie.

Vérification manuelle attendue :
1. Activer le test : `TestMode=1`, `CarFound=1`, `CurrentRPM=5500` ; toutes les LED éteintes.
2. Lancer le balayage : observer les propriétés `N9_LED01..09` et `N12_LED01..12`, puis les LED physiques.
3. Désactiver le test : `TestMode=0`. Sans jeu, `CurrentRPM=0`, `CarFound=0` et les LED sont éteintes.
4. En LMU, observer les identifiants réellement transmis et tester plusieurs rapports/voitures.

Les noms ont été vérifiés dans le SDK local : SimHub préfixe les propriétés/actions par le nom de classe `AutoRPM`. Leur affichage dans l'interface reste à confirmer.

## Diagnostic et fonctionnement

Propriétés utiles : `AutoRPM.Status`, `TestMode`, `CarFound`, `CarName`, `CarClass`, `CarId`, `CurrentGame`, `CurrentGear`, `CurrentRPM`, `SourceLedCount`, `RedlineRPM` (toutes avec le préfixe `AutoRPM.`).

Le client utilise `DataCorePlugin.CurrentGame` et directement `GameData.NewData.CarId` dans la télémétrie (repli sur la propriété complète `DataCorePlugin.GameData.CarId`). Le raccourci `DataCorePlugin.CarId` renvoyait une valeur vide sur l'installation testée et bloquait la version 0.3.0. Le client cherche le chemin exact dans `data/manifest.json`, puis utilise le chemin normalisé si le manifeste est indisponible. La page affiche désormais l'identifiant reçu et le rapport brut (`N`, `R`, `1`…).
Les erreurs sont exposées dans `Status`, avec le jeu et l'identifiant voiture. Une erreur est réessayée après cinq minutes ; `ReloadProfile` permet de relancer plus tôt. Cette action réutilise le cache encore valable.

Cache : `%LOCALAPPDATA%\AutoRPM\cache`.
Un fichier valide de moins de sept jours est réutilisé. Si son rafraîchissement échoue, le dernier fichier valide reste utilisable. Le profil actif reste en mémoire ; il est rechargé au changement de voiture, à la demande ou au redémarrage.
Le réseau et les accès disque sont effectués sur un worker, hors du callback de télémétrie.

L'élément 0 d'une courbe est la redline. Les éléments suivants correspondent aux positions LED ; interpolation linéaire et arrondi au plus proche, avec conservation des deux extrémités. Les profils symétriques sont conservés.
Depuis 0.3.3, N et R utilisent leurs propres courbes si le profil les fournit. Un rapport absent ou inconnu reste éteint ; aucun seuil n'est inventé.
Le mode test est temporaire et n'est pas enregistré entre les sessions.

## Recompiler et vérifier

Depuis le dossier `AutoRPM` :

```powershell
dotnet build .\AutoRPM.csproj -c Release
dotnet build .\tests\AutoRPM.Tests.csproj -c Release
& .\tests\bin\Release\net48\AutoRPM.Tests.exe
& .\tests\bin\Release\net48\AutoRPM.Tests.exe 'C:\Program Files (x86)\SimHub2\SimHub' --online
```

Pour une autre installation, fournir `-p:SimHubDir='CHEMIN'` aux deux compilations et son chemin au programme de test.
Le SDK .NET et le targeting pack .NET Framework 4.8 sont nécessaires ; ils sont présents sur ce PC.
Aucune copie dans SimHub n'est effectuée par la compilation. Aucune DLL tierce n'est embarquée dans la sortie.

Fichiers : `Plugin.cs` pour SimHub et les actions, `Profile.cs` pour les courbes, `CarDataClient.cs` pour le manifeste/cache/réseau, `tests/Program.cs` pour les vérifications.

## Attribution des données

## Association LMP2 LMU (0.3.4)

À la demande de Jérôme, les identifiants LMU commençant par `LMP2_` (après normalisation) utilisent le profil Lovely `Oreca 07`, sauf ceux contenant le segment `ELMS`, qui conservent leur recherche spécifique. L'identifiant voiture reçu reste affiché séparément du nom de profil choisi. Cela couvre `LMP2_DKR Engineering 2026_3` malgré l'absence de cette livrée dans le manifeste.

Profil Oreca 07 vérifié : 10 LED source, premier seuil 6565 RPM, redline 7735 RPM, courbes N/R incluses. 69 vérifications réussies, dont le chargement réel de l'Oreca depuis l'identifiant DKR et la séparation ELMS/GT3/autres jeux. Version 0.3.4 installée ; copie vérifiée par SHA-256 et ancienne DLL sauvegardée.

## Attribution des données Lovely

Lovely Car Data : © Lovely Sim Racing, collaboration ATSR et Gomez Sim Industries.
Le dépôt indique la licence [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/).
Les profils complets sont récupérés au fonctionnement et ne sont pas inclus dans la livraison.
Les valeurs Ferrari de référence des tests proviennent du profil `data/lmu/gt3-296gt3-custom-team-2024.json`.
Ce prototype n'est pas un produit officiel Lovely.


## Correctif 0.3.2

Propriété de repli confirmée dans l’interface : DataCorePlugin.GameData.CarId. Les seuils zéro sont des positions désactivées et ne font plus rejeter le profil ; l’interpolation ne mélange pas ces zéros aux seuils voisins. 58 vérifications réussies, dont le téléchargement et la lecture du profil exact GT3_Team WRT 2026_32 (12 positions source, positions 3 et 10 désactivées). Version 0.3.2 installée, copie vérifiée par SHA-256. Validation en session LMU à poursuivre.
