# Trabajo Práctico 1 de Arquitectura del Software (75.73/TB034) del 2do cuatrimestre de 2026

> **La fecha de entrega para el informe y el código es el jueves 01/10** :bangbang:

La forma de entrega será a través de un canal **privado** del grupo en Slack, al que deben invitar a los docentes. Deben poner ahí un link al repositorio con el código y el informe (o avisar si está en el repositorio).

El informe debe entregarse en formato PDF. **Debe** incluir screenshots del dashboard de métricas para cada caso analizado que permitan observar los resultados obtenidos.

## Objetivos

El objetivo principal es comparar algunas tecnologías, ver cómo diversos aspectos impactan en los atributos de calidad (QA) y probar cuáles tácticas se podrían implementar para mejorarlos.
El objetivo menor es que aprendan a usar una variedad de tecnologías útiles y muy usadas hoy en día, incluyendo:

- Node.js (+ Express)
- Docker
- Docker Compose
- Nginx
- Algún generador de carga (la propuesta es usar Artillery, pero pueden cambiarlo)
- Alguna forma de tomar mediciones varias y visualizarlas, preferentemente en tiempo real, con persistencia, y en un dashboard unificado (la propuesta es usar el plugin de Artillery + cAdvisor + StatsD + Graphite + Grafana, pero pueden cambiarlo).

## Antecedentes

La startup **arVault** es una fintech que opera una billetera digital, de reciente creación. Su fundador, un desarrollador aficionado y entusiasta con algo de dinero y muchas ideas, consciente de que la clave de su negocio es la implementación de su backend, tercerizó el desarrollo de las apps para dispositivos móviles, y se concentró en implementar rápidamente (bajo la consigna _fake it until you make it_) un núcleo de servicios que le permitieran conseguir más fondos a través de inversores.

Luego de la ronda inicial, los primeros fondos fueron utilizados, no para robustecer la arquitectura existente, sino para agregar más funcionalidad. Una de estas funcionalidades nuevas permite abrir cuentas en distintas monedas (que se respaldan en bancos existentes), y realizar operaciones de cambio entre éstas. El diferencial de arVault es tener la tasa de cambio más conveniente dentro de las aplicaciones que proveen este servicio. Para ganar usuarios, las tasas de cambio no tienen gap entre el valor de compra y de venta.

Si bien el lanzamiento fue exitoso, comenzaron a aparecer reclamos de los usuarios sobre problemas en el uso de la función de cambio de monedas. Estos reclamos llegaron en el peor momento, dado que arVault necesita captar más inversiones y, debido a la caída de la reputación y las reviews negativas, los potenciales inversores se niegan a aportar más fondos si no se realiza una auditoría y se mejora el servicio.

Frente a este reclamo, arVault decide contratar a un grupo de arquitectos (ustedes) para que evalúen, propongan e implementen soluciones que mejoren los atributos de calidad del servicio de cambio de monedas.

## Consigna

Realizar un análisis de la arquitectura, código e infraestructura recibidos. Determinar, indicar y **justificar** cuáles son los QA clave para este servicio.

Estudiar cómo los distintos QAs se ven influenciados según las decisiones de diseño que tomó el desarrollador (recorrer los QAs vistos en clase, incluir los que no son clave). Realizar una crítica fundamentada.

Hacer pruebas de carga, obtener y graficar métricas relevantes, para tener un panorama del comportamiento del servicio.

Proponer e implementar modificaciones aplicando tácticas. Verificar el impacto sobre los distintos QAs y mostrar resultados (métricas en los casos mensurables, ejecuciones en los otros). Si la evidencia no permite ver una diferencia, o si empeora algún atributo, discutir los motivos. Analizar el tradeoff entre distintos atributos (siempre aplicado a lo que implementen en el TP).

Realizar un **diagrama Components & Connectors** para el "caso base" (el recibido) y para todos los casos en los que se altere la arquitectura.

El informe debe estar correctamente redactado, asumiendo que quienes vayan a leerlo comprenden los conceptos de arquitectura, pero necesitan ver justificaciones de las recomendaciones.

### Pedidos adicionales

El fundador de arVault tiene un par de pedidos adicionales al análisis:

