# Caso práctico NoSQL: Neo4J

## Caso de negocio

Para contextualizar, se parte de la complicada situación que se está atravesando en plena pandemia de la COVID-19. Hoy en día, se habla mucho sobre los planes de acción cuando la situación haya concluido y sobre cómo países y localidades comenzarán de nuevo a reactivar sus economías.

Una acción que la mayor parte de los países están tomando es el “rastreo de contactos”. Utilizando la tecnología que nos proporcionan nuestros teléfonos móviles, a través de aplicaciones como puede ser Radar Covid, se busca una forma de lograr el distanciamiento o aislamiento de personas vulnerables o en riesgo de contraer la enfermedad.

Esto ha despertado el interés por la utilidad que tienen las bases de datos orientadas a grafos para resolver este tipo de problemas, puesto que tan importante es ser capaces de identificar a contactos directos como indirectos.

Con estas bases de datos, se puede tratar de dar respuesta a las siguientes preguntas:

1. ¿Cómo se puede maximizar el efecto de las medidas de distanciamiento social?
2. ¿Cómo se puede maximizar el efecto de las aplicaciones de rastreo de contactos?
3. ¿Cómo se puede utilizar el poder predictivo de un grafo para averiguar quién de las conexiones de una persona tiene mayor riesgo de contraer la enfermedad?

## Dataset 

Está dividido en tres CSV:

### Personas

* Id de la persona (id_persona).
* Nombre de la persona (nombre_presona).
* Estado de la persona (estado_salud: Contagiado/Sano).
* Timestamp que representa el momento en que se notifica a una persona que se había realizado el test PCR, si está o no contagiada (hota_test_resultado).
* Latitud: coordenada de latitud del domicilio donde reside la persona (latitud_domicilio).
* Longitud: coordenada de longitud del domicilio donde reside la persona (longitud_domicilio). Corresponden a coordenadas de Valladolid y Salamanca.

### Ubicación

* Id del lugar (id_ubicacion).
* Nombre del establecimiento (nombre_establecimiento).
* Tipo de establecimiento (tipo_establecimiento).

### Visitas

Visitas de personas a distintos lugares (hospital, colegio, bar, restaurante, etc.) de dos ciudades: Valladolid y Salamanca.

* Identificador de la visita (id_visita).
* Identificador de la persona (id_ubicacion).
* Hora de comienzo de la visita (inicio_visita).
* Hora de fin de la visita (fin_visita).

### Relaciones

Hay tres tipos de relaciones entre los nodos:

* (Persona) [VISITA_EMPLAZAMIENTO] (Ubicacion).
* (Persona) [REALIZA_VISITA] (Visita) [A_ESTABLECIMIENTO] (Ubicacion).

## Preparación de Base de Datos en Neo4J

* Abriendo el fichero de configuración, se observa que las bases de datos se guardan en el directorio ```/var/lib/neo4j/data/databases```.
* Acudir al fichero de configuración de Neo4J y modificar la database activa. Por defecto, Neo4J utiliza la BBDD por defecto “graph.db". Para esto ejecutar: ```sudo nano /etc/neo4j/neo4j.conf```. Posteriormente, se procede a descomentar la línea de database activa y se cambia el nombre de la base de datos activa a “Rastreo_COVID.db”, que será la que se utilice para el caso de negocio.

