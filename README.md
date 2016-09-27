Ansible Role: nginx-site
========================

[![Build Status](https://travis-ci.org/gronke/ansible-nginx-site.svg?branch=add-travis-tests)](https://travis-ci.org/gronke/ansible-nginx-site)

This Role is used to configure individual Nginx vhost-sites stored in `/etc/nginx/sites-available/`

## Configuration

### General

| Parameter       | Required   | Default         | Options / Comment                                         |
| --------------- | ---------- | --------------- | --------------------------------------------------------- |
| site_name       | no         | "example.org"   |                                                           |
| ip_version      | no         | both            | `both`, `IPv4`, `IPv6`                                    |
| http_port       | no         | 80              | 0 - 65535                                                 |
| https_port      | no         | 443             | 0 - 65535                                                 |
| aliases         | no         | [{{site_name}}] | List of strings with domain names (Wildcard `*`)          |
| log_access_file | no         | `/var/log/nginx/{{site_name}}_access.log` |                                 |
| log_error_file  | no         | `/var/log/nginx/{{site_name}}_error.log` |                                  |
| log_level       | no         | error   | `debug`, `info`, `notice`, `warn`, `error`, `crit`, `alert`, or `emerg` |
| nginx_disable_default_site | no | `true`       | `true` disables the default nginx vhost                   |


### Encryption

When encryption is not explicitly disabled all `ssl_` prefixed options are required. The default values harmonize with the [ansible-letsencrypt](https://github.com/jaywink/ansible-letsencrypt) role by @jaywink.


| Parameter                    | Required  | Default         | Options / Comment                                         |
| ---------------------------- | --------- | --------------- | --------------------------------------------------------- |
| encryption                   | no        | "redirect"      | `force`, `redirect`, `optional`, `off`                    |
| ssl_ciphers                  | **yes**   | `EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH` | [[see](https://raymii.org/s/tutorials/Strong_SSL_Security_On_nginx.html#The_Cipher_Suite)] |
| ssl_protocols                | **yes**   | `TLSv1 TLSv1.1 TLSv1.2` | [[see](https://raymii.org/s/tutorials/Strong_SSL_Security_On_nginx.html#SSLv2_and_SSLv3)] |
| ssl_certificate_path         | **yes**   | `/etc/letsencrypt/live/{{site_name}}/fullchain.pem` | defaults to LetsEncrypt |
| ssl_trusted_certificate_path | **yes**   | `/etc/letsencrypt/live/{{site_name}}/fullchain.pem` | defaults to LetsEncrypt, needed for [OCSP Stapling](https://raymii.org/s/tutorials/Strong_SSL_Security_On_nginx.html#OCSP_Stapling) |
| ssl_key_path                 | **yes**   | `/etc/letsencrypt/live/{{site_name}}/privkey.pem` | defaults to LetsEncrypt |
| ssl_dh_size                  | **yes**   | 4096      | *this will take a while to generate*                          |
| ssl_dh_file                  | **yes**   | `/etc/ssl/certs/dhparam-{{ ssl_dh_size }}.pem` | *consider pre-generating this file* |
|ssl_hsts_max_age              | **yes**   | 604800 (1 week) | `encryption` must be set to `force` or `redirect`    |                                                        |
|ssl_hsts_enabled              |   no     | `true` |  This is only enabled when `encryption` is set to `force` or `redirect` |
|

### Features

Each vhost-site can have it's own purpose, while only one feature at a time can be used. (To build more complex configurations, custom templates are the better option.)

All options are grouped in a dict structure called `features`.

#### serve_htdocs

Enabling this feature configures the vhost to serve static content

```yaml
- role: gronke.nginx-site
  features:
    serve_htdocs:
      document_root: /var/www
```

optionally PHP can be installed and enabled too

```yaml
- role: gronke.nginx-site
  features:
    serve_htdocs:
      document_root: /var/www
      php: true
      index: 'index.html index.php'
```

#### proxy

Incoming requests are proxied to a different http(s) server. Very useful when the Nginx vhost is acting as SSL proxy for other services.

```yaml
- role: gronke.nginx-site
  features:
    proxy:
      target: 'http://example.com'
      rewrite_rules:
        - '^/foo(.*)$ /bar$1'
```

#### seafile_fasgcgi

Seafile wants a lot extra configuration. This feature is planned to be deprecated in future versions and replaced with a more generic solution for complex configurations.

```yaml
- role: gronke.nginx-site
  features:
    seafile_fastcgi:
      seafile_org_name: 'My Organization'
```
