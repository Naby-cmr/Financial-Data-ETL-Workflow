# Financial Data ETL Pipeline


Dans ce projet, je construit un petit ETL qui récupère des données financière de l’API **AlphaVantage**.  
Le projet suit les principes du livre Financial Data Engineering de Tamer Khraisha: découpage clair du workflow, nettoyage progressif, agrégations journalières, et stockage dans une base SQL.

Le pipeline tourne entièrement dans **Docker** (Mage, Postgres, pgAdmin) via :

```bash
docker compose up -d
```

---

## Structure du pipeline

### Data Loaders

- **fetch_intraday_data.py** : récupère les données intraday pour plusieurs tickers, les combine, ajoute l’horodatage d’ingestion, et renomme les colonnes clés.

### Transformers

Les transformations sont découpées en plusieurs blocs Mage :

- **select_columns_for_deduplication.py** → sélection des colonnes utiles pour la déduplication  
- **drop_duplicates.py** → suppression des doublons basés sur `price_date` + `symbol`  
- **compute_daily_aggregates.py** → calcul des moyennes journalières (`daily_open`, `daily_close`, etc.)  
- **select_columns_for_aggregation.py** → sélection finale avant export

### Data Exporters

- **export_intraday_data.py** : export vers PostgreSQL via `io_config.yaml`, dans la table **intraday_adjusted**

---

## SQL

Le dossier `sql/` contient les scripts pour initialiser les tables :

```
sql/
│
├── create_intraday_table.sql
└── create_dailyadjusted_table.sql
```

Ces scripts permettent de créer les tables nécessaires dans PostgreSQL avant d'exécuter le pipeline.

---

## Docker

Tout tourne via Docker :

```bash
docker compose up -d
```

Le fichier **docker-compose.yaml** lance :

- **mage** → exécution des blocs ETL + interface web  
- **postgres** → stockage des données intraday + agrégations  
- **pgadmin** → interface SQL (http://localhost:8080)

Les variables (user, password, API key…) sont dans `.env`.

---

##️ Exécution du pipeline

### 1. Lancer les services Docker
```bash
docker compose up -d
```

### 2. Ouvrir Mage
http://localhost:6789

### 3. Exécuter les blocs dans cet ordre

1. `fetch_intraday_data`  
2. `select_columns_for_deduplication`  
3. `drop_duplicates`  
4. `compute_daily_aggregates`  
5. `select_columns_for_aggregation`  
6. `export_intraday_data`

### 4. Vérifier les résultats  
Les données sont visibles dans PostgreSQL (`intraday_adjusted`) via pgAdmin.

---

## Objectif du projet

Mettre en pratique :

- un workflow ETL réaliste et découpé  
- l’utilisation de Mage.ai  
- la modélisation SQL  
- le traitement de données financières  
- une architecture entièrement containerisée  

---

## Auteur  
**Naby Camara**  
Data Engineering • Finance • Python • SQL
