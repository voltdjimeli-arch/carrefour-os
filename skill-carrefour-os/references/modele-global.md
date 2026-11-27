# Modèle global : groupes, urgences et interblocage

Ce document propose une architecture mathématique paramétrable. Elle ne fixe pas encore le nombre de branches ni un code de la route national. Les priorités et durées sont des paramètres à définir avec l’utilisateur.

## 1. Vue d’ensemble

Partir d’un **automate temporisé global**, formé à partir des automates des processus et d’un automate contrôleur des feux. Le compléter avec :

- un graphe des conflits entre mouvements;
- un tableau d’incidence mouvement-zone;
- un protocole d’attribution atomique des zones et de l’espace de sortie;
- une politique de priorité, avec prise en charge des groupes et urgences.

Les chaînes de Markov sont une extension pour modéliser les arrivées aléatoires et évaluer les performances. Elles ne remplacent pas les contraintes de sûreté ni le protocole qui empêche l’interblocage.

## 2. Graphe, zones et mouvements

Soit \(M\) l’ensemble des mouvements, \(Z\) l’ensemble des zones de conflit, et \(B\in\{0,1\}^{|M|\times |Z|}\) la matrice d’incidence :

\[
B_{mz}=1 \quad\text{si le mouvement }m\text{ utilise la zone }z.
\]

On peut définir un graphe simple de conflits par :

\[
C_{mn}=1 \quad\text{si }m\ne n\text{ et }\exists z: B_{mz}B_{nz}=1.
\]

Les variantes de capacité (p. ex. plusieurs processus parallèles suivant le même couloir) se modélisent par une capacité \(K_z\) pour chaque zone et une contrainte d’occupation; ne pas supposer systématiquement que chaque voie n’admet qu’un seul processus. Inclure les zones de sortie ou une réserve d’espace aval pour empêcher un processus de s’engager s’il ne peut pas dégager le carrefour.

## 3. État global et automates

Un état global peut s’écrire :

\[
x=(q_1,\ldots,q_N, O, \phi, Q, R),
\]

où :

- \(q_i\) est l’état du processus \(i\) : approche, attente, admis, en traversée, dégagé;
- \(O\) décrit les occupants et capacités restantes des zones;
- \(\phi\) décrit la phase de chaque signal et les mouvements autorisés;
- \(Q\) contient les files et l’ordre d’arrivée;
- \(R\) contient les réservations accordées et les requêtes en cours.

Un processus ordinaire suit les transitions `APPROCHE → ATTENTE → RÉSERVÉ → EN_TRAVERSÉE → DÉGAGÉ`. Une horloge d’exécution mesure le temps jusqu’à la sortie complète des zones réservées. Les événements comprennent arrivée, demande, admission, changement de phase, entrée/sortie de zone, et libération.

L’automate contrôleur a des phases de service, de fermeture aux nouvelles entrées, de dégagement et de reprise. Les noms précis (vert/jaune/tout-rouge, ou autres) sont des paramètres de la politique simulée.

## 4. Processus groupé non dissociable : convoi

Un convoi déclaré indivisible est **une requête composite**, pas une suite de requêtes indépendantes. Il porte notamment :

\[
g=(m_g, L_g, v_g, r_g, c_g),
\]

avec mouvement \(m_g\), longueur ou taille \(L_g\), vitesse simulée \(v_g\), instant de demande \(r_g\) et classe \(c_g\). Une estimation simple de sa durée est \(d_g\simeq L_g/v_g+d_{\text{dégagement}}\); pour une simulation discrète on peut paramétrer directement \(d_g\).

Protocole :

1. L’ordonnanceur vérifie que l’ensemble des zones du mouvement et l’espace aval peuvent être réservés pour le passage du convoi.
2. Il ferme les nouvelles admissions de mouvements incompatibles avant l’entrée du premier élément.
3. Il admet le convoi comme une unité logique. La réservation reste active jusqu’à ce que le dernier élément ait franchi la zone de sortie.
4. Un changement nominal de phase n’interrompt pas le convoi : les mouvements conflictuels restent bloqués durant le dégagement, puis le contrôleur reprend son cycle.

L’intervalle de blocage des autres mouvements peut être coûteux. Le modèle doit donc suivre la longueur/durée du convoi, limiter l’entrée si la réservation ne peut pas être garantie, mesurer les files induites, et prévoir un scénario de forte charge. Une priorité élevée ne doit pas impliquer une durée infinie ni une admission sans dégagement possible.

## 5. Véhicules prioritaires et préemption des feux

Les ambulances, véhicules de police et de secours sont des processus avec classe prioritaire et demande de passage. L’urgence préempte le **cycle de contrôle des feux**, pas le processus physique d’un véhicule déjà engagé.

Séquence de préemption abstraite :

```text
NORMAL
  → URGENCE_DEMANDÉE
  → FERMER_LES_NOUVELLES_ADMISSIONS_CONFLICTUELLES
  → DÉGAGEMENT_ET_TRANSITION_DE_SÉCURITÉ
  → CHEMIN_URGENCE_AUTORISÉ
  → DÉGAGEMENT_URGENCE
  → REPRISE_ORDONNÉE_DU_CYCLE
```

