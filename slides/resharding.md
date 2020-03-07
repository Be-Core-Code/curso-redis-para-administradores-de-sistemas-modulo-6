### Resharding

Proceso para mover un grupo de _hash slots_ de un nodo a otro del cluster.

^^^^^^

#### 💻 Resharding

En nuestro caso, vamos a hacer el siguiente movimiento:

* 500 _hash slots_ de `nodeb` -> `nodea`
* 500 _hash slots_ de `nodeb` -> `nodec`

![](/slides/images/setting_up_a_redis_cluster/setting_up_a_redis_cluster.001.jpeg)<!-- .element: style="height: 30vh" -->

^^^^^^

#### 💻 Resharding

```bash
(nodeX) > redis-cli --cluster reshard 192.168.158.100:6379
```

Podemos utilizar la dirección IP y puerto de cualquier nodo.

^^^^^^
#### 💻 Resharding

```bash
(nodeX) >
>>> Performing Cluster Check (using node 192.168.158.100:6379)
M: 640673d51a2a3a16e6db2bae8108cfb0b6e6d68b 192.168.158.100:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: ab799618658931c1099be6adaa1a5456e696566c 192.168.158.101:6379 <-----------
   slots:[5461-10922] (5462 slots) master                        <-----------
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
How many slots do you want to move (from 1 to 16384)?
```

^^^^^^

#### 💻 Resharding

Seleccionamos el número de nodos, el origen y el destino:

```bash
How many slots do you want to move (from 1 to 16384)? 500
What is the receiving node ID? 640673d51a2a3a16e6db2bae8108cfb0b6e6d68b
Please enter all the source node IDs.
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
Source node #1: ab799618658931c1099be6adaa1a5456e696566c
Source node #2: done
```

^^^^^^

#### 💻 Resharding

Seleccionamos el número de nodos, el origen y el destino:

```bash
...
    Moving slot 5955 from ab799618658931c1099be6adaa1a5456e696566c
    Moving slot 5956 from ab799618658931c1099be6adaa1a5456e696566c
    Moving slot 5957 from ab799618658931c1099be6adaa1a5456e696566c
    Moving slot 5958 from ab799618658931c1099be6adaa1a5456e696566c
    Moving slot 5959 from ab799618658931c1099be6adaa1a5456e696566c
    Moving slot 5960 from ab799618658931c1099be6adaa1a5456e696566c
Do you want to proceed with the proposed reshard plan (yes/no)?
```

notes:

El script nos mostrará una línea por cada slot que vaya a mover.

^^^^^^

#### 💻 Resharding

Seleccionamos el número de nodos, el origen y el destino:

```bash
...
Moving slot 5952 from 192.168.158.101:6379 to 192.168.158.100:6379:
Moving slot 5953 from 192.168.158.101:6379 to 192.168.158.100:6379:
Moving slot 5954 from 192.168.158.101:6379 to 192.168.158.100:6379:
Moving slot 5955 from 192.168.158.101:6379 to 192.168.158.100:6379:
Moving slot 5956 from 192.168.158.101:6379 to 192.168.158.100:6379:
Moving slot 5957 from 192.168.158.101:6379 to 192.168.158.100:6379:
Moving slot 5958 from 192.168.158.101:6379 to 192.168.158.100:6379:
Moving slot 5959 from 192.168.158.101:6379 to 192.168.158.100:6379:
Moving slot 5960 from 192.168.158.101:6379 to 192.168.158.100:6379:
(nodeX) > _
```
^^^^^^

#### 💻 Resharding

Movemos los otros 500 _hash slots_ a `nodec`:

```bash
How many slots do you want to move (from 1 to 16384)? 500
What is the receiving node ID? 744121782a1f3d5f057b5466aa3e2fad28f7c300
Please enter all the source node IDs.
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
Source node #1: ab799618658931c1099be6adaa1a5456e696566c
Source node #2: done
```

^^^^^^

#### 💻 Resharding

Vemos que `nodea` y `nodec` tiene 500 _hash slots_ más cada uno (y `nodeb` tiene 1000 _hash slots_ menos):

```bash
redis-cli -h 192.168.158.100 -c cluster nodes
640673d51a2a3a16e6db2bae8108cfb0b6e6d68b 192.168.158.100:6379@16379 myself,master - 0 1583610633000 7 connected 0-5960
ab799618658931c1099be6adaa1a5456e696566c 192.168.158.101:6379@16379 master - 0 1583610632481 2 connected 6461-10922
cbfa4f833bd8941f277f51928b1f2cb1c702a61d 192.168.158.101:6380@16380 slave 640673d51a2a3a16e6db2bae8108cfb0b6e6d68b 0 1583610634499 7 connected
744121782a1f3d5f057b5466aa3e2fad28f7c300 192.168.158.102:6379@16379 master - 0 1583610633488 8 connected 5961-6460 10923-16383
05bf5255f82d93e522393372056f63aa21dbffb6 192.168.158.102:6380@16380 slave ab799618658931c1099be6adaa1a5456e696566c 0 1583610632000 6 connected
6856a97e5144bc676cbcc3fa4590b154f03862c2 192.168.158.100:6380@16380 slave 744121782a1f3d5f057b5466aa3e2fad28f7c300 0 1583610633000 4 connected
```

^^^^^^

#### 💻 Resharding

También podemos verlo usando el siguiente comando:

```bash
> redis-cli --cluster check 192.168.158.100:6379
192.168.158.100:6379 (640673d5...) -> 0 keys | 5961 slots | 1 slaves.
192.168.158.101:6379 (ab799618...) -> 1 keys | 4462 slots | 1 slaves.
192.168.158.102:6379 (74412178...) -> 0 keys | 5961 slots | 1 slaves.
[OK] 1 keys in 3 masters.
0.00 keys per slot on average.
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
nodea:~$
```

