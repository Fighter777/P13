# 🍷 P13 — Documentation de la démarche

## Amélioration du livrable du P6 : segmentation descriptive d’un catalogue de caviste

> **Livrable associé :** [notebook P13_P6_segmentation_kmeans.ipynb](https://github.com/Fighter777/P13/blob/main/notebook/P13_P6_segmentation_kmeans.ipynb)  
> **Finalité :** produire une aide à la revue du catalogue, pas une prévision de ventes ni une décision automatique.

---

## 🧭 Sommaire

1. [Cahier des charges fonctionnel](#1--cahier-des-charges-fonctionnel)
2. [Veille métier et technologique](#2--veille-métier-et-technologique)
3. [Démarche analytique, essais et décisions](#3--démarche-analytique-essais-et-décisions)
4. [Mini-formation métier](#4--mini-formation-métier)
5. [Organisation, traçabilité et reproductibilité](#5--organisation-traçabilité-et-reproductibilité)

---

# 1. 🎯 Cahier des charges fonctionnel

## 1.1 Contexte et besoin métier

Le livrable initial du P6 rapproche des données ERP et un export du site web d’un caviste. Il permet d’analyser le catalogue, mais ne fournit pas de lecture synthétique des profils de références.

Le responsable catalogue, les achats et le pilotage commercial ont besoin d’un point d’entrée pour examiner les références qui méritent une vérification : stock potentiellement immobilisé, produit premium à faible volume, cœur de catalogue ou référence inactive.

> **Problématique reformulée**  
> Comment regrouper les références à partir de leur prix, marge, stock et ventes cumulées afin de prioriser une revue humaine du catalogue ?

La segmentation ne remplace jamais la connaissance métier : elle signale des profils à vérifier avant toute décision de réassort, de prix, de mise en avant ou de déstockage.

## 1.2 Parties prenantes et décisions soutenues

| Partie prenante | Usage du résultat | Décision soutenue |
| --- | --- | --- |
| Responsable catalogue | Contrôle des profils et références atypiques. | Audit ou correction de fiche produit. |
| Équipe achats | Vérification avant réassort. | Contrôle du niveau de stock, de la marge et de la disponibilité fournisseur. |
| Pilotage commercial | Lecture des familles de références. | Mise en avant, suivi ou revue de stock. |
| Analyste data | Relance et contrôle sur un nouvel export. | Validation des données et restitution des évolutions. |

## 1.3 Périmètre

| Inclus | Exclu |
| --- | --- |
| Références rapprochées ERP / site web. | Données clients et transactions individuelles. |
| Prix HT, taux de marge, stock et ventes cumulées. | Prévision de la demande, saisonnalité et calendrier de réassort. |
| Segmentation descriptive, profilage et recommandations de vérification. | Modification automatique des stocks, prix ou commandes. |
| K-means comme méthode principale, CAH/Ward comme contrôle structurel, et ACP comme représentation testée. | Publication de données commerciales brutes dans le portfolio. |

> [!WARNING]
> La colonne post_date_gmt est une date de publication sur le site web, pas une date d’achat. Les ventes sont cumulées et non datées : aucune analyse temporelle crédible ne peut être produite à partir de cet export.

## 1.4 Contraintes et ressources

| Catégorie | Élément | Conséquence |
| --- | --- | --- |
| Données | 714 références utilisables ; ventes cumulées non datées. | Segmentation descriptive uniquement. |
| Qualité | Prix, marge ou stock potentiellement incohérents. | Contrôles préalables et validation métier. |
| Confidentialité | Données commerciales sensibles. | Publication limitée à la méthode et aux résultats agrégés. |
| Budget / volume | Volume réduit, budget logiciel nul. | Exécution locale avec des outils open source. |
| Matériel | PC portable de développement ; serveur dédié pour veille/VPN ; Raspberry Pi Kanboard ; PC IA local. | Les outils de suivi sont séparés du notebook P6. |
| Outillage | Python, Jupyter, Git, pandas, NumPy, SciPy, scikit-learn, Matplotlib, Seaborn, openpyxl. | Environnement relançable sans cloud. |

## 1.5 Critères d’acceptation

| Axe | Critère de réussite |
| --- | --- |
| Données | Chaque référence est utilisée ou associée à un motif d’exclusion identifiable. |
| Analyse | K=3 et K=4 comparés avec silhouette, inertie, tailles et profils. |
| Interprétabilité | Chaque segment décrit avec les variables métier et une action de vérification humaine. |
| Reproductibilité | Paramètres, transformations et aléas fixés et documentés. |
| Métier | Une personne non technique sait interpréter résultat et limites après la prise en main. |
| Confidentialité | Aucune ligne de données source dans le portfolio public. |

---

# 2. 🔎 Veille métier et technologique

## 2.1 Question de veille

> Quelle méthode de segmentation descriptive convient à un petit nombre de variables métier tout en restant interprétable, reproductible et honnête sur les limites de l’export ?

La veille porte d’abord sur les **méthodes d’analyse** pertinentes pour le P6. Elle est complétée par le suivi des versions des outils réellement utilisés, via [Veille Data & IA](https://github.com/Fighter777/veille-data-ia-app).

## 2.2 Méthodes étudiées

| Option | Cas d’usage | Atouts | Limites | Décision |
| --- | --- | --- | --- | --- |
| **K-means** standardisé | Construire des profils de références. | Rapide, reproductible, profils simples à restituer. | Le nombre de groupes doit être justifié. | **Retenu**. |
| **CAH / Ward** | Lire la structure et contrôler le découpage. | Dendrogramme et sauts de distance lisibles. | Moins directe pour une restitution opérationnelle. | **Contrôle**. |
| **ACP puis K-means** | Réduire les dimensions et visualiser. | Contrôle la redondance ; projection graphique. | Axes moins parlants, peu de variables. | **Essai, pas pipeline final**. |
| **DBSCAN** | Tester densité / atypies sans imposer k. | Peut isoler du bruit. | Paramètres sensibles ; profils moins immédiats. | **Veille seulement**. |

Prévision de ventes, modèle supervisé et recommandations automatisées sont écartés : aucune cible labellisée ni transaction datée n’est disponible.

## 2.3 Critères de comparaison

| Critère | Application au projet |
| --- | --- |
| Qualité | Silhouette, inertie, taille et cohérence des profils. |
| Robustesse | Standardisation, transformation logarithmique et contrôle CAH. |
| Interprétabilité | Explication par prix, marge, stock et ventes. |
| Coût / délai | Exécution locale, stack open source, faible volume. |
| Reproductibilité | Notebook versionné ; random_state=42 et n_init=30. |
| Maintenabilité | Quatre variables sans dépendance externe. |
| Sécurité | Données brutes hors dépôt public. |

## 2.4 Sources consultées

| Source | Apport retenu |
| --- | --- |
| [scikit-learn — Silhouette analysis](https://scikit-learn.org/stable/auto_examples/cluster/plot_kmeans_silhouette_analysis.html) | Comparer plusieurs valeurs de k. |
| [scikit-learn — Hypothèses de K-means](https://scikit-learn.org/stable/auto_examples/cluster/plot_kmeans_assumptions.html) | Cadrer les limites de forme, échelle et atypies. |
| [scikit-learn — Clustering hiérarchique](https://scikit-learn.org/stable/modules/clustering.html#hierarchical-clustering) | Utiliser la CAH comme contrôle de structure. |
| [scikit-learn — ACP](https://scikit-learn.org/stable/modules/decomposition.html#pca) | Interpréter variance expliquée et projection. |
| [scikit-learn — DBSCAN](https://scikit-learn.org/stable/modules/clustering.html#dbscan) | Cadrer l’usage d’une méthode par densité. |

Les versions de Python, pandas, SciPy, scikit-learn, Matplotlib, Seaborn et Jupyter sont suivies dans l’application de veille. Une mise à jour n’est retenue que pour une correction de sécurité, de compatibilité, de reproductibilité ou une valeur métier démontrée.

> [!NOTE]
> L’agent IA a pu lire, modifier et exécuter le notebook dans le workspace. Il a assisté l’exploration, la structuration, le code et certains traitements. Les résultats retenus ont été vérifiés dans le notebook et les décisions méthodologiques finales ont été validées par l’analyste.

---

# 3. 🧪 Démarche analytique, essais et décisions

## 3.1 Données et préparation

Le clustering s’appuie sur quatre variables, disponibles pour les 714 références.

| Variable | Rôle | Préparation |
| --- | --- | --- |
| Prix HT | Niveau de prix. | Standardisation. |
| Taux de marge | Rentabilité relative. | Standardisation. |
| Stock disponible | Niveau de stock à examiner. | log1p puis standardisation. |
| Ventes cumulées | Activité historique non temporelle. | log1p puis standardisation. |

Le chiffre d’affaires et la valorisation du stock sont réservés au **profilage** : ils ne sont pas utilisés dans la distance K-means afin de ne pas compter deux fois les mêmes phénomènes.

## 3.2 ACP : contrôle méthodologique, pas modèle final

L’ACP est conservée pour répondre à une question méthodologique : la réduction de dimension modifie-t-elle matériellement le résultat ? Avec seulement quatre variables métier lisibles, elle n’est pas imposée à la segmentation finale.

![Scree plot du notebook : trois composantes expliquent 93,7 % de la variance.](https://raw.githubusercontent.com/Fighter777/P13/main/assets/p13_scree_plot_acp.png)

Trois composantes expliquent 93,7 % de la variance. Ce résultat est utile pour visualiser les données, mais ne justifie pas de remplacer les quatre variables métier par des axes moins explicables.

| Représentation | K=3 | K=4 | Usage |
| --- | ---: | ---: | --- |
| Axes ACP (3 composantes) | 0,475 | 0,444 | Essai de robustesse et visualisation. |
| Variables standardisées (4 variables) | 0,461 | 0,447 | **Entrée du modèle final.** |

## 3.3 K-means et contrôle hiérarchique

Le notebook compare K-means de K=2 à K=7. La silhouette est maximale à K=3 (0,461), mais K=4 reste proche (0,447) et isole un profil de références sans stock ni ventes cumulées.

![Silhouette et inertie selon le nombre de clusters.](https://raw.githubusercontent.com/Fighter777/P13/main/assets/p13_silhouette_coude.png)

La CAH avec liaison de Ward est utilisée comme second regard. Comme au P11, le tableau des derniers sauts est calculé : le plus grand saut apparaît au passage de 3 à 2 groupes (+18,69). Trois groupes sont donc statistiquement défendables ; quatre doivent être retenus pour une raison métier explicite.

![Dendrogramme CAH avec méthode de Ward.](https://raw.githubusercontent.com/Fighter777/P13/main/assets/p13_dendrogramme_cah.png)

| Comparaison | Résultat |
| --- | --- |
| Silhouette K=3 | 0,461 |
| Silhouette K=4 | 0,447 |
| Accord CAH / K-means à quatre groupes (ARI) | 0,754 |
| Décision | K=4 pour distinguer un profil inactif utile à l’audit. |

## 3.4 Arbitrage K=3 versus K=4

![Comparaison des partitions K=3 et K=4, projetées sur les deux premiers axes ACP pour la lecture.](https://raw.githubusercontent.com/Fighter777/P13/main/assets/p13_comparaison_k3_k4.png)

| K=3 | K=4 | Arbitrage |
| --- | --- | --- |
| Meilleure silhouette. | Silhouette légèrement inférieure. | Le seul score ne suffit pas. |
| Pas de segment spécifiquement dédié aux références inactives. | 32 références sans stock ni ventes médianes forment un groupe distinct. | K=4 rend visible un profil inactif utile à l’audit catalogue. |
| Lecture plus compacte. | Quatre actions de contrôle distinctes. | **K=4 retenu.** |

## 3.5 Résultat : quatre profils à vérifier

![Profils moyens standardisés des quatre segments.](https://raw.githubusercontent.com/Fighter777/P13/main/assets/p13_profils_standardises.png)

| Segment métier | Références | Lecture synthétique | Vérification humaine prioritaire |
| --- | ---: | --- | --- |
| Cœur de catalogue actif | 449 | Prix faible, stock et ventes cumulées élevés. | Sécuriser le réassort et surveiller les ruptures. |
| Premium à faible volume | 203 | Prix élevé, peu de stock et ventes modérées. | Vérifier positionnement, marge et disponibilité. |
| Surstock à surveiller | 30 | Stock élevé, marge plus faible et volume limité. | Réduire ou geler le réassort ; examiner millésime et promotion. |
| Références inactives à auditer | 32 | Stock et ventes cumulées nuls dans l’export. | Vérifier fiche, retrait, erreur de donnée ou déstockage. |

![Vue portefeuille : stock, ventes cumulées et chiffre d’affaires par segment.](https://raw.githubusercontent.com/Fighter777/P13/main/assets/p13_portefeuille_actions.png)

## 3.6 Traçabilité de l’usage de l’IA

Principe de traçabilité — L’IA a été utilisée comme agent d’assistance pour explorer, implémenter et challenger certaines pistes. Les idées et décisions ne lui sont pas attribuées par défaut : leur origine est distinguée entre propositions de l’analyste, apports du mentor, suggestions de l’IA et constats issus des résultats du notebook. Les décisions finales ont été prises après vérification dans le notebook.

| Sujet                          | Origine                                                                                              | Rôle de l’IA                                                                                 | Vérification / résultat                                                                                     | Décision finale de l'analyste                                           |
| ------------------------------ | ---------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| Prévision des ventes           | **Analyste** : idée initiale d’ajouter un modèle prédictif.                                          | A challengé la faisabilité en examinant la nature de `post_date_gmt` et des ventes cumulées. | Pas de transactions datées exploitables ; `post_date_gmt` correspond à la publication de la fiche produit.  | Prévision et saisonnalité écartées.                       |
| Choix de K-means               | **Analyste** : choix explicite de partir sur K-means.                                                | A aidé à cadrer et implémenter la segmentation.                                              | Résultats calculés dans le notebook.                                                                        | K-means retenu comme méthode principale.                  |
| Reprise de la méthodologie P11 | **Analyste** : demande explicite de reprendre la méthode déjà utilisée au P11.                       | A adapté cette démarche aux données du P6 et aidé à l’implémentation.                        | Standardisation, CAH/Ward, ACP de contrôle et profilage exécutés dans le notebook.                          | Méthode P11 adaptée au P6.                                |
| Comparaison avec / sans ACP    | **Discussion analyste–IA, essai décidé par l’analyste** après constat du faible nombre de variables. | A aidé à implémenter la comparaison entre axes ACP et variables standardisées.               | Résultats très proches ; l’ACP n’apporte pas assez pour justifier son utilisation dans le pipeline final.   | ACP conservée comme contrôle et visualisation uniquement. |
| ACM / FAMD / DBSCAN            | **Mentor** : méthodes évoquées lors de l’accompagnement.                                             | A expliqué leur adéquation théorique avec les types de variables.                            | L’analyste a explicitement demandé de ne pas les intégrer au pipeline.                                      | DBSCAN cité en veille seulement ; ACM/FAMD non utilisées. |
| Arbitrage K=3 / K=4            | **Résultats du notebook + besoin métier**, pas une idée attribuée à l’IA.                            | A aidé à mettre les partitions côte à côte et à produire les contrôles/visualisations.       | K=3 meilleur statistiquement ; K=4 permet une lecture métier supplémentaire.                                | **K=4 retenu par l’analyste** pour la restitution métier. |


## 3.7 Limites et suites

| Limite | Risque | Réponse |
| --- | --- | --- |
| Ventes cumulées non datées | Aucune conclusion sur demande récente ou saisonnalité. | Ne pas parler de prévision. |
| Date de publication web | Confusion avec une date de vente. | Ne pas l’utiliser temporellement. |
| Valeurs catalogue incohérentes | Groupes artificiels. | Contrôler les références avant action. |
| Export ancien | Profils obsolètes. | Relancer sur un export pertinent. |
| Variables et K choisis | Les segments peuvent évoluer. | Documenter les paramètres et conserver la validation métier. |

Une prévision de demande ne deviendrait crédible qu’après obtention de transactions réellement datées et un nouveau cadrage.

---

# 4. 🧑‍🏫 Mini-formation métier

L’objectif est d’utiliser les segments comme une liste de contrôle, sans les confondre avec une décision automatisée.

Le [support de prise en main métier](https://github.com/Fighter777/P13/blob/main/formation/presentation_prise_en_main_metier.pptx) accompagne cette formation.

| Séquence | Durée | Contenu | Résultat attendu |
| --- | ---: | --- | --- |
| Limites des données | 5 min | Ventes cumulées et absence de date d’achat exploitable. | Les utilisateurs savent ce que le livrable ne permet pas. |
| Lecture des segments | 8 min | Prix, marge, stock et ventes des quatre profils. | Relier un segment à une question métier. |
| Cas pratique | 5 min | Examiner une référence et rechercher un faux positif. | Comprendre la validation humaine. |
| Relance et retours | 2 min | Identifier la relance du notebook et la remontée d’incohérences. | Connaître les responsabilités. |

## 4.1 Assistant IA de lecture du notebook

L’[assistant IA de lecture du notebook](https://github.com/Fighter777/assistant_IA_notebook) facilite l’explication de la démarche et des résultats à partir du notebook chargé : code, cellules Markdown, sorties textuelles sauvegardées et graphiques. Il permet de poser des questions en français sur l’ensemble du notebook ou sur une étape ciblée, sans avoir à parcourir toutes les cellules manuellement.

L’application n’exécute pas le notebook : elle restitue ses contenus et sorties enregistrées. Ses réponses constituent une aide à la lecture ; elles doivent être confrontées au code, aux données et aux limites déjà documentées avant toute interprétation métier.


---

# 5. 📋 Organisation, traçabilité et reproductibilité

Cette section porte sur la **Mission 1 uniquement**. Le portfolio et l’application de veille ont un autre périmètre : ils ne sont ni inclus dans les charges, ni utilisés comme preuve d’avancement du P6.

## 5.1 Découpage en lots

| Lot | Objectif | Sortie |
| --- | --- | --- |
| L1 — Cadrage | Besoin, périmètre, contraintes et recette. | Section 1. |
| L2 — Veille | Comparer et justifier les méthodes. | Section 2. |
| L3 — Préparation | Contrôler, transformer, standardiser. | Notebook. |
| L4 — Modélisation | K-means, CAH/Ward et essai ACP. | Métriques et graphiques. |
| L5 — Validation | Arbitrage K=3 / K=4, profils et limites. | Décision documentée. |
| L6 — Restitution | Documentation, formation et démonstration. | Livrables relus. |
| L7 — Automatisation future | Automatiser l’exécution des scripts. | Lot prospectif, hors charge actuelle. |

## 5.2 Backlog de la Mission 1

Charges indicatives issues du backlog existant ; elles ne sont pas un temps réellement pointé.

| ID | Tâche / user story | Lot | Charge | Dépendances | Definition of Done | État |
| --- | --- | --- | ---: | --- | --- | --- |
| P13-01 | Cadrer besoin, utilisateurs, périmètre et exclusions. | L1 | 0,5 j | — | Critères de réussite formalisés. | ✅Terminé |
| P13-02 | Réaliser la veille méthodes / outils. | L2 | 0,5 j | P13-01 | Options sourcées, comparées et justifiées. | ✅Terminé |
| P13-03 | Préparer les variables. | L3 | 0,5 j | P13-01 | log1p, standardisation et contrôles exécutés. | ✅Terminé |
| P13-04 | Tester K-means. | L4 | 0,5 j | P13-03 | K=2 à K=7, silhouette, inertie et tailles disponibles. | ✅Terminé |
| P13-05 | Contrôler par CAH/Ward et comparer K=3 / K=4. | L4 | 0,5 j | P13-04 | Dendrogramme, sauts, ARI et profils comparatifs. | ✅Terminé |
| P13-06 | Profiler et retenir la solution. | L5 | 0,5 j | P13-05 | K=4 justifié comme arbitrage métier ; actions et limites rédigées. | ✅Terminé |
| P13-07 | Tracer l’usage de l’IA. | L5 | 0,5 j | P13-02, P13-06 | Usages, vérifications et décisions documentés. | ✅Terminé |
| P13-08 | Préparer la mini-formation. | L6 | 0,5 j | P13-06 | Support de 20 min compréhensible par le métier. | ✅Terminé |
| P13-09 | Finaliser documentation et reproductibilité. | L6 | 0,5 j | P13-02, P13-07, P13-08 | Document unique cohérent et relu. | ✅Terminé |
| P13-10 | Préparer la restitution. | L6 | 0,5 j | P13-09 | Notebook relancé et démonstration prête. | 🕒En cours |

Charge indicative totale : **5 jours**.

## 5.3 Dépendances entre tâches

~~~mermaid
flowchart LR
  A[P13-01 Cadrage] --> B[P13-02 Veille]
  A --> C[P13-03 Préparation]
  C --> D[P13-04 K-means]
  D --> E[P13-05 CAH et comparaison]
  E --> F[P13-06 Profilage et décision]
  B --> G[P13-07 Trace IA]
  F --> G
  F --> H[P13-08 Mini-formation]
  B --> I[P13-09 Documentation]
  G --> I
  H --> I
  I --> J[P13-10 Restitution]
~~~

L’ACP est testée dans L4 comme représentation alternative : elle n’est pas un troisième algorithme de clustering.

## 5.4 Kanban

### État de pilotage de la Mission 1 au moment de la finalisation

Cette représentation est construite à partir du backlog P13-01 à P13-10 et des livrables actuellement présents dans la Mission 1. Elle décrit l’état de pilotage actuel ; elle ne prétend pas reconstituer un historique Kanban.

| Terminé | En cours | À faire |
| --- | --- | --- |
| P13-01 — Cadrage | P13-09 — Documentation et reproductibilité | P13-10 — Préparation de la restitution |
| P13-02 — Veille |  |  |
| P13-03 — Préparation des variables |  |  |
| P13-04 — Tests K-means |  |  |
| P13-05 — Contrôle CAH / K=3-K=4 |  |  |
| P13-06 — Profilage et décision |  |  |
| P13-07 — Traçabilité IA |  |  |
| P13-08 — Mini-formation métier |  |  |

Les éléments terminés sont vérifiables dans le notebook, les sections 1 à 4 et le support de prise en main. La documentation reste en cours de finalisation ; la restitution reste à préparer après la dernière exécution complète du notebook.

## 5.5 Jalons et planning

| Date vérifiable | Trace |
| --- | --- |
| 21 août 2026 | Ajout de la base du projet P13 dans Git. |
| 29 août 2026 | Commit de structuration des livrables P13. |

Les dates de début/fin de chaque lot ne sont pas traçables de manière fiable. Le planning est donc relatif :

| Jalon | Condition de passage | État |
| --- | --- | --- |
| J1 — Cadrage | Périmètre fixé ; aucune prévision sans ventes datées. | ✅Validé |
| J2 — Veille | Méthodes comparées selon des critères explicites. | ✅Validé |
| J3 — POC | Notebook exécuté et résultats lisibles. | ✅Validé |
| J4 — Validation | K=3/K=4 comparés ; K=4 justifié pour l’usage métier. | ✅Validé |
| J5 — Livraison | Document, formation, traçabilité IA et notebook cohérents. | ✅Validé |
| J6 — Restitution | Démonstration préparée. | 🕒En cours |

## 5.6 Gantt relatif de la Mission 1

Les dates ci-dessous sont artificielles et servent uniquement au rendu Mermaid ; elles ne représentent pas les dates réelles du projet.

~~~mermaid

%%{init: {
  "gantt": {
    "barHeight": 30,
    "barGap": 8,
    "leftPadding": 180,
    "fontSize": 16,
    "sectionFontSize": 16,
    "topPadding": 55
  }
}}%%

gantt
    title Séquencement relatif des lots de la Mission 1
    dateFormat  YYYY-MM-DD
    axisFormat  %d/%m
    section Enchaînement logique
    L1 — Cadrage (0,5 j)          :l1, 2000-01-01, 0.5d
    L2 — Veille (0,5 j)           :l2, after l1, 0.5d
    L3 — Préparation (0,5 j)      :l3, after l1, 0.5d
    L4 — Modélisation (1 j)       :l4, after l3, 1d
    %% L5 dépend de L2 et L4 ; L2 s'achève avant L4 dans ce séquencement.
    L5 — Validation / IA (1 j)    :l5, after l4, 1d
    L6 — Restitution (1,5 j)      :l6, after l5, 1.5d
~~~

L2 et L3 peuvent être menés en parallèle après le cadrage.

## 5.7 Registre des risques

| Risque | Probabilité | Impact | Prévention / parade | Statut |
| --- | --- | --- | --- | --- |
| Confondre publication et vente. | Élevée | Élevé | Exclure prévision et saisonnalité. | ✅Maîtrisé |
| Surinterpréter les clusters. | Moyenne | Élevé | Validation métier obligatoire. | ⚠️À surveiller |
| Données obsolètes ou incohérentes. | Élevée | Moyen | Contrôles puis relance sur export pertinent. | ⚠️À surveiller |
| Sensibilité aux variables ou à K. | Moyenne | Moyen | Standardisation, comparaison K=3/K=4 et CAH/Ward. | ✅Maîtrisé |
| Fuite de données brutes. | Faible | Élevé | Sources hors dépôt public ; résultats agrégés seulement. | ✅Maîtrisé |
| Notebook non relançable. | Moyenne | Élevé | Exécution complète avant soutenance. | Ouvert |
| Support peu accessible aux métiers. | Moyenne | Moyen | Formation courte et cas pratique. | ✅Maîtrisé |

## 5.8 Points de contrôle

| Moment | Contrôle | Preuve |
| --- | --- | --- |
| Avant modélisation | Variables, transformations et absence de date de vente exploitable. | Cellules de préparation. |
| Après K-means | Silhouette, inertie et tailles sur plusieurs K. | Tableaux / graphique. |
| Après CAH / ACP | Sauts, ARI et comparaison des représentations. | Dendrogramme et sorties notebook. |
| Avant décision | K=4 présenté comme arbitrage métier, pas comme meilleur score. | Section 3.4. |
| Avant restitution | Exécution complète, liens et limites relus. | Notebook et documentation. |

## 5.9 Reproductibilité

1. Conserver les données P6 localement, sans publier les exports.
2. Utiliser un environnement Python isolé avec pandas, NumPy, SciPy, scikit-learn, Matplotlib, Seaborn, openpyxl et Jupyter.
3. Ouvrir le [notebook](https://github.com/Fighter777/P13/blob/main/notebook/P13_P6_segmentation_kmeans.ipynb), redémarrer le noyau et exécuter les cellules dans l’ordre.
4. Vérifier l’absence d’erreur, les tailles de groupes, les métriques et les graphiques avant restitution.

Les paramètres déterminants sont fixes : quatre variables, log1p pour stock et ventes, standardisation, random_state=42, n_init=30 et K=4.

> ⚠️[!IMPORTANT]⚠️
> Les visuels sont des sorties du notebook. Une dernière exécution complète est nécessaire avant la soutenance si l’export source ou l’environnement évolue.
