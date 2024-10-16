# Overview
This project is a **Flask web** application deployed using Docker Compose. The application logs requests to a file, which is then processed by Logstash and sent to Elasticsearch. Kibana is used for visualizing and analyzing the logs. Nginx acts as a reverse proxy, directing traffic to both the Flask application and Kibana.
# Repository Structure
**Key Files and Directories:**

- **app/**: Contains the source code of the Flask application.
  - `app.py`: The main Flask application file.
  - `requirements.txt`: Python dependencies for the application.
  - **Dockerfile**: Instructions to build the Docker image for the Flask application.
- **docker-compose.yml**: Defines all the services and their configurations for Docker Compose.
- **elastic/**: Contains dockerfile to run the ElasticSearch
- **kibana/**: Contains dockerfile to run the Kibana for visualization
- **logstash/**: Contains the Logstash configuration.
  - `logstash.conf`: Configuration file for Logstash.
- **balancer/**: Contains the Nginx configuration.
  - `nginx.con`: The Nginx configuration file.
- **.env**: Environment variables for parameterizing the configurations.
- **README.md**: Documentation and instructions for the project.
---
## Components Description

### 1. Flask Application

**File:** `backend-app/main.py`

A simple Flask web application that:

- Handles GET requests to the root route `/`.
- Logs incoming requests to a file in a format suitable for parsing by Logstash.

**Logging:**

- Logs are written to the file `/var/log/app/app.log`.
- Log format follows the standard `werkzeug` format, for example:
> INFO:werkzeug:127.0.0.1 - - [15/Oct/2024 11:59:09] "GET / HTTP/1.1" 200 -
### 2. Logstash

**Configuration File:** `logstash/pipeline/logstash.conf`

Logstash is configured to:

- Read logs from the file generated by the Flask application.
- Parse logs using the `dissect` filter.
- Convert timestamp strings into date objects.
- Send processed logs to Elasticsearch.

### 3. Elasticsearch
Elasticsearch is used for storing and indexing logs.

**Configuration**:

- Uses the official Elasticsearch Docker image (exact version is set in .env file).
- Security features are disabled (xpack.security.enabled=false) for simplicity in a development environment.
- Memory usage is optimized

### 4. Kibana
Kibana provides visualization and analysis of logs stored in Elasticsearch.

**Configuration**:

- Configured with a base path /kibana to work behind a reverse proxy.
- Connects to Elasticsearch using the service name elasticsearch.

### 5. Nginx
**Configuration File**: `balancer/config/nginx.conf`

Nginx serves as a reverse proxy for:

- Routing requests to the Flask application.
- Proxying requests to Kibana at the /kibana/ path.

### 6. Docker Compose
**File**: `docker-compose.yml`

Defines and configures all the services in the project.

**Services**:

- `elasticsearch`: Stores logs.
- `kibana`: Visualizes logs.
- `app-back`: Runs the Flask application.
- `logstash`: Processes logs from the file and sends them to Elasticsearch.
- `balancer`: Acts as a reverse proxy for the Flask application and Kibana.

**General Settings:**

- All services are connected via the elk network for inter-service communication.
- Uses volumes to share log files between the Flask application and Logstash.
- Environment variables are loaded from the .env file.

# Instructions to Run the Project
## Step 1: Clone the Repository:
```
git clone https://github.com/Dstewie/log_collection.git
cd log_collection
```
## Step 2: Configure Environment Variables
- If necessary, edit the .env file to adjust the default settings (ELK stask version).
## Step 3: Start Docker Compose
```
docker-compose up --build
```
- The `--build` flag ensures that all images are built with the latest changes.

## Step 4: Test the Flask Application
- Open a web browser and navigate to `http://<ip docker>/`.
- You should see the message Hello, World!

## Step 5: Generate Logs
- Refresh the page multiple times.
- Navigate to non-existent routes, e.g., `http://<ip docker>/unknown`, to generate logs with different response codes.

## Step 6: Set Up Kibana
- Navigate to `http://<ip docker>/kibana/`.
- You may need to configure the index in the Discover section.

### Configuring the Index in Kibana:

- Go to Management > Stack Management > Index Management.
- Click on `Indices` tab and find the index name `app_back`.
- Find Discover and choose index `app_back`
- Choose needed fields
![Final result](/assest/img/kibana_img.png>)
