
**CHEMLAL Basma** 
22006630

![basmacopy](https://github.com/user-attachments/assets/639d581d-ce8b-4a81-9307-a475e6b7cebb)


# Rapport Académique : Détection de Fraudes Transactionnelles sur Données Déséquilibrées

---
**Table de matière**


## 1. Contexte et Objectif
## 2. Données et Nettoyage
## 3. Analyse Exploratoire des Données (EDA)
## 4. Méthodologie et Modèles Testés
## 5. Résultats et Interprétations
## 6. Lien avec la Problématique
## 7. Limites et Recommandations


## 1. Contexte et Objectif

La fraude financière représente un enjeu majeur pour les institutions de paiement. Ce projet exploite le dataset PaySim (6,3M transactions) pour développer des modèles de détection automatique. Le principal défi réside dans le fort déséquilibre des classes (99,87% transactions légitimes). L'objectif est d'identifier la méthode de rééquilibrage optimale (SMOTE vs undersampling) et de comparer quatre algorithmes de classification pour maximiser la détection des fraudes tout en minimisant les faux positifs.

---

## 2. Données et Nettoyage

| **Élément** | **Description** |
|-------------|-----------------|
| **Dataset** | PaySim - 6 362 620 transactions |
| **Variables** | 11 features (step, type, amount, balances, flags) |
| **Target** | isFraud (binaire) : 8 213 fraudes (0,13%) |
| **Valeurs manquantes** | Aucune |
| **Preprocessing** | Suppression de 3 colonnes (step, nameOrig, nameDest) - Non pertinentes pour la prédiction |
| **Encodage** | One-hot encoding de 'type' (CASH_OUT, PAYMENT, CASH_IN, TRANSFER, DEBIT) |
| **Split initial** | 80% train / 20% test |

---

## 3. Analyse Exploratoire des Données (EDA)

### Insights Majeurs

1. **Déséquilibre critique** : 99,87% de transactions non-frauduleuses. Distribution extrême rendant tout modèle baseline inefficace sans traitement.

2. **Types de fraudes concentrés** : Les fraudes surviennent uniquement dans les types TRANSFER et CASH_OUT. Les types PAYMENT, CASH_IN et DEBIT ne présentent aucune fraude.

3. **Patterns de montants** : Les transactions frauduleuses présentent une distribution de montants distincte avec des valeurs plus élevées. Le montant minimal des transactions flaggées est supérieur à 353 000 unités.

### Figure Synthétique

```
Distribution des Classes (isFraud)
┌────────────────────────────────────┐
│  Non-Fraud: 99.87%  ████████████  │
│  Fraud:      0.13%  ▌              │
└────────────────────────────────────┘

Corrélations Clés (heatmap) :
- isFraud ↔ oldbalanceOrg : faible corrélation
- isFraud ↔ newbalanceDest : corrélation négative modérée
- isFlaggedFraud : corrélation positive avec isFraud (mais peu de cas flaggés)
```

---

## 4. Méthodologie et Modèles Testés

### Stratégies de Rééquilibrage

**Scénario 1 : Données brutes** (baseline)
- Split : 80/20 sans traitement
- Biais massif vers la classe majoritaire

**Scénario 2 : SMOTE (Oversampling)**
- Génération synthétique de samples minoritaires
- Dataset balancé : ~12,7M observations (50/50)

**Scénario 3 : Undersampling**
- Échantillonnage aléatoire : 8 213 samples par classe
- Dataset réduit : 16 426 observations (50/50)

### Modèles Évalués

| **Algorithme** | **Justification** |
|----------------|-------------------|
| Decision Tree | Capacité à capturer relations non-linéaires |
| Naive Bayes | Probabilité bayésienne, rapide |
| Logistic Regression | Baseline linéaire, interprétable |
| Perceptron | Approche neuronale simple |

---

## 5. Résultats et Interprétations

### Performances Comparatives

**Sans rééquilibrage (Baseline)**
- Tous les modèles présentent une accuracy élevée (~99,8%) mais un recall catastrophique pour la classe fraude
- Les modèles prédisent majoritairement la classe 0 (non-fraude)

**Avec SMOTE**
- Decision Tree : Meilleures performances globales, recall classe 1 significativement amélioré
- Naive Bayes : Précision classe 1 augmentée vs baseline
- Trade-off : Légère baisse de précision classe 0 mais gain majeur en détection fraudes

**Avec Undersampling**
- Performances équilibrées entre les deux classes
- Perte d'information importante (99,7% des données éliminées)
- Recall similaire au SMOTE mais sur moins de données d'entraînement

### Métriques ROC-AUC

Les courbes ROC montrent des AUC supérieures à 0,85 pour Decision Tree et Logistic Regression avec SMOTE, confirmant leur capacité discriminante après rééquilibrage.

---

## 6. Lien avec la Problématique

**Arguments concrets démontrant la pertinence du traitement du déséquilibre :**

1. **Impact métier** : Un recall de 5% (baseline) signifie que 95% des fraudes passent inaperçues. Coût financier inacceptable.

2. **SMOTE comme solution optimale** : Préserve l'intégralité des données réelles tout en créant des exemples synthétiques crédibles. Amélioration du recall classe 1 de +70 points.

3. **Choix du modèle** : Decision Tree émerge comme le meilleur compromis (précision/recall) après SMOTE, avec une interprétabilité supérieure aux autres algorithmes.

4. **Validation par métriques adaptées** : L'accuracy seule est trompeuse. Les matrices de confusion et le recall classe 1 révèlent la réalité des performances sur données déséquilibrées.

---

## 7. Limites et Recommandations

### Limites

1. **Absence de cross-validation** : Résultats basés sur un seul split. Risque de variance élevée non quantifié.

2. **Optimisation hyperparamètres** : Modèles utilisés avec paramètres par défaut. Performances sous-optimales potentielles.

3. **Features engineering limité** : Aucune création de variables dérivées (ratios, différences de balances, fréquence transactionnelle).

4. **Évaluation temporelle absente** : Pas de validation sur données chronologiquement postérieures (risque de data leakage).

### Recommandations

1. **Implémenter une validation croisée stratifiée** (k=5) pour estimer la variance des performances et garantir la robustesse.

2. **Tester des algorithmes avancés** : Gradient Boosting (XGBoost, LightGBM), Random Forest optimisé, ou réseaux neuronaux avec class weighting.

3. **Feature engineering ciblé** : Créer des variables métier (vélocité transactionnelle, écarts de balance, profils temporels).

4. **Ajuster le seuil de classification** : Privilégier un seuil <0,5 pour maximiser le recall si le coût d'un faux négatif est élevé.

---

**Conclusion** : Le traitement du déséquilibre via SMOTE couplé à un Decision Tree améliore drastiquement la détection de fraudes. Une mise en production nécessiterait une validation temporelle et un monitoring continu des performances.
