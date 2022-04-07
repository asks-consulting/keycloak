`bin/kc.sh start` without any arguments produces much more informative messages
than when using `--auto-build`. 

This seems to suggest that we should be installing some Java PostgreSQL driver.
Also, the warning about `JdbcDataSource` is interesting.

```
taha@muscat:~
$ /opt/keycloak/bin/kc.sh start 
2022-04-06 03:39:45,563 WARN  [io.quarkus.runtime.configuration.ConfigRecorder] (main) Build time property cannot be changed at runtime:
 - quarkus.datasource.jdbc.driver is set to 'org.h2.jdbcx.JdbcDataSource' but it is build time fixed to 'org.postgresql.xa.PGXADataSource'. Did you change the property quarkus.datasource.jdbc.driver after building the application?
2022-04-06 03:39:45,911 INFO  [org.keycloak.quarkus.runtime.hostname.DefaultHostnameProvider] (main) Hostname settings: FrontEnd: keycloak.asks.se, Strict HTTPS: true, Path: <request>, Strict BackChannel: false, Admin: <request>, Port: -1, Proxied: true
2022-04-06 03:39:45,975 WARN  [io.agroal.pool] (agroal-11) Datasource '<default>': No suitable driver found for jdbc:postgresql://localhost/keycloak
2022-04-06 03:39:45,976 WARN  [org.hibernate.engine.jdbc.env.internal.JdbcEnvironmentInitiator] (JPA Startup Thread: keycloak-default) HHH000342: Could not obtain connection to query metadata: java.sql.SQLException: No suitable driver found for jdbc:postgresql://localhost/keycloak
	at org.h2.jdbcx.JdbcDataSource.getJdbcConnection(JdbcDataSource.java:191)
	at org.h2.jdbcx.JdbcDataSource.getXAConnection(JdbcDataSource.java:352)
	at io.agroal.pool.ConnectionFactory.createConnection(ConnectionFactory.java:216)
	at io.agroal.pool.ConnectionPool$CreateConnectionTask.call(ConnectionPool.java:513)
	at io.agroal.pool.ConnectionPool$CreateConnectionTask.call(ConnectionPool.java:494)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
	at io.agroal.pool.util.PriorityScheduledExecutor.beforeExecute(PriorityScheduledExecutor.java:75)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1126)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)
	at java.base/java.lang.Thread.run(Thread.java:829)

2022-04-06 03:39:46,455 WARN  [io.agroal.pool] (agroal-11) Datasource '<default>': No suitable driver found for jdbc:postgresql://localhost/keycloak
2022-04-06 03:39:46,574 WARN  [org.infinispan.CONFIG] (keycloak-cache-init) ISPN000569: Unable to persist Infinispan internal caches as no global state enabled
2022-04-06 03:39:46,584 WARN  [org.infinispan.PERSISTENCE] (keycloak-cache-init) ISPN000554: jboss-marshalling is deprecated and planned for removal
2022-04-06 03:39:46,594 INFO  [org.infinispan.CONTAINER] (keycloak-cache-init) ISPN000556: Starting user marshaller 'org.infinispan.jboss.marshalling.core.JBossUserMarshaller'
2022-04-06 03:39:46,729 INFO  [org.infinispan.CONTAINER] (keycloak-cache-init) ISPN000128: Infinispan version: Infinispan 'Triskaidekaphobia' 13.0.6.Final
2022-04-06 03:39:46,818 INFO  [org.infinispan.CLUSTER] (keycloak-cache-init) ISPN000078: Starting JGroups channel `ISPN`
2022-04-06 03:39:46,818 INFO  [org.infinispan.CLUSTER] (keycloak-cache-init) ISPN000088: Unable to use any JGroups configuration mechanisms provided in properties {}. Using default JGroups configuration!
2022-04-06 03:39:46,880 WARN  [org.jgroups.protocols.UDP] (keycloak-cache-init) JGRP000015: the send buffer of socket MulticastSocket was set to 1.00MB, but the OS only allocated 212.99KB
2022-04-06 03:39:46,880 WARN  [org.jgroups.protocols.UDP] (keycloak-cache-init) JGRP000015: the receive buffer of socket MulticastSocket was set to 20.00MB, but the OS only allocated 212.99KB
2022-04-06 03:39:46,880 WARN  [org.jgroups.protocols.UDP] (keycloak-cache-init) JGRP000015: the send buffer of socket MulticastSocket was set to 1.00MB, but the OS only allocated 212.99KB
2022-04-06 03:39:46,880 WARN  [org.jgroups.protocols.UDP] (keycloak-cache-init) JGRP000015: the receive buffer of socket MulticastSocket was set to 25.00MB, but the OS only allocated 212.99KB
2022-04-06 03:39:48,888 INFO  [org.jgroups.protocols.pbcast.GMS] (keycloak-cache-init) muscat-61771: no members discovered after 2002 ms: creating cluster as coordinator
2022-04-06 03:39:48,893 INFO  [org.infinispan.CLUSTER] (keycloak-cache-init) ISPN000094: Received new cluster view for channel ISPN: [muscat-61771|0] (1) [muscat-61771]
2022-04-06 03:39:48,896 INFO  [org.infinispan.CLUSTER] (keycloak-cache-init) ISPN000079: Channel `ISPN` local address is `muscat-61771`, physical addresses are `[10.252.116.9:38157]`
2022-04-06 03:39:49,271 INFO  [org.infinispan.CLUSTER] (main) ISPN000080: Disconnecting JGroups channel `ISPN`
2022-04-06 03:39:49,416 ERROR [org.keycloak.quarkus.runtime.cli.ExecutionExceptionHandler] (main) ERROR: Failed to start server in (production) mode
2022-04-06 03:39:49,416 ERROR [org.keycloak.quarkus.runtime.cli.ExecutionExceptionHandler] (main) ERROR: Failed to obtain JDBC connection
2022-04-06 03:39:49,416 ERROR [org.keycloak.quarkus.runtime.cli.ExecutionExceptionHandler] (main) ERROR: No suitable driver found for jdbc:postgresql://localhost/keycloak
2022-04-06 03:39:49,417 ERROR [org.keycloak.quarkus.runtime.cli.ExecutionExceptionHandler] (main) For more details run the same command passing the '--verbose' option. Also you can use '--help' to see the details about the usage of the particular command.
```

