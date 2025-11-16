# Labor09 - Network és Paging (Unsplash)

## Bevezetés

Ezen a laboron megismerkedünk Android a hálózati hívások elvégzésének egyik legelterjedtebb megvalósításával (Retrofit), a Paging Library-vel, illetve azzal, hogy a Compose keretrendszerben hogyan tudunk különböző képernyőméreteket támogatni.

<p style="text-align: center">
    <img src="./assets/phone_feed.png" width="300">
    <img src="./assets/phone_details.png" width="300">
</p>
<p style="text-align: center">
    <img src="./assets/tablet_feed_and_details.png" width="650">
</p>

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

    A [Retrofit](https://square.github.io/retrofit/) egy általános célú HTTP könyvtár Java és Kotlin környezetben. Széles körben használják, számos projektben bizonyított már (kvázi ipari standard). Azért használjuk, hogy ne kelljen alacsony szintű hálózati hívásokat implementálni.

    Segítségével elég egy interface-ben annotációk segítségével leírni az API-t (ez pl. a [Swagger](https://swagger.io/) eszközzel generálható is), majd e mögé készít a Retrofit egy olyan osztályt, mely a szükséges hálózati hívásokat elvégzi. A Retrofit a háttérben az [OkHttp3](https://github.com/square/okhttp)-at használja, valamint az objektumok JSON formátumba történő sorosítását a [Moshi](https://github.com/square/moshi) library-vel végzi. Ezért ezeket is be kell hivatkozni.

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

    A Coil (Coroutine Image Loader) egy kép betöltő könyvtár Androidra, amelyet a Kotlin coroutinokra épül.   
    A Coil használatának előnyei:  

    - Gyors: A Coil számos optimalizálást végez, beleértve a memória- és lemeztároló gyorsítótárazást, az átméretezést a memóriában, az automatikus kérések szüneteltetését/leállítását és még sok mást.  
    - Könnyű: A Coil kb. 2000 metódust ad az APK-hoz (azoknak az alkalmazásoknak, amelyek már használják az OkHttp és a Coroutines könyvtárakat), ami hasonló a Picasso-hoz és jelentősen kevesebb, mint a Glide és a Fresco könyvtárak.  
    - Könnyen használható: A Coil API-ja a Kotlin nyelv funkcióit használja a könnyű használat és a minimális boilerplate kód érdekében.  
    - Modern: A Coil a Kotlin nyelv sajátosságait használja ki és modern könyvtárakon alapszik, beleértve a Coroutines-t, az OkHttp-t, az Okio-t és az AndroidX Lifecycles-t.  

### Függőségek

Tekintsük át a kiinduló projektben lévő függőségeket!

### Modellek és adatbázis

A `data.model` package-be hozzuk létre az alábbi két fájlt, melyek az API használatához szükségesek:

`UnsplashPhoto.kt`:
```kotlin
package hu.bme.aut.android.network.data.model

import androidx.room.Embedded
import androidx.room.Entity
import androidx.room.PrimaryKey
import com.squareup.moshi.Json

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
    @Json(name = "total_likes")
    val totalLikes: Int,
    @Json(name = "total_photos")
    val totalPhotos: Int,
    @Json(name = "profile_image") @Embedded
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

Az `@Embedded` annotációval megjelölt mezők osztályainak mezői közvetlenül hivatkozhatók az SQL lekérdezésekben.  

A `Moshi` automatikusan megoldja majd az egyes tagváltozók szerializálását, kivéve ott, ahol eltér a név. Ezt a `Json` annotációval jelezhetjük.

`SearchResult.kt`:
```kotlin
package hu.bme.aut.android.network.data.model

import com.squareup.moshi.Json

data class SearchResult(
    @Json(name = "results") val photos: List<UnsplashPhoto>
)
```

Hozzuk létre az első DAO-t a `data.local.dao` package-ben:

`UnsplashPhotoDao.kt`:
```kotlin
package hu.bme.aut.android.network.data.local.dao

import androidx.paging.PagingSource
import androidx.room.Dao
import androidx.room.Insert
import androidx.room.Query
import hu.bme.aut.android.network.data.model.UnsplashPhoto
import kotlinx.coroutines.flow.Flow

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

Majd a `data.local.database` package-ben hozzuk létre az adatbázist:

`UnsplashDatabase.kt`:
```kotlin
package hu.bme.aut.android.network.data.local.database

import androidx.room.Database
import androidx.room.RoomDatabase
import hu.bme.aut.android.network.data.local.dao.UnsplashPhotoDao
import hu.bme.aut.android.network.data.model.UnsplashPhoto

@Database(entities = [UnsplashPhoto::class], version = 1)
abstract class UnsplashDatabase : RoomDatabase() {
    abstract val photosDao: UnsplashPhotoDao
}
```

!!!example "BEADANDÓ (1 pont)" 
	Készíts egy **képernyőképet**, az **adatábzishoz tartozó kódrészlet**, valamint a **neptun kódod a kódban valahol kommentként**. 

	A képet a megoldásban a repository-ba f1.png néven töltsd föl.

	A képernyőkép szükséges feltétele a pontszám megszerzésének.

### A Retrofit interfész

Az Unsplash API három végpontját fogjuk használni a képek, illetve egy adott kép lekérésére, valamint a keresés végrehajtására.
Hozzuk létre a lenti interface-t a `data.remote.api` package-ben.

`UnsplashApi.kt`:
```kotlin
package hu.bme.aut.android.network.data.remote.api

import hu.bme.aut.android.network.data.model.SearchResult
import hu.bme.aut.android.network.data.model.UnsplashPhoto
import retrofit2.Response
import retrofit2.http.GET
import retrofit2.http.Path
import retrofit2.http.Query

interface UnsplashApi {

    @GET("/photos")
    suspend fun getPhotosFromEditorialFeed(
        @Query("page") page: Int = 1,
        @Query("per_page") perPage: Int = 10,
        @Query("client_id") clientId: String = CLIENT_ID,
    ): Response<List<UnsplashPhoto>>

    @GET("/photos/{id}")
    suspend fun getPhotoById(
        @Path("id") id: String,
        @Query("client_id") clientId: String = CLIENT_ID,
    ): Response<UnsplashPhoto>

    @GET("/search/photos")
    suspend fun getSearchResults(
        @Query("client_id") clientId: String = CLIENT_ID,
        @Query("page") page: Int = 1,
        @Query("per_page") perPage: Int = 10,
        @Query("query") searchTerms: String,
    ): Response<SearchResult>

    companion object {
        private const val CLIENT_ID: String = "ACCESS_KEY"
    }
}
```

A Retrofit annotációi segítségével egyszerűen tudjuk definiálni a kéréseinket.  
Cseréljük le az itt szereplő `ACCESS_KEY` stringet az Unsplashen regisztráció után elérhető saját kulcsunkra.
Az https://unsplash.com/oauth/applications oldalon regisztráció után egy új applikációt kell létrehozni, és azt megnyitva találhatjuk meg a kulcsot.

### PagingUtil

Hozzuk létre a `PagingUtil` osztályt az `util` package-ben a paging konfigurálására és placeholderek beállítására:

`PagingUtil.kt`:
```kotlin
package hu.bme.aut.android.network.util

import android.os.Parcel
import android.os.Parcelable
import androidx.compose.foundation.lazy.grid.LazyGridItemScope
import androidx.compose.foundation.lazy.grid.LazyGridScope
import androidx.compose.runtime.Composable
import androidx.paging.compose.LazyPagingItems

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

### A PagingSource osztály

Ezután határozzunk meg egy PagingSource implementációt, hogy az adatforrás azonosítható legyen. A PagingSource API osztálya tartalmazza a load() metódust, amelyet felül kell írni, hogy jelentse, hogyan lehet lapozott adatokat visszanyerni a megfelelő adatforrásból.

Hozzuk létre az alábbi osztályt a `data.paging` package-ben:

`SearchPagingSource.kt`:
```kotlin
package hu.bme.aut.android.network.data.paging

import android.util.Log
import androidx.paging.PagingSource
import androidx.paging.PagingState
import hu.bme.aut.android.network.data.model.UnsplashPhoto
import hu.bme.aut.android.network.data.remote.api.UnsplashApi
import hu.bme.aut.android.network.util.PagingUtil.INITIAL_PAGE_SIZE

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
  
A `PagingSource` implementációjának emellett tartalmaznia kell egy `getRefreshKey()` metódust, amely egy `PagingState` objektumot vár paraméterként, és visszaadja a kulcsot, amelyet át kell adni a `load()` metódusnak, amikor az adat frissítése vagy érvénytelenítése történik az első betöltés után. A Paging könyvtár automatikusan meghívja ezt a metódust az adat későbbi frissítésekor.

### Távoli kulcsok

Kezelnünk kell azt a helyzetet, amikor a helyi gyorsítótárban tárolt adatok elavultak a távoli adatforráshoz képest.

A távoli kulcsok lehetővé teszik, hogy információt menthessünk el a legutóbbi oldalról, amelyet a szerverről kértek le. Az alkalmazás felhasználhatja ezt az információt a következő betöltendő adatoldal azonosításához és kéréséhez.

A távoli kulcsok olyan kulcsok, amelyeket a RemoteMediator implementáció arra használ, hogy közölje a backend szolgáltatással, melyik adatot kell legközelebb betölteni. A legegyszerűbb esetben minden lapozott adatelemhez tartozik egy távoli kulcs, amelyre könnyen hivatkozhat. Azonban ha a távoli kulcsok nem feleltethetőek meg a konkrét elemeknek, nekünk kell őket kezelni a load hívásban.

Amikor a távoli kulcsok nem közvetlenül kapcsolódnak a listaelemekhez, célszerű őket külön táblázatban tárolni a helyi adatbázisban. Definiálni kell egy Room entitást, amely egy távoli kulcsokból álló táblázatot reprezentál.

Hozzuk létre az alábbi osztályt a `data.model` package-ben:

`UnsplashPhotoRemoteKeys.kt`:
```kotlin
package hu.bme.aut.android.network.data.model

import androidx.room.Entity
import androidx.room.PrimaryKey

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
package hu.bme.aut.android.network.data.local.dao

import androidx.room.Dao
import androidx.room.Insert
import androidx.room.OnConflictStrategy
import androidx.room.Query
import hu.bme.aut.android.network.data.model.UnsplashPhotoRemoteKeys

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

Ezeket az osztályokat az adatbázisunkba is vegyük fel. Ne felejtsük el az adatbázis verziószámát növelni!

`UnsplashDatabase.kt`:
```kotlin
package hu.bme.aut.android.network.data.local.database

import androidx.room.Database
import androidx.room.RoomDatabase
import hu.bme.aut.android.network.data.local.dao.UnsplashPhotoDao
import hu.bme.aut.android.network.data.local.dao.UnsplashPhotoRemoteKeysDao
import hu.bme.aut.android.network.data.model.UnsplashPhoto
import hu.bme.aut.android.network.data.model.UnsplashPhotoRemoteKeys

@Database(entities = [UnsplashPhoto::class, UnsplashPhotoRemoteKeys::class], version = 2)
abstract class UnsplashDatabase : RoomDatabase() {
    abstract val photosDao: UnsplashPhotoDao
    abstract val remoteKeysDao: UnsplashPhotoRemoteKeysDao
}
```

### A RemoteMediator osztály

Jobb felhasználói élményt biztosíthatunk azzal, ha gondoskodunk arról, hogy az alkalmazásunk használható legyen akkor is, ha az internetkapcsolat instabil vagy ha a felhasználó offline. Az egyik módja ennek, hogy egyszerre lapozunk a hálózatról és egy helyi adatbázisból is. Ezzel az alkalmazás az UI-t egy helyi adatbázisból vezérli és csak akkor kér le adatokat a hálózatról, ha már nincs több adat az adatbázisban.

A Paging könyvtár a `RemoteMediator` komponenst biztosítja ehhez az esethez. A `RemoteMediator` a Paging könyvtár jelzéseként működik, amikor az alkalmazás kimerül az előre eltárolt adatokból. Ezt a jelzést lehet használni további adatok betöltésére a hálózatról, és azokat helyi adatbázisban tárolni, ahol egy `PagingSource` betöltheti és a felhasználói felületen megjelenítheti.

Ha további adatokra van szükség, a Paging könyvtár meghívja a `RemoteMediator` implementációjából a `load()` metódust. Ez egy suspending függvény, így hosszú ideig futó munkát végezhet biztonságosan. Ez a függvény az új adatokat általában egy hálózati forrásból szedi le, majd elmenti helyi tárolóba.

Ez a folyamat új adatokkal működik, de idővel az adatok tárolása az adatbázisban érvénytelenítést igényelhet, például amikor a felhasználó kézzel indítja el a frissítést. Ezt a `LoadType` tulajdonság jelzi, amelyet át kell adni a `load()` metódusnak. A `LoadType` tájékoztatja a `RemoteMediator`-t arról, hogy a meglévő adatokat frissíteni kell-e, vagy olyan további adatokat kell-e lekérni, amelyeket a meglévő listához kell hozzáadni.

Így a `RemoteMediator` biztosítja, hogy az alkalmazás azokat az adatokat töltse be, amelyeket a felhasználók a megfelelő sorrendben szeretnének látni.

Hozzuk létre az alábbi osztályt a `data.paging` package-ben:

`UnsplashRemoteMediator.kt`:
```kotlin
package hu.bme.aut.android.network.data.paging

import androidx.paging.ExperimentalPagingApi
import androidx.paging.LoadType
import androidx.paging.PagingState
import androidx.paging.RemoteMediator
import androidx.room.withTransaction
import coil3.network.HttpException
import hu.bme.aut.android.network.data.local.database.UnsplashDatabase
import hu.bme.aut.android.network.data.model.UnsplashPhoto
import hu.bme.aut.android.network.data.model.UnsplashPhotoRemoteKeys
import hu.bme.aut.android.network.data.remote.api.UnsplashApi
import hu.bme.aut.android.network.util.PagingUtil.INITIAL_PAGE
import java.io.IOException
import java.io.InvalidObjectException

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
                    val remoteKeys = getRemoteKeysForLastItem(state) ?: throw InvalidObjectException(
                        "Result is empty"
                    )
                    remoteKeys.nextKey ?: return MediatorResult.Success(endOfPaginationReached = true)
                }
            }

            val response = api.getPhotosFromEditorialFeed(page = page, perPage = 10)
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

A `data.repository` package-be vegyük fel a lenti `DataSource` osztályt, melyen keresztül a ViewModel eléri a kéréseinket:

`UnsplashPhotoDataSource.kt`:
```kotlin
package hu.bme.aut.android.network.data.repository

import androidx.paging.ExperimentalPagingApi
import androidx.paging.Pager
import androidx.paging.PagingConfig
import androidx.paging.PagingData
import hu.bme.aut.android.network.data.local.database.UnsplashDatabase
import hu.bme.aut.android.network.data.model.UnsplashPhoto
import hu.bme.aut.android.network.data.paging.SearchPagingSource
import hu.bme.aut.android.network.data.paging.UnsplashRemoteMediator
import hu.bme.aut.android.network.data.remote.api.UnsplashApi
import hu.bme.aut.android.network.util.PagingUtil.INITIAL_PAGE_SIZE
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.first
import kotlinx.coroutines.flow.flow

@ExperimentalPagingApi
class UnsplashPhotoDataSource(
    private val api: UnsplashApi,
    private val db: UnsplashDatabase
) {

    fun getAllPhotos(): Flow<PagingData<UnsplashPhoto>> {
        return Pager(
            config = PagingConfig(pageSize = INITIAL_PAGE_SIZE),
            remoteMediator = UnsplashRemoteMediator(api, db),
            pagingSourceFactory = { db.photosDao.getAllPhotos() }
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

Itt láthatjuk a `Pager` osztály használatát a lapozott adatok folyamának beállításához.
	
Végül hozzuk létre a saját Application osztályunkat a gyökér package-ben:

`UnsplashApplication.kt`:
```kotlin	
package hu.bme.aut.android.network

import android.app.Application
import androidx.paging.ExperimentalPagingApi
import androidx.room.Room
import com.squareup.moshi.Moshi
import com.squareup.moshi.kotlin.reflect.KotlinJsonAdapterFactory
import hu.bme.aut.android.network.data.local.database.UnsplashDatabase
import hu.bme.aut.android.network.data.remote.api.UnsplashApi
import hu.bme.aut.android.network.data.repository.UnsplashPhotoDataSource
import okhttp3.OkHttpClient
import retrofit2.Retrofit
import retrofit2.converter.moshi.MoshiConverterFactory
import java.util.concurrent.TimeUnit

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

        val moshi = Moshi.Builder().addLast(KotlinJsonAdapterFactory()).build()
        val retrofit = Retrofit.Builder()
            .baseUrl("https://api.unsplash.com/")
            .client(client)
            .addConverterFactory(MoshiConverterFactory.create(moshi))
            .build()

        val api = retrofit.create(UnsplashApi::class.java)

        photoDataSource = UnsplashPhotoDataSource(api, db)

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
Ezzel elsőként a `util` package-ben hozzunk létre egy osztályt a kategóriáinkra:

`WindowSize.kt`:
```kotlin
enum class WindowSize { Compact, Medium, Expanded }
```

Ezt követően hozzuk létre a maradék két elrendezésünket a `feature.photos_feed.screensbysize` package-ben:

`PhotosFeed_Compact.kt`:
```kotlin
package hu.bme.aut.android.network.feature.photos_feed.screensbysize

import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.material.ExperimentalMaterialApi
import androidx.compose.material.pullrefresh.PullRefreshIndicator
import androidx.compose.material.pullrefresh.PullRefreshState
import androidx.compose.material.pullrefresh.pullRefresh
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.material3.Scaffold
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.paging.compose.LazyPagingItems
import hu.bme.aut.android.network.ui.common.PhotoItem
import hu.bme.aut.android.network.ui.common.SearchTopAppBar
import hu.bme.aut.android.network.ui.model.UnsplashPhotoUiModel

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
    Scaffold(
        modifier = modifier,
        topBar = {
            SearchTopAppBar(
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
                    items(photos.itemCount) { index ->
                        photos[index]?.let { model ->
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

`PhotosFeed_Expanded.kt`:
```kotlin
package hu.bme.aut.android.network.feature.photos_feed.screensbysize

import androidx.compose.foundation.background
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.aspectRatio
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.size
import androidx.compose.foundation.layout.width
import androidx.compose.foundation.lazy.grid.GridCells
import androidx.compose.foundation.lazy.grid.LazyVerticalGrid
import androidx.compose.foundation.lazy.grid.rememberLazyGridState
import androidx.compose.foundation.shape.CircleShape
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material.ExperimentalMaterialApi
import androidx.compose.material.MaterialTheme
import androidx.compose.material.pullrefresh.PullRefreshIndicator
import androidx.compose.material.pullrefresh.PullRefreshState
import androidx.compose.material3.CircularProgressIndicator
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.saveable.rememberSaveable
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.layout.ContentScale
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.unit.dp
import androidx.compose.ui.window.Dialog
import androidx.paging.compose.LazyPagingItems
import coil3.compose.AsyncImage
import coil3.request.ImageRequest
import coil3.request.crossfade
import hu.bme.aut.android.network.R
import hu.bme.aut.android.network.ui.common.PhotoItem
import hu.bme.aut.android.network.ui.common.SearchTopAppBar
import hu.bme.aut.android.network.ui.model.UnsplashPhotoUiModel

@ExperimentalMaterial3Api
@ExperimentalMaterialApi
@Composable
fun PhotosFeedScreen_Expanded(
    refreshState: PullRefreshState,
    refreshing: Boolean,
    modifier: Modifier = Modifier,
    photo: UnsplashPhotoUiModel? = null,
    onPhotoItemClick: (String) -> Unit,
    photos: LazyPagingItems<UnsplashPhotoUiModel>,
    value: String,
    onValueChange: (String) -> Unit
) {
    val state = rememberLazyGridState()

    var isPhotoOpen by rememberSaveable { mutableStateOf(false) }

    Scaffold(
        modifier = Modifier.fillMaxSize(),
        topBar = {
            SearchTopAppBar(
                value = value,
                onValueChange = onValueChange
            )
        }
    ) {
        Box(
            modifier = modifier
                .fillMaxSize()
                .padding(it)
        ) {
            if(!refreshing) {
                LazyVerticalGrid(
                    state = state,
                    columns = GridCells.Adaptive(240.dp),
                    modifier = Modifier
                        .fillMaxSize()
                ) {
                    items(photos.itemCount) { index ->
                        photos[index]?.let { model ->
                            PhotoItem(
                                photo = model,
                                onClick = {
                                    onPhotoItemClick(model.id)
                                    isPhotoOpen = true
                                }
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
    if (isPhotoOpen) {
        Dialog(
            onDismissRequest = {
                isPhotoOpen = false
            }
        ) {
            photo?.let {
                val context = LocalContext.current
                Column (
                    modifier = Modifier
                        .fillMaxSize(),
                    horizontalAlignment = Alignment.CenterHorizontally,
                    verticalArrangement = Arrangement.Center
                ) {
                    AsyncImage(
                        model = ImageRequest.Builder(context)
                            .data(photo.photoUrl)
                            .crossfade(enable = true)
                            .build(),
                        contentDescription = null,
                        contentScale = ContentScale.Crop,
                        placeholder = painterResource(id = R.drawable.placeholder),
                        modifier = Modifier
                            .fillMaxWidth()
                            .aspectRatio(1f)
                    )
                    Row (
                        horizontalArrangement = Arrangement.Center,
                        verticalAlignment = Alignment.CenterVertically,
                        modifier = Modifier
                            .fillMaxWidth()
                            .padding(vertical = 8.dp)
                            .clip(RoundedCornerShape(8.dp))
                            .background(MaterialTheme.colors.surface)
                    ) {
                        AsyncImage(
                            model = ImageRequest.Builder(context)
                                .data(photo.userProfileImageUrl)
                                .crossfade(enable = true)
                                .build(),
                            contentDescription = null,
                            contentScale = ContentScale.Crop,
                            placeholder = painterResource(id = R.drawable.placeholder),
                            modifier = Modifier
                                .padding(8.dp)
                                .size(80.dp)
                                .clip(CircleShape)
                        )
                        Spacer(modifier = Modifier.width(10.dp))
                        Text(text = photo.username)
                    }
                }
            } ?: CircularProgressIndicator()
        }
    }
}
```

Frissítsük a PhotosFeedScreen-ünket, hogy használja a fenti Composable-öket:

`PhotosFeedScreen.kt`:
```kotlin
package hu.bme.aut.android.network.feature.photos_feed

import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material.ExperimentalMaterialApi
import androidx.compose.material.pullrefresh.rememberPullRefreshState
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.ui.Modifier
import androidx.lifecycle.compose.collectAsStateWithLifecycle
import androidx.lifecycle.viewmodel.compose.viewModel
import androidx.paging.ExperimentalPagingApi
import androidx.paging.compose.collectAsLazyPagingItems
import hu.bme.aut.android.network.feature.photos_feed.screensbysize.PhotosFeedScreen_Compact
import hu.bme.aut.android.network.feature.photos_feed.screensbysize.PhotosFeedScreen_Expanded
import hu.bme.aut.android.network.feature.photos_feed.screensbysize.PhotosFeedScreen_Medium
import hu.bme.aut.android.network.util.WindowSize

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
    val state by viewModel.state.collectAsStateWithLifecycle()

    val photos = state.photos?.collectAsLazyPagingItems()

    val selectedPhoto = state.photo?.collectAsStateWithLifecycle(null)

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
                    refreshState = refreshState,
                    refreshing = state.isLoading,
                    onPhotoItemClick = viewModel::loadSelectedPhoto,
                    photo = selectedPhoto?.value,
                    photos = photos,
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
	
Frissítsük az AppNavigation-t, hogy megkapjon egy WindowSize típusú objektumot és átadja a PhotosFeedScreen-nek:
`AppNavigation.kt`:
```kotlin
package hu.bme.aut.android.network.navigation

import androidx.compose.material.ExperimentalMaterialApi
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.navigation3.runtime.entryProvider
import androidx.navigation3.runtime.rememberNavBackStack
import androidx.navigation3.ui.NavDisplay
import androidx.paging.ExperimentalPagingApi
import hu.bme.aut.android.network.feature.loaded_photo.LoadedPhotoScreen
import hu.bme.aut.android.network.feature.photos_feed.PhotosFeedScreen
import hu.bme.aut.android.network.util.WindowSize

@OptIn(ExperimentalMaterial3Api::class, ExperimentalMaterialApi::class,
    ExperimentalPagingApi::class
)
@Composable
fun AppNavigation(
    modifier: Modifier = Modifier,
    windowSize: WindowSize = WindowSize.Compact,
) {
    val backStack = rememberNavBackStack(Screen.PhotosFeedDestination)

    NavDisplay(
        modifier = modifier,
        backStack = backStack,
        onBack = { backStack.removeLastOrNull() },
        entryProvider = entryProvider {

            entry<Screen.PhotosFeedDestination> {
                PhotosFeedScreen(
                    windowSize = windowSize,
                    onPhotoItemClick = {
                        backStack.add(Screen.LoadedPhotoDestination(photoId = it))
                    }
                )
            }

            entry<Screen.LoadedPhotoDestination> { key ->
                LoadedPhotoScreen(
                    photoId = key.photoId,
                    onNavigateBack = {
                        backStack.removeLastOrNull()
                    }
                )
            }
        }
    )
}
```
Vegyük fel a `material3` könyvtár mintájára a `material3-window-size-class` függőséget (link)[https://developer.android.com/jetpack/androidx/releases/compose-material3]:

```kotlin
implementation "androidx.compose.material3:material3-window-size-class:1.4.0"
```

Láthatjuk, hogy a dokumentációból kiemelt kódsor használatával megkerüljük a könyvtárnak és verziójának a libs.version.toml fájlban való definiálását, és az Android Studio sárga aláhúzással figyelmeztet is, hogy ehelyett kövessük az ajánlott módszert a könyvtár hozzáadásához. Szerencsére egy kattintásra, automatikusan átalakítja nekünk ezt a megfelelő módon, figyeljük meg a változásokat!

Majd az Activity-t is állítsuk be a különböző képernyőméretek használatára:

`MainActivity.kt`:
```kotlin
package hu.bme.aut.android.network

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.safeDrawingPadding
import androidx.compose.material.ExperimentalMaterialApi
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.material3.windowsizeclass.ExperimentalMaterial3WindowSizeClassApi
import androidx.compose.material3.windowsizeclass.WindowWidthSizeClass
import androidx.compose.material3.windowsizeclass.calculateWindowSizeClass
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.paging.ExperimentalPagingApi
import hu.bme.aut.android.network.navigation.AppNavigation
import hu.bme.aut.android.network.ui.theme.UnsplashTheme
import hu.bme.aut.android.network.util.WindowSize


class MainActivity : ComponentActivity() {
    @OptIn(
        ExperimentalMaterial3Api::class,
        ExperimentalPagingApi::class,
        ExperimentalMaterialApi::class,
        ExperimentalMaterial3WindowSizeClassApi::class
    )
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            val windowSize = when (calculateWindowSizeClass(this).widthSizeClass) {
                WindowWidthSizeClass.Compact -> WindowSize.Compact
                WindowWidthSizeClass.Medium -> WindowSize.Medium
                WindowWidthSizeClass.Expanded -> WindowSize.Expanded
                else -> WindowSize.Compact
            }
            MainActivityContent(windowSize = windowSize)
        }
    }
}

@Composable
fun MainActivityContent(windowSize: WindowSize) {
    UnsplashTheme {
        AppNavigation(
            modifier = Modifier.safeDrawingPadding(),
            windowSize = windowSize
        )
    }
}
```

!!!example "BEADANDÓ (1 pont)" 
	Készíts egy **képernyőképet**, amelyen látszik a **működő alkalmazás a más méretben megjelenő képekkel** (emulátoron, készüléket tükrözve vagy képernyőfelvétellel),  a **más méretű képernyőkhöz tartozó kódrészlet**, valamint a **neptun kódod a kódban valahol kommentként**. 

	A képet a megoldásban a repository-ba f4.png néven töltsd föl.

	A képernyőkép szükséges feltétele a pontszám megszerzésének.

	
## Önálló feladat 2 - Dependency Injection

Írjuk át a projektet úgy, hogy az előző laboron megismert `Dependency Injection` keretrendszereket használja!

!!!example "BEADANDÓ (1 pont)" 
	Készíts egy **képernyőképet**, amelyen látszik a **működő alkalmazás** (emulátoron, készüléket tükrözve vagy képernyőfelvétellel),  a **dependency injectionjöz tartozó releváns kódrészlet**, valamint a **neptun kódod a kódban valahol kommentként**. 

	A képet a megoldásban a repository-ba f5.png néven töltsd föl.

	A képernyőkép szükséges feltétele a pontszám megszerzésének.
