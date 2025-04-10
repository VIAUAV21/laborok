# Labor08 - Függőséginjektálás a Dagger és a Hilt segítségével, tesztelés (Todo)

## Bevezetés

A korábbi laborokon már elsajátítottuk, hogyan lehet az Android alkalmazásunkat lazán csatolt réteges architektúrával megvalósítani. Ez egyértelműen segíti az alkalmazás rugalmas fejlesztését, esetleg az egyes rétegek is lecserélhetők, de ha megnézzük a kódot, még mindig viszonylag jelentős függést találunk, hiszen ahol egy másik rétegbeli komponenst hozunk létre, ott a példányosítás a kódba van "égetve", a másik réteg lecseréléséhez itt is módosítanunk kellene a kódot. Ezekre a problémákra nyújt megoldást a függőséginjektálás (dependency injection). Ez egy általános szoftverfejlesztési technika, amelyet nemcsak Androidon, hanem más platformokon is használunk. Ebben a laborban az Androidon használható Dagger és Hilt könyvtárakat ismerjük meg, amellyel Androidon tudunk függőséginjektálást végezni. A két könyvtárat gyakran együtt használjuk, a Dagger alapvetőbb, alacsonyabb szintű funkciókat nyújt, a Hilt pedig erre épül rá, hogy könnyebbé tegye a fejlesztést.

A labornak egy másik fontos témája az Android alkalmazások tesztelése.

## Előkészületek

A feladatok megoldása során ne felejtsd el követni a [feladat beadás folyamatát](../../tudnivalok/github/GitHub.md).

### Git repository létrehozása és letöltése

1. Moodle-ben keresd meg a laborhoz tartozó meghívó URL-jét és annak segítségével hozd létre a saját repository-dat.

2. Várd meg, míg elkészül a repository, majd checkout-old ki.

    !!! tip ""
        Egyetemi laborokban, ha a checkout során nem kér a rendszer felhasználónevet és jelszót, és nem sikerül a checkout, akkor valószínűleg a gépen korábban megjegyzett felhasználónévvel próbálkozott a rendszer. Először töröld ki a mentett belépési adatokat (lásd [itt](../../tudnivalok/github/GitHub-credentials.md)), és próbáld újra.

3. Hozz létre egy új ágat `megoldas` néven, és ezen az ágon dolgozz.

4. A `neptun.txt` fájlba írd bele a Neptun kódodat. A fájlban semmi más ne szerepeljen, csak egyetlen sorban a Neptun kód 6 karaktere.

A Hilt megismeréséhez ebben a laborban egy előre elkészített projektbe fogjuk integrálni a különböző szolgáltatásokat, ez megtalálható a repository-n belül. Indítsuk el az Android Studio-t, majd nyissuk meg a projektet.

!!!danger "FILE PATH"
	A projekt a repository-ban lévő `Todo` könyvtárba kerüljön, és beadásnál legyen is felpusholva! A kód nélkül nem tudunk maximális pontot adni a laborra!

Ellenőrízzük, hogy a létrejött projekt lefordul és helyesen működik!

## A Dagger és a Hilt inicializálása

A Dagger/Hilt feladata tehát az lesz, hogy az alkalmazásunk egymástól függő komponenseit lazábban csatolt módon köti össze. A gyakorlatban ez azt jelenti, hogy ha egy bizonyos típusú függőségre van szükségünk, és az adott függőség meg van jelölve mint injektálandó függőség, akkor a könyvtárak el fogják végezni nekünk az adott függőség megkeresését és beállítását. Ez történhet például egy konstruktornak történő paraméterátadáson keresztül. Ennek előnye, hogy ha egy komponenst lecserélünk - például ahogyan a Firebase laboron lecseréltük a memóriabeli implementációkat éles, Firebase-ben működőkre - akkor nem szükséges a kódunkat módosítani, csupán meg kell adni a Daggernek/Hiltnek, hogy az elérhető implementációk közül melyik legyen az, amelyiket szükség esetén alkalmazza.

Többféle függőséginjektáló keretrendszer létezik Androidon, és az Android platformon kívül is, ezek némileg más elveken működnek. A Dagger a legjobb teljesítmény érdekében úgy működik, hogy nem futás közben oldja fel a függőségeket, hanem a fordítási folyamatba avatkozik bele, és már aközben feltérképezi a függőségi viszonyok jelölésére alkalmazott annotációkat. Ezért a projekt inicializálásának részeként szükséges felvennünk egy gradle plugint is a folyamatba.

Először a `libs.versions.toml` fájlunkba vegyük fel az új függőségekhez kapcsolódó bejegyzéseket:

