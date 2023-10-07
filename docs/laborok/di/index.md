# Labor08 - Függőséginjektálás a Dagger és a Hilt segítségével (Todo)

## Bevezetés

A korábbi laborokon már elsajátítottuk, hogyan lehet az Android alkalmazásunkat lazán csatolt réteges architektúrával megvalósítani. Ez egyértelműen segíti az alkalmazás rugalmas fejlesztését, esetleg az egyes rétegek is lecserélhetők, de ha megnézzük a kódot, még mindig viszonylag jelentős függést találunk, hiszen ahol egy másik rétegbeli komponenst hozunk létre, ott a példányosítás a kódba van "égetve", a másik réteg lecseréléséhez itt is módosítanunk kellene a kódot. Ezekre a problémákra nyújt megoldást a függőséginjektálás (dependency injection). Ez egy általános szoftverfejlesztési technika, amelyet nemcsak Androidon, hanem más platformokon is használunk. Ebben a laborban az Androidon használható Dagger és Hilt könyvtárakat ismerjük meg, amellyel Androidon tudunk függőséginjektálást végezni. A két könyvtárat gyakran együtt használjuk, a Dagger alapvetőbb, alacsonyabb szintű funkciókat nyújt, a Hilt pedig erre épül rá, hogy könnyebbé tegye a fejlesztést.


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

Többféle függőséginjektáló keretrendszer létezik Androidon, és az Android platformon kívül is, ezek némileg más elveken működnek. A Dagger a legjobb teljesítmény érdekében úgy működik, hogy nem futás közben oldja fel a függőségeket, hanem a fordítási folyamatba avatkozik bele, és már aközben feltérképezi a függőségi viszonyok jelölésére alkalmazott annotációkat. Ezért a projekt inicializálásának részeként szükséges felvennünk egy gradle plugint is a folyamatba. Először a projekt szintű `build.gradle.kts` fájlba vegyük fel a a következő sort a pluginek közé:

```kotlin
id("com.google.dagger.hilt.android") version "2.48" apply false
```

Majd a modul szintű `build.gradle` fájlban alkalmazzuk a plugint:

```kotlin
plugins {
    ...

    id("com.google.dagger.hilt.android")
}
```

És a kapthoz kapcsoljuk is be a hibás típusok korrekcióját:

```kotlin
kapt {
    correctErrorTypes = true
}
```

És vegyünk fel még két függőséget, majd szinkronizáljuk a projektet:

```kotlin
// Hilt
implementation("com.google.dagger:hilt-android:2.48")
kapt("com.google.dagger:hilt-compiler:2.48")
implementation("androidx.hilt:hilt-navigation-compose:1.0.0")
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

!!!example "BEADANDÓ (1 pont)" 
	Készíts egy **képernyőképet**, amelyen látszik a **fenti lépésekhez tartozó kódrészlet**, valamint a **neptun kódod a kódban valahol kommentként**. 

	A képet a megoldásban a repository-ba f1.png néven töltsd föl.

	A képernyőkép szükséges feltétele a pontszám megszerzésének.

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

	A képet a megoldásban a repository-ba f2.png néven töltsd föl.

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
                TodosViewModel(
                    todoOperations = todoOperations
                )
            }
        }
    }
```

Ezeket törölnünk kell, viszont gondoskodnunk kell róla, hogy a `TodoUseCases` is inicializálódjon. Jelenlegi készültségben ezt nem tudjuk még injektálni, csak a `TodoRepository` komponenst, ennek segítségével viszont példányosítható a `TodoUseCases`. Illetve még el kell helyezni a viewmodel osztályokon a `@HiltViewModel` annotációt, hogy a Hilt által menedzselt viewmodellekké váljanak. A `TodosViewModel` végül így alakul:

```kotlin
@HiltViewModel
class TodosViewModel @Inject constructor(
    private val repository: TodoRepository
) : ViewModel() {

    val todoOperations: TodoUseCases

    private val _state = MutableStateFlow(TodosState())
    val state = _state.asStateFlow()

    init {
        todoOperations = TodoUseCases(repository)
        loadTodos()
    }
    private fun loadTodos() {

        viewModelScope.launch {
            _state.update { it.copy(isLoading = true) }
            try {
                CoroutineScope(coroutineContext).launch(Dispatchers.IO) {
                    val todos = todoOperations.loadTodos().getOrThrow().map { it.asTodoUi() }
                    _state.update { it.copy(
                        isLoading = false,
                        todos = todos
                    ) }
                }
            } catch (e: Exception) {
                _state.update {  it.copy(
                    isLoading = false,
                    error = e
                ) }
            }
        }
    }
}
```

