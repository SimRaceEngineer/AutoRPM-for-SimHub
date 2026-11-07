# AutoRPM for SimHub — modifications

## 0.6.0

- **Nouvelle interface** aux couleurs de simrace.app : fond sombre « instrument », titres JetBrains Mono, accent rouge,
  vert citron / bleu pour les états.
- **Carte « En direct » toujours visible** : voiture, pastille d'état colorée (profil chargé / générique / test / en attente),
  régime et rapport en grand, barre de LED en temps réel.
- **Trois onglets** : *Fonctionnement* (les 3 étapes du plugin — identifier la voiture → charger le profil → allumer les LED —
  avec la valeur en cours de chacune), *Brancher les LED* (3 étapes : nombre de LED, script à coller, vérification),
  *Test* (mode test sans jeu).
- Titres d'étapes surlignés : pastille numérotée + bandeau teinté avec liseré de couleur. Textes ajoutés en 9 langues.

## 0.5.0

- Barres de **8 à 15 LED** (au lieu de 8 à 12) : propriétés `N13_*`, `N14_*`, `N15_*` (`LED`, `RPM`, `COLOR`) publiées en plus,
  sélecteur « Aperçu des LED » jusqu'à 15 et script de barre généré pour le nombre choisi. Bornes centralisées
  (`Profile.MinLeds` / `Profile.MaxLeds`). Utile pour les 14 LED RPM du P1Sim Arnage.
- Interface et messages d'état en **9 langues** (en, fr, de, es, it, pt, nl, ja, ko) : suit la langue de Windows, sélecteur
  en haut de page, choix mémorisé dans `%LOCALAPPDATA%\AutoRPM\language.txt`.
- Vérifications de profils étendues à 15 LED : 93 réussies (81 auparavant). Le test « PluginSmoke » (WPF) reste à lancer sous Windows.
- Sources versionnées dans le dépôt simrace (`AutoRPM/`), auparavant uniquement en local.

## 0.4.1

- Identification ATSR et recherche des profils appliquées à tous les jeux ; normalisation des noms de dossiers et alias explicites LMU, ACC, AC, AMS2, RaceRoom. Assetto Corsa Rally reste un jeu distinct d'Assetto Corsa.
- En cas de profil indisponible, mode générique AutoRPM pour tout jeu : limite `ATSRHubMain.Telemetry.RedlineData.GenericRedlineValue`, puis limites SimHub (rapport courant, redline, max RPM). Pas de limite inventée lorsque ces valeurs sont absentes.
- Seuils génériques basés sur les valeurs par défaut ATSR 88/93/98 %, interpolés sur 8 à 12 LED, couleurs vert/jaune/rouge et redline rouge clignotante à 300 ms. Ce mode n'est ni un profil constructeur ni la reproduction de toutes les personnalisations d'un périphérique ATSR.
- La limite suit la télémétrie et les changements de rapport ; le profil spécifique reprend la priorité dès qu'il est chargé. Reprise automatique de la recherche après cinq minutes ou via Recharger le profil live.
- Mode générique signalé dans l'interface et `IsGenericProfile`. L'erreur de recherche reste consultable dans `ProfileError`.
- 113 vérifications réussies, dont trame simulée avec profil Rally absent, LED génériques, évolution de la limite, retour à un profil spécifique et arrêt du jeu. Compilation sans avertissement et rendu WPF vérifié. Installation en attente de fermeture de SimHub ; validation en jeu à effectuer.

## 0.4.0

- Identification par le modèle `ATSRHubMain.Telemetry.CurrentCarId` en priorité, évitant les recherches par nom d'équipe lorsque le modèle est disponible. Repli sur l'identification SimHub en l'absence d'ATSR.
- Lecture prioritaire de la copie locale Lovely Car Data d'ATSR, sans écriture dans ce dossier. Repli sur le cache AutoRPM et le téléchargement Lovely pour les fichiers absents ou invalides.
- Provenance des données et modèle ATSR affichés dans la page et exposés en propriétés SimHub.
- 89 vérifications réussies, incluant la Corvette TF Sport via ATSR, Ligier, Ginetta, Duqueine, et le profil Corvette de l'installation locale ATSR. Compilation sans avertissement. Validation en jeu encore à effectuer.
- DLL 0.4.0 installée avec SimHub fermé, version et SHA-256 vérifiés ; DLL précédente sauvegardée. Construction et rendu WPF vérifiés.

## 0.3.5

- LMU `LMP3_R-ace GP 2026_85` utilise `Duqueine D09 P3`, association confirmée par la capture utilisateur.
- Les identifiants LMP3 contenant explicitement le modèle Duqueine D09, Ligier JS P325, Ginetta G61-LT-P3/P325 ou ADESS AD25 utilisent leur propre profil constructeur Lovely.
- Aucun remplacement global de toutes les LMP3 par Duqueine. Les autres livrées conservent leur recherche exacte ; une équipe inconnue n'est pas attribuée à un constructeur par supposition.
- Les LMP2 non-ELMS restent associées à Oreca 07. ELMS, GT3 et autres jeux inchangés.
- Compilation réussie et 77 vérifications, dont téléchargement des quatre profils constructeur LMP3. Essais en jeu de cette version encore à effectuer.
- DLL 0.3.5 installée après fermeture de SimHub ; copie vérifiée par SHA-256 et version précédente sauvegardée.
