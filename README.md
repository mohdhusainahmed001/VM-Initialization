# VM Initialization Automation

This repository contains Ansible playbooks, roles, and a Jenkins pipeline designed to automate the initialization, configuration, and provisioning of various Virtual Machines (VMs) across multiple environments.

## Overview

The automation framework uses **Ansible** to configure servers and is orchestrated via a **Jenkins Pipeline**. The pipeline supports deploying to several distinct project environments by applying role-based configurations to target host groups. 

The default SSH user for bootstrapping is `onexadmin`, and sensitive credentials (like user and become passwords) are securely encrypted using Ansible Vault.

## Supported Environments (Projects)

You can initialize the following projects using this repository:

* **`omni-api`**: Sets up an Omni API Cluster. It initializes a Docker Swarm cluster (handling primary leaders, secondary managers, and workers), configures a Nexus registry mirror, and sets up Nginx and Prometheus Node Exporter.
* **`engage`**: Provisions servers with Node.js, PM2, Nginx, and applies necessary system tuning.
* **`warcs`**: Sets up Docker, Nginx, system tuning, and Node Exporter.
* **`odp` (Open Data Platform)**: Bootstraps the `onexodp` service user, configures passwordless sudo, provisions SSH keys, disables swap, installs dependencies (Java 17/21, Python 3), and creates strict directory structures for Kafka, Zookeeper, Doris, and Grafana.
* **`odp-lakehouse`**: Similar to ODP, but tailored specifically for Lakehouse components across a large cluster of nodes.

## Repository Structure

* **`Jenkinsfile`**: The CI/CD pipeline definition that manages the deployment lifecycle.
* **`playbooks/`**: Contains the main Ansible playbooks for each project (e.g., `omni-api.yml`, `odp.yml`, `engage.yml`).
* **`playbooks/roles/`**: Modular Ansible roles defining specific system configurations:
    * `system-tuning`: Disables swap permanently, sets ulimits, and optimizes TCP/kernel parameters via `sysctl`.
    * `docker` & `docker-swarm`: Installs Docker and automates the creation of a Docker Swarm overlay network.
    * `create-odp-user` & `create-odp-dirs`: Creates the `onexodp` user and provisions necessary storage directories with correct permissions.
    * `install-java` / `install-python` / `node.js`: Manages the installation of runtime environments.
    * `nexus-mirror`: Configures the Docker daemon to use an internal Nexus registry mirror.
    * `disable-unattended-upgrades`: Stops automated APT upgrades and sets the timezone to `Asia/Kolkata`.
* **`inventory/`**: 
    * `hosts`: INI-based inventory file defining the host IP addresses and groupings (e.g., `[omni_managers]`, `[odp-doris-be]`, `[odp-lakehouse]`).
    * `group_vars/`: Contains standard and vault-encrypted variables (like `vault.yml` and `vars.yml`).

## Jenkins Pipeline Execution

Deployments are managed through Jenkins using a parameterized pipeline. The execution flow is as follows:

1.  **Parameter Selection:** The user selects the target environment (`project`) they wish to initialize.
2.  **Dry Run:** Ansible runs the selected playbook in check mode (`--check`) using the `ansible-vault-pass` credentials to validate changes without applying them.
3.  **Approval Gate:** The pipeline pauses for up to 10 minutes, asking the user to review the dry-run output and explicitly select "Continue Deployment".
4.  **Deployment:** Once approved, the playbook is executed fully to apply the configurations to the targeted VMs.
5.  **Cleanup:** The Jenkins workspace is cleaned up automatically after execution.

## Prerequisites

* **Jenkins**: With the Ansible plugin installed and configured.
* **Ansible**: Installed on the Jenkins agent.
* **Credentials**: 
    * The `ansible-vault-pass` credential must be configured in Jenkins to decrypt the Ansible Vault.
    * SSH keys for the `onexadmin` user must be authorized on the target infrastructure for initial connectivity.