```
[versions]
hilt = "2.56.1"
hilt-navigation-compose = "1.2.0"

[libraries]
hilt-android = { module = "com.google.dagger:hilt-android", version.ref = "hilt" }
hilt-compiler = { module = "com.google.dagger:hilt-compiler", version.ref = "hilt" }
hilt-navigation-compose = { module = "androidx.hilt:hilt-navigation-compose", version.ref = "hilt-navigation-compose"}

[plugins]
google-dagger-hilt-android = { id = "com.google.dagger.hilt.android", version.ref = "hilt"}
```

Majd a projekt szintű `build.gradle.kts` fájlba vegyük fel a a következő sort a pluginek közé:

```kotlin
alias(libs.plugins.google.dagger.hilt.android) apply false
```

Majd a modul szintű `build.gradle` fájlban alkalmazzuk a plugint:

```kotlin
plugins {
    ...

    alias(libs.plugins.google.dagger.hilt.android)
}
```

És vegyük még fel a szükséges függőségeket, majd szinkronizáljuk a projektet:

```kotlin
// Hilt
implementation(libs.hilt.android)
implementation(libs.hilt.navigation.compose)
ksp(libs.hilt.compiler)
```

Ezzel a build folyamat és a függőségek rendben vannak. Most globálisan, az alkalmazás szintjén inicializálnunk kell a Daggert, hogy létrejöjjön egy kontextus, amelyben a függőségeket menedzseli. Ehhez a `TodoApplication` osztályra tegyük rá a `@HiltAndroidApp` annotációt:

```kotlin
@HiltAndroidApp
class TodoApplication : Application() {
	// ...
}
```

Majd nyissuk meg a `MainActivity` osztályt is, ezen pedig az `@AndroidEntryPoint` annotációt helyezzük el:

```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            TodoTheme {
                NavGraph()
            }
        }
    }
}
```

Ezzel a közös inicializációs feladatok elkészültek, de még tényleges injektálható komponenseket és injektálandó függőségeket nem hoztunk létre. Most elkezdjük a "bedrótozott" függőségi viszonyokat függőséginjektálásra cserélni.

## Az adatbázismodul elkészítése

A Dagger és a Hilt által kezelt komponensek a jobb átláthatóság érdekében modulokra oszthatók. Minden modul komponenseket hoz létre, amelyeket a megjelölt injektálási pontokon a könyvtárak fel fognak használni. Az első modulunk a `TodoDatabase` és a `TodoDao` létrehozását fogja elvégezni. Hozzunk létre ennek a modulnak egy `data.di` package-et, és ebbe vegyük fel a modult megvalósító osztályunkat:

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {

    @Provides
    @Singleton
    fun provideDatabaseInstance(
        @ApplicationContext context: Context
    ): TodoDatabase = Room.databaseBuilder(
        context,
        TodoDatabase::class.java,
        "todo_database"
    ).fallbackToDestructiveMigration().build()

    @Provides
    @Singleton
    fun provideTodoDao(
        db: TodoDatabase
    ): TodoDao = db.dao
}
```

Ebben a `@Module` annotáció azt jelenti, hogy az osztály komponensekkel járul hozzá a Dagger/Hilt által kezelt objektumgráfhoz. Az osztályban legyártott komponensek tehát függőségként injektálhatóak lesznek más komponensekbe, illetve maguk is hivatkozhatnak függőségekre. Az `@InstallIn(SingletonComponent::class)` a SingletonComponenthez köti a modult, aminek az lesz az eredménye, hogy a komponensek injektálása az egész alkalmazáson belül működik majd.

A metódusokon levő `@Provides` határozza meg, hogy a metódusok "factory" metódusként szolgáljanak, és az általuk visszaadott objektumok a Dagger által kezelt komponensekké váljanak. A `@Singleton` annotáció azt adja meg, hogy ezekből a komponensekből egyetlen példány készüljön, és az egész alkalmazáson keresztül mindenhol ezt használjuk függőségként. Legtöbbször elegendő egy példány, és ezért ez a célravezető megoldás.

Most, hogy a komponenseket legyártottuk, arról kell gondoskodni, eltávolítsuk ezeknek a komponenseknek a korábbi módon történő létrehozását, és megjelöljük azokat a helyeket, ahova a Dagger és Hilt könyvtáraknak ezeket a komponenseket függőségként injektálniuk kell. Korábban a `TodoDatabase` létrehozása a `TodoApplication` osztályban volt. Innen eltávolítjuk a példányosítást, viszont a repository komponensünket egyelőre nem bíztuk a Dagger/Hilt párosra, ezért ezt még mindig létre kell hozni, ehhez viszont szükség van a `TodoDao`-ra, amit viszont már ide is injektálhatunk. 

A `TodoApplication` osztályunk most így fest:

```kotlin
@HiltAndroidApp
class TodoApplication : Application() {

    @Inject
    lateinit var dao: TodoDao

    companion object {
        lateinit var repository: TodoRepositoryImpl
    }

