# workspace_setup: Okruzenje PotsgreSQL, PostgREST, OmniDB, PGADMIN

## Postgresql

```
docker run -d \
    --restart=always\
    --name=postgres \
    -p 5432:5432 \
    -e POSTGRES_PASSWORD=xxx \
    -e PGDATA=/var/lib/postgresql/data/pgdata \
    -v /opt/kontenjeri/postgres/data:/var/lib/postgresql/data \
    postgres
```

Postresql 9
```
docker run -d \   
    -p 5439:5432 \                                                                                             
    --name postgres9 \                                                                                     
    -e POSTGRES_PASSWORD=xxx \            
    -e PGDATA=/var/lib/postgresql/data/pgdatapg9 \
    -v /opt/kontenjeri/postgres/datapg9:/var/lib/postgresql/data \
    postgres:9
```

## PostgREST




## DB managament apps

### OmniDB

http://X.X.X.X:8000/
admin:xxx

Instalacija:
```
cat /etc/systemd/system/omnidb.service 
[Unit]
Description=OmniDB server daemon
After=network.target

[Service]
Type=forking
ExecStart=/bin/bash -c "/home/omnidb/omnidb-server/omnidb-server &"
RemainAfterExit=yes
User=omnidb
Group=omnidb

[Install]
WantedBy=multi-user.target
```

### Pgadmin4

http://X.X.X.X:5050/
XXX@xxx.rs:xxx

```
docker run -p 5050:80 \
    -e 'PGADMIN_DEFAULT_EMAIL=XXX@xxx.rs' \
    -e 'PGADMIN_DEFAULT_PASSWORD=xxx' \
    -e 'PGADMIN_CONFIG_ENHANCED_COOKIE_PROTECTION=True' \
    -e 'PGADMIN_CONFIG_LOGIN_BANNER="Authorised users only!"' \
    -e 'PGADMIN_CONFIG_CONSOLE_LOG_LEVEL=10' \
    --restart=always \
    --name=pgadmin4 \
    -d dpage/pgadmin4
```

### Swagger

```
docker run -d --name=swagger --restart=always -p 8080:8080 -e "API_URL=http://X.X.X.X:3000/" swaggerapi/swagger-ui
```


### selinux stuf
```

semanage  fcontext -a -t postgresql_log_t '/mnt/data/postgres13/*\.log'
restorecon -R -v /mnt/data/postgres13/data


cat /etc/selinux/targeted/contexts/files/file_contexts | grep postgres
cat /etc/selinux/targeted/contexts/files/file_contexts.local

[root@invtest files]# cat file_contexts.local
# This file is auto-generated by libsemanage
# Do not edit directly.

/var/lib/pgadmin(/.*)?    system_u:object_r:httpd_var_lib_t:s0
/var/log/pgadmin(/.*)?    system_u:object_r:httpd_log_t:s0

### for rpm pg installation
/mnt/data/postgres13/data(/.*)?    system_u:object_r:postgresql_db_t:s0
/mnt/data/postgres13/data/log(/.*)?    system_u:object_r:postgresql_log_t:s0
/mnt/data/postgres13/data/*\.log    system_u:object_r:postgresql_log_t:s0
/mnt/data/postgres13/*\.log    system_u:object_r:postgresql_log_t:s0
/mnt/data/postgres13/pgstartup.log    system_u:object_r:postgresql_log_t:s0

## for tomcat installation by hand
/opt/apache-tomcat-10.0.6(/.*)?    system_u:object_r:tomcat_var_lib_t:s0
/opt/apache-tomcat-10.0.6/temp/.*\.pid    system_u:object_r:tomcat_var_run_t:s0
/opt/apache-tomcat-10.0.6/.*\.pid    system_u:object_r:tomcat_var_run_t:s0
/opt/apache-tomcat-10.0.6/.*\.log    system_u:object_r:tomcat_log_t:s0
/opt/apache-tomcat-10.0.6/log(s)?(/.*)?    system_u:object_r:tomcat_log_t:s0
/opt/apache-tomcat-10.0.6/bin(/.*)?    system_u:object_r:bin_t:s0

```


### Tomcat

systemd example
```
[Unit]
Description=Apache Tomcat Web Application Container
Wants=network.target
After=network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.8.10-0.el8_2.x86_64

Environment=CATALINA_PID=/usr/local/tomcat9/temp/tomcat.pid
Environment=CATALINA_HOME=/usr/local/tomcat9
Environment='CATALINA_OPTS=-Xms512M -Xmx1G -Djava.net.preferIPv4Stack=true'
Environment='JAVA_OPTS=-Djava.awt.headless=true'

ExecStart=/usr/local/tomcat9/bin/startup.sh
ExecStop=/usr/local/tomcat9/bin/shutdown.sh
SuccessExitStatus=143

User=tomcat
Group=tomcat
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target

```


clean example

```
[Unit]
Description=Tomcat Server
After=syslog.target network.target

[Service]
Type=forking
User=tomcat
Group=tomcat

Environment=JAVA_HOME=/usr/lib/jvm/jre
Environment='JAVA_OPTS=-Djava.awt.headless=true'
Environment=CATALINA_HOME=/usr/share/tomcat
Environment=CATALINA_BASE=/usr/share/tomcat
Environment=CATALINA_PID=/usr/share/tomcat/temp/tomcat.pid
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M'
ExecStart=/usr/share/tomcat/bin/catalina.sh start
ExecStop=/usr/share/tomcat/bin/catalina.sh stop

[Install]
WantedBy=multi-user.target
```

```
sudo useradd -r tomcat


sudo ln -s /usr/local/tomcat/apache-tomcat-9.0.33 /usr/local/tomcat/tomcat

sudo vi /usr/share/tomcat/conf/tomcat-users.xml
```
 /usr/local/tomcat/conf/tomcat-users.xml
```
<role rolename="manager-gui"/>
<role rolename="admin-gui"/>
<user username="username" password="password" roles="manager-gui,admin-gui"/>

```

 /usr/local/tomcat/webapps/manager/META-INF/context.xml
```
<!--
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
-->
```

### change server limits 
```
cat /etc/systemd/system/mariadb.service.d/limits.conf 
[Service]
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
```
