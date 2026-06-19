2do Parcial - Parte Practica
Que se solicita:

El codigo tiene 10 errores. Recae en usted analizar que es un error dentro del codigo.
Los Alumnos tendran que forkear este repo como propio, hacer un issue desde Github con Comentarios refiriendo en que linea esta el error, y como se debe solucionar.
La respuesta sera con el link a ese Fork, y adentro deben estar los issues. Los profesores tenemos que poder ingresar al mismo. Recae en los alumnos asegurarse de que los profesores puedan ingresar.
Tambien pueden editar el Archivo Readme y poner los resultados dentro de sus propios forks.
https://github.com/ExBattou/SimpsonsApp


RESOLUCION (Alumno: Eliel Lanzillotta - LU 1180681)
GitHub Fork: https://github.com/Eliel-L/SimpsonsApp

================================================================

ERROR N°1

Archivo: domain/repository/EpisodeRepository.kt
Linea: 8

Problema detectado:
El metodo se declara como get_episodes() (snake_case), pero la implementacion
en EpisodeRepositoryImpl lo define como getEpisodes() (camelCase). Los nombres
no coinciden, por lo que la clase no implementa correctamente el contrato de la
interfaz y el codigo no compila.

Por que esta mal:
En Kotlin la convencion obligatoria para funciones es camelCase. EpisodeRepositoryImpl
hace override de getEpisodes() que no existe en la interfaz, y la interfaz deja
get_episodes() sin implementacion concreta.

Como deberia corregirse:
Renombrar get_episodes() a getEpisodes() en la interfaz para que coincida
con la implementacion.

================================================================

ERROR N°2

Archivo: di/DataModule.kt
Lineas: 34-37

Problema detectado:
El builder de Retrofit no incluye llamada a .baseUrl(). Se construye directamente
con .client(), .addConverterFactory() y .build() sin especificar la URL base.

Por que esta mal:
Retrofit lanza IllegalStateException("Base URL required.") en tiempo de ejecucion
si no se establece la base URL antes de llamar a .build(). La app crashea al
iniciar antes de realizar cualquier llamada de red.

Como deberia corregirse:
Agregar .baseUrl("https://thesimpsonsapi.com/") en el Retrofit.Builder(), y
cambiar la URL del @GET en SimpsonsApi a solo el path relativo "/api/episodes".

================================================================

ERROR N°3

Archivo: data/remote/EpisodeRemoteMediator.kt
Lineas: 105-112

Problema detectado:
La interfaz SimpsonsApi esta declarada al final de EpisodeRemoteMediator.kt en
lugar de tener su propio archivo dedicado.

Por que esta mal:
Viola la separacion de responsabilidades. Un archivo de RemoteMediator no debe
contener el contrato de red. Cada interfaz/contrato debe residir en su propio
archivo.

Como deberia corregirse:
Extraer SimpsonsApi a un archivo propio SimpsonsApi.kt dentro del paquete
data/remote.

================================================================

ERROR N°4

Archivo: data/repository/EpisodeRepositoryImpl.kt
Linea: 18 (uso) / bloque de imports (ausencia)

Problema detectado:
Se usa SimpsonsApi como dependencia en el constructor (linea 18), pero no hay
ningun import de com.example.simpsonsapp.data.remote.SimpsonsApi en el archivo.

Por que esta mal:
En Kotlin, los simbolos de paquetes distintos deben importarse explicitamente.
La ausencia del import genera error de compilacion.

Como deberia corregirse:
Agregar "import com.example.simpsonsapp.data.remote.SimpsonsApi" en el bloque
de imports del archivo.

================================================================

ERROR N°5

Archivo: main/MainScreen.kt
Lineas: 51-53

Problema detectado:
viewModel.refreshSeasons() se ejecuta directamente en el cuerpo del Composable
sin estar dentro de un LaunchedEffect. Es un side effect desprotegido.

Por que esta mal:
El cuerpo de un Composable puede recomponerse multiples veces. Ejecutar
operaciones con efectos secundarios (llamadas al ViewModel) directamente en la
recomposicion viola las reglas de Compose y causa invocaciones repetidas e
incontroladas.

