# Nginx with Letsencrypt

This role sets up nginx with letsencrypt.

It tries to be minimal by just creating snippets in
`/etc/nginx/snippets/letsencrypt-{your.domain}.conf` so you can
configure nginx yourself as you which.


## Role Variables

The only mandatory variables are:
- `letsencrypt_certificates`: A list of certificates, each being a list of domain names.
- `letsencrypt_email`: Your email address.

Optional variables are:

- `dhparam_numbits` (`4096`): Number of bits for the DH parameters file.
- `rsa_key_size` (`4096`): Number of bits for the RSA key.
- snippet_prefix (`"letsencrypt-"`): Prefix for nginx snippets, used here: `/etc/nginx/snippets/{{ snippet_prefix }}{{ your.domain.here }}.conf`
- ssl_ciphers (`"ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-SHA256:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES128-GCM-SHA256:AES256+EECDH:AES256+EDH"`): [Specifies the enabled ciphers](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_ciphers).
- ssl_protocols (`"TLSv1.1 TLSv1.2"`): [Enables the specified protocols.](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_protocols)
- ssl_prefer_server_ciphers (`on`): [Specifies that server ciphers should be preferred over client ciphers when using the SSLv3 and TLS protocols.](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_prefer_server_ciphers)
- ssl_session_cache (`shared:ssl_session_cache:10m`): [Sets the types and sizes of caches that store session parameters.](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_session_cache).
- HSTS_header (`Strict-Transport-Security "max-age=63072000; includeSubDomains"`): HTTP header to inject.


## Example Playbook

### One certificate, one domain

Simpliest playbook with a single certificate and a single domain would
look like:

```yaml
- hosts: static_websites
  roles:
    - role: julienpalard.nginx_letsencrypt
      vars:
        letsencrypt_certificates: [[mdk.fr]]
        letsencrypt_email: julien@palard.fr
```

Note the double brackets, it's because we're asking for a *single*
domain in a *single* certificate, see following examples for
clarification.


### One certificate, multiple domains

Just add your domains to the inner list, like:

```yaml
- hosts: static_websites
  roles:
    - role: julienpalard.nginx_letsencrypt
      vars:
        letsencrypt_certificates: [[mdk.fr, www.mdk.fr]]
        letsencrypt_email: julien@palard.fr
```


### Multiple certificates, multiple domains

The following playbook will generate three differnt certificates, the
first one valid for mdk.fr, www.mdk.fr, and julien.palard.fr, the
second one for wyz.fr and www.wyz.fr and the last one valid for tuw.fr
and www.tuz.fr:

```yaml
- hosts: static_websites
  roles:
    - role: julienpalard.nginx_letsencrypt
      vars:
        letsencrypt_certificates:
          - [mdk.fr, www.mdk.fr, julien.palard.fr]
          - [wyz.fr, www.wyz.fr]
          - [tuw.fr, www.tuw.fr]
        letsencrypt_email: julien@palard.fr
```


### As a role dependency

This role fit nicely as another role dependency (in `meta/main.yml`),
but do not use `roles`, use `include_role`, see
[ansible#34736](https://github.com/ansible/ansible/issues/34736):

```yaml
dependencies:
  - role: julienpalard.nginx_letsencrypt
    vars:
      letsencrypt_certificates: [["{{ domain }}"]]
```

Or to concatenate a domain with a list of extra domains:

```yaml
dependencies:
  - role: julienpalard.nginx_letsencrypt
    vars:
      letsencrypt_certificates: ["{{ [domain] | union(extra_certificates) }}"]
```

You can even extract the domain names from attributes of a map:

```yaml
dependencies:
  - role: julienpalard.nginx_letsencrypt
    letsencrypt_certificates: {{ projects|map(attribute='domain')|list }}
```


### License

MIT


### Author Information

Julien Palard — https://mdk.fr
