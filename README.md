# Caso práctico NoSQL: Neo4J

## Caso de negocio

Para contextualizar, se parte de la complicada situación que se está atravesando en plena pandemia de la COVID-19. Hoy en día, se habla mucho sobre los planes de acción cuando la situación haya concluido y sobre cómo países y localidades comenzarán de nuevo a reactivar sus economías.

Una acción que la mayor parte de los países están tomando es el “rastreo de contactos”. Utilizando la tecnología que nos proporcionan nuestros teléfonos móviles, a través de aplicaciones como puede ser Radar Covid, se busca una forma de lograr el distanciamiento o aislamiento de personas vulnerables o en riesgo de contraer la enfermedad.

Esto ha despertado el interés por la utilidad que tienen las bases de datos orientadas a grafos para resolver este tipo de problemas, puesto que tan importante es ser capaces de identificar a contactos directos como indirectos.

Con estas bases de datos, se puede tratar de dar respuesta a las siguientes preguntas:

1. ¿Cómo se puede maximizar el efecto de las medidas de distanciamiento social?
2. ¿Cómo se puede maximizar el efecto de las aplicaciones de rastreo de contactos?
3. ¿Cómo se puede utilizar el poder predictivo de un grafo para averiguar quién de las conexiones de una persona tiene mayor riesgo de contraer la enfermedad?

# Dataset 

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
