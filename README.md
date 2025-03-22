En la práctica se conectara Hadoop con ElasticSearch. En concrecto, haremos una conexión entre Hive y un índice (o index) de ElasticSearch.

**PARTE 1 - Configuración ES-Hadoop**

Para configurar nuestro ES-Hadoop, vamos a necesitar herramientas:

- Un cluster standalone o standard en Dataproc.
- Un ELK (Elastic Search, Logstash, Kibana) de Elasticsearch.

También tendremos que hacer algunos cambios en la configuración previamente, como instalar librerías de soporte en Hadoop y poner elasticsearch en modo inseguro 
para esta prueba.

a) Creamos un cluster Hadoop.
b) Cargamos en el cluster los jars de configuración de elasticsearch-hadoop y commons-httpclient.

- Creamos un nuevo bucket para nuestro ES-Hadoop.

  Subimos ambos jar a nuestro bucket en google storage.

c) Descargamos los jar desde el bucket al filesystem del cluster

-Para eso tenemos que abrir la consola del nodo maestro del cluster y subirlos con los siguientes comandos:

  gsutil cp gs://bucket-para-elastic/jars/elastic/elasticsearch-hadoop-8.14.1.jar .
  gsutil cp gs://bucket-para-elastic/jars/elastic/commons-httpclient-3.1.jar .

**ENTREGABLE PARTE 1**: Captura de pantalla de la consola SSH del cluster Hadoop una vez finalizada la configuración y carga.


**PARTE 2 - Configuración server Elasticsearch**

- Creamos una nueva VM para elastic
- Configuramos la instancia de Elasticsearch
- Crear las reglas de firewall (necesitamos abrir la comunicación al puerto 9200 y 5601 para acceder desde el cluster Hadoop y desde nuestro equipo local).

**ENTREGABLE PARTE 2**: Captura de pantalla de la consola del server Elastic donde se vea la configuración de elastic, desde 'Enable security features' hasta el final 
(el fichero elasticsearch.yml).


**PARTE 3 - Configuración en Cluster Hadoop de Conexión con ES.**

Una vez que ya hemos creado y configurado el server Elasticsearch y tenemos los ficheros en el master del cluster Dataproc, ya podemos cargar los jars en la configuración 
de HIVE. Para ello, debemos modificar el hive-site.xml.

sudo sed -i '$d' /etc/hive/conf.dist/hive-site.xml

sudo sed -i '$a \  <property>\n    <name>es.nodes</name>\n    <value>AQUÍ LA IP DE ELASTIC</value>\n  </property>\n' /etc/hive/conf.dist/hive-site.xml

sudo sed -i '$a \  <property>\n    <name>es.port</name>\n    <value>9200</value>\n  </property>\n' /etc/hive/conf.dist/hive-site.xml

sudo sed -i '$a \  <property>\n    <name>es.nodes.wan.only</name>\n    <value>true</value>\n  </property>\n' /etc/hive/conf.dist/hive-site.xml

sudo sed -i '$a \  <property>\n    <name>hive.aux.jars.path</name>\n   <value>/usr/lib/hive/lib/elasticsearch-hadoop-8.14.1.jar,/usr/lib/hive/lib/commons-httpclient-3.1.jar</value>\n  </property>\n</configuration>' /etc/hive/conf.dist/hive-site.xml

sudo cp elasticsearch-hadoop-8.14.1.jar /usr/lib/hive/lib/
sudo cp commons-httpclient-3.1.jar /usr/lib/hive/lib/

- Debemos reiniciar hive para que se apliquen los cambios.

**ENTREGABLE PARTE 3**: Captura de pantalla del proceso de configuración en Cluster Hadoop de Conexión con ES completo.


**PARTE 4 - A conectar datos!**

Desde nuestro server Elasticsearch, vamos a crear un índice sobre el que despúés trabajaremos en el cluster Hadoop:

curl -X POST "localhost:9200/alumnos/_doc/6" -H 'Content-Type: application/json' -d'
{
  "title": "New Document",
  "content": "This is a new document for the master class",
  "tag": ["general", "testing"]
}
'

Finalmente, en nuestro cluster Hadoop, vamos a agregar documentos al index alumnos recién creado:

curl -X POST "IP-SERVER-ELASTICSEARCH:9200/_bulk" -H 'Content-Type: application/json' -d'
{ "index": { "_index": "alumnos", "_id": "3" } }
{ "id": 3, "name": "Carlos", "last_name": "González" }
{ "index": { "_index": "alumnos", "_id": "4" } }
{ "id": 4, "name": "María", "last_name": "López" }
{ "index": { "_index": "alumnos", "_id": "5" } }
{ "id": 5, "name": "Luis", "last_name": "Martínez" }
{ "index": { "_index": "alumnos", "_id": "7" } }
{ "id": 7, "name": "Sofía", "last_name": "Ramírez" }
{ "index": { "_index": "alumnos", "_id": "8" } }
{ "id": 8, "name": "Pedro", "last_name": "Hernández" }
'

Hagamos una consulta para ahora para ver los datos insertados:

curl -X GET "http://IP-SERVER-ELASTIC:9200/alumnos/_search?pretty"

**ENTREGABLE PARTE 4**: Captura de pantalla de la consola del cluster Hadoop con el resultado la consulta.


**PARTE 5 - Opcional. KIBANA**

Para esta parte, debemos hacer un sencillo Dashboard.

En nuestro navegador, visitemos: http://IP_ELASTIC_SERVER:5601

**ENTREGABLE 5**: Opcional. Captura de pantalla de la consola de Kibana con alguna visualización sencilla.
