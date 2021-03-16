# Workflow TUMatcher: [TU, TSSs, TTS ]HT DataSet vs RegulonDB

MAPEO TUS, TSSs y TTSs entre datos de HT y datos de RegulonDB

 

## Objetivo 
Identificar cuáles [TUs, TSSs, TTSs] ya existen y cuales no. A los objetos ya existentes se les agrega la evidencia y referencia nueva, los que no existen se dan de alta como objetos nuevos.


## Determinación de umbrales usando known data

### 1.- Determinar el umbral para TSS**

Antes de determinar el umbral, primero tenemos que saber la distribución de distancias entre TSSs de los TSS conocidos en RegulonDB.

**A. Distancia entre TSS y su vecino anterior en la misma orientación [RegulonDB]**

Distancias de los Promotores anotados en RegulonDB usando posición y strand.

Pasos:

* Se filtraron todos los TSS de Morett, todo aquel promotor con nombre *TSS_*.
* Se filtraron los promotores sin posición y sin strand.
* Se obtuvo la distancia de todos los promotores respecto a su Promotor Anterior Vecino en el mismo strand
* Se reportan solo aquellos promotores cuya distancia es menor a 800 bp.

**B.Distancia entre TSS y su vecino anterior asociados a un gene**

No todos los TSS estan asociados a genes. Una opción es calcular la distancia entre TSS que tienen el mismo nombre [ese nombre usa el gene] y el subfijo p#, donde # es un número consecutivo.

Pasos:

* Se filtraron todos los TSS de Morett, todo aquel promotor con nombre TSS
* Se eliminaron los promotores sin posición, y todos aquellos que eran IS, porque tienen el mismo nombre y en distintas regiones.
* A Todos los promotores se les quitó el postfijo p1,p2,p3,etc para quedarnos solo con el nombre (del gene)
* A todos los promotores con el mismo nombre de promotor/gene se les agrupó
* Se calculó la distancia de todos los promotores todos contra todos   A B C -->  AB AC  BC
* Se reportaron todas las distancias obtenidas



#### Datasets vs [TSS,TTS,TU] RegulonDB

##### Comparación entre TSS

Distancias entre Promotores conocidos y del Datasets a Evaluar de TSS-HT

Pasos:

* Se proceso el Dataset a Evualuar para que tuviera el mismo formato que los promotores de RegulonDB, se marcaron como TSS_EXP##
* Se mezclaron ambos archivos, promotores de RegulonDB y del Dataset a evaluar.
* Se calcularon las distancias de todos los promotores del archivo mezclado, siguiendo los pasos y filtros descritos en la Pestaña 1
        Cálculo de distancias del Promotor anterior
* Se filtraron todos los promotores tipo TSS_EXP con cálculo de distancia relacionado con un promotor de RegulonDB.

##### Conclusión

**"Ya revisamos y concluímos que la distancia máxima para que dos TSSs pertenezcan a un mismo promotor es de 5 pares de bases."(JCV)**


### 2.- Determinar el umbral para TTSs**

No sabemos si los TTSs van a mapear DENTRO de Terminadores existentes. Necesitamos saber cuantos mapean, para entender mejor cómo funcionan los terminadores. Pasa que los terminadores actuales son regiones de “hairpins” o estructrura del DNA que puede ser que vaya antes o después del mero sitio de terminación.


B) Terminadores.
En todos los casos necesitamos saber la distancia de la posición de los terminadores del dataset a evaluar vs terminadores en RegulonDB, indicando si están dentro del terminador de Regulon, o a x distancia arriba o a x distancia abajo. Una vez teniendo estos datos tendremos que analizarlos para tomar la sig decisión.


## Programa TU-maker        


1. TSS-HT match [Buscando/Creando el promotor]

* Si un TSS-HT esta a una distancia > 5 bp de un TSS-RegulonDB
	* Se crea un nuevo promotor _[indicar valores ademas de la posición]_
	* Se asigna su evidencia
	* Se asigna su referencia

* De otro modo, Si un TSS-HT esta a menos de 5bp de distancia de UNO Y SOLO UN TSS-RegulonDB; Y no existe otro TSS-HT con una distancia <= 5bp al mismo TSS-RegulonDB, entonces
	* Asignar evidencia al TSS-RegulonDB "_[indicar evidencia]_".
	* Asignar una referencia al TSS-RegulonDB y asociada a la referencia "_[indicar referencia]_".

* De otro modo SI Hay MAS DE UN TSS-HT asignado A UNO Y SOLO UN TSS-RegulonDB con distancia <= 5bp, entonces
 	* Se asigna el TSS-HT más cercano al TSS-RegulonDB.
 	* El otro TSS-HT se crea como nuevo.	
     "Ojo si tenemos dos promotores nuevos a menos de 5pb de uno conocido, pero digamos que uno está a 3pb por la derecha y el otro a 4 pb por la izquierda, entonces dejemos el más cercano (3 ob) como parte del promotor conocido y el otro será un TSS de un nuevo promotor." (JCV)

Al final de este paso, tendremos un promotor, sea que ya exista uno o que se haya creado.


2.TTS-Match [Buscando/Creando un TTS]


Al final de este paso, tendremos un terminador, sea que ya exista uno o se haya creado.



3.TUs-Match

* Por cada uno de los TSS [sea nuevo o ya existente en RegulonDB -ver punto 1 TSS-Match]
	* Tomar su TTs [sea nuevo o ya existente en RegulonDB]
	* Verficar si existe una TU, con ese TSS y TTS
		* Si existe SOLO una TU, entonces
			* Asignar evidencia a la TU existente _[Evidencia]_
			* Asignar referencia a la TU existente _[Referencia]_
		* Sino existe ninguna TU, entonces
			* Crear TU
			* Asignar evidencia a la TU  _[Evidencia]_
			* Asignar referencia a la TU _[Referencia]_


A.I) Los TSSs del dataset a evaluar que caigan en un promotor conocido.
Con estos debemos ver si las TUs correspondientes (la de Regulon y la del dataset) cubren los mismos genes. 
A.I. 1. Si no lo hacen, son dos TUs diferentes, que arrancan en el mismo promotor.
A.I. 2. Si sí cubren a los mismo genes pasamos a la sig etapa: comparar la terminación.

A.II) Los TSSs del dataset a evaluar que definen un nuevo promotor, por definición definen una nueva TU.
En estos lo que hay que ver es donde caen sus terminadores para saber si son terminadores nuevos o no, o sea caso (B).



Al final de todo este proceso podremos contar cuantas TUs mapean con ya conocidas y cuantas son nuevas ya sea porque el promotor es diferente o no, porque no cubren a los mismos genes aunque arranquen del mismo promotor, o porque tengan una terminación diferente.
