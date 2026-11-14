# Produits structurés : coûts implicites, biais comportementaux et protection de l'investisseur

**Valorisation Monte-Carlo sous mesures Q et P de quatre produits structurés (Autocall Phoenix Worst-of), extraction des coûts implicites, et preuve de concept RegTech-IA par simulation multi-agents, dans le cadre réglementaire PRIIPs et AMMC.**

Projet académique de groupe (Groupe n° 09), Master en Finance d'Entreprise et Computationnelle, semestre S9 – Recherche appliquée en ingénierie financière et réglementation.

---

## Problématique

> Dans quelle mesure le couplage entre un moteur de valorisation stochastique Monte-Carlo (Q/P) et un dispositif RegTech d'audit précontractuel automatisé permet-il de quantifier l'érosion de valeur liée à la complexité, d'atténuer les biais cognitifs au-delà d'une simple synthèse humaine et de sécuriser la décision d'adéquation client dans le contexte réglementaire marocain ?

Un épargnant qui souscrit au pair un produit structuré achète une obligation risquée et vend à l'émetteur des options exotiques : la juste valeur du produit est donc inférieure à 100 % du nominal, et l'écart (µ0 = 100 % − FV0) correspond aux frais implicites.

## Contenu du dépôt

| Dossier | Contenu |
|---|---|
| `notebooks/` | Notebook Python : moteur Monte-Carlo, protocole à trois bras, économétrie, génération des figures |
| `report/` | Article scientifique complet (PDF) |
| `figures/` | Figures et tableau LaTeX générés par le notebook |

## Partie 1 : moteur stochastique (résultat principal)

- **Dynamique** : mouvement brownien géométrique multivarié, corrélations par factorisation de Cholesky, horizon 5 ans, observations annuelles.
- **Deux mesures** : valorisation sous la mesure risque-neutre Q (dérive r) ; risques réels sous la mesure physique P (dérive µ).
- **Simulation** : 100 000 trajectoires par produit, graine fixée (`SEED_MC = 2026`) pour la reproductibilité.
- **Paramètres communs** : r = 3,00 %, q = 2,00 %, spread émetteur = 0,80 %, volatilité σ = 22 %, corrélation ρ = 0,40.
- **Panel de produits** construit de façon incrémentale :
  - **P1** : capital garanti + call spread
  - **P2** : autocall snowball mono-indice
  - **P3** : autocall Phoenix à mémoire mono-indice
  - **P4** : autocall Phoenix à mémoire Worst-of sur 3 actions
- **Indice ISCO** : indice synthétique de complexité et d'opacité, avec une version équipondérée pour tester la robustesse du classement.

| Produit | FV0 (%) | Frais implicites µ0 (%) | Prob. de perte en capital (%) | Perte conditionnelle moyenne (%) | ISCO (/10) |
|---|---|---|---|---|---|
| P1 | 98,48 | 1,52 | 0,0 | – | 2,09 |
| P2 | 97,36 | 2,64 | 7,6 | −52,7 | 3,73 |
| P3 | 95,89 | 4,11 | 7,6 | −52,7 | 5,32 |
| P4 | 95,27 | 4,73 | 20,6 | −53,2 | 9,11 |

**Lecture** : à paramètres de marché identiques, l'empilement de clauses (P1 → P4) réduit la juste valeur et fait passer les frais implicites de 1,52 % à 4,73 %. Le passage d'un mono-indice (P3) à un panier Worst-of de 3 actions (P4) fait presque tripler la probabilité de perte en capital (7,6 % → 20,6 %), ce qui illustre l'effet falaise de la barrière à 60 %.

## Partie 2 : preuve de concept RegTech-IA (simulation in-silico)

> **Important : cette partie est une simulation.** Les 180 « investisseurs » sont des agents synthétiques générés par le code. Les effets des trois dispositifs d'information sont des paramètres de la simulation, et non des mesures obtenues auprès de vrais épargnants. Les résultats ci-dessous illustrent le fonctionnement du protocole ; ils ne constituent pas une preuve empirique sur des investisseurs réels.

Trois bras de 60 agents : brochure standard (A), fiche synthétique humaine (B1), fiche d'audit générée par le pipeline Monte-Carlo (B2). Estimations par MCO (erreurs-types robustes HC3) et régression Logit, avec validation croisée à 5 plis.

| Indicateur (agents simulés) | Bras A | Bras B1 | Bras B2 |
|---|---|---|---|
| Score de compréhension du produit P4 (/100) | 34,3 | 57,0 | 86,3 |
| Décision d'adéquation correcte | 16,7 % | 50,0 % | 91,7 % |

Le modèle Logit complet atteint une AUC de 0,946 en validation croisée (0,874 sans les variables de biais).

## Cadre réglementaire

Comparaison du règlement européen PRIIPs (document d'informations clés, indicateur synthétique de risque, scénarios de performance) avec l'architecture marocaine : OPCVM à formules (AMMC), titres de créance structurés par appel public à l'épargne (Loi 44-12, circulaire AMMC 03/19), conseillers en investissement financier (circulaire AMMC 20/01) et dépôts structurés (Bank Al-Maghrib). Trois recommandations opérationnelles sont formulées dans le rapport.

## Limites

- Volatilité et corrélation constantes (modèle de Black-Scholes multivarié) ; extensions possibles : Heston multivarié, copules de Student.
- Calibration sur des paramètres standards, et non sur des valeurs du MASI.
- Protocole comportemental simulé : à valider avec une cohorte réelle d'épargnants et de conseillers.
- Variables de biais mesurées après l'exposition au document, sans mesure préalable.

## Lancer le projet

```bash
git clone https://github.com/<votre-nom-utilisateur>/structured-products-regtech.git
cd structured-products-regtech
python -m venv .venv
source .venv/bin/activate      # Windows : .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook notebooks/Module_1_Stochastique_Pur.ipynb
```

Exécutez les cellules **dans l'ordre** : la dernière cellule (figures) réutilise le moteur et les données créés par les deux premières.

## Outils

Python, NumPy, pandas, SciPy, statsmodels, scikit-learn, matplotlib, seaborn.

## Références principales

Black & Scholes (1973) ; Stoimenov & Wilkens (2005) ; Henderson & Pearson (2011) ; Célérier & Vallée (2017) ; Vokata (2021) ; Kahneman & Tversky (1979) ; Bordalo, Gennaioli & Shleifer (2012) ; Règlement (UE) n° 1286/2014 (PRIIPs) ; Circulaires AMMC n° 03/19 et n° 20/01. La liste complète figure dans le rapport.
