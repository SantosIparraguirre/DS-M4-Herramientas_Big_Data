# Proyecto Integrador

Durante esta practica la idea es emular un ambiente de trabajo, desde un área de innovación solicitan construir un MVP(Producto viable mínimo) de un ambiente de Big Data donde se deban cargar unos archivos CSV que anteriormente se utilizaban en un datawarehouse en MySQl, pero ahora en un entorno de Hadoop.

Desde la gerencia de Infraestructura no están muy convencidos de utilizar esta tecnología por lo que no se asigno presupuesto alguna para esta iniciativa, de forma tal que por el momento no es posible utilizar un Vendor(Azure, AWS, Google) para implementar dicho entorno, es por esto que todo el MVP se deberá implementar utilizando Docker de forma tal que se pueda hacer una demo al sector de infraestructura mostrando las ventajas de utilizar tecnologías de Big Data.

## Entorno Docker con Hadoop, Spark y Hive

Se presenta un entorno Docker con Hadoop (HDFS) y la implementación de:
* Spark
* Hive
* HBase
* MongoDB
* Neo4J
* Zeppelin
* Kafka

Es importante mencionar que el entorno completo consume muchos recursos de su equipo, motivo por el cuál, se propondrán ejercicios pero con ambientes reducidos, en función de las herramientas utilizadas.

Como primer paso fundamental, debemos clonar el repositorio en nuestra máquina virtual:
```
sudo git clone https://github.com/SantosIparraguirre/Proyecto_Integrador.git
```

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/4ff0bc82-e73b-400f-8b6a-413250c7b4ca)

Ejecute `docker network inspect` en la red (por ejemplo, `docker-hadoop-spark-hive_default`) para encontrar la IP en la que se publican las interfaces de hadoop. Acceda a estas interfaces con las siguientes URL:

```
Namenode: http://<IP_Anfitrion>:9870/dfshealth.html#tab-overview
Datanode: http://<IP_Anfitrion>:9864/
Spark master: http://<IP_Anfitrion>:8080/
Spark worker: http://<IP_Anfitrion>:8081/	
HBase Master-Status: http://<IP_Anfitrion>:16010
HBase Zookeeper_Dump: http://<IP_Anfitrion>:16010/zk.jsp
HBase Region_Server: http://<IP_Anfitrion>:16030
Zeppelin: http://<IP_Anfitrion>:8888
Neo4j: http://<IP_Anfitrion>:7474
```

## 1) HDFS


#### Ejecución de entorno

En primer lugar nos ubicamos en la carpeta del proyecto:
```
cd Proyecto_Integrador
```

Ejecutamos la version 1 de docker-compose:

```
sudo docker-compose -f docker-compose-v1.yml up -d
```

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/d3180e3d-e995-43fc-b8ad-84ee819e42aa)

#### Copia de los archivos ubicados en 'Datasets' al contenedor 'namenode'

Creamos el directorio 'Datasets' dentro del namenode y salimos del mismo:

```
sudo docker exec -it namenode bash
```

```
cd home
```

```
mkdir Datasets
```

```
exit
```

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/97ca34cc-9d7a-4aa3-b5c8-1bf52106ab2e)


Ejecutamos el archivo 'Paso00.sh', que contiene los comandos para copiar los archivos al namenode. Primero modificamos su permiso de ejecución:

```
sudo chmod +x Paso00.sh
```

```
sudo ./Paso00.sh
```

Ingresamos al contenedor 'namenode':

```
sudo docker exec -it namenode bash
```

Nos ubicamos en el directorio 'home':

```
cd home
```

Creamos el directorio 'data':

```
hdfs dfs -mkdir -p /data
```

Pegamos los archivos:

```
hdfs dfs -put /home/Datasets/* /data
```

**También podemos ingresar a la interfaz de hadoop para verificar que los archivos esten en el HDFS, utilizando nuestro navegador, pegando la IP de nuestra máquina virtual y luego el puerto :9870.**

Ejemplo:

```
http://xxx.xxx.x.xxx:9870/
```

Luego nos dirigimos a *Utilities > Browse the file system*:

![tempsnip](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/9be6d630-f384-4a67-aed1-c6ddd5587bbf)

Si cumpliste con todos los pasos anteriores, debería aparecerte la carpeta data con los archivos .csv dentro de ella.


## 2) Hive

Debemos usar el entorno docker-compose-v2, por lo que es necesario detener el anterior, en este caso vamos a detener todos los contenedores:

```
sudo docker stop $(sudo docker ps -a -q)
```

Ejecutamos el entorno que necesitamos para este paso:

```
sudo docker-compose -f docker-compose-v2.yml up -d
```

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/14322d92-6425-4063-a797-f3b680ac2cda)

**Vamos a ejecutar el script 'Paso02.hql' para crear y poblar las tablas, por lo tanto debemos primero copiarlo de nuestra máquina virtual al server de Hive:**

```
sudo docker cp ./Paso02.hql hive-server:/opt/
```

Ingresamos al server de Hive:

```
sudo docker exec -it hive-server bash
```

Ejecutamos el script configurado previamente para crear y poblar las tablas:

```
hive -f Paso02.hql
```

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/241884bc-6caf-4e8f-9389-1c8038568865)
