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
ansible-galaxy install ahmedali70x.elk_rsyslog_role
```

### From GitHub

```bash
git clone git clone [https://github.com/ahmedali70x/elk-rsyslog-role.git roles/elk_role](https://github.com/AhmedAli70x/ansible-elk-rsyslog-role.git)
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

### Server IP Configuration

Update the server IP addresses in the role's variables file:

```yaml
# elk_role/vars/main.yml
---
# Server IP Addresses
elk_server: add_server_ip_address
rsyslog_server: add_server_ip_address
client_server: add_server_ip_address
```

Replace `add_server_ip_address` with your actual server IP addresses before deployment.

## Usage

### Basic Deployment

```yaml
# site.yml
---
- name: Deploy ELK + Rsyslog Infrastructure
  hosts: all
  become: yes
  roles:
    - elk_role
```

Run the playbook:

```bash
ansible-playbook -i inventory site.yml
```

### Production Deployment with Custom Variables

```yaml
# production.yml
---
- name: Production ELK + Rsyslog Deployment
  hosts: all
  become: yes
  roles:
    - role: elk_role
      vars:
        elasticsearch_heap_size: "2g"
        logstash_heap_size: "1g"
        elasticsearch_cluster_name: "production-logs"
        environment: "production"
        elk_security_enabled: true
```

### Selective Deployment

Deploy only specific components:

```bash
# Deploy only ELK stack
ansible-playbook -i inventory site.yml --tags "elk"

# Deploy only rsyslog server
ansible-playbook -i inventory site.yml --tags "rsyslog"

# Deploy only client configuration
ansible-playbook -i inventory site.yml --tags "client"
```

## Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Client        │    │   Rsyslog       │    │   ELK Stack     │
│   Servers       │────┤   Server        │────┤   Server        │
│                 │    │                 │    │                 │
│ - Log Generator │    │ - Log Collector │    │ - Elasticsearch │
│ - Rsyslog Client│    │ - Log Parser    │    │ - Logstash      │
│ - Log Forwarding│    │ - Log Forwarding│    │ - Kibana        │
└─────────────────┘    └─────────────────┘    └─────────────────┘
        │                        │                        │
        │ Syslog (TCP/UDP 514)   │ Parsed Logs (TCP 5000)│
        └────────────────────────┼────────────────────────┘
                                 │
                        ┌─────────────────┐
                        │   Log Storage   │
                        │   & Analysis    │
                        │                 │
                        │ - Search Index  │
                        │ - Visualizations│
                        │ - Dashboards    │
                        └─────────────────┘
```

### Data Flow

1. **Log Generation**: Applications and system services generate logs on client servers
2. **Log Collection**: Client rsyslog forwards logs to centralized rsyslog server
3. **Log Processing**: Rsyslog server processes and forwards logs to Logstash
4. **Log Indexing**: Logstash parses and indexes logs into Elasticsearch
5. **Log Visualization**: Kibana provides web interface for searching and visualizing logs

## Testing

### Automated Testing

The role includes built-in testing and verification:

```bash
# Run connectivity tests
ansible-playbook -i inventory site.yml --tags "health-check"

# Generate test logs
ansible-playbook -i inventory site.yml --tags "test-logs"
```

### Manual Testing

#### Test Log Generation

On client servers:
```bash
# Generate test logs
logger "Test message from $(hostname) - $(date)"
logger -p local0.error "ERROR: Test error message"
logger -p auth.info "AUTH: Test authentication log"
```

#### Verify Log Reception

On rsyslog server:
```bash
# Check client logs
sudo tail -f /var/log/client/client1.log
sudo tail -f /var/log/client/192.168.181.134.log
```

On ELK server:
```bash
# Check Elasticsearch indices
curl "localhost:9200/_cat/indices?v"

# Search for recent logs
curl "localhost:9200/_search?q=*&sort=@timestamp:desc&pretty"
```

#### Service Health Checks

```bash
# Check all services
sudo systemctl status elasticsearch kibana logstash rsyslog

# Check listening ports
sudo netstat -tlnp | grep -E "(9200|5601|5044|5000|514)"

# Test web interfaces
curl http://localhost:9200/
curl -I http://localhost:5601/
```

### Performance Testing

#### Generate Load

```bash
# Generate continuous test logs
for i in {1..1000}; do
    logger "Load test message $i from $(hostname) - $(date)"
    sleep 0.1
done
```

#### Monitor Performance

```bash
# Monitor Elasticsearch performance
curl "localhost:9200/_cluster/stats?pretty"

# Check JVM heap usage
curl "localhost:9200/_nodes/stats/jvm?pretty"

# Monitor system resources
htop
iotop
```

## Troubleshooting

### Common Issues

#### Elasticsearch Won't Start

```bash
# Check logs
sudo journalctl -u elasticsearch -f

# Common fixes:
sudo chown -R elasticsearch:elasticsearch /var/lib/elasticsearch
sudo sysctl vm.max_map_count=262144  # Add to /etc/sysctl.conf for persistence
```

#### Rsyslog Not Receiving Logs

```bash
# Check if rsyslog is listening
sudo netstat -tlnp | grep 514

# Test configuration
sudo rsyslogd -N1

# Check firewall
sudo firewall-cmd --list-ports | grep 514
```

#### Kibana Not Accessible

```bash
# Check Kibana logs
sudo journalctl -u kibana -f

# Ensure Elasticsearch is running
curl localhost:9200/_cluster/health

# Wait for Kibana to fully start (can take 2-3 minutes)
```

#### Connection Timeouts

```bash
# Test network connectivity
telnet rsyslog-server 514
telnet elk-server 5000

# Check iptables/firewall
sudo iptables -L -n
sudo firewall-cmd --list-all
```

### Debug Commands

```bash
# Service status overview
sudo systemctl status elasticsearch kibana logstash rsyslog

# Port listening check
sudo ss -tlnp | grep -E "(9200|5601|5044|5000|514)"

# Log monitoring
sudo journalctl -f -u elasticsearch -u kibana -u logstash -u rsyslog

# Configuration validation
sudo /usr/share/elasticsearch/bin/elasticsearch-config --check
sudo rsyslogd -N1
```

### Log Locations

- **Elasticsearch**: `/var/log/elasticsearch/`
- **Kibana**: `/var/log/kibana/`
- **Logstash**: `/var/log/logstash/`
- **Rsyslog**: `/var/log/messages`, `/var/log/client/`
- **System**: `journalctl -u service-name`

## Examples

### Environment-Specific Deployments

#### Development Environment

```yaml
# vars/dev.yml
---
elk_server: "192.168.1.10"
rsyslog_server: "192.168.1.11"
client_server: "192.168.1.12"
elasticsearch_heap_size: "256m"
logstash_heap_size: "256m"
environment: "development"
```

#### Production Environment

```yaml
# vars/prod.yml
---
elk_server: "10.0.1.10"
rsyslog_server: "10.0.1.11"
client_server: "10.0.1.12"
elasticsearch_heap_size: "4g"
logstash_heap_size: "2g"
elasticsearch_cluster_name: "production-logs"
environment: "production"
elk_security_enabled: true
```

#### Multi-Client Setup

```yaml
# inventory for multiple clients
all:
  children:
    elk:
      hosts:
        elk-server:
          ansible_host: 192.168.1.10
    rsyslog:
      hosts:
        rsyslog-server:
          ansible_host: 192.168.1.11
    client:
      hosts:
        web-server-1:
          ansible_host: 192.168.1.20
        web-server-2:
          ansible_host: 192.168.1.21
        db-server-1:
          ansible_host: 192.168.1.30
        app-server-1:
          ansible_host: 192.168.1.40
```

### Custom Configuration Examples

#### High-Performance Setup

```yaml
- name: High-performance ELK deployment
  hosts: all
  become: yes
  roles:
    - role: elk_role
      vars:
        elasticsearch_heap_size: "8g"
        logstash_heap_size: "4g"
        elasticsearch_cluster_name: "high-perf-cluster"
        max_connections: 5000
        buffer_size: "128k"
```

#### Secure Setup

```yaml
- name: Secure ELK deployment
  hosts: all
  become: yes
  roles:
    - role: elk_role
      vars:
        elk_security_enabled: true
        elk_ssl_enabled: true
        elasticsearch_cluster_name: "secure-cluster"
```

### Accessing Services

After successful deployment:

- **Kibana Dashboard**: http://192.168.181.132:5601
- **Elasticsearch API**: http://192.168.181.132:9200
- **Rsyslog Server**: 192.168.181.133:514

## Dependencies

None. This role manages all required dependencies internally.

## Compatibility

- **Rocky Linux**: 8, 9
- **CentOS**: 8, 9
- **RHEL**: 8, 9
- **Ansible**: 2.9+

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature-name`)
3. Make your changes
4. Test thoroughly
5. Commit your changes (`git commit -am 'Add new feature'`)
6. Push to the branch (`git push origin feature-name`)
7. Create a Pull Request


---

*Last updated: $(date '+%Y-%m-%d')*
