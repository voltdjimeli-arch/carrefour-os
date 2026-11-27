# Carrefour OS

Documentation de conception pour un projet de systèmes d’exploitation qui vise à simuler un carrefour multivoies et multibranches.

Le modèle est en cours d’élaboration. Il s’appuie sur des automates temporisés, un graphe de conflits entre trajectoires et un ordonnanceur central. Il doit pouvoir représenter les zones partagées, les priorités, les convois indivisibles, les véhicules d’urgence, les temps d’attente et les interblocages. Aucune préemption n’est possible sur un véhicule engagé : l’interblocage se traite par prévention ou évitement.

## Documents

- [Modèle global : convois, urgences et interblocage](skill-carrefour-os/references/modele-global.md)
- [Boîte à outils mathématique, non-préemption et chaînes de Markov](skill-carrefour-os/references/outils-mathematiques-et-preemption.md)
- [Skill de continuité du projet](skill-carrefour-os/SKILL.md)
- [Notes de continuité](skill-carrefour-os/references/continuite.md)
- [Introduction aux réseaux de Petri](documentation-reseaux-de-petri.md)

Les réseaux de Petri sont conservés comme une option d’analyse. Le modèle principal doit rester fondé sur les outils mathématiques déjà maîtrisés par l’équipe.
