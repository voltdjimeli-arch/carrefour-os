# Boîte à outils mathématique, non-préemption et chaînes de Markov

Ce document complète [modele-global.md](modele-global.md). Il répond à trois questions de l’utilisateur : (1) quels outils qu’il maîtrise déjà servent réellement ; (2) comment traiter l’interblocage quand **aucune préemption** n’est possible ; (3) quel rôle donner aux chaînes de Markov.

Vocabulaire : en systèmes d’exploitation, **préemption** = retirer de force une ressource ou un processus en cours d’exécution. L’utilisateur a pu écrire « appréhension » par dictée vocale ; dans le rapport, employer « préemption ».

## 1. Chaque outil à sa place

Principe : un outil n’entre dans le modèle que s’il démontre ou calcule quelque chose. Ici, « élégant » signifie cohérent, minimal et démontrable.

| Outil | Usage réel dans le projet | Statut |
|---|---|---|
| Automates (temporisés) | Processus, arbitre, feux ; produit synchrone ; vérification des invariants par exploration des états atteignables | Noyau |
| Graphes | Graphe de conflits ; coloration = phases de feux ; graphe d’attente (cycle = interblocage) | Noyau |
| Algèbre linéaire / tenseurs | Incidence \(B\), contrainte de capacité \(B^\top a\le K\), tenseur de réservation espace-temps | Noyau |
| Géométrie | Trajectoires \(\gamma_m\), zones de conflit, empreinte et marge de sécurité | Noyau (construction) |
| Mesure / intégration | Temps d’occupation, loi de Little (Beppo Levi), arrivées comme mesure positive | Utile (métriques) |
| Topologie (ouverts/fermés) | Distinguer conflit strict et contact ; zones semi-ouvertes \([a,b)\) | Utile (rigueur des définitions) |
| Banach (contractions) | Loi stationnaire de Markov, itération de la valeur (MDP), série de Neumann | Utile si Markov/MDP est retenu |
| Barycentre | Position de référence d’un convoi, instant de sortie de sa queue | Secondaire |
| Markov / MDP | Arrivées aléatoires, performances, choix parmi des actions déjà sûres | Optimisation, jamais la sûreté |

À ne pas faire : ajouter Banach, la topologie générale ou les mesures « pour faire sérieux ». Chacun doit répondre à une question précise du tableau.

## 2. Pas de préemption : que reste-t-il ?

Conditions de Coffman : un interblocage exige **simultanément** (i) exclusion mutuelle, (ii) détention et attente, (iii) absence de préemption, (iv) attente circulaire.

- (i) est physique : une zone \(z\) contient au plus \(K_z\) véhicules.
- (iii) est posée par l’énoncé : on ne retire pas un véhicule engagé pour le déposer ailleurs.
- Il faut donc briser (ii) ou (iv).
- « Détecter l’interblocage puis retirer un véhicule » suppose une préemption ou un retour arrière : **exclu**. Seules la prévention et l’évitement restent.

Protocoles possibles :

1. **Prévention par réservation tout-ou-rien** (brise (ii)) : le point de décision est la ligne d’arrêt, où le processus ne détient encore aucune zone. Il réserve d’un coup tout son chemin et l’espace de sortie, ou il reste en attente sans rien détenir. En bases de données, on parle de verrouillage conservatif en deux phases.
2. **Prévention par ordre total des zones** (brise (iv)) : alternative, moins naturelle sur un anneau.
3. **Évitement par état sûr** (algorithme du banquier) : n’admettre que si, après admission, il existe encore un ordre de sortie complet pour tous les processus engagés.

**Proposition (absence d’interblocage).** Hypothèses : H1 toute traversée dure au plus \(d_{\max}\) ; H2 réservation tout-ou-rien incluant l’espace de sortie ; H3 aucune panne externe ; H4 toute requête est réalisable seule dans un carrefour vide. Alors aucune configuration n’est bloquée.

Idée de preuve : dans le graphe d’attente (arc \(i\to j\) si \(i\) attend une ressource détenue par \(j\)), un processus en attente ne détient rien, donc personne ne l’attend ; un processus admis n’attend rien. Le graphe est acyclique. Les processus admis finissent (H1) et libèrent ; par H4, une requête en attente finit par devenir admissible.

## 3. Synchronisation et refus de priorité avec graphes et automates

