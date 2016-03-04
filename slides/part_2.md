
### Wie benutze ich Docker?

<!-- .slide: data-background="img/background-orange-orig.jpg" -->

- Install
- Define
- Build
- Run
- Compose
- Scale

---

<!-- .slide: data-background="img/background-title-orig.jpg" -->

### Install

1. Download des Docker Toolkit's
2. Installation eines Docker Hosts

---

<!-- .slide: data-background="img/background-title-orig.jpg" -->

### Installation des Docker Hosts

```
➜  ~  docker-machine create --driver virtualbox demo
➜  ~  docker-machine ls
NAME      ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER    ERRORS
demo      -        virtualbox   Running   tcp://192.168.99.100:2376           v1.10.0
➜  ~  eval "$(docker-machine env demo)"
➜  ~  docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
➜  ~
```

Note:
docker-machine kann sich auch zu remote Hosts verbinden

---

<!-- .slide: data-background="img/background-green-orig.jpg" -->

### Define Container Image - Dockerfile

```
FROM java:8

COPY fakeSMTP /usr/local/fakeSMTP

WORKDIR /usr/local/fakeSMTP

EXPOSE 25

CMD ["java", "-jar", "fakeSMTP-2.0.jar", "-s", "-b"]
```

---

<!-- .slide: data-background="img/background-title-orig.jpg" -->

### Build Container Image

```
➜  fakemail  docker build -t fakemail .
Sending build context to Docker daemon 31.01 MB
Step 1 : FROM java:8
 => 736600fd4ae5
Step 2 : COPY fakeSMTP /usr/local/fakeSMTP
 => d88498a06977
Removing intermediate container 1ad6216906fb
...
Step 5 : CMD java -jar fakeSMTP-2.0.jar -s -b
 => Running in 2f2e3255a40d
 => 6fb86d42e1c4
Removing intermediate container 2f2e3255a40d
Successfully built 6fb86d42e1c4
```

---

<!-- .slide: data-background="img/background-green-orig.jpg" -->

### Run Container

```
➜  fakemail  docker run -d -p 2525:25 --name fakemail_instance_1 fakemail
d2eea3b8b8db22093f5e404fdca0487e6de8471e8ebfde70ba0218a409089d61
➜  fakemail  docker logs fakemail_instance_1
18 Feb 2016 08:12:34 INFO  org.subethamail.smtp.server.SMTPServer - SMTP server *:25 starting
18 Feb 2016 08:12:34 INFO  org.subethamail.smtp.server.ServerThread - SMTP server *:25 started
➜  fakemail  docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
d2eea3b8b8db        fakemail            "java -jar fakeSMTP-2"   7 minutes ago       Up 23 seconds       0.0.0.0:2525->25/tcp     fakemail_instance_1
```

---

<!-- .slide: data-background="img/background-title-orig.jpg" -->

### Compose Infrastructure

```
version: '2'

services:
  jboss:
    build: ./jboss
    ports:
     - "8080:8080"
     - "9080:9080"
     - "9990:9990"
  postgres:
    build: ./postgres
    ports:
     - "5432:5432"

volumes:
  logvolume01: {}
```

Note:
Definition einer kompletten Infrastruktur für eine App

---

<!-- .slide: data-background="img/background-title-orig.jpg" -->

### Run composed Infrastructure

```
➜  docker  docker-compose build
➜  docker  docker-compose up -d
Creating network "docker_default" with the default driver
Creating docker_postgres_1
Creating docker_hazelcast_1
Creating docker_felix_1
Creating docker_jboss_1
Creating docker_geoserver_1
Creating docker_mail_1
```

Note:
Es wird ein eigenens isoliertes Netzwerk angelegt

---

<!-- .slide: data-background="img/background-title-orig.jpg" -->

### Stop composed Infrastructure

```
➜  docker  docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                    NAMES
60784cfe6984        docker_mail         "java -jar fakeSMTP-2"   6 minutes ago       Up 5 seconds        0.0.0.0:2525->25/tcp                                                     docker_mail_1
7a9a02502d27        docker_geoserver    "./bin/catalina.sh ru"   6 minutes ago       Up 5 seconds        0.0.0.0:8180->8180/tcp                                                   docker_geoserver_1
f5bdcd2d0c8e        docker_jboss        "/bin/sh -c '/var/lib"   6 minutes ago       Up 5 seconds        0.0.0.0:8080->8080/tcp, 0.0.0.0:9080->9080/tcp, 0.0.0.0:9990->9990/tcp   docker_jboss_1
f6a9bee82be0        docker_hazelcast    "./server.sh"            6 minutes ago       Up 6 seconds        0.0.0.0:5701->5701/tcp, 0.0.0.0:54000->55000/tcp                         docker_hazelcast_1
da3d7d349166        docker_postgres     "/usr/lib/postgresql/"   6 minutes ago       Up 7 seconds        0.0.0.0:5432->5432/tcp                                                   docker_postgres_1
➜  docker  docker-compose down
Stopping docker_mail_1 ... done
Stopping docker_geoserver_1 ... done
...
Stopping docker_postgres_1 ... done
Removing docker_mail_1 ... done
...
Removing docker_postgres_1 ... done
Removing network docker_default
```

