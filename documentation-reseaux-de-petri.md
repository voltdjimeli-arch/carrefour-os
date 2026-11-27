# Réseaux de Petri — base pour le projet de carrefour

## 1. Idée centrale

Un réseau de Petri est un modèle graphique et mathématique pour les systèmes où plusieurs activités peuvent attendre, s’exécuter en parallèle et entrer en conflit pour des ressources communes.

Il convient donc à un carrefour : les processus demandent un passage, l’ordonnanceur les admet selon les règles, ils occupent des zones, puis les libèrent.

## 2. Les quatre objets de base

| Objet | Dessin usuel | Sens général | Dans notre carrefour |
|---|---|---|---|
| Place | cercle | état, ressource, file | « véhicule en attente », « zone libre », « passage occupé » |
| Transition | barre/rectangle | événement atomique | « admission accordée », « entrer », « sortir » |
| Arc | flèche | relation entre état et événement | condition ou conséquence d’un événement |
| Jeton | point dans une place | instance présente dans cet état | un processus particulier ou un jeton représentant une ressource libre |

Une place peut contenir plusieurs jetons. La distribution des jetons dans les places est appelée **marquage** : c’est l’état global du système.

## 3. Forme mathématique d’un réseau ordinaire

Un réseau de Petri pondéré est le tuple :

\[
N = (P, T, F, W, M_0)
\]

où :

- \(P\) est l’ensemble fini des places ;
- \(T\) est l’ensemble fini des transitions, avec \(P \cap T = \varnothing\) ;
- \(F \subseteq (P \times T) \cup (T \times P)\) est l’ensemble des arcs ;
- \(W : F \rightarrow \mathbb{N}^{+}\) attribue un poids à chaque arc ;
- \(M_0 : P \rightarrow \mathbb{N}\) est le marquage initial.

Pour une transition \(t\), les places d’entrée sont :

\[
{}^\bullet t = \{p \in P \mid (p,t) \in F\}
\]

La transition \(t\) est **franchissable** dans le marquage \(M\) si chaque place d’entrée dispose d’assez de jetons :

\[
\forall p \in {}^\bullet t,\quad M(p) \geq W(p,t)
\]

