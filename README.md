
# letsencrypt

An Ansible role that creates or renews a cert with https://letsencrypt.org/ and optionally installs it to servers. Does not require root, or installation of any software on the remote host.

## Requirements

You need to have `letsencrypt` installed in your *local* environment path, or set `letsencrypt_command`.

You need to have a proxy setup on your web host(s), something like:

    <IfModule mod_proxy.c>
      ProxyPass "/.well-known/acme-challenge/" "http://{{letsencrypt.bind}}:{{letsencrypt_local_port}}/.well-known/acme-challenge/" retry=1
      ProxyPassReverse "/.well-known/acme-challenge/" "http://{{letsencrypt.bind}}:{{letsencrypt_local_port}}/.well-known/acme-challenge/"

      <Location "/.well-known/acme-challenge/">
        ProxyPreserveHost On
        Order allow,deny
        Allow from all
        Require all granted
      </Location>
    </IfModule>


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
    # if set, will use this @$domain for email address, overridable per domain
    letsencrypt_email_user: root
    # create cert locally for later sync
    letsencrypt_local: True
    # local port
    letsencrypt_local_port: 7010
    # always push certs to server
    letsencrypt_force_install: True
    letsencrypt_server: https://acme-v01.api.letsencrypt.org/directory
    letsencrypt_command: letsencrypt
    letsencrypt_dir: letsencrypt
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

## License

Apache

## Author Information

https://github.com/20c/ansible-role-letsencrypt

