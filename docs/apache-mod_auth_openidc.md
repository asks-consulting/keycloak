# Apache config

This Apache virtualhost config seems to work.
Apache 2.4.29 on Ubuntu Bionic;
Keycloak 17.0.1 (Quarkus) with Java 17 on an Ubuntu Focal container.

The web service runs on its own container in the same VLAN
as Keycloak and the Apache reverse proxy.
Only the proxy is listening to remote connections (and terminates TLS traffic).
Pretty straight-forward and a very common setup for self-hosted web services
in small-office networks.

Ansible facts:

```
vhost:
  domain: "awesome.example.se"
  ip: "192.168.1.22"
  port: "8000"
  email: "webmaster@example.se"
  clientid: "awesome-clientid"
  clientsecret: "longandsecretstring"
  oidccrypto: "anotherlongandsecretstring"
keycloak:
  host: keycloak.example.se
  realm: demo
```

Apache vhost config:

```
<VirtualHost *:80>
   ServerAdmin {{ vhost.email }}
   ServerName {{ vhost.domain }}
   Redirect permanent / https://{{ vhost.domain }}/
   ErrorLog ${APACHE_LOG_DIR}/{{ vhost.domain }}_error.log
   CustomLog ${APACHE_LOG_DIR}/{{ vhost.domain }}_access.log combined
</VirtualHost>

<VirtualHost *:443>
   ServerAdmin {{ vhost.email }}
   ServerName {{ vhost.domain }}

   ProxyRequests Off
   ProxyPreserveHost On
   <Proxy *>
      Order deny,allow
      Allow from all
   </Proxy>
   <Location />
      ProxyPass http://{{ vhost.ip }}:{{ vhost.port }}/
      ProxyPassReverse http://{{ vhost.ip }}:{{ vhost.port }}/
   </Location>

   # Keycloak authentication
   OIDCProviderMetadataURL https://{{ keycloak.host }}/realms/{{ keycloak.realm }}/.well-known/openid-configuration
   OIDCClientID {{ vhost.clientid }}
   OIDCClientSecret {{ vhost.clientsecret }}
   OIDCCryptoPassphrase {{ vhost.oidccrypto }}
   OIDCRedirectURI https://{{ vhost.domain }}/oauth2callback
   <Location />
      AuthType openid-connect
      Require valid-user
      # LogLevel debug
   </Location>

   SSLEngine on
   SSLCertificateKeyFile   /etc/letsencrypt/live/{{ vhost.domain }}/privkey.pem
   SSLCertificateFile      /etc/letsencrypt/live/{{ vhost.domain }}/cert.pem
   SSLCertificateChainFile /etc/letsencrypt/live/{{ vhost.domain }}/chain.pem
   ErrorLog ${APACHE_LOG_DIR}/{{ vhost.domain }}_error.log
   CustomLog ${APACHE_LOG_DIR}/{{ vhost.domain }}_access.log combined
</VirtualHost>
```
