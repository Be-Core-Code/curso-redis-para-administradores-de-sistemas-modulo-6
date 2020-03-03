### Redis Cluster

* Distribuir los datos automáticamente entre múltiples nodos
* Continuar operando cuando un subconjunto de nodos está teniendo fallos o pierde la comunicación con el resto del cluster


^^^^^^

#### Redis Cluster: _Cluster bus_

Cualquier nodo del cluster requiere de dos puertos:

* El puerto TCP de redis (6379 por defecto) para dar servicio a los clientes
* Un segundo puerto (puerto de redis + 10000, 16379 en este caso) utilizado por el `Cluster bus`


^^^^^^

#### Redis Cluster: _Cluster bus_

* Este puerto (16379) se utiliza para:
  * detectar fallos en nodos
  * actaulizar la configuración
  * coordinación de los procesos de failover
  * etc...
  
* **Todos los nodos del cluster deben ser capaces de acceder al puerto 16379 de cualquier otro nodo**


^^^^^^

#### Redis Cluster: _Cluster bus_

* El `Cluster bus` utiliza un protocolo de comunicación binario, lo que reduce el tiempo de procesamiento y de 
  transferencia entre los nodos

^^^^^^

#### Redis Cluster: Docker

* ⚠️ Cuidado con Docker: el mapeo de puertos que hace Docker no es compatible con el modo Cluster de redis

* Si los nodos del cluster van a estar dentro de contenedores docker, aseguraos de 
  utilizar el modo de red `host` en esos contenedores
  
notes:

