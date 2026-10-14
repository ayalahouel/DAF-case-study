<p align="center">
  <img src="DAF%20case%20study/images/banniere.png" alt="Prédire les pannes de camions avant qu'elles n'arrivent" width="820">
</p>

<h1 align="center">IA appliquée aux poids lourds</h1>

<p align="center">
  <b>Une étude de maintenance prédictive sur de vrais camions, et cinq démonstrations de méthodes d'IA</b><br>
  <sub>Eya Lahouel · EFREI Paris, cycle ingénieur data et IA · Octobre 2026</sub>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3-3776AB?logo=python&logoColor=white" alt="Python">
  <img src="https://img.shields.io/badge/scikit--learn-ML-F7931E?logo=scikitlearn&logoColor=white" alt="scikit-learn">
  <img src="https://img.shields.io/badge/pandas-data-150458?logo=pandas&logoColor=white" alt="pandas">
  <img src="https://img.shields.io/badge/Jupyter-notebooks-F37626?logo=jupyter&logoColor=white" alt="Jupyter">
</p>

---

Ce projet fait suite à la session de présentation du stage *Ingénieur Informatique / IA appliquée* de DAF Trucks France. Il montre, sur des données publiques, comment l'IA peut répondre à des questions concrètes d'un constructeur de camions.

> **Aucune de ces données ne provient de DAF.** Chaque partie indique si ses données sont ✅ **réelles** ou ⚠️ **simulées**.

## Résultats en un coup d'œil

| | |
|:---:|:---:|
| **98 %** des pannes détectées | **−94 %** de coût de maintenance |
| 270 pannes sur 275 repérées à l'avance | de 137 500 à 7 800, par rapport à l'absence de modèle |

## Sommaire