Hasonló átalakításokat kell végeznünk a `CheckTodoViewModel` osztályon:

```kotlin
@HiltViewModel
class CheckTodoViewModel @Inject constructor(
    private val savedState: SavedStateHandle,
    private val repository: TodoRepository
) : ViewModel() {

    val todoOperations: TodoUseCases

    private val _state = MutableStateFlow(CheckTodoState())
    val state: StateFlow<CheckTodoState> = _state

    private val _uiEvent = Channel<UiEvent>()
    val uiEvent = _uiEvent.receiveAsFlow()

    fun onEvent(event: CheckTodoEvent) {
        when (event) {
            CheckTodoEvent.EditingTodo -> {
                _state.update {
                    it.copy(
                        isEditingTodo = true
                    )
                }
            }

            CheckTodoEvent.StopEditingTodo -> {
                _state.update {
                    it.copy(
                        isEditingTodo = false
                    )
                }
            }

            is CheckTodoEvent.ChangeTitle -> {
                val newValue = event.text
                _state.update {
                    it.copy(
                        todo = it.todo?.copy(title = newValue)
                    )
                }
            }

            is CheckTodoEvent.ChangeDescription -> {
                val newValue = event.text
                _state.update {
                    it.copy(
                        todo = it.todo?.copy(description = newValue)
                    )
                }
            }

            is CheckTodoEvent.SelectPriority -> {
                val newValue = event.priority
                _state.update {
                    it.copy(
                        todo = it.todo?.copy(priority = newValue)
                    )
                }
            }

            is CheckTodoEvent.SelectDate -> {
                val newValue = event.date.toString()
                _state.update {
                    it.copy(
                        todo = it.todo?.copy(dueDate = newValue)
                    )
                }
            }

            CheckTodoEvent.DeleteTodo -> {
                onDelete()
            }

            CheckTodoEvent.UpdateTodo -> {
                onUpdate()
            }
        }
    }

    init {
        todoOperations = TodoUseCases(repository)
        load()
    }

    private fun load() {
        val todoId = checkNotNull<Int>(savedState["id"])
        viewModelScope.launch {
            _state.update { it.copy(isLoadingTodo = true) }
            try {
                val todo = todoOperations.loadTodo(todoId)
                CoroutineScope(coroutineContext).launch(Dispatchers.IO) {
                    _state.update {
                        it.copy(
                            isLoadingTodo = false,
                            todo = todo.getOrThrow().asTodoUi()
                        )
                    }
                }
            } catch (e: Exception) {
                _uiEvent.send(UiEvent.Failure(e.toUiText()))
            }
        }
    }

    private fun onUpdate() {
        viewModelScope.launch(Dispatchers.IO) {
            try {
                todoOperations.updateTodo(
                    _state.value.todo?.asTodo()!!
                )
                _uiEvent.send(UiEvent.Success)
            } catch (e: Exception) {
                _uiEvent.send(UiEvent.Failure(e.toUiText()))
            }
        }
    }

    private fun onDelete() {
        viewModelScope.launch {
            try {
                todoOperations.deleteTodo(state.value.todo!!.id)
                _uiEvent.send(UiEvent.Success)
            } catch (e: Exception) {
                _uiEvent.send(UiEvent.Failure(e.toUiText()))
            }
        }
    }
}
```

Majd a `CreateTodoViewModel` osztályon is:

```kotlin
@HiltViewModel
class CreateTodoViewModel @Inject constructor(
    private val repository: TodoRepository
) : ViewModel() {

    val todoOperations: TodoUseCases

    private val _state = MutableStateFlow(CreateTodoState())
    val state = _state.asStateFlow()

    private val _uiEvent = Channel<UiEvent>()
    val uiEvent = _uiEvent.receiveAsFlow()
    
    init {
        todoOperations = TodoUseCases(repository)
    }

    fun onEvent(event: CreateTodoEvent) {
        when(event) {
            is CreateTodoEvent.ChangeTitle -> {
                val newValue = event.text
                _state.update { it.copy(
                    todo = it.todo.copy(title = newValue)
                ) }
            }
            is CreateTodoEvent.ChangeDescription -> {
                val newValue = event.text
                _state.update { it.copy(
                    todo = it.todo.copy(description = newValue)
                ) }
            }
            is CreateTodoEvent.SelectPriority -> {
                val newValue = event.priority
                _state.update { it.copy(
                    todo = it.todo.copy(priority = newValue)
                ) }
            }
            is CreateTodoEvent.SelectDate -> {
                val newValue = event.date
                _state.update { it.copy(
                    todo = it.todo.copy(dueDate = newValue.toString())
                ) }
            }
            CreateTodoEvent.SaveTodo -> {
                onSave()
            }
        }
    }

    private fun onSave() {
        viewModelScope.launch {
            try {
                todoOperations.saveTodo(state.value.todo.asTodo())
                _uiEvent.send(UiEvent.Success)
            } catch (e: Exception) {
                _uiEvent.send(UiEvent.Failure(e.toUiText()))
            }
        }
    }
}
```

