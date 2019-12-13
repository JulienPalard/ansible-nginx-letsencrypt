Nginx with Letsencrypt
======================

This role sets up nginx with letsencrypt.

It tries to be minimal by just creating snippets in
`/etc/nginx/snippets/letsencrypt-{your.domain}.conf` so you can
configure nginx yourself as you which.


Role Variables
--------------

The only mandatory variable is: `letsencrypt_email`.

Optional variables are:

- `dhparam_numbits` (`4096`): Number of bits for the DH parameters file.
- `rsa_key_size` (`4096`): Number of bits for the RSA key.
- snippet_prefix (`"letsencrypt-"`): Prefix for nginx snippets, used here: `/etc/nginx/snippets/{{ snippet_prefix }}{{ your.domain.here }}.conf`
- ssl_ciphers (`"ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-SHA256:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES128-GCM-SHA256:AES256+EECDH:AES256+EDH"`): [Specifies the enabled ciphers](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_ciphers).
- ssl_protocols (`"TLSv1.1 TLSv1.2"`): [Enables the specified protocols.](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_protocols)
- ssl_prefer_server_ciphers (`on`): [Specifies that server ciphers should be preferred over client ciphers when using the SSLv3 and TLS protocols.](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_prefer_server_ciphers)
- ssl_session_cache (`shared:ssl_session_cache:10m`): [Sets the types and sizes of caches that store session parameters.](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_session_cache).
- HSTS_header (`Strict-Transport-Security "max-age=63072000; includeSubDomains"`): HTTP header to inject.


Example Playbook
----------------

This role fits nicely as another role dependency, like in a `meta/main.yml`:

```
dependencies:
  - role: nginx-letsencrypt
    domains: ["{{ domain }}"]
```

Don't forget the `[]`, as `domains` is a list, you can create multiple HTTPS certificates at the same time, like:

```yaml
dependencies:
  - role: nginx-letsencrypt
  domains: {{ projects|map(attribute='fqdn')|list }}
```

But it could also work standalone in a playbook:

```yaml
- hosts: my_pelican
  roles:
    - { role: letsencrypt, domains: [mdk.fr] }
```


License
-------

MIT


Author Information
------------------

Julien Palard — https://mdk.fr
