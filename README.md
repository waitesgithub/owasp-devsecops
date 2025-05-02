# DevSecOps on Raspberry Pi 5 with GitHub Actions using OWASP Project Tools

This project demonstrates a DevSecOps pipeline running primarily on a Raspberry Pi 5, orchestrated by GitHub Actions. It incorporates various open-source tools for static and dynamic application security testing (SAST/DAST), software composition analysis (SCA), infrastructure-as-code (IaC) security scanning, and vulnerability management.

## Architecture

The core services run as Docker containers on a Raspberry Pi 5, defined in the `docker-compose.yml` file. GitHub Actions workflows in this repository automate the build and security analysis process, interacting with these services.

**Services Running on Raspberry Pi 5:**

* **SonarQube:** (`sonarqube`) - Code quality and static security analysis server.
* **SonarQube Database:** (`sonarqube_db`) - PostgreSQL database for SonarQube.
* **Prometheus:** (`prometheus`) - Monitoring and alerting system.
* **Grafana:** (`grafana`) - Data visualization and dashboarding for Prometheus metrics.
* **OWASP Threat Dragon:** (`threatdragon`) - Threat modeling tool.
* **DefectDojo:** (`defectdojo`) - Vulnerability management and collaboration platform.
* **DefectDojo Database:** (`defectdojo_db`) - PostgreSQL database for DefectDojo.

**Security Scanning Tools Used in GitHub Actions:**

* **Semgrep:** (`semgrep/semgrep`) - Fast, rule-based static code analysis for security and code quality.
* **OWASP Dependency Check:** (`owasp/dependency-check`) - Identifies known vulnerabilities in project dependencies.
* **Trivy:** (`aquasec/trivy`) - Comprehensive vulnerability scanner for containers, file systems, and more.
* **Checkov:** (`bridgecrew/checkov`) - Infrastructure-as-code security scanner.

## Getting Started

### Prerequisites

* A Raspberry Pi 5 with Docker and Docker Compose installed.
* Sufficient storage and RAM on the Raspberry Pi 5 to run all the services. Consider at least 4GB of RAM, ideally 8GB.
* Docker Compose installed on your Raspberry Pi 5.
* This GitHub repository cloned to your local machine.
* Network connectivity between your Raspberry Pi 5 and the internet (for pulling Docker images) and potentially between your development machine and the Pi.

### Installation

1.  **Clone the repository:**
    ```bash
    git clone <repository_url>
    cd <repository_directory>
    ```

2.  **Start the Docker Compose environment on your Raspberry Pi 5:**
    ```bash
    docker compose up -d
    ```
    This will download and start all the defined services in the background.

3.  **Access the services:**
    * **SonarQube:** `http://<your_pi_ip>:9000` (initial credentials: `admin`/`admin`)
    * **Grafana:** `http://<your_pi_ip>:3000` (initial credentials: `admin`/`your_grafana_password` - set in `docker-compose.yml`)
    * **Threat Dragon:** `http://<your_pi_ip>:8081`
    * **DefectDojo:** `http://<your_pi_ip>` (initial credentials: `admin`/`your_defectdojo_password` - set in `docker-compose.yml`)
    * **Prometheus:** `http://<your_pi_ip>:9090`
    * **cAdvisor:** `http://<your_pi_ip>:8081`

## GitHub Actions Workflow

The `.github/workflows/devsecops.yml` file defines the automated security scanning pipeline that runs on code pushes and pull requests.

### Workflow Steps

1.  **Checkout Code:** Checks out the repository code to the GitHub Actions runner.
2.  **Set up Docker:** Sets up the Docker environment on the runner.
3.  **Run Semgrep Scan:** Executes Semgrep to scan the Python code (adjust the `--config` and scanned directories as needed).
4.  **Run Dependency Check:** Executes OWASP Dependency Check to identify vulnerabilities in project dependencies.
5.  **Run Trivy Scan:** Builds a Docker image (if a `Dockerfile` exists) and scans it for vulnerabilities using Trivy. **Remember to replace `your_registry/your_image` with your actual container registry and image name.**
6.  **Run Checkov Scan:** Executes Checkov to scan infrastructure-as-code files in the `./infra` directory (if it exists).
7.  **SonarQube Scan:** Analyzes the code with SonarQube running on your Raspberry Pi 5. Ensure you have configured the `SONAR_HOST_URL` and `SONAR_TOKEN` GitHub secrets.
8.  **Upload Findings to DefectDojo:** Uploads the security scan reports (Semgrep, Dependency Check, Trivy, Checkov) to the DefectDojo instance running on your Raspberry Pi 5. Ensure you have configured the `DEFECTDOJO_API_KEY` GitHub secret and adjust the `curl` commands if necessary.

### Setup for GitHub Actions

1.  **Create Secrets:** In your GitHub repository settings (`Settings > Secrets and variables > Actions`), create the following secrets:
    * `SONARQUBE_TOKEN`: A SonarQube user token with permissions to analyze projects.
    * `DEFECTDOJO_API_KEY`: An API key for a DefectDojo user with permissions to import scans.

2.  **Adjust Configuration:**
    * Update the `SONAR_HOST_URL` in the GitHub Actions workflow with the actual IP address or hostname of your Raspberry Pi 5.
    * Replace `your_registry/your_image` in the Trivy step with your container registry and image name.
    * Ensure the paths to your Semgrep rules (`./semgrep_rules`), infrastructure code (`./infra`), and the report output directories match your project structure.

