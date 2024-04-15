# Proyecto Integrador Big Data

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/5336c72a-074d-4c55-907b-543c7fd00046)


Durante esta practica la idea es emular un ambiente de trabajo, desde un área de innovación solicitan construir un MVP (Producto viable mínimo) de un ambiente de Big Data donde se deban cargar unos archivos CSV que anteriormente se utilizaban en un datawarehouse en MySQl, pero ahora en un entorno de Hadoop.

Desde la gerencia de Infraestructura no están muy convencidos de utilizar esta tecnología por lo que no se asignó presupuesto alguno para esta iniciativa, de forma tal que por el momento no es posible utilizar un Vendor (Azure, AWS, Google) para implementar dicho entorno, es por esto que todo el MVP se deberá implementar utilizando Docker de forma tal que se pueda hacer una demo al sector de infraestructura mostrando las ventajas de utilizar tecnologías de Big Data.

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

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/3cc80940-f3e3-44a4-9e92-e47944acb731)



### Ejecución de entorno

En primer lugar nos ubicamos en la carpeta del proyecto:
```
cd Proyecto_Integrador
```

Ejecutamos la version 1 de docker-compose:

```
sudo docker-compose -f docker-compose-v1.yml up -d
```

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/d3180e3d-e995-43fc-b8ad-84ee819e42aa)

### Copia de los archivos ubicados en 'Datasets' al contenedor 'namenode'

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
***Nota:*** *donde están las x debemos colocar nuestra IP*

Luego nos dirigimos a *Utilities > Browse the file system:*

![tempsnip](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/9be6d630-f384-4a67-aed1-c6ddd5587bbf)

Si cumpliste con todos los pasos anteriores, debería aparecerte la carpeta data con los archivos .csv dentro de ella:

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/0181fe55-f732-4491-be50-c8676fbd6a48)



## 2) Hive

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/9dd71f32-c417-4f24-8c6b-76fd809505b6)




### Ejecución de entorno

Debemos usar el entorno docker-compose-v2, por lo que es necesario detener el anterior, en este caso vamos a detener todos los contenedores:

```
sudo docker stop $(sudo docker ps -a -q)
```

Recuerden que debemos estar en el directorio 'Proyecto_Integrador':

```
cd Proyecto_Integrador
```

Ejecutamos el entorno que necesitamos para este paso:

```
sudo docker-compose -f docker-compose-v2.yml up -d
```

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/14322d92-6425-4063-a797-f3b680ac2cda)

### Creación y población de las tablas

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


**Podemos chequear mediante consultas en hive que las tablas se crearon y poblaron con éxito:**

Ingresar a Hive:
```
hive
```

Realizar las consultas:
```
use integrador;
```

```
select COUNT(*) from venta;
```

***Nota: es importante establecer la cláusula limit ya que la mayoría de las tablas tienen miles de registros***

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/23ae6226-a04d-4114-870c-f4019fce5d00)

*Otra opción es entrar a HUE desde nuestro navegador utilizando nuestra IP y el puerto :8888*


## 3) Formatos de Almacenamiento

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/9775be53-7b81-42d1-8dee-02ecc3f42377)


Vamos a crear una nueva base de datos en la cual vamos a alojar todas las tablas creadas en el punto anterior, pero en formato **Parquet + Snappy** aplicando algunas particiones a las tablas.

Primero, copiamos el archivo 'Paso03.hql' al servidor de Hive:

```
sudo docker cp ./Paso03.hql hive-server:/opt/
```

Ingresamos al servidor de Hive y ejecutamos el archivo 'Paso03.hql':

```
sudo docker exec -it hive-server bash
```

```
hive -f Paso03.hql
```

*Recomendación: revisar el script Paso03.hql para ver y comprender los querys que estamos ejecutando*

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/fdab68e9-0907-45e8-8f6a-4cbd2ff7588a)


### Consultas para verificar que las tablas se crearon y poblaron

Ingresamos a Hive y seleccionamos la DB 'integrador2':

```
hive
```

```
use integrador2;
```

```
show tables;
```

#### Consultas

```
select IdTipoGasto, sum(Monto) from gasto group by IdTipoGasto;
```

```
select * from tipo_gasto;
```

```
select COUNT(*) from calendario;
```

```
select COUNT(*) from gasto;
```

```
select COUNT(*) from venta;
```

```
select COUNT(*) from proveedor;
```


![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/51f3abd8-c5cf-4db3-ad53-9e6c11c3640e)


## 4) SQL

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/14d2ca45-14c7-4329-acd6-be4973202113)


#### Ejecutamos consultas con funciones de agregado en la tabla venta y registramos el tiempo:

