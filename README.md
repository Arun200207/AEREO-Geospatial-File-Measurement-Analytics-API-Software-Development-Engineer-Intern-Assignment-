# AEREO Geospatial File Measurement & Analytics API

> **AEREO Cloud Internship Program — Software Development Engineer Intern Assignment**
>
> A high-performance, robust backend API for uploading, validating, measuring, persisting, and analyzing geospatial vector files, including ZIP Shapefiles and KML documents.

---

## Table of Contents

- [Project Overview](#-project-overview)
- [Developed Modules](#-developed-modules)
- [Technology Stack](#-technology-stack)
- [System Architecture](#-system-architecture)
- [File-Processing Flow](#-file-processing-flow)
- [Measurement Calculation and CRS Handling](#-measurement-calculation-and-crs-handling)
- [Database and Persistence](#-database-and-persistence)
- [Analytics and Anomaly Detection](#-analytics-and-anomaly-detection)
- [API Endpoints](#-api-endpoints)
- [Installation and Local Setup](#️-installation-and-local-setup)
- [API Usage Examples](#-api-usage-examples)
- [Security and Data Validation](#-security-and-data-validation)
- [Design Decisions and Trade-offs](#-design-decisions-and-trade-offs)
- [Testing and Quality Assurance](#-testing-and-quality-assurance)
- [Key Learnings](#-key-learnings)
- [Future Scope](#-future-scope)

---

## 🚀 Project Overview

The **AEREO Geospatial File Measurement & Analytics API** automates the ingestion, validation, extraction, measurement, storage, and statistical analysis of geospatial vector datasets.

Built using **FastAPI, GeoPandas, Shapely, PyProj, Pandas, NumPy, and SQLite**, the application processes geographic data, projects geometries into suitable local coordinate reference systems (CRS), calculates metric areas and lengths, stores feature-level measurements in a relational database, and identifies statistical anomalies.

The API supports:

- Uploading ZIP archives containing Shapefiles.
- Uploading KML geographic data files.
- Validating file types, sizes, and structural integrity.
- Extracting geospatial features and their attributes.
- Sanitizing feature properties for JSON serialization.
- Evaluating geometry validity and emptiness.
- Calculating polygon areas and line lengths in metric units.
- Automatically selecting an appropriate local UTM projection.
- Persisting file metadata, feature attributes, and measurements in SQLite.
- Generating statistical profiles and detecting outliers using the Interquartile Range (IQR) method.
- Exposing interactive API documentation through Swagger UI and ReDoc.

The application is designed with security, reliability, maintainability, and extensibility in mind.

## 📦 Developed Modules

The project is organized into seven modules.

### Module 01: API Request Handling and System Foundation

- Initialize the FastAPI application.
- Register API routers and shared dependencies.
- Configure application startup and shutdown behavior.
- Provide a health-check endpoint.
- Establish consistent request and response handling.
- Integrate interactive API documentation.

### Module 02: Secure File Ingestion and Format Validation

- Accept ZIP Shapefile archives and KML documents.
- Sanitize uploaded filenames to prevent path traversal.
- Enforce a maximum upload size of 50 MB.
- Validate file extensions and expected file structures.
- Check ZIP entry counts, compression ratios, and archive integrity.
- Reject unsafe or malformed uploads.
- Store uploaded files in isolated storage directories.

### Module 03: Geospatial Feature Extraction and JSON-Safe Serialization

- Extract Shapefile archives into controlled temporary directories.
- Load supported vector datasets using GeoPandas.
- Parse KML documents into geospatial feature collections.
- Extract feature geometries and their associated attributes.
- Convert Pandas and NumPy values into JSON-compatible primitives.
- Handle missing values, dates, non-finite numbers, and unsupported attribute types.
- Evaluate geometry health using validity and emptiness checks.
- Handle multipart geometries and unsupported geometry types safely.

### Module 04: Measurement and Local UTM CRS Engine

- Determine a suitable projected CRS for each feature.
- Transform geographic coordinates into an appropriate local UTM projection.
- Calculate polygon and multipolygon areas in square meters.
- Convert polygon areas into hectares.
- Calculate line and multiline lengths in meters.
- Identify points and unsupported geometries without interrupting batch processing.
- Return structured measurement results and per-feature error information.

### Module 05: Persistence and Relational Data Access

- Use SQLite for lightweight relational persistence.
- Store uploaded file metadata.
- Persist extracted feature attributes and geometry metadata.
- Store feature-level measurement results.
- Maintain relationships between uploaded files and their features.
- Enforce foreign-key constraints and cascade deletes where appropriate.
- Provide database access functions for retrieving stored information.

### Module 06: Analytics Engine and IQR-Based Statistical Profiling

- Calculate descriptive statistics for supported measurements.
- Generate statistical profiles for area and length values.
- Calculate quartiles, medians, and interquartile ranges.
- Identify potential outliers using the 1.5 × IQR rule.
- Report feature-level anomalies and associated measurements.
- Summarize data quality and geometry validity.
- Handle empty datasets and insufficient observations safely.

### Module 07: Comprehensive Integration and Automated QA Test Suite

- Test API endpoints and request validation.
- Verify upload and file-format validation behavior.
- Test valid and invalid Shapefile and KML inputs.
- Validate measurement calculations and CRS transformations.
- Verify database persistence and retrieval.
- Test statistical profiling and anomaly detection.
- Cover malformed files, edge cases, and security-related scenarios.
- Validate end-to-end application workflows.

---

## 🛠️ Technology Stack

| Technology | Purpose |
|---|---|
| Python 3.10+ | Core programming language |
| FastAPI | REST API framework |
| Uvicorn | ASGI server |
| GeoPandas | Geospatial dataset loading and processing |
| Shapely | Geometry operations and validation |
| PyProj | CRS transformations and projection handling |
| Pandas | Tabular data processing and descriptive statistics |
| NumPy | Numerical operations and statistical calculations |
| SQLite | Relational data persistence |
| GDAL / Fiona / Pyogrio | Geospatial file I/O support, depending on the installed GeoPandas environment |
| HTTPX | HTTP client for API integration tests |
| Pytest | Automated testing framework |

---

## 🏗️ System Architecture

### Application Structure

```text
geospatial-measurement-api/
├── app/
│   ├── api/
│   │   ├── routes.py
│   │   ├── measurement_routes.py
│   │   ├── file_details_routes.py
│   │   └── analytics_routes.py
│   │
│   ├── services/
│   │   ├── ingestion.py
│   │   ├── feature_extraction.py
│   │   ├── measurement.py
│   │   ├── database.py
│   │   └── analytics.py
│   │
│   └── main.py
│
├── data/
│   ├── database/
│   │   └── .gitkeep
│   ├── uploads/
│   │   └── .gitkeep
│   └── extracted/
│       └── .gitkeep
│
├── tests/
│   ├── test_health.py
│   ├── test_ingestion.py
│   ├── test_measurements.py
│   ├── test_file_details.py
│   ├── test_analytics.py
│   └── test_integration.py
│
├── .gitignore
├── requirements.txt
└── README.md
```

> The directory and test filenames above describe the intended organization. Actual files and test coverage depend on the implementation.

### Component Responsibilities

**API Layer — `app/api/`**

Defines HTTP endpoints, validates incoming requests, invokes application services, and returns structured JSON responses.

**Service Layer — `app/services/`**

Contains the core application logic for ingestion, extraction, measurement, database operations, and analytics.

**Application Entry Point — `app/main.py`**

Creates the FastAPI application, registers routers, configures shared behavior, and exposes the health-check endpoint.

**Data Layer — `data/`**

Separates uploaded source files, temporary extraction directories, and persistent database storage.

**Testing Layer — `tests/`**

Contains unit, integration, and end-to-end tests for validating application behavior.

### High-Level Data Flow

```text
Client
  |
  v
FastAPI Application
  |
  v
File Upload and Validation
  |
  v
Secure Storage / ZIP Extraction
  |
  v
Geospatial Feature Extraction
  |
  v
Geometry Validation and Sanitization
  |
  v
Local UTM Projection and Measurement
  |
  v
SQLite Persistence
  |
  +------------------------+
  |                        |
  v                        v
File Details API       Analytics Engine
  |                        |
  v                        v
Feature Measurements   Statistics and IQR Outliers
  |                        |
  +------------+-----------+
               |
               v
          JSON Responses
```

---

## 📂 File-Processing Flow

The file-processing pipeline consists of three primary stages, followed by measurement, persistence, and analytics.

### 1. Upload — `POST /api/files/`

The client uploads one of the supported input formats:

- `.zip` — A ZIP archive containing the required Shapefile components.
- `.kml` — A Keyhole Markup Language document containing geographic features.

A Shapefile normally requires at least the following components:

- `.shp` — Feature geometry.
- `.shx` — Geometry index.
- `.dbf` — Attribute table.

A `.prj` file is strongly recommended when the dataset uses a projected coordinate system or any CRS other than the assumed source CRS. Without reliable CRS metadata, accurate measurements may not be possible.

### 2. Ingestion and Sanitization — Module 02

Before processing, the ingestion service validates the uploaded file.

Validation includes:

- Filename sanitization to prevent path traversal attacks.
- File-extension and supported-format checks.
- A maximum upload size of 50 MB.
- ZIP archive integrity validation.
- Limits on archive entry counts and decompression ratios.
- Rejection of absolute paths and unsafe archive entries.
- Validation of required Shapefile components.
- XML structural validation for KML documents.
- Protection against unsafe XML entity processing.
- Controlled storage of accepted uploads.

Files that fail validation should be rejected with an appropriate HTTP status code and a descriptive error response.

### 3. Extraction and Feature Processing — Module 03

Validated files are loaded into GeoPandas GeoDataFrames.

The extraction service:

1. Reads geographic features and their attributes.
2. Identifies geometry types.
3. Checks whether geometries are empty or invalid.
4. Sanitizes feature attributes for JSON serialization.
5. Handles null values and non-finite numerical values.
6. Preserves feature-level processing results.
7. Reports unsupported geometries without failing the entire batch where possible.

### 4. Measurement — Module 04

Each supported feature is measured using a suitable projected coordinate reference system.

Polygon areas are returned in square meters and hectares. Line lengths are returned in meters. Point geometries are identified but do not receive area or length measurements.

### 5. Persistence — Module 05

File metadata, extracted feature information, and measurement results are stored in SQLite.

The system maintains the relationship between the original uploaded file and its associated feature records.

### 6. Analytics — Module 06

Persisted measurements are analyzed to calculate descriptive statistics, assess data quality, and identify potential outliers using the IQR method.

---

## 📐 Measurement Calculation and CRS Handling

Geographic coordinates are commonly represented using latitude and longitude in degrees. These coordinates should not be used directly to calculate metric lengths or areas.

The measurement engine uses projected coordinates expressed in meters.

### Local UTM Projection

The measurement workflow is:

1. Read the feature geometry and its source CRS.
2. Confirm that the geometry uses a known geographic or projected CRS.
3. Transform the geometry to geographic coordinates when necessary for UTM zone estimation.
4. Determine a representative geographic location for the feature.
5. Estimate the appropriate UTM zone and hemisphere.
6. Project the feature into the selected UTM CRS.
7. Calculate area or length using the projected geometry.
8. Return the measurement and associated CRS information.

UTM zones are typically 6 degrees of longitude wide, with separate northern and southern EPSG codes.

The engine aims to select a local projection for each feature rather than forcing one projected CRS across datasets covering multiple regions.

> **Accuracy consideration:** UTM is generally suitable for local measurements within its intended geographic extent. Features spanning multiple UTM zones, crossing the antimeridian, or located near the poles may require a different projection strategy. For high-accuracy applications, geodesic calculations or an explicitly selected regional CRS may be more appropriate.

### Measurement Rules

| Geometry Type | Measurement | Output Unit |
|---|---|---|
| Polygon | Area | Square meters (`m²`) |
| Polygon | Area conversion | Hectares (`ha`) |
| MultiPolygon | Combined area | Square meters and hectares |
| LineString | Length | Meters (`m`) |
| MultiLineString | Combined length | Meters (`m`) |
| Point | No area or length measurement | Not applicable |
| MultiPoint | No area or length measurement | Not applicable |
| GeometryCollection | Depends on supported components and implementation policy | Geometry-dependent |
| Unsupported geometry | Safely flagged | Not applicable |

### Area Conversion

One hectare equals 10,000 square meters.

```python
area_hectares = area_square_meters / 10_000
```

### Example Measurement Result

```json
{
  "feature_id": 1,
  "geometry_type": "Polygon",
  "is_valid": true,
  "is_empty": false,
  "area_sqm": 125000.0,
  "area_hectares": 12.5,
  "length_m": null,
  "measurement_crs": "EPSG:32643",
  "status": "success"
}
```

The example CRS is illustrative. The actual CRS must be determined from the feature's location and hemisphere.

For unsupported geometries or failed transformations, the API should return an explicit status and error description rather than silently reporting an incorrect measurement.

---

## 🗄️ Database and Persistence

### SQLite

SQLite is used to persist uploaded file metadata, feature information, measurements, and associated processing results.

The database layer is responsible for:

- Initializing the database and required tables.
- Managing database connections.
- Creating file and feature records.
- Storing measurement results.
- Retrieving file details and measurements.
- Supporting foreign-key relationships.
- Applying transactions and appropriate rollback behavior.
- Maintaining referential integrity.

### Suggested Relational Model

**Files**

Stores information about each uploaded dataset.

Possible fields:

- `id`
- `original_filename`
- `stored_filename`
- `file_type`
- `file_size`
- `upload_timestamp`
- `processing_status`
- `feature_count`

**Features**

Stores feature-level metadata and geometry information.

Possible fields:

- `id`
- `file_id`
- `feature_index`
- `geometry_type`
- `is_valid`
- `is_empty`
- `properties_json`

**Measurements**

Stores measurement results associated with each feature.

Possible fields:

- `id`
- `feature_id`
- `area_sqm`
- `area_hectares`
- `length_m`
- `measurement_crs`
- `measurement_status`
- `error_message`

These fields describe a suggested schema; the actual database structure depends on the implementation.

---

## 📊 Analytics and Anomaly Detection

The analytics engine profiles feature measurements and identifies values that differ substantially from the rest of the dataset.

### Statistical Profiling

For supported area or length measurements, the API can calculate:

- Feature count.
- Valid measurement count.
- Missing measurement count.
- Minimum and maximum values.
- Mean and median.
- First and third quartiles.
- Interquartile range.
- Standard deviation, where appropriate.
- Number and proportion of identified outliers.

Only comparable measurements should be analyzed together. For example, polygon areas and line lengths must not be combined into a single statistical distribution.

### IQR-Based Outlier Detection

The Interquartile Range (IQR) is calculated as:

\[
IQR = Q_3 - Q_1
\]

Where:

- \(Q_1\) is the first quartile, or 25th percentile.
- \(Q_3\) is the third quartile, or 75th percentile.

The lower and upper outlier fences are:

\[
L = Q_1 - 1.5 \times IQR
\]

\[
U = Q_3 + 1.5 \times IQR
\]

A measurement is flagged as a potential outlier when:

\[
x < L \quad \text{or} \quad x > U
\]

### Example Analytics Response

```json
{
  "file_id": 1,
  "analytics": {
    "area_sqm": {
      "count": 100,
      "mean": 1250.5,
      "median": 980.0,
      "q1": 450.0,
      "q3": 1600.0,
      "iqr": 1150.0,
      "lower_bound": -1275.0,
      "upper_bound": 3325.0,
      "outlier_count": 3
    }
  }
}
```

The values are illustrative and do not represent a real uploaded dataset.

### Interpretation

IQR-based outlier detection provides a robust baseline for identifying unusually large or small measurements without relying solely on the mean and standard deviation.

However, an outlier is not necessarily an error. A large land parcel or unusually long road segment may be entirely valid. Statistical flags should therefore be treated as indicators for further inspection rather than automatic data rejection.

For datasets with fewer than four observations, identical values, or an IQR of zero, the implementation should document and consistently apply its outlier-handling policy.

---

## 🔌 API Endpoints

The API exposes the following endpoints.

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/health` | Checks application health. |
| `POST` | `/api/files/` | Uploads and validates a ZIP Shapefile or KML file. |
| `GET` | `/api/files/{file_id}/measurements/` | Calculates or retrieves and persists feature measurements. |
| `GET` | `/api/files/{file_id}/` | Retrieves file metadata and stored feature measurements. |
| `GET` | `/api/files/{file_id}/analytics/` | Returns statistical profiles, data-quality information, and IQR anomaly results. |

The exact response formats, status codes, and persistence behavior should be documented by the implemented endpoint contracts.

### Health Check

```http
GET /health
```

Example response:

```json
{
  "status": "healthy"
}
```

### Upload a File

```http
POST /api/files/
Content-Type: multipart/form-data
```

Accepts a supported ZIP Shapefile archive or KML document through a multipart form field named `file`.

### Retrieve Measurements

```http
GET /api/files/1/measurements/
```

Calculates and persists measurements, or retrieves stored results, depending on the endpoint's implementation.

### Retrieve File Details

```http
GET /api/files/1/
```

Returns the stored file metadata and associated feature measurements.

### Retrieve Analytics

```http
GET /api/files/1/analytics/
```

Returns descriptive statistics, data-quality summaries, and potential outliers for the requested file.

---

## ⚙️ Installation and Local Setup

### Prerequisites

Install the following before running the application:

- Python 3.10 or later.
- pip.
- A compatible environment for GeoPandas and its geospatial dependencies.

GeoPandas installations may require compatible GDAL, GEOS, PROJ, Fiona, or Pyogrio components. Using a clean virtual environment and the official installation guidance for the chosen package manager can help avoid dependency conflicts.

### 1. Clone the Repository

Replace the placeholder repository URL with the actual repository URL.

```bash
git clone https://github.com/your-username/geospatial-measurement-api.git
cd geospatial-measurement-api
```

### 2. Create a Virtual Environment

**Windows:**

```bash
python -m venv .venv
.venv\Scripts\activate
```

**Linux / macOS:**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install Dependencies

Install the Python packages:

```bash
python -m pip install --upgrade pip
pip install fastapi uvicorn python-multipart geopandas shapely pyproj pandas numpy httpx pytest
```

`python-multipart` is required for FastAPI multipart file uploads. `pytest` is used for automated testing.

For reproducible installations, define and maintain the project's dependencies in a `requirements.txt` file.

Example:

```text
fastapi
uvicorn[standard]
python-multipart
geopandas
shapely
pyproj
pandas
numpy
httpx
pytest
```

### 4. Create Required Directories

Create the storage directories if they do not already exist.

```bash
python -c "from pathlib import Path; [Path(p).mkdir(parents=True, exist_ok=True) for p in ['data/database', 'data/uploads', 'data/extracted']]"
```

The application should also create any required database tables during its initialization process.

### 5. Run the Application

From the project root directory, execute:

```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

The application should now be accessible at:

- **Base URL:** http://localhost:8000
- **Swagger UI:** http://localhost:8000/docs
- **ReDoc:** http://localhost:8000/redoc
- **Health Check:** http://localhost:8000/health

> The commands above assume that `app.main` exposes a FastAPI application instance named `app`.

---

## 🧪 API Usage Examples

### Upload a KML File

Use the following `curl` command:

```bash
curl -X POST \
  "http://localhost:8000/api/files/" \
  -H "accept: application/json" \
  -F "file=@sample.kml"
```

### Upload a ZIP Shapefile

```bash
curl -X POST \
  "http://localhost:8000/api/files/" \
  -H "accept: application/json" \
  -F "file=@sample.zip"
```

Ensure that the ZIP archive contains the required Shapefile components.

### Retrieve Measurements

```bash
curl -X GET \
  "http://localhost:8000/api/files/1/measurements/" \
  -H "accept: application/json"
```

### Retrieve File Details

```bash
curl -X GET \
  "http://localhost:8000/api/files/1/" \
  -H "accept: application/json"
```

### Retrieve Analytics

```bash
curl -X GET \
  "http://localhost:8000/api/files/1/analytics/" \
  -H "accept: application/json"
```

Replace `1` with the actual file ID returned by the upload endpoint.

### Explore the API Interactively

Open [Swagger UI](http://localhost:8000/docs) after starting the application. It allows you to inspect the endpoint definitions, submit upload requests, and review the generated responses.

---

## 🔐 Security and Data Validation

Security is an important part of processing user-supplied geospatial files.

### Filename and Path Safety

- Sanitize filenames before using them in storage paths.
- Generate server-controlled storage names rather than trusting client filenames.
- Prevent path traversal and absolute-path archive entries.
- Resolve extraction paths and verify that they remain inside the designated extraction directory.
- Avoid extracting files directly into shared or publicly accessible directories.

### Upload Size and Archive Limits

- Enforce a maximum upload size of 50 MB.
- Limit the number of entries in ZIP archives.
- Check compressed and uncompressed sizes.
- Enforce reasonable compression-ratio limits.
- Reject corrupted archives and unexpected file structures.
- Stream uploads where practical and avoid unnecessary memory buffering.

The 50 MB limit should be enforced while reading the upload, not only by trusting the declared content length.

### ZIP Shapefile Validation

- Verify archive integrity.
- Require the essential `.shp`, `.shx`, and `.dbf` components.
- Validate related component filenames and their consistency.
- Reject suspicious archive entries.
- Prevent extraction outside the controlled temporary directory.
- Clean up temporary extraction data after processing.

### KML and XML Safety

- Validate XML document structure.
- Disable external entity resolution and other unsafe XML features.
- Protect against XML entity expansion and excessive nesting.
- Apply reasonable limits on document size, feature count, and processing time.
- Reject malformed documents with descriptive errors.

### Geospatial Processing Safety

- Validate source CRS metadata.
- Handle invalid, empty, and unsupported geometries.
- Avoid silently treating unknown coordinate systems as WGS 84.
- Prevent invalid or non-finite measurement values from entering statistical calculations.
- Isolate per-feature failures where possible.
- Apply resource limits to computationally expensive datasets.

### Database Safety

- Use parameterized SQL queries.
- Enable SQLite foreign-key enforcement.
- Use transactions for multi-step writes.
- Avoid storing untrusted data in dynamically constructed SQL.
- Keep uploaded files and database artifacts out of public source-control repositories.

---

## 💡 Design Decisions and Trade-offs

### 1. SQLite for Persistence

**Decision:** Use SQLite as the relational database.

**Rationale:**

- Zero-configuration deployment.
- Lightweight and portable storage.
- Built-in relational tables and constraints.
- Straightforward integration with Python.
- Suitable for a self-contained internship project.

**Trade-off:** SQLite can become a bottleneck under high concurrent write loads and does not provide the full spatial indexing and query capabilities of a dedicated spatial database.

**Alternative:** PostgreSQL with PostGIS for larger deployments requiring spatial indexing, spatial joins, and advanced geospatial queries.

### 2. Dynamic Local UTM Selection

**Decision:** Select a suitable local UTM projection for each feature.

**Rationale:**

- Geographic coordinates are not directly suitable for metric area and length calculations.
- UTM coordinates use meters.
- Feature-specific projections can improve local measurement accuracy when datasets cover different geographic regions.

**Trade-off:** UTM is not equally appropriate for all geographic extents. Features crossing zone boundaries or spanning large regions may require geodesic measurement or another projected CRS.

### 3. IQR for Anomaly Detection

**Decision:** Use the 1.5 × IQR rule to flag unusual measurements.

**Rationale:**

- Quartile-based statistics are relatively robust to extreme values.
- The method does not require measurements to follow a normal distribution.
- It is straightforward to implement and explain.
- It provides interpretable thresholds for identifying unusual feature areas and lengths.

**Trade-off:** IQR thresholds do not understand geographic context or business rules. A statistically unusual feature may still be valid, and different datasets may require different anomaly policies.

### 4. Modular Service Architecture

**Decision:** Separate API routes, business logic, database access, and analytics.

**Rationale:**

- Improves maintainability and testability.
- Keeps HTTP handling separate from geospatial processing.
- Allows measurement and analytics logic to be tested independently.
- Makes it easier to replace storage or introduce background processing later.

**Trade-off:** Additional modules require clear interfaces and careful dependency management.

### 5. Per-Feature Error Handling

**Decision:** Isolate feature-level measurement failures where practical.

**Rationale:**

- One malformed geometry should not necessarily prevent processing all valid features.
- Individual failures can be recorded and inspected.
- Partial results can be useful for large datasets.

**Trade-off:** The API must clearly communicate partial success and distinguish failed measurements from valid zero-valued measurements.

---

## 🧪 Testing and Quality Assurance

Module 07 focuses on validating the entire processing pipeline.

### Recommended Test Coverage

| Test Area | Examples |
|---|---|
| Health check | Application responds successfully. |
| Upload validation | Valid KML and ZIP files are accepted. |
| File security | Oversized uploads and unsafe archives are rejected. |
| Shapefile extraction | Required components are detected. |
| KML parsing | Valid, malformed, and empty documents are handled. |
| Feature extraction | Attributes are serialized into JSON-compatible values. |
| Geometry validation | Empty and invalid geometries are identified. |
| Measurement engine | Polygon areas and line lengths are calculated correctly. |
| CRS handling | Source coordinates are projected into a suitable CRS. |
| Persistence | File metadata and measurements survive subsequent requests. |
| File details | Correct file and feature records are returned. |
| Analytics | Descriptive statistics and IQR fences are calculated correctly. |
| Error handling | Unknown file IDs and unsupported inputs return appropriate errors. |
| Integration | Upload, measurement, retrieval, and analytics work together. |

### Run the Tests

From the project root directory:

```bash
pytest -v
```

For a coverage report, install `pytest-cov` and run:

```bash
pip install pytest-cov
pytest --cov=app --cov-report=term-missing
```

### Important Edge Cases

The automated test suite should cover:

- ZIP archives missing required Shapefile components.
- ZIP Slip and path traversal attempts.
- Corrupted archives and suspicious compression ratios.
- Malformed KML and unsafe XML constructs.
- Files exceeding the upload limit.
- Missing or unknown source CRS metadata.
- Empty geometries and invalid polygons.
- Multipart polygons and lines.
- Features near UTM zone boundaries.
- Coordinates near the antimeridian or poles.
- Null attributes and non-finite numeric values.
- Files containing no measurable features.
- Datasets with too few observations for meaningful anomaly detection.
- Repeated requests and database transaction failures.
- Unknown file IDs and partial processing failures.

---

## 📚 Key Learnings

### Spatial Data Pipeline Engineering

- Loading vector datasets using GeoPandas.
- Understanding geometry types and multipart features.
- Working with coordinate reference systems and projections.
- Transforming geographic coordinates into projected metric coordinates.
- Calculating areas and lengths using suitable projected geometries.
- Handling invalid geometries and floating-point precision.

### Secure File Processing

- Preventing path traversal and ZIP Slip attacks.
- Validating file formats and archive structures.
- Defending against decompression bombs.
- Processing XML documents safely.
- Managing temporary extraction directories.
- Applying upload and resource limits.

### Backend Development

- Designing REST APIs with FastAPI.
- Separating routing from business logic.
- Persisting application data with SQLite.
- Implementing relational constraints and transactions.
- Serializing complex data into JSON-compatible structures.
- Testing API endpoints and service functions.

### Statistical Analytics

- Calculating descriptive statistics and quartiles.
- Applying the IQR method to identify potential outliers.
- Handling missing and invalid measurements.
- Distinguishing statistical anomalies from genuine data errors.
- Communicating analytical results through structured API responses.

---

## 🚀 Future Scope

### 1. PostgreSQL and PostGIS Integration

Migrate from SQLite to PostgreSQL with PostGIS to support:

- Spatial indexes.
- Spatial joins and intersection queries.
- Efficient queries over large geographic datasets.
- Advanced geometry operations.
- Concurrent workloads across multiple application instances.

### 2. Asynchronous Processing

Introduce a background task queue using Celery and Redis, or another suitable job-processing system, to handle expensive file ingestion and measurement workflows.

Potential benefits include:

- Non-blocking upload workflows.
- Background feature processing.
- Job status tracking.
- Retry mechanisms.
- Better scalability for computationally intensive datasets.

The existing 50 MB upload limit should be revisited separately before supporting larger files.

### 3. Docker Containerization

Package the application and its system dependencies into a Docker image.

Benefits include:

- Reproducible environments.
- Consistent GDAL, GEOS, and PROJ dependencies.
- Easier local development.
- Simplified deployment.
- Improved compatibility across development and production environments.

### 4. Cloud Object Storage

Store original uploads and large intermediate artifacts in a service such as Amazon S3 or another compatible object-storage platform.

This would reduce reliance on local filesystem storage and make horizontal scaling easier.

### 5. Advanced Geospatial Analytics

Extend the analytics engine with:

- Geometry validity repair workflows.
- Spatial clustering.
- Area and length distribution comparisons.
- Geodesic measurement support.
- Cross-dataset spatial comparisons.
- Configurable anomaly thresholds.
- Dataset quality scoring and reporting.

### 6. Observability and Production Hardening

Introduce:

- Structured logging.
- Request correlation IDs.
- Processing-duration metrics.
- Health and readiness checks.
- Rate limiting.
- Authentication and authorization.
- Configurable retention and cleanup policies.
- Centralized error reporting.

---

## 📄 License

Add the appropriate license for the project before distributing it publicly.

---

## 👨‍💻 Author and Project Information

**Project:** AEREO Geospatial File Measurement & Analytics API

**Program:** AEREO Cloud Internship Program

**Role:** Software Development Engineer Intern Assignment

**Primary Objective:** Build a secure, reliable, and maintainable backend API for geospatial file ingestion, metric measurement, persistence, and statistical analysis.

---

## ✅ Conclusion

The AEREO Geospatial File Measurement & Analytics API brings together secure file ingestion, geospatial feature extraction, projected-coordinate measurement, relational persistence, and statistical anomaly detection in a modular FastAPI application.

By combining GeoPandas, Shapely, PyProj, SQLite, and an IQR-based analytics engine, the project provides a foundation for automated vector-data processing while emphasizing data quality, security, accuracy, and testability.

The modular architecture also provides a path toward future enhancements such as PostGIS integration, asynchronous processing, containerized deployment, and more advanced spatial analytics.
