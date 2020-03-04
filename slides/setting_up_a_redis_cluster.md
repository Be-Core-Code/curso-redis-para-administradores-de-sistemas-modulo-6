### Configurando Redis Cluster

Vamos a desplegar un Redis Cluster a mano con objeto de aprender y entender los diferentes aspectos operativos del mismo.

Para levantarlo de forma más rápida, podemos utilizar el script `create-cluster` que usaremos en la siguiente sección 
de este módulo. 

^^^^^^

#### 💻️ Configurando Redis Cluster

![](/slides/images/setting_up_a_redis_cluster/setting_up_a_redis_cluster.001.jpeg)<!-- .element: style="height: 60vh" -->

notes:

Este es el cluster que queremos montar.

^^^^^^

#### 💻️ Configurando Redis Cluster

Para esto necesitaremos levantar tres máquinas virtuales y dentro de cada una de ellas dos instancias de Redis:

* Una en el puerto 6379, que hará de maestro
* Otra en el puerto 6380, que hará de réplica de un maestro situado en otra máquina virtual

^^^^^^

#### 💻️ Configurando Redis Cluster

A continuación tenéis un extracto de la configuración mínima de Redis para que opere en modo cluster:

```bash
port 6379
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

notes:

En nuestro caso, vamos a necesitar una configuración un poco más detallada ya que queremos ejecutar 
dos instancias en cada máquina. En la siguiente diapositiva veréis enlaces a las configuraciones 
completas de las instancias de Redis.
 
^^^^^^

#### 💻️ Configurando Redis Cluster

Dado que en nuestras máquinas vamos a levantar dos instancias de Redis, necesitaremos dos ficheros de configuración:

* [`/etc/redis.conf`](https://github.com/Be-Core-Code/curso-redis-para-administradores-de-sistemas-vagrant/blob/master/cluster/redis-nodea-master.conf): 
  configura la instancia de redis que actuará como maestro
* [`/etc/redis-replica.conf`](https://github.com/Be-Core-Code/curso-redis-para-administradores-de-sistemas-vagrant/blob/master/cluster/redis-nodea-replica.conf):
  configura la instancia de redis que actuará como réplica

^^^^^^

#### 💻️ Configurando Redis Cluster

De igual manera, necesitaremos un [script de arranque](https://github.com/Be-Core-Code/curso-redis-para-administradores-de-sistemas-vagrant/blob/master/cluster/redis-replica.rc) 
de la segunda instancia de redis.

^^^^^^

#### 💻️ Configurando Redis Cluster

En el siguiente repositorio:

[https://github.com/Be-Core-Code/curso-redis-para-administradores-de-sistemas-vagrant](https://github.com/Be-Core-Code/curso-redis-para-administradores-de-sistemas-vagrant)

disponéis de la configuración para levantar las tres máquinas virtuales con la configuración inicial de
las instancias de Redis.

^^^^^^

#### 💻️ Configurando Redis Cluster

La configuración [Vagrant para el cluster](https://github.com/Be-Core-Code/curso-redis-para-administradores-de-sistemas-vagrant/blob/master/cluster/Vagrantfile) 
se encarga de levantar las tres máquinas e incluir todos los ficheros de configuración dentro de cada nodo.

^^^^^^

#### 💻️ Configurando Redis Cluster

Levantamos el cluster usando Vagrant:

```bash
cluster > vagrant up 
```

Y nos conectamos a cualquiera de los nodos:

```bash 
cluster > vagrant ssh nodea
```

^^^^^^

#### 💻️ Configurando Redis Cluster

Podremos ver en el log del maestro `/var/log/redis/redis-6379.log` y de la réplica `/var/log/redis/redis-6380.log`
que las instancias se han levantado en modo cluster:

```bash
# `/var/log/redis/redis-6379.log`
...
2855:M 04 Mar 2020 12:27:22.262 * No cluster configuration found, I'm 3ed7305206f547ec70f331b9e93ff5e8345df1a9
2855:M 04 Mar 2020 12:27:22.377 * Running mode=cluster, port=6379.
...
```

notes:

Si apagamos las máquinas y las volvemos a encender, dado que existirán los ficheros `/var/lib/redis/nodes-*.conf`
estos mensajes no aparecerán.

^^^^^^

#### 💻️ Configurando Redis Cluster

Dentro de este nodo (`nodea`) también podemos ver los dos ficheros de configuración del cluster que se han
creado en la carpeta `/var/lib/redis`:

```bash
nodea > ls /var/lib/redis
-rw-r--r--    1 redis    redis          114 Mar  4 12:27 nodes-6379.conf
-rw-r--r--    1 redis    redis          114 Mar  4 12:27 nodes-6380.conf
```

^^^^^^

#### 💻️ Configurando Redis Cluster

Si miramos dentro de uno de esos ficheros de configuración, veremos lo siguiente:

```bash
nodea > cat /var/lib/redis/nodes-6379.conf
3ed7305206f547ec70f331b9e93ff5e8345df1a9 :0@0 myself,master - 0 0 0 connected
vars currentEpoch 0 lastVoteEpoch 0
```

^^^^^^

#### 💻️ Configurando Redis Cluster

Y si ejecutamos el comando [`INFO`](https://redis.io/commands/info) veremos que todos los nodos están en modo cluster:

```bash
nodea > redis-cli -h 192.168.158.100 -p 6379 INFO Cluster
# Cluster
cluster_enabled:1
nodea > redis-cli -h 192.168.158.100 -p 6380 INFO Cluster
# Cluster
cluster_enabled:1
```

^^^^^^

#### 💻️ Configurando Redis Cluster

Todos esto que hemos descrito para el nodo `nodea`, se describe de la misma manera para los nodos `nodeb` y `nodec`

^^^^^^

#### 💻️ Configurando Redis Cluster

Preparados los nodos, lo siguiente que necesitamos es crear el cluster con ellos. Para hacerlo utilizamos el comando
[`CLUSTER MEET`](https://redis.io/commands/cluster-meet).

La siguiente operación la llevamos a cabo desde cualquier instancia de redis (por ejemplo, en el nodo `nodec`)

```bash
(nodec) > redis-cli -h 192.168.158.102 -p 6380
```

^^^^^^

#### 💻️ Configurando Redis Cluster


```redis-cli
redis-cli (nodec:6380) > CLUSTER MEET 192.168.158.100 6379
OK
redis-cli (nodec:6380) > CLUSTER MEET 192.168.158.100 6380
OK
redis-cli (nodec:6380) > ...
  
```

^^^^^^

#### 💻️ Configurando Redis Cluster

Después de ejecutar estos comandos, podemos ver cuál es el contenido de los ficheros `/var/lib/redis/nodes-*.conf`:

```bash
a46334a276c871fa95b1fd5b90f006518dadec7d 192.168.158.101:6379@16379 master - 0 1583333560298 3 connected
33bc4dc83aa3a11c86e35bb0ca84a3ab84280c79 192.168.158.102:6380@16380 master - 0 1583333559994 1 connected
a30ba6afb3b76414b51a8a57542aa76f753b1da4 192.168.158.100:6379@16379 master - 0 1583333560096 2 connected
2125eb65a7fef9af62fd90774db0614587f44464 192.168.158.102:6379@16379 myself,master - 0 1583333560000 5 connected
fdfe15734fd5d82d6d3b108b21800a0f7346a41a 192.168.158.101:6380@16380 master - 0 1583333560298 4 connected
fe233b965198b91a36134a887705bf848137ea74 192.168.158.100:6380@16380 master - 0 1583333560097 0 connected
vars currentEpoch 5 lastVoteEpoch 0
```

notes:

Vemos que se enumeran todos los nodos y que todos son maestros. Estos ficheros serán muy parecidos en los tres nodos
de nuestro cluster.

^^^^^^

#### 💻️ Configurando Redis Cluster

El siguiente paso es asignar los _hash slots_ a los diferentes nodos, para lo que utilizamos el comando
[`CLUSTER ADDSLOTS`](https://redis.io/commands/cluster-addslots):

```bash
nodec > for i in {0..5500}; do redis-cli -h 192.168.158.100 -p 6379 CLUSTER ADDSLOTS $i; done
nodec > for i in {5501..11000}; do redis-cli -h 192.168.158.101 -p 6379 CLUSTER ADDSLOTS $i; done
nodec > for i in {11001..16383}; do redis-cli -h 192.168.158.102 -p 6379 CLUSTER ADDSLOTS $i; done
```

^^^^^^

#### 💻️ Configurando Redis Cluster

Utilizando el comando [`CLUSTER NODES`](https://redis.io/commands/cluster-nodes) podemos listar los nodos
del cluster:

```bash 
nodec > redis-cli -h 192.168.158.100 -p 6379 CLUSTER NODES
33bc4dc83aa3a11c86e35bb0ca84a3ab84280c79 192.168.158.102:6380@16380 master - 0 1583334453000 1 connected
fe233b965198b91a36134a887705bf848137ea74 192.168.158.100:6380@16380 master - 0 1583334453000 0 connected
fdfe15734fd5d82d6d3b108b21800a0f7346a41a 192.168.158.101:6380@16380 master - 0 1583334451667 4 connected
a46334a276c871fa95b1fd5b90f006518dadec7d 192.168.158.101:6379@16379 master - 0 1583334454697 3 connected 5501-11000
2125eb65a7fef9af62fd90774db0614587f44464 192.168.158.102:6379@16379 master - 0 1583334453685 5 connected 11001-16383
a30ba6afb3b76414b51a8a57542aa76f753b1da4 192.168.158.100:6379@16379 myself,master - 0 1583334450000 2 connected 0-5500  
```

y ver quién tiene asignado cada _hash slot_. 

^^^^^^

#### 💻️ Configurando Redis Cluster

El siguiente y último paso es configurar las réplicas con el comando [`CLUSTER REPLICATE`](https://redis.io/commands/cluster-replicate).

^^^^^^

#### 💻️ Configurando Redis Cluster

![](/slides/images/setting_up_a_redis_cluster/setting_up_a_redis_cluster.001.jpeg)<!-- .element: style="height: 60vh" -->

^^^^^^

#### 💻️ Configurando Redis Cluster

Empezamos configurando la réplica del maestro del node `nodea`.

Según el esquema de nuestro cluster, la réplica debe estar en la instancia de redis del node `nodec` en el puerto 6380.

^^^^^^

#### 💻️ Configurando Redis Cluster

Con el comando [`CLUSTER NODES`](https://redis.io/commands/cluster-nodes) buscamos cuál es ID del maestro que queremos 
replicar, nos conectamos a la réplica y ejecutamos:

```bash 
nodec > redis-cli -h 192.168.158.102 -p 6380 CLUSTER REPLICATE a30ba6afb3b76414b51a8a57542aa76f753b1da4
OK
```

^^^^^^

#### 💻️ Configurando Redis Cluster

Seguimos configurando la réplica del maestro del nodo `nodeb`.

Según el esquema de nuestro cluster, la réplica debe estar en la instancia de redis del node `nodea` en el puerto 6380.


```bash 
nodec > redis-cli -h 192.168.158.100 -p 6380 CLUSTER REPLICATE a46334a276c871fa95b1fd5b90f006518dadec7d 
OK
```

^^^^^^

#### 💻️ Configurando Redis Cluster

Seguimos configurando la réplica del maestro del nodo `nodec`.

Según el esquema de nuestro cluster, la réplica debe estar en la instancia de redis del node `nodeb` en el puerto 6380.


```bash 
nodec > redis-cli -h 192.168.158.101 -p 6380 CLUSTER REPLICATE  
OK
```

^^^^^^

#### 💻️ Configurando Redis Cluster

Usando de nuevo el comando [`CLUSTER NODES`](https://redis.io/commands/cluster-nodes) podemos revisar los nodos
de nuestro cluster:

```bash
redis-cli -h 192.168.158.102 -p 6380 CLUSTER NODES
fe233b965198b91a36134a887705bf848137ea74 192.168.158.100:6380@16380 slave a46334a276c871fa95b1fd5b90f006518dadec7d 0 1583336005000 3 connected
a46334a276c871fa95b1fd5b90f006518dadec7d 192.168.158.101:6379@16379 master - 0 1583336005000 3 connected 5501-11000
fdfe15734fd5d82d6d3b108b21800a0f7346a41a 192.168.158.101:6380@16380 slave 2125eb65a7fef9af62fd90774db0614587f44464 0 1583336007845 5 connected
2125eb65a7fef9af62fd90774db0614587f44464 192.168.158.102:6379@16379 master - 0 1583336006835 5 connected 11001-16383
a30ba6afb3b76414b51a8a57542aa76f753b1da4 192.168.158.100:6379@16379 master - 0 1583336008855 2 connected 0-5500
33bc4dc83aa3a11c86e35bb0ca84a3ab84280c79 192.168.158.102:6380@16380 myself,slave a30ba6afb3b76414b51a8a57542aa76f753b1da4 0 1583336007000 1 connected
```

^^^^^^

#### 💻️ Configurando Redis Cluster

Usando el comando [`CLUSTER INFO`](https://redis.io/commands/cluster-info) podemos ver el estado de nuestro cluster:

```bash
nodec > redis-cli -h 192.168.158.102 -p 6380 CLUSTER INFO
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:5
cluster_my_epoch:2
cluster_stats_messages_ping_sent:2513
cluster_stats_messages_pong_sent:2678
cluster_stats_messages_meet_sent:14
cluster_stats_messages_sent:5205
cluster_stats_messages_ping_received:2676
cluster_stats_messages_pong_received:2527
cluster_stats_messages_meet_received:2
cluster_stats_messages_received:5205
```

^^^^^^

#### 💻️ Configurando Redis Cluster

Y ahora, si todo ha ido bien, deberíamos ser capaces de crear una clave en un nodo (por ejemplo en el nodo `nodea`)
y recuperarla si nos conectamos a otro nodo (por ejemplo el nodo `nodec`):

```bash
nodec > bin/redis-cli -h 192.168.158.100 -p 6379 -c
redis-cli (nodea) > set nombre "Curso de redis"
-> Redirected to slot [9465] located at 192.168.158.101:6379
OK
```

```bash
nodec > bin/redis-cli -h 192.168.158.102 -p 6379 -c
redis-cli (nodec) > get nombre 
-> Redirected to slot [9465] located at 192.168.158.101:6379
"Curso de redis"
```