    override fun onCreate() {
        super.onCreate()

        repository = TodoRepositoryImpl(dao)
    }
}
```

!!!example "BEADANDÓ (1 pont)"
	Készíts egy **képernyőképet**, amelyen látszik a **futó alkalmazás**, a **fenti lépésekhez tartozó kódrészlet**, valamint a **neptun kódod a kódban valahol kommentként**.

	A képet a megoldásban a repository-ba f1.png néven töltsd föl.

	A képernyőkép szükséges feltétele a pontszám megszerzésének.

## A repository modul elkészítése

A fentihez hasonlóan most a repository-t is a Dagger/Hilt kezelésére bízzuk, és megszüntetjük
a kézi létrehozást. Ehhez először szintén egy modult kell létrehoznunk. Tegyük ezt szintén
a `data.di` package-be:

```kotlin
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {

    @Binds
    @Singleton
    abstract fun bindTodoRepository(
        todoRepositoryImpl: TodoRepositoryImpl
    ): TodoRepository

}
```

Ez a modul némileg másképp működik, mint a korábbi. Egyrészt az osztály absztrakt, illetve nem a `@Provides` annotációt használtuk, hanem a `@Binds`-et, és az ezzel megjelölt metódus azt határozza meg, hogy amikor `TodoRepository` típusú függőségre van szükségünk, akkor annak a konkrét `TodoRepositoryImpl` típusú implementációját kell használni. Most már kivehetjük a `TodoApplication` osztályból a repository kezelését is, így az osztályunk üres lesz, de a Hilt inicializációja miatt továbbra is szükség van rá, hogy megtartsuk:

```kotlin
@HiltAndroidApp
class TodoApplication : Application()
```

Viszont a `TodoRepositoryImpl` példányosításához szükség van a `TodoDao` osztályre, és mivel innentől ezt a Dagger/Hilt végzi, ezért az `@Inject` annotáció használatával jelezni kell, hogy példányosításkor a `TodoDao`-t függőségként szeretnénk injektálni. Írjuk át az osztály fejlécét az alábbira:

```kotlin
class TodoRepositoryImpl @Inject constructor(
    private val dao: TodoDao
) : TodoRepository {
	// ...
}
```

A repository viszont szorosan volt csatolva a viewmodelekhez, amelyeket a három feature-höz írtunk. Még ezeket is át kell alakítanunk hozzá, hogy az alkalmazás újra használható legyen. Jelenleg mindhárom viewmodel osztályunkban az alábbihoz hasonló factory-k vannak:

```kotlin
    companion object {
        val Factory: ViewModelProvider.Factory = viewModelFactory {
            initializer {
                val todoOperations = TodoUseCases(TodoApplication.repository)
                TodoListViewModel(
                    todoOperations = todoOperations
                )
            }
        }
    }
