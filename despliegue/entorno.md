### 2.1 Explicación del entorno

Para desplegar un entorno de Apache Hadoop, he optado por utilizar contenedores Docker, lo que simplifica enormemente el proceso de implementación y gestión de los distintos nodos.

Para ello he utilizado una version de la comunidad y la he modificado para adaptarla a este proyecto , te dejo el [enlace a mi repositorio](https://github.com/javierasping/docker-hadoop.git) .

Para desplegar el clúster de forma cómoda y eficiente, he optado por utilizar docker-compose. En nuestro archivo docker-compose.yml, hemos definido los siguientes nodos:

- Namenode: El corazón del sistema de archivos distribuido de Hadoop. Administra el acceso a los datos y coordina el almacenamiento en el clúster.
- Datanode: Almacena y gestiona los datos en el clúster, siguiendo las instrucciones del namenode.
- Resourcemanager: Controla los recursos del clúster, asignando tareas a los nodos y gestionando la escalabilidad.
- Nodemanager: Gestiona los recursos disponibles en cada nodo del clúster y ejecuta tareas específicas asignadas por el resourcemanager.
- Historyserver: Ofrece un historial detallado de las aplicaciones que se han ejecutado en el clúster, facilitando el análisis retrospectivo y la depuración.

Cada servicio está configurado con detalles específicos, como la imagen de Docker a utilizar, el nombre del contenedor, los puertos expuestos y cualquier otra configuración necesaria, como variables de entorno.

Además, hemos definido volúmenes para hacer permanente los datos generados por los servicios de Hadoop, asegurando así la disponibilidad de la información crítica incluso en caso de reinicios o detenciones de los contenedores.

Te dejo aquí el fichero aunque puedes encontrarlo en el enlace al principio de este :

```bash
debian@cruces-k8s-1:~/docker-hadoop$ cat docker-compose.yml 
version: "3"

services:
  namenode:
    image: bde2020/hadoop-namenode:2.0.0-hadoop3.2.1-java8
    container_name: namenode
    restart: always
    ports:
      - 9870:9870
      - 9000:9000
    volumes:
      - hadoop_namenode:/hadoop/dfs/name
    environment:
      - CLUSTER_NAME=test
    env_file:
      - ./hadoop.env

  datanode:
    image: bde2020/hadoop-datanode:2.0.0-hadoop3.2.1-java8
    container_name: datanode
    restart: always
    volumes:
      - hadoop_datanode:/hadoop/dfs/data
    environment:
      SERVICE_PRECONDITION: "namenode:9870"
    env_file:
      - ./hadoop.env

  resourcemanager:
    image: bde2020/hadoop-resourcemanager:2.0.0-hadoop3.2.1-java8
    container_name: resourcemanager
    restart: always
    ports:
      - 8088:8088
    environment:
      SERVICE_PRECONDITION: "namenode:9000 namenode:9870 datanode:9864"
    env_file:
      - ./hadoop.env

  nodemanager1:
    image: bde2020/hadoop-nodemanager:2.0.0-hadoop3.2.1-java8
    container_name: nodemanager
    restart: always
    environment:
      SERVICE_PRECONDITION: "namenode:9000 namenode:9870 datanode:9864 resourcemanager:8088"
    env_file:
      - ./hadoop.env
  
  historyserver:
    image: bde2020/hadoop-historyserver:2.0.0-hadoop3.2.1-java8
    container_name: historyserver
    restart: always
    environment:
      SERVICE_PRECONDITION: "namenode:9000 namenode:9870 datanode:9864 resourcemanager:8088"
    volumes:
      - hadoop_historyserver:/hadoop/yarn/timeline
    env_file:
      - ./hadoop.env
  
volumes:
  hadoop_namenode:
  hadoop_datanode:
  hadoop_historyserver:
```

Cabe destacar que cada nodo esta configurado con variables de entorno , asi como en los nodos resourcemanager y namenode hay configurada una redirection para acceder posteriormente a los servicios web que nos ofrece Hadoop .
