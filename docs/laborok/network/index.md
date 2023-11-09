# Labor09 - Network és Paging (Unsplash)

## Bevezetés

Ezen a laboron megismerkedünk Android a hálózati hívások elvégzésének egyik legelterjedtebb megvalósításával (Retrofit), a Paging Library-vel, illetve azzal, hogy a Compose keretrendszerben hogyan tudunk különböző képernyőméreteket támogatni.


## Előkészületek

A feladatok megoldása során ne felejtsd el követni a [feladat beadás folyamatát](../../tudnivalok/github/GitHub.md).

### Git repository létrehozása és letöltése

1. Moodle-ben keresd meg a laborhoz tartozó meghívó URL-jét és annak segítségével hozd létre a saját repository-dat.

2. Várd meg, míg elkészül a repository, majd checkout-old ki.

    !!! tip ""
        Egyetemi laborokban, ha a checkout során nem kér a rendszer felhasználónevet és jelszót, és nem sikerül a checkout, akkor valószínűleg a gépen korábban megjegyzett felhasználónévvel próbálkozott a rendszer. Először töröld ki a mentett belépési adatokat (lásd [itt](../../tudnivalok/github/GitHub-credentials.md)), és próbáld újra.

3. Hozz létre egy új ágat `megoldas` néven, és ezen az ágon dolgozz.

4. A `neptun.txt` fájlba írd bele a Neptun kódodat. A fájlban semmi más ne szerepeljen, csak egyetlen sorban a Neptun kód 6 karaktere.

Indítsuk el az Android Studio-t, majd nyissuk meg a repository-ban lévő projektet.

## Meglévő projekt áttekintése

A meglévő projektben megtalálható az elkészített felület, illetve a hozzá tartozó Viewmodellek.
Tekintsük át a laborvezetővel a meglévő kódot!

## Adat- és hálózati réteg

### Könyvtárak

