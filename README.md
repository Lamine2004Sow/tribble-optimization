# Optimisation d’un système dynamique d’élevage et de commercialisation

Planifier l’élevage, la conservation, la vente et l’achat de **tribbles** sur un horizon de plusieurs périodes, en tenant compte de la reproduction, de la demande et des capacités de l’exploitation.

Ce projet de recherche opérationnelle vise à comparer des formulations mathématiques, des méthodes exactes, des heuristiques et des matheuristiques. L’enjeu est d’étudier la structure algorithmique du problème et les compromis entre qualité des solutions et temps de calcul.

> Ce README décrit le périmètre et la démarche prévus. Les méthodes présentées ne constituent pas une liste de fonctionnalités déjà implémentées.

## Sommaire

- [Contexte et objectifs](#contexte-et-objectifs)
- [Description du système](#description-du-système)
- [Modélisation mathématique](#modélisation-mathématique)
- [Méthodes de résolution](#méthodes-de-résolution)
- [Génération des instances](#génération-des-instances)
- [Protocole expérimental](#protocole-expérimental)
- [Extension sous incertitude](#extension-sous-incertitude)
- [Feuille de route et livrables](#feuille-de-route-et-livrables)
- [Installation et utilisation](#installation-et-utilisation)

## Contexte et objectifs

Une entreprise élève et commercialise des tribbles, animaux à reproduction rapide. Elle doit satisfaire des commandes sur un horizon allant de quelques mois à plusieurs années, tout en maîtrisant ses coûts.

À chaque période, elle choisit combien d’animaux conserver pour la reproduction, vendre ou acheter auprès de fournisseurs externes. Selon la variante étudiée, elle peut également reporter ou refuser une partie de la demande.

Ces décisions ont des effets différés : conserver davantage d’adultes augmente les coûts immédiats, mais peut accroître la production future et limiter les achats externes. Vendre davantage peut améliorer la marge à court terme tout en réduisant le potentiel reproducteur.

Le projet poursuit quatre objectifs :

- construire et valider un modèle cohérent de la dynamique des populations ;
- analyser les formulations et la qualité de leurs relaxations linéaires ;
- développer des méthodes adaptées aux instances difficiles ;
- comparer les performances et la robustesse des approches proposées.

## Description du système

### Dynamique de l’élevage

Le temps est discrétisé en périodes $t=1,\ldots,T$. Les animaux sont répartis en classes d’âge : les nouveau-nés traversent plusieurs classes avant de devenir adultes et commercialisables.

La dynamique comprend la reproduction, le vieillissement, la mortalité éventuelle et les mouvements liés aux ventes et aux achats. Les naissances dépendent du nombre d’adultes reproducteurs conservés.

La première version repose sur des paramètres déterministes. Des rendements de reproduction variables pourront ensuite être introduits.

### Demande et commercialisation

Une demande $d_t$ doit être traitée à chaque période. Elle peut être satisfaite par l’élevage ou par des achats externes. Le report et le refus sont des options de modélisation, associées à des pénalités ou à une perte de revenus.

Des catégories de clients pourront être distinguées selon leurs prix de vente, leurs pénalités et leurs exigences de service.

### Contraintes opérationnelles

Les variantes du modèle intégreront progressivement :

- une capacité totale d’hébergement et des capacités par classe d’âge ;
- un effectif minimal de reproducteurs à conserver ;
- des plafonds d’achat par fournisseur et par période ;
- des coûts fixes d’activation des fournisseurs ;
- des capacités et coûts de transport ;
- des délais d’approvisionnement propres aux fournisseurs.

## Modélisation mathématique

### Données et décisions

| Élément | Description |
| --- | --- |
| Horizon | Nombre de périodes et durée d’une période |
| Population initiale | Effectifs par classe d’âge au début de l’horizon |
| Paramètres biologiques | Âge de maturité, reproduction et survie |
| Demande | Commandes par période et, éventuellement, par catégorie de clients |
| Fournisseurs | Prix, capacités, coûts fixes et délais |
| Exploitation | Capacités d’hébergement, de transport et coûts de conservation |
| Décisions | Effectifs conservés, ventes, achats et activations de fournisseurs |
| Décisions optionnelles | Quantités reportées ou refusées |

### Fonction objectif

L’objectif principal est de maximiser la marge totale :

$$
\max\; \left\{\text{revenus des ventes} - \left(
C_{\mathrm{achats}} + C_{\mathrm{élevage}} + C_{\mathrm{stockage/capacité}}
+ C_{\mathrm{transport}} + C_{\mathrm{pénalités}} + C_{\mathrm{fixes}}
\right)\right\}.
$$

Lorsque les revenus sont fixés et indépendants des décisions, cette formulation est équivalente à une minimisation des coûts. Cette équivalence doit être réexaminée si des ventes peuvent être refusées ou si les revenus varient avec les décisions.

### Principes de formulation

Les contraintes devront assurer la conservation des populations, les transitions entre classes d’âge, le traitement de la demande et le respect des capacités.

Avant d’écrire les équations, il faudra préciser :

1. l’ordre des événements dans une période : livraisons, ventes, reproduction, vieillissement et mortalité ;
2. la date à laquelle un animal acheté ou devenu adulte peut se reproduire ou être vendu ;
3. le moment où les capacités sont contrôlées ;
4. le traitement du stock final et des commandes encore en attente à la fin de l’horizon.

Ces conventions évitent les doubles comptages et les effets artificiels de fin d’horizon.

### Nature des variables

- **Continues** : approximation agrégée des populations ou relaxation linéaire.
- **Entières** : effectifs et mouvements lorsque chaque animal est représenté individuellement.
- **Binaires** : activation d’un fournisseur ou déclenchement d’un coût fixe.

Une reproduction proportionnelle à taux connu reste linéaire. L’ajout de variables entières conduit à un programme linéaire en nombres entiers, éventuellement mixte (**PLNE / MILP**). Des rendements dépendant non linéairement des décisions peuvent nécessiter une linéarisation ou un modèle non linéaire.

L’application de taux fractionnaires à des effectifs entiers exige une convention explicite : approximation continue, arrondi modélisé ou autre règle biologique cohérente.

Une représentation en réseau temporel sera étudiée pour les variantes adaptées. La reproduction ne conserve pas le nombre d’animaux : une formulation de flot classique et ses propriétés d’intégralité ne peuvent donc pas être supposées sans démonstration.

## Méthodes de résolution

| Approche | Travail prévu |
| --- | --- |
| Modèle de référence | Formulation déterministe et validation sur de petites instances calculables manuellement |
| Résolution exacte | Implémentation progressive des contraintes avec Gurobi, CPLEX ou SCIP |
| Analyse polyédrique | Étude de la relaxation PL, de l’écart d’intégralité et de formulations renforcées |
| Heuristiques constructives | Politique gloutonne, anticipation de la demande, horizon glissant et exploitation de la relaxation PL |
| Recherche locale | Modification des achats, des reproducteurs conservés, des dates et des fournisseurs |
| Métaheuristique | Au moins une méthode parmi recherche tabou, recuit simulé, GRASP, VNS ou algorithme génétique |
| Matheuristique | Combinaison d’une recherche heuristique et de sous-problèmes PLNE : LNS + MIP, fix-and-optimize ou relax-and-fix |

Le choix des voisinages et de la métaheuristique devra être justifié par la dynamique du système. Toute modification d’une décision devra être propagée aux périodes suivantes pour contrôler sa faisabilité et son coût réel.

## Génération des instances

Le générateur devra produire des instances reproductibles à partir d’une graine aléatoire et de paramètres contrôlés :

- horizon et nombre de classes d’âge ;
- nombre de fournisseurs et délais d’approvisionnement ;
- niveau, saisonnalité et volatilité de la demande ;
- prix d’achat et coûts fixes ;
- capacités d’hébergement, d’achat et de transport ;
- population initiale, reproduction et mortalité.

Les familles couvriront des situations de demande stable, de pics saisonniers, de capacités tendues et de forte variabilité des prix. Leur difficulté sera caractérisée à partir des résultats de résolution.

La faisabilité devra être contrôlée. Les instances infaisables seront identifiées séparément ; si le refus de demande est autorisé, son volume et son coût seront également mesurés.

## Protocole expérimental

### Validation

Les premières expériences vérifieront les bilans de population, les délais de maturation, le calcul des coûts et le respect des capacités sur de petites instances. Des cas limites seront inclus : demande nulle, absence d’achats possibles, capacité saturée ou reproduction nulle.

### Comparaison des méthodes

Les méthodes seront évaluées sur les mêmes instances, avec des budgets de calcul comparables. Chaque expérience conservera les paramètres, la graine, le matériel, les versions logicielles et les réglages du solveur.

| Critère | Mesures prévues |
| --- | --- |
| Qualité | Valeur de l’objectif et écart à l’optimum ou à une borne connue |
| Performance | Temps CPU et temps écoulé, avec limite de temps documentée |
| Résolution exacte | Meilleure solution réalisable, borne duale, gap final et nœuds de Branch-and-Bound |
| Relaxation | Valeur de la relaxation PL et écart d’intégralité |
| Robustesse | Dispersion des résultats sur plusieurs graines pour les méthodes aléatoires |
| Service | Demande satisfaite, reportée et refusée, selon la variante |

Pour un problème de minimisation, lorsque l’optimum $z^*\ne 0$ est connu :

$$
\operatorname{gap}_{\mathrm{heur}}(\%) =
100\frac{z_{\mathrm{heur}}-z^*}{|z^*|}.
$$

L’écart d’intégralité de la formulation est mesuré par :

$$
\operatorname{gap}_{\mathrm{int}}(\%) =
100\frac{z^*-z_{\mathrm{PL}}}{|z^*|},
$$

où $z_{\mathrm{PL}}$ est l’optimum de la relaxation linéaire. Si $z^*=0$, un écart absolu sera présenté. Si l’optimum est inconnu, la référence utilisée sera explicitement indiquée : borne duale ou meilleure solution connue. Les résultats en maximisation utiliseront une convention de signe adaptée.

## Extension sous incertitude

Une seconde phase pourra rendre aléatoires la demande, la reproduction, la mortalité ou les prix d’achat. Elle comparera trois approches :

1. **Déterministe** : optimisation à partir de prévisions ponctuelles.
2. **Robuste** : protection contre un ensemble défini de réalisations défavorables.
3. **Stochastique par scénarios** : optimisation sur plusieurs trajectoires possibles.

Les modèles à deux ou plusieurs étapes distingueront les décisions prises avant l’observation des aléas et celles qui peuvent être adaptées ensuite. Des contraintes de non-anticipativité empêcheront une décision d’utiliser des informations futures non encore disponibles.

L’évaluation portera sur les performances hors échantillon, le niveau de service et la valeur de l’information, notamment le gain potentiel associé à une meilleure connaissance de la demande future.

## Feuille de route et livrables

- [ ] Définir les hypothèses et formaliser le modèle de référence.
- [ ] Valider la dynamique sur des exemples manuels.
- [ ] Implémenter la formulation PLNE et sa relaxation linéaire.
- [ ] Étudier et renforcer la formulation.
- [ ] Développer le générateur d’instances.
- [ ] Réaliser le benchmark exact sous limite de temps.
- [ ] Implémenter les heuristiques constructives et la recherche locale.
- [ ] Développer et justifier au moins une métaheuristique.
- [ ] Développer, idéalement, une matheuristique.
- [ ] Comparer les méthodes et analyser leurs limites.
- [ ] Explorer, en extension, les modèles sous incertitude.

Les livrables attendus comprennent les formulations documentées, le code des méthodes, le générateur et les instances, les configurations expérimentales, les résultats et une synthèse des enseignements.

## Installation et utilisation

Le langage, les dépendances, le solveur retenu et les commandes d’exécution restent à préciser selon l’implémentation. Cette section devra documenter l’installation, le format des instances, le lancement d’une résolution et la reproduction des benchmarks dès que le code sera disponible.