1. [Étude principale : prédire les pannes (Scania APS)](#1-étude-principale--prédire-les-pannes-scania-aps) ✅
2. [Segmenter les flottes pour l'électrique](#2-segmenter-les-flottes-pour-lélectrique) ⚠️
3. [Estimer la valeur des occasions](#3-estimer-la-valeur-des-occasions) ⚠️
4. [Prévoir la demande en pièces détachées](#4-prévoir-la-demande-en-pièces-détachées) ✅
5. [Lire automatiquement les plaintes](#5-lire-automatiquement-les-plaintes) ✅
6. [Un assistant pour les techniciens](#6-un-assistant-pour-les-techniciens) ✅

---

## 1. Étude principale : prédire les pannes (Scania APS)

✅ **Données réelles** · [`01_scania_aps.ipynb`](DAF%20case%20study/01_scania_aps.ipynb)

**La question :** peut-on prédire qu'un camion va tomber en panne de son système d'air comprimé (freins, vitesses) pour intervenir avant qu'il soit immobilisé ?

**Les données :** 76 000 camions, 170 capteurs, seulement 1,8 % de pannes. Rater une panne coûte **500**, une fausse alerte coûte **10** (coûts définis par Scania pour ce jeu de données). C'est ce coût, et non l'accuracy, qui juge le modèle.

### Des données imparfaites, mais un problème localisé

![Part de valeurs manquantes](DAF%20case%20study/images/qualite_donnees.png)

8,3 % des cases sont vides et 8 capteurs le sont à plus de 50 %. **Trouvaille :** les camions en panne ont en moyenne **17,3 %** de mesures manquantes, contre **8,2 %** pour les autres. Un capteur muet est déjà un signal.

### Le bon seuil divise le coût par trois

![Coût selon le seuil de décision](DAF%20case%20study/images/cout_selon_seuil.png)

Par défaut, un modèle annonce une panne au-delà de 50 % de certitude. C'est trop prudent quand rater une panne coûte 50 fois plus qu'une fausse alerte. Le seuil optimal du Random Forest est **0,10**. Il est choisi par validation croisée sur le jeu d'entraînement : le jeu de test ne sert qu'au résultat final.

### Le résultat

![Coût avant / après](DAF%20case%20study/images/cout_avant_apres.png)

| Approche | Pannes détectées | Fausses alertes | Coût total |
|---|---:|---:|---:|
| Sans modèle | 0 / 275 | 0 | 137 500 |
| Random Forest, seuil par défaut (0,50) | 228 / 275 | 80 | 24 300 |
| Régression logistique, seuil optimisé (0,34) | 256 / 275 | 524 | 14 740 |
| **Random Forest, seuil optimisé (0,10)** | **270 / 275** | 530 | **7 800** |

> **L'accuracy est trompeuse :** sans modèle, elle atteint 98,2 %… sans détecter une seule panne.

### Ce sur quoi le modèle s'appuie

<p align="center"><img src="DAF%20case%20study/images/importance_variables.png" alt="Importance des variables" width="600"></p>

Une quinzaine de capteurs portent l'essentiel de la décision : le modèle n'est pas une boîte noire. **Limite :** les capteurs sont anonymisés par Scania, on ne peut donc pas en donner le sens mécanique. Avec de vraies données, les experts atelier pourraient le vérifier.

---

## 2. Segmenter les flottes pour l'électrique

⚠️ **Données simulées** · [`02_segmentation_flottes.ipynb`](DAF%20case%20study/02_segmentation_flottes.ipynb) · Clustering (K-means), ACP

**La question :** parmi les clients diesel, lesquels utilisent déjà leurs camions d'une façon compatible avec l'électrique ?

![Choix du nombre de groupes](DAF%20case%20study/images/choix_k.png)

![Les 4 profils et leur compatibilité électrique](DAF%20case%20study/images/segments_electrique.png)

- Le K-means retrouve les **4 profils d'usage cachés** (ARI = 0,98) : la méthode est validée.
- **97,9 %** des camions en livraison urbaine et **93,9 %** sur chantier ont un usage compatible avec l'électrique, contre 0 % en longue distance.
- **Limite :** le critère électrique (pire jour ≤ 300 km, retour au dépôt ≥ 80 % des jours) ignore la charge utile.

---

## 3. Estimer la valeur des occasions

⚠️ **Données simulées** · [`03_valeur_occasion.ipynb`](DAF%20case%20study/03_valeur_occasion.ipynb) · Régression

**La question :** combien vaut ce camion d'occasion ?

![Prix prédit contre prix réel](DAF%20case%20study/images/occasion_predit_vs_reel.png)

| Modèle | Erreur moyenne (MAE) | R² |
|---|---:|---:|
| Régression linéaire | 5 163 € | 0,931 |
| **Random Forest** | **3 310 €** | **0,966** |

La décote n'est pas proportionnelle : la régression linéaire sous-estime les camions chers et prédit même des **prix négatifs** pour les moins chers.

---

## 4. Prévoir la demande en pièces détachées

✅ **Données réelles** (2 674 pièces automobiles, 1998-2002, jeu de R. Hyndman) · [`04_pieces_detachees.ipynb`](DAF%20case%20study/04_pieces_detachees.ipynb) · Prévision de demande

**La question :** combien de pièces stocker, quand **75 % des mois n'ont aucune vente** ?

![Comparaison des méthodes de prévision](DAF%20case%20study/images/pieces_comparaison.png)

Le modèle d'IA global (gradient boosting appris sur toutes les pièces à la fois) a la plus petite erreur et réduit les pièces en rupture de **33 % à 24 %** par rapport à la moyenne des 12 derniers mois, pour un stock équivalent. Croston, pourtant la référence théorique, fait moins bien ici.

---

## 5. Lire automatiquement les plaintes

✅ **Données réelles** (537 plaintes de conducteurs de poids lourds, NHTSA) · [`05_plaintes_texte.ipynb`](DAF%20case%20study/05_plaintes_texte.ipynb) · Traitement du texte

**La question :** de quoi parlent des centaines de textes libres, et quel défaut est en train de monter ?

<p align="center">
  <img src="DAF%20case%20study/images/plaintes_matrice_confusion.png" alt="Matrice de confusion" width="380">
  <img src="DAF%20case%20study/images/plaintes_evolution.png" alt="Évolution des plaintes par famille" width="460">
</p>

- **72 %** de plaintes bien classées à partir du texte seul (contre 29 % en répondant toujours la famille la plus fréquente).
- Les plaintes sur les **aides à la conduite** passent de 0 avant 2019 à **21 en 2023** : un nouveau système déployé qui génère des plaintes.
- Sans aucune étiquette, le modèle isole un **défaut précis** qui revient : le boîtier du filtre à carburant.

---

## 6. Un assistant pour les techniciens

✅ **Document réel** (*California Commercial Driver Handbook*, 208 pages) · [`06_assistant_technicien_rag.ipynb`](DAF%20case%20study/06_assistant_technicien_rag.ipynb) · Recherche documentaire (première étape d'un RAG)

**La question :** un technicien pose une question en langage courant ; le système retrouve la page du manuel qui y répond.

![Résultats de l'assistant](DAF%20case%20study/images/assistant_technicien.png)

**3 bonnes réponses sur 4**, page à l'appui. L'échec est instructif : la réponse est page 108, mais le manuel écrit *leakage* là où la question dit *leaks*. La recherche par mots-clés cherche des mots, pas des idées ; l'amélioration prévue est la **recherche sémantique** par *embeddings*, puis la rédaction de la réponse par un LLM.

---

## Reproduire

```bash
pip install -r requirements.txt
```

| Notebook | Données |
|---|---|
| `01` | téléchargées automatiquement via OpenML (`fetch_openml`) |
| `04` | `data/carparts_ventes_mensuelles.csv`, inclus dans le dépôt |
| `05` | récupérées via l'API publique de la NHTSA |
| `06` | PDF *California Commercial Driver Handbook* (site du DMV de Californie), à placer dans `data/comlhdbk.pdf` ; non redistribué ici |

**Outils :** Python, pandas, scikit-learn, matplotlib, pypdf.
