# Notes de continuité — projet carrefour et systèmes d’exploitation

## Prompt de reprise

> Nous travaillons sur un projet de systèmes d’exploitation : simuler un carrefour multivoies et multibranches. Le professeur a présenté le sujet comme l’implémentation, en quelque sorte, d’un code de la route. Aide-moi à comprendre et formaliser le problème, puis à construire avec moi un modèle mathématique rigoureux et une solution simulable. Il faut étudier les conflits/priorités entre processus, l’interblocage et les files/embouteillages. Le vocabulaire du rapport/code doit être celui des systèmes d’exploitation (processus, ordonnancement, ressources, verrou, mutex, etc.), tout en expliquant clairement les analogies. Le code sera en C, probablement C/POSIX, avec une interface graphique dans un langage au choix. Ne suppose pas les règles du carrefour avant d’avoir recueilli l’énoncé et les choix du cours. Fais avancer le travail étape par étape et garde une trace des décisions et questions ouvertes.

## Échanges à retenir

### Demande initiale de l’utilisateur

L’utilisateur explique qu’il s’agit d’un projet de systèmes d’exploitation simulant une circulation routière dans un carrefour multivoies. Les thèmes évoqués sont les conflits de priorité entre processus (exemple d’un piéton et d’une voiture), l’interblocage et les embouteillages. Il veut un modèle mathématique rigoureux et élégant, puis une solution et une simulation. Il prévoit du C POSIX ou une autre variante de C et une interface graphique dans un langage libre. Sa difficulté principale est de comprendre le problème et ses analogies mathématiques. Il souhaite un travail au long cours et collaboratif.

### Première réponse de l’assistant

Le carrefour a été cadré comme un système concurrent : acteurs comme processus, voies/zones comme ressources partagées, collisions comme états interdits, interblocage comme attente circulaire, embouteillage comme files et attente, priorité comme politique d’ordonnancement. Il a été suggéré de découper le carrefour en zones de conflit et de représenter chaque trajectoire par l’ensemble des zones requises. Cette représentation reste une piste, à valider selon le schéma réel.

### Précision de l’utilisateur

Le professeur a expliqué que le projet consiste « en quelque sorte » à implémenter un code de la route.

### Réponse de l’assistant

Le code de la route a été interprété comme une politique de décision pour autoriser les processus selon leur position, direction et priorité. Les éléments à fixer sont les états, priorités, règles de sûreté et progression. Les exemples (priorité à droite, panneaux, feux, passage piéton, rond-point) ont été présentés comme options possibles, pas comme faits établis.

### Dernière précision de l’utilisateur

Il s’agit d’un carrefour multivoies et multibranches. L’utilisateur demande de conserver le contexte dans un skill et d’obtenir une copie incluant prompt, réponses, notes et méthode de raisonnement.

### Traversée piétonne décrite par l’utilisateur

L’utilisateur décrit deux situations :

- Sans feu piéton clairement spécifié : regarder les deux côtés, estimer si les véhicules sont assez loin pour laisser le temps de traverser, franchir les voies proches en surveillant les véhicules venant de gauche (conduite à droite), puis réévaluer les voies venant de droite. Il évoque éventuellement une étape au milieu de la chaussée.
- Avec signalisation lumineuse : observer les feux tout en continuant à vérifier la circulation.

Ne pas confondre les feux : un feu piéton rouge signifie généralement attendre, tandis qu’un feu rouge destiné aux véhicules commande leur arrêt. Demander de quel feu et de quelle couleur l’utilisateur parle avant d’en faire une règle du modèle. Le regard du piéton est une observation et une estimation de créneau, pas un verrou. Cette estimation ne garantit pas l’exclusion mutuelle, car la situation peut changer après l’observation. Distinguer le modèle sans signal (acceptation d’un créneau sûr) du modèle avec signal (phases/autorisation); ne pas les fusionner sans décision explicite.

## Limites des informations actuelles

À ce stade, aucun schéma réel ni règle précise du code de la route n’a été communiqué. Le nombre de branches/voies, les sens de circulation, les mouvements permis, les priorités, les feux/panneaux et les exigences de démonstration restent inconnus. Demander l’énoncé ou un dessin avant de figer le modèle.

## Orientation mathématique à privilégier

L’utilisateur maîtrise déjà les automates et souhaite réutiliser ses acquis (automates, géométrie, algèbre, mesure/intégration, topologie) plutôt que d’apprendre les réseaux de Petri sans nécessité. Ne pas imposer Petri : c’est une option graphique, pas un prérequis.

Piste de base à construire avec lui : système de transitions/automate global pour les états et événements, plus graphe de conflits pour les trajectoires incompatibles. Les automates peuvent décrire chaque processus (arrivée, attente, admis, traverse, libère) et un automate ordonnanceur (règle de priorité). En version bornée, leur composition décrit le système concurrent complet. Définir la contrainte de capacité/sûreté des ressources dans les états admissibles.

