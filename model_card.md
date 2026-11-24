# Sentiment Model

## Use case
Classification binaire de sentiment.

## Version
Version 1 (MLflow Run ID : `3b59642d399c4f31a0d38ed310b3caa1`).

## Dataset
`data/reviews.csv` — jeu pedagogique (20 avis clients en francais, 10 positifs / 10 negatifs, decoupage 75% entrainement / 25% test avec `random_state=42`).

## Method
TF-IDF + Logistic Regression (scikit-learn `Pipeline`).

## Parameters
- `C`: 1.0
- `max_features`: 5000
- `ngram_range`: [1, 2]
- `max_iter`: 1000

## Metrics
- Accuracy : 0.400
- F1 : 0.571

## Quality Gates
- Unit tests : Passed (3/3, couverture de code `src/` : 100% >= 70%)
- Data tests : Passed (4/4 : aucune valeur manquante, labels binaires, textes non vides, equilibre des classes respecte)
- Model tests : Failed sur `test_model_quality_gate` (`accuracy = 0.400 < 0.60`, bien que `f1 = 0.571 >= 0.50`) ; Passed sur `test_model_can_predict`
- Security checks : `bandit -r src` Passed (0 vulnerabilite sur 55 lignes) ; `pip-audit` Failed en local (83 vulnerabilites detectees dans l'environnement virtuel partage)

## Known limitations
- Petit jeu de donnees pedagogique (20 exemples, dont seulement 5 en test).
- Langue principalement francaise.
- Evaluation non representative d'un contexte industriel : avec 15 exemples en entrainement (8 positifs contre 7 negatifs) et `C=1.0`, la regression logistique predit la classe majoritaire sur le jeu de test.

## Governance
- `validation_status`: passed (taggue dans le script pedagogique `train_and_register.py` afin de tester le workflow de promotion)
- `approval_status`: approved (via `python promote.py --approve`)
- `alias`: candidate / champion

## Deployment decision
- **Decision industrielle (Quality Gates)** : **REJETEE** — le modele echoue a la Gate G3 (`accuracy = 0.400 < 0.60`) et a la Gate G6 (`pip-audit`), ce qui bloque automatiquement le pipeline CI/CD avant Staging et Production.
- **Simulation pedagogique locale** : Version 1 enregistree dans MLflow Registry sous l'alias `candidate`, puis promue en `champion` uniquement apres approbation explicite (`--approve`) pour valider le mecanisme de gouvernance.
- **Auditabilite** : decision reliee a la chaine complete Code (commit Git) + Data (`data/reviews.csv`) + Parametres (`params.yaml`) + Modele (`sentiment-model` v1 dans MLflow) + Tests (`pytest` / `ruff` / `bandit` / `pip-audit`) + Approbation (`--approve` / GitHub Environment `production`) + Deploiement (workflow `.github/workflows/mlops-pipeline.yml`).