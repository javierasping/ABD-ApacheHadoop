## 1.4 Alternativas a Hadoop

Para evaluar las alternativas a Hadoop que existen en el mercado debemos tener en cuenta que menudo podemos integrar diversas tecnologías para resolver el problema que se nos plantea.

Apache Hadoop ha formado todo un ecosistema alrededor muy flexible, del que debemos seleccionar solamente los componentes que necesitemos para evitar recargar nuestro sistema con piezas a las que no se va a sacar partido.

Podemos poner el conocido Hadoop vs Spark como ejemplo. Ambas tecnologías pueden convivir perfectamente en una solución Big Data completa. HDFS proporcionarán la base sobre la que podrán ejecutar las cargas Spark en el caso de que necesitemos realizar transformaciones de los datos complejas. De esta forma, nos podremos aprovechar de un sistema de almacenamiento distribuido y con las réplicas necesarias para garantizar la alta disponibilidad del dato.

### Hadoop vs Spark

#### ¿Cuál es la diferencia entre Hadoop y Spark?
Apache Hadoop y Apache Spark son dos marcos de código abierto que puede utilizar para administrar y procesar grandes volúmenes de datos para su análisis. Las organizaciones deben procesar los datos a gran escala y velocidad para obtener información en tiempo real para la inteligencia empresarial. Apache Hadoop permite agrupar varios equipos para analizar conjuntos de datos enormes en paralelo con mayor rapidez. Apache Spark utiliza el almacenamiento en memoria caché y una ejecución de consultas optimizada para permitir consultas de análisis rápidas en datos de cualquier tamaño. Spark es una tecnología más avanzada que Hadoop, ya que utiliza inteligencia artificial y machine learning (IA y ML) en el procesamiento de datos. Sin embargo, muchas empresas utilizan Spark y Hadoop juntos para cumplir sus objetivos de análisis de datos.


Apache Hadoop fue diseñado para distribuir el procesamiento de datos entre múltiples servidores en lugar de confiar en una sola máquina para ejecutar la carga de trabajo, lo que lo hace ideal para análisis en lotes pero puede tener una latencia considerable.

Por otro lado, Apache Spark es una tecnología más reciente que supera las limitaciones principales de Hadoop, ya que está diseñado para procesar datos en tiempo real y utiliza almacenamiento en memoria caché, lo que lo hace más rápido y eficiente.

#### Diferencias en arquitectura

En términos de arquitectura, Hadoop utiliza su propio sistema de archivos distribuido (HDFS), mientras que Spark puede integrarse con varios sistemas de almacenamiento, como HDFS, Amazon Redshift o Amazon S3.

#### Diferencias en rendimiento

En cuanto al rendimiento, Hadoop es más lento ya que lee y escribe datos constantemente desde y hacia el almacenamiento externo, mientras que Spark procesa los datos en la memoria antes de escribirlos de vuelta al almacenamiento externo, lo que lo hace considerablemente más rápido.

#### Diferencias en machine learning

Spark también ofrece una biblioteca de machine learning integrada llamada MLlib, mientras que Hadoop requiere integraciones con otros programas como Apache Mahout para tareas de machine learning.

#### Diferencias en seguridad

En términos de seguridad, Hadoop tiene características sólidas integradas, mientras que Spark requiere que se habiliten las características de seguridad y se configure un entorno seguro.

#### Diferencias en escalabilidad

En cuanto a la escalabilidad, Hadoop es más fácil de escalar agregando nodos adicionales, mientras que Spark generalmente requiere inversiones en más RAM, lo que puede resultar más costoso.

#### Diferencias en costes

Finalmente, en términos de costes, Hadoop tiende a ser más económico debido a su uso de discos duros para almacenamiento y procesamiento, mientras que Spark es más caro ya que utiliza RAM, que es más cara que los discos duros.