# Prévision de la production éolienne par modèles de fondation pour séries temporelles

**Étude comparative, adaptation des modèles et évaluation des performances**

## Présentation
La prévision de la production éolienne est un enjeu pour la gestion des systèmes électriques : les fluctuations du vent rendent la production variable et compliquent l'anticipation à court terme.

Ce projet étudie plusieurs méthodes de prévision sur une série horaire de production éolienne en Allemagne. Il part de modèles de référence simples et de méthodes d'apprentissage automatique, puis examine des **modèles de fondation pour séries temporelles** (*Time Series Foundation Models*, TSFM), utilisés sans adaptation préalable (*zero-shot*) ou après adaptation aux données étudiées (*fine-tuning* et *few-shot*).

L'objectif est de comparer les approches selon un protocole chronologique commun, d'analyser leurs erreurs et de documenter une démarche expérimentale reproductible. Le projet est développé en **Python**, sous forme de trois notebooks Jupyter.

## Organisation de l'étude

| Partie | Notebook | Contenu |
|---|---|---|
| **A - Données et protocole expérimental** | `01_Donnees_et_Protocole.ipynb` | Exploration et préparation des données ; séparation chronologique ; modèle naïf saisonnier ; Gradient Boosting ; test de Diebold–Mariano ; prévisions probabilistes. |
| **B - Modèles et prévisions** | `02_Modeles_et_Previsions.ipynb` | Prévisions sans réentraînement local avec **Tiny Time Mixers (TTM)** et **Chronos-Bolt** ; comparaison aux références. |
| **C - Fine-tuning et analyse des performances** | `03_Fine_tuning_et_Analyse_des_Performances.ipynb` | Adaptation de TTM (*fine-tuning*), expérimentation *few-shot* et analyse des erreurs de prévision. |

Chaque notebook présente les choix méthodologiques, le code, les résultats et leur interprétation. Une synthèse clôt chaque partie.

## Données

Les données proviennent du jeu de séries temporelles horaires d'**Open Power System Data (OPSD)** :

- Fichier source : `time_series_60min_singleindex.csv` ;
- Variable cible : `DE_wind_onshore_generation_actual` (production éolienne terrestre allemande) ;
- Fréquence : horaire ;
- Période étudiée : du **31 décembre 2014 à 23 h UTC** au **30 septembre 2020 à 23 h UTC** ;
- Nombre d'horodatages : **50 401** ;
- Valeurs manquantes sur la cible : **73**, traitées selon les besoins de chaque expérience.