- **Automate d’un processus** : `APPROCHE → ATTENTE → RÉSERVÉ → EN_TRAVERSÉE → DÉGAGÉ`, avec une horloge \(x\) et l’invariant \(x\le d_m\) en traversée (garantit la durée finie).
- **Automate arbitre** : il traite `demande_m` et émet `accord_m`. Admission de \(m\) si (a) \(B^\top(a+e_m)\le K\), (b) aucune requête plus ancienne en attente n’est en conflit avec \(m\), (c) la signalisation l’autorise.
- **Système global** : produit synchrone des automates sur les événements communs (`accord_m`, `sortie_m`).
- **Sûreté** : l’ensemble des états atteignables ne rencontre pas \(\text{Mauvais}=\{o : \exists z,\ o_z>K_z\}\). Sur une instance bornée, une exploration en largeur codée en C vérifie la propriété par machine.
- **Refus de priorité** = automate « mutant » : on ajoute à un processus la transition `ATTENTE → EN_TRAVERSÉE` sans accord. Le produit atteint alors un état Mauvais (collision) ; le produit conforme non. C’est le scénario d’injection de faute de la simulation.
- **Canne du non-voyant** = événement `demande` ; la priorité du piéton engagé est une classe dans l’admission, et un processus engagé n’est pas préemptible. Le regard à gauche/droite n’est pas un verrou : c’est un test de créneau (voir §4 et `continuite.md`).
- **Phases de feux par coloration** : une coloration propre du graphe de conflits partitionne les mouvements en groupes compatibles ; le nombre minimal de phases est le nombre chromatique. Le graphe est exact si tous les \(K_z=1\) ; sinon la contrainte exacte est \(B^\top a\le K\).

## 4. Le temps : tubes espace-temps et réservation

L’algèbre linéaire donne les conflits : la matrice de Gram \(G=BB^\top\) compte les zones partagées, et \(G_{mn}\ge1\) pour \(m\ne n\) signale un conflit (zones à capacité 1).

Pour le temps, chaque processus est un **tube** \(T=\{(x,t): x\in F(t)\}\) où \(F(t)\) est son empreinte (compacte, élargie de la marge \(\rho\)). Deux processus sont en conflit si leurs tubes se rencontrent. Un convoi est un tube rigide.

Version discrète : tenseur de réservation \(R\in\{0,1\}^{M\times Z\times T}\) et dates de départ \(s_m\) ;

\[
O_{z\tau}=\sum_m a_m\,R_{m,z,\tau-s_m}\le K_z\quad\forall (z,\tau).
\]

Points de rigueur :
- **Topologie** : l’ensemble des collisions \(\{\text{dist}<\rho\}\) est ouvert, l’ensemble sûr \(\{\text{dist}\ge\rho\}\) est fermé. Les zones sont semi-ouvertes \([a,b)\) : une sortie en \(t\) et une entrée en \(t\) ne se chevauchent pas.
- **Mesure** : temps d’occupation de la zone \(z\) par \(m\) : \(\lambda(\{t:\gamma_m(t)\in z\})\). Sûreté : l’occupation simultanée de deux mouvements en conflit est de mesure nulle (nulle car les zones sont semi-ouvertes).
- **Loi de Little** : avec \(N(t)=\sum_i \mathbf 1_{[a_i,d_i)}(t)\), Beppo Levi (convergence monotone) permet d’échanger somme et intégrale : \(\int_0^\infty N(t)\,dt=\sum_i (d_i-a_i)\). L’aire sous la courbe de file égale la somme des attentes individuelles.
- **Test de créneau du piéton** (sans signal) : un véhicule à distance \(d\), vitesse \(v\), arrive en \(d/v\) ; traversée de largeur \(w\) à vitesse \(v_p\) ; on traverse si \(d/v>w/v_p+\text{marge}\). Entre observation et action la situation peut changer (TOCTOU) : d’où la marge.

## 5. Le paradoxe du convoi

Le paradoxe naît quand on voit le feu comme la **source** du droit de passage. Dans le modèle, le feu est une **sortie** du contrôleur ; le droit de passage est la **réservation**. Un convoi est une seule tâche, de tube \(T_g\) et de durée \(d_g\). Il n’est jamais coupé : sa réservation n’est libérée qu’à la sortie de sa queue (invariant).

Si le convoi est modélisé comme un segment rigide de longueur \(L_g\) centré sur son barycentre, la queue quitte une zone d’abscisse de sortie \(s_{out}\) quand l’abscisse du barycentre dépasse \(s_{out}+L_g/2\).

Règle d’admission : soit \([t_0,t_1]\) la fenêtre de vert du mouvement, \(c\) le dégagement, \(t_{adm}\) l’instant d’admission. Les mouvements conflictuels s’ouvrent à

\[
t_{ouv}=\max\bigl(t_1+c,\ t_{adm}+d_g\bigr).
\]

Prolongation \(\delta=\max(0,t_{adm}+d_g-t_1-c)\). Si \(\delta\le\delta_{\max}\), on admet avec prolongation ; sinon le convoi attend **en bloc** à la ligne d’arrêt la fenêtre suivante. Le retard imposé aux autres est borné par \(\delta_{\max}\). Pour éviter la famine des grosses requêtes, l’admission respecte l’ordre d’arrivée entre requêtes en conflit.

## 6. Véhicules prioritaires

