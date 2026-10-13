# DAF-case-study
# IA appliquée aux poids lourds : étude de cas et démonstrations

Projet réalisé par **Eya Lahouel** (EFREI Paris, cycle ingénieur data et IA) à la suite de la session de présentation du stage *Ingénieur Informatique / IA appliquée* chez DAF Trucks France, en octobre 2026.

L'objectif : montrer, sur des données publiques, comment l'IA peut répondre à des questions concrètes d'un constructeur de camions. **Aucune de ces données ne provient de DAF.** Chaque notebook indique si ses données sont **réelles** ou **simulées**.

## Contenu

| Notebook | Question métier | Méthode | Données |
|---|---|---|---|
| `01_scania_aps.ipynb` | **Étude principale** : ce camion va-t-il tomber en panne du système d'air comprimé ? | Classification déséquilibrée, réglage du seuil sur le coût métier | ✅ Réelles : capteurs de camions Scania (APS Failure, UCI / OpenML) |
| `02_segmentation_flottes.ipynb` | Quels clients ont un usage compatible avec l'électrique ? | Clustering (K-means), ACP | ⚠️ Simulées |
| `03_valeur_occasion.ipynb` | Combien vaut un camion d'occasion ? | Régression (linéaire vs Random Forest) | ⚠️ Simulées |
| `04_pieces_detachees.ipynb` | Combien de pièces stocker ? | Prévision de demande intermittente (Croston vs gradient boosting) | ✅ Réelles : ventes de 2 674 pièces automobiles (R. Hyndman) |
| `05_plaintes_texte.ipynb` | De quoi parlent les plaintes, et quel défaut monte ? | Traitement du texte (TF-IDF, NMF, classification) | ✅ Réelles : plaintes de conducteurs de poids lourds (NHTSA, API publique) |
| `06_assistant_technicien_rag.ipynb` | Retrouver la bonne page du manuel pour une question de technicien | Recherche documentaire, première étape d'un RAG | ✅ Réel : manuel du permis poids lourd de Californie |

## Résultat principal : maintenance prédictive (Scania APS)

76 000 camions, 170 capteurs, 1,8 % de pannes et 8,3 % de valeurs manquantes. Rater une panne coûte 500, une fausse alerte coûte 10 (coûts définis par Scania pour ce jeu de données).

| Approche | Pannes détectées | Fausses alertes | Coût total |
|---|---|---|---|
| Sans modèle | 0 / 275 | 0 | 137 500 |
| Random Forest, seuil par défaut (0,50) | 228 / 275 | 80 | 24 300 |
| Régression logistique, seuil optimisé (0,34) | 256 / 275 | 524 | 14 740 |
| **Random Forest, seuil optimisé (0,10)** | **270 / 275** | 530 | **7 800** |

**98 % des pannes détectées et un coût réduit de 94 %.** Le seuil est choisi par validation croisée sur le jeu d'entraînement, et le jeu de test ne sert qu'au résultat final.

![Coût avant / après](images/cout_avant_apres.png)

Autres enseignements :
- **L'accuracy est trompeuse** : sans modèle, elle atteint 98,2 %… sans détecter une seule panne.
- **Un capteur muet est un signal** : les camions en panne ont en moyenne 17,3 % de mesures manquantes, contre 8,2 % pour les autres.
- **Limite** : les capteurs sont anonymisés, on ne peut donc pas interpréter mécaniquement les variables les plus importantes.

## Résultats des démonstrations

- **Segmentation (simulée)** : le K-means retrouve les 4 profils d'usage cachés (ARI = 0,98). Livraison urbaine et chantier sont très majoritairement compatibles avec l'électrique.
- **Occasions (simulées)** : le Random Forest se trompe de 3 310 € en moyenne, contre 5 163 € pour la régression linéaire.
- **Pièces détachées (réelles)** : sur une demande où 75 % des mois sont à zéro, le modèle global réduit les pièces en rupture d'environ 33 % à 24 % par rapport à la moyenne mobile, pour un stock équivalent.
- **Plaintes (réelles)** : 72 % de bonnes classifications à partir du texte seul, détection d'une montée des plaintes sur les aides à la conduite depuis 2019, et d'un défaut précis (boîtier de filtre à carburant) sans aucune étiquette.
- **Assistant technicien (réel)** : 3 bonnes réponses sur 4 avec la page citée. L'échec (« leaks » contre « leakage ») montre la limite de la recherche par mots-clés ; l'amélioration prévue est la recherche sémantique par *embeddings*.

## Reproduire

```bash
pip install -r requirements.txt
```

- `01` télécharge les données Scania automatiquement via OpenML (`fetch_openml`).
- `04` utilise `data/carparts_ventes_mensuelles.csv`, inclus dans le dépôt.
- `05` récupère les plaintes via l'API publique de la NHTSA.
- `06` nécessite le PDF *California Commercial Driver Handbook* (site du DMV de Californie), à placer dans `data/comlhdbk.pdf`. Il n'est pas redistribué ici.

## Outils

Python, pandas, scikit-learn, matplotlib, pypdf.