```

Ezeket törölnünk kell, viszont gondoskodnunk kell róla, hogy a `TodoUseCases` is inicializálódjon. Jelenlegi készültségben ezt nem tudjuk még injektálni, csak a `TodoRepository` komponenst, ennek segítségével viszont példányosítható a `TodoUseCases`. Illetve még el kell helyezni a viewmodel osztályokon a `@HiltViewModel` annotációt, hogy a Hilt által menedzselt viewmodellekké váljanak. A `TodoListViewModel` végül így alakul:

```kotlin
@HiltViewModel
class TodoListViewModel @Inject constructor(
    private val repository: TodoRepository
) : ViewModel() {

    val todoOperations: TodoUseCases

    private val _state = MutableStateFlow<TodoListState>(TodoListState.Loading)
    val state = _state.asStateFlow()

    init {
        todoOperations = TodoUseCases(repository)
        loadTodos()
    }

    fun loadTodos() {
        viewModelScope.launch {
            try {
                _state.value = TodoListState.Loading
                val todos = todoOperations.loadTodos().getOrThrow().map { it.asTodoUi() }
                _state.value = TodoListState.Result(
                    todoList = todos
                )
            } catch (e: Exception) {
                _state.value = TodoListState.Error(e)
            }
        }
    }
}
```

Hasonló átalakításokat kell végeznünk a `TodoDetailViewModel` osztályon:

```kotlin
@HiltViewModel
class TodoDetailViewModel @Inject constructor(
    private val savedStateHandle: SavedStateHandle,
    private val repository: TodoRepository
) : ViewModel() {

    val todoOperations: TodoUseCases

    private val _state = MutableStateFlow<TodoDetailState>(TodoDetailState.Loading)
    val state = _state.asStateFlow()

    init {
        todoOperations = TodoUseCases(repository)
        loadTodos()
    }

    private fun loadTodos() {
        val id = checkNotNull<Int>(savedStateHandle["id"])
        viewModelScope.launch {
            try {
                _state.value = TodoDetailState.Loading
                val todo = todoOperations.loadTodo(id)
                _state.value = TodoDetailState.Result(
                    todo = todo.getOrThrow().asTodoUi()
                )
            } catch (e: Exception) {
                _state.value = TodoDetailState.Error(e)
            }
        }
    }
}
```

Majd a `TodoCreateViewModel` osztályon is:

```kotlin
@HiltViewModel
@HiltViewModel
class TodoCreateViewModel @Inject constructor(
    private val repository: TodoRepository
) : ViewModel() {

    val todoOperations: TodoUseCases

    private val _state = MutableStateFlow(TodoCreateState())
    val state = _state.asStateFlow()

    private val _uiEvent = Channel<TodoCreateUiEvent>()
    val uiEvent = _uiEvent.receiveAsFlow()

    init {
        todoOperations = TodoUseCases(repository)
    }

    fun onEvent(event: TodoCreateEvent) {
        when (event) {
            is TodoCreateEvent.ChangeTitle -> {
                val newValue = event.text
                _state.update {
                    it.copy(
                        todo = it.todo.copy(title = newValue)
                    )
                }
            }

            is TodoCreateEvent.ChangeDescription -> {
                val newValue = event.text
                _state.update {
                    it.copy(
                        todo = it.todo.copy(description = newValue)
                    )
                }
            }

            is TodoCreateEvent.SelectPriority -> {
                val newValue = event.priority
                _state.update {
                    it.copy(
                        todo = it.todo.copy(priority = newValue)
                    )
                }
            }

            is TodoCreateEvent.SelectDate -> {
                val newValue = event.date
                _state.update {
                    it.copy(
                        todo = it.todo.copy(dueDate = newValue.toString())
                    )
                }
            }

            TodoCreateEvent.SaveTodo -> {
                _state.update {
                    it.copy(
                        todo = it.todo.copy(id = (Math.random() * Int.MAX_VALUE).toInt())
                    )
                }
                onSave()
            }
        }
    }

    private fun onSave() {
        viewModelScope.launch {
            try {
                todoOperations.saveTodo(state.value.todo.asTodo())
                _uiEvent.send(TodoCreateUiEvent.Success)
            } catch (e: Exception) {
                _uiEvent.send(TodoCreateUiEvent.Failure(e.toUiText()))
            }
        }
    }
}
```

Majd végül az összes screen osztályban változtassuk meg a viewmodel létreozásának módját úgy, hogy a `hiltViewModel()` hívást használjuk. Pl. a `TodoDetailScreen` eddig így nézett ki:

```kotlin
fun TodoDetailScreen(
    onNavigateBack: () -> Unit,
    viewModel: TodoDetailViewModel = viewModel(factory = TodoDetailViewModel.Factory)
) {
	// ...
}
```

A következőképpen módosítsuk:

```kotlin
fun TodoDetailScreen(
    onNavigateBack: () -> Unit,
    viewModel: TodoDetailViewModel = hiltViewModel()
) {
	// ...
}
```

## A usecases modul elkészítése

Már csak a usecases modult kell elkészítenünk. Ehhez készítsünk először egy `domain.di` package-et, és ebbe hozzuk létre az alábbit:

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object TodoUseCaseModule {

    @Provides
    @Singleton
    fun provideLoadTodosUseCase(
        repository: TodoRepository
    ): LoadTodosUseCase = LoadTodosUseCase(repository)

    @Provides
    @Singleton
    fun provideTodoUseCases(
        repository: TodoRepository,
        loadTodos: LoadTodosUseCase
    ): TodoUseCases = TodoUseCases(repository, loadTodos)

}
```

A `TodoUseCases` osztályt pedig így alakítsuk át:

```kotlin
class TodoUseCases(
    val repository: TodoRepository,
    val loadTodos: LoadTodosUseCase
) {
    
    val loadTodo = LoadTodoUseCase(repository)
    val saveTodo = SaveTodoUseCase(repository)
    val updateTodo = UpdateTodoUseCase(repository)
    val deleteTodo = DeleteTodoUseCase(repository)
}
```

Most már a viewmodel osztályokba nem szükséges a `TodoRepository` injektálása és a `TodoUseCases` kézi létrehozása, hiszen a `TodoUseCases` közvetlen is injektálhatóvá vált. Módosítsuk ennek megfelelően a viewmodel osztályokat!

A `TodoDetailViewModel` kódja:

```kotlin
@HiltViewModel
class TodoDetailViewModel @Inject constructor(
    private val todoOperations: TodoUseCases,
    private val savedStateHandle: SavedStateHandle
) : ViewModel() {

    private val _state = MutableStateFlow<TodoDetailState>(TodoDetailState.Loading)
    val state = _state.asStateFlow()

    init {
        loadTodos()
    }

    private fun loadTodos() {
        val id = checkNotNull<Int>(savedStateHandle["id"])
        viewModelScope.launch {
            try {
                _state.value = TodoDetailState.Loading
                val todo = todoOperations.loadTodo(id)
                _state.value = TodoDetailState.Result(
                    todo = todo.getOrThrow().asTodoUi()
                )
            } catch (e: Exception) {
                _state.value = TodoDetailState.Error(e)
            }
        }
    }
}
```

