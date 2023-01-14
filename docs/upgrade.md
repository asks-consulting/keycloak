# Upgrade from Keycloak v17 to v20

We are on version 17, which is starting to get very outdated (v20.0.2 was just
[released](https://www.keycloak.org/2022/12/keycloak-2002-released)).
Since we are still learning how to configure realms/clients etc., we benefit
by not trailing the current documentation too far.


## Installing fresh Keycloak v20 and restoring database

Dump the working Keycloak v17 PSQL database.

```
taha@muscat:~/backups
$ sudo -u postgres psql
psql (12.12 (Ubuntu 12.12-1.pgdg20.04+1))
Type "help" for help.

postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------+----------+-------------+-------------+-----------------------
 keycloak  | keycloak | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =Tc/keycloak         +
           |          |          |             |             | keycloak=CTc/keycloak
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(4 rows)
```

Dump it:
```
taha@muscat:~/backups
$ sudo -u postgres pg_dump -Fc keycloak > keycloak.dump
```
238 KB. Transferred to backup directory on BAY.

+ Backup the LXC container `muscat`. OK.
+ Delete the `muscat` container: `lxc stop muscat; lxc delete muscat`.
+ Run `ansible-playbook playbook-host.yml --ask-become-pass --tags lxd-server` to
  create the a new empty `muscat` container.
+ Set `keycloak.version: "20.0.3"`, `psql_version: "12"`, and `java_version: "17"`
  in `group_vars`, then run the playbook to provision `muscat`:
  `ansible-playbook playbook-containers.yml --ask-become-pass --limit muscat -v`


On the freshly provisioned container, dump the fresh v20 database (to make
reversals easier):
```
taha@muscat:~/backups
$ sudo -u postgres pg_dump -Fc keycloak > keycloak-20.0.3-fresh.dump
```

Replace the Keycloak database with our dump from v17:
```
$ sudo systemctl stop keycloak.service
$ sudo -u postgres dropdb keycloak
$ sudo -u postgres pg_restore -C -d postgres keycloak.dump
```

And now, for the moment of truth. Will a service restart work?
```
taha@muscat:~
$ sudo systemctl restart keycloak.service
$ sudo systemctl status keycloak.service
● keycloak.service - Keycloak server
     Loaded: loaded (/etc/systemd/system/keycloak.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2023-01-14 02:00:40 CET; 3s ago
   Main PID: 797 (java)
      Tasks: 46 (limit: 38014)
     Memory: 312.7M
        CPU: 14.111s
     CGroup: /system.slice/keycloak.service
             └─797 java -Xms64m -Xmx512m -XX:MetaspaceSize=96M -XX:MaxMetaspaceSize=256m -Djava.net.preferIPv4Stack=true -Dfile.encoding=UTF-8 -Dkc.home.dir=/opt/keycloak/bin/.. -Djboss.server.config.dir=/opt/keycloak/bin/../conf -Djava.util.logging.manager=org.jboss.logmanager.LogManager -Dquarkus-log-max-startup-records=10000 -cp /opt/keycloak/bin/../lib/quarkus-run.jar:/opt/keycloak/bin/../lib/bootstrap/* io.quarkus.bootstrap.runner.QuarkusEntryPoint start --optimized

jan 14 02:00:42 muscat kc.sh[797]: 2023-01-14 02:00:42,989 WARN  [org.infinispan.CONFIG] (keycloak-cache-init) ISPN000569: Unable to persist Infinispan internal caches as no global state enabled
jan 14 02:00:42 muscat kc.sh[797]: 2023-01-14 02:00:42,999 INFO  [org.infinispan.CONTAINER] (keycloak-cache-init) ISPN000556: Starting user marshaller 'org.infinispan.jboss.marshalling.core.JBossUserMarshaller'
jan 14 02:00:43 muscat kc.sh[797]: 2023-01-14 02:00:43,109 INFO  [org.keycloak.broker.provider.AbstractIdentityProviderMapper] (main) Registering class org.keycloak.broker.provider.mappersync.ConfigSyncEventListener
jan 14 02:00:43 muscat kc.sh[797]: 2023-01-14 02:00:43,181 INFO  [org.infinispan.CONTAINER] (keycloak-cache-init) ISPN000128: Infinispan version: Infinispan 'Triskaidekaphobia' 13.0.10.Final
jan 14 02:00:43 muscat kc.sh[797]: 2023-01-14 02:00:43,304 INFO  [org.infinispan.CLUSTER] (keycloak-cache-init) ISPN000078: Starting JGroups channel `ISPN`
jan 14 02:00:43 muscat kc.sh[797]: 2023-01-14 02:00:43,304 INFO  [org.infinispan.CLUSTER] (keycloak-cache-init) ISPN000088: Unable to use any JGroups configuration mechanisms provided in properties {}. Using default JGroups configuration!
jan 14 02:00:43 muscat kc.sh[797]: 2023-01-14 02:00:43,382 WARN  [org.jgroups.protocols.UDP] (keycloak-cache-init) JGRP000015: the send buffer of socket MulticastSocket was set to 1.00MB, but the OS only allocated 212.99KB
jan 14 02:00:43 muscat kc.sh[797]: 2023-01-14 02:00:43,382 WARN  [org.jgroups.protocols.UDP] (keycloak-cache-init) JGRP000015: the receive buffer of socket MulticastSocket was set to 20.00MB, but the OS only allocated 212.99KB
jan 14 02:00:43 muscat kc.sh[797]: 2023-01-14 02:00:43,382 WARN  [org.jgroups.protocols.UDP] (keycloak-cache-init) JGRP000015: the send buffer of socket MulticastSocket was set to 1.00MB, but the OS only allocated 212.99KB
jan 14 02:00:43 muscat kc.sh[797]: 2023-01-14 02:00:43,383 WARN  [org.jgroups.protocols.UDP] (keycloak-cache-init) JGRP000015: the receive buffer of socket MulticastSocket was set to 25.00MB, but the OS only allocated 212.99KB
```

Exactly the same problem we saw when migrating in-place from v17 to v18 (error shows
up when loading the Keycloak landing page):
```
kc.sh[797]: WARN  [org.keycloak.services] (executor-thread-2) KC-SERVICES0075: Failed to get theme request: java.lang.RuntimeException: Temporary directory /opt/keycloak/bin/../data/tmp does not exist and it was not possible to create it.
```

Creating this directory (owned by `keycloak` user/group) just leads to another
error (also exactly the same as before):
```
kc.sh[1063]: WARN  [org.keycloak.encoding.GzipResourceEncodingProviderFactory] (executor-thread-2) Failed to create gzip cache directory
kc.sh[1063]: WARN  [org.keycloak.encoding.GzipResourceEncodingProviderFactory] (executor-thread-1) Failed to create gzip cache directory
```

+ [PR to log path of failed cache directory (closed, not merged)](https://github.com/keycloak/keycloak/pull/10347)

Could the section of "hardening" commands in our service file be the cause of
these errors (they appear to be related to file permissions, after all).
Let's comment all of them out.

**Wow, that solved it!** Upgrade appears to be completely successful.

I have found only a few examples of `systemd` service files for Quarkus-based
Keycloak. And they often give contradictory advice.

+ https://www.keycloak.org/server/reverseproxy
+ https://github.com/keycloak/keycloak/issues/10357#issuecomment-1257289831
+ https://github.com/keycloak/keycloak/discussions/10180#discussioncomment-2499174



## Migrating in-place to v18.0.0 (failed)

+ Backup Keycloak (most easily by exporting a tarball of the LXC container).
+ Backup the database (dump psql, or be content with container backup).
+ Copy `conf/`, `providers/` and `themes/` from the previous installation
  to the new installation.

+ https://www.keycloak.org/docs/latest/upgrading/index.html#migrating-to-18-0-0
+ https://www.keycloak.org/docs/latest/upgrading/index.html#_upgrading


Build first (to configure database providers, among other things):
```
taha@muscat:~
$ sudo -u keycloak /opt/keycloak/bin/kc.sh build --db=postgres
Updating the configuration and installing your custom providers, if any. Please wait.
2023-01-13 22:04:38,555 INFO  [io.quarkus.deployment.QuarkusAugmentor] (main) Quarkus augmentation completed in 3820ms
Server configuration updated and persisted.
```

Note that the "server configuration" spoken of above is something else than
`./conf/keycloak.conf`, as timestamps (and contents) show that that file was
not touched by the above command. This "persistent configuration" can be observed:
```
taha@muscat:~
$ sudo -u keycloak /opt/keycloak/bin/kc.sh show-config
Current Mode: none
Runtime Configuration:
	kc.cache =  ispn (PersistedConfigSource)
	kc.config.args =  show-config (SysPropConfigSource)
	kc.db =  postgres (PropertiesConfigSource[source=file:/opt/keycloak/bin/../conf/keycloak.conf])
	kc.db-password =  ******* (PropertiesConfigSource[source=file:/opt/keycloak/bin/../conf/keycloak.conf])
	kc.db-url =  jdbc:postgresql://localhost:5432/keycloak (PropertiesConfigSource[source=file:/opt/keycloak/bin/../conf/keycloak.conf])
	kc.db-username =  keycloak (PropertiesConfigSource[source=file:/opt/keycloak/bin/../conf/keycloak.conf])
	kc.health-enabled =  false (PersistedConfigSource)
	kc.home.dir =  /opt/keycloak/bin/../ (SysPropConfigSource)
	kc.hostname =  keycloak.example.se (PropertiesConfigSource[source=file:/opt/keycloak/bin/../conf/keycloak.conf])
	kc.http-enabled =  true (PropertiesConfigSource[source=file:/opt/keycloak/bin/../conf/keycloak.conf])
	kc.http-host =  0.0.0.0 (PropertiesConfigSource[source=file:/opt/keycloak/bin/../conf/keycloak.conf])
	kc.http-port =  8080 (PropertiesConfigSource[source=file:/opt/keycloak/bin/../conf/keycloak.conf])
	kc.http-relative-path =  / (PersistedConfigSource)
	kc.log-console-output =  default (PropertiesConfigSource[source=jar:file:///opt/keycloak/lib/lib/main/org.keycloak.keycloak-quarkus-server-18.0.0.jar!/META-INF/keycloak.conf])
	kc.log-file =  /opt/keycloak/bin/../data/log/keycloak.log (PropertiesConfigSource[source=jar:file:///opt/keycloak/lib/lib/main/org.keycloak.keycloak-quarkus-server-18.0.0.jar!/META-INF/keycloak.conf])
	kc.metrics-enabled =  false (PersistedConfigSource)
	kc.proxy =  edge (PropertiesConfigSource[source=file:/opt/keycloak/bin/../conf/keycloak.conf])
	kc.quarkus-properties-enabled =  false (PersistedConfigSource)
	kc.show.config =  none (SysPropConfigSource)
	kc.version =  18.0.0 (SysPropConfigSource)
```

Nonetheless, the above build step is absolutely necessary, otherwise the
Keycloak service start fails with this kind of error:
```
taha@muscat:~
$ sudo -u keycloak /opt/keycloak/bin/kc.sh start
2023-01-13 21:49:50,389 WARN  [io.quarkus.runtime.configuration.ConfigRecorder] (main) Build time property cannot be changed at runtime:
 - quarkus.datasource.jdbc.driver is set to 'org.h2.jdbcx.JdbcDataSource' but it is build time fixed to 'org.postgresql.xa.PGXADataSource'. Did you change the property quarkus.datasource.jdbc.driver after building the application?
```

Perform an automatic database migration (is this actually needed?):
Note! Since this command **starts** the server, it never shuts down by itself.
```
taha@muscat:~
$ sudo -u keycloak /opt/keycloak/bin/kc.sh start --spi-connections-jpa-legacy-migration-strategy=update
2023-01-13 22:16:33,189 INFO  [org.keycloak.quarkus.runtime.hostname.DefaultHostnameProvider] (main) Hostname settings: FrontEnd: keycloak.asks.se, Strict HTTPS: true, Path: <request>, Strict BackChannel: false, Admin: <request>, Port: -1, Proxied: true
2023-01-13 22:16:33,710 WARN  [org.infinispan.PERSISTENCE] (keycloak-cache-init) ISPN000554: jboss-marshalling is deprecated and planned for removal
2023-01-13 22:16:33,739 WARN  [org.infinispan.CONFIG] (keycloak-cache-init) ISPN000569: Unable to persist Infinispan internal caches as no global state enabled
2023-01-13 22:16:33,763 INFO  [org.infinispan.CONTAINER] (keycloak-cache-init) ISPN000556: Starting user marshaller 'org.infinispan.jboss.marshalling.core.JBossUserMarshaller'
2023-01-13 22:16:33,915 INFO  [org.infinispan.CONTAINER] (keycloak-cache-init) ISPN000128: Infinispan version: Infinispan 'Triskaidekaphobia' 13.0.8.Final
2023-01-13 22:16:33,985 INFO  [org.infinispan.CLUSTER] (keycloak-cache-init) ISPN000078: Starting JGroups channel `ISPN`
2023-01-13 22:16:33,985 INFO  [org.infinispan.CLUSTER] (keycloak-cache-init) ISPN000088: Unable to use any JGroups configuration mechanisms provided in properties {}. Using default JGroups configuration!
2023-01-13 22:16:34,049 WARN  [org.jgroups.protocols.UDP] (keycloak-cache-init) JGRP000015: the send buffer of socket MulticastSocket was set to 1.00MB, but the OS only allocated 212.99KB
2023-01-13 22:16:34,050 WARN  [org.jgroups.protocols.UDP] (keycloak-cache-init) JGRP000015: the receive buffer of socket MulticastSocket was set to 20.00MB, but the OS only allocated 212.99KB
2023-01-13 22:16:34,050 WARN  [org.jgroups.protocols.UDP] (keycloak-cache-init) JGRP000015: the send buffer of socket MulticastSocket was set to 1.00MB, but the OS only allocated 212.99KB
2023-01-13 22:16:34,050 WARN  [org.jgroups.protocols.UDP] (keycloak-cache-init) JGRP000015: the receive buffer of socket MulticastSocket was set to 25.00MB, but the OS only allocated 212.99KB
2023-01-13 22:16:36,057 INFO  [org.jgroups.protocols.pbcast.GMS] (keycloak-cache-init) muscat-34460: no members discovered after 2002 ms: creating cluster as coordinator
2023-01-13 22:16:36,063 INFO  [org.infinispan.CLUSTER] (keycloak-cache-init) ISPN000094: Received new cluster view for channel ISPN: [muscat-34460|0] (1) [muscat-34460]
2023-01-13 22:16:36,065 INFO  [org.infinispan.CLUSTER] (keycloak-cache-init) ISPN000079: Channel `ISPN` local address is `muscat-34460`, physical addresses are `[10.252.116.9:50505]`
2023-01-13 22:16:36,415 INFO  [org.keycloak.connections.infinispan.DefaultInfinispanConnectionProviderFactory] (main) Node name: muscat-34460, Site name: null
2023-01-13 22:16:37,282 INFO  [org.keycloak.quarkus.runtime.storage.database.liquibase.QuarkusJpaUpdaterProvider] (main) Updating database. Using changelog META-INF/jpa-changelog-master.xml
2023-01-13 22:16:38,498 INFO  [io.quarkus] (main) Keycloak 18.0.0 on JVM (powered by Quarkus 2.7.5.Final) started in 7.109s. Listening on: http://0.0.0.0:8080
2023-01-13 22:16:38,500 INFO  [io.quarkus] (main) Profile prod activated.
2023-01-13 22:16:38,500 INFO  [io.quarkus] (main) Installed features: [agroal, cdi, hibernate-orm, jdbc-h2, jdbc-mariadb, jdbc-mssql, jdbc-mysql, jdbc-oracle, jdbc-postgresql, keycloak, narayana-jta, reactive-routes, resteasy, resteasy-jackson, smallrye-context-propagation, smallrye-health, smallrye-metrics, vault, vertx]
```

+ https://github.com/keycloak/keycloak/issues/13937
+ https://github.com/keycloak/keycloak/issues/14657
+ https://github.com/keycloak/keycloak/issues/12718