Majd végül az összes screen osztályban változtassuk meg a viewmodel létreozásának módját úgy, hogy a `hiltViewModel()` hívást használjuk. Pl. a `CheckTodoScreen` eddig így nézett ki:

```kotlin
fun CheckTodoScreen(
    onNavigateBack: () -> Unit,
    viewModel: CheckTodoViewModel = viewModel(factory = CheckTodoViewModel.Factory)
) {
	// ...
}
```

A következőképpen módosítsuk:

```kotlin
fun CheckTodoScreen(
    onNavigateBack: () -> Unit,
    viewModel: CheckTodoViewModel = hiltViewModel()
) {
	// ...
}
```

!!!example "BEADANDÓ (1 pont)" 
	Készíts egy **képernyőképet**, amelyen látszik a **futó alkalmazás**, a **fenti lépésekhez tartozó kódrészlet**, valamint a **neptun kódod a kódban valahol kommentként**. 

	A képet a megoldásban a repository-ba f3.png néven töltsd föl.

	A képernyőkép szükséges feltétele a pontszám megszerzésének.

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

A `CheckTodoViewModel` kódja:

```kotlin
@HiltViewModel
class CheckTodoViewModel @Inject constructor(
    private val savedState: SavedStateHandle,
    private val todoOperations: TodoUseCases
) : ViewModel() {

    private val _state = MutableStateFlow(CheckTodoState())
    val state: StateFlow<CheckTodoState> = _state

    private val _uiEvent = Channel<UiEvent>()
    val uiEvent = _uiEvent.receiveAsFlow()

    fun onEvent(event: CheckTodoEvent) {
        when (event) {
            CheckTodoEvent.EditingTodo -> {
                _state.update {
                    it.copy(
                        isEditingTodo = true
                    )
                }
            }

            CheckTodoEvent.StopEditingTodo -> {
                _state.update {
                    it.copy(
                        isEditingTodo = false
                    )
                }
            }

            is CheckTodoEvent.ChangeTitle -> {
                val newValue = event.text
                _state.update {
                    it.copy(
                        todo = it.todo?.copy(title = newValue)
                    )
                }
            }

            is CheckTodoEvent.ChangeDescription -> {
                val newValue = event.text
                _state.update {
                    it.copy(
                        todo = it.todo?.copy(description = newValue)
                    )
                }
            }

            is CheckTodoEvent.SelectPriority -> {
                val newValue = event.priority
                _state.update {
                    it.copy(
                        todo = it.todo?.copy(priority = newValue)
                    )
                }
            }

            is CheckTodoEvent.SelectDate -> {
                val newValue = event.date.toString()
                _state.update {
                    it.copy(
                        todo = it.todo?.copy(dueDate = newValue)
                    )
                }
            }

            CheckTodoEvent.DeleteTodo -> {
                onDelete()
            }

            CheckTodoEvent.UpdateTodo -> {
                onUpdate()
            }
        }
    }

    init {
        load()
    }

    private fun load() {
        val todoId = checkNotNull<Int>(savedState["id"])
        viewModelScope.launch {
            _state.update { it.copy(isLoadingTodo = true) }
            try {
                val todo = todoOperations.loadTodo(todoId)
                CoroutineScope(coroutineContext).launch(Dispatchers.IO) {
                    _state.update {
                        it.copy(
                            isLoadingTodo = false,
                            todo = todo.getOrThrow().asTodoUi()
                        )
                    }
                }
            } catch (e: Exception) {
                _uiEvent.send(UiEvent.Failure(e.toUiText()))
            }
        }
    }

    private fun onUpdate() {
        viewModelScope.launch(Dispatchers.IO) {
            try {
                todoOperations.updateTodo(
                    _state.value.todo?.asTodo()!!
                )
                _uiEvent.send(UiEvent.Success)
            } catch (e: Exception) {
                _uiEvent.send(UiEvent.Failure(e.toUiText()))
            }
        }
    }

    private fun onDelete() {
        viewModelScope.launch {
            try {
                todoOperations.deleteTodo(state.value.todo!!.id)
                _uiEvent.send(UiEvent.Success)
            } catch (e: Exception) {
                _uiEvent.send(UiEvent.Failure(e.toUiText()))
            }
        }
    }
}
```

