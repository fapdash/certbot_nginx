Certbot NGINX
=========

Simple Ansible role to install `certbot` with NGINX plugin on **Ubuntu 16.04**.

This role will:
1. Add `certbot` PPA repository
2. Install `certbot` and `python-certbot-nginx` packages
3. `certbot` package will add a `renew` cron job and a systemd-timer ([More info](https://certbot.eff.org/#ubuntuxenial-nginx)
4. Generate a Let's Encrypt SSL certificates for the given `domain_name` or `domain_names`.

Also allow to generate and manage multiple certificates in the same host.
You can define the var `domain_names` and configure a role to incude the this role in a loop. See [Multiple certificates creation](#example-playbook---multiple-certificates-creation) section.

Role Variables
--------------
```yaml
domain_name: www.mydomain.io
letsencrypt_email: myaccount@letsencrypt.org
certbot_nginx_cert_name: mycert # optional
```

if set, `certbot_nginx_cert_name`'s value will be passed to the certbot's `--cert-name` argument, which is used to identify the certificate in certbot command such as `certbot delete`. You will see a list of certificates identified with this name by running `certbot certificates`. This name will also be used as the file paths for the certificate in `/etc/letsencrypt/live/`.

Example Playbook - Single certificate
----------------

```yaml
# Playbook
- hosts: servers
  roles:
    - role: coopdevs.certbot-nginx
      vars:
        domain_name: www.mydomain.io
        letsencrypt_email: myaccount@letsencrypt.org
```

Example Playbook - Multiple certificates creation
-----------------------------------------

```yaml
# Playbook
- hosts: servers
  roles:
    - role: coopdevs.certbot-nginx
      vars:
        letsencrypt_email: myaccount@letsencrypt.org
    - role: certificates
      vars:
        domain_names:
          - community.coopdevs.org
          - forms.coopdevs.org
```

Create a custom role including the `certbot_nginx` role that generates the certificates:

```yaml
# certificates.yml Role
---
- name: Install SSL certificates
  include_role:
    name: vendor/coopdevs.certbot_nginx
    tasks_from: certificate.yml
  with_items: "{{ domain_names }}"
  loop_control:
    loop_var: domain_name
```

> You need to declare the `loop_control` to map the `item` var of the `with_item` loop with the `loop_var` value as `domain_name`. See the [`loop_controll` doc](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html?highlight=loop_control#loop-control)

Let's Encrypt Staging Environment
---------------------------------

This role includes `letsencrypt_staging` variable which defaults to `no`. For development or debugging purposes, one can set it to `yes`,
for example by [Passing Variables On The Command Line](http://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#passing-variables-on-the-command-line) `--extra-vars "letsencrypt_staging=yes"`

This will result in use of [Let's Encrypt Staging Environment](https://letsencrypt.org/docs/staging-environment/) and reducing chance of
running up against rate limits.

License
-------

BSD

Author Information
------------------

Coopdevs http://coopdevs.org
