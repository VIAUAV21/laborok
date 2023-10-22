
# Labor07 - Firebase

## Bevezető

A labor során a meglévő feladatkezelő alkalmazás kerül továbbfejlesztésre a Firebase Backend as a Service (BaaS) felhasználásával. A feladat célja, hogy szemléltesse, hogyan lehet közös backendet használó alkalmazást fejleszteni saját backend kód fejlesztése nélkül.

A Firebase manapság az egyik legnépszerűbb Backend as a Service megoldás Android, iOS és web kliensek támogatásával, mely számos szolgáltatást biztosít, például:
- real-time adatbáziskezelés
- storage
- authentikáció
- push értesítések
- analytics
- crash reporting

További általános információk a Firebase-ről: [https://firebase.google.com/](https://firebase.google.com/).

A laborfoglalkozás célja, hogy bemutassa a Firebase legfontosabb szolgáltatásait egy komplett alkalmazás megvalósítása keretében. 
Az alkalmazás az alábbi fő funkciókat támogatja:
- regisztráció, bejelentkezés
- üzenetek listázása
- üzenetírás
- képek csatolása üzenetekhez
- üzenetek megjelenítése valós időben
- crash reporting
- analitika

**Az anyag részletes megértéséhez javasoljuk, hogy figyelje a laborvezető utasításait és labor után is 10-20 percet szánjon a kódrészek megértésére.**

## Előkészületek

A feladatok megoldása során ne felejtsd el követni a [feladat beadás folyamatát](../../tudnivalok/github/GitHub.md).

### Git repository létrehozása és letöltése

1. Moodle-ben keresd meg a laborhoz tartozó meghívó URL-jét és annak segítségével hozd létre a saját repository-dat.

2. Várd meg, míg elkészül a repository, majd checkout-old ki.

    !!! tip ""
        Egyetemi laborokban, ha a checkout során nem kér a rendszer felhasználónevet és jelszót, és nem sikerül a checkout, akkor valószínűleg a gépen korábban megjegyzett felhasználónévvel próbálkozott a rendszer. Először töröld ki a mentett belépési adatokat (lásd [itt](../../tudnivalok/github/GitHub-credentials.md)), és próbáld újra.

3. Hozz létre egy új ágat `megoldas` néven, és ezen az ágon dolgozz.

4. A `neptun.txt` fájlba írd bele a Neptun kódodat. A fájlban semmi más ne szerepeljen, csak egyetlen sorban a Neptun kód 6 karaktere.

A Firebase megismeréséhez ebben a laborban egy előre elkészített projektbe fogjuk integrálni a különböző szolgáltatásokat. Ez megtalálható a repository-n belül is, valamilyen probléma esetén a kezdőprojektet [erről a linkről érhető el](./assets/Todo_firebase_starter.zip).

Ezután indítsuk el az Android Studio-t, majd nyissuk meg a kicsomagolt projektet.

!!!danger "FILE PATH"
	A projekt a repository-ban lévő `Todo` könyvtárba kerüljön, és beadásnál legyen is felpusholva! A kód nélkül nem tudunk maximális pontot adni a laborra!

Ellenőrízzük, hogy a létrejött projekt lefordul és helyesen működik!

## Projekt előkészítése, konfiguráció

Első lépésként létre kell hozni egy Firebase projektet a Firebase admin felületén (Firebase console), majd egy Android Studio projektet és a kettőt össze kell kötni:
- Navigáljunk a Firebase console felületére: [https://console.firebase.google.com/](https://console.firebase.google.com/) !
- Jelentkezzünk be!
- Hozzunk létre egy új projektet a *Create project* elemet választva!

<p align="center">
<img src="./assets/firebase_create_project_1.png">
</p>

<p align="center">
<img src="./assets/firebase_create_project_3.png">
</p>

- A projekt neve legyen *BMETodoNEPTUN_KOD*, ahol a `NEPTUN_KOD` helyére a saját Neptun kódunkat helyettesítsük!
- Az analitikát most még nem szükséges konfigurálni.

!!!danger "projekt név"
	A Neptun kódra azért van szükség, mert ugyanazon laborgép kulcsával ugyanolyan nevű projektet nem hozhatunk létre többször, és több laborcsoport lévén ebből probléma adódhatna. Ugyanerre lesz majd szükség a package név esetén is.

Sikeres projekt létrehozás után fussák át a laborvezetővel közösen a Firebase console felületét az alábbi elemekre kitérve:
- Authentication, Firestore és Storage.

Nézzük át a megnyitott projektet! Különös figyelemmel vizsgáljuk át a projekt mostani felépítését, az új részeket (todo_auth, data package), illetve hogyan lehetséges ebben átállni a Firebase szolgáltatásaira.

Adjuk hozzá az `AndroidManifest.xml` fájlhoz az internet használati engedélyt:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

## Firebase inicilaizáció, Authentication

Ezek után válasszuk Android Studioban a *Tools -> Firebase* menüpontot, melynek hatására jobb oldalt megnyílik a *Firebase Assistant* funkció.

A *Firebase Assistant* akkor fogja megtalálni a Firebase console-on létrehozott projektet, ha Android Studioba is ugyanazzal a Google accounttal vagyunk bejelentkezve, mint amivel a console-on létrehoztuk a projektet. Ellenőrizzük ezt mindkét helyen! Amennyiben a *Firebase Assistant*-ot nem sikerül beüzemelni, manuálisan is összeköthető a két projekt. A leírásban ismertetni fogjuk a lépéseket, amelyeket az *Assistant* végez el.

Válasszuk az *Assistant*-ban az *Authentication* szakaszt és azon belül az *Authenticate using a custom authentication system*-t, majd a *Connect to Firebase* gombot.
Ezt követően egy dialog nyílik meg, ahol ha megfelelőek az accountok, a második szakaszt (*Choose an existing Firebase or Google project*) választva kiválaszthatjuk a projektet, amit a Firebase console-on már létrehoztunk. Itt egyébként lehetőség van új projektet is létrehozni. (Ha elsőre hibát látunk a projekttel való összekapcsolásnál, próbáljuk újra, másodszorra általában sikeresen megtörténik az Android Studio projekt szinkronizálása a Firebase projekttel.)

!!!info ""
	A háttérben valójában annyi történik, hogy az alkalmazásunk package neve és az aláíró kulcs *SHA-1 hash-e* alapján hozzáadódik egy Android alkalmazás a Firebase console-on lévő projektünkhöz, és az ahhoz tartozó konfigurációs (`google-services.json`) fájl letöltődik a projektünk könyvtárába az alapértelmezett (`app`) modul alá.

Ezt a lépéssorozatot manuálisan is végrehajthatjuk a Firebase console-on az *Add Firebase to your Android app*-et választva. A debug kulcs *SHA-1* lenyomata ilyenkor a jobb oldalon található Gradle fülön a *Gradle -> [projektnév] -> Tasks -> android -> signingReport* taskot futtatva kinyerhető alul az *execution/text* módot választva.

<p align="center">
<img src="./assets/android_studio_signingreport.png">
</p>

Következő lépésben szintén az *Assistant*-ban az *Authenticate using a custom authentication system* alatt válasszuk az *Add the Firebase Authentication SDK to your app* elemet, itt látható is, hogy milyen módosítások történnek a projekt és modul szintű `build.gradle` fájlokban.

<p align="center">
<img src="./assets/firebase_auth_connect.png">
</p>


Sajnos a Firebase plugin nincs rendszeresen frissítve, és így előfordul, hogy a függőségek régi verzióját adja hozzá a `build.gradle` fájlokhoz. Ezért most frissíteni fogjuk az imént automatikusan felvett függőségeket, valamint innentől manuálisan fogjuk hozzáadni az újabbakat az *Assistant* használata helyett. Fontos, hogy mindenből az itt leírt verziót használjuk.

Ellenőrizzük a projekt szintű `build.gradle` fájlban a `google-services`-t, hogy az alábbi verzióval rendelkezik:

```groovy
classpath 'com.google.gms:google-services:4.4.0'
```

A Firebase BoM segítségével egységesen tudjuk kezelni az összes firebase könyvtárunk verziószámát.
Cseréljük le a modul szintű `build.gradle`-ben a `firebase-auth` verziót a következőre:
```groovy
implementation(platform("com.google.firebase:firebase-bom:32.4.0"))
implementation("com.google.firebase:firebase-auth-ktx")
```

A generált projektváz többi általános függősége (pl. appcompat és ktx-core könyvtárak) is elavult lehet, ezt az Android Studio jelzi is sötétsárga háttérrel. Ezekre ráállva a kurzorral az Alt-Enter gyorsbillenytűvel kiválaszthatjuk ezeknek a frissítését.

Ahhoz, hogy az e-mail alapú regisztráció és authentikáció megfelelően működjön, a *Firebase console*-ban az *Authentication -> Sign-in method* alatt az *Email/Password* providert engedélyezni kell.

<p align="center">
<img src="./assets/firebase_console_auth_method.png">
</p>

Készítsük el a megfelelő `Service` osztályt. Hozzunk létre a `data/auth` package-en belül a `FirebaseAuthService` osztályt! Valósítsuk meg az `AuthService` interfész egyes metódusait! Ehhez szükségünk lesz egy `FirebaseAuth` objektumra, melyet külső forrásból fogunk megkapni:

```kotlin
package hu.bme.aut.android.todo.data.auth  
  
import com.google.firebase.auth.FirebaseAuth  
import com.google.firebase.auth.UserProfileChangeRequest  
import hu.bme.aut.android.todo.domain.model.User  
import kotlinx.coroutines.channels.awaitClose  
import kotlinx.coroutines.flow.Flow  
import kotlinx.coroutines.flow.callbackFlow  
import kotlinx.coroutines.tasks.await

class FirebaseAuthService(private val firebaseAuth: FirebaseAuth) : AuthService {
    override val currentUserId: String? get() = firebaseAuth.currentUser?.uid
    override val hasUser: Boolean get() = firebaseAuth.currentUser != null
    override val currentUser: Flow<User?> get() = callbackFlow {
        this.trySend(currentUserId?.let { User(it) })
        val listener =
            FirebaseAuth.AuthStateListener { auth ->
                this.trySend(auth.currentUser?.let { User(it.uid) })
            }
        firebaseAuth.addAuthStateListener(listener)
        awaitClose { firebaseAuth.removeAuthStateListener(listener) }
    }


    override suspend fun signUp(email: String, password: String) {
        firebaseAuth.createUserWithEmailAndPassword(email,password)
            .addOnSuccessListener {  result ->
                val user = result.user
                val profileChangeRequest = UserProfileChangeRequest.Builder()
                    .setDisplayName(user?.email?.substringBefore('@'))
                    .build()
                user?.updateProfile(profileChangeRequest)
            }.await()
    }

    override suspend fun authenticate(email: String, password: String) {
        firebaseAuth.signInWithEmailAndPassword(email, password).await()
    }

    override suspend fun sendRecoveryEmail(email: String) {
        firebaseAuth.sendPasswordResetEmail(email).await()
    }

    override suspend fun deleteAccount() {
        firebaseAuth.currentUser!!.delete().await()
    }

    override suspend fun signOut() {
        firebaseAuth.signOut()
    }
}
```
Nézzük át, hogyan tudtuk az általunk definiált `AuthService` interfészhez illeszteni a Firebase által biztosított API-t!

Sokszor a biztosított API közvetlenül megfeleltethető az általunk definiált szolgáltatásokkal, mint például a `currentUser` vagy `hasUser` mezőknél. Itt egyedül arra kell figyelnünk, hogy ne maradjon le a `get()` definíció, akkor ugyanis a `Service` létrejöttekor történne egy értékadás, nem pedig minden egyes kiolvasásnál egy függvényhívás.

Ha szerencsénk van, akkor a felhasznált API beépítetten támogatni fogja a Kotlin különböző funkcionalitásait, mint például a Kotlinos típusok, contract-ok és coroutine-ok. Sokszor viszont Java alapú könyvtárakat kapunk, amiket nekünk kell adaptálni Kotlin környezetre. Erre egy jó példa a `currentUser` mező. Ez a `callbackFlow` metódust használja, melynek segítségével a callback jellegű API-t tudjuk átalakítani `Flow` jellegű eredménnyé. A blokkon belül egy listenert regisztrálunk be, mellyel meg tudjuk figyelni, ha változás történik az aktuális felhasználók körében. Ekkor a `trySend()` metódussal tudjuk a Flow-ra feliratkozó felé elküldeni az új felhasználó adatait. Érdemes arra is figyelni, hogy ez a listener csak a feliratkozás után kezd el értékeket kiküldeni. Annak érdekében, hogy a UI egyből megkapja az aktuális értéket, a feliratkozás előtt kiküldjük az aktuális felhasználó adatait is.

A többi utasításnál a Firebase egy `Task` típusú eredmény objektummal tér vissza. A Java világában erre fel tudunk iratkozni, és az eredményét egy Callback-ben le tudjuk kezelni. Szerencsére azonban a Firebase általános könytvárában megtalálható egy Kotlin kiegészítő metódus, mellyel a `Task` bevárható coroutine kontextusban az `await()` kulcsszóval. A teljesség kedvéért nézzük meg az alábbi példát, hogyan lehet egy Callback jellegű API-t átalakítani `suspend` fajtájú függvénnyé:

```kotlin
override suspend fun authenticate(email: String, password: String) = suspendCoroutine { continuation ->
	firebaseAuth
		.signInWithEmailAndPassword(email, password)
		.addOnSuccessListener { continuation.resume(Unit) }
		.addOnFailureListener { continuation.resumeWithException(it) }
}
```

A `suspendCoroutine` metódus le fogja futtatni a benne megadott blokkot, majd addig várakoztatja a coroutine-t, ameddig a blokkban megkapott `Continuation` objektumon keresztül nem jelezzük a hívás végeredményét. Ezzel a függvénnyel könnyedén át tudjuk alakítani a `Callback` jellegű működéseket `suspend` alapúra. Arra azonban figyeljünk, hogy minden esetben meghívódjon a `Continuation` valamelyik `resume` metódusa, ellenkező esetben ugyanis befagy az adott coroutine, sose fog tudni továbblépni. Hasonlóan hasznos függvény a `suspendCancellableCoroutine`, mellyel azokat az eseteket is le tudjuk kezelni, ha a coroutine-t a folyamat közben törlik.

Állítsuk át az alkalmazásunkat, hogy ezt az új `FirebaseAuthService`-t használja! Ehhez módosítsuk a `TodoApplication` osztályunkat:

```kotlin
package hu.bme.aut.android.todo  
  
import android.app.Application  
import com.google.firebase.auth.FirebaseAuth  
import hu.bme.aut.android.todo.data.auth.AuthService  
import hu.bme.aut.android.todo.data.auth.FirebaseAuthService  
import hu.bme.aut.android.todo.data.todos.MemoryTodoService  
import hu.bme.aut.android.todo.data.todos.TodoService

class TodoApplication : Application(){
    override fun onCreate() {
        super.onCreate()
        authService = FirebaseAuthService(FirebaseAuth.getInstance())
        todoService = MemoryTodoService()
    }

    companion object{
        lateinit var authService: AuthService
        lateinit var todoService: TodoService
    }
}
```

Próbáljuk ki az alkalmazást! Hozzunk létre egy új felhasználót!

!!!example "BEADANDÓ (1 pont)" 
	Készíts egy képernyő képet, amin látszódik Firebase Authentication oldalán a beregisztrált felhasználó, illetve a `FirebaseAuthService` forráskódja, melyben a Neptun-kód komment formájában látható. A képernyőkép szükséges feltétele a pontszám megszerzésének.

## Feladatok listázása, készítése

Következő lépésben a feladatok listázását fogjuk implementálni a projekten belül.
 
Adjuk hozzá a projekthez a *Cloud Firestore* támogatást.

```groovy
implementation("com.google.firebase:firebase-firestore-ktx")
```

Kapcsoljuk be a *Cloud Firestore*-t a *Firebase console*-on is . Az adatbázist *test mode*-ban fogjuk használni, így egyelőre publikusan írható/olvasható lesz, de cserébe nem kell konfigurálnunk a hozzáférés-szabályozást. Ezt természetesen később mindenképp meg kellene tenni egy éles projektben.

<p align="center">
<img src="./assets/firebase_create_firestore.png">
</p>

Locationnek válasszunk egy hozzánk közel eső opciót.

Hozzuk létre a `todos` package-en belül a `firebase` package-et. Ebben két osztályt fogunk definiálni: a Firestore-ban tárolt adatobjektum osztály modelljét, illetve a kommunikációt megvalósító service kódját.

Hozzuk először létre az adatot reprezentáló osztályt `FirebaseTodo` néven:
```kotlin
package hu.bme.aut.android.todo.data.todos.firebase

import com.google.firebase.Timestamp
import com.google.firebase.firestore.DocumentId
import hu.bme.aut.android.todo.domain.model.Priority
import hu.bme.aut.android.todo.domain.model.Todo
import kotlinx.datetime.*
import java.time.Instant
import java.time.LocalDateTime
import java.time.ZoneId
import java.util.Date

data class FirebaseTodo(
    @DocumentId val id: String = "",
    val title: String = "",
    val priority: Priority = Priority.NONE,
    val dueDate: Timestamp = Timestamp.now(),
    val description: String = ""
)

fun FirebaseTodo.asTodo() = Todo(
    id = id,
    title = title,
    priority = priority,
    dueDate = LocalDateTime
        .ofInstant(Instant.ofEpochSecond(dueDate.seconds), ZoneId.systemDefault())
        .toKotlinLocalDateTime()
        .date,
    description = description,
)

fun Todo.asFirebaseTodo() = FirebaseTodo(
    id = id,
    title = title,
    priority = priority,
    dueDate = Timestamp(Date.from(dueDate.atStartOfDayIn(TimeZone.currentSystemDefault()).toJavaInstant())),
    description = description,
)
```
Ebben a fájlban definiáltuk a két átalakító függvényt is, mellyel a Firebase és az alkalmazás többi részében használt Todo osztály között tudunk átalakítani. Az egyedüli bonyolult rész a Firebase által használt `Timestamp` osztály használata az időpont eltárolására, erre most részletesen nem térünk ki.

Hozzuk létre a feladatok tárolását végző `FirebaseTodoService` osztályt is ebben a package-ben:
```kotlin
package hu.bme.aut.android.todo.data.todos.firebase

import com.google.firebase.firestore.FirebaseFirestore
import com.google.firebase.firestore.ktx.snapshots
import com.google.firebase.firestore.ktx.toObjects
import com.google.firebase.firestore.ktx.toObject
import hu.bme.aut.android.todo.data.auth.AuthService
import hu.bme.aut.android.todo.data.todos.TodoService
import hu.bme.aut.android.todo.domain.model.Todo
import kotlinx.coroutines.flow.*
import kotlinx.coroutines.tasks.await

class FirebaseTodoService(
    private val firestore: FirebaseFirestore,
    private val authService: AuthService
) : TodoService {

    override val todos: Flow<List<Todo>> = authService.currentUser.flatMapLatest { user ->
        if (user == null) flow { emit(emptyList()) }
        else currentCollection(user.id)
            .snapshots()
            .map { snapshot ->
                snapshot
                    .toObjects<FirebaseTodo>()
                    .map {
                        it.asTodo()
                    }
            }
    }

    override suspend fun getTodo(id: String): Todo? =
        authService.currentUserId?.let {
            currentCollection(it).document(id).get().await().toObject<FirebaseTodo>()?.asTodo()
        }

    override suspend fun saveTodo(todo: Todo) {
        authService.currentUserId?.let {
            currentCollection(it).add(todo.asFirebaseTodo()).await()
        }
    }

    override suspend fun updateTodo(todo: Todo) {
        authService.currentUserId?.let {
            currentCollection(it).document(todo.id).set(todo.asFirebaseTodo()).await()
        }
    }

    override suspend fun deleteTodo(id: String) {
        authService.currentUserId?.let {
            currentCollection(it).document(id).delete().await()
        }
    }

    private fun currentCollection(userId: String) =
        firestore.collection(USER_COLLECTION).document(userId).collection(TODO_COLLECTION)

    companion object {
        private const val USER_COLLECTION = "users"
        private const val TODO_COLLECTION = "todos"
    }
}
```
Végül ne felejtsük el befrissíteni a `TodoApplication` osztályunkat, hogy a Firestoreban tárolt feladatokat használja az alkalmazás:

```kotlin
package hu.bme.aut.android.todo

import android.app.Application
import com.google.firebase.auth.FirebaseAuth
import com.google.firebase.firestore.FirebaseFirestore
import hu.bme.aut.android.todo.data.auth.AuthService
import hu.bme.aut.android.todo.data.auth.FirebaseAuthService
import hu.bme.aut.android.todo.data.todos.TodoService
import hu.bme.aut.android.todo.data.todos.firebase.FirebaseTodoService

class TodoApplication : Application(){
    override fun onCreate() {
        super.onCreate()
        authService = FirebaseAuthService(FirebaseAuth.getInstance())
        todoService = FirebaseTodoService(FirebaseFirestore.getInstance(), authService)
    }

    companion object{
        lateinit var authService: AuthService
        lateinit var todoService: TodoService
    }
}
```

Próbáljuk ki az alkalmazásunkat! Ellenőrizzük, hogy tényleg létrejönnek az adatbázisban is a feladatok.

!!!example "BEADANDÓ (1 pont)" 
	Készíts egy képernyő képet, amin látszódik Firebase Firestore oldalán a létrehozott feladat, illetve a futó alkalmazás, melyben az egyik létrehozott feladat tartalmazza a Neptun-kódot. A képernyőkép szükséges feltétele a pontszám megszerzésének.

!!!warning "Messaging, Crashlytics, Analytics"
	A kövezkező technológiák átfutási ideje sajnos hosszabb, így az eredményre nem ritkán órákat is várni kell. (A notificationnek néhány perc alatt meg kell jönnie.)
	
## Push értesítések

Adjuk hozzá a projektünkhöz a `firebase-messaging` függőséget:

```groovy
implementation("com.google.firebase:firebase-messaging-ktx")
```

Csupán ennyi elegendő a push alapvető működéséhez, ha így újrafordítjuk az alkalmazást, a Firebase felületéről vagy API-jával küldött push üzeneteket automatikusan megkapják a mobil kliensek és egy *Notification*-ben megjelenítik.

Próbáljuk ki a push küldést a *Firebase console*-ról (*Cloud messaging menüpont* alatt *Send your first message*), és vizsgáljuk meg, hogyan érkezik meg telefonra, **ha nem fut az alkalmazás**. (Amikor fut az alkalmazás, akkor tőlünk várja az üzenet lekezelését az API.)
A `Notification` szekció alatt írjuk be az üzenet címét és szövegét, a `Target` résznél pedig
válasszuk ki az alkalmazást, hogy minden futó példány megkapja az üzenetet.

<p align="center">
<img src="./assets/firebase_push_1.png">
</p>

<p align="center">
<img src="./assets/firebase_push_2.png">
</p>

<p align="center">
<img src="./assets/firebase_push_notification_success.png" width="320">
</p>

Természetesen lehetőség van saját push üzenet feldolgozó szolgáltatás készítésére is egy `FirebaseMessagingService` létrehozásával, melyről további részletek [itt olvashatók](https://firebase.google.com/docs/cloud-messaging/android/receive).  

## Crashlytics

A Firebase Console-on először navigáljunk a Crashlytics menüpontra, és kapcsoljuk be a funkciót. Válasszuk az új Firebase alkalmazás integrációját.

Ezután a projekt szintű `build.gradle` fájlban fel kell vennünk függőségként egy plugint a `buildscript` rész `dependencies` részébe:
 
```groovy
id("com.google.firebase.crashlytics") version "2.9.9" apply false
```

Ezekkel a módosításokkal egy Gradle plugint adtunk hozzá a projektünkhöz, amit a modul szintű `build.gradle` fájl elején be kell kapcsolnunk a már meglévők után:

```groovy
id("com.google.firebase.crashlytics")
```

Végül pedig szükségünk van két egyszerű Gradle függőségre is, amit a meglévő Firebase függőségek mellé helyezhetünk, a modul szintű `build.gradle` fájlban:

```groovy
implementation("com.google.firebase:firebase-crashlytics-ktx")
implementation("com.google.firebase:firebase-analytics-ktx")
```

<p align="center">
<img src="./assets/firebase_crash.png">
</p>

Vegyünk fel egy új akciót `TodosScreen` `TodoAppBar` részébe, amivel az alkalmazást hibával be tudjuk zárni:

```kotlin
TodoAppBar(
	title = stringResource(id = StringResources.app_bar_title_todos),
	actions = {
		IconButton(onClick = {
			viewModel.signOut()
			onSignOut()
		}) {
			Icon(imageVector = Icons.Default.Logout, contentDescription = null)
		}
		IconButton(onClick = {
			throw RuntimeException("Test crash!")
		}) {
			Icon(imageVector = Icons.Default.Close, contentDescription = null)
		}
	}
)
```

Végül a *Firebase console-ban* is engedélyezzük a funkciót a *Crashlytics* menüpont alatt.

Próbáljuk ki saját hibajelzések készítését a menü eseménykezelőjében. Vizsgáljuk meg, megérkezik-e a Firebase Console-ba a hibaüzenet!

## Analitika

Most engedélyezzük az analitikát a *Firebase console* *Analytics* menüpontja alatt!

Az SDK-t már beállítottuk, ezért a megfelelő Account kiválasztása után a Nextre, majd a Finishre kattinhatunk.

Ezután az alkalmazás már naplóz alapvető analitikákat, használati statisztikákat, melyek
ugyanezen menüpont alatt lesznek elérhetők.

Emellett természetesen lehetőség van az analitika kibővítésére és testreszabására is. Az ehhez szükséges Firebase függőséget a Crashlyticsnél már felvettük.

Készítsünk saját analitika üzeneteket egy újabb akcióból küldve, ami szintén a `TopAppBar`-ba kerül:

```xml
IconButton(onClick = {
	val bundle = Bundle()
	bundle.putString("demo_key", "idabc")
	bundle.putString("data_key", "mydata")

	FirebaseAnalytics.getInstance(context)
		.logEvent(FirebaseAnalytics.Event.LOGIN, bundle)
}) {
	Icon(imageVector = Icons.Default.Message, contentDescription = null)
}
```

Fontos kiemelni, hogy nem garantált, hogy az analitika valós időben látszik a *Firebase console*-on. 30 percig vagy akár tovább is tarthat, mire egy-egy esemény itt megjelenik.

<p align="center">
<img src="./assets/firebase_analytics.png">
</p>

!!!example "BEADANDÓ (1 pont)" 
	Készíts egy képernyő képet, amin látszódik a futó alkalmazás a lista oldalon, illetve a két új akciógomb forráskódja, melyben a Neptun-kód komment formájában látható. A képernyőkép szükséges feltétele a pontszám megszerzésének.

## Önálló feladatok

### Automatikus bejelentkezés
Valósítsuk meg, hogy a bejelentkező képernyő helyett egyből a lista oldalra ugorjunk, ha a felhasználó kijelentkezés helyett csak bezárta az alkalmazást!

!!!example "BEADANDÓ (1 pont)" 
	Készíts egy képernyő képet, amin látszódik a futó alkalmazás a lista oldalon, az automatikus bejelentkezést megvalósító forráskód, melyben a Neptun-kód komment formájában látható. A képernyőkép szükséges feltétele a pontszám megszerzésének.

### Navigáció esemény jelzése
Küldjünk egy analitikai eseményt abban az esetben, ha a felhasználó megnyitja valamelyik feladatát! Az esemény tartalmazza a feladat azonosítóját is. Figyeljünk arra, hogy megfelelő névvel küldjük el az eseményt!

!!!example "BEADANDÓ (1 pont)" 
	Készíts egy képernyő képet, amin látszódik a futó alkalmazás a lista oldalon, a navigáció során az eseményt kiküldő forráskód, melyben a Neptun-kód komment formájában látható. A képernyőkép szükséges feltétele a pontszám megszerzésének.