!!! info "Retrofit" 

    A [Retrofit](https://square.github.io/retrofit/) egy általános célú HTTP könyvtár Java és Kötlin környezetben. Széles körben használják, számos projektben bizonyított már (kvázi ipari standard). Azért használjuk, hogy ne kelljen alacsony színtű hálózati hívásokat implementálni.

    Segítségével elég egy interface-ben annotációk segítségével leírni az API-t (ez pl. a [Swagger](https://swagger.io/) eszközzel generálható is), majd e mögé készít a Retrofit egy olyan osztályt, mely a szükséges hálózati hívásokat elvégzi. A Retrofit a háttérben az [OkHttp3](https://github.com/square/okhttp)-at használja, valamint az objektumok JSON formátumba történő sorosítását a [Moshi](https://github.com/square/moshi) libraryvel végzi. Ezért ezeket is be kell hivatkozni.

!!! info "Paging 3.0"  

    A Paging Library az Android Jetpack része. Segít az adatok oldalankénti betöltésében és megjelenítésében nagyobb adatkészletből, helyi tárolóból vagy hálózatról. Ez a megközelítés lehetővé teszi az alkalmazásunk számára, hogy mind a hálózati sávszélességet, mind pedig a rendszer erőforrásait hatékonyabban használja.

    A Paging Library használatának előnyei:  

    - Az oldalankénti adatok memóriában történő gyorsítótárazása. Ez biztosítja, hogy az alkalmazás hatékonyan használja a rendszer erőforrásait oldalankénti adatokkal való munka során.  
    - Beépített kérések duplikálódásának megakadályozása, hogy az alkalmazás hatékonyan használja a hálózati sávszélességet és a rendszer erőforrásait.  
    - Konfigurálható RecyclerView adapterek, amelyek automatikusan lekérik az adatokat, amikor a felhasználó görget a betöltött adatok végére.  
    - Elsőosztályú támogatás Kotlin coroutines és Flow, valamint LiveData és RxJava számára.  
    - Beépített hibakezelés-támogatás, beleértve a frissítési és újrapróbálási képességeket.  

    A Paging Library főbb elemei:

    - PagingData - egy tároló 'oldalazott' adatok számára. Az adatok frissítésekor külön PagingData tartályt használunk.  
    - PagingSource - a PagingSource az alap osztály az adatok részletekben való betöltéséhez PagingData streambe.  
    - Pager.flow - egy Flow<PagingData>-t hoz létre, amely a PagingConfig és egy függvény alapján konstruálja a megvalósított PagingSource-t.  
    - RemoteMediator - segít az oldalazás megvalósításában hálózatról és adatbázisból.  

!!! info "Coil"

    A Coil (Coroutine Image Loader) egy kép betöltő könyvtár Androidra, amelyet a Kotlin koroutinokra épül.   
    A Coil használatának előnyei:  

    - Gyors: A Coil számos optimalizálást végez, beleértve a memória- és lemeztároló gyorsítótárazást, az átméretezést a memóriában, az automatikus kérések szüneteltetését/leállítását és még sok mást.  
    - Könnyű: A Coil kb. 2000 metódust ad az APK-hoz (azoknak az alkalmazásoknak, amelyek már használják az OkHttp és a Coroutines könyvtárakat), ami hasonló a Picasso-hoz és jelentősen kevesebb, mint a Glide és a Fresco könyvtárak.  
    - Könnyen használható: A Coil API-ja a Kotlin nyelv funkcióit használja a könnyű használat és a minimális boilerplate kód érdekében.  
    - Modern: A Coil a Kotlin nyelvűséget helyezi előtérbe és a modern könyvtárakat használja, beleértve a Coroutines-t, az OkHttp-t, az Okio-t és az AndroidX Lifecycles-t.  

### Függőségek

A Retrofit, Paging és Coil könyvtárak használatához a következő függőségek szükségesek (ezek már szerepelnek a projektben, ne vegyük fel őket újra):
```kotlin
    // Retrofit
    val retrofit_version = "2.9.0"
    implementation("com.squareup.retrofit2:retrofit:$retrofit_version")
    implementation("com.squareup.retrofit2:converter-gson:$retrofit_version")
    implementation("com.squareup.retrofit2:converter-moshi:$retrofit_version")

    // Paging 3.0
    implementation("androidx.paging:paging-compose:3.3.0-alpha02")

    // Coil
    implementation("io.coil-kt:coil-compose:2.5.0")
```

A `data.model` package-be hozzuk létre az alábbi két fájlt, melyek az API használatához szükségesek:

`UnsplashPhoto.kt`:
```kotlin
@Entity(tableName = "photos")
data class UnsplashPhoto(
    @PrimaryKey(autoGenerate = false)
    val id: String,
    val likes: Int,
    @Embedded
    val user: UserData,
    @Embedded
    val urls: Urls
)

data class UserData(
    val username: String,
    val name: String,
    @field:Json(name = "total_likes")
    val totalLikes: Int,
    @field:Json(name = "total_photos")
    val totalPhotos: Int,
    @field:Json(name = "profile_image") @Embedded
    val profileImage: UserProfileImage,
)

data class UserProfileImage(
    val small: String,
    val medium: String,
    val large: String
)

data class Urls(
    val regular: String,
    val full: String
)
```

Az `@Embedded` annotációval megjelelőt mezők osztályainak mezői közvetlenül hivatkozhatók az SQL querykben.  

A `Moshi` automatikusan megoldja majd az egyes tagváltozók szerializálását, kivéve ott, ahol eltér a név. Ezt a `@field:Json` annotációval jelezhetjük.

`SearchResult.kt`:
```kotlin
data class SearchResult(
    @field:Json(name = "results") val photos: List<UnsplashPhoto>
)
```

Hozzuk létre az első DAO-t a `data.local.dao` package-ben:

`UnsplashPhotoDao.kt`:
```kotlin
@Dao
interface UnsplashPhotoDao {
    @Insert
    suspend fun insertPhotos(photos: List<UnsplashPhoto>)

    @Query("SELECT EXISTS (SELECT * FROM photos WHERE id = :id)")
    fun exists(id: String): Flow<Boolean>

    @Query("DELETE FROM photos")
    suspend fun deleteAllPhotos()

    @Query("SELECT * FROM photos")
    fun getAllPhotos(): PagingSource<Int, UnsplashPhoto>

    @Query("SELECT * FROM photos WHERE id = :id")
    fun getPhotoById(id: String): Flow<UnsplashPhoto>
}
```

Majd a `data.local.database` package-ben hozzuk létre az adatbázist (az `UnsplashPhotoRemoteKeysDao` osztályt később hozzuk létre):

`UnsplashDatabase.kt`:
```kotlin
@Database(entities = [UnsplashPhoto::class, UnsplashPhotoRemoteKeys::class], version = 1)
abstract class UnsplashDatabase : RoomDatabase() {
    abstract val photosDao: UnsplashPhotoDao
    abstract val remoteKeysDao: UnsplashPhotoRemoteKeysDao
}
```

!!!example "BEADANDÓ (1 pont)" 
	Készíts egy **képernyőképet**, az **adatábzishoz tartozó kódrészlet**, valamint a **neptun kódod a kódban valahol kommentként**. 

	A képet a megoldásban a repository-ba f1.png néven töltsd föl.

	A képernyőkép szükséges feltétele a pontszám megszerzésének.

### A Retrofit interfész

Az Unsplash API három végpontját fogjuk használni a képek, egy adott kép lekérésére, valamint a keresés végrehajtására.
Hozzuk létre a lenti interface-t a `data.remote.api` package-ben.

`UnsplashApi.kt`:
```kotlin
interface UnsplashApi {

    @GET("/photos")
    suspend fun getPhotosFromEditorialFeed(
        @Query("page") page: Int = 1,
        @Query("per_page") perPage: Int = 10,
        @Query("client_id") clientId: String = "ACCESS_KEY",
    ): Response<List<UnsplashPhoto>>

    @GET("/photos/{id}")
    suspend fun getPhotoById(
        @Path("id") id: String,
        @Query("client_id") clientId: String = "ACCESS_KEY",
    ): Response<UnsplashPhoto>

    @GET("/search/photos")
    suspend fun getSearchResults(
        @Query("client_id") clientId: String = "ACCESS_KEY",
        @Query("page") page: Int = 1,
        @Query("per_page") perPage: Int = 10,
        @Query("query") searchTerms: String,
    ): Response<SearchResult>

}
```

Ha a Reponse-t nem tudná beimportálni az Android Studio, vegyük fel kézzel az `import retrofit2.Response` sort.

A Retrofit annotációi segítségével egyszerűen tudjuk definiálni a kéréseinket.  
Cseréljük le az itt szereplő `ACCESS_KEY` stringet az Unsplashen regisztráció után elérhető saját kulcsunkra.
Az https://unsplash.com/oauth/applications oldalon regisztráció után egy új applikációt kell létrehozni, és azt megnyitva találhatjuk meg a kulcsot.

### A PagingSource osztály

Az első lépés egy PagingSource implementáció meghatározása, hogy az adatforrás azonosítható legyen. A PagingSource API osztálya tartalmazza a load() metódust, amelyet felül kell írni, hogy jelentse, hogyan lehet lapozott adatokat visszanyerni a megfelelő adatforrásból.

Hozzuk létre az alábbi osztályt a `data.paging` package-ben:

`SearchPagingSource.kt`:
```kotlin
class SearchPagingSource(
    private val api: UnsplashApi,
    private val searchTerms: String
): PagingSource<Int, UnsplashPhoto>() {

    override fun getRefreshKey(state: PagingState<Int, UnsplashPhoto>): Int? = state.anchorPosition

    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, UnsplashPhoto> {
        val currentPage = params.key ?: 1
        return try {
            val response = api.getSearchResults(searchTerms = searchTerms, perPage = INITIAL_PAGE_SIZE, page = currentPage)
            if (response.isSuccessful) {
                val photos = response.body()?.photos ?: emptyList()
                val endOfPaginationReached = photos.isEmpty()
                LoadResult.Page(
                    data = photos,
                    prevKey = if (currentPage == 1) null else currentPage - 1,
                    nextKey = if (endOfPaginationReached) null else currentPage + 1
                )
            } else throw Exception(response.errorBody()?.string() ?: "Unsuccessful request.")
        } catch (e: Exception) {
            Log.e("error",e.stackTraceToString())
            LoadResult.Error(e)
        }
    }
}
```

A `PagingSource<Key, Value>` két típusparamétert tartalmaz: `Key` és `Value`. A kulcs meghatározza az azonosítót, amelyet az adat betöltéséhez használnak, és az érték maga az adat típusa. 

Egy tipikus `PagingSource` implementáció a konstruktorában megadott paramétereket továbbítja a `load()` metódusnak, hogy betölthesse a megfelelő adatokat egy lekérdezéshez. 

A `LoadParams` objektum információkat tartalmaz az elvégzendő betöltési műveletről. Ide tartozik a betöltendő kulcs és a betöltendő elemek száma.

A `LoadResult` objektum az adatbetöltés eredményét tartalmazza. A `LoadResult` egy zárt osztály, amely két formában jelenhet meg attól függően, hogy a `load()` hívás sikeres volt-e vagy sem:  

- Sikeres esetben `LoadResult.Page` objektum  
- Sikertelen esetben `LoadResult.Error` objektum.  

A try-catch blokkban látható, hogy hogyan tudjuk használni a Retrofit interfészünket hálózati hívás elvégzésére.
  
A `PagingSource` implementációja emellett tartalmaznia kell egy `getRefreshKey()` metódust, amely egy `PagingState` objektumot vár paraméterként, és visszaadja a kulcsot, amelyet át kell adni a `load()` metódusnak, amikor az adat frissítése vagy érvénytelenítése történik az első betöltés után. A Paging könyvtár automatikusan meghívja ezt a metódust az adat későbbi frissítésekor.

### A RemoteMediator osztály

Jobb felhasználói élményt biztosíthatunk azzal, ha gondoskodunk arról, hogy az alkalmazásunk használható legyen akkor is, ha az internetkapcsolat instabil vagy ha a felhasználó offline. Az egyik módja ennek, hogy egyszerre lapozunk a hálózatról és egy helyi adatbázisból is. Ezzel az alkalmazás az UI-t egy helyi adatbázisból vezérli és csak akkor kér le adatokat a hálózatról, ha már nincs több adat az adatbázisban.

A Paging könyvtár a `RemoteMediator` komponenst biztosítja ehhez az esethez. A `RemoteMediator` a Paging könyvtár jelzéseként működik, amikor az alkalmazás kimerül az előtárolt adatból. Ezt a jelzést lehet használni további adatok betöltésére a hálózatról, és azokat helyi adatbázisban tárolni, ahol egy `PagingSource` betöltheti és a felhasználói felületen megjelenítheti.

Ha további adatokra van szükség, a Paging könyvtár meghívja a `RemoteMediator` implementációjából a `load()` metódust. Ez egy suspending függvény, így hosszú ideig futó munkát végezhet biztonságosan. Ez a függvény általában az új adatokat egy hálózati forrásból szedi le, majd elmenti helyi tárolóba.

Ez a folyamat új adatokkal működik, de idővel az adatok tárolása az adatbázisban érvénytelenítést igényelhet, például amikor a felhasználó kézzel indítja el a frissítést. Ezt a `LoadType` tulajdonság jelzi, amelyet át kell adni a `load()` metódusnak. A `LoadType` tájékoztatja a `RemoteMediator`-t arról, hogy a meglévő adatokat frissíteni kell-e, vagy olyan további adatokat kell-e lekérni, amelyeket a meglévő listához kell hozzáadni.

Így a `RemoteMediator` biztosítja, hogy az alkalmazás azokat az adatokat töltse be, amelyeket a felhasználók a megfelelő sorrendben szeretnének látni.

Hozzuk létre az alábbi osztályt a `data.paging` package-ben:

`UnsplashRemoteMediator.kt`:
```kotlin
@ExperimentalPagingApi
class UnsplashRemoteMediator(
    private val api: UnsplashApi,
    private val db: UnsplashDatabase
): RemoteMediator<Int, UnsplashPhoto>() {

    override suspend fun load(
        loadType: LoadType,
        state: PagingState<Int, UnsplashPhoto>
    ): MediatorResult {
        return try {
            val page = when (loadType) {
                LoadType.REFRESH -> {
                    val remoteKeys = getRemoteKeysForClosestToPosition(state)
                    remoteKeys?.nextKey?.minus(1) ?: INITIAL_PAGE
                }
                LoadType.PREPEND -> return MediatorResult.Success(endOfPaginationReached = true)
                LoadType.APPEND -> {
                    val remoteKeys = getRemoteKeysForLastItem(state) ?: throw InvalidObjectException("Result is empty")
                    remoteKeys.nextKey ?: return MediatorResult.Success(endOfPaginationReached = true)
                }
            }

            val response = api.getPhotosFromEditorialFeed(page = page, perPage = INITIAL_PAGE_SIZE)
            var endOfPaginationReached = false
            if (response.isSuccessful) {
                val photos = response.body() ?: emptyList()
                endOfPaginationReached = photos.isEmpty()
                db.withTransaction {
                    if (loadType == LoadType.REFRESH) {
                        db.photosDao.deleteAllPhotos()
                        db.remoteKeysDao.deleteAllKeys()
                    }
                    val prevKey = if (page == INITIAL_PAGE) null else page - 1
                    val nextKey = if (photos.isEmpty()) null else page + 1
                    val keys = photos.map { photo ->
                        UnsplashPhotoRemoteKeys(
                            id = photo.id,
                            prevKey = prevKey,
                            nextKey = nextKey
                        )
                    }
                    db.remoteKeysDao.insertAllKeys(keys)
                    db.photosDao.insertPhotos(photos)
                }
            }

            MediatorResult.Success(endOfPaginationReached = endOfPaginationReached)

        } catch (e: IOException) {
            MediatorResult.Error(e)
        } catch (e: HttpException) {
            MediatorResult.Error(e)
        } catch (e: InvalidObjectException) {
            MediatorResult.Error(e)
        }
    }

    private suspend fun getRemoteKeysForLastItem(
        state: PagingState<Int, UnsplashPhoto>
    ): UnsplashPhotoRemoteKeys? {
        return state.lastItemOrNull()?.let { photo ->
            db.withTransaction {
                db.remoteKeysDao.getKeysById(photo.id)
            }
        }
    }

    private suspend fun getRemoteKeysForClosestToPosition(
        state: PagingState<Int, UnsplashPhoto>
    ): UnsplashPhotoRemoteKeys? {
        return state.anchorPosition?.let { position ->
            state.closestItemToPosition(position)?.id?.let { id ->
                db.withTransaction {
                    db.remoteKeysDao.getKeysById(id)
                }
            }
        }
    }
}
```

A `load()` metódus visszatérési értéke egy `MediatorResult` objektum. A `MediatorResult` lehet `MediatorResult.Error` (amely tartalmazza az hiba leírását) vagy `MediatorResult.Success` (amely tartalmaz egy jelzést arról, hogy van-e még több adat betöltésre).

A `load()` metódusnak a következő lépéseket kell végrehajtania:

- Meg kell határozni, hogy melyik oldalt kell a hálózatról betölteni a betöltési típus és az eddig betöltött adatok alapján.  
- Kiváltani a hálózati kérést.  
- Végrehajtani a betöltési művelet kimenetétől függő cselekvéseket:  
  - Ha a betöltés sikeres és az kapott elemek listája nem üres, akkor tárolja el a lista elemeit az adatbázisban, majd térjen vissza a `MediatorResult.Success` `(endOfPaginationReached = false)` értékkel. Az adatok tárolása után érvénytelenítse az adatforrást, hogy értesítse a Paging könyvtárat az új adatokról.
  - Ha a betöltés sikeres és a kapott elemek listája üres vagy az utolsó oldal indexe, akkor térjen vissza a `MediatorResult.Success` `(endOfPaginationReached = true)` értékkel. Az adatok tárolása után érvénytelenítse az adatforrást, hogy értesítse a Paging könyvtárat az új adatokról.
  - Ha a kérés hibát okoz, akkor térjen vissza a `MediatorResult.Error` értékkel.

### Távoli kulcsok

Kezelnünk kell azt a helyzetet, amikor a helyi gyorsítótárban tárolt adatok elavultak a távoli adatforráshoz képest.

A távoli kulcsok lehetővé teszik, hogy információt menthessünk el a legutóbbi oldalról, amelyet a szerverről kértek le. Az alkalmazás felhasználhatja ezt az információt a következő betöltendő adatoldal azonosításához és kéréséhez.

A távoli kulcsok olyan kulcsok, amelyeket a RemoteMediator implementáció arra használ, hogy közölje a backend szolgáltatással, melyik adatot kell legközelebb betölteni. A legegyszerűbb esetben minden lapozott adat elemhez tartozik egy távoli kulcs, amelyre könnyen hivatkozhat. Azonban ha a távoli kulcsok nem feleltethetőek meg a konkrét elemeknek, nekünk kell őket kezelni a load hívásban.

Amikor a távoli kulcsok nem közvetlenül kapcsolódnak a listaelemekhez, célszerű őket külön táblázatban tárolni a helyi adatbázisban. Definiálni kell egy Room entitást, amely egy távoli kulcsokból álló táblázatot reprezentál.

Hozzuk létre az alábbi osztályt a `data.local.model` package-ben:

`UnsplashPhotoRemoteKeys.kt`:
```kotlin
@Entity(tableName = "remote_keys")
data class UnsplashPhotoRemoteKeys(
    @PrimaryKey val id: String,
    val prevKey: Int?,
    val nextKey: Int?
)
```

Emellett definiálni kell egy DAO-t a RemoteKey entitásra.

Hozzuk létre az alábbi osztályt a `data.local.dao` package-ben:

`UnsplashPhotoRemoteKeysDao.kt`:
```kotlin
@Dao
interface UnsplashPhotoRemoteKeysDao {

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertAllKeys(keys: List<UnsplashPhotoRemoteKeys>)

    @Query("SELECT * FROM remote_keys WHERE id = :id")
    suspend fun getKeysById(id: String): UnsplashPhotoRemoteKeys

    @Query("DELETE FROM remote_keys")
    suspend fun deleteAllKeys()
}
```

### PagingUtil


Hozzuk létre a `PagingUtil` osztályt az `util` package-ben a paging konfigurálására és placeholderek beállítására:

`PagingUtil.kt`:
```kotlin
object PagingUtil {
    const val INITIAL_PAGE_SIZE = 10
    const val INITIAL_PAGE = 1

    fun <T: Any> LazyGridScope.items(
        items: LazyPagingItems<T>,
        key: ((item: T) -> Any)? = null,
        itemContent: @Composable LazyGridItemScope.(value: T?) -> Unit
    ) {
        items(
            count = items.itemCount,
            key = if (key == null) null else { index ->
                val item = items.peek(index)
                if (item == null) {
                    PagingPlaceholderKey(index)
                } else {
                    key(item)
                }
            }
        ) { index ->
            itemContent(items[index])
        }
    }

    data class PagingPlaceholderKey(private val index: Int) : Parcelable {
        override fun writeToParcel(parcel: Parcel, flags: Int) {
            parcel.writeInt(index)
        }

        override fun describeContents(): Int {
            return 0
        }

        companion object {
            @Suppress("unused")
            @JvmField
            val CREATOR: Parcelable.Creator<PagingPlaceholderKey> =
                object : Parcelable.Creator<PagingPlaceholderKey> {
                    override fun createFromParcel(parcel: Parcel) =
                        PagingPlaceholderKey(parcel.readInt())

                    override fun newArray(size: Int) = arrayOfNulls<PagingPlaceholderKey?>(size)
                }
        }
    }
}
```

A `data.repository` package-be vegyük fel a lenti `DataSource` osztályt, melyen keresztül a ViewModel eléri a kéréseinket:

`UnsplashPhotoDataSource.kt`:
```kotlin
@ExperimentalPagingApi
class UnsplashPhotoDataSource(
    private val api: UnsplashApi,
    private val db: UnsplashDatabase
) {

    fun getAllPhotos(): Flow<PagingData<UnsplashPhoto>> {
        return Pager(
            config = PagingConfig(pageSize = INITIAL_PAGE_SIZE),
            remoteMediator = UnsplashRemoteMediator(api, db),
            pagingSourceFactory = {  db.photosDao.getAllPhotos() }
        ).flow
    }

    fun getPhotoByIdFromDatabase(id: String): Flow<UnsplashPhoto> = db.photosDao.getPhotoById(id)

    fun getSearchResults(searchTerms: String): Flow<PagingData<UnsplashPhoto>> {
        return Pager(
            config = PagingConfig(pageSize = INITIAL_PAGE_SIZE),
            pagingSourceFactory = {
                SearchPagingSource(
                    api = api,
                    searchTerms = searchTerms
                )
            }
        ).flow
    }

    suspend fun exists(id: String): Boolean = db.photosDao.exists(id).first()

    suspend fun getPhotoByIdFromApi(id: String): Flow<UnsplashPhoto> {
        val response = api.getPhotoById(id)
        return if (response.isSuccessful) {
            flow { emit(response.body() ?: throw NullPointerException()) }
        } else throw Exception("Unsuccessful request")
    }
}
```

Ha a flow-emit résznél hibát kapunk, cseréljük le az importot a következőre: `import kotlinx.coroutines.flow.flow`.

Itt láthatjuk a `Pager` osztály használatát a lapozott adatok folyamának beállításához.
	
Végül hozzuk létre a saját Application osztályunkat a gyökér package-ben:

`UnsplashApplication.kt`:
```kotlin	
@ExperimentalPagingApi
class UnsplashApplication : Application() {
    companion object {
        lateinit var photoDataSource: UnsplashPhotoDataSource
    }

    override fun onCreate() {
        super.onCreate()
        val db = Room.databaseBuilder(
            this.baseContext,
            UnsplashDatabase::class.java,
            "unsplash_db"
        ).build()

        val client = OkHttpClient.Builder()
            .readTimeout(15, TimeUnit.SECONDS)
            .connectTimeout(15, TimeUnit.SECONDS)
            .build()

        val retrofit = Retrofit.Builder()
            .baseUrl("https://api.unsplash.com/")
            .client(client)
            .addConverterFactory(MoshiConverterFactory.create())
            .build()

        val api = retrofit.create(UnsplashApi::class.java)

        photoDataSource = UnsplashPhotoDataSource(api,db)

    }
}
```

!!!example "BEADANDÓ (1 pont)" 
	Készíts egy **képernyőképet**, amelyen látszik a **működő alkalmazás listázó képernyője** (emulátoron, készüléket tükrözve vagy képernyőfelvétellel),  a **Paging-hez tartozó kódrészlet**, valamint a **neptun kódod a kódban valahol kommentként**. 

	A képet a megoldásban a repository-ba f2.png néven töltsd föl.

	A képernyőkép szükséges feltétele a pontszám megszerzésének.

!!!example "BEADANDÓ (1 pont)" 
	Készíts egy **képernyőképet**, amelyen látszik a **működő alkalmazás részletes képernyője** (emulátoron, készüléket tükrözve vagy képernyőfelvétellel),  a **Paging-hez tartozó kódrészlet**, valamint a **neptun kódod a kódban valahol kommentként**. 

	A képet a megoldásban a repository-ba f3.png néven töltsd föl.

	A képernyőkép szükséges feltétele a pontszám megszerzésének.

## Önálló feladat 1 - Különböző képernyőméretek támogatása

Szeretnénk felkészíteni az alkalmazásunkat, hogy különböző méretű képernyők esetén máshogy, az adott készülék számára optimális módon jelenleg meg.
Ezzez elsőként a `util` package-ben hozzunk létre egy osztályt a kategóriáinkra:

`WindowSize.kt`:
```kotlin
enum class WindowSize { Compact, Medium, Expanded }
```

Ezt követően hozzuk létre a maradék két elrendezésünket a `feature.photos_feed.screensbysize` package-ben:

`PhotosFeedScreen_Compact.kt`:
```kotlin
@ExperimentalMaterial3Api
@ExperimentalMaterialApi
@Composable
fun PhotosFeedScreen_Compact(
    refreshState: PullRefreshState,
    refreshing: Boolean,
    onPhotoItemClick: (String) -> Unit,
    photos: LazyPagingItems<UnsplashPhotoUiModel>,
    value: String,
    onValueChange: (String) -> Unit,
    modifier: Modifier = Modifier,
) {
    val state = rememberLazyListState()
    Scaffold(
        modifier = modifier,
        topBar = {
            SearchTopAppBar(
                isScrollInProgress = state.isScrollInProgress,
                value = value,
                onValueChange = onValueChange
            )
        }
    ) {
        Box(
            modifier = modifier
                .fillMaxSize()
                .pullRefresh(refreshState)
                .padding(it)
        ) {
            if (!refreshing) {
                LazyColumn(
                    modifier = modifier.fillMaxSize()
                ) {
                    items(photos) { photo ->
                        photo?.let { model ->
                            PhotoItem(
                                photo = model,
                                onClick = onPhotoItemClick
                            )
                        }
                    }
                }
            }
            PullRefreshIndicator(
                refreshing = refreshing,
                state = refreshState,
                modifier = Modifier.align(Alignment.TopCenter)
            )
        }
    }
}
```

`PhotosFeedScreen_Expanded.kt`:
```kotlin
@ExperimentalMaterial3Api
@ExperimentalMaterialApi
@Composable
fun PhotosFeedScreen_Expanded(
    onPhotoItemClick: (String) -> Unit,
    photos: LazyPagingItems<UnsplashPhotoUiModel>,
    value: String,
    onValueChange: (String) -> Unit,
    modifier: Modifier = Modifier,
    loadedPhoto: UnsplashPhotoUiModel? = null,
) {
    val density = LocalDensity.current
    val configuration = LocalConfiguration.current
    val screenWidth = configuration.screenWidthDp.dp
    val state = rememberLazyGridState()

    Row(modifier = modifier.fillMaxSize()) {
        Row(
            modifier = Modifier.fillMaxSize()
        ) {
            Scaffold(
                modifier = Modifier.fillMaxSize(),
                topBar = {
                    SearchTopAppBar(
                        isScrollInProgress = state.isScrollInProgress,
                        value = value,
                        onValueChange = onValueChange
                    )
                }
            ) {
                LazyVerticalGrid(
                    state = state,
                    columns = GridCells.Adaptive(240.dp),
                    modifier = Modifier
                        .fillMaxSize()
                        .padding(it)
                        .weight(1f)
                ) {
                    items(photos) { photo ->
                        photo?.let { model ->
                            PhotoItem(
                                photo = model,
                                onClick = onPhotoItemClick
                            )
                        }
                    }
                }
            }

            AnimatedVisibility(
                visible = loadedPhoto != null && screenWidth >= 1000.dp,
                enter = slideInHorizontally {
                    with(density) { 40.dp.roundToPx() }
                } + expandHorizontally(
                    expandFrom = Alignment.End
                ) + fadeIn(
                    initialAlpha = 0.3f
                ),
                exit = slideOutHorizontally() + shrinkHorizontally() + fadeOut(),
                modifier = Modifier
                    .fillMaxSize()
                    .padding(5.dp)
                    .weight(1f)
            ) {
                PhotoDetails(
                    loadedPhoto = loadedPhoto!!,
                    windowSize = WindowSize.Expanded
                )
            }
        }
    }
}
```

Frissítsük a Screenünket, hogy használja a fenti Composable-öket:

`PhotosFeedScreen.kt`:
```kotlin
@ExperimentalMaterial3Api
@ExperimentalMaterialApi
@ExperimentalPagingApi
@Composable
fun PhotosFeedScreen(
    modifier: Modifier = Modifier,
    windowSize: WindowSize = WindowSize.Compact,
    onPhotoItemClick: (String) -> Unit = {},
    viewModel: PhotosFeedViewModel = viewModel(factory = PhotosFeedViewModel.Factory)
) {
    
    val state by viewModel.state.collectAsState()

    val photos = state.photos?.collectAsLazyPagingItems()
    val selectedPhoto = state.photo?.collectAsState(null)

    val configuration = LocalConfiguration.current
    val screenWidth = configuration.screenWidthDp.dp

    val refreshState = rememberPullRefreshState(
        refreshing = state.isLoading,
        onRefresh = viewModel::refreshPhotos
    )

    if (photos != null) {
        when(windowSize) {
            WindowSize.Compact -> {
                PhotosFeedScreen_Compact(
                    refreshState = refreshState,
                    refreshing = state.isLoading,
                    onPhotoItemClick = onPhotoItemClick,
                    photos = photos,
                    value = state.searchTerms,
                    onValueChange = viewModel::onSearchTermsChange
                )
            }
            WindowSize.Medium -> {
                PhotosFeedScreen_Medium(
                    refreshState = refreshState,
                    refreshing = state.isLoading,
                    onPhotoItemClick = onPhotoItemClick,
                    photos = photos,
                    value = state.searchTerms,
                    onValueChange = viewModel::onSearchTermsChange
                )
            }
            WindowSize.Expanded -> {
                PhotosFeedScreen_Expanded(
                    onPhotoItemClick = if (screenWidth >= 1000.dp) {
                            viewModel::loadSelectedPhoto
                    } else onPhotoItemClick,
                    photos = photos,
                    loadedPhoto = selectedPhoto?.value,
                    value = state.searchTerms,
                    onValueChange = viewModel::onSearchTermsChange
                )
            }
        }
    } else if (state.isError) {
        Box(modifier = modifier.fillMaxSize()) {
            Text(text = state.throwable?.message.toString())
        }
    }
}
```

A `PhotoDetails.kt`-ban szüntessük meg a három windowsize-ot tartalmazó sor kommentezését.
	
Frissítsük a NavGraph-ot:
`NavGraph.kt`:
```kotlin
@ExperimentalMaterial3Api
@ExperimentalPagingApi
@ExperimentalMaterialApi
@Composable
fun NavGraph(
    windowSize: WindowSize = WindowSize.Compact,
) {
    val navController = rememberNavController()

    NavHost(
        navController = navController,
        startDestination = Screen.PhotosFeed.route
    ) {
        composable(route = Screen.PhotosFeed.route) {
            PhotosFeedScreen(
                windowSize = windowSize,
                onPhotoItemClick = { photoId ->
                    navController.navigate(Screen.LoadedPhoto.passPhotoId(photoId))
                }
            )
        }
        composable(
            route = Screen.LoadedPhoto.route,
            arguments = listOf(
                navArgument("photoId") {
                    type = NavType.StringType
                }
            )
        ) {
            LoadedPhotoScreen()
        }
    }
}
```

Majd az Activity-t is állítsuk be ezek használatára:

`MainActivity.kt`:
```kotlin
class MainActivity : ComponentActivity() {
    @OptIn(
        ExperimentalMaterial3WindowSizeClassApi::class,
        ExperimentalMaterial3Api::class,
        ExperimentalPagingApi::class,
        ExperimentalMaterialApi::class
    )
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            UnsplashTheme {
                val windowSize = when (calculateWindowSizeClass(this).widthSizeClass) {
                    WindowWidthSizeClass.Compact -> WindowSize.Compact
                    WindowWidthSizeClass.Medium -> WindowSize.Medium
                    WindowWidthSizeClass.Expanded -> WindowSize.Expanded
                    else -> WindowSize.Compact
                }

                NavGraph(windowSize = windowSize)
            }
        }
    }
}
```
Alt+Enterrel vegyük fel az annotációra kattintva a hiányző függőséget.

!!!example "BEADANDÓ (1 pont)" 
	Készíts egy **képernyőképet**, amelyen látszik a **működő alkalmazás a más méretben megjelenő képekkel** (emulátoron, készüléket tükrözve vagy képernyőfelvétellel),  a **más méretű képernyőkhöz tartozó kódrészlet**, valamint a **neptun kódod a kódban valahol kommentként**. 

	A képet a megoldásban a repository-ba f4.png néven töltsd föl.

	A képernyőkép szükséges feltétele a pontszám megszerzésének.

	
## Önálló feladat 2 - Dependency Injection

Írjuk át a projektet úgy, hogy az előző laboron megismert Dependency Injection keretrendszereket használja!

!!!example "BEADANDÓ (1 pont)" 
	Készíts egy **képernyőképet**, amelyen látszik a **működő alkalmazás** (emulátoron, készüléket tükrözve vagy képernyőfelvétellel),  a **dependency injectionjöz tartozó releváns kódrészlet**, valamint a **neptun kódod a kódban valahol kommentként**. 

	A képet a megoldásban a repository-ba f5.png néven töltsd föl.

	A képernyőkép szükséges feltétele a pontszám megszerzésének.
