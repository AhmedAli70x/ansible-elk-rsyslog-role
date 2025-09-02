# ELK Stack + Rsyslog Ansible Role

![Ansible](https://img.shields.io/badge/ansible-2.9%2B-blue)
![Rocky Linux](https://img.shields.io/badge/Rocky%20Linux-8%2F9-red)
![License](https://img.shields.io/badge/license-MIT-green)

An Ansible role for deploying and configuring ELK Stack (Elasticsearch, Logstash, Kibana) with integrated Rsyslog server for centralized logging on Rocky Linux systems.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Role Variables](#role-variables)
- [Inventory Configuration](#inventory-configuration)
- [Usage](#usage)
- [Architecture](#architecture)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)
- [Examples](#examples)
- [License](#license)
- [Author](#author)

## Overview

This role automates the deployment of a complete logging infrastructure including:

- **ELK Server**: Elasticsearch for storage, Logstash for processing, Kibana for visualization
- **Rsyslog Server**: Centralized log collection and forwarding
- **Client Configuration**: Automatic log forwarding from multiple clients

## Features

- ✅ **Full ELK Stack**: Elasticsearch 8.x, Logstash, and Kibana installation
- ✅ **Rsyslog Integration**: Centralized syslog server with client forwarding
- ✅ **Rocky Linux Optimized**: Proper user/group handling for RHEL-based systems
- ✅ **Security Configuration**: Firewall rules and service hardening
- ✅ **Health Monitoring**: Built-in service verification and connectivity tests
- ✅ **Flexible Variables**: Extensive customization options
- ✅ **Log Separation**: Client-specific log file organization
- ✅ **JVM Optimization**: Memory management for Java-based services

## Requirements

- **Ansible**: 2.9 or higher
- **Operating System**: Rocky Linux 8/9, CentOS 8/9, or RHEL 8/9
- **Memory**: Minimum 2GB RAM for ELK server, 1GB for rsyslog server
- **Java**: Java 17 (automatically installed)
- **Python**: 3.6 or higher
- **Network**: TCP/UDP ports 514, 5000, 5044, 9200, 5601

## Installation

### From Ansible Galaxy

```bash
ansible-galaxy install your_namespace.elk_rsyslog_role
```

### From GitHub

```bash
git clone https://github.com/your_username/elk-rsyslog-role.git roles/elk_role
```

## Role Variables

### Server Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `elk_server` | `192.168.181.132` | ELK stack server IP address |
| `rsyslog_server` | `192.168.181.133` | Rsyslog server IP address |
| `client_server` | `192.168.181.134` | Client server IP address |

### ELK Stack Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `elasticsearch_version` | `8.11.0` | Elasticsearch version to install |
| `elasticsearch_heap_size` | `256m` | JVM heap size for Elasticsearch |
| `elasticsearch_port` | `9200` | Elasticsearch HTTP port |
| `elasticsearch_cluster_name` | `my-elk-cluster` | Elasticsearch cluster name |
| `kibana_port` | `5601` | Kibana web interface port |
| `logstash_port` | `5000` | Logstash syslog input port |
| `logstash_beats_port` | `5044` | Logstash beats input port |
| `logstash_heap_size` | `256m` | JVM heap size for Logstash |

### Rsyslog Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `rsyslog_port` | `514` | Rsyslog listening port (TCP/UDP) |
| `rsyslog_client_log_path` | `/var/log/client` | Directory for client logs |
| `rsyslog_config_path` | `/etc/rsyslog.d` | Rsyslog configuration directory |

### Java Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `java_version` | `17` | Java version to install |
| `java_home_path` | `/usr/lib/jvm/java-17-openjdk` | Java installation path |

### Security Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `elk_security_enabled` | `false` | Enable ELK security features |
| `elk_ssl_enabled` | `false` | Enable SSL/TLS encryption |

See [vars/main.yml](vars/main.yml) for complete variable list.

## Inventory Configuration

### Basic Inventory Example

```ini
[elk]
192.168.181.132 ansible_user=ansible ansible_ssh_private_key_file=<Private_key_Path>

[rsyslog]
192.168.181.133 ansible_user=ansible ansible_ssh_private_key_file=~/.ssh/ahmed.priv

[client]
192.168.181.134 ansible_user=ansible ansible_ssh_private_key_file=~/.ssh/ahmed.priv
```

### YAML Inventory Example

```yaml
# inventory/hosts.yml
all:
  vars:
    ansible_user: ansible
    ansible_ssh_private_key_file: ~/.ssh/ahmed.priv
    ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
  children:
    elk:
      hosts:
        elk-server:
          ansible_host: 192.168.181.132
    rsyslog:
      hosts:
        rsyslog-server:
          ansible_host: 192.168.181.133
    client:
      hosts:
        client-server:
          ansible_host: 192.168.181.134
```

### Advanced Inventory with Variables

```yaml
# inventory/hosts.yml
all:
  vars:
    ansible_user: ansible
    ansible_ssh_private_key_file: ~/.ssh/ahmed.priv
  children:
    elk:
      hosts:
        elk-server:
          ansible_host: 192.168.181.132
          elasticsearch_heap_size: "512m"
          logstash_heap_size: "512m"
    rsyslog:
      hosts:
        rsyslog-server:
          ansible_host: 192.168.181.133
          rsyslog_retention_days: 30
    client:
      hosts:
        client-server-1:
          ansible_host: 192.168.181.134
        client-server-2:
          ansible_host: 192.168.181.135
      vars:
        log_level: info
```

## Usage

### Basic
