### 3.2 Gestión de permisos

La gestión de permisos en HDFS es crucial para controlar quién puede acceder, leer y escribir en los archivos y directorios almacenados en un clúster de Apache Hadoop. En esta sección, exploraremos una serie de comandos que te permitirán administrar los permisos de los archivos y directorios en HDFS.


| COMANDO                              | DESCRIPCIÓN                                                             |
|--------------------------------------|-------------------------------------------------------------------------|
| hdfs dfs -checksum /hadoop/file1     | Muestra la información checksum del fichero                            |
| hdfs dfs -chmod 775 /hadoop/file1    | Cambia los permisos del fichero en HDFS                                 |
| hdfs dfs -chmod -R 755 /hadoop       | Cambia los permisos de los ficheros recursivamente                      |
| hdfs dfs -chown hadoop:hadoop /file1 | Cambia el propietario y el grupo del fichero                            |
| hdfs dfs -chown -R hadoop:hadoop /file1 | Cambia el propietario y el grupo recursivamente                       |
| hdfs dfs -chgrp hadoop /file1        | Cambia el grupo del fichero                                             |
| hdfs dfs -setfacl -m user:username:perm /file1 | Establece los permisos de acceso para un usuario en particular       |
| hdfs dfs -setfacl -b /file1 | Elimina todos los permisos especiales para el archivo especificado     |