![image](https://user-images.githubusercontent.com/29135836/217672159-9502744a-e481-4b15-a72c-4d231682ab77.png)

* También se debe cambiar la configuración del fichero neo4j.conf para que Neo4J permita importar datos desde la URL de archivos. Para esto, se comentan las líneas marcadas en blanco en las siguientes imágenes añadiendo una almohadilla al comienzo de la línea.

![image](https://user-images.githubusercontent.com/29135836/217674788-795f05ff-15d2-49fd-bf50-d020d6f1651d.png)

![image](https://user-images.githubusercontent.com/29135836/217674816-5687858b-202b-4282-b384-a8f9285e34b5.png)

* Se reinicia el servidor de Neo4J para que apliquen los cambios: ```sudo systemctl restart neo4j```

* Activar la opción que permite que el editor de queries inserte varias consultas (statements) de una vez.

![image](https://user-images.githubusercontent.com/29135836/217675744-b41a7553-5bc8-4c8b-b620-a38f2cdfce71.png)


## Carga de BBDD Neo4J a partir de CSVs

Se quieren crear 40 nodos con label "Persona", 10 nodos "Establecimiento" y 100 nodos "Visita" a partir de los tres CSV.

Para añadir los diferentes nodos de una vez se pueden cargar a partir de un CSV gracias al comando LOAD CSV WITH HEADERS FROM, utilizando la interfaz de Neo4J en el navegador.

### Crear los 40 nodos "Persona"

```
LOAD CSV WITH HEADERS FROM 'file:///home/imfbigdata/neo4j/Nodos_Persona.csv' AS csv
```

```
CREATE (p:Persona{id:toInt(csv.id_persona), nombre: csv.nombre_persona, estado: csv.estado_salud, hora_result_test: datetime(csv.hora_test_resultado), ubicacion_domicilio: point({x: toFloat(csv.latitud_domicilio), y: toFloat(csv.longitud_domicilio), crs:'wgs-84'})})
```

### Crear los 10 nodos "Establecimiento"

```
LOAD CSV WITH HEADERS FROM 'file:///home/imfbigdata/neo4j/Nodos_Establecimiento.csv' AS csv
```

```
CREATE (u:Ubicacion{id:toInt(csv.id_ubicacion), nombre: csv.nombre_establecimiento, tipo: csv.tipo_establecimiento})
```

### Crear Índices

```
CREATE INDEX ON: Ubicacion(id);
CREATE INDEX ON: Ubicacion(nombre);
CREATE INDEX ON: Ubicacion (tipo);
CREATE INDEX ON: Persona(id);
CREATE INDEX ON: Persona(nombre);
CREATE INDEX ON: Persona(estado);
CREATE INDEX ON: Persona(hora_result_test);
CREATE INDEX ON: Persona(ubicacion_domicilio);
```

### Crear los 100 nodos "Visita"

Se realiza la carga del CSV de visitas, la cual es bastante diferente a las dos anteriores porque, a la vez que se lee cada visita, se crearán las dos relaciones entre personas y visitas por un lado, y entre visitas y ubicaciones por otro, ya que, al introducir timestamps desde que se inicia hasta que finaliza la visita, se ha decidido incluir como nodo las visitas:

```
LOAD CSV WITH HEADERS FROM 'file:///home/imfbigdata/neo4j/Nodos_Visita.csv' AS csv

MATCH (p:Persona{id:toInt(csv.id_persona)}), (u:Ubicacion{id:toInt(csv.id_ubicacion)})

CREATE (p)-[:REALIZA_VISITA]->(v:Visita {id_visita: toInt(csv.id_visita), inicio_visita:datetime(csv.inicio_visita), fin_visita:datetime(csv.fin_visita) })-[:A_ESTABLECIMIENTO]->(u)

CREATE (p)-[vi:VISITA_EMPLAZAMIENTO{id: toInt(csv.id_visita), inicio_visita:datetime(csv.inicio_visita), fin_visita:datetime(csv.fin_visita)}]->(u)

set v.duration=duration.inSeconds(v.inicio_visita,v.fin_visita)

set vi.duration=duration.inSeconds(vi.inicio_visita,vi.fin_visita);
```

Además de la relación entre persona – visita – ubicación del primer CREATE, se ha añadido otra otra relación que añade una relación directa entre persona y ubicación. Es decir: hay dos sentencias CREATE porque se crean dos tipos de relaciones diferentes (REALIZA_VISITA y VISITA_EMPLAZAMIENTO).

Tanto al nodo visita como a esta última relación se les añade la propiedad “duracion”, que indica el tiempo que duró la visita. Aunque se puede pensar que se ha añadido algo de información redundante, lo cual es cierto, esto es algo normal en este tipo de BBDD.

De esta forma, cuando se quiera hacer una consulta sobre la duración de una visita sin prestar atención a la fecha en la que se produjo, se utilizará la segunda relación que se ha creado (persona - ubicacion), ya que devolverá mucho más rápido la información al no tener que prestar atención a los nodos visita.

### Crear los 4 nodos de "Ubicación"

```
CREATE (c:Ciudad {nombre:"Valladolid"})-[:PARTE_DE]->(r:Region {nombre:"Castilla y Leon"})-[:PARTE_DE]->(p:Pais {nombre:"España"})
```

```
MATCH (r:Region)

MATCH (p:Pais)

MERGE (c:Ciudad{nombre:'Salamanca'})-[:PARTE_DE]->(r)-[:PARTE_DE]->(p)
```

En la consulta anterior, primero se recuperan la region y el país ya existentes en la BBDD, y estos dos nodos recuperados son los que se utilizan para la consulta con la cláusula MERGE, de manera que esta clave lo que consigue es que busque si existe todo el patrón (Ciudad Salamanca PARTE DE Castilla y Leon PARTE DE España), y como no existe, lo crea. 

```
MATCH (c:Ciudad {nombre:"Valladolid"}), (u:Ubicacion) WHERE u.id < 6

MERGE (u)-[:PARTE_DE]->(c);

MATCH (c:Ciudad {nombre:"Salamanca"}), (u:Ubicacion) WHERE u.id > 5

MERGE (u)-[:PARTE_DE]->(c);
```
