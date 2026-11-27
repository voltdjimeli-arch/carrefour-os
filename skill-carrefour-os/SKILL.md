---
name: carrefour-os
description: Aider à modéliser et simuler un carrefour multivoies et multibranches comme système concurrent, avec règles de circulation, ordonnancement et synchronisation POSIX.
metadata:
  short-description: Modéliser un carrefour comme système concurrent
---

# Projet : simulation d’un carrefour concurrent

Accompagner l’utilisateur sur la durée dans la compréhension, la modélisation mathématique, la conception de la solution, l’implémentation C/POSIX et la simulation graphique d’un carrefour **multivoies et multibranches** pour un projet de systèmes d’exploitation.

## Contexte établi

- Le sujet porte sur la simulation d’un carrefour routier multivoies et multibranches.
- Le professeur a décrit l’objectif comme étant, en quelque sorte, d’implémenter un code de la route.
- Le vocabulaire attendu est celui des systèmes : processus, ordonnanceur, priorité, ressource, section critique, verrou, mutex, sémaphore, interblocage et famine. Garder le lien avec les situations routières pour expliquer les analogies, puis employer le vocabulaire système dans la formalisation.
- Les problèmes évoqués sont les conflits de passage (y compris un processus piéton et un processus véhicule), l’interblocage et les files/embouteillages au carrefour.
- Le modèle doit aussi pouvoir représenter des groupes non dissociables (ex. convoi présidentiel traversant plusieurs phases de feu) ainsi que des processus prioritaires (ambulance, police, sapeurs-pompiers).
- L’utilisateur maîtrise les automates, matrices, géométrie, mesures/intégration et d’autres outils mathématiques. Partir de ses acquis; ne pas imposer les réseaux de Petri.
- Le livrable envisagé est une simulation en C, notamment C/POSIX (ou autre variante C), avec interface graphique dans un langage au choix.
- L’utilisateur souhaite travailler en collaboration, progressivement, en comprenant et en construisant le modèle plutôt que de recevoir uniquement une solution toute faite.
- Les détails des branches, voies, mouvements autorisés, règles de priorité et exigences du professeur restent à recueillir. Ne pas les inventer comme des faits.
- Circulation à droite (volant à gauche) : indiqué par l’utilisateur ; le véhicule le plus proche d’un piéton qui traverse vient de sa gauche.
- Aucune préemption possible d’un véhicule engagé : on ne le retire pas pour le déposer ailleurs. L’utilisateur peut écrire « appréhension » par dictée vocale pour « préemption » ; employer « préemption » dans les documents.
- L’utilisateur veut réutiliser ses acquis (automates, barycentres, topologie, Banach, mesure et intégration, géométrie, algèbre multilinéaire) et a entendu parler des chaînes de Markov sur des états matriciels. Voir la boîte à outils pour leur place réelle.

## Méthode de travail

1. **Absorber l’énoncé avant de fixer les règles.** Recueillir le schéma du carrefour, les branches et voies, les mouvements possibles (tout droit, tourner, traverser), les règles à appliquer et les contraintes du rendu. Distinguer les règles données par l’enseignant des hypothèses proposées.
2. **Construire une abstraction vérifiable.** Modéliser les acteurs comme processus concurrents, les segments/zones de conflit comme ressources, les mouvements comme demandes d’ensembles de ressources et les règles de circulation comme politique d’admission/ordonnancement. La représentation par zones de conflit est un point de départ utile, pas une contrainte si un autre modèle convient mieux.
3. **Énoncer les propriétés attendues.** Définir séparément sûreté (aucun mouvement incompatible simultané), vivacité (un processus admissible finit par avancer), absence d’interblocage, absence de famine si possible, et qualité de service (attente, débit, longueur des files). Préciser les hypothèses sous lesquelles chaque propriété est démontrée.
4. **Comparer les politiques avant de coder.** Examiner les priorités du code de la route retenu, files FIFO ou par priorité, admission atomique de toutes les ressources, mutex/sémaphores par zone, ou ordonnanceur centralisé. Expliquer les compromis et les risques de famine ou d’attente circulaire.
5. **Représenter explicitement les cas groupés et prioritaires.** Un convoi indivisible est une seule requête composite/non préemptible, dont le passage prend le temps nécessaire à la sortie de son dernier élément. Une demande d’urgence déclenche une séquence sûre de changement de phase; elle ne coupe pas un processus déjà engagé. Lire [modèle global : convois, urgences et interblocage](references/modele-global.md) pour ce volet.
6. **Relier preuve et implémentation.** Traduire chaque état, transition et invariant en mécanismes C/POSIX. Puis définir les scénarios de simulation qui illustrent fonctionnement normal, conflits, forte charge et cas limites. Ne pas prétendre qu’une simulation seule prouve les propriétés générales.
7. **Avancer de façon pédagogique.** Expliquer les analogies et les choix en français clair, expliciter les hypothèses, garder une trace des décisions et des questions ouvertes, et inviter l’utilisateur à corriger le modèle avec les consignes de son cours.
8. **Choisir les outils mathématiques par leur utilité.** Lire la [boîte à outils mathématique, non-préemption et Markov](references/outils-mathematiques-et-preemption.md) : rôle réel de chaque outil, conditions de Coffman, tubes espace-temps, convoi, urgences, chaînes de Markov et MDP, solution globale du problème des philosophes.

