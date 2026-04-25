# 📈 Statistical Factor Models for Alpha Research – S&P 500

> **Cours :** Gestion de Projet et TER (Travail d'Étude et de Recherche) – Master 1 Économétrie & Statistiques  
> **Institution :** ISFA – Institut de Science Financière et d'Assurances, Lyon 1  
> **Superviseur :** Pr. Ying JIAO  
> **Auteurs :** Abdoul Barik ASSOUMANA · Manuel J. CEREZO · Anthony COTTET  
> **Année :** 2024–2025

---

## 🎯 Problématique

Les rendements financiers sont-ils entièrement expliqués par des facteurs systématiques observables ? Ce projet explore l'application de l'**Analyse Factorielle Exploratoire (EFA)** à la construction d'un **modèle statistique de facteurs latents** pour les actions du S&P 500, dans l'objectif de développer une stratégie d'investissement basée sur l'**alpha orthogonal** — la composante de rendement non capturée par les facteurs communs.

**Données :** Excès de log-rendements journaliers de **352 actions du S&P 500** sur la période **2000–2025**, collectés via l'API Yahoo Finance.

---

## 📁 Structure du projet

```
├── Rapport_ISFA_TER_Statistical_Factor_Model.pdf
└── README.md
```

---

## ⚙️ Méthodologie

### 1. Pré-traitement des données
- Téléchargement des prix ajustés via **Yahoo Finance API** (`yfinance`)
- Calcul des **excès de log-rendements** : soustraction du taux sans risque US 10Y (FRED)
- Winsorisation colonne par colonne (quantiles 5%–95%) pour la robustesse aux outliers
- Standardisation (z-scores) avant estimation factorielle
- Exclusion de 151 actions avec historique incomplet → **352 actions retenues**

### 2. Diagnostic de suitabilité
Deux tests statistiques valident l'adéquation des données à l'analyse factorielle :

| Test | Valeur |
|---|---|
| **Kaiser-Meyer-Olkin (KMO)** | **0.9936** (excellent) |
| **Bartlett's Test χ²** | 1 202 106.69 |
| **Bartlett's Test p-value** | < 0.000001 |

### 3. Sélection du nombre de facteurs latents
Trois critères appliqués sur les valeurs propres de la matrice de covariance :

| Méthode | Nombre de facteurs |
|---|---|
| **Threshold Method** (Random Matrix Theory) | **25** ✅ retenu |
| Maximum Change Point Method | 114 |
| Penalty-Based Method (Bai & Ng, 2002) | 342 |

Choix retenu : **25 facteurs** — meilleur compromis parcimonie / variance expliquée.

### 4. Estimation : Principal Axis Factoring (PAF)

Estimé via `statsmodels.multivariate.factor.Factor` (method=`'pa'`) :

| Facteur | Variance expliquée (%) | Cumulée (%) |
|---|---|---|
| F1 (Marché) | 32.15 | 32.15 |
| F2 | 3.81 | 35.95 |
| F3 | 2.73 | 38.69 |
| ... | ... | ... |
| **Total 25 facteurs** | — | **51.17%** |

> La variance expliquée de 51% est intentionnelle : le modèle factoriel isole la variation systématique commune de la variance idiosyncratique — ce n'est pas une limitation mais une propriété fondamentale du modèle (vs. PCA).

**Qualité d'estimation :**
- Matrice d'unicité Ψ diagonale ✅
- Mean Absolute Residual Correlation : **0.0081** ✅

### 5. Rotation Varimax
Rotation orthogonale appliquée via `factor_analyzer.Rotator` — maximise la variance des loadings carrés pour améliorer l'interprétabilité économique. Résultat : 25 facteurs associés à des **groupements sectoriels clairs** :

| Facteur | Secteur identifié |
|---|---|
| F1 | Industrie & Manufacturing (PH, CAT, ITW) |
| F2 | Utilities (ED, SO, AEP) |
| F4 | Énergie & Pétrole (APA, DVN, COP) |
| F5 | Technologie & Semi-conducteurs (AMAT, KLAC, TXN) |
| F6 | Banque & Services financiers (BAC, WFC, RF) |
| F7 | Biotechnologie & Pharma (BIIB, GILD, VRTX) |
| F11 | Assurance (CB, TRV, WRB, AON) |
| ... | 18 autres secteurs identifiés |

### 6. Estimation de l'Alpha
Régression OLS par actif sur les scores factoriels estimés :

```
x_{i,t} = α_i + Λ_i · f_t + ε_{i,t}
```

**Alpha orthogonal** isolé par projection sur le complément orthogonal de l'espace factoriel :

```
α⊥ = α - Λ(Λ'Λ)⁻¹Λ'α
```

**Top 10 actions par alpha orthogonal :**

| Ticker | Alpha | Description |
|---|---|---|
| URI | 0.1361 | United Rentals – Location de matériel |
| TXT | 0.1207 | Textron – Aérospatiale |
| CMI | 0.0932 | Cummins – Moteurs industriels |
| FCX | 0.0714 | Freeport-McMoRan – Cuivre & Or |
| ODFL | 0.0694 | Old Dominion – Logistique |

---

## 📊 Résultats : Backtest des stratégies (2019–2024)

### Portefeuille Long-Only (Top 10 alpha)

| Métrique | Portefeuille | S&P 500 |
|---|---|---|
| Rendement annuel moyen | **23.02%** | 15.09% |
| Volatilité annuelle | 29.40% | 20.17% |
| **Sharpe Ratio** | **0.78** | **0.75** |
| Beta CAPM | 1.20 | — |

✅ **Surperformance out-of-sample** — stable par rapport à l'in-sample (Sharpe 0.69 vs 0.25)

### Portefeuille Long-Short

| Métrique | Portefeuille | S&P 500 |
|---|---|---|
| Rendement annuel moyen | 2.13% | 15.09% |
| Sharpe Ratio | 0.13 | 0.75 |
| Beta CAPM | 0.20 | — |

⚠️ Sous-performance — mais beta très faible (0.20) suggère une présence d'alpha idiosyncratique à explorer dans une stratégie market-neutral.

---

## 💡 Conclusions clés

1. **EFA identifie des structures sectorielles cohérentes** dans les rendements S&P 500 sans supervision
2. **L'alpha orthogonal est un signal robuste** pour la construction de portefeuilles long-only
3. **La variance expliquée de 51% est suffisante** dans le cadre factoriel — la partition ΛΛ' + Ψ est l'objectif, pas la maximisation de variance totale
4. **Limites identifiées** : absence de coûts de transaction, loadings statiques, benchmark du long-short à approfondir

---

## 🛠️ Stack technique

| Catégorie | Outils |
|---|---|
| **Modélisation factorielle** | `statsmodels` (PAF), `factor_analyzer` (Varimax) |
| **Alpha & Backtest** | `statsmodels.api` (OLS), `numpy` |
| **Données** | `yfinance`, `pandas_datareader` (FRED) |
| **Data** | `pandas`, `numpy`, `scikit-learn` |
| **Visualisation** | `matplotlib`, `seaborn`, `plotly` |

---

## 📚 Références clés

- Paleologo, G.A. (2025). *The Elements of Quantitative Investing*
- Bai, J. & Li, K. (2012). Statistical analysis of factor models of high dimension
- Fama, E.F. & French, K.R. (1993). Common risk factors in stock and bond returns
- Ledoit, O. & Wolf, M. (2004). Well-conditioned estimator for large covariance matrices

---

## 📌 Contexte académique

Ce TER s'inscrit dans le Master 1 Économétrie & Statistiques de l'ISFA et constitue une application rigoureuse de la **finance quantitative** et de l'**économétrie financière** — deux axes centraux du Master MOSEF — à une problématique réelle de gestion de portefeuille : l'identification de rendements anormaux via des modèles statistiques à facteurs latents.