**Source :** [Open Power System Data — Time series](https://data.open-power-system-data.org/time_series/).

Le fichier CSV brut, volumineux, n'est pas destiné à être versionné dans le dépôt. Il doit être téléchargé depuis la source et placé dans `data/raw/`.

## Protocole expérimental

Pour éviter les fuites d'information entre passé et futur, les observations sont réparties **chronologiquement**, sans mélange aléatoire :

| Ensemble | Part approximative | Nombre d'observations |
|---|---:|---:|
| Entraînement | 70 % | 35 280 |
| Validation | 15 % | 7 560 |
| Test | 15 % | 7 561 |

L'ensemble d'entraînement sert à ajuster les modèles ; la validation permet de comparer les approches et de guider les choix méthodologiques ; le test est réservé à l'évaluation finale.

Les prévisions sont notamment évaluées à l'aide de la **MAE** (*Mean Absolute Error*) et, selon les expériences, de la **MASE**, d'une comparaison statistique par le **test de Diebold–Mariano** et de critères adaptés aux prévisions probabilistes.

Les comparaisons doivent porter sur des instants communs et utiliser uniquement les informations disponibles au moment de la prévision.

## Méthodes étudiées

- **Naïf saisonnier à 24 heures** : référence fondée sur la production observée à la même heure la veille.
- **Gradient Boosting** : apprentissage à partir de retards temporels et de variables calendaires ; comparaison entre la prédiction du niveau et celle de la variation horaire.
- **Tiny Time Mixers (TTM)** : modèle de fondation temporel étudié en *zero-shot*, puis avec adaptation.
- **Chronos-Bolt** : modèle de fondation temporel étudié en *zero-shot*.
- **Prévision probabiliste** : étude de l'incertitude et de la qualité des intervalles de prévision.

Les résultats numériques détaillés, les figures et les limites de chaque approche figurent dans les notebooks. Les performances observées dépendent de la période, de l'horizon, du protocole et des paramètres retenus ; elles ne doivent pas être interprétées comme des garanties générales.

## Structure du projet

```text
TSFM_Wind_Forecasting/
├── data/
│   ├── raw/                 # Données sources à télécharger
│   └── processed/           # Données préparées, si nécessaires
├── results/
│   ├── figures/
│   └── tables/
├── 01_Donnees_et_Protocole.ipynb
├── 02_Modeles_et_Previsions.ipynb
├── 03_Fine_tuning_et_Analyse_des_Performances.ipynb
├── README.md
├── requirements.txt
└── .gitignore
```

Cette arborescence est la **structure cible** du dépôt ; les chemins de lecture et d'écriture dans les notebooks doivent correspondre à leur emplacement définitif.

## Installation et exécution

**Environnement utilisé lors du développement :** Windows, Python **3.11.9**, exécution sur CPU.

1. Cloner le dépôt, puis se placer dans son répertoire :

   ```powershell
   git clone https://github.com/<UTILISATEUR>/<DEPOT>.git
   cd <DEPOT>
   ```

2. Créer et activer un environnement virtuel :

   ```powershell
   py -3.11 -m venv .venv
   .\.venv\Scripts\Activate.ps1
   python -m pip install --upgrade pip
   ```

3. Installer les dépendances :

   ```powershell
   python -m pip install -r requirements.txt
   ```

4. Télécharger le CSV OPSD et le placer à l'emplacement suivant :

   ```text
   data/raw/time_series_60min_singleindex.csv
   ```

5. Lancer Jupyter :

   ```powershell
   python -m jupyter lab
   ```

6. Exécuter les notebooks dans l'ordre **A → B → C**, en vérifiant que les chemins relatifs correspondent au répertoire de travail utilisé.

**Remarque sur la reproductibilité :** 
Le fichier requirements.txt précise les dépendances Python utilisées dans ce projet. Les expériences ont été réalisées avec Python 3.11.9, dans un environnement Windows, avec une exécution des modèles sur CPU.

Bien que les trois notebooks aient été exécutés avec succès dans cet environnement, la reproductibilité sur d'autres configurations matérielles ou logicielles peut nécessiter certains ajustements. L'installation des dépendances dans un environnement neuf reste à vérifier.

## Reproductibilité et bonnes pratiques

- Conserver la séparation chronologique des données.
- Ne pas utiliser d'informations futures dans les variables retardées.
- Documenter le traitement des observations manquantes.
- Comparer les modèles sur des observations communes lorsque cela est nécessaire.
- Ne pas versionner l'environnement virtuel, les données volumineuses ni les poids de modèles téléchargés.
- Conserver les sorties essentielles permettant de comprendre les résultats, sans publier de fichiers temporaires.

## Cadre du projet

Ce travail s'inscrit dans une démarche personnelle d'approfondissement des méthodes de prévision des séries temporelles et des modèles de fondation. Il vise à mettre en pratique un protocole expérimental rigoureux, à confronter différentes familles de modèles et à analyser leurs performances et leurs limites.

## Références et ressources

- [Open Power System Data — Time series](https://data.open-power-system-data.org/time_series/) : données de production et de consommation électriques.
- [Granite Time Series / Tiny Time Mixers](https://github.com/ibm-granite/granite-tsfm) : outils et documentation TTM.
- [Chronos](https://github.com/amazon-science/chronos-forecasting) : modèles et documentation Chronos.
- [Scikit-learn](https://scikit-learn.org/stable/) : méthodes d'apprentissage automatique et métriques.