A `TodoCreateViewModel` így fest:

```kotlin
@HiltViewModel
class TodoCreateViewModel @Inject constructor(
    private val todoOperations: TodoUseCases
) : ViewModel() {

    private val _state = MutableStateFlow(TodoCreateState())
    val state = _state.asStateFlow()

    private val _uiEvent = Channel<TodoCreateUiEvent>()
    val uiEvent = _uiEvent.receiveAsFlow()

    fun onEvent(event: TodoCreateEvent) {
        when (event) {
            is TodoCreateEvent.ChangeTitle -> {
                val newValue = event.text
                _state.update {
                    it.copy(
                        todo = it.todo.copy(title = newValue)
                    )
                }
            }

            is TodoCreateEvent.ChangeDescription -> {
                val newValue = event.text
                _state.update {
                    it.copy(
                        todo = it.todo.copy(description = newValue)
                    )
                }
            }

            is TodoCreateEvent.SelectPriority -> {
                val newValue = event.priority
                _state.update {
                    it.copy(
                        todo = it.todo.copy(priority = newValue)
                    )
                }
            }

            is TodoCreateEvent.SelectDate -> {
                val newValue = event.date
                _state.update {
                    it.copy(
                        todo = it.todo.copy(dueDate = newValue.toString())
                    )
                }
            }

            TodoCreateEvent.SaveTodo -> {
                _state.update {
                    it.copy(
                        todo = it.todo.copy(id = (Math.random() * Int.MAX_VALUE).toInt())
                    )
                }
                onSave()
            }
        }
    }

    private fun onSave() {
        viewModelScope.launch {
            try {
                todoOperations.saveTodo(state.value.todo.asTodo())
                _uiEvent.send(TodoCreateUiEvent.Success)
            } catch (e: Exception) {
                _uiEvent.send(TodoCreateUiEvent.Failure(e.toUiText()))
            }
        }
    }
}
```

A `TodoListViewModel` átalakított verziója pedig az alábbi:

```kotlin
@HiltViewModel
class TodoListViewModel @Inject constructor(
    private val todoOperations: TodoUseCases
) : ViewModel() {

    private val _state = MutableStateFlow<TodoListState>(TodoListState.Loading)
    val state = _state.asStateFlow()

    fun loadTodos() {
        viewModelScope.launch {
            try {
                _state.value = TodoListState.Loading
                val todos = todoOperations.loadTodos().getOrThrow().map { it.asTodoUi() }
                _state.value = TodoListState.Result(
                    todoList = todos
                )
            } catch (e: Exception) {
                _state.value = TodoListState.Error(e)
            }
        }
    }
}
```

!!!example "BEADANDÓ (1 pont)"
	Készíts egy **képernyőképet**, amelyen látszik a **futó alkalmazás**, a **fenti lépésekhez tartozó kódrészlet**, valamint a **neptun kódod a kódban valahol kommentként**.

	A képet a megoldásban a repository-ba f2.png néven töltsd föl.

	A képernyőkép szükséges feltétele a pontszám megszerzésének.

## Az alkalmazás tesztelése

Az elkészült alkalmazásnak most egy-egy részét automatizált tesztekkel fogjuk ellenőrizni. Az automatizált teszteket két fő típusra oszthatjuk Androidon:

* Lokális tesztek: ezek olyan tesztek amelyek bármiféle virtualizált környezet nélkül, pusztán Java kódnak a fejlesztői gépen történő futtatásával végrehajthatók.
* Instrumentált tesztek: ezek a tesztek az emulátoron futnak.

### Lokális tesztek futtatása

A lokális tesztek közt is megkülönböztethetünk különböző típusokat aszerint, hogy architekturálisan mekkora részét érintik a kódnak. A kód kis egységeit, gyakorlatban metódusait ellenőrző teszteket unitteszteknek nevezzük. Ha a teszt néhány osztály kódján is áthív, akkor integrációs tesztről beszélünk, ha pedig valamilyen komplexebb, sok komponenst érintő folyamatot tesztel, akkor rendszertesztnek nevezzük.

A lokális tesztek közül leggyakrabban unitteszteket készítünk, a nagyobb egységeket pedig gyakrabban már instrumentált tesztekkel ellenőrizzük. Most a lokális tesztek közül a unittesztekre koncentrálunk. Ezek a legkönnyebben elkészíthetőek és számos előnyük van:

* Szisztematikus módszert adnak a rendszer teljes leteszteléséhez
* Gyorsan lefutnak
* Mivel minden teszt egy kis egységre vonatkozik, a meghiúsuló teszt jól rámutat a probléma helyére is

