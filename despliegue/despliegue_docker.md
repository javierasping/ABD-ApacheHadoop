### 2.2 Despliegue en docker

Partiremos de la base de que tenemos docker y docker-compose instalado en nuestra maquina . En mi caso yo tengo instalada la version de la comunidad que viene en los repositorios de Debian 12 para ambas .

Una vez entendido lo que vamos a desplegar clonaremos el repositorio , en mi caso como soy dueño del repositorio lo clonare por ssh por si necesito subir cambios al mismo :

```bash
debian@cruces-k8s-1:~$ git clone git@github.com:javierasping/docker-hadoop.git
```

Una vez hecho esto nos meteremos dentro de la carpeta donde contiene nuestro repositorio y nos aseguraremos de que tenemos copiados todos los ficheros :

```bash
debian@cruces-k8s-1:~/docker-hadoop$ ls -l 
total 21088
-rw-r--r-- 1 debian debian     1437 Apr 14 15:48 Makefile
-rw-r--r-- 1 debian debian     2171 Apr 14 15:48 README.md
drwxr-xr-x 2 debian debian     4096 Apr 14 15:48 base
drwxr-xr-x 2 debian debian     4096 Apr 14 15:48 datanode
-rw-r--r-- 1 debian debian     2522 Apr 14 15:48 docker-compose-v3.yml
-rw-r--r-- 1 debian debian     1587 Apr 14 16:35 docker-compose.yml
-rw-r--r-- 1 debian debian   697323 Apr 14 16:01 hadoop-mapreduce-examples-2.7.1-sources.jar
-rw-r--r-- 1 debian debian     2507 Apr 14 15:48 hadoop.env
drwxr-xr-x 2 debian debian     4096 Apr 14 15:48 historyserver
-rw-r--r-- 1 debian debian 20840207 Apr 14 16:01 input1.txt 
drwxr-xr-x 2 debian debian     4096 Apr 14 15:48 namenode
drwxr-xr-x 2 debian debian     4096 Apr 14 15:48 nginx
drwxr-xr-x 2 debian debian     4096 Apr 14 15:48 nodemanager
drwxr-xr-x 2 debian debian     4096 Apr 14 15:48 resourcemanager
drwxr-xr-x 2 debian debian     4096 Apr 14 15:48 submit
```

Ahora levantaremos el escenario haciendo uso de docker-compose , este levantara todas las configuraciones que le hemos indicado en el fichero docker-compose.yml . Ademas creara una red "personalizada" que contara con resolución dns para nuestro escenario . Por ultimo es importante que indiquemos el parámetro -d para que se ejecute en segundo plano .

```bash
debian@cruces-k8s-1:~/docker-hadoop$ docker-compose up -d 
```

La primera vez que lo ejecutes , tardara mas de lo normal ya que se esta bajando las imágenes .

Cuando finalice este podemos listar los contenedores que se han creado haciendo un docker ps , asegúrate que se han creado los 5 contenedores definidos en el docker-compose.yml :

```bash
debian@cruces-k8s-1:~/docker-hadoop$ docker ps
CONTAINER ID   IMAGE                                                    COMMAND                  CREATED             STATUS                    PORTS                                                                                  NAMES
978208a4bb44   bde2020/hadoop-resourcemanager:2.0.0-hadoop3.2.1-java8   "/entrypoint.sh /run…"   About an hour ago   Up 2 hours (healthy)   0.0.0.0:8088->8088/tcp, :::8088->8088/tcp                                              resourcemanager
1886eb9c334b   bde2020/hadoop-namenode:2.0.0-hadoop3.2.1-java8          "/entrypoint.sh /run…"   2 hours ago         Up 2 hours (healthy)      0.0.0.0:9000->9000/tcp, :::9000->9000/tcp, 0.0.0.0:9870->9870/tcp, :::9870->9870/tcp   namenode
21256d355687   bde2020/hadoop-nodemanager:2.0.0-hadoop3.2.1-java8       "/entrypoint.sh /run…"   2 hours ago         Up 2 hours (healthy)      8042/tcp                                                                               nodemanager
319cf2cff022   bde2020/hadoop-datanode:2.0.0-hadoop3.2.1-java8          "/entrypoint.sh /run…"   2 hours ago         Up 2 hours (healthy)      9864/tcp                                                                               datanode
054ef200e9a5   bde2020/hadoop-historyserver:2.0.0-hadoop3.2.1-java8     "/entrypoint.sh /run…"   2 hours ago         Up 2 hours (healthy)      8188/tcp                                                                               historyserver

```


 