Chaque transition est temporisée si elle exige un délai. L’ordonnanceur ne passe pas instantanément un feu contradictoire au vert. Une urgence qui arrive pendant un mouvement ou un convoi déjà admis attend la fin du dégagement sûr, sauf mécanisme particulier expressément modélisé. La hiérarchie entre catégories d’urgence, la prise en compte du temps d’attente et la possibilité de demandes simultanées restent des choix de politique.

La priorité stricte peut affamer les processus ordinaires si les urgences sont continues. Mesurer ce risque et définir un compromis (urgence absolue dans certaines conditions, seuils, files dédiées ou politique d’équité) selon les consignes du projet.

## 6. Problème des philosophes et interblocage

Correspondance classique :

| Philosophes | Carrefour |
|---|---|
| Philosophe | Processus qui veut suivre un mouvement |
| Fourchette | Zone de conflit / ressource |
| Prendre une fourchette | Verrouiller une zone |
| Garder une fourchette en attendant l’autre | Entrer dans le carrefour puis attendre une zone suivante |
| Attente circulaire | Cycle de processus se bloquant mutuellement |

La mauvaise politique est : chaque processus verrouille une première zone puis attend la suivante. Cela réunit attente avec ressources détenues et attente circulaire.

Politique de base recommandée : **arbitre central et réservation tout-ou-rien**. Une admission est une transition atomique qui réserve simultanément toutes les zones requises, ainsi qu’une capacité de sortie. Sinon, la requête reste à l’extérieur sans détenir de ressource. Après la traversée, les ressources sont libérées. Cette méthode brise l’attente avec allocation partielle et empêche le scénario des philosophes où chacun garde une fourchette en attendant celle du voisin.

Alternative : imposer un ordre total commun \(z_1<z_2<\cdots<z_k\) à toutes les demandes de verrouillage; un cycle d’attente serait alors impossible. Mais un contrôleur central qui attribue atomiquement des réservations est souvent plus simple à expliquer pour un carrefour et un convoi.

Sous les hypothèses que chaque admission réserve réellement son chemin de sortie, qu’aucune panne externe ne bloque un processus engagé, et que la durée de traversée est finie, chaque admis peut finir puis libérer ses ressources. La preuve d’absence d’interblocage devra énoncer explicitement ces hypothèses.

## 7. Invariants de sûreté

Pour chaque zone exclusive \(z\), au plus un utilisateur actif :

\[
\sum_i \mathbf{1}[i\text{ occupe }z]\le K_z.
\]

Pour chaque paire de mouvements en conflit, ils ne doivent pas occuper en même temps leur zone de conflit. En plus :

- aucune admission sans autorisation de signalisation ou réservation correspondante;
- aucun mouvement engagé sans capacité de sortie réservée;
- un convoi indivisible ne libère la réservation de corridor qu’après le passage de sa queue;
- un processus admis n’est pas retiré physiquement de la ressource par préemption.

Ces invariants sont des propriétés de sûreté. La priorité est une propriété d’ordonnancement; l’absence de famine et le débit sont des propriétés de vivacité/performance et se vérifient séparément.

## 8. Temps et simulation

Une simulation à événements discrets est adaptée : on avance l’horloge au prochain événement parmi arrivée, fin de dégagement, fin de traversée, fin de phase et échéance d’urgence. Chaque processus porte date de demande et durée estimée. Les files produisent des mesures telles que temps moyen d’attente, attente maximale, débit, longueur maximale de file, et délai de réponse aux urgences.

Les chaînes de Markov conviennent si les arrivées/durées sont aléatoires et si l’on veut calculer des probabilités d’état ou des performances moyennes. Si le contrôleur choisit les phases, c’est un processus de décision markovien. Dans tous les cas, retirer du modèle les transitions interdites par les invariants : les probabilités ne doivent jamais rendre possible une collision.

## 9. Correspondance C/POSIX possible

- threads/processus simulés soumettent une demande décrivant mouvement, type, date et taille de groupe;
- mutex central protège l’état des files, phases, zones et réservations;
- variable de condition bloque un demandeur jusqu’à son admission;
- ordonnanceur sélectionne un ensemble compatible de requêtes et attribue atomiquement leurs réservations;
- les durées sont simulées par une horloge d’événements (ou par temporisations maîtrisées si le projet exige des threads physiques).

Éviter que chaque thread prenne individuellement un mutex de zone puis bloque en attendant un autre mutex. Cela reproduirait directement le problème des philosophes. Si des mutex par zone sont utilisés, l’ordre total doit être cohérent ou leur attribution doit rester atomique sous le verrou du contrôleur.

## 10. Scénarios indispensables à étudier

1. Deux mouvements incompatibles demandent simultanément le passage.
2. Deux mouvements compatibles sont admis ensemble.
3. Un convoi arrive à l’approche d’un changement de feu et son dernier élément met du temps à dégager.
4. Une ambulance arrive pendant le cycle normal puis pendant qu’un convoi est engagé.
5. Plusieurs urgences demandent des mouvements en conflit.
6. Forte charge continue : vérifier famine des processus ordinaires et files induites.
7. Mouvement sans espace de sortie : il doit rester en attente et ne pas bloquer les zones internes.
8. Interblocage des philosophes créé par une stratégie incorrecte, puis éliminé par réservation atomique.
