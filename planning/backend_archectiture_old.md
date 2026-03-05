Chronoscope API: System Architecture Summary


1. Executive Summary

Chronoscope is a high-performance, asynchronous Python backend built with FastAPI. Its primary purpose is to serve as an analytics and visualization layer on top of existing external analytical databases (like BigQuery and Databricks).
It does not store the raw time-series data itself. Instead, it stores all application-specific metadata—such as user accounts, saved dashboard configurations (called "Chronos"), data source connections, and saved views (called "Labels")—in a Google Firestore database.
The application's core function is to receive visualization requests from a frontend, dynamically generate complex, optimized SQL queries (including for composite metrics), execute those queries against the appropriate external database, and return the data in a format ready for charting. The entire system is designed for deployment on Google Cloud Platform (GCP) using Cloud Run, with a full CI/CD pipeline and automated infrastructure setup.

2. Core Architecture & Functionality

The application follows a clean, three-tier architecture:
API Layer (app/api/v1/): Handles all incoming HTTP requests, validates input using Pydantic models, and routes traffic to the appropriate service. It is responsible for handling authentication and routing.
Service Layer (app/services/): Contains all core business logic. This layer is responsible for permission checks, orchestrating data access, and performing complex operations. For example, ChronoService handles the logic for creating and managing dashboards, while SQLGenerationService is the "brain" that builds queries.
Data Layer (app/data/): This layer is split into two distinct parts:
Application State (app/data/): A set of Repositories (e.g., ChronoRepository, UserRepository) that manage all CRUD (Create, Read, Update, Delete) operations for the app's own metadata stored in Firestore.
External Data (app/data/db_clients/): An abstraction layer (AnalyticalDBInterface) with concrete implementations (e.g., BigQueryClient, DatabricksClient) for communicating with the external databases that hold the actual analytics data.

3. Key Data Flows


Flow 1: Creating a New Chrono (Schema Discovery)

This flow demonstrates how the app connects to an external database to build a new dashboard configuration.
A user sends a POST request to /api/v1/chronos with a data source ID and table name (chronos.py).
The API calls chrono_service.py, which fetches the user's connection details from connection_service.py.
chrono_service.py initializes a SchemaDiscoveryService (schema_discovery.py).
This service uses the appropriate DB client (e.g., bigquery_client.py) to run queries against the external table (e.g., get_table_schema, get_approximate_cardinality).
The service identifies the timestamp column, metrics (numeric columns), and dimensions (string columns).
chrono_service.py receives this schema and constructs a new Chrono Pydantic model (chrono.py).
This new Chrono object is saved to Firestore via chrono_repository.py.

Flow 2: Requesting Graph Data (The Main Query)

This is the most critical and complex flow in the application.
A user's frontend sends a POST request to /api/v1/chronos/{chrono_id}/data with a JSON body defining multiple graphs, metrics (including composite formulas), and filters (data.py).
The API endpoint loads the Chrono object using chrono_service.py to get its configuration (which also verifies user permissions).
It fetches the correct external database client (e.g., bigquery_client.py) using the AnalyticalClientProvider(dependencies.py).
It initializes the SQLGenerationService (sql_generator_service.py) with the Chrono object.
The SQLGenerationService batches all requested graphs based on their time logic (granularity, date range, etc.).
For each batch, it selects the correct SQL dialect generator (e.g., bigquery_generator.py) from its registry.
The generator builds one single, optimized SQL query for the entire batch. It creates a WITH clause for base data, then uses conditional aggregations (e.g., SUM(CASE WHEN ... END)) for each requested time series. It correctly handles composite metrics (e.g., SUM(metric_a) / SUM(metric_b)) and date/time logic.
The db_client executes this single, large query against the external database.
The SQLGenerationService receives the results as a Pandas DataFrame. Its reconstitute_batch_resultsmethod then unpacks this "wide" DataFrame into the separate JSON responses (timestamps, data arrays) expected by the frontend for each graph.
The data.py endpoint logs an analytics event (logging_service.py) and returns the final list of graph data.

4. Module-by-Module Breakdown


main.py

This is the FastAPI application entry point. It initializes the app, sets up the lifespan function (which loads datasources.yaml and system_connections.yaml on startup), registers all API routers, and defines global exception handlers.

app/api/v1/ (API Layer)

auth.py: Handles user login (/login), token refresh (/refresh), session checking (/session), and logout (/logout). Uses security.py to create and verify JWTs.
chronos.py: Manages the lifecycle of Chronos (dashboards). Handles GET (list/get), POST (create), PUT (update), DELETE, and POST /rescan for schema updates.
connections.py: Allows users to POST new database connections, GET their existing ones, and DELETE them.
data.py: Contains the single, most important endpoint: POST /chronos/{chrono_id}/data. This endpoint drives all data visualization.
datasources.py: A simple public endpoint that returns the list of supported data source types from the datasources.yaml config file.
labels.py: Handles CRUD operations for "Labels" (saved views/bookmarks) that are stored as a sub-collection under a Chrono in Firestore.
uploads.py: Provides endpoints for file uploads. It generates signed GCS URLs (/generate-url) for direct client upload and then triggers a table creation from that CSV in BigQuery (/create-table-from-csv).

