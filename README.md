# Ansible role: Logstash
[![CI Molecule](https://github.com/darexsu/ansible-role-logstash/actions/workflows/ci.yml/badge.svg)](https://github.com/darexsu/ansible-role-logstash/actions/workflows/ci.yml)&emsp;![](https://img.shields.io/static/v1?label=idempotence&message=ok&color=success)&emsp;![Ansible Role](https://img.shields.io/ansible/role/d/58438?color=blue&label=downloads)

  - Role:
      - [platforms](#platforms)
      - [install](#install)
      - [requirements](#requirements)
      - [merge behaviour](#merge-behaviour)
  - Playbooks (merge version):
      - [install and configure: Logstash, FirewallD](#install-and-configure-logstash-firewalld-merge-version)
          - [install: Logstash](#install-logstash-merge-version)
          - [configure: Logstash](#configure-logstash-merge-version)
  - Playbooks (full version):
      - [install and configure: Logstash, FirewallD](#install-and-configure-logstash-firewalld-full-version)
          - [install: Logstash](#install-logstash-full-version)
          - [configure: Logstash](#configure-logstash-full-version)

### Platforms

|  Testing         | repo: elastic      |
| :--------------: | :----------------: |
| Debian 11        |  elastic.co        |
| Debian 10        |  elastic.co        |
| Ubuntu 20.04     |  elastic.co        |
| Ubuntu 18.04     |  elastic.co        |
| Oracle Linux 8   |  elastic.co        |
| Rocky Linux 8    |  elastic.co        |

### Install

```
ansible-galaxy install darexsu.logstash --force
```

### Requirements
collections: [ansible.posix](https://docs.ansible.com/ansible/latest/collections/ansible/posix/index.html)

roles: [FirewallD](https://github.com/darexsu/ansible-role-firewalld) (will automatically be installed)


### Merge behaviour

Replace or Merge dictionaries (with "hash_behaviour=replace" in ansible.cfg):
```
# Replace             # Merge
---                   ---
  vars:                 vars:
    dict:                 merge:
      a: "value"            dict: 
      b: "value"              a: "value" 
                              b: "value"

# How does merge work?:
Your vars [host_vars]  -->  default vars [current role] --> default vars [include role]
  
  dict:          dict:              dict:
    a: "1" -->     a: "1"    -->      a: "1"
                   b: "2"    -->      b: "2"
                                      c: "3"
    
```

##### Install and configure: Logstash, FirewallD (merge version)
```yaml
---
- hosts: all
  become: true

  vars:
    merge:
      # Logstash
      logstash:
        enabled: true
        version: "8.x"
      # Logstash -> install
      logstash_install:
        enabled: true
      # Logstash -> config -> logstash.yml
      logstash_yml:
        enabled: true
        data: |
          path.data: /var/lib/logstash
          path.logs: /var/log/logstash
      # Logstash -> config -> logstash.conf
      logstash_conf:
        input_conf:
          enabled: true
          data:
            port: '5044'
        output_conf:
          enabled: true
          data:
            hosts: '["http://localhost:9200"]'
            index: '"%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"'
#           user: 'elastic'
#           password: 'changeme'

      # FirewallD
      firewalld:
        enabled: true
      # FirewallD -> rules
      firewalld_rules:
        logstash_port_5044:
          enabled: true

  tasks:
    - name: role darexsu logstash
      include_role:
        name: darexsu.logstash

```
##### Install: Logstash (merge version)
```yaml
---
- hosts: all
  become: true

  vars:
    merge:
      # Logstash
      logstash:
        enabled: true
        version: "8.x"
      # Logstash -> install
      logstash_install:
        enabled: true

  tasks:
    - name: role darexsu logstash
      include_role:
        name: darexsu.logstash

```
##### Configure: Logstash (merge version)
```yaml
---
- hosts: all
  become: true

  vars:
    merge:
      # Logstash
      logstash:
        enabled: true
      # Logstash -> config -> logstash.yml
      logstash_yml:
        enabled: true
        file: "logstash.yml"
        src: "logstash_yml.j2"
        backup: false
        data: |
          path.data: /var/lib/logstash
          path.logs: /var/log/logstash
      # Logstash -> config -> logstash.conf
      logstash_conf:
        input_conf:
          enabled: true
          file: "input.conf"
          src: "input_conf.j2"
          backup: false
          data:
            port: '5044'
        output_conf:
          enabled: true
          file: "output.conf"
          src: "output_conf.j2"
          backup: false
          data:
            hosts: '["http://localhost:9200"]'
            index: '"%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"'
#           user: 'elastic'
#           password: 'changeme'
      # ElasticSearch -> config -> jvm.options
      logstash_jvm_options:
        enabled: true
        file: "jvm.options"
        src: "jvm_options.j2"
        backup: false
        data: |
          -Xms1g
          -Xmx1g
          11-13:-XX:+UseConcMarkSweepGC
          11-13:-XX:CMSInitiatingOccupancyFraction=75
          11-13:-XX:+UseCMSInitiatingOccupancyOnly
          -Djava.awt.headless=true
          -Dfile.encoding=UTF-8
          -Djruby.compile.invokedynamic=true
          -Djruby.jit.threshold=0
          -Djruby.regexp.interruptible=true
          -XX:+HeapDumpOnOutOfMemoryError
          -Djava.security.egd=file:/dev/urandom
          -Dlog4j2.isThreadContextMapInheritable=true
          11-:--add-opens=java.base/java.security=ALL-UNNAMED
          11-:--add-opens=java.base/java.io=ALL-UNNAMED
          11-:--add-opens=java.base/java.nio.channels=ALL-UNNAMED
          11-:--add-opens=java.base/sun.nio.ch=ALL-UNNAMED
          11-:--add-opens=java.management/sun.management=ALL-UNNAMED

  tasks:
    - name: role darexsu logstash
      include_role:
        name: darexsu.logstash

```
##### Install and configure: Logstash, FirewallD (full version)
```yaml
---
- hosts: all
  become: true

  vars:
    # Logstash
    logstash:
      enabled: true
      version: "8.x"
      repo: "elastic"
      service:
        enabled: true
        state: "started"
    # Logstash -> install
    logstash_install:
      enabled: true
    # Logstash -> config -> logstash.yml
    logstash_yml:
      enabled: true
      file: "logstash.yml"
      src: "logstash_yml.j2"
      backup: false
      data: |
        path.data: /var/lib/logstash
        path.logs: /var/log/logstash
    # Logstash -> config -> logstash.conf
    logstash_conf:
      input_conf:
        enabled: true
        file: "input.conf"
        src: "input_conf.j2"
        backup: false
        data:
          port: '5044'
      output_conf:
        enabled: true
        file: "output.conf"
        src: "output_conf.j2"
        backup: false
        data:
          hosts: '["http://localhost:9200"]'
          index: '"%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"'
#         user: 'elastic'
#         password: 'changeme'
    # ElasticSearch -> config -> jvm.options
    logstash_jvm_options:
      enabled: true
      file: "jvm.options"
      src: "jvm_options.j2"
      backup: false
      data: |
        -Xms1g
        -Xmx1g
        11-13:-XX:+UseConcMarkSweepGC
        11-13:-XX:CMSInitiatingOccupancyFraction=75
        11-13:-XX:+UseCMSInitiatingOccupancyOnly
        -Djava.awt.headless=true
        -Dfile.encoding=UTF-8
        -Djruby.compile.invokedynamic=true
        -Djruby.jit.threshold=0
        -Djruby.regexp.interruptible=true
        -XX:+HeapDumpOnOutOfMemoryError
        -Djava.security.egd=file:/dev/urandom
        -Dlog4j2.isThreadContextMapInheritable=true
        11-:--add-opens=java.base/java.security=ALL-UNNAMED
        11-:--add-opens=java.base/java.io=ALL-UNNAMED
        11-:--add-opens=java.base/java.nio.channels=ALL-UNNAMED
        11-:--add-opens=java.base/sun.nio.ch=ALL-UNNAMED
        11-:--add-opens=java.management/sun.management=ALL-UNNAMED

    # FirewallD
    firewalld:
      enabled: true
      service:
        enabled: true
        state: "started"
    # FirewallD -> rules
    firewalld_rules:
      logstash_port_5044:
        enabled: true
        zone: "public"
        state: "enabled"
        port: "5044/tcp"
        permanent: true

  tasks:
    - name: role darexsu logstash
      include_role:
        name: darexsu.logstash

```
##### Install: Logstash (full version)
```yaml
---
- hosts: all
  become: true

  vars:
    # Logstash
    logstash:
      enabled: true
      version: "8.x"
      repo: "elastic"
      service:
        enabled: true
        state: "started"
    # Logstash -> install
    logstash_install:
      enabled: true

  tasks:
    - name: role darexsu logstash
      include_role:
        name: darexsu.logstash

```
##### Configure: Logstash (full version)
```yaml
---
- hosts: all
  become: true

  vars:
    # Logstash
    logstash:
      enabled: true
    # Logstash -> config -> logstash.yml
    logstash_yml:
      enabled: true
      file: "logstash.yml"
      src: "logstash_yml.j2"
      backup: false
      data: |
        path.data: /var/lib/logstash
        path.logs: /var/log/logstash
    # Logstash -> config -> logstash.conf
    logstash_conf:
      input_conf:
        enabled: true
        file: "input.conf"
        src: "input_conf.j2"
        backup: false
        data:
          port: '5044'
      output_conf:
        enabled: true
        file: "output.conf"
        src: "output_conf.j2"
        backup: false
        data:
          hosts: '["http://localhost:9200"]'
          index: '"%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"'
#         user: 'elastic'
#         password: 'changeme'
    # ElasticSearch -> config -> jvm.options
    logstash_jvm_options:
      enabled: true
      file: "jvm.options"
      src: "jvm_options.j2"
      backup: false
      data: |
        -Xms1g
        -Xmx1g
        11-13:-XX:+UseConcMarkSweepGC
        11-13:-XX:CMSInitiatingOccupancyFraction=75
        11-13:-XX:+UseCMSInitiatingOccupancyOnly
        -Djava.awt.headless=true
        -Dfile.encoding=UTF-8
        -Djruby.compile.invokedynamic=true
        -Djruby.jit.threshold=0
        -Djruby.regexp.interruptible=true
        -XX:+HeapDumpOnOutOfMemoryError
        -Djava.security.egd=file:/dev/urandom
        -Dlog4j2.isThreadContextMapInheritable=true
        11-:--add-opens=java.base/java.security=ALL-UNNAMED
        11-:--add-opens=java.base/java.io=ALL-UNNAMED
        11-:--add-opens=java.base/java.nio.channels=ALL-UNNAMED
        11-:--add-opens=java.base/sun.nio.ch=ALL-UNNAMED
        11-:--add-opens=java.management/sun.management=ALL-UNNAMED

  tasks:
    - name: role darexsu logstash
      include_role:
        name: darexsu.logstash

```