1. **[OBLIGATORIO]** Enterado de que se van a utilizar métricas para analizar el servicio, solicita que se agreguen métricas que muestren el **volumen** operado en cada moneda (compras y ventas sumadas por moneda), como así también el **neto** (compras suman y ventas restan), ambos a medida que transcurre el tiempo. Estas métricas deberían aparecer en el dashboard de alguno de los casos estudiados.
2. **[OPCIONAL]** Más allá de las tácticas que prueben para favorecer distintos QA, él les comenta que, según le parece, el servicio funcionaría mejor si la información que se almacena actualmente en archivos `.json` fuera almacenada en un base de datos externa (propone usar [Redis](https://redis.io/es/), pero pueden utilizar cualquier otro motor que prefieran). No sabe a cuáles QA impactaría (recién está leyendo sobre el tema), pero está dispuesto a dar un bonus para que este cambio forme parte del análisis e implementación y se discutan en el informe los pros y contras. Este pedido es **opcional** en esta entrega, recomendamos hacerlo si ven que el resto del TP está hecho de manera coEn los atributos mensurables, obtener métricas y mostrar evidencia que permita constatar que las modificaciones impactaron positivamente. Para los demás atributos, presentar nsistente y balanceada. Tengan en cuenta que en el TP 2 va a aparecer de manera obligatoria. Si lo prueban ahora, ganan tiempo para el siguiente TP.

## Desarrollo

- El servicio se encuentra en el directorio `app/`
- La única documentación de la que disponen es
  - El README del servicio, escrito por el desarrollador/fundador de arVault
  - Los comentarios en el código
  - Una colección de [Postman](https://www.postman.com/) en el directorio `doc/` que pueden utilizar para probar el servicio.
- En `docker-compose.yml` tienen la solución tal como corre en el único servidor disponible al momento de comenzar el proyecto.
- Para cambiar la configuración de nginx deben editar el archivo `nginx_reverse_proxy.conf`.

Para generar carga y ver las mediciones obtenidas, en el directorio `perf/` tienen un dashboard de Grafana ya armado (`dashboard.json`) al que **deberán ajustar según las características de su equipo de pruebas (RAM, cores)**, y al que pueden modificar agregando métricas o alterando las visualizaciones. Además, tienen un ejemplo de un escenario básico de Artillery (**deben** crear sus propios escenarios de manera apropiada para lo que estén probando). También hay un script y una configuración en el `package.json` para que puedan ejecutar los escenarios corriendo:

```./run-scenario.sh <filename> <env>```

donde `<filename>` es el nombre del archivo con el escenario (sin la extensión `.yaml`) y `<env>` es el entorno en el cual correrá la prueba (vean la sección `environments` dentro del yaml del escenario).

### Generación de carga para las pruebas

> **Importante**: Generen valores de carga que tengan relación con los tiempos que ven en la aplicación. No agrega valor que generen una carga enorme y luego cueste saber cuál de todos los componentes está fallando. Vayan de a poco con la carga y verifiquen cómo se van afectando los atributos de calidad.

Hay muchos tipos de escenarios de carga y pruebas de performance en general. Pueden leer por ejemplo [aquí](https://www.softwaretestingclass.com/what-is-performance-testing/) (o en cualquiera de los miles de links al googlear sobre el tema) sobre algunos tipos de escenarios que pueden implementar. Queda a decisión de cada grupo elegirlos, considerando siempre cuál es el que más útil les resulta para analizar lo que quieran estudiar.

## Tener en cuenta

- El tráfico entre el cliente y el servidor debe pasar por el nginx, para que tenga la latencia del salto extra.
- Asumimos que existe un componente que se encarga de los aspectos de **autenticación** y **autorización**, y que dicho componente solo permite que nos lleguen los requests apropiados. Tener en cuenta que ese servicio _solo se encarga de esos dos aspectos de Seguridad_.
- Para este TP, simplificamos la dependencia en un servicio de transferencia de fondos, que siempre funciona. La demora de cada request es generada al azar entre 200 y 400 milisegundos.

## Condiciones sobre la entrega

1. El trabajo debe entregarse **completo**. No se aceptan entregas parciales.
2. **Toda** afirmación que se realice debe estar apoyada en **evidencia**. Si se encuentra alguna afirmación que no está correctamente fundamentada con evidencia, se disminuirá la nota. Esto incluye las observaciones que realicen sobre el código y su funcionamiento en general. Que no existan métricas para un determinado QA no implica que no pueda evidenciarse el efecto de una táctica.
3. Asumimos que todo el grupo participa en la resolución del trabajo. De ocurrir problemas o surgir contratiempos, es el grupo quien debe responder y solucionarlos. Pueden consultar a los docentes pero deben demostrar primero que intentaron solucionarlos internamente.
4. Si utilizan uno o más agentes o herramientas de IA en la realización del TP, deberán agregar un apartado en el que detallen cuáles fueron los usos específicos que les dieron. Si se detecta un uso de IA no declarado se lo tomará como una falta grave.
5. De haber defectos importantes en el desarrollo o en el informe del TP, se solicitará una re-entrega. Esto tiene un impacto considerable en la nota final, por lo que les recomendamos que controlen entre todo el grupo el cumplimiento del enunciado, las conclusiones y las justificaciones antes de entregar el trabajo. Una vez más, no se considera a un TP hecho parcialmente como un caso de re-entrega. Tampoco se aceptan entregas con secciones sin desarrollar o con contenido de relleno.

-----------

## Links útiles

- Node.js:
  - https://nodejs.org/
  - https://github.com/creationix/nvm
- Express:
  - https://expressjs.com/en/starter/hello-world.html
- Nginx:
  - https://nginx.org/
- Redis:
  - https://redis.io/
  - https://www.npmjs.com/package/redis
- Docker:
  - https://docker-k8s-lab.readthedocs.io/en/latest/docker/docker-engine.html
  - https://www.docker.com/
- Docker-compose:
  - https://docs.docker.com/compose/
- StatsD:
  - https://github.com/etsy/statsd
  - https://github.com/etsy/statsd/blob/master/docs/graphite.md
- Graphite:
  - https://graphiteapp.org/
  - https://graphite.readthedocs.io/en/latest/
- Grafana:
  - https://grafana.com/
  - https://docs.grafana.org/guides/getting_started/
- Imagen usada (statsd + graphite):
  - https://hub.docker.com/r/graphiteapp/graphite-statsd/
  - https://github.com/graphite-project/docker-graphite-statsd
- Gotchas:
  - http://dieter.plaetinck.be/post/25-graphite-grafana-statsd-gotchas/
- Artillery:
  - https://artillery.io/docs/
  - https://www.npmjs.com/package/artillery
  - https://www.npmjs.com/package/artillery-plugin-statsd
- JMeter:
  - https://jmeter.apache.org/
- Artículos sobre generación de carga:
  - https://queue-it.com/blog/load-vs-stress-testing/
  - https://www.artillery.io/blog/load-testing-workload-models

## Pequeño cheatsheet de docker

Es posible que necesiten ejecutar los comandos con `sudo`, según el sistema que usen y cómo lo hayan instalado.

```sh
# Ver qué containers existen
docker ps [-a]

# Ver qué imagenes hay en mi máquina
docker images

# Ver uso de recursos de containers (como "top" en linux)
# Ejemplo con formato específico: docker stats --format '{{.Name}}\t{{.ID}}\t{{.CPUPerc}}\t{{.MemUsage}}'
docker stats [--format <format_string>]

# Descargar una imagen
docker pull <image>[:<tag>]

# Eliminar un container
docker rm <container_id> [-f]

# Eliminar una imagen
docker rmi <image_id> [-f]

# Eliminar imágenes "colgadas" (dangling)
docker rmi $(docker images -q -f dangling=true)

# Versión instalada
docker version
```

## Pequeño cheatsheet de docker-compose

Todos los siguientes comandos deben ejecutarse desde el directorio en donde está el archivo `docker-compose.yml` del proyecto.

Es posible que necesiten ejecutar los comandos con `sudo`, según el sistema que usen y cómo lo hayan instalado.

```sh
# ALIAS para escribir menos
alias docc='docker-compose'

# Ayuda general
docc --help

# Ayuda genral para cualquier comando
docc [COMMAND] --help

# Levantar servicios.
# Sugerencia: Usar la opción -d para levantar en background, y poder seguir usando la terminal
# También sirve para escalar horizontalmente un servicio que ya se esté ejecutando [buscar opción --scale].
# Si no se especifica al menos un servicio, se levantan todos
docc up [options] [SERVICE...]

# Ver logs de un servicio ejecutándose en background
docc logs [options] [SERVICE]

# Listar containers y sus estados
docc ps

# Restartear servicios
# Si no se indica al menos un servicio, se restartean todos
docc restart [SERVICE...]

# Frenar servicios corriendo en background (con la opción --detach del `up`)
# Si no se lista ningún servicio, se frenan todos.
# Esto solo frena servicio, no borra el container ni los datos que hayan en el mismo
docc stop [SERVICE...]

# Frenar containers y borrar tanto los containers como las imágenes y los volúmenes de almacenamiento
# (se pierden todos los datos que hubiera en el container).
# Esto aplica a TODOS los levantados con `up`, no filtra por servicio
docc down

# Levantar un nuevo container de un servicio y ejecutar un comando adentro
# (util para tener por ejemplo una terminal dentro de un container e inspeccionarlo o hacer pruebas manuales).
# Como es siempre sobre un container nuevo, lo que ven es el resultado de su docker-compose.yml y sus dockerfiles
# Ejemplo: docc run graphite bash
docc run SERVICE COMMAND

# Correr un comando en un container que ya existe y ya está corriendo.
# Parecido a `run` pero sobre un container en ejecución.
# Útil para alterar o inspeccionar algo que se está ejecutando.
# Lo que ven adentro puede no ser el resultado directo del docker-compose.yml + dockerfiles, así que mucho cuidado
# si van a modificar sus containers así, porque puede ser difícil de reproducir luego.
# Ejemplo: docc exec graphite bash
docc exec SERVICE COMMAND

# Versión instalada
docc version
```