- Classes de priorité avec poids \(w\) : l’admission maximise \(w\cdot a\) sous \(B^\top a\le K\) (empaquetage pondéré). Les urgences sont la classe la plus haute, **sous réserve des invariants de sûreté**.
- On préempte le **plan de signalisation**, jamais un véhicule engagé (séquence dans `modele-global.md`).
- **Borne de latence (à démontrer)** : une urgence demandée en \(t_r\) voit son chemin libre au plus tard en \(t_r+d_{\max}+c+\sum(\text{urgences conflictuelles antérieures})\), si après \(t_r\) aucune tâche conflictuelle non prioritaire n’est admise (H1 et H2 en vigueur).
- **Inversion de priorité** : une ambulance bloquée derrière des véhicules ordinaires dans la même voie FIFO attend en réalité ces véhicules. Remède : **héritage de priorité**, les véhicules devant elle héritent temporairement de sa classe.
- Une priorité stricte continue affame les ordinaires : paramétrer un plafond (au plus \(n_E\) fenêtres d’urgence consécutives) ou une garde.

## 7. Chaînes de Markov : ce qu’elles font et ne font pas

**Formulation.** État \(X\) = matrices d’occupation et de files (par exemple occupation \(\{0,1\}^{Z}\) et longueurs de files par branche). Matrice stochastique \(P\) (lignes de somme 1) ; \(\pi_{n+1}=\pi_nP\). Une chaîne **prédit des lois de probabilité**, pas l’état suivant avec certitude.

**Ce qu’elles apportent.**
- *Loi stationnaire* : si \(P\) est irréductible et apériodique, \(\pi\mapsto\pi P\) est une contraction pour la norme \(\ell^1\) sur le simplexe (fermé, donc complet) ; le théorème de Banach donne une loi stationnaire unique et une convergence géométrique.
- *Absorption* : avec \(Q\) le bloc transitoire, \(N=(I-Q)^{-1}=\sum_k Q^k\) (série de Neumann, valable si \(\rho(Q)<1\)) ; le temps moyen avant absorption est \(N\mathbf 1\).
- *Décision (MDP)* : l’opérateur de Bellman est une \(\gamma\)-contraction pour la norme sup ; l’itération de la valeur converge (Banach).

**Ce qu’elles ne font pas.**
- Une probabilité de collision ou d’interblocage faible n’est pas zéro. La sûreté se démontre sur le **support** de \(P\) (c’est un automate) : \(P_{xy}=0\) pour tout \(y\) mauvais. Un interblocage est un état absorbant non final ; il doit être **inaccessible** dans le graphe du support.
- Elles ne rendent pas la préemption possible. Elles servent à **anticiper** : agir avant la zone dangereuse, puisqu’après il n’y a pas de retour. C’est de l’évitement probabiliste (par exemple fermer l’entrée plus tôt si la saturation de l’anneau est prédite).

**Architecture recommandée.** Filtre de sûreté déterministe \(A_{sûr}(x)\) (la réservation est-elle réalisable ?), puis optimiseur de type MDP qui choisit parmi \(A_{sûr}(x)\).

**Taille.** Anneau de 8 segments (\(2^8\) occupations) et 4 branches avec file \(\le 3\) (\(4^4\)) : \(2^8\cdot4^4=65\,536\) états. Calculable avec \(P\) creuse. Au-delà, agréger les états.

**Expérience conseillée.** Politique naïve (réserver zone par zone) : un état d’interblocage est accessible, donc la probabilité d’absorption dans l’interblocage tend vers 1. Politique à réservation atomique : zéro. Mesurer l’écart sur la même instance est un bon résultat de rapport.

## 8. Problème des philosophes : solution globale du carrefour

| Mécanisme | Rôle | Condition de Coffman brisée |
|---|---|---|
| M1 Réservation atomique (arc + sortie) | Aucune zone détenue en attente | Détention et attente |
| M2 FIFO entre requêtes conflictuelles | Pas de famine des grosses requêtes (convoi) | (vivacité) |
| M3 Capacité \(k-1\) sur l’anneau | Variante décentralisée : toujours une case libre | Attente circulaire |
| M4 Ordre total des zones | Alternative sans arbitre central | Attente circulaire |

Implémentation POSIX : un mutex central protège la table de réservation, une variable de condition bloque chaque demandeur jusqu’à son admission, l’ordonnanceur choisit un ensemble compatible et accorde atomiquement. Ne pas faire prendre à chaque thread un mutex de zone puis attendre le suivant.

## 9. Décisions et questions ouvertes

Décisions prises avec l’utilisateur :
- On roule à droite (volant à gauche) ; le véhicule le plus proche d’un piéton vient de sa gauche.
- On construit d’abord un noyau commun (zones, mouvements, conflits) à \(N\) branches paramétrées, puis on ajoute le rond-point, les feux et les panneaux.
- Aucune préemption de processus engagés.
- Deux versions du modèle : processus coopératifs (on prouve la sûreté) et processus fautif (injection de faute).

À valider :
- Modèle sans signal (test de créneau) ou avec signal (phases), ou les deux séparément.
- Rond-point : priorité à l’anneau ou non ? Nombre de segments de l’anneau.
- Taille et capacité des zones, durée maximale \(d_{\max}\), hypothèse H4 pour les convois.
