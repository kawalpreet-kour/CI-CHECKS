# ScyllaDB Ansible Role – Design Document

<img width="200" height="200" alt="scylladnb" src="https://github.com/user-attachments/assets/5bf6d92d-6d18-4910-b71a-1027c3328986" />

---
## Author Information

| Last Updated On | Version | Author           | Level            | Reviewer                      |
|-----------------|---------|------------------|------------------|-------------------------------|
| 01-09-2025      | V1.0    | Kawalpreet Kour  | Internal Review  | Pritam                        |
|                 |         | Kawalpreet Kour  | L0               | Shreya / Sharvari             |
|                 |         | Kawalpreet Kour  | L1               | Abhishek V                    |
|                 |         | Kawalpreet Kour  | L2               | Abhishek Dubey / Rishabh Sharma |

---

<details>
  <summary><h2><strong>Table of Contents</strong></h2></summary>

1. [Introduction](#introduction)  
2. [What is ScyllaDB?](#what-is-scylladb)  
3. [Why Use Ansible for ScyllaDB Deployment?](#why-use-ansible-for-scylladb-deployment)  
4. [Role Components](#role-components)  
5. [Variables and Templates](#variables-and-templates)  
6. [Pre-requisites](#pre-requisites)  
7. [Advantages](#advantages)  
8. [Best Practices](#best-practices)  
9. [FAQs](#faqs)  
10. [Contact Information](#contact-information)  
11. [References](#references)

</details>

---
## Pre-requisites

| Item              | Description                                                  |
|------------------|--------------------------------------------------------------|
| OS Compatibility | CentOS 7/8, RHEL 7/8, Ubuntu 20.04+                          |
| Python           | Python 3.x (required by Ansible)                            |
| Ansible Version  | >= 2.9                                                       |
| Inventory File   | Must list all nodes with correct IPs or hostnames           |
| SSH Access       | Password-less SSH to target nodes (using keys)              |
| Ports            | Open ports like 9042, 7000, 7001, 7199, etc.                |

---

## Introduction

This document outlines the design of an Ansible role for deploying and managing **ScyllaDB**, a high-performance NoSQL database compatible with Apache Cassandra. The role will automate ScyllaDB installation, configuration, and clustering across environments.

---

## What is ScyllaDB?

**ScyllaDB** is a distributed NoSQL wide-column database designed for high throughput and low latency. It is fully compatible with Cassandra Query Language (CQL) and optimized for modern hardware.

---

## Why Use Ansible for ScyllaDB Deployment?

| Reason                   | Explanation                                                                 |
|--------------------------|-----------------------------------------------------------------------------|
| Automation               | Eliminates manual steps across multiple servers.                           |
| Consistency              | Ensures uniform configuration across environments.                         |
| Scalability              | Easily scale cluster nodes with inventory updates.                         |
| Idempotency              | Ansible ensures operations are repeatable without adverse effects.         |
| CI/CD Integration        | Can be integrated into pipelines (e.g., Jenkins) for automated delivery.   |

---

# Workflow Diagram

```mermaid
flowchart LR
    A[Read Inventory\n& Variables] --> B[Install Dependencies\n(Python, curl)]
    B --> C[Add ScyllaDB Repository]
    C --> D[Install ScyllaDB Package]
    D --> E[Run scylla_setup]
    E --> F[Configure scylla.yaml\nUsing Template]
    F --> G[Start & Enable\nscylla-server Service]
    G --> H{Multi-node Setup?}
    H -- Yes --> I[Join Cluster]
    H -- No --> J[Skip Clustering]
    I --> K[Verify Service Status\n& Cluster Health]
    J --> K
    K --> L[End]


```
---

## Ansible Role Directory Structure

```bash
scylladb/
├── defaults/
│   └── main.yml            # Default variable definitions
├── files/
│   └── scylla.repo         # Static repo file (if needed)
├── handlers/
│   └── main.yml            # Handlers for restarting services
├── meta/
│   └── main.yml            # Role metadata and dependencies
├── tasks/
│   ├── main.yml            # Master task list
│   ├── install.yml         # Install ScyllaDB
│   ├── configure.yml       # Apply configurations
│   └── cluster.yml         # Cluster-specific logic
├── templates/
│   └── scylla.yaml.j2      # Templated configuration file
├── tests/
│   ├── inventory           # Sample inventory
│   └── test.yml            # Role testing playbook
└── vars/
    └── main.yml            # Variable overrides (env-specific)
```
---

## Role Components

| Component     | Description                                                               |
|---------------|---------------------------------------------------------------------------|
| `defaults/`   | Contains default values for role variables                                |
| `tasks/`      | Includes all operational steps like install, configure, cluster setup     |
| `templates/`  | Contains Jinja2 templates used for config files like `scylla.yaml`         |
| `handlers/`   | Defines service restart or reload triggers                                |
| `files/`      | Used for static files like custom repo files or certificates              |
| `vars/`       | Contains environment or site-specific variable overrides                  |

---

## Variables and Templates

| Variable Name           | Description                          | Default Value                    |
|-------------------------|--------------------------------------|----------------------------------|
| `scylla_version`        | Version to install                   | `stable`                         |
| `scylla_cluster_name`   | Name of the ScyllaDB cluster         | `scylla-cluster`                 |
| `scylla_seeds`          | Seed nodes for clustering            | `["10.0.0.1", "10.0.0.2"]`       |
| `scylla_listen_address` | Node listen IP                       | `{{ ansible_host }}`             |
| `scylla_rpc_address`    | RPC IP address                       | `{{ ansible_host }}`             |

### Template: `scylla.yaml.j2`

This file will dynamically render the following values into the `scylla.yaml` config:

- `cluster_name`
- `seeds`
- `listen_address`
- `rpc_address`
- `endpoint_snitch`

---

## Advantages

| Benefit             | Explanation                                                        |
|---------------------|--------------------------------------------------------------------|
| Faster Deployment   | Automate setup across all environments in minutes                 |
| Reusability         | Role can be reused across different clusters or use-cases         |
| Consistency         | Avoid configuration drift                                         |
| Cluster Flexibility | Easily scale nodes and configure clustering                       |
| CI/CD Ready         | Role can be plugged into Jenkins or other automation pipelines     |

---

## Best Practices

| Practice              | Description                                                   |
|-----------------------|---------------------------------------------------------------|
| Use Vault for Secrets | Store credentials or keys using Ansible Vault                 |
| Tag Tasks             | Use tags like `install`, `configure`, `cluster` for control   |
| Run in Check Mode     | Validate changes before applying                              |
| Test in Dev First     | Always test role in non-prod before rolling out               |
| Enable Logging        | Collect logs to troubleshoot deployment issues                |

---

## FAQs

**Q: Can this role handle multi-node clusters?**  
**A:** Yes. By setting the `scylla_seeds` variable and providing an accurate inventory, the role can initialize and join multiple nodes into a cluster.

**Q: Does the role install Scylla Manager or Monitoring Stack?**  
**A:** Not in the initial version. These can be implemented in future iterations.

**Q: How do I override default config values?**  
**A:** Use `group_vars` or `host_vars` in your playbook to override variables.

---

## Contact Information

| Name            | Email                                      |
|------------------|--------------------------------------------|
| Kawalpreet Kour | kawalpreet.kour.snaatak@mygurukulam.co     |

---

## References

| Description                | Link                                                       |
|----------------------------|------------------------------------------------------------|
| ScyllaDB Official Docs     | [https://docs.scylladb.com/](https://docs.scylladb.com/)   |
| Ansible Best Practices     | [Ansible Docs](https://docs.ansible.com/ansible/latest/user_guide) |
| ScyllaDB Installation Guide| [Getting Started](https://docs.scylladb.com/stable/getting-started) |

---