A unittesztek esetén fontos kihívás, hogy a függőségeket izoláljuk, leválasszuk, hiszen ha azok is meghívódnának, akkor nem unittesztről beszélnénk, hanem integrációs tesztről, és a fenti előnyök nem teljesülnének. Különösen az az előny veszne el, hogy a teszt jól mutatja a hiba helyét. Ezért a tesztekben a függőségeket valamilyen *test double* objektummal, tipikusan *mock* objektummal cseréljük le. A mock objektum egy "buta", de "felprogramozható" komponens, ami éppen csak annyit csinál, amit a teszt idejére elvárunk tőle, azaz tipikusan valamilyen beégetett adatot ad vissza. Ezen kívül a segítségével ellenőrizhető az is, hogy a lecserélt függőségen a tesztelt kódrészlet tényleg elvégezte a várt hívást.

Az alkalmazásban nincsenek túl bonyolult üzleti logika részek, de a tesztelés technikáját jól meg tudjuk figyelni. Most a `TodoRepositoryImpl` osztályt fogjuk tesztelni. Konvenció szerint osztályokhoz készítünk tesztosztályokat, és a tesztosztályokban minden tesztelt metódus egy lehetséges lefutásához készítünk egy tesztmetódust. A tesztosztályokat a tesztelt osztályokkal azonos package-be tesszük, és nevükben a `Test` utótagot használjuk.

Először fel kell vennünk a teszteléshez használandó függőségeket a projektbe! A tesztelés alapvető fontosságú feladat, ezért az Android Studio új projektnél a legfontosabb függőségeket beleteszi a projektvázba, így a kiindulú projektben is már benne vannak a legfontosabb függőségek. Fogunk viszont mockokat is használni, és ehhez még fel kell vennünk a népszerű Mockito könyvtárat. A lokális tesztekhez a `testImplementation` scope-ot kell használnunk. Vegyük fel az alábbi függőségeket, először a `libs.versions.toml` fájllal kezdve:

```
[versions]
mockitoCore = "5.11.0"
mockitoInline = "5.2.0"
mockitoKotlin = "5.2.1"

[libraries]
mockito-inline = { module = "org.mockito:mockito-inline", version.ref = "mockitoInline" }
mockito-core = { module = "org.mockito:mockito-core", version.ref = "mockitoCore" }
mockito-kotlin = { module = "org.mockito.kotlin:mockito-kotlin", version.ref = "mockitoKotlin" }
```

Majd folytassuk a modulszintű `build.gradle.kts` fájllal:

```kotlin
// Testing - mocking
testImplementation(libs.mockito.core)
testImplementation(libs.mockito.inline)
testImplementation(libs.mockito.kotlin)
```

Hozzuk létre a szükséges könyvtárstruktúrát a teszteknek. A lokális tesztek a `src/test` könyvtárban vannak, az `src/androidTest` könyvtár pedig az instrumentált tesztek helye. Ezeken belül ugyanúgy `java` vagy `kotlin` könyvtárba kerülnek a forráskódok.

Most hozzuk létre a `data.repository` package-et a `test` könyvtárban is, mivel azt a gyakorlatot fogjuk követni, hogy egy osztályhoz tartozó tesztek azonos package-ben, egy azonosan kezdődő, de `Test` utótaggal ellátott tesztosztályban lesznek megvalósítva. A létrehozott package-ben hozzunk létre ezért egy `TodoRepositoryImplTest` osztályt az alábbi módon:

```kotlin
package hu.bme.aut.android.todo.data.repository

import hu.bme.aut.android.todo.data.dao.TodoDao
import hu.bme.aut.android.todo.data.entities.TodoEntity
import hu.bme.aut.android.todo.domain.model.Priority
import kotlinx.coroutines.flow.first
import kotlinx.coroutines.flow.flowOf
import kotlinx.coroutines.runBlocking
import kotlinx.datetime.LocalDate
import org.junit.Assert
import org.junit.Test
import org.junit.runner.RunWith
import org.mockito.Mock
import org.mockito.junit.MockitoJUnitRunner
import org.mockito.kotlin.doReturn
import org.mockito.kotlin.mock
import org.mockito.kotlin.times
import org.mockito.kotlin.verify

@RunWith(MockitoJUnitRunner::class)
class TodoRepositoryImplTest {

    @Mock
    lateinit var todoDao: TodoDao

    @Test
    fun testGetAllTodos() {
        val sampleTodo = TodoEntity(
            1,
            "Test app",
            Priority.HIGH,
            LocalDate(2024, 4, 30),
            "Write unit tests for our Todo app"
        )
        val mockDao = mock<TodoDao> {
            on { getAllTodos() } doReturn (flowOf(listOf(sampleTodo)))
        }
        val todoRepositoryImpl = TodoRepositoryImpl(mockDao)
        val result = todoRepositoryImpl.getAllTodos()
        runBlocking {
            Assert.assertTrue(result.first().contains(sampleTodo))
            verify(mockDao, times(1)).getAllTodos()
        }
    }
}
```

Nézzük át a laborvezetővel együtt, hogyan működik a teszt!

