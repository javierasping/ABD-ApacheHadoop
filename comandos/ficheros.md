### 3.1 Gestión de ficheros

La gestión de ficheros en Apache Hadoop es fundamental para administrar y manipular datos en entornos distribuidos. En esta sección, exploraremos una serie de comandos y técnicas para interactuar con el sistema de archivos distribuido de Hadoop (HDFS)

#### Listar Ficheros en HDFS

| COMANDO             | DESCRIPCIÓN                                                         |
|---------------------|---------------------------------------------------------------------|
| hdfs dfs -ls /      | Lista todos los ficheros y directorios para el path /              |
| hdfs dfs -ls -h /   | Lista los ficheros con su tamaño en formato legible                |
| hdfs dfs -ls -R /   | Lista todos los ficheros y directorios recursivamente              |
| hdfs dfs -ls /file*| Lista todos los ficheros que cumplen el patrón                      |
| hadoop fs -stat    | Imprime estadísticas del fichero o directorio en el formato indicado|

#### Leer y Escribir Ficheros

| COMANDO                         | DESCRIPCIÓN                                                                 |
|---------------------------------|-----------------------------------------------------------------------------|
| hdfs dfs -text /app.log         | Imprime el fichero en modo texto por la terminal                             |
| hdfs dfs -cat /app.log          | Muestra el contenido del fichero en la salida estándar                       |
| hdfs dfs -appendToFile          | Añade el contenido del fichero local ‘file1’ al fichero en HDFS ‘file2’      |

#### Cargar y Descargar Ficheros

| COMANDO                         | DESCRIPCIÓN                                                                 |
|---------------------------------|-----------------------------------------------------------------------------|
| hdfs dfs -put /home/file1 /hadoop | Copia el fichero ‘file1’ del sistema de ficheros local a HDFS          |
| hdfs dfs -put -f /home/file1 /hadoop | Copia el fichero ‘file1’ del sistema de ficheros local a HDFS y lo sobreescribe en el caso de que ya exista |
| hdfs dfs -put -l /home/file1 /hadoop | Copia el fichero ‘file1’ del sistema de ficheros local a HDFS. Fuerza replicación 1 y permite al DataNode persistir los datos de forma perezosa. |
| hdfs dfs -put -p /home/file1 /hadoop | Copia el fichero ‘file1’ del sistema de ficheros local a HDFS. Mantiene los tiempos de acceso, de modificación y propietario original |
| hdfs dfs -get /file1 /home/ | Copia el fichero ‘file1’ de HDFS al sistema de ficheros local |
| hdfs dfs -copyToLocal /file1 /home/ | Copia el fichero ‘file1’ de HDFS al sistema de ficheros local (igual que el anterior) |
| hdfs dfs -moveFromLocal /home/file1 /hadoop | Copia el fichero ‘file1’ del sistema de ficheros local a HDFS y luego lo borra del sist. ficheros local |

#### Gestión de Ficheros

| COMANDO                         | DESCRIPCIÓN                                                                 |
|---------------------------------|-----------------------------------------------------------------------------|
| hdfs dfs -cp /hadoop/file1 /hadoop1 | Copia el fichero al directorio destino en HDFS |
| hdfs dfs -cp -p /hadoop/file1 /hadoop1 | Copia el fichero al directorio destino en HDFS conservando tiempos de acceso y de modificación, propietario y modo |
| hdfs dfs -rm /hadoop/file1 | Elimina el fichero ‘file1’ de HDFS y lo envía a la papelera |
| hdfs dfs -rm -r /hadoop | Elimina el directorio y su contenido en HDFS |
| hdfs dfs -rm -skipTrash /file1 | Elimina el fichero sin dejarlo en la papelera |
| hdfs dfs -mkdir /hadoop2 | Crea un directorio en HDFS |
| hdfs dfs -touchz /hadoop3 | Crea un fichero en HDFS con tamaño 0 |
| hadoop fs -getmerge -nl /file1 /file2 /output | Combina varios ficheros en uno solo |

