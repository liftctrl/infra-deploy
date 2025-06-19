# infra-deploy

infra-deploy is a modular infrastructure deployment repository that brings together automation tools, configuration files, and service provisioning scripts for building and managing modern infrastructure environments.

## 🔧 Modules

This project is organized into directories by component or technology. Each folder contains its own README.md with usage details, setup steps, and configurations.

| Module        | Description                              |
| ------------- | ---------------------------------------- |
| `ansible/`    | Infrastructure automation with Ansible   |
| `docker/`     | Dockerfiles and container configurations |
| `gitlab/`     | GitLab installation and configuration    |
| `grafana/`    | Dashboards and monitoring with Grafana   |
| `jenkins/`    | CI/CD pipelines using Jenkins            |
| `keepalive/`  | High availability with Keepalived        |
| `kubernetes/` | Kubernetes cluster setup and management  |
| `lvs/`        | Load balancing with Linux Virtual Server |
| `mysql/`      | MySQL database provisioning              |
| `nginx/`      | NGINX configuration and reverse proxy    |
| `prometheus/` | Monitoring with Prometheus               |
| `redis/`      | Redis configuration and deployment       |
| `terraform/`  | IaC provisioning with Terraform          |
| `tomcat/`     | Apache Tomcat server setup               |
| `zabbix/`     | Monitoring with Zabbix                   |

## 🚀 Usage

1. Clone the repository:
```bash
git clone https://your.repo.url/infra-deploy.git
cd infra-deploy
```
2. Navigate to the desired module and follow the setup steps in the local `README.md`.

3. Run your provisioning tool (e.g., Ansible, Docker, Terraform) according to your infrastructure needs.

## 📁 Project Layout

```bash
infra-deploy/
├── <component>/
│   └── README.md
├── LICENSE
└── README.md
```
Each <component> directory is independently maintainable and contains all files necessary for its deployment.

## 📝 License

Distributed under the MIT License.

## 🙌 Contributions

Contributions are welcome. Submit issues and pull requests to help improve this project or add new infrastructure components.