---

<!-- .slide: data-background="img/background-green-orig.jpg" -->

### Create Swarm Infrastructure

```
➜  docker  docker-machine create --driver virtualbox manager
➜  docker  docker-machine create --driver virtualbox agent1
➜  docker  docker-machine create --driver virtualbox agent2
➜  docker  eval $(docker-machine env manager)
➜  docker  docker run swarm create
e20e99f163356700d88ec8dd895ea574
➜  docker  docker run -d -p 3376:3376 -t \
 -v /var/lib/boot2docker:/certs:ro swarm manage -H 0.0.0.0:3376 \
 --tlsverify --tlscacert=/certs/ca.pem --tlscert=/certs/server.pem \
 --tlskey=/certs/server-key.pem token://e20e99f163356700d88ec8dd895ea574
➜  docker  eval $(docker-machine env agent1)
➜  docker  docker run -d swarm join --addr=$(docker-machine ip agent1):2376 \
 token://e20e99f163356700d88ec8dd895ea574
```

Note:
eval wird benutzt um den client auf einen anderen Docker Daemon zu zeigen

---

<!-- .slide: data-background="img/background-green-orig.jpg" -->

### Swarm Info (1/2)

```
➜  docker  docker info
...
Role: primary
Strategy: spread
Filters: health, port, dependency, affinity, constraint
Nodes: 2
 agent1: 192.168.99.103:2376
  └ Status: Healthy
  └ Containers: 1
  └ Reserved CPUs: 0 / 1
  └ Reserved Memory: 0 B / 1.021 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=4.1.17-boot2docker, operatingsystem=Boot2Docker 1.10.1 (TCL 6.4.1); master : b03e158 - Thu Feb 11 22:34:01 UTC 2016, provider=virtualbox, storagedriver=aufs
  └ Error: (none)
  └ UpdatedAt: 2016-02-18T09:22:15Z
 agent2: 192.168.99.104:2376
 ...
```

---

<!-- .slide: data-background="img/background-green-orig.jpg" -->

### Swarm Info (2/2)

```
 ...
agent2: 192.168.99.104:2376
  └ Status: Healthy
  └ Containers: 1
  └ Reserved CPUs: 0 / 1
  └ Reserved Memory: 0 B / 1.021 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=4.1.17-boot2docker, operatingsystem=Boot2Docker 1.10.1 (TCL 6.4.1); master : b03e158 - Thu Feb 11 22:34:01 UTC 2016, provider=virtualbox, storagedriver=aufs
  └ Error: (none)
  └ UpdatedAt: 2016-02-18T09:22:38Z
Plugins:
 Volume:
 Network:
Kernel Version: 4.1.17-boot2docker
Operating System: linux
Architecture: amd64
CPUs: 2 Total Memory: 2.043 GiB Name: b42c73c637b8
```

---

<!-- .slide: data-background="img/background-green-orig.jpg" -->

### Scale composed Infrastructure

```
➜  demo  docker-compose up -d
➜  demo  docker-compose scale hazelcast=2
Creating and starting 2 ... done
➜  demo  docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                                         NAMES
aafa51ce8703        docker_hazelcast    "./server.sh"            4 seconds ago       Up 3 seconds                                                                                                      agent1/docker_hazelcast_2
671501576c9f        docker_mail         "java -jar fakeSMTP-2"   2 minutes ago       Up 2 minutes        192.168.99.103:2525->25/tcp                                                                   agent1/docker_mail_1
44251e2fb71a        docker_geoserver    "./bin/catalina.sh ru"   2 minutes ago       Up 2 minutes        192.168.99.103:8180->8180/tcp                                                                 agent1/docker_geoserver_1
d04bab9a5f02        docker_jboss        "/bin/sh -c '/var/lib"   2 minutes ago       Up 2 minutes        192.168.99.103:8080->8080/tcp, 192.168.99.103:9080->9080/tcp, 192.168.99.103:9990->9990/tcp   agent1/docker_jboss_1
980d4ac6b481        docker_hazelcast    "./server.sh"            2 minutes ago       Up 2 minutes        192.168.99.103:5701->5701/tcp, 192.168.99.103:54000->55000/tcp                                agent1/docker_hazelcast_1
1a3454fe328e        docker_postgres     "/usr/lib/postgresql/"   2 minutes ago       Up 2 minutes        192.168.99.103:5432->5432/tcp                                                                 agent1/docker_postgres_1
```