# Rapport de Nettoyage des Données : `bat_surveys.csv`

**Fichier cible** : [`data/ecology/bat_surveys.csv`](file:///c:/Users/hi/code/data_mining/20262027-UE-DataMining-Project/data/ecology/bat_surveys.csv)  
**Script automatisé** : [`mining_scripts/clean_bat_surveys.py`](file:///c:/Users/hi/code/data_mining/20262027-UE-DataMining-Project/mining_scripts/clean_bat_surveys.py)  
**Date d'exécution** : 15 Septembre 2026  

---

## 1. Vue d'ensemble et métriques

| Métrique | Avant nettoyage | Après nettoyage | Différence / Action |
| :--- | :--- | :--- | :--- |
| **Nombre de lignes** | 4 994 | 4 992 | -2 doublons stricts supprimés |
| **Nombre de colonnes** | 14 | 14 | Inchangé |
| **Valeurs nulles totales** | 20 | 0 | Toutes imputées avec certitude métier |
| **Conformité ISO 8601 (dates)** | 4 984 / 4 994 (10 formats disparates) | 4 992 / 4 992 (100%) | Harmonisation complète |
| **Typage numérique strict** | Variables polluées en `object` | `int64` / `float64` stricts | Nettoyage unités et séparateurs |

---

## 2. Problèmes identifiés et corrections appliquées

### A. Dédoublonnage strict
- **Constat** : 2 lignes étaient exactement dupliquées à la fin du fichier (`BAT-0003183` et `BAT-0003567`).
- **Correction** : Suppression des 2 doublons redondants, ramenant le dataset à 4 992 observations uniques.

---

### B. Harmonisation et imputation de `observer_team`
- **Constat** : 20 entrées présentaient des valeurs manquantes (`NaN`) dans la colonne `observer_team`.
- **Analyse approfondie** : Chaque `site_code` est attribué de manière fixe et unique à une seule équipe d'observateurs (relation bijective stricte sur les 64 sites).
- **Correction** : Imputation déterministe à 100% de fiabilité à partir de la cartographie `site_code` $\rightarrow$ `observer_team` (ex. `AVL-ARU` $\rightarrow$ `TEAM-12-3`). Aucune perte d'information ni biais statistique.

---

### C. Harmonisation de la casse (`survey_method`)
- **Constat** : Présence de valeurs avec casses hétérogènes (`FIXED_ACOUSTIC_STATION`, `Fixed_Acoustic_Station`, `ACOUSTIC_TRANSECT`, etc.).
- **Correction** : Conversion uniforme en minuscules avec suppression d'espaces (`strip().lower()`) :
  - `acoustic_and_visual`
  - `fixed_acoustic_station`
  - `acoustic_transect`

---

### D. Nettoyage des colonnes numériques polluées
- **Constat** :
  - Présence de suffixes textuels comme `" count"` dans les colonnes `observed_bat_count`, `acoustic_detection_count`, `nightly_visit_count`.
  - Séparateurs décimaux francophones à virgule (`,`) au lieu de points (`.`) dans `observation_hours` et `weather_interference`.
  - Conséquence : Ces colonnes étaient reconnues comme du texte (`object`), empêchant les calculs statistiques et le machine learning.
- **Correction** :
  - Suppression systématique du suffixe `" count"`.
  - Remplacement des virgules par des points (`replace(',', '.')`).
  - Typage strict en `int` pour les variables discrètes (`observed_bat_count`, `acoustic_detection_count`, `nightly_visit_count`, `comparison_species_count`).
  - Typage strict en `float` arrondi à 2 décimales pour les variables continues (`observation_hours`, `weather_interference`, `colony_estimate`).

---

### E. Standardisation des dates (`survey_start` et `survey_end`)
- **Constat** : 10 observations présentaient un format de date divergent dans `survey_end` :
  - Formats `DD/MM/YYYY HH:MM:SS` (ex. `12/01/2022 00:00:00`)
  - Formats `YYYY/MM/DD HH:MM:SS` (ex. `2020/07/12 00:00:00`)
  - Formats textuels anglophones (ex. `12-Apr-2019 00:00:00`)
- **Correction** :
  - Parsing robuste garantissant la cohérence avec la date de début (durée exacte de chaque mission de surveillance : 150 heures soit 6,25 jours).
  - Normalisation au format standard ISO 8601 : `YYYY-MM-DDTHH:MM:SS`.

---

## 3. Schéma final des données

| Colonne | Type initial | Type final | Nulls restants | Description |
| :--- | :--- | :--- | :---: | :--- |
| `survey_id` | `object` | `object` | 0 | Identifiant unique de l'inventaire |
| `site_code` | `object` | `object` | 0 | Code du site de suivi écologique |
| `survey_start` | `object` | `object` (ISO) | 0 | Horodatage de début de mission |
| `survey_end` | `object` | `object` (ISO) | 0 | Horodatage de fin de mission |
| `survey_method` | `object` | `object` (lowercased) | 0 | Méthode d'échantillonnage standardisée |
| `observation_hours` | `object` | `float64` | 0 | Heures d'observation effectives |
| `observed_bat_count` | `object` | `int64` | 0 | Nombre de chauves-souris observées |
| `acoustic_detection_count` | `object` | `int64` | 0 | Détections ultrasonores enregistrées |
| `nightly_visit_count` | `object` | `int64` | 0 | Nombre de passages nocturnes détectés |
| `colony_estimate` | `float64` | `float64` | 0 | Estimation de la taille de la colonie |
| `comparison_species_count`| `int64` | `int64` | 0 | Nombre d'espèces témoins répertoriées |
| `observer_team` | `object` (20 nuls) | `object` | 0 | Identifiant de l'équipe (100% complété) |
| `survey_quality` | `object` | `object` | 0 | Indice de qualité (`high`, `acceptable`, `limited`) |
| `weather_interference` | `object` | `float64` | 0 | % de perturbation météo (0 à 100) |