Como deberia corregirse:
Envolver el bloque en LaunchedEffect(episodes.loadState.refresh, seasons) para
que el efecto solo se ejecute cuando cambie el estado, no en cada recomposicion.

================================================================

ERROR N°6

Archivo: main/MainScreenViewModel.kt
Linea: 12

Problema detectado:
MainScreenViewModel existe como segundo ViewModel para la misma pantalla pero
no tiene @HiltViewModel ni @Inject. No es utilizado por ningun Composable real;
la pantalla usa MainViewModel. Es codigo muerto no eliminado de la plantilla.

Por que esta mal:
Sin @HiltViewModel e @Inject, Hilt no puede instanciarlo via hiltViewModel().
Su existencia genera ambiguedad arquitectonica y la dependencia DataRepository
que usa retorna datos de placeholder ("Android"), sin relacion con la app real.

Como deberia corregirse:
Eliminar MainScreenViewModel del proyecto al ser codigo muerto sin funcionalidad
real en la aplicacion.

================================================================

ERROR N°7

Archivo: data/DataRepository.kt
Lineas: 6-12

Problema detectado:
La interfaz DataRepository y su implementacion DefaultDataRepository son codigo
de plantilla no eliminado. Exponen Flow<List<String>> emitiendo listOf("Android"),
sin relacion con los datos de la app. No estan registradas en DataModule ni son
usadas por ningun componente funcional.

Por que esta mal:
La materia exige que el Repository abstraiga el origen de datos real. Este
repositorio es un placeholder que poluciona la arquitectura con un contrato
falso, sin binding en el grafo de dependencias de Hilt.

Como deberia corregirse:
Eliminar DataRepository y DefaultDataRepository. La abstraccion real ya esta
implementada correctamente en EpisodeRepository y EpisodeRepositoryImpl.

================================================================

ERROR N°8

Archivo: data/local/entity/EpisodeEntity.kt (linea 7)
         data/local/entity/RemoteKeyEntity.kt (linea 7)

Problema detectado:
Las entidades Room se declaran como class en lugar de data class.

Por que esta mal:
Paging 3 usa DiffCallback para detectar cambios en la lista, lo que depende
de equals() y hashCode() basados en los valores de los campos. Al usar class,
Kotlin genera igualdad por referencia, no por valor. Esto puede causar que
Paging no detecte actualizaciones correctamente.

Como deberia corregirse:
Cambiar "class EpisodeEntity" por "data class EpisodeEntity" y lo mismo para
RemoteKeyEntity.

================================================================

ERROR N°9

Archivo: AppNavigation.kt
Lineas: 9-10

Problema detectado:
Se usan wildcard imports: "import androidx.navigation.*" y "import androidx.compose.*".

Por que esta mal:
Los wildcard imports son antipatron en Kotlin/Android. Introducen ambiguedad de
nombres cuando dos paquetes exportan el mismo simbolo, ocultan que se esta
usando realmente y pueden generar errores de compilacion por colision de nombres.

Como deberia corregirse:
Reemplazar los wildcard imports por imports explicitos de cada simbolo utilizado.

================================================================

ERROR N°10

Archivo: main/MainScreen.kt
Linea: 119

Problema detectado:
La lista principal de episodios se renderiza con LazyRow (scroll horizontal)
en lugar de LazyColumn (scroll vertical).

Por que esta mal:
LazyRow es para colecciones pequenas con orientacion horizontal (carruseles).
Una lista de episodios con tarjetas que contienen imagen, titulo, sinopsis y
metadatos debe usar LazyColumn con scroll vertical, que es el patron estandar
en Android. Las EpisodeCard tienen ancho fijo de 250dp, lo que produce una
experiencia incorrecta en orientacion portrait.

Como deberia corregirse:
Reemplazar LazyRow por LazyColumn y ajustar EpisodeCard para que use
fillMaxWidth() en lugar de ancho fijo.

================================================================

