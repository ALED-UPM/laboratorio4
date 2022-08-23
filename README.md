# laboratorio4

En el laboratorio 3 desarrollamos algoritmos efecicientes para cacultar el conjunto
de paradas incluidas en una circunferencia cuyo centro es una parada de autobús
y el radio una cierta distancia en metros. Ese conjunto de paradas están ordenadas
en función de la proximidad de la parada que fija el centro de la circunferencia.

En esa práctica calculabamos las paradas próxima a otra parada, pero el cálculo
se hacía para una sola parada. El calculo lleva asociado un proceso de filtrado
para recuperar las paradas que están a menos de una cierta distancia, y de un proceso de
ordenación de las paradas.

Si incluyesemos esos cálculos en un proceso de búsqued de las rutas de los viajes,
el cálculo de las rutas puede hacerse demasiado lento. Podemos hacer esos cálculos
para todas las paradas de Madrid, y mantenerlo calculado para el resto de ejecuciones.
Podemos hacer ese cálculo en el proceso de inicialización (por ejemplo, en paralelo con
la carga de los tiempos de paradas que lleva asociado un fichero muy grande y su inicialización es lenta).

Calcular una a una todas las paradas próximas a cada parada puede tener un tiempo
de computo importante, pero este conjunto de cálculos es facilmente paralelizable.
En vez de calcular todas las paradas próximas a cada parada en un único procesador,
podemos repartir los cálculos entre un conjunto de hebras, y todas esas hebras
realizan los mismos cálculos, pero para un conjunto diferemte de paradas. Si
tenemos tantas hebras como procesadores tenga nuestre ordenador (puede ser facilmente entre 4 y
16 unidades de procesamiento), podemos reducir del orden de 5-10 veces el tiempo de cómputo
(esto depende del número de unidades de cómputo de nuestro ordenador, y de cómo de cargado de procesos ejecutándose
esté nuestro ordenador).

## Servicios de ejecución paralela de proximidad de paradas

En esta práctica daremos por implementado el método getNearStops de la clase StopServicesP3 de la práctica anterior.
El paquete es.up.dit.aled.p3 de la práctica anterior se puede copiar completo en este proyecto Eclipse, y estaremos
utilizando esa clase y método. Si no lo copiamos estaremos útilizando una biblioteca binaria incluida en el
proyecto, esa biblioteca incluye una implementación eficiente de ese método.

En esta práctica vamos a crear una nueva clase StopServicicesP4 que implementa el interfaz IStopServicesP4 que incluye
un conjunto de métodos para paralelizar el cálculo de las paradas próximas a otras paradas. Esta implementación estará
basada en un pool de Threads. Ese pool dedicará un número limitado de threads a hacer los cálculos de paradas próximas
que se pide calcular. El constructor de StopServicesP4 permite fijar el número máximo de threads con los que va a trabajar
esta implementación. Si el número que fijamos es 1, esta clase no podrá calcular paradas próximas para mas de una parada
en un instante cualquiera, y si el número es el número de unidades de cómputo de nuestro ordenador, podemos tener todas esas unidades
haciendo estos cálculos en un instante dado (depende de las necesidades de cómputo de nuestro ordenador en ese instante).
El interfaz IStopServicesP4 incluye dos operaciones básica:

```java
public void startComputeNearStops(Stop[] stops,int from, int to, double distance);
public Map<Stop,List<Stop>> getStartedComputeResults();
```

startComputeNearStops arranca el cálculo de las paradas próximas para un conjunto de paradas. Recibe como parámetro
un array de paradas, y para ese array calculará parádas próximas para las paradas que van desde from (incluído) hasta
to (excluído). Este método pedirá al pool de threads (mediante el método submit del pool) que ejecute una tarea que 
devolverá como resultado un Map<Stop,List<Stop>> (el parámetro genérico de Callable), que incluye como claves las mismas 
paradas que fijan stops, from y to. Para cada clave, el valor asociado en el mapa es la lista de paradas próximas para 
esa clave, con lo que el mapa nos permite incluir todos los resultados. startComputeNearStops arranca el
cálculo y retorna aunque no haya terminado de calcular. Será el pool de threads quién se encargue de hacer el cálculo
cuando haya alguna hebra disponible.
  
Después de llamar a startComputeNearStops se podrán hacer mas llamadas a startComputeNearStops (para calcular para
indices from-to diferentes, aunque el array sea el mismo) y cuando todos los cálculos estén arrancados llamaremos 
a getStartedComputeResults, para esperar a que todos los cálculos estén terminados, y nos devolverá todos los cálculos hechos. 
Los resultados para todos los cálculos se devolverán juntos en un único mapa (cada cálculo arrancado devuelve un mapa,
getStartedComputeResults espera a que vayan terminando los cálculos y junta todos los mapas devueltos en un único mapa).
  
El interfaz IStopServiesP4 incluye dos métodos adicionales:
  
  ```java
public ThreadPoolExecutor getExecutor();
public void shutdownThreadPool();
```
  
getExecutor se podrá emplear únicamente en pruebas, para poder comprobar que el pool de threads está
correctamente construído. Y shutdownThreadPool que lo podremos utilizar al final de la ejecución de nuestro
programa, y con eso nos aseguramos que todas las hebras están terminadas. Cuando no quedan hebras vivas, la máquina virtual
java da por terminado el programa (si quedan hebras sin terminar, la ejecución de nuestro programa no termina, y si no
damos por terminadas las hebras del pool de threads, el programa no termina nunca porque las hebras estarán esperando nuevos
trabajos a ejecutar). El siguiente método de la clase java.lang.Runtime nos devuelve el número de unidades de cómputo de nuestro
ordenador:
	
```java
Runtime.getRuntime().availableProcessors()
```
	
La clase StopSevicesP4 incluye un constructor que recibe como parámetro un QTFSStop a partir del cual podemos obtener 
todas las paradas de autobús, y un parámetro que fija el número máximo de threads que puede ejecutar el pool en un instante dado:

```java
public StopServicesP(Stop.GTFSStops factory,int maxThreads) { ... }
```

La clase StopServicesP4 debe implementar el interface IStopServicesP4 que no podemos modificar en la práctica.

	