Employer les autres outils selon leur rôle : géométrie pour bâtir les trajectoires et zones de conflit ; matrices/vecteurs pour encoder l’incidence trajectoire-zone ; mesures et intégrales pour les métriques temporelles (attente moyenne, occupation, débit) ou arrivées aléatoires si cela est requis. Barycentres, topologie générale et espaces de Banach ne sont pas nécessaires au noyau discret; ne les ajouter que si une question précise (distribution spatiale, espace d’états continu, convergence d’un contrôle) le justifie.

Avant de figer le formalisme, comparer sa lisibilité, son pouvoir de démonstration, sa facilité de simulation C/POSIX et l’adéquation aux connaissances de l’utilisateur. Le réseau de Petri reste une alternative si ses places/jetons éclairent mieux les files et ressources, mais la compréhension utilisateur prime sur l’élégance d’un formalisme choisi d’avance.

## Convois, véhicules prioritaires et problème des philosophes

L’utilisateur demande un modèle global intégrant le paradoxe du convoi présidentiel (groupe non dissociable, traversée pouvant continuer pendant plusieurs phases de feu), les véhicules d’urgence (ambulance, police, pompiers), l’impossibilité de retirer physiquement un véhicule déjà engagé, et l’interblocage analogue au problème des philosophes.

Orientation à proposer : automate temporisé/hybride global + graphe de conflits et ressources. Un convoi indivisible est un processus composite non préemptible une fois admis; son temps de traversée se calcule jusqu’à ce que le dernier élément ait dégagé le corridor. L’admission ferme les mouvements incompatibles et réserve les zones conflictuelles ainsi qu’une capacité de sortie suffisante. Le feu peut changer sa phase d’admission, mais ne coupe pas le convoi déjà engagé; le contrôleur suit la séquence de dégagement (jaune/all-red selon la politique simulée) jusqu’à confirmation de sortie.

Un véhicule d’urgence soumet une demande prioritaire. Le contrôleur peut préempter le cycle normal des feux de façon ordonnée : arrêter les nouvelles admissions conflictuelles, assurer le dégagement/temps de transition de sécurité, puis ouvrir le chemin réservé à l’urgence. Il ne peut pas interrompre la traversée d’un processus déjà admis. Les arbitrages quand une urgence arrive pendant un convoi sont à spécifier et à simuler, pas à résoudre en supposant une téléportation ou un arrêt instantané.

Analogie des philosophes : processus = mouvements, fourchettes = zones de conflit. L’interblocage survient si chaque processus détient une zone et attend la suivante. La solution de base est un ordonnanceur central qui accorde atomiquement l’ensemble des zones nécessaires et l’espace de sortie avant l’entrée. Une allocation par ordre total des zones est une alternative. L’allocation atomique supprime l’attente en détenant partiellement des ressources et s’applique aux véhicules ordinaires, convois et urgences.

Modèle mathématique suggéré : état global \(x=(q_i,O_z,\phi,Q,R)\), où \(q_i\) sont les états d’automates des processus, \(O_z\) l’occupation des zones, \(\phi\) les phases de feux, \(Q\) les files et \(R\) les réservations/capacités de sortie. Événements temporisés : arrivée, demande, admission, changement de phase, entrée/sortie de zone et dégagement du dernier véhicule d’un convoi. Invariants de sûreté interdisent l’occupation simultanée de zones incompatibles et toute admission sans espace de sortie.

Une chaîne de Markov peut représenter la stochasticité des arrivées et évaluer les délais; un processus de décision markovien peut comparer des actions de contrôle. Ni l’un ni l’autre ne remplace les invariants d’admission qui garantissent la sûreté et préviennent l’interblocage.

## Façon de raisonner à préserver

Cette section documente une **méthode explicable**, pas un journal de raisonnement interne privé :

1. Extraire les acteurs, états, événements, ressources partagées, règles et critères de réussite depuis le scénario concret.
2. Séparer faits fournis, hypothèses de travail et choix de conception.
3. Construire une représentation minimale des mouvements et conflits; vérifier qu’elle couvre les mouvements réels.
4. Formaliser les états/transitions et invariants de sûreté, puis les propriétés de progression et d’équité.
5. Étudier les cycles d’attente pour l’interblocage et les politiques de priorité pour la famine.
6. Choisir un mécanisme POSIX qui implémente la politique formalisée; ne pas confondre outil de synchronisation et règle d’ordonnancement.
7. Définir des scénarios permettant d’observer les cas pertinents et mesurer attente, débit et files.
8. Revenir aux hypothèses quand une règle du professeur ou un cas de simulation contredit le modèle.

## Questions ouvertes à poser au bon moment

- Le professeur a-t-il fourni un énoncé complet, un dessin ou un nombre de branches et de voies ?
- Quelles trajectoires chaque branche permet-elle et quels mouvements se croisent ?
- Quel code de la route doit être modélisé (priorité, cédez-le-passage, stop, feux, giratoire, passages piétons) ?
- Les processus doivent-ils être des threads ou des processus POSIX séparés ?
- Quelles propriétés et quels livrables doivent apparaître dans le rapport et la démonstration ?
- Quelle interface ou plateforme graphique est attendue, s’il y en a une ?
