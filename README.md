# [Ansible role haproxy]

Install and configure haproxy on your system.

## Requirements

None.

## Use the role

Ansible pulls the role from GitLab directly. Add this to your `requirements.yaml` file at the root of your playbook:

```yaml
roles:
  - name: usi_server
    src: git+https://github.com/jmcnab006/anro_haproxy.git
    version: main
```

Use the `ansible-galaxy` command to install all roles defined in your `requirements.yaml`:

```command
ansible-galaxy install -r requirements.yaml
```

Use the name of the role directory in the `roles:` list to use it in your playbook.

```yaml
- name: ROLES | Run anro_haproxy role on all servers
  hosts: all
  roles:
    - anro_haproxy
```

## [Role Variables](#role-variables)

Refer to [argument specifications](meta/argument_specs.yaml) for detailed information on all role variables.

The default values for the variables are set in [`defaults/main.yml`]:

### HAProxy 'stats'
```yaml
anro_haproxy_stats_enabled: true
anro_haproxy_stats_port: 1936
anro_haproxy_stats_address: 0.0.0.0
anro_haproxy_stats_uri: /
anro_haproxy_stats_config: []
```

### HAProxy 'global'
```yaml
anro_haproxy_global_log: 
    - /dev/log    local0
    - /dev/log    local1 notice
```

```yaml
anro_haproxy_global_settings:
  chroot: /var/lib/haproxy
  user: haproxy
  group: haproxy
  ca-base: /etc/ssl/certs
  crt-base: /etc/ssl/private
  ssl-default-bind-ciphers: "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACUN20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384"
  ssl-default-bind-ciphersuites: "TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256"
  ssl-default-bind-options: "ssl-min-ver TLSv1.2 no-tls-tickets"
```

```yaml
anro_haproxy_global_stats:
  - socket /run/haproxy/admin.sock mode 660 level admin
  - timeout 30s
```

For any remaining configuration not covered by predefined variables: 
```yaml
anro_haproxy_global_config: []
```

```yaml
anro_haproxy_global_daemon: true
```

### HAProxy 'default' section
```yaml
anro_haproxy_defaults_option:
  - httplog
  - dontlognull
```

```yaml
anro_haproxy_defaults_timeout:
  - connect 5000
  - client 500000
  - server 500000
```

```yaml
anro_haproxy_defaults_errorfile:
  - 400 /etc/haproxy/errors/400.http
  - 403 /etc/haproxy/errors/403.http
  - 408 /etc/haproxy/errors/408.http
  - 500 /etc/haproxy/errors/500.http
  - 502 /etc/haproxy/errors/502.http
  - 503 /etc/haproxy/errors/503.http
  - 504 /etc/haproxy/errors/504.http
```
```yaml
anro_haproxy_defaults_config: []
```

### HAProxy 'peer'
```yaml
anro_haproxy_peer_enabled: false
anro_haproxy_peer_name: haproxy_peer
anro_haproxy_peer_mode: auto
anro_haproxy_peer_port: 1024
anro_haproxy_peer_peers: []
```

Example peer list: 
```yaml
haproxy_peer_peers:
  - hostname1 192.168.15.120:1024
  - hostname2 192.168.15.121:1024
```

### HAProxy 'cache'
```yaml
anro_haproxy_cache_enabled: false
anro_haproxy_cache:
  - name: cache
    total_max_size: 20
    max_object_size: 10485760
    max_age: 120
    process_vary: true
    max_secondary_entries: 10
```

### HAProxy 'frontends'
```yaml
anro_haproxy_frontends: []
```

### HAProxy 'backends'
```yaml
anro_haproxy_backend_default_balance: roundrobin
```
```yaml
anro_haproxy_backends: []
```

### HAProxy 'listens'
```yaml
anro_haproxy_listen_default_balance: roundrobin
```
```yaml
anro_haproxy_listens: []
```
### Example1 playbook

