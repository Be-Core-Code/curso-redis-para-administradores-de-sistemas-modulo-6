### _Manual failover_

Redis soporta _Manual failovers_ a través del comando [`CLUSTER FAILOVER`](https://redis.io/commands/cluster-failover).

Este comando debe ejecutarse en una de las réplicas del maestro del que vamos a hacer _failover_.


^^^^^^

#### 💻️ _Manual failover_

Miramos la configuración de nuestro cluster:

```bash
> redis-cli --cluster check 192.168.158.100:6379
  192.168.158.100:6379 (640673d5...) -> 2400 keys | 5961 slots | 1 slaves.
  192.168.158.101:6379 (ab799618...) -> 1785 keys | 4462 slots | 1 slaves.
  192.168.158.102:6379 (74412178...) -> 2372 keys | 5961 slots | 1 slaves.
  [OK] 6557 keys in 3 masters.
  0.40 keys per slot on average.
  >>> Performing Cluster Check (using node 192.168.158.100:6379)
  M: 640673d51a2a3a16e6db2bae8108cfb0b6e6d68b 192.168.158.100:6379
     slots:[0-5960] (5961 slots) master
     1 additional replica(s)
  M: ab799618658931c1099be6adaa1a5456e696566c 192.168.158.101:6379
     slots:[6461-10922] (4462 slots) master
     1 additional replica(s)
  S: cbfa4f833bd8941f277f51928b1f2cb1c702a61d 192.168.158.101:6380
     slots: (0 slots) slave
     replicates 640673d51a2a3a16e6db2bae8108cfb0b6e6d68b
  M: 744121782a1f3d5f057b5466aa3e2fad28f7c300 192.168.158.102:6379
     slots:[5961-6460],[10923-16383] (5961 slots) master
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

#### 💻️ _Manual failover_

Nos conectamos a `192.168.158.102:6380`:

```bash
(nodeX) > redis-cli -c -h  192.168.158.102 -p 6380
```

^^^^^^

#### 💻️ _Manual failover_

Ejecutamos el _failover_:

```redis-cli
redis-cli (192.168.158.102:6380) > CLUSTER FAILOVER 
```

^^^^^^

#### 💻️ _Manual failover_

Si miramos el log de `192.168.158.101:6379`:

```bash
4931:M 07 Mar 2020 22:46:30.715 # Manual failover requested by replica 05bf5255f82d93e522393372056f63aa21dbffb6.
4931:M 07 Mar 2020 22:46:30.917 # Failover auth granted to 05bf5255f82d93e522393372056f63aa21dbffb6 for epoch 9
4931:M 07 Mar 2020 22:46:30.919 # Connection with replica 192.168.158.102:6380 lost.
4931:M 07 Mar 2020 22:46:30.921 # Configuration change detected. Reconfiguring myself as a replica of 05bf5255f82d93e522393372056f63aa21dbffb6
4931:S 07 Mar 2020 22:46:30.921 * Before turning into a replica, using my master parameters to synthesize a cached master: I may be able to synchronize with the new master with just a partial transfer.
4931:S 07 Mar 2020 22:46:31.118 * Connecting to MASTER 192.168.158.102:6380
4931:S 07 Mar 2020 22:46:31.118 * MASTER <-> REPLICA sync started
4931:S 07 Mar 2020 22:46:31.118 * Non blocking connect for SYNC fired the event.
4931:S 07 Mar 2020 22:46:31.119 * Master replied to PING, replication can continue...
4931:S 07 Mar 2020 22:46:31.120 * Trying a partial resynchronization (request bd9b88cb553aecb7a37bb4fe33dbcd59f5ad6265:295772).
4931:S 07 Mar 2020 22:46:31.120 * Successful partial resynchronization with master.
4931:S 07 Mar 2020 22:46:31.120 # Master replication ID changed to 1af62dbd56f9cfd80e4d12239862c134004941c9
4931:S 07 Mar 2020 22:46:31.120 * MASTER <-> REPLICA sync: Master accepted a Partial Resynchronization.
```


^^^^^^

#### 💻️ _Manual failover_

Si miramos el log de `192.168.158.101:6380`:

```bash
4970:S 07 Mar 2020 22:46:30.715 # Manual failover user request accepted.
4970:S 07 Mar 2020 22:46:30.814 # Received replication offset for paused master manual failover: 295771
4970:S 07 Mar 2020 22:46:30.915 # All master replication stream processed, manual failover can start.
4970:S 07 Mar 2020 22:46:30.915 # Start of election delayed for 0 milliseconds (rank #0, offset 295771).
4970:S 07 Mar 2020 22:46:30.915 # Starting a failover election for epoch 9.
4970:S 07 Mar 2020 22:46:30.918 # Currently unable to failover: Waiting for votes, but majority still not reached.
4970:S 07 Mar 2020 22:46:30.919 # Failover election won: I'm the new master.
4970:S 07 Mar 2020 22:46:30.919 # configEpoch set to 9 after successful failover
4970:M 07 Mar 2020 22:46:30.919 # Setting secondary replication ID to bd9b88cb553aecb7a37bb4fe33dbcd59f5ad6265, valid up to offset: 295772. New replication ID is 1af62dbd56f9cfd80e4d12239862c134004941c9
4970:M 07 Mar 2020 22:46:30.919 # Connection with master lost.
4970:M 07 Mar 2020 22:46:30.919 * Caching the disconnected master state.
4970:M 07 Mar 2020 22:46:30.919 * Discarding previously cached master state.
4970:M 07 Mar 2020 22:46:31.120 * Replica 192.168.158.101:6379 asks for synchronization
4970:M 07 Mar 2020 22:46:31.120 * Partial resynchronization request from 192.168.158.101:6379 accepted. Sending 0 bytes of backlog starting from offset 295772.
```

^^^^^^

#### 💻️ _Manual failover_

Si miramos la configuración de nuestro cluster:

```bash
redis-cli --cluster check 192.168.158.100:6379
192.168.158.100:6379 (640673d5...) -> 2400 keys | 5961 slots | 1 slaves.
192.168.158.102:6379 (74412178...) -> 2372 keys | 5961 slots | 1 slaves.
192.168.158.102:6380 (05bf5255...) -> 1785 keys | 4462 slots | 1 slaves.
[OK] 6557 keys in 3 masters.
0.40 keys per slot on average.
>>> Performing Cluster Check (using node 192.168.158.100:6379)
M: 640673d51a2a3a16e6db2bae8108cfb0b6e6d68b 192.168.158.100:6379
   slots:[0-5960] (5961 slots) master
   1 additional replica(s)
S: ab799618658931c1099be6adaa1a5456e696566c 192.168.158.101:6379
   slots: (0 slots) slave
   replicates 05bf5255f82d93e522393372056f63aa21dbffb6
S: cbfa4f833bd8941f277f51928b1f2cb1c702a61d 192.168.158.101:6380
   slots: (0 slots) slave
   replicates 640673d51a2a3a16e6db2bae8108cfb0b6e6d68b
M: 744121782a1f3d5f057b5466aa3e2fad28f7c300 192.168.158.102:6379
   slots:[5961-6460],[10923-16383] (5961 slots) master
   1 additional replica(s)
M: 05bf5255f82d93e522393372056f63aa21dbffb6 192.168.158.102:6380
   slots:[6461-10922] (4462 slots) master
   1 additional replica(s)
S: 6856a97e5144bc676cbcc3fa4590b154f03862c2 192.168.158.100:6380
   slots: (0 slots) slave
   replicates 744121782a1f3d5f057b5466aa3e2fad28f7c300
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

^^^^^^

#### 💻️ _Manual failover_

Veremos que la réplica ha pasado a ser el maestro y el maestro la réplica:

```bash
S: ab799618658931c1099be6adaa1a5456e696566c 192.168.158.101:6379
   slots: (0 slots) slave
   replicates 05bf5255f82d93e522393372056f63aa21dbffb6

M: 05bf5255f82d93e522393372056f63aa21dbffb6 192.168.158.102:6380
   slots:[6461-10922] (4462 slots) master
   1 additional replica(s)
```


^^^^^^

#### 💻️ _Manual failover_

La operación [`CLUSTER FAILOVER`](https://redis.io/commands/cluster-failover) lleva a cabo el proceso
de una manera segura para evitar la pérdida de información.

notes:

Como se indica en la [documentación de Redis](https://redis.io/topics/cluster-tutorial#manual-failover),
este proceso para los clientes conectados al antiguo maestro, realiza el proceso de failover, y una vez el nuevo 
maestro está activo, los clientes se desbloquean.

^^^^^^

#### 💻️ _Manual failover_

Redis nos facilita un script en ruby para verificar que durante este proceso de failover no estamos perdiendo ninguna 
escritura.

El script `consistency-test.rb` se encuentra en [este repositorio](https://github.com/antirez/redis-rb-cluster).

notes:

Si estás utilizando las máquinas virtuales generadas a través de nuestro Vagrantfile, ya tienes 
ruby instalado en la máquina virtual y el repositorio clonado en la carpeta `/usr/local/redis-db-cluster`.

Si no estás usando nuestras máquinas virtuales, deberás instalar ruby y la gema `redis` para que este script
funcione.

^^^^^^

#### 💻️ _Manual failover_

Vamos a deshacer el Failover, pero en este caso ejecutaremos el script en otra terminal mientras lo hacemos:

```bash
(nodeX) > cd /usr/local/redis-rb-cluster
(nodeX) > ruby consistency-test.rb 192.168.158.100 6379
1049 R (0 err) | 1049 W (0 err) |
2114 R (0 err) | 2114 W (0 err) |
3097 R (0 err) | 3097 W (0 err) |
...
```  

^^^^^^

#### 💻️ _Manual failover_

Nos conectamos a `192.168.158.101:6379` (antiguo maestro):

```bash
redis-cli -c -h 192.168.158.101 -p 6379
```

^^^^^^

#### 💻️ _Manual failover_

Ejecutamos [`CLUSTER FAILOVER`](https://redis.io/commands/cluster-failover):

```redis-cli
redis-cli (192.168.158.101:6379) > CLUSTER FAILOVER 
```

^^^^^^

#### 💻️ _Manual failover_

Volvemos a la situación inicial:

```bash
redis-cli --cluster check 192.168.158.100:6379
192.168.158.100:6379 (640673d5...) -> 2400 keys | 5961 slots | 1 slaves.
192.168.158.102:6379 (74412178...) -> 2372 keys | 5961 slots | 1 slaves.
192.168.158.102:6380 (05bf5255...) -> 1785 keys | 4462 slots | 1 slaves.
[OK] 6557 keys in 3 masters.
0.40 keys per slot on average.
>>> Performing Cluster Check (using node 192.168.158.100:6379)
M: 640673d51a2a3a16e6db2bae8108cfb0b6e6d68b 192.168.158.100:6379
   slots:[0-5960] (5961 slots) master
   1 additional replica(s)
S: ab799618658931c1099be6adaa1a5456e696566c 192.168.158.101:6379
   slots: (0 slots) slave
   replicates 05bf5255f82d93e522393372056f63aa21dbffb6
S: cbfa4f833bd8941f277f51928b1f2cb1c702a61d 192.168.158.101:6380
   slots: (0 slots) slave
   replicates 640673d51a2a3a16e6db2bae8108cfb0b6e6d68b
M: 744121782a1f3d5f057b5466aa3e2fad28f7c300 192.168.158.102:6379
   slots:[5961-6460],[10923-16383] (5961 slots) master
   1 additional replica(s)
M: 05bf5255f82d93e522393372056f63aa21dbffb6 192.168.158.102:6380
   slots:[6461-10922] (4462 slots) master
   1 additional replica(s)
S: 6856a97e5144bc676cbcc3fa4590b154f03862c2 192.168.158.100:6380
   slots: (0 slots) slave
   replicates 744121782a1f3d5f057b5466aa3e2fad28f7c300
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

^^^^^^

#### 💻️ _Manual failover_

Si miramos la salida del script `consistency-test.rb` veremos que no se ha producido ningún error.