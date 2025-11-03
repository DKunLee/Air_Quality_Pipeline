# Air Quality Monitoring Dashboard

An end-to-end data engineering project that builds an ETL pipeline and interactive dashboard for air quality analysis. The pipeline extracts data from AWS S3, transforms it with SQL, and visualizes it through a web application.

---

![Map Page](README_images/MapPage.png)
![Detail Graph Page](README_images/DetailGraphPage.png)

---

## Technical Skills Demonstrated

- **ETL Pipeline Development**: Automated data extraction, transformation, and loading from cloud storage (AWS S3)
- **SQL & Database Design**: Schema design, query optimization, and view creation with DuckDB
- **Python Programming**: Object-oriented design, error handling, command-line interfaces
- **Data Visualization**: Interactive dashboards with Plotly Dash and callbacks
- **Template Engineering**: Dynamic SQL generation using Jinja2
- **Version Control**: Git workflow and project organization

---

## Technology Stack

| Category | Technologies |
|----------|-------------|
| **Language** | Python 3.13 |
| **Database** | DuckDB (analytical SQL database) |
| **Web Framework** | Dash, Flask |
| **Data Processing** | Pandas, SQL |
| **Visualization** | Plotly Express |
| **Cloud** | AWS S3 |
| **Template Engine** | Jinja2 |
| **Data Source** | OpenAQ Public Dataset |

---

## Project Architecture

```
┌─────────────────┐
│   AWS S3        │  Raw air quality data (CSV.GZ files)
│   OpenAQ Data   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  extraction.py  │  Python + Jinja2 templates
│                 │  - Generate dynamic file paths
│                 │  - Batch data extraction
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│    DuckDB       │  In-process analytical database
│  raw.air_quality│  - 12 columns, millions of rows
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│transformation.py│  SQL-based transformations
│                 │  - Data cleaning & deduplication
│                 │  - Aggregations & views
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Presentation    │  3 optimized views
│     Views       │  - Latest values per location
│                 │  - Daily statistics
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Dashboard     │  Dash web application
│    (app.py)     │  - Interactive map
│                 │  - Time-series plots
│                 │  - Statistical analysis
└─────────────────┘
```

---

## Key Features

- **Data Pipeline**: Extracts 12 months of data from 24 locations (~millions of records)
- **Interactive Map**: Mapbox visualization showing sensor locations with latest readings
- **Time-Series Analysis**: Line charts showing air quality trends over time
- **Statistical Insights**: Box plots showing distribution patterns by weekday
- **Dynamic Filtering**: Location, parameter, and date range selection

---

## Project Structure

```
Air_Quality_Pipeline/
├── pipeline/
│   ├── extraction.py              # Extract data from S3 (Jinja2, argparse)
│   ├── transformation.py          # SQL transformations orchestration
│   └── database_manager.py        # Database utilities and CLI
├── sql/
│   ├── ddl/                       # Schema definitions
│   │   ├── 0_schemas.sql          # raw & presentation schemas
│   │   └── 1_raw_air_quality.sql  # Table creation
│   └── dml/
│       ├── raw/                   # Data insertion templates
│       └── presentation/          # Analytical views (3 SQL files)
├── dashboard/
│   └── app.py                     # Dash application with callbacks
├── notebooks/                     # Data exploration (Jupyter)
│   ├── api_exploration.ipynb
│   ├── data_quality_check.ipynb
│   └── s3_exploration.ipynb
├── locations.json                 # 24 sensor locations config
├── requirements.txt               # Python dependencies
└── README.md
```

---

## Quick Start

### 1. Install Dependencies
```bash
pip install -r requirements.txt
```

### 2. Create Database
```bash
python3 pipeline/database_manager.py \
    --create \
    --database-path ./air_quality.db \
    --ddl-query-parent-dir ./sql/ddl
```

### 3. Extract Data
```bash
python3 pipeline/extraction.py \
    --locations_file_path ./locations.json \
    --start_date 2024-01 \
    --end_date 2024-12 \
    --database_path ./air_quality.db \
    --extract_query_template_path ./sql/dml/raw/0_raw_air_quality_insert.sql \
    --source_base_path s3://openaq-data-archive/records/csv.gz
```

### 4. Transform Data
```bash
python3 pipeline/transformation.py \
    --database_path ./air_quality.db \
    --query_directory ./sql/dml/presentation
```