app/services/ (Service Layer)

sql_generator_service.py: The core query engine. It batches graph requests and uses a dialect-specific generator (sql_dialects/) to build and execute queries. It also reconstitutes the results.
chrono_service.py: Manages all business logic for Chronos, including creation (using SchemaDiscoveryService), permission checking, updates, and deletion.
connection_service.py: Manages the logic for creating and accessing Connection objects. It securely handles credentials by using SecretManagerService to store and retrieve keys.
user_service.py: Handles logic for users, such as get_or_create_user_for_login and managing the datasource_map that links a user to their specific connection.
label_service.py: Business logic for managing "Labels," primarily handling authorization (e.g., only an owner can delete) before calling the repository.
uploads_service.py: Orchestrates the file upload flow, from generating a signed GCS URL to using a db_clientto load the resulting CSV into a table.
schema_discovery.py: A dedicated service that uses a db_client to inspect a table's schema and classify its columns into metrics and dimensions.
cache_manager.py: Manages in-memory caching (CacheInstance) for data, supporting both GLOBAL and per-user SESSION caching policies defined in datasources.yaml.
secret_manager_service.py: A singleton service that acts as a client for Google Secret Manager, used to store and access database credentials.
date_logic.py: A pure-logic helper for calculating comparison date ranges (e.g., "Last 28 Days").

app/data/ (Repository Layer - Firestore)

app_state_client.py: A singleton that initializes and provides the global firestore.Client.
chrono_repository.py: Handles all direct Firestore queries for the chronos collection.
connection_repository.py: Handles all direct Firestore queries for the connections collection.
label_repository.py: Handles all direct Firestore queries for the labels sub-collection.
user_repository.py: Handles all direct Firestore queries for the users collection.

app/data/db_clients/ (External DB Layer)

database_interface.py: An Abstract Base Class (ABC) that defines the contract all database clients must follow (e.S., execute_query, get_table_schema).
bigquery_client.py: The concrete implementation of the interface for Google BigQuery.
databricks_client.py: The concrete implementation of the interface for Databricks.
database_exceptions.py: Defines custom exceptions for this layer (e.g., QueryExecutionError, SchemaNotFoundError).

app/models/ (Data Models)

This directory contains all Pydantic models used for API validation and data structuring. Key models include:
user.py: Defines the User object stored in Firestore.
chrono.py: Defines the Chrono object (the main dashboard config).
connection.py: Defines the Connection object, which uses a Pydantic Union to handle different connection details (e.g., BigQueryConnectionDetails, DatabricksConnectionDetails).
metric.py & dimension.py: Define the data structures for metrics (base and composite) and dimensions.
api_data.py: Defines the request (ApiGraphRequest) and response (GraphDataResponse) models for the main /data endpoint.
label.py: Defines the Label (saved view) object.

app/core/ (Core Services)

config.py: Defines the application Settings using pydantic-settings, loading configuration from environment variables (defined in .env).
security.py: Handles all JWT creation (create_access_token) and validation (verify_access_token). The get_current_user dependency is the core of all API authentication.
dependencies.py: Defines complex FastAPI dependencies, most notably the AnalyticalClientProvider, which efficiently provides and caches DB clients per-request.
exceptions.py: Defines the application's base custom exceptions (e.g., NotFoundError, PermissionError) and maps them to HTTP status codes.
logging_service.py: A singleton service used for all logging. It provides standard logging (info, error) and an analytics method.
pubsub_service.py: A singleton that provides a simple interface for publishing messages to a Pub/Sub topic. It is used exclusively by the LoggingService.
datasource_config.py & system_connection_config.py: Services that load and manage the static YAML configurations (datasources.yaml, system_connections.yaml) at startup.

planning/

These files are not part of the running application but are critical for understanding its design.
user_stories.md: The functional requirements document, detailing every feature from a user's perspective.
tech_design.md: The technical blueprint, outlining the architecture, data models, and API endpoints.

5. Deployment & Operations (CI/CD)

The application is designed for a fully automated GCP deployment.
deploy/: A folder of shell scripts that automate the entire GCP infrastructure setup. 00-config.sh sets variables, and the other scripts (01- to 09-) provision IAM roles, GCS buckets, Firestore, BigQuery datasets (for uploads and analytics), Pub/Sub topics, and Secret Manager.
Dockerfile: A multi-stage Dockerfile that builds a slim, production-ready container for the application.
cloudbuild.yaml: A Google Cloud Build configuration file. It defines the CI/CD pipeline:
Builds the Docker image from the Dockerfile.
Pushes the image to Google Artifact Registry.
Deploys the new image to Google Cloud Run, injecting all necessary environment variables and secrets (like JWT_SECRET_KEY_NAME) into the running service.
Analytics Pipeline: The deploy scripts create a robust, serverless analytics pipeline:
The LoggingService in the app publishes events to a Pub/Sub topic (06-setup-pubsub.sh).
A Pub/Sub subscription is created that is configured to write directly to a BigQuery table (05-setup-bigquery.sh).
This allows all application analytics (e.g., data_query_executed, user_login) to be collected in BigQuery for analysis with zero maintenance.