This error was very perplexing, only finally solved after I read 
[this comment](https://github.com/keycloak/keycloak/issues/10722#issuecomment-1087520025)
suggesting that it is necessary to *first* `build` before starting keycloak, which worked:

```
taha@muscat:~
$ sudo -u keycloak /opt/keycloak/bin/kc.sh build
Updating the configuration and installing your custom providers, if any. Please wait.
2022-04-07 00:49:32,259 INFO  [io.quarkus.deployment.QuarkusAugmentor] (main) Quarkus augmentation completed in 5226ms
Server configuration updated and persisted. Run the following command to review the configuration:

	kc.sh show-config

taha@muscat:~
$ sudo -u keycloak /opt/keycloak/bin/kc.sh start 
2022-04-07 00:49:40,942 INFO  [org.keycloak.quarkus.runtime.hostname.DefaultHostnameProvider] (main) Hostname settings: FrontEnd: keycloak.asks.se, Strict HTTPS: true, Path: <request>, Strict BackChannel: false, Admin: <request>, Port: -1, Proxied: true
2022-04-07 00:49:41,503 WARN  [org.infinispan.PERSISTENCE] (keycloak-cache-init) ISPN000554: jboss-marshalling is deprecated and planned for removal
2022-04-07 00:49:41,559 WARN  [org.infinispan.CONFIG] (keycloak-cache-init) ISPN000569: Unable to persist Infinispan internal caches as no global state enabled
2022-04-07 00:49:41,589 INFO  [org.infinispan.CONTAINER] (keycloak-cache-init) ISPN000556: Starting user marshaller 'org.infinispan.jboss.marshalling.core.JBossUserMarshaller'
2022-04-07 00:49:41,758 INFO  [org.infinispan.CONTAINER] (keycloak-cache-init) ISPN000128: Infinispan version: Infinispan 'Triskaidekaphobia' 13.0.6.Final
2022-04-07 00:49:41,871 INFO  [org.infinispan.CLUSTER] (keycloak-cache-init) ISPN000078: Starting JGroups channel `ISPN`
2022-04-07 00:49:41,871 INFO  [org.infinispan.CLUSTER] (keycloak-cache-init) ISPN000088: Unable to use any JGroups configuration mechanisms provided in properties {}. Using default JGroups configuration!
2022-04-07 00:49:41,921 WARN  [org.jgroups.protocols.UDP] (keycloak-cache-init) JGRP000015: the send buffer of socket MulticastSocket was set to 1.00MB, but the OS only allocated 212.99KB
2022-04-07 00:49:41,921 WARN  [org.jgroups.protocols.UDP] (keycloak-cache-init) JGRP000015: the receive buffer of socket MulticastSocket was set to 20.00MB, but the OS only allocated 212.99KB
2022-04-07 00:49:41,922 WARN  [org.jgroups.protocols.UDP] (keycloak-cache-init) JGRP000015: the send buffer of socket MulticastSocket was set to 1.00MB, but the OS only allocated 212.99KB
2022-04-07 00:49:41,922 WARN  [org.jgroups.protocols.UDP] (keycloak-cache-init) JGRP000015: the receive buffer of socket MulticastSocket was set to 25.00MB, but the OS only allocated 212.99KB
2022-04-07 00:49:43,928 INFO  [org.jgroups.protocols.pbcast.GMS] (keycloak-cache-init) muscat-62799: no members discovered after 2002 ms: creating cluster as coordinator
2022-04-07 00:49:43,951 INFO  [org.infinispan.CLUSTER] (keycloak-cache-init) ISPN000094: Received new cluster view for channel ISPN: [muscat-62799|0] (1) [muscat-62799]
2022-04-07 00:49:43,962 INFO  [org.infinispan.CLUSTER] (keycloak-cache-init) ISPN000079: Channel `ISPN` local address is `muscat-62799`, physical addresses are `[10.252.116.9:47929]`
2022-04-07 00:49:44,319 INFO  [org.keycloak.connections.infinispan.DefaultInfinispanConnectionProviderFactory] (main) Node name: muscat-62799, Site name: null
2022-04-07 00:49:45,293 INFO  [org.keycloak.quarkus.runtime.storage.database.liquibase.QuarkusJpaUpdaterProvider] (main) Initializing database schema. Using changelog META-INF/jpa-changelog-master.xml
2022-04-07 00:49:49,592 INFO  [org.keycloak.services] (main) KC-SERVICES0050: Initializing master realm
2022-04-07 00:49:50,839 INFO  [io.quarkus] (main) Keycloak 17.0.1 on JVM (powered by Quarkus 2.7.5.Final) started in 11.752s. Listening on: http://0.0.0.0:8080
2022-04-07 00:49:50,839 INFO  [io.quarkus] (main) Profile prod activated. 
2022-04-07 00:49:50,839 INFO  [io.quarkus] (main) Installed features: [agroal, cdi, hibernate-orm, jdbc-h2, jdbc-mariadb, jdbc-mssql, jdbc-mysql, jdbc-oracle, jdbc-postgresql, keycloak, narayana-jta, reactive-routes, resteasy, resteasy-jackson, smallrye-context-propagation, smallrye-health, smallrye-metrics, vault, vertx]
```

Alright, so no JDBC driver was missing. Just needed to build the Keycloak conf
before starting it.

Also seems to work using `--auto-build` (so in a single command, instead of `build` *then* `start`):

```
$ sudo -u keycloak /opt/keycloak/bin/kc.sh start --auto-build
2022-04-07 01:07:18,298 INFO  [org.keycloak.quarkus.runtime.hostname.DefaultHostnameProvider] (main) Hostname settings: FrontEnd: keycloak.asks.se, Strict HTTPS: true, Path: <request>, Strict BackChannel: false, Admin: <request>, Port: -1, Proxied: true
2022-04-07 01:07:18,993 WARN  [org.infinispan.PERSISTENCE] (keycloak-cache-init) ISPN000554: jboss-marshalling is deprecated and planned for removal
2022-04-07 01:07:19,075 WARN  [org.infinispan.CONFIG] (keycloak-cache-init) ISPN000569: Unable to persist Infinispan internal caches as no global state enabled
2022-04-07 01:07:19,096 INFO  [org.infinispan.CONTAINER] (keycloak-cache-init) ISPN000556: Starting user marshaller 'org.infinispan.jboss.marshalling.core.JBossUserMarshaller'
2022-04-07 01:07:19,339 INFO  [org.infinispan.CONTAINER] (keycloak-cache-init) ISPN000128: Infinispan version: Infinispan 'Triskaidekaphobia' 13.0.6.Final
2022-04-07 01:07:19,506 INFO  [org.infinispan.CLUSTER] (keycloak-cache-init) ISPN000078: Starting JGroups channel `ISPN`
2022-04-07 01:07:19,507 INFO  [org.infinispan.CLUSTER] (keycloak-cache-init) ISPN000088: Unable to use any JGroups configuration mechanisms provided in properties {}. Using default JGroups configuration!
2022-04-07 01:07:19,600 WARN  [org.jgroups.protocols.UDP] (keycloak-cache-init) JGRP000015: the send buffer of socket MulticastSocket was set to 1.00MB, but the OS only allocated 212.99KB
2022-04-07 01:07:19,601 WARN  [org.jgroups.protocols.UDP] (keycloak-cache-init) JGRP000015: the receive buffer of socket MulticastSocket was set to 20.00MB, but the OS only allocated 212.99KB
2022-04-07 01:07:19,601 WARN  [org.jgroups.protocols.UDP] (keycloak-cache-init) JGRP000015: the send buffer of socket MulticastSocket was set to 1.00MB, but the OS only allocated 212.99KB
2022-04-07 01:07:19,602 WARN  [org.jgroups.protocols.UDP] (keycloak-cache-init) JGRP000015: the receive buffer of socket MulticastSocket was set to 25.00MB, but the OS only allocated 212.99KB
2022-04-07 01:07:21,613 INFO  [org.jgroups.protocols.pbcast.GMS] (keycloak-cache-init) muscat-6313: no members discovered after 2004 ms: creating cluster as coordinator
2022-04-07 01:07:21,622 INFO  [org.infinispan.CLUSTER] (keycloak-cache-init) ISPN000094: Received new cluster view for channel ISPN: [muscat-6313|0] (1) [muscat-6313]
2022-04-07 01:07:21,627 INFO  [org.infinispan.CLUSTER] (keycloak-cache-init) ISPN000079: Channel `ISPN` local address is `muscat-6313`, physical addresses are `[10.252.116.9:40373]`
2022-04-07 01:07:22,093 INFO  [org.keycloak.connections.infinispan.DefaultInfinispanConnectionProviderFactory] (main) Node name: muscat-6313, Site name: null
2022-04-07 01:07:22,303 INFO  [io.quarkus] (main) Keycloak 17.0.1 on JVM (powered by Quarkus 2.7.5.Final) started in 6.759s. Listening on: http://0.0.0.0:8080
2022-04-07 01:07:22,303 INFO  [io.quarkus] (main) Profile prod activated. 
2022-04-07 01:07:22,304 INFO  [io.quarkus] (main) Installed features: [agroal, cdi, hibernate-orm, jdbc-h2, jdbc-mariadb, jdbc-mssql, jdbc-mysql, jdbc-oracle, jdbc-postgresql, keycloak, narayana-jta, reactive-routes, resteasy, resteasy-jackson, smallrye-context-propagation, smallrye-health, smallrye-metrics, vault, vertx]
^C2022-04-07 01:07:40,698 INFO  [org.infinispan.CLUSTER] (Thread-22) ISPN000080: Disconnecting JGroups channel `ISPN`
2022-04-07 01:07:40,737 INFO  [io.quarkus] (Shutdown thread) Keycloak stopped in 0.097s
```

Now can we make the systemd job start successfully? Yes, we can.