A `CreateTodoViewModel` így fest:

```kotlin
@HiltViewModel
class CreateTodoViewModel @Inject constructor(
    private val todoOperations: TodoUseCases
) : ViewModel() {

    private val _state = MutableStateFlow(CreateTodoState())
    val state = _state.asStateFlow()

    private val _uiEvent = Channel<UiEvent>()
    val uiEvent = _uiEvent.receiveAsFlow()

    fun onEvent(event: CreateTodoEvent) {
        when(event) {
            is CreateTodoEvent.ChangeTitle -> {
                val newValue = event.text
                _state.update { it.copy(
                    todo = it.todo.copy(title = newValue)
                ) }
            }
            is CreateTodoEvent.ChangeDescription -> {
                val newValue = event.text
                _state.update { it.copy(
                    todo = it.todo.copy(description = newValue)
                ) }
            }
            is CreateTodoEvent.SelectPriority -> {
                val newValue = event.priority
                _state.update { it.copy(
                    todo = it.todo.copy(priority = newValue)
                ) }
            }
            is CreateTodoEvent.SelectDate -> {
                val newValue = event.date
                _state.update { it.copy(
                    todo = it.todo.copy(dueDate = newValue.toString())
                ) }
            }
            CreateTodoEvent.SaveTodo -> {
                onSave()
            }
        }
    }

    private fun onSave() {
        viewModelScope.launch {
            try {
                todoOperations.saveTodo(state.value.todo.asTodo())
                _uiEvent.send(UiEvent.Success)
            } catch (e: Exception) {
                _uiEvent.send(UiEvent.Failure(e.toUiText()))
            }
        }
    }
}
```

A `TodosViewModel` átalakított verziója pedig az alábbi:

```kotlin
@HiltViewModel
class TodosViewModel @Inject constructor(
    private val todoOperations: TodoUseCases
) : ViewModel() {

    private val _state = MutableStateFlow(TodosState())
    val state = _state.asStateFlow()

    init {
        loadTodos()
    }
    private fun loadTodos() {

        viewModelScope.launch {
            _state.update { it.copy(isLoading = true) }
            try {
                CoroutineScope(coroutineContext).launch(Dispatchers.IO) {
                    val todos = todoOperations.loadTodos().getOrThrow().map { it.asTodoUi() }
                    _state.update { it.copy(
                        isLoading = false,
                        todos = todos
                    ) }
                }
            } catch (e: Exception) {
                _state.update {  it.copy(
                    isLoading = false,
                    error = e
                ) }
            }
        }
    }
}
```

!!!example "BEADANDÓ (1 pont)" 
	Készíts egy **képernyőképet**, amelyen látszik a **futó alkalmazás**, a **fenti lépésekhez tartozó kódrészlet**, valamint a **neptun kódod a kódban valahol kommentként**. 

	A képet a megoldásban a repository-ba f4.png néven töltsd föl.

	A képernyőkép szükséges feltétele a pontszám megszerzésének.

## Önálló feladat 

A `TodoUseCases` osztályban egyelőre csak a `LoadTodosUseCase` függőséget hozzuk létre a modulban, és injektáljuk a Dagger/Hilt segítségével, a többi usecase most is manuálisan példányosodik a repository átadásával. Folytasd az átalakítást, és hozd létre az összes usecase-t a usecase modulban, hogy utána már a Dagger/Hilt kezelje őket!

!!!example "BEADANDÓ (1 pont)" 
	Készíts egy **képernyőképet**, amelyen látszik a **futó alkalmazás**, az **átalakított kódrészlet**, valamint a **neptun kódod a kódban valahol kommentként**. 

	A képet a megoldásban a repository-ba f5.png néven töltsd föl.

	A képernyőkép szükséges feltétele a pontszám megszerzésének.