## Garde-fous de modélisation

- Ne pas choisir arbitrairement « priorité à droite », rond-point, feux ou panneau stop sans confirmation de l’énoncé. Le professeur a seulement dit que le projet ressemble à l’implémentation d’un code de la route.
- « Piéton contre voiture » est une analogie à formaliser comme conflits entre trajectoires/processus et règles de priorité; éviter d’assimiler le refus de priorité à un mécanisme d’OS sans définir précisément les événements et conséquences.
- Un mutex protège une ressource/section critique; il ne définit pas à lui seul une politique équitable ni l’absence de famine. Des verrous pris dans des ordres différents peuvent créer un interblocage.
- Réserver atomiquement l’ensemble des ressources d’un mouvement ou imposer un ordre total cohérent des ressources peut prévenir certaines formes d’interblocage; choisir seulement après avoir établi le modèle.
- Ne jamais modéliser le découpage d’un convoi déclaré indivisible. Un feu qui change ferme les admissions nouvelles; le contrôleur assure une phase de dégagement jusqu’à la sortie de la queue du convoi.
- Traiter les véhicules d’urgence par une classe/priorité paramétrée et une préemption sûre des phases de signalisation, jamais par interruption physique d’un processus déjà engagé dans une zone de conflit.
- Le problème des philosophes se transpose à des processus qui demandent plusieurs zones : éviter qu’ils prennent une zone puis attendent les suivantes. Favoriser un arbitre central accordant atomiquement l’ensemble des zones du mouvement, avec espace de sortie réservé.
- Présenter « élégant » comme une demande de modèle cohérent, minimal et démontrable, pas comme une promesse de résultat exceptionnel ni comme une formalité mathématique décorative.
- Sans préemption, l’interblocage se traite par prévention ou évitement, jamais par « détection puis retrait d’un véhicule ». Briser la détention-attente (réservation tout-ou-rien) ou l’attente circulaire (ordre total), et énoncer les hypothèses de la preuve.
- Les chaînes de Markov prédisent des lois de probabilité et optimisent des performances; elles ne démontrent ni la sûreté ni l’absence d’interblocage. La sûreté se garantit sur le support de la matrice (un automate) ou par un filtre déterministe. Ne pas présenter Markov comme un moyen de préempter.
- N’employer topologie, Banach, mesure ou barycentre que là où ils démontrent ou calculent quelque chose (liste dans la boîte à outils); ne pas les ajouter par décor.
- Canne du non-voyant : c’est une demande, pas un verrou. Le regard à gauche/droite est un test de créneau (avec marge, à cause du délai entre observation et action). Prévoir un modèle coopératif (preuve de sûreté) et un modèle avec processus fautif (injection de faute pour le refus de priorité).
- Convoi : un seul tube espace-temps, prolongation de dégagement bornée; si elle dépasse la borne, le convoi attend en bloc. Éviter la famine des grosses requêtes par un ordre d’arrivée entre requêtes en conflit.
- Urgences : préemption du plan de signalisation, héritage de priorité aux véhicules qui précèdent l’urgence dans la même voie, borne de latence à démontrer, plafond contre la famine des ordinaires.

## Continuité

Consulter [notes de continuité](references/continuite.md) pour le prompt récapitulatif, les échanges établis et les questions encore ouvertes, et la section « Décisions et questions ouvertes » de la [boîte à outils](references/outils-mathematiques-et-preemption.md). Mettre ces notes à jour quand l’utilisateur donne de nouvelles règles, hypothèses ou décisions structurantes.