Podéis ver esta recomendación en la [documentación oficial de redis](https://redis.io/topics/cluster-tutorial#redis-cluster-and-docker).

^^^^^^

#### Redis Cluster: _Sharding_

El algoritmo de _Sharding_ es el que permite a Redis distribuir los datos entre los nodos del cluster de manera unívoca.

^^^^^^

#### Redis Cluster: _Sharding_

¿Cómo decidimos a qué nodo debemos asignar una determinada clave?

Redis Cluster dispone de lo que se conoce como _hash slots_. 

En concreto, dispone de 16384 _hash_slots_.

**Un _hash_slot_ almacena múltiples claves.**

^^^^^^

#### Redis Cluster: _Sharding_

Si queremos almacenar una clave de tipo texto:

```redis-cli
redis-cli > SET nombre_del_curso "Curso de redis" 
```

La fórmula es sencilla:

```bash
CRC16(nombre_del_curso) % 16384 = 18755 % 16384 = 2371
```

**La clave se almacenará en el _hash slot_ 2371.**

^^^^^^

#### Redis Cluster: _Sharding_

En un cluster típico de tres nodos, la distribución de los _hash slots_ podría ser la siguiente:

* Nodo A: 0 a 5500
* Nodo B: de 5501 a 11000
* Nodo C: de 11001 a 16383

^^^^^^

#### Redis Cluster: _Sharding_

Siguiendo con el ejemplo anterior, el _hash slot_ era el 2371, lo que significa que en este cluster la clave se almacenará
en el Nodo A.

^^^^^^

#### Redis Cluster: _Sharding_

Esta forma de distribuir las claves, permite a Redis Cluster redistribuir las claves entre nodos de manera sencilla
sin necesidad de dejar de dar servicio.

^^^^^^

#### Redis Cluster: _Sharding_

Además, tiene la ventaja de que la distribución de claves entre nodos **no supone caída del servicio**
 
^^^^^^

#### Redis Cluster: _Sharding_

Ejemplos de operaciones que implican redistribuir claves: 
* Sacar un nodo del cluster
* Añadir un nuevo nodo al clusteer
* Asignar un porcentaje más alto de _hash slots_ a un nodo con más memoria 

^^^^^^

#### Redis Cluster: _Sharding_

**Redis Cluster soporta operaciones con múltiples claves siempre que estas estén en el mismo _hash slot_.**

El usuario puede forzar a que varias claves estén en el mismo _hash slot_ utilizando _hash tags_.

^^^^^^

#### Redis Cluster: _Sharding_

Un _hash tag_ se añade a la utilizando llaves `{ }`.

Ejemplo: las siguientes claves se almacenarán en el mismo _hash slot_

* `Nombre{user:1}:1`
* `Apellido{user:1}:1`

Redis utilizará la cadena dentro de `{ }` (en este caso `user:1`) para calcular el _hash slot_ 

^^^^^^

#### Redis Cluster: master-slave model

En el caso anterior, en el que tenemos un cluster con tres nodos, si perdemos el Nodo B no podremos acceder a las
claves almacenadas los _hash slots_ del 5501 a 11000.

^^^^^^

#### Redis Cluster: master-slave model

Redis Cluster implementa un modelo maestro-esclavo en el cada _hash slot_ tiene entre 1 (el nodo maestro) y N réplicas 
(N-1 nodos esclavos o réplicas).

Siguiendo con el ejemplo anterior, nuestro cluster tendría:

* Tres nodos maestros: A, B y C
* Tres réplicas: A1, B1, C1

^^^^^^

#### Redis Cluster: master-slave model

En esta situación, si el nodo B se cae, el nodo B1 puede promocionar y convertirse en maestro para los _hash slots_
correspondientes.

^^^^^^

#### Redis Cluster: master-slave model

**Si caen el nodo B y el nodo B1, redis no podrá operar con normalidad ya que habrá perdido todos
los _hash slots_ entre 5501 y 11000.**  
 
^^^^^^

#### Redis Cluster: Consistencia

**Redis Cluster no puede garantizar una consistencia fuerte de los datos.**

Se dan ciertas situaciones en las que podemos perder escrituras por el echo de que Redis utiliza replicación asíncrona.

notes:

En las diapositivas vamos a comentar un caso en el que esta pérdida de escrituras se puede dar. En
[la documentación de redis](https://redis.io/topics/cluster-tutorial#redis-cluster-consistency-guarantees) podéis
encontrar un poco más de información al respecto así como otros casos que pueden resultar en datos inconsistentes. 


^^^^^^

#### Redis Cluster: Consistencia

![strong-consistency.001](/slides/images/strong-consistency/strong-consistency.001.jpeg)<!-- .element: style="height: 60vh"-->

^^^^^^

#### Redis Cluster: Consistencia

![strong-consistency.002](/slides/images/strong-consistency/strong-consistency.002.jpeg)<!-- .element: style="height: 60vh"-->


^^^^^^

#### Redis Cluster: Consistencia

![strong-consistency.003](/slides/images/strong-consistency/strong-consistency.003.jpeg)<!-- .element: style="height: 60vh"-->


^^^^^^

#### Redis Cluster: Consistencia

![strong-consistency.004](/slides/images/strong-consistency/strong-consistency.004.jpeg)<!-- .element: style="height: 60vh"-->

^^^^^^

#### Redis Cluster: Consistencia

![strong-consistency.005](/slides/images/strong-consistency/strong-consistency.005.jpeg)<!-- .element: style="height: 60vh"-->

^^^^^^

#### Redis Cluster: Consistencia

![strong-consistency.006](/slides/images/strong-consistency/strong-consistency.006.jpeg)<!-- .element: style="height: 60vh"-->

^^^^^^

#### Redis Cluster: Consistencia

![strong-consistency.007](/slides/images/strong-consistency/strong-consistency.007.jpeg)<!-- .element: style="height: 60vh"-->


^^^^^^

#### Redis Cluster: Consistencia

A esta situación estamos acostumbrados en una base de datos SQL tradicional en la que podemos perder información si el servidor
se cae antes de la llamada a `flush()` que persiste los datos en disco.

^^^^^^

#### Redis Cluster: Consistencia

Si queremos que eso no nos pase en un servidor de base de datos tradicional, tendríamos que haacer una llamada a `flush()`
en cada escritura y esperar a que se guardasen los datos antes de responder al cliente.

El rendimiento de un sistema así configurado sería bajísimo. 

^^^^^^

#### Redis Cluster: Consistencia

**Al igual que con otros sistemas de base de datos, debemos buscar un compromiso entre rendimiento y consistencia de los datos.**