Lorsque \(t\) est tirée, elle retire les jetons de ses entrées et ajoute des jetons à ses sorties. Le résultat est un nouveau marquage \(M'\).

## 4. Exemple minimal : une zone critique

On veut protéger une zone \(Z\) utilisable par un seul processus à la fois.

Places :

- `attente` : processus qui demandent Z ;
- `Z_libre` : contient initialement un jeton ;
- `dans_Z` : processus qui occupent Z ;
- `sorti` : processus ayant fini.

Transitions :

- `entrer_Z` consomme un jeton de `attente` et le jeton de `Z_libre`, puis produit un jeton dans `dans_Z` ;
- `sortir_Z` déplace le processus vers `sorti` et remet un jeton dans `Z_libre`.

Le seul jeton dans `Z_libre` joue le rôle logique d’un mutex : tant qu’il est absent, aucun autre processus ne peut déclencher `entrer_Z`.

## 5. Traduction pour le carrefour

### Processus

Chaque véhicule ou piéton est un processus. Au début, il possède une demande de mouvement : origine, destination et trajectoire.

### Ressources

Une zone de conflit devient une ressource. Son jeton `zone_libre` signifie qu’elle peut être attribuée. Lorsqu’un processus est admis, il retire ce jeton jusqu’à sa sortie de la zone.

### Ordonnanceur

L’ordonnanceur n’est pas le jeton de ressource. C’est une règle qui détermine quelles transitions d’admission peuvent être tirées.

Exemple : la transition `admettre(p)` est tirable seulement si :

1. le processus \(p\) est en attente ;
2. les zones de sa trajectoire sont libres ;
3. sa priorité est suffisante face aux demandes incompatibles.

### Refus de priorité

Le refus de priorité correspond à l’autorisation de la transition `admettre(q)` alors qu’un processus \(p\), prioritaire et incompatible avec \(q\), devrait passer avant lui. C’est donc une violation de la **politique d’admission**, pas une simple absence de mutex.

## 6. Propriétés à étudier

| Propriété | Question pour le carrefour |
|---|---|
| Sûreté | Deux processus incompatibles peuvent-ils occuper la même zone ? |
| Bornitude | Une file ou une zone peut-elle recevoir un nombre non borné de jetons ? |
| Vivacité | Une transition utile pourra-t-elle encore se produire ? |
| Absence d’interblocage | Peut-on atteindre un marquage où tous les processus attendent définitivement ? |
| Équité / absence de famine | Un processus en attente peut-il être repoussé indéfiniment ? |
| Atteignabilité | Un état interdit, tel qu’une collision, peut-il être atteint ? |

### Invariant de sûreté

Pour une ressource exclusive \(z\), une forme d’invariant est :

\[
M(\text{z\_libre}) + \sum_{p \text{ occupant } z} 1 = 1
\]

Il signifie qu’une zone est soit libre, soit occupée par exactement un processus. Cet invariant exclut l’occupation simultanée de la zone par deux processus incompatibles.

## 7. Pourquoi les réseaux colorés ?

Un réseau ordinaire utiliserait souvent une place ou une transition répétée pour chaque type de processus et chaque trajectoire. Un **réseau de Petri coloré** évite cette répétition : un jeton porte des données, appelées sa couleur.

Pour notre projet, une couleur peut être :

\[
\texttt{Processus} = (id, type, entree, sortie, trajectoire, priorite, t_{arrivee})
\]

Exemples :

- `type` : véhicule, piéton, véhicule prioritaire ;
- `trajectoire` : nord-vers-est, ouest-vers-sud, traversée-piétonne ;
- `priorite` : valeur calculée à partir des panneaux et de l’attente.

Une garde sur une transition exprime une condition. Par exemple :

\[
\texttt{garde(admettre(p))} = \texttt{zonesLibres(p)} \land \texttt{prioritaire(p)}
\]

## 8. Pourquoi les réseaux temporisés ?

Dans un réseau temporisé, une transition ou un jeton porte une durée. Cela permet de représenter :

- le temps de traversée d’une zone ;
- l’instant d’arrivée dans la file ;
- le temps d’attente ;
- le débit et la longueur des files ;
- une politique de vieillissement de priorité.

Une priorité dynamique possible est :

\[
\pi_p(t) = \pi_p^0 + \alpha (t - t_{arrivee,p})
\]

où \(\pi_p^0\) est la priorité issue de la signalisation et \(\alpha\) augmente progressivement la priorité d’un processus qui attend. Cette règle doit être discutée : elle réduit le risque de famine, mais modifie l’ordre strict des priorités.

## 9. Le modèle que nous construirons

Nous utiliserons un réseau de Petri coloré temporisé et un graphe de conflits.

Le graphe de conflits contient :

- un sommet pour chaque type de trajectoire ;
- une arête entre deux sommets si ces trajectoires ne peuvent pas être autorisées en même temps.

Le réseau de Petri utilise ce graphe pour contrôler les transitions d’admission. Il représente la dynamique des processus et ressources dans le temps.

## 10. Correspondance avec C/POSIX

| Réseau de Petri | Simulation C/POSIX |
|---|---|
| Jeton processus | structure décrivant un thread/processus simulé |
| Place d’attente | file protégée par mutex |
| Transition d’admission | décision de l’ordonnanceur central |
| Jeton `zone_libre` | état disponible du mutex/sémaphore de zone |
| Tir d’une transition | modification atomique de l’état partagé |
| Garde | condition testée sous le mutex du contrôleur |
| Temporisation | horloge de simulation ou attente contrôlée |

Le contrôleur doit décider l’admission et réserver toutes les zones nécessaires dans une même section critique. Cela évite qu’un processus détienne une zone tout en attendant une autre, situation qui peut contribuer à un interblocage.

## 11. Méthode de construction du modèle

1. Définir les branches, voies et zones de conflit du carrefour de référence.
2. Lister toutes les trajectoires possibles.
3. Construire le graphe de conflits entre trajectoires.
4. Définir la politique de priorité issue des panneaux et des feux retenus.
5. Dessiner le réseau de base : arrivée, attente, admission, occupation, libération, sortie.
6. Ajouter les couleurs des processus et les temporisations.
7. Énoncer les invariants et tester les scénarios de conflit, forte affluence et attentes longues.

## 12. Vocabulaire à retenir

- **Place** : condition ou état susceptible de contenir des jetons.
- **Transition** : événement atomique qui modifie le marquage.
- **Jeton** : présence d’un processus ou d’une unité de ressource.
- **Marquage** : état global du réseau.
- **Tir/franchissement** : exécution d’une transition activée.
- **Transition activée/franchissable** : toutes ses conditions d’entrée sont satisfaites.
- **Invariant** : relation vraie pour tous les marquages atteignables.
- **Réseau coloré** : jetons portant des données typées.
- **Réseau temporisé** : modèle incluant des durées ou dates.