Alapvetően minden teszt három lépésből áll:

1. A *test fixture*, azaz a tesztelni kívánt kódrészlethez szükséges kezdeti állapot felállítása. Ha egy "sikeres" lefutást szeretnénk tesztelni, akkor ennek megfelelően készítjük elő a környezetet. Ha pedig egy hiba utána elvárt hatást, pl. exception dobódik, hibaüzenet íródik ki stb. szeretnénk tesztelni, akkor ennek megfelelően. Idetartozik a függőségek kiváltása is.
1. A tesztelni kívánt kódrészlet futtatása.
1. Az elvárt eredmények megfogalmazása, annak ellenőrzése, hogy teljesültek-e (*assertions*). Ha mock objektumokat használtunk, akkor idetartozik annak ellenőrzése is, hogy rajtuk meghívódtak-e azok a metódusok, amelyek meghívódására számítottunk.

A példánkban a `TodoRepositoryImpl` osztály `getAllTodos` metódusa csupán annyit tesz, hogy továbbhívja a `TodoDao` osztály `getAllTodos` metódusát, majd ennek eredményét visszaadja. A tesztünk ezért nem lesz túl bonyolult. Alapvetően abból áll, hogy készítenünk kell egy `TodoDao` mock objektumot, amely beégetett `TodoEntity` listát fog visszaadni. Ezt a mockot kell odaadnunk a `TodoRepositoryImp` függőségének, majd meg kell hívnunk a tesztelni kívánt metódust, és meg kell vizsgálni, hogy a beégetett listát adta-e vissza, valamint a mockunknak az azonos nevű metódusa szintén hívódott-e.

Futtassuk le a tesztet!

### Instrumentált tesztek futtatása

Bonyolultabb teszteket sem lehetetlen lokális tesztként futtatni, de a függőségek szövevényessége miatt ez egy jóval bonyolultabb feladat lenne. Praktikusabb ezért ha összetettebb folyamatok teszteléséhez az emulátort is segítségül hívjuk. Ilyen például a Compose segítségével készített UI tesztelése. Az Android által biztosított eszközökkel létre tudjuk hozni a komponenseinket egy emulált környezetben, és a teszt kódjából interakciókat is ki tudunk váltani (pl. írjunk be szöveget egy mezőbe, kattintsunk egy gombon stb.). Ez a fajta tesztelés láthatóan jóval közelebb áll ahhoz a módhoz, ahogyan az alkalmazás majd ténylegesen futni fog. Logikusan belétható ugyanakkor az is, hogy ezek a tesztek jóval bonyolultabbak, és lassabban is fognak futni.

Az instrumentált teszteket az `androidTest` könyvtárban lehet létrehozni. Mivel ezek nagyobb léptékű tesztek is lehetnek, nem feltétlen tartoznak logikailag egy komponenshez. Amennyiben azonban odatartoznak, javasolt ezeket is azonos package-be tenni és a lokális tesztekéhez hasonló elnevezési konvenció szerint elnevezni.

Először itt is a függőségek felvételével kezdünk, a `libs.versions.toml` fájllal. Az alapvető függőségek az instrumentált tesztekhez is bekerültek már a projektvázba, de mivel Hiltet használunk, a függőséginjektálásnak az instrumentált tesztekben is működniük kell:

```
[versions]
hiltAndroidTesting = "2.56.1"
hiltAndroidCompiler = "2.56.1"

[libraries]
hilt-android-testing = { module = "com.google.dagger:hilt-android-testing", version.ref = "hiltAndroidTesting" }
hilt-android-compiler = { module = "com.google.dagger:hilt-android-compiler", version.ref = "hiltAndroidCompiler" }
```

Majd hivatkozzuk is meg ezeket a modulszintű `build.gradle.kts` fájlban:

```kotlin
// Hilt for testing
androidTestImplementation(libs.hilt.android.testing)
kspAndroidTest(libs.hilt.android.compiler)
```

A példánkban azt fogjuk tesztelni, hogy ha új teendő létrehozásánál a dátumválasztó ikonjára kattintunk, akkor valóban előugrik a dátumválasztó komponens. Mielőtt a tényleges tesztet megírjuk, gondoskodnunk kell róla, hogy a tesztből majd a felhasználói felületen a dátumválasztó ikonját meg tudjuk hivatkozni. Ha lenne rajta megjelenített szöveg, a tesztből ez alapján is lehetne hivatkozni, de jelen esetben csak egy ikonról van szó. Úgy tudjuk azonosíthatóvá tenni, hogy a `Modifierén` keresztül egy test taggel látjuk el. Módosítsuk eszerint a `TodoEditor` osztályban a `DateSelector` komponens hívását:

```kotlin
DateSelector(
    pickedDate = pickedDate,
    onClick = onDateSelectorClicked,
    modifier = Modifier
        .fillMaxWidth()
        .testTag("datePickerIcon"),
    enabled = enabled
)
```