### 5. Launch Dashboard
```bash
python3 dashboard/app.py
```
Open browser to: http://127.0.0.1:8050

---

## Technical Implementation Details

### ETL Pipeline Design

**Extraction** ([extraction.py](pipeline/extraction.py))
- Uses Jinja2 templates to generate dynamic S3 file paths
- Implements batch processing for multiple locations and date ranges
- Error handling for missing data files
- Command-line interface with argparse

**Transformation** ([transformation.py](pipeline/transformation.py))
- Executes SQL scripts in sorted order
- Creates materialized views for query performance
- Implements two-layer architecture (raw → presentation)

**Loading** ([database_manager.py](pipeline/database_manager.py))
- DuckDB connection management
- Schema initialization from SQL files
- Database lifecycle management (create/destroy)

### Database Schema

**Raw Layer**: `raw.air_quality`
```sql
CREATE TABLE raw.air_quality (
    location_id BIGINT,
    sensors_id BIGINT,
    location VARCHAR,
    datetime TIMESTAMP,
    lat DOUBLE,
    lon DOUBLE,
    parameter VARCHAR,      -- PM10, PM2.5, SO2, etc.
    units VARCHAR,
    value DOUBLE,
    month VARCHAR,
    year BIGINT,
    ingestion_datetime TIMESTAMP
);
```

**Presentation Layer**: 3 optimized views
1. `presentation.air_quality` - Cleaned and deduplicated data
2. `presentation.latest_param_values_per_location` - Map visualization
3. `presentation.daily_air_quality_stats` - Aggregated statistics

### Dashboard Implementation

**Technologies**: Dash (React.js wrapper), Plotly, Pandas

**Interactive Components**:
- Callbacks for real-time filtering
- Mapbox integration for geographic visualization
- Dynamic dropdown population from database
- Date range picker for temporal filtering

**Code Example** ([app.py](dashboard/app.py)):
```python
@app.callback(
    [Output("line-plot", "figure"), Output("box-plot", "figure")],
    [Input("location-dropdown", "value"),
     Input("parameter-dropdown", "value"),
     Input("date-picker-range", "start_date"),
     Input("date-picker-range", "end_date")]
)
def update_plots(location, parameter, start_date, end_date):
    # Query database and filter data
    # Generate visualizations
    return line_fig, box_fig
```

---

## Data Source

**Dataset**: OpenAQ Air Quality Data
- **Source**: AWS S3 Public Dataset (`s3://openaq-data-archive`)
- **Format**: Compressed CSV files (`.csv.gz`)
- **Coverage**: 24 sensor locations in Utah
- **Parameters**: PM10, PM2.5, SO2, and other air quality indicators
- **Time Period**: 2024 (12 months)

---

## Monitored Locations (24 Sensors)

Utah air quality monitoring stations including:
- Salt Lake City, Rose Park, West Valley City
- North Provo, Lindon, Heber
- Tooele, Erda, Herriman
- And 15 additional locations

See [locations.json](locations.json) for complete list.

---

## Key Learnings

1. **Data Engineering**: Built production-style ETL pipeline with error handling
2. **SQL Optimization**: Designed two-layer schema for query performance
3. **Cloud Integration**: Worked with AWS S3 for large-scale data extraction
4. **Web Development**: Created interactive dashboards with callbacks and state management
5. **Python Best Practices**: CLI tools, type hints, logging, modular design

---

## Future Enhancements

- Add real-time data updates with scheduled jobs (Apache Airflow)
- Implement data quality monitoring and alerting
- Deploy dashboard to cloud (AWS EC2 or Heroku)
- Add machine learning for air quality prediction
- Extend to multiple geographic regions

---

## Acknowledgments

- **Data**: [OpenAQ](https://openaq.org/) - Open Air Quality Data
- **Tutorial**: [TrentDoesMath](https://www.youtube.com/watch?v=3gZickVbFfw&list=PLjWBnQvWCMLqtgQKraBXSuPBuXlxPuVdG) - ETL Pipeline Design Patterns

---

## License

MIT License - See [LICENSE](LICENSE) file for details.

---

**Project Type**: Data Engineering Portfolio Project
**Completion Date**: December 2024
**Contact**: Open an issue for questions or feedback
