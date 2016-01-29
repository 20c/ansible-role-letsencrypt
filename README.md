
# Ansible Role for letsencrypt

An Ansible role that creates or renews a certificate with https://letsencrypt.org/ and optionally installs it to servers. Does not require root, stopping any processes, or installation of any software on the remote host(s).

This method currently only supports HTTP/S validation, it uses a proxy from your public facing website (probably over a private management network) to avoid installing or running anything extra on production servers.

Generated certificates will be put in `letsencrypt_dir` (default `gen/letsencrypt`), specifying `letsencrypt_install_dir` will make it also push to all hosts.

## Requirements

You need to have `letsencrypt` installed in your *local* environment path, or set variable `letsencrypt_command`.

To install either follow [their directions](http://letsencrypt.readthedocs.org/en/latest/using.html), or to install directly to the current venv (I wasn't able to get it to find the correct deps for a direct from git install, hopefully it'll work from pypi soon):

    git clone https://github.com/letsencrypt/letsencrypt
    cd letsencrypt
    pip install -e ../c/gh/letsencrypt/
    pip install -e ../c/gh/letsencrypt/acme/

## Proxy Setup

With ansible it's quite easy to configure a proxy from all your front end servers back to the deploy machine. Grab the address from the environment and apply where needed, some examples:

#### Apache, from config file template

    {% set ansible_local_host = ansible_env.SSH_CONNECTION.split(' ')[0] -%}
    <IfModule mod_proxy.c>
      ProxyPass "/.well-known/acme-challenge/" "http://{{ansible_local_host}}:{{letsencrypt_local_port}}/.well-known/acme-challenge/" retry=1
      ProxyPassReverse "/.well-known/acme-challenge/" "http://{{ansible_local_host}}:{{letsencrypt_local_port}}/.well-known/acme-challenge/"

      <Location "/.well-known/acme-challenge/">
        ProxyPreserveHost On
        Order allow,deny
        Allow from all
        Require all granted
      </Location>
    </IfModule>

#### Nginx, from config file template
    {% set ansible_local_host = ansible_env.SSH_CONNECTION.split(' ')[0] -%}
    server {
       listen 80;
       location /.well-known/acme-challenge/ {
           proxy_pass http://{{ansible_local_host}}:{{letsencrypt_local_port}}/;
           rewrite /(.*) /$1 break;
       }
    }

#### Nginx, from ansible role [jdauphant.nginx](https://galaxy.ansible.com/jdauphant/nginx/)

    nginx_sites:
      redir80:
        - listen 80
        - "location /.well-known/acme-challenge/ {
            proxy_pass http://{{ansible_env.SSH_CONNECTION.split(' ')[0]}}:{{letsencrypt_local_port}}/;
            rewrite /(.*) /$1 break;
            }"

## Role Variables

The only required variable is `letsencrypt_certs` which must be a list of dicts that contain at least `domain` and may also contain `san` as a lit of Subject Alternative Names for the cert, and email if you want to override the cert email.

    letsencrypt_certs:
      - domain: example.net
    # optional:
        sans: [www.example.net, example.com, www.example.com]
        email: admin@example.net
      - domain: example.org
        sans: [www.example.org]

If you want it to install the certs you must set `letsencrypt_install_dir`.
    letsencrypt_install_dir: /srv/certs/

Default variables

    # will use this @$domain for email address, overridable per domain
    letsencrypt_email_user: root

    # create cert locally for later sync
    letsencrypt_local: True

    # local port
    letsencrypt_local_port: 7010

    # always push certs to server
    letsencrypt_force_install: True

    letsencrypt_server: https://acme-v01.api.letsencrypt.org/directory
    letsencrypt_command: letsencrypt
    letsencrypt_dir: gen/letsencrypt
    letsencrypt_config_dir: "{{letsencrypt_dir}}/config"
    letsencrypt_work_dir: "{{letsencrypt_dir}}/tmp"
    letsencrypt_logs_dir: "{{letsencrypt_dir}}/log"

    # renew the cert when expiration is within this number of seconds, default is 2 weeks
    letsencrypt_renew_at: 1209600
    letsencrypt_cert_file: fullchain.pem

## Example Playbook

    - hosts: servers
      roles:
        - role: letsencrypt
          letsencrypt_install_dir: /srv/cert/
          letsencrypt_certs:
            - domain: example.net
              sans: [www.example.net, example.com, www.example.com]
              email: admin@unitedix.net

## Maintenance

Uses tag `letsencrypt` to only run these tasks, and will automatically renew if the certificate will expire in less than `letsencrypt_renew_at` seconds.

So as an example command to keep your certs renewed from a nightly crontab would be:

    ansible-playbook -i prod site.yml --tag=letsencrypt

## License

Apache

## Author Information

https://github.com/20c/ansible-role-letsencrypt