Most már elkészíthetjük a tesztet! Mivel a teszt a `TodoCreateScreen` osztályhoz köthető, ez pedig a `screen.todo_create` package-ben van, először hozzuk létre ezt a package-et az `androidTest` mappában is.

Majd készítsük el a `TodoCreateScreenTest` osztályunkat:

```kotlin
package hu.bme.aut.android.todo.screen.todo_create

import androidx.activity.compose.setContent
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.ui.ExperimentalComposeUiApi
import androidx.compose.ui.test.assertIsDisplayed
import androidx.compose.ui.test.assertIsNotDisplayed
import androidx.compose.ui.test.hasTestTag
import androidx.compose.ui.test.junit4.createAndroidComposeRule
import androidx.compose.ui.test.onNodeWithText
import androidx.compose.ui.test.performClick
import hu.bme.aut.android.todo.MainActivity
import org.junit.Rule
import org.junit.Test

class TodoCreateScreenTest {
    @get:Rule
    val composeTestRule = createAndroidComposeRule<MainActivity>()

    @OptIn(ExperimentalComposeUiApi::class, ExperimentalMaterial3Api::class)
    @Test
    fun testDatePickerDialogIsShownWhenClickedOnDatePickerIcon() {
        composeTestRule.activity.setContent {
            TodoCreateScreen(
                onNavigateBack = { })
        }

        composeTestRule.onNodeWithText("Select date").assertIsNotDisplayed()

        composeTestRule.onNode(hasTestTag("datePickerIcon")).performClick()

        composeTestRule.onNodeWithText("Select date").assertIsDisplayed()
    }
}
```

A teszt fő eleme a `createAndroidComposeRule` hívás, amely egy `teszt rule`-t ad vissza. Ezen keresztül renderelhetjük a kívánt Compose tartalmat, és ezen keresztül történik a kívánt akciók kiváltása és az elvárt eredmény ellenőrzése is. Ha valamilyen elemnek a megjelenését akarjuk tesztelni, érdemes azt is megfogalmazni, hogy a kiváltott interakció előtt még nincs megjelenítve.

Futtassuk le a tesztet! Figyeljük meg, hogy végigkövethető az emulátoron is a teszt futása.

!!!example "BEADANDÓ (1 pont)"
	Készíts egy **képernyőképet**, amelyen látszik a **lefutott teszt**, a **hozzá tartozó kódrészlet**, valamint a **neptun kódod a kódban valahol kommentként**.

	A képet a megoldásban a repository-ba f3.png néven töltsd föl.

	A képernyőkép szükséges feltétele a pontszám megszerzésének.

## Önálló feladat 1. - Dependency Injection befejezése

A `TodoUseCases` osztályban egyelőre csak a `LoadTodosUseCase` függőséget hozzuk létre a modulban, és injektáljuk a Dagger/Hilt segítségével, a többi usecase most is manuálisan példányosodik a repository átadásával. Folytasd az átalakítást, és hozd létre az összes usecase-t a usecase modulban, hogy utána már a Dagger/Hilt kezelje őket!

!!!example "BEADANDÓ (1 pont)" 
	Készíts egy **képernyőképet**, amelyen látszik a **futó alkalmazás**, az **átalakított kódrészlet**, valamint a **neptun kódod a kódban valahol kommentként**. 

	A képet a megoldásban a repository-ba f4.png néven töltsd föl.

	A képernyőkép szükséges feltétele a pontszám megszerzésének.


## Önálló feladat 2. - Újabb teszt készítése

Készítsünk egy tesztet arra vonatkozóan, hogy ha a prioritásválasztóra kattintunk, akkor az összes lehetséges választható prioritás megjelenik a képernyőn!

Segítség az implementációhoz:

* Most elég a `TodoEditor` komponensből kiindulni, nem szükséges a teljes `Screen` tesztelése.
* A `TodoEditor` meghívásakor ki kell töltenünk a kötelező paramétereit, de mivel ezekre a tesztben nem támaszkodunk, ezért üres sztringeket, üres függvényeket használhatunk helyettük.
* Lássuk el test taggel a legördülő listát.
* Fogalmazzuk meg, hogy a választott alapértelmezett prioritás látható a képernyőn, de a többi szövege nem.
* Emuláljunk kattintást!
* Fogalmazzuk meg, hogy most a többi prioritás is megjelent!
* A teszt a közös példa szerint is működik, de itt elég lehet az egyszerűbb `createComposeRule()`, majd a `composeTestRule.setContent` is.

!!!example "BEADANDÓ (1 pont)"
	Készíts egy **képernyőképet**, amelyen látszik a **lefutott teszt**, a **hozzá tartozó kódrészlet**, valamint a **neptun kódod a kódban valahol kommentként**.

	A képet a megoldásban a repository-ba f5.png néven töltsd föl.

	A képernyőkép szükséges feltétele a pontszám megszerzésének.