Read [](file:///c%3A/DEV/2doParcialLanzillotta/app/src/main/java/com/example/simpsonsapp/main/MainScreenViewModel.kt#1-1), lines 1 to 12

Read [](file:///c%3A/DEV/2doParcialLanzillotta/app/src/main/java/com/example/simpsonsapp/main/MainScreen.kt#115-115), lines 115 to 130

Read [](file:///c%3A/DEV/2doParcialLanzillotta/app/src/main/java/com/example/simpsonsapp/data/DataRepository.kt#1-1), lines 1 to 12

Read [](file:///c%3A/DEV/2doParcialLanzillotta/app/src/main/java/com/example/simpsonsapp/data/local/entity/EpisodeEntity.kt#1-1), lines 1 to 8

Read [](file:///c%3A/DEV/2doParcialLanzillotta/app/src/main/java/com/example/simpsonsapp/AppNavigation.kt#1-1), lines 1 to 12

Searched for text `interface SimpsonsApi` (`**/*.kt`), 1 result

Searched for text `class RemoteKeyEntity` (`**/*.kt`), 1 result

Aquí está la devolución técnica en texto plano con líneas específicas:

---

RESOLUCIÓN
Alumno: Eliel Lanzillotta - LU 1180681
GitHub: https://github.com/Eliel-L/SimpsonsApp.git 

================================================================

ERROR N°1

Archivo: domain/repository/EpisodeRepository.kt
Línea: 8

Problema detectado:
El método se declara como get_episodes() (snake_case), pero la implementación
en EpisodeRepositoryImpl lo define como getEpisodes() (camelCase). Los nombres
no coinciden, por lo que la clase no implementa correctamente el contrato de la
interfaz y el código no compila.

Por qué está mal:
En Kotlin la convención obligatoria para funciones es camelCase. Más grave aún,
EpisodeRepositoryImpl hace override de getEpisodes() que no existe en la
interfaz, y la interfaz deja get_episodes() sin implementación concreta.

Cómo debería corregirse:
Renombrar get_episodes() a getEpisodes() en la interfaz para que coincida
con la implementación.

================================================================

ERROR N°2

Archivo: di/DataModule.kt
Líneas: 34-37

Problema detectado:
El builder de Retrofit no incluye llamada a .baseUrl(). Se construye directamente
con .client(), .addConverterFactory() y .build() sin especificar la URL base.

Por qué está mal:
Retrofit lanza IllegalStateException("Base URL required.") en tiempo de ejecución
si no se establece la base URL antes de llamar a .build(). La app crashea al
iniciar antes de realizar cualquier llamada de red.

Cómo debería corregirse:
Agregar .baseUrl("https://thesimpsonsapi.com/") en el Retrofit.Builder(), y
cambiar la URL del @GET en SimpsonsApi a solo el path relativo "/api/episodes".

================================================================

ERROR N°3

Archivo: data/remote/EpisodeRemoteMediator.kt
Líneas: 105-112

Problema detectado:
La interfaz SimpsonsApi está declarada al final de EpisodeRemoteMediator.kt en
lugar de tener su propio archivo dedicado.

Por qué está mal:
Viola la separación de responsabilidades. Un archivo de RemoteMediator no debe
contener el contrato de red. Cada interfaz/contrato debe residir en su propio
archivo. Dificulta el testing, la navegación y el mantenimiento.

Cómo debería corregirse:
Extraer SimpsonsApi a un archivo propio SimpsonsApi.kt dentro del paquete
data/remote.

================================================================

ERROR N°4

Archivo: data/repository/EpisodeRepositoryImpl.kt
Línea: 18 (uso) / bloque de imports (ausencia)

Problema detectado:
Se usa SimpsonsApi como dependencia en el constructor (línea 18), pero no hay
ningún import de com.example.simpsonsapp.data.remote.SimpsonsApi en el archivo.

Por qué está mal:
En Kotlin, los símbolos de paquetes distintos deben importarse explícitamente.
La ausencia del import genera error de compilación.

Cómo debería corregirse:
Agregar "import com.example.simpsonsapp.data.remote.SimpsonsApi" en el bloque
de imports del archivo.

================================================================

ERROR N°5

Archivo: main/MainScreen.kt
Líneas: 51-53

Problema detectado:
viewModel.refreshSeasons() se ejecuta directamente en el cuerpo del Composable
sin estar dentro de un LaunchedEffect. Es un side effect desprotegido.

Por qué está mal:
El cuerpo de un Composable puede recomponerse múltiples veces. Ejecutar
operaciones con efectos secundarios (llamadas al ViewModel) directamente en la
recomposición viola las reglas de Compose y causa invocaciones repetidas e
incontroladas.

Cómo debería corregirse:
Envolver el bloque en LaunchedEffect(episodes.loadState.refresh, seasons) para
que el efecto solo se ejecute cuando cambie el estado de carga o las temporadas,
no en cada recomposición.

================================================================

ERROR N°6

Archivo: main/MainScreenViewModel.kt
Línea: 12

Problema detectado:
MainScreenViewModel existe como segundo ViewModel para la misma pantalla pero
no tiene @HiltViewModel ni @Inject. No es utilizado por ningún Composable real;
la pantalla usa MainViewModel. Es código muerto que no fue eliminado de la
plantilla de Android Studio.

Por qué está mal:
Sin @HiltViewModel e @Inject, Hilt no puede instanciarlo ni inyectarlo mediante
hiltViewModel(). Su existencia genera ambigüedad arquitectónica y la dependencia
DataRepository que usa retorna datos de placeholder ("Android"), sin relación
con la app real.

Cómo debería corregirse:
Eliminar MainScreenViewModel del proyecto al ser código muerto sin funcionalidad
real en la aplicación.

================================================================

ERROR N°7

Archivo: data/DataRepository.kt
Líneas: 6-12

Problema detectado:
La interfaz DataRepository y su implementación DefaultDataRepository son código
de plantilla no eliminado. Exponen Flow<List<String>> emitiendo listOf("Android"),
sin relación con los datos de la app. No están registradas en DataModule ni son
usadas por ningún componente funcional.

Por qué está mal:
La materia exige que el Repository abstraiga el origen de datos real. Este
repositorio es un placeholder que poluciona la arquitectura con un contrato
falso, sin binding en el grafo de dependencias de Hilt.

Cómo debería corregirse:
Eliminar DataRepository y DefaultDataRepository del proyecto. La abstracción
real ya está implementada correctamente en EpisodeRepository y
EpisodeRepositoryImpl.

================================================================

ERROR N°8

Archivo: data/local/entity/EpisodeEntity.kt (línea 7)
         data/local/entity/RemoteKeyEntity.kt (línea 7)

Problema detectado:
Las entidades Room se declaran como class en lugar de data class.

Por qué está mal:
Paging 3 usa DiffCallback para detectar cambios en la lista, lo que depende
de equals() y hashCode() basados en los valores de los campos. Al usar class,
Kotlin genera igualdad por referencia, no por valor. Esto puede causar que
Paging no detecte actualizaciones correctamente.

Cómo debería corregirse:
Cambiar "class EpisodeEntity" por "data class EpisodeEntity" y lo mismo para
RemoteKeyEntity.

================================================================

ERROR N°9

Archivo: AppNavigation.kt
Líneas: 9-10

Problema detectado:
Se usan wildcard imports: "import androidx.navigation.*" y "import androidx.compose.*".

Por qué está mal:
Los wildcard imports son antipatrón en Kotlin/Android. Introducen ambigüedad de
nombres cuando dos paquetes exportan el mismo símbolo, ocultan qué se está
usando realmente y pueden generar errores de compilación por colisión. El
paquete androidx.compose.* a nivel raíz no expone clases directamente útiles.

Cómo debería corregirse:
Reemplazar los wildcard imports por imports explícitos de cada símbolo utilizado.

================================================================

ERROR N°10

Archivo: main/MainScreen.kt
Línea: 119

Problema detectado:
La lista principal de episodios se renderiza con LazyRow (scroll horizontal)
en lugar de LazyColumn (scroll vertical).

Por qué está mal:
LazyRow es para colecciones pequeñas con orientación horizontal (carruseles).
Una lista de episodios con tarjetas que contienen imagen, título, sinopsis y
metadatos debe usar LazyColumn con scroll vertical, que es el patrón estándar
en Android para listas de contenido. Las EpisodeCard tienen ancho fijo de 250dp,
lo cual produce una experiencia de usuario incorrecta en orientación portrait.

Cómo debería corregirse:
Reemplazar LazyRow por LazyColumn y ajustar EpisodeCard para que use
fillMaxWidth() en lugar de ancho fijo.

================================================================