# kafka-spark-hive
L'idée de ce projet est de vous fournir un bootstrap pour notre architecture Lambda. Nous abordons certains des principaux défis auxquels tout le monde est confronté lorsqu'il commence avec le Big Data. Ce projet vous aidera certainement à comprendre le monde du traitement des données et vous fera gagner beaucoup de temps dans la configuration de votre architecture Lambda initiale.

Dans cet article de blog, je passerai en revue certains concepts et technologies que nous avons placés dans notre projet bootstrap Lambda. Je ne prévois pas d'approfondir les concepts et les outils - nous avons beaucoup d'articles à ce sujet - l'intention ici est de présenter un exemple d'application contenant les modèles, les outils et les technologies utilisés pour développer des applications de traitement de données volumineuses .

Dans ce projet, nous utiliserons une architecture Lambda pour analyser et traiter les données des véhicules connectés à l'IoT et envoyer les données traitées à un tableau de bord de surveillance du trafic en temps réel.

Un système de fichiers distribué est une application client/serveur qui permet aux clients d'accéder aux données stockées sur le serveur et de les traiter comme s'il s'agissait de leur propre ordinateur. Le système de fichiers distribués Hadoop (HDFS) est le principal système de stockage de données utilisé par les applications Big Data. Notre projet utilise l'architecture HDFS car elle fournit un moyen fiable de gérer des pools d'ensembles de données volumineuses. Plus important encore, avec HDFS, nous avons un emplacement centralisé où tout travailleur Spark peut accéder aux données.

Notre architecture