## Usage

1.  Commit and push code changes to your GitHub repository.
2.  The `DevSecOps Pipeline` workflow will automatically trigger.
3.  Monitor the workflow execution in the "Actions" tab of your repository.
4.  View the security analysis results in:
    * **SonarQube:** For code quality and static security findings.
    * **DefectDojo:** For a centralized view of all vulnerability findings from the different scanners.
    * **Grafana:** For monitoring the health and performance of your services on the Raspberry Pi 5.
    * **cAdvisor:** For detailed resource usage of your containers and host.

## Considerations for Raspberry Pi 5

* **Resource Limitations:** Running multiple resource-intensive services like SonarQube and DefectDojo concurrently on a Raspberry Pi 5 can lead to performance degradation. Monitor the Pi's CPU and RAM usage. Consider running only essential services or offloading heavier tasks to more powerful machines if needed.
* **ARM64 Architecture:** Ensure that the Docker images used are compatible with the ARM64 architecture of the Raspberry Pi 5. Most of the images used in this `docker-compose.yml` have multi-architecture support, but performance might vary.

## Project Directory 
```
owasp-devsecops/
├── code/                     # Directory to mount your application code for scanning
├── dependencycheck_data/     # Persistent data for OWASP Dependency Check
├── dependencycheck_report/   # Reports generated by OWASP Dependency Check
├── grafana_dashboard_providers.yml # Grafana dashboard provider configuration
├── grafana_dashboards/       # Directory to store Grafana dashboard JSON files
├── grafana_datasources.yml   # Grafana datasource configuration
├── infra/                    # Directory to store your Infrastructure-as-Code files (e.g., Terraform, CloudFormation)
├── prometheus/               # Persistent data for Prometheus
├── prometheus.yml            # Prometheus configuration file
├── semgrep_rules/            # Directory to store custom Semgrep rules
├── semgrep_results/          # Directory for Semgrep scan results
├── sonarqube_data/           # Persistent data for SonarQube
├── sonarqube_db_data/        # Persistent data for SonarQube database
├── sonarqube_extensions/     # Directory for SonarQube extensions and plugins
├── sonarqube_logs/           # Logs for SonarQube
├── trivy_cache/              # Cache directory for Trivy
├── trivy_report/             # Directory for Trivy scan reports
├── checkov_report/           # Directory for Checkov scan reports
├── defectdojo_db/            # Persistent data for DefectDojo database
├── defectdojo_media/         # Media files for DefectDojo
├── docker-compose.yml        # Your Docker Compose file
└── README.md
```
 ## Directories you might need to create:

- [ ] code/: If you intend to scan application code within the Semgrep and DependencyCheck containers, you'll need to create this directory and place your project's code inside it.
- [ ] dependencycheck_data/: Docker will create this volume, but you might want to create the directory locally beforehand for easier management of permissions or initial data.
- [ ] dependencycheck_report/: Create this directory to store the reports generated by the Dependency Check scans.
- [ ] grafana_dashboards/: If you plan to use custom Grafana dashboards, create this directory and place the .json dashboard files here.
- [ ] prometheus/: Docker will create this volume, but creating the directory locally can help with managing the Prometheus configuration if you choose to store it there instead of directly in prometheus.yml.
- [ ] semgrep_rules/: Create this directory to store any custom Semgrep rules you write.
- [ ] semgrep_results/: Create this directory to store the JSON output of Semgrep scans.
- [ ] sonarqube_data/, sonarqube_db_data/, sonarqube_extensions/, sonarqube_logs/: Docker will create these volumes, but creating the directories locally can aid in managing permissions or adding initial plugins to the sonarqube_extensions directory.
- [ ] trivy_cache/: Docker will create this volume, but you might want to create the directory locally.
- [ ] trivy_report/: Create this directory to store the JSON output of Trivy scans.
- [ ] checkov_report/: Create this directory to store the JSON output of Checkov scans.
- [ ] defectdojo_db/, defectdojo_media/: Docker will create these volumes, but creating the directories locally can help with managing permissions or initial media files.
- [ ] infra/: If you have infrastructure-as-code files (e.g., Terraform .tf files, CloudFormation .yaml or .json templates), create this directory and place them here for Checkov to scan.

## Configuration Files:

- [ ] prometheus.yml: You will need to create this file to configure how Prometheus scrapes metrics from cAdvisor and potentially other services.
- [ ] grafana_datasources.yml: Create this file to configure Prometheus as a data source for Grafana.
- [ ] grafana_dashboards/: Place any custom Grafana dashboard .json files in this directory.
- [ ] grafana_dashboard_providers.yml: Create this file to tell Grafana where to find your dashboard files.
- [ ] semgrep_rules/: Place your custom Semgrep rule files (typically .yaml) in this directory.

## Grafana Dashboards for Monitoring Used
- [ ] https://grafana.com/grafana/dashboards/17642-sonarqube-scan-dashboard/
- [ ] 

## Contributing

Feel free to contribute to this project by suggesting improvements, adding more security tools, or providing more detailed setup instructions.

## License

MIT License

Copyright (c) 2025 UnitedCyberLLC

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