```yaml
---
- name: HAPROXY Example1
  hosts: all
  become: true
  vars:
    ansible_become_pass: vagrant
  # pre_tasks required for cert addition
  pre_tasks:
    - name: Creates directory for certs
      ansible.builtin.file:
        path: /etc/haproxy/certs
        state: directory
        mode: "0751"

    - name: Create private key (RSA, 4096 bits)
      community.crypto.openssl_privatekey:
        path: /etc/haproxy/certs/ansible.pem.key

    - name: Create simple self-signed certificate
      community.crypto.x509_certificate:
        path: /etc/haproxy/certs/ansible.pem
        privatekey_path: /etc/haproxy/certs/ansible.pem.key
        provider: selfsigned

# modeled off https://github.com/haproxy/haproxy/blob/master/examples/basic-config-edge.cfg
  roles:
    - role: anro_haproxy
      vars:
        anro_haproxy_global_chroot: /var/empty
        anro_haproxy_global_settings:
          chroot: /var/empty
          default-path: config
          hard-stop-after: 5m
          ssl-default-bind-ciphers: >-
            ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
          ssl-default-bind-ciphersuites: TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
          ssl-default-bind-options: prefer-client-ciphers no-sslv3 no-tlsv10 no-tlsv11 no-tls-tickets
        anro_haproxy_global_config:
          - zero-warning
        anro_haproxy_global_stats:
          - "socket /var/run/haproxy-svc1.sock level admin mode 660 user haproxy expose-fd listeners"
          - "timeout 1h"
        # override the defaults
        anro_haproxy_defaults_option:
          - httplog
        # override the defaults
        anro_haproxy_defaults_timeout:
          - client 1m
          - server 1m
          - connect 1m
          - http-keep-alive 2m
          - queue 15s
          - tunnel 4h
        anro_haproxy_stats_enabled: true
        anro_haproxy_stats_port: 8181
        anro_haproxy_stats_uri: /
        anro_haproxy_stats_settings:
          - show-modules
        anro_haproxy_cache_enabled: true
        anro_haproxy_frontends:
          - name: pub1
            binds:
              - address: "*"
                port: 80
                options: ["name clear"]
              - address: "*"
                port: 443
                options: ["name secure", "ssl", "crt /etc/haproxy/certs/ansible.pem"]
            default_backend: app1
            force_https: true
            config:
              - 'option socket-stats'
              - 'http-after-response set-header Strict-Transport-Security "max-age=31536000"'
              - " "
              - '# silently ignore connect probes and pre-connect without request'
              - 'option http-ignore-probes'
              - " "
              - '# pass client IP address to the server and prevent against attempts to inject bad contents'
              - 'http-request del-header x-forwarded-for'
              - 'option forwardfor'
              - " "
              - '# enable HTTP compression of text contents'
              - 'compression algo deflate gzip'
              - 'compression type text/ application/javascript application/xhtml+xml image/x-icon'
              - " "
              - '# enable HTTP caching of any cacheable content'
              - 'http-request  cache-use cache'
              - 'http-response cache-store cache'

        anro_haproxy_backends:
          - name: app1
            port: 80
            balance: random
            config:
              - "option httpchk"
              - "http-check send meth GET uri / ver HTTP/1.1 hdr host svc1.example.com"
            cookie: "app1 insert indirect nocache"
            servers:
              - name: srv1
                address: 192.168.255.101
                config: ["cookie s1", "maxconn 100", "check inter 1s"]
              - name: srv2
                address: 192.168.255.102
                config: ["cookie s2", "maxconn 100", "check inter 1s"]
              - name: srv3
                address: 192.168.255.103
                config: ["cookie s3", "maxconn 100", "check inter 1s"]
              - name: srv4
                address: 192.168.255.104
                config: ["cookie s4", "maxconn 100", "check inter 1s"]
```

### Example1 playbook
```yaml
---
- name: HAPROXY Example2
  hosts: all
  become: true
  vars:
    ansible_become_pass: vagrant
  pre_tasks:
    - name: Creates directory for certs
      ansible.builtin.file:
        path: /etc/haproxy/certs
        state: directory
        mode: "0751"

    - name: Create private key (RSA, 4096 bits)
      community.crypto.openssl_privatekey:
        path: /etc/haproxy/certs/ansible.pem.key

    - name: Create simple self-signed certificate
      community.crypto.x509_certificate:
        path: /etc/haproxy/certs/ansible.pem
        privatekey_path: /etc/haproxy/certs/ansible.pem.key
        provider: selfsigned

    - name: Set system hostname
      ansible.builtin.hostname:
        name: "{{ inventory_hostname }}"

  roles:
    - role: anro_haproxy
      vars:
        anro_haproxy_global_settings:
          ssl-server-verify: "none"
          spread-checks: 5
          maxcomprate: 0
          maxcompcpuusage: 100
          maxconn: 60000

        anro_haproxy_defaults_option:
          - dontlognull
          - redispatch
        anro_haproxy_defaults_timeout:
          - connect 5000
          - client 30000
          - server 30000

        # keep our stats
        anro_haproxy_peer_enabled: true
        anro_haproxy_peer_name: "{{ group_names[0] }}"
        anro_haproxy_stats_enabled: true
        anro_haproxy_stats_address: "{{ ansible_host }}"

        # define our frontends
        anro_haproxy_frontends:
          - name: fe-web1
            binds:
              - address: "*"
                port: 80
                options: ["name http"]
              - address: "*"
                port: 443
                options: ["ssl", "crt", "/etc/haproxy/certs/ansible.pem"]

            default_backend: be-web1
            force_https: true
            config:
              - "option http-server-close"
              - "option forwardfor header X-Forwarded-For"
              - "timeout client 120000"
              - "maxconn 20000"

        # define our backends
        anro_haproxy_backends:
          - name: be_web1
            port: 443
            balance: leastconn
            mode: http
            config:
              - "option httpchk"
              - "option http-server-close"
              - "option forwardfor header X-Forwarded-For"
              - "default-server ssl check maxconn 2000"
            servers:
              - name: srv1
                address: 192.168.255.101
                port: 443
              - name: srv2
                address: 192.168.255.102
                port: 443

```