![image](https://user-images.githubusercontent.com/82560535/211113119-840bee7e-796e-4c12-8e28-5e7af3c7d55e.png)

# Distributed File system

## Batch processing

Batch processing is responsible for creating the batch view from the master data set stored in the Hadoop distributed file system (HDFS). It might take a large amount of time for that file to be processed, so, for this reason, we also have the real-time processing layer. We are processing the batch data using Spark and storing the pre-computed views into Hivemetastore.

## Stream processing (speed layer)

Le traitement de flux nous permet de traiter les données en temps réel à mesure qu'elles arrivent et de détecter rapidement les conditions dans un court laps de temps. Du point de vue des performances, la latence du traitement par lots sera mesurée en minutes ou en heures, tandis que la latence du traitement par flux sera en secondes ou en millisecondes. Dans notre couche de vitesse, nous traitons les données en streaming à l'aide de Kafka avec le streaming Spark et les deux tâches principales sont effectuées dans cette couche.

## Serving layer

Une fois que la vue calculée à partir des couches de lot et de vitesse est stockée dans la base de données Hive, nous avons créé un processing pour tester la conformité  et la cohérance entre  le batch et le speed layer.


Mise en place d’un data lake pour la visualisation des tweets en utilisant Apache Kafka, Apache Spark, Apache Parquet  Pour réaliser ce projet on va créer un cluster Docker qui contient le conteneur Docker de Apache Hive 2.3.2 et le conteneur de Apache Zeppelin 0.9.0
Docker Compose
Service 	image
zeppelin 	apache/zeppelin :0.9.0
namenode 	bde2020/hadoop-namenode:2.0.0-hadoop2.7.4-java8
kafka 	        wurstmeister/kafka
datanode 	bde2020/hadoop-datanode:2.0.0-hadoop2.7.4-java8
hive-server 	bde2020/hive:2.3.2-postgresql-metastore
hive-metastore 	bde2020/hive:2.3.2-postgresql-metastore
hive-metastore-postgresql 	bde2020/hive-metastore-postgresql:2.3.0
presto-coordinator 	shawnzhu/prestodb:0.181
Docker Zeppelin

This repository contains Apache Zeppelin 0.9.0 docker image, which is tuned to work with BDE clusters.

To clone this Github repository:

    git clone https://github.com/SoufianeSrr/spark-project.git

Pour créer les conteneurs et les exécuter :

    docker-compose up -d

Hive
Hive Docker

This is a docker container for Apache Hive 2.3.2. It is based on https://github.com/big-data-europe/docker-hadoop so check there for Hadoop configurations. This deploys Hive and starts a hiveserver2 on port 10000. Metastore is running with a connection to postgresql database. The hive configuration is performed with HIVE_SITE_CONF_ variables (see hadoop-hive.env for an example).

To run Hive with postgresql metastore:

    docker-compose up -d

Hive Interpreter for Apache Zeppelin

Hive Interpreter has been deprecated and merged into JDBC Interpreter. You can use Hive Interpreter by using JDBC Interpreter with same functionality. See the example below of settings and dependencies.
Properties
Property 	Value
default.driver 	org.apache.hive.jdbc.HiveDriver
default.url 	jdbc:hive2://localhost:10000
default.user 	hiveUser
default.password 	hivePassword
Dependencies
Artifact 	Exclude
org.apache.hive:hive-jdbc:0.14.0 	
org.apache.hadoop:hadoop-common:2.6.0 	
Adding jars in the Zeppelin container org.apache.hive:hive-jdbc:0.14.0.jar AND org.apache.hadoop:hadoop-common:2.6.0.jar

    docker exec -u root -it zeppelin-test bash
    cd /opt/zeppelin/interpreter/jdbc
    wget -c https://repo1.maven.org/maven2/org/apache/hive/hive-jdbc/2.3.2/hive-jdbc-2.3.2-standalone.jar  -O hive-jdbc-2.3.2-standalone.jar 
    wget -c https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-common/2.6.0/hadoop-common-2.6.0.jar -O hadoop-common-2.6.0.jar

Configuration
Property 	Default 	Description
default.driver 	org.apache.hive.jdbc.HiveDriver 	Class path of JDBC driver
default.url 	jdbc:hive2://localhost:10000 	Url for connection
default.user 		( Optional ) Username of the connection
default.password 		( Optional ) Password of the connection
default.xxx 		( Optional ) Other properties used by the driver
zeppelin.jdbc.hive.timeout.threshold 	60000 	Timeout for hive job timeout
zeppelin.jdbc.hive.monitor.query_interval 	1000 	Query interval for hive statement
zeppelin.jdbc.hive.engines.tag.enable 	true 	Set application tag for applications started by hive engines
Spark
Spark Interpreter for Apache Zeppelin

Apache Spark is a fast and general-purpose cluster computing system. It provides high-level APIs in Java, Scala, Python and R, and an optimized engine that supports general execution graphs. Apache Spark is supported in Zeppelin with Spark interpreter group which consists of following interpreters.
Name 	Class 	Description
%spark 	SparkInterpreter 	Creates a SparkContext/SparkSession and provides a Scala environment
%spark.pyspark 	PySparkInterpreter 	Provides a Python environment
%spark.ipyspark 	IPySparkInterpreter 	Provides a IPython environment
%spark.r 	SparkRInterpreter 	Provides an vanilla R environment with SparkR support
%spark.ir 	SparkIRInterpreter 	Provides an R environment with SparkR support based on Jupyter IRKernel
%spark.shiny 	SparkShinyInterpreter 	Used to create R shiny app with SparkR support
%spark.sql 	SparkSQLInterpreter 	Provides a SQL environment
%spark.kotlin 	KotlinSparkInterpreter 	Provides a Kotlin environment
Connect Hive Interpreter With Spark in Apache Zeppelin

To run a Spark application on the local/cluster, you need to set some configurations and parameters, that's what SparkConf helps. It provides configurations to run a Spark application. Initially, we will create a SparkConf object with SparkConf(), which will also load the values ​​of spark's Java system properties.* We can now set various parameters using the SparkConf object and their parameters will take precedence over system properties. The following code block contains the lines that define the basic configurations for connecting spark to hive.

Move a file from the local machine to a Docker container

docker cp fichier.csv 'identifiant du conteneur':/fichier.csv
