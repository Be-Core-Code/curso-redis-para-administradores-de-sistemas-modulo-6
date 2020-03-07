### Configurando Redis Cluster: usando --cluster

En la sección anterior, creamos un cluster de redis de manera manual.

Redis nos facilita un script, embebido en `redis-cli` a partir de la versión 5, **para facilitar este proceso de 
configuración del cluster**.

En esta sección utilizaremos este script en lugar de hacelo manualmente.

^^^^^^

#### Configurando Redis Cluster: usando --cluster

```bash
(nodea) > redis-cli --cluster create \ 
192.168.158.100:6379 192.168.158.101:6379 192.168.158.102:6379 \
192.168.158.100:6380  192.168.158.101:6380 192.168.158.102:6380 \
--cluster-replicas 1
```

^^^^^^
 
 #### Configurando Redis Cluster: usando --cluster
 
```bash
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 192.168.158.101:6380 to 192.168.158.100:6379
Adding replica 192.168.158.102:6380 to 192.168.158.101:6379
Adding replica 192.168.158.100:6380 to 192.168.158.102:6379
M: 640673d51a2a3a16e6db2bae8108cfb0b6e6d68b 192.168.158.100:6379
   slots:[0-5460] (5461 slots) master
M: ab799618658931c1099be6adaa1a5456e696566c 192.168.158.101:6379
   slots:[5461-10922] (5462 slots) master
M: 744121782a1f3d5f057b5466aa3e2fad28f7c300 192.168.158.102:6379
   slots:[10923-16383] (5461 slots) master
S: 6856a97e5144bc676cbcc3fa4590b154f03862c2 192.168.158.100:6380
   replicates 744121782a1f3d5f057b5466aa3e2fad28f7c300
S: cbfa4f833bd8941f277f51928b1f2cb1c702a61d 192.168.158.101:6380
   replicates 640673d51a2a3a16e6db2bae8108cfb0b6e6d68b
S: 05bf5255f82d93e522393372056f63aa21dbffb6 192.168.158.102:6380
   replicates ab799618658931c1099be6adaa1a5456e696566c
Can I set the above configuration? (type 'yes' to accept): 
```


^^^^^^
 
#### Configurando Redis Cluster: usando --cluster
 
```bash
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
......
>>> Performing Cluster Check (using node 192.168.158.100:6379)
M: 640673d51a2a3a16e6db2bae8108cfb0b6e6d68b 192.168.158.100:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: ab799618658931c1099be6adaa1a5456e696566c 192.168.158.101:6379
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: cbfa4f833bd8941f277f51928b1f2cb1c702a61d 192.168.158.101:6380
   slots: (0 slots) slave
   replicates 640673d51a2a3a16e6db2bae8108cfb0b6e6d68b
M: 744121782a1f3d5f057b5466aa3e2fad28f7c300 192.168.158.102:6379
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 05bf5255f82d93e522393372056f63aa21dbffb6 192.168.158.102:6380
   slots: (0 slots) slave
   replicates ab799618658931c1099be6adaa1a5456e696566c
S: 6856a97e5144bc676cbcc3fa4590b154f03862c2 192.168.158.100:6380
   slots: (0 slots) slave
   replicates 744121782a1f3d5f057b5466aa3e2fad28f7c300
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

^^^^^^

#### Configurando Redis Cluster: usando --cluster
 
```redis-cli
redis-cli (nodea) > cluster nodes
640673d51a2a3a16e6db2bae8108cfb0b6e6d68b 192.168.158.100:6379@16379 myself,master - 0 1583608107000 1 connected 0-5460
ab799618658931c1099be6adaa1a5456e696566c 192.168.158.101:6379@16379 master - 0 1583608106000 2 connected 5461-10922
cbfa4f833bd8941f277f51928b1f2cb1c702a61d 192.168.158.101:6380@16380 slave 640673d51a2a3a16e6db2bae8108cfb0b6e6d68b 0 1583608109970 5 connected
744121782a1f3d5f057b5466aa3e2fad28f7c300 192.168.158.102:6379@16379 master - 0 1583608108000 3 connected 10923-16383
05bf5255f82d93e522393372056f63aa21dbffb6 192.168.158.102:6380@16380 slave ab799618658931c1099be6adaa1a5456e696566c 0 1583608108963 6 connected
6856a97e5144bc676cbcc3fa4590b154f03862c2 192.168.158.100:6380@16380 slave 744121782a1f3d5f057b5466aa3e2fad28f7c300 0 1583608107000 4 connected
```

^^^^^^
 
#### Configurando Redis Cluster: usando --cluster

En las versiones 3 y 4 de Redis, esta opción de `redis-cli` se distribuye como un script independiente llamado `redis-trib.rb`.