```
select IdProducto, SUM(Precio * Cantidad) FROM venta GROUP BY IdProducto;
```

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/79aa8646-0a0e-49e7-93da-b1d3f98e2fe3)


```
select IdCliente, SUM(Precio * Cantidad) FROM venta GROUP BY IdCliente;
```

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/51b66f98-2787-4b87-81cf-f062e7ba724e)


```
select IdCanal, SUM(Precio * Cantidad) FROM venta GROUP BY IdCanal;
```

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/e7abcf46-9b1b-4e19-a45f-70340daed55b)


```
select IdEmpleado, SUM(Precio * Cantidad) FROM venta GROUP BY IdEmpleado;
```

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/c53c71f5-f0e6-4e76-8fe7-191010a03974)


```
SELECT v.IdCliente, SUM(v.Precio * v.Cantidad) FROM venta v JOIN cliente c USING (IdCliente) WHERE c.Localidad = 'CIUDAD DE BUENOS AIRES' GROUP BY v.IdCliente;
```

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/73c0944c-c727-4a8f-8e95-8a3482436163)


#### Creamos los índices correspondientes en la tabla venta y cliente:

```
CREATE INDEX index_venta_producto ON TABLE venta(IdProducto) AS 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler' WITH DEFERRED REBUILD;
```

```
CREATE INDEX index_venta_cliente ON TABLE venta(IdCliente) AS 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler' WITH DEFERRED REBUILD;
```

```
CREATE INDEX index_venta_canal ON TABLE venta(IdCanal) AS 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler' WITH DEFERRED REBUILD;
```

```
CREATE INDEX index_venta_empleado ON TABLE venta(IdEmpleado) AS 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler' WITH DEFERRED REBUILD;
```

```
CREATE INDEX index_cliente ON TABLE cliente(IdCliente) AS 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler' WITH DEFERRED REBUILD;
```

```
CREATE INDEX index_venta ON TABLE venta(IdVenta) AS 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler' WITH DEFERRED REBUILD;
```

#### Repetimos las consultas anteriores y comparamos los resultados:

```
select IdProducto, SUM(Precio * Cantidad) FROM venta GROUP BY IdProducto;
```

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/a2ce2608-2de3-4e92-9906-4afc12d27b8b)


```
select IdCliente, SUM(Precio * Cantidad) FROM venta GROUP BY IdCliente;
```

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/47e10188-d25e-42f0-971b-bee01bb4986a)


```
select IdCanal, SUM(Precio * Cantidad) FROM venta GROUP BY IdCanal;
```

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/3ea2e9b2-f624-4e52-a61f-9d8a2aaaf8f0)

```
select IdEmpleado, SUM(Precio * Cantidad) FROM venta GROUP BY IdEmpleado;
```

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/64d79a80-d790-4191-96ca-4807994e136f)


```
SELECT v.IdCliente, SUM(v.Precio * v.Cantidad) FROM venta v JOIN cliente c USING (IdCliente) WHERE c.Localidad = 'CIUDAD DE BUENOS AIRES' GROUP BY v.IdCliente;
```

![image](https://github.com/SantosIparraguirre/Proyecto_Integrador/assets/154923689/d2ba14af-911f-4ab1-9210-230270440102)



5) ## No-SQL

#### Ejecución de entorno

En este caso vamos a utilizar **HBase, MongoDB, Neo4J y Zeppelin** por lo que vamos a detener los contenedores nuevamente:

```
sudo docker stop $(sudo docker ps -a -q)
```

Ejecutamos la versión 3 de docker-compose que contiene lo necesario para este paso:

```
sudo docker-compose -f docker-compose-v3.yml up -d
```

#### 1) HBase

Ingresamos a HBase:

```
sudo docker exec -it hbase-master hbase shell
```

Creamos las siguientes tablas:

```
create 'personal','personal_data'
```

```
list 'personal'
```

Ingresamos los datos:

```
put 'personal',1,'personal_data:name','Juan'
```

```
put 'personal',1,'personal_data:city','Córdoba'
```

```
put 'personal',1,'personal_data:age','25'
```

```
put 'personal',2,'personal_data:name','Franco'
```

```
put 'personal',2,'personal_data:city','Lima'
```

```
put 'personal',2,'personal_data:age','32'
```

```
put 'personal',3,'personal_data:name','Ivan'
```

```
put 'personal',3,'personal_data:age','34'
```

```
put 'personal',4,'personal_data:name','Eliecer'
```

```
put 'personal',4,'personal_data:city','Caracas'
```

Consulta para verificar:

```
get 'personal','4'
```

