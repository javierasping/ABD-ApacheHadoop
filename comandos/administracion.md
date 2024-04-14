### 3.3 Comandos de administración

La administración del sistema de archivos distribuido de Hadoop (HDFS) requiere una serie de herramientas y comandos específicos para gestionar eficientemente su funcionamiento. En esta sección, encontraras una lista de comandos de administración de HDFS que te permitirán realizar tareas importantes para mantener y supervisar tu clúster Hadoop.

Estos comandos abarcan desde la verificación del estado de salud del sistema de archivos hasta la configuración de parámetros importantes como el factor de replicación de los archivos. Al utilizar estos comandos, podrás realizar tareas como monitorear el espacio disponible en el sistema de archivos, deshabilitar el modo seguro del NameNode, formatear el NameNode, entre otras funciones esenciales para la gestión de un clúster Hadoop.

| COMANDO                                | DESCRIPCIÓN                                                                |
|----------------------------------------|----------------------------------------------------------------------------|
| hdfs dfs -df /hadoop                   | Muestra la capacidad y el espacio libre y usado del sistema de ficheros   |
| hdfs dfs -df -h /hadoop                | Muestra la capacidad y el espacio libre y usado del sistema de ficheros en formato legible |
| hadoop version                        | Muestra la versión de hadoop                                              |
| hdfs fsck /                            | Comprueba el estado de salud del sistema de ficheros                       |
| hdfs dfsadmin -safemode leave         | Deshabilita el modo seguro del NameNode                                   |
| hdfs namenode -format                 | Formatea el NameNode                                                      |
| hadoop fs -test -e filename           | Si el path existe en HDFS, devuelve 0                                     |
| hadoop fs -setrep -w 3 /file1         | Cambia el factor de replicación de un fichero a 3. Si se indica un directorio, cambia el factor de replicación de todos los ficheros que contiene |
