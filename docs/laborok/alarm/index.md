# Labor10 - Időzítés és értesítések (Alarm)

## Bevezetés

Ebben a laborban egy időzítő alkalmazást készítünk, amely a beállított időintervallum eltelte után értesítést küld, akkor is, ha az alkalamzás felhasználói felülete nincs az előtérben, mert közben a felhasználó másik alkalmazást indított. 

A laborban érintett főbb témakörök:

- Háttérfeladat futtatása Service segítségével
- Időzített feladatok
- Értesítések küldése


## Előkészületek

A feladatok megoldása során ne felejtsd el követni a [feladat beadás folyamatát](../../tudnivalok/github/GitHub.md).

### Git repository létrehozása és letöltése

1. Moodle-ben keresd meg a laborhoz tartozó meghívó URL-jét és annak segítségével hozd létre a saját repository-dat.

2. Várd meg, míg elkészül a repository, majd checkout-old ki.

    !!! tip ""
        Egyetemi laborokban, ha a checkout során nem kér a rendszer felhasználónevet és jelszót, és nem sikerül a checkout, akkor valószínűleg a gépen korábban megjegyzett felhasználónévvel próbálkozott a rendszer. Először töröld ki a mentett belépési adatokat (lásd [itt](../../tudnivalok/github/GitHub-credentials.md)), és próbáld újra.

3. Hozz létre egy új ágat `megoldas` néven, és ezen az ágon dolgozz.

4. A `neptun.txt` fájlba írd bele a Neptun kódodat. A fájlban semmi más ne szerepeljen, csak egyetlen sorban a Neptun kód 6 karaktere.

## A projekt megnyitása

Nyissuk meg a template-ben levő projektet, és a laborvezetővel tekintsük át a tartalmát. A projektben
a UI építőelemei és a `drawable` erőforrások már megtalálhatók. Ezekhez fogjuk elkészíteni az időzítő
üzleti logikáját, amit összekötünk az előkészített felhasználói felülettel.

## A függőségek beállítása

Vegyük fel az alábbi függőségeket a modul szintű `build.gradle.kts` fájlba. Ezekre az idő megadásához
lesz majd szükségünk.

```kotlin
implementation("com.maxkeppeler.sheets-compose-dialogs:core:1.0.3")
implementation("com.maxkeppeler.sheets-compose-dialogs:clock:1.0.3")
implementation("com.maxkeppeler.sheets-compose-dialogs:duration:1.0.3")
```

## A doménmodellek elkészítése

Először néhány olyan osztályt készítünk, amelyekkel az alkalmazás aktuális állapota, illetve az állapotváltozást előidéző események reprezentálhatók. Ehhez készítsünk egy `util` package-et. Ebbe hozzuk létre az alábbi osztályt:

```kotlin
sealed class AlarmEvent {
    data class SetAlarmDuration(val duration: Duration): AlarmEvent()
    data class SetAlarm(val context: Context): AlarmEvent()
    data class PauseAlarm(val context: Context): AlarmEvent()
    data class ResumeAlarm(val context: Context): AlarmEvent()
    data class StopAlarm(val context: Context): AlarmEvent()
}
```

Az osztály tartalma könnyen megérthető, az egyes felhasználói interakciókat mint lehetséges eseményeket jelképezi.

Most hozzuk létre az alábbi osztályokat egy fájlban:

```kotlin
enum class AlarmServiceState {
    IDLE, SET, PAUSE, CANCELED
}

data class AlarmState(
    val currentAlarmDuration: Duration = Duration.ZERO,
    val alarmDuration: Duration = Duration.ZERO,
    val alarmState: AlarmServiceState = AlarmServiceState.IDLE,
) {
    companion object {
        val state = MutableStateFlow(AlarmState())

        fun isAlarmSet(): Boolean = state.value.alarmState == AlarmServiceState.SET

        fun isAlarmPaused(): Boolean = state.value.alarmState == AlarmServiceState.PAUSE

        fun isAlarmIdle(): Boolean = state.value.alarmState == AlarmServiceState.IDLE || state.value.alarmState == AlarmServiceState.CANCELED

        fun isStopEnabled(): Boolean = state.value.alarmState == AlarmServiceState.SET

    }
}
```

Az `AlarmServiceState` lehetséges értékei írják le, hogy az időzítő épp várakozik, fut, pauzált vagy leállított. Az `AlarmState` ezt az aktuális állapotot tárolja, és mellé még egységbe zárja, hogy mennyi az időzítő célideje, és mennyi telt el eddig.

## A felhasználói felület és a doménmodell összekötése

A felhasználói felületünk a template-ben már rendelkezésre áll, de a viewmodelleket még el kell készítenünk, hogy
az imént létrehozott doménmodellel a UI-t össze tudjuk kapcsolni. A `ui.alarm` package-be hozzuk létre a ViewModelünket:

```kotlin
@HiltViewModel
class AlarmViewModel @Inject constructor(): ViewModel() {

    private val _state = AlarmState.state
    val state = _state.asStateFlow()

    fun onEvent(event: AlarmEvent) {
        when(event) {
            is AlarmEvent.SetAlarmDuration -> {
                _state.update { it.copy(
                    currentAlarmDuration = event.duration,
                    alarmDuration = event.duration
                ) }
            }
            is AlarmEvent.SetAlarm -> {
                val context = event.context

                _state.update { it.copy(
                    alarmState = AlarmServiceState.SET,
                ) }

                // TODO: Service kommunikáció
            }
            is AlarmEvent.PauseAlarm -> {
                val context = event.context

                _state.update { it.copy(
                    alarmState = AlarmServiceState.PAUSE,
                ) }

                // TODO: Service kommunikáció
            }
            is AlarmEvent.ResumeAlarm -> {
                val context = event.context

                _state.update { it.copy(alarmState = AlarmServiceState.SET) }

                // TODO: Service kommunikáció
            }
            is AlarmEvent.StopAlarm -> {
                val context = event.context

                _state.update { it.copy(
                    currentAlarmDuration = Duration.ZERO,
                    alarmDuration = Duration.ZERO,
                    alarmState = AlarmServiceState.CANCELED,
                ) }

                // TODO: Service kommunikáció
            }
        }
    }
}
```

Látható, hogy a ViewModel kezeli az eseményeket, és az alkalmazás állapotát ennek megfelelően frissíti, azonban a háttérben futó, tényleges időzítést végző `AlarmService` még nincs kész, így annak hívásai ide nincsenek bekötve.

Most ugyanebbe a package-ben hozzuk létre a teljes képernyőt is:

```kotlin
import androidx.compose.animation.ExperimentalAnimationApi
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.width
import androidx.compose.foundation.layout.wrapContentSize
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.runtime.Composable
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.unit.dp
import androidx.compose.ui.window.Popup
import androidx.hilt.navigation.compose.hiltViewModel
import com.google.accompanist.permissions.ExperimentalPermissionsApi
import com.maxkeppeker.sheets.core.models.base.rememberSheetState
import com.maxkeppeler.sheets.duration.DurationDialog
import com.maxkeppeler.sheets.duration.models.DurationConfig
import com.maxkeppeler.sheets.duration.models.DurationFormat
import com.maxkeppeler.sheets.duration.models.DurationSelection
import hu.bme.aut.android.alarm.R
import hu.bme.aut.android.alarm.ui.common.AlarmStateButton
import hu.bme.aut.android.alarm.ui.common.DurationCounter
import hu.bme.aut.android.alarm.ui.theme.pauseColor
import hu.bme.aut.android.alarm.ui.theme.startColor
import hu.bme.aut.android.alarm.ui.theme.stopColor
import hu.bme.aut.android.alarm.util.AlarmEvent
import hu.bme.aut.android.alarm.util.AlarmState
import kotlin.time.DurationUnit
import kotlin.time.toDuration

@OptIn(ExperimentalMaterial3Api::class)
@ExperimentalPermissionsApi
@ExperimentalAnimationApi
@Composable
fun AlarmScreen(
    viewModel: AlarmViewModel = hiltViewModel()
) {

    val state by viewModel.state.collectAsState()

    val context = LocalContext.current

    Column(
        modifier = Modifier
            .fillMaxSize()
            .background(MaterialTheme.colorScheme.background),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {

        var isDialogShown by remember { mutableStateOf(false) }
        DurationCounter(
            duration = state.currentAlarmDuration,
            clickEnabled = AlarmState.isAlarmIdle(),
            onClick = { isDialogShown = !isDialogShown }
        )

        if (isDialogShown) {
            Popup(
                alignment = Alignment.Center,
                onDismissRequest = {
                    isDialogShown = !isDialogShown
                }
            ) {
                Box(
                    modifier = Modifier.wrapContentSize()
                ) {
                    Surface(
                        shape = MaterialTheme.shapes.medium,
                        color = MaterialTheme.colorScheme.surface,
                    ) {
                        DurationDialog(
                            state = rememberSheetState(
                                visible = true,
                                onCloseRequest = { isDialogShown = !isDialogShown }
                            ),
                            selection = DurationSelection {
                                val duration = it.toDuration(DurationUnit.SECONDS)
                                viewModel.onEvent(AlarmEvent.SetAlarmDuration(duration))
                            },
                            config = DurationConfig(
                                timeFormat = DurationFormat.HH_MM_SS,
                                currentTime = 0L,
                            ),
                        )
                    }
                }
            }
        }

        Row(
            modifier = Modifier
                .padding(vertical = 10.dp)
                .wrapContentSize(Alignment.Center)
        ) {
            AlarmStateButton(
                iconId = if (AlarmState.isAlarmSet() && !AlarmState.isAlarmPaused()) {
                    R.drawable.pause_24
                } else R.drawable.play_24,
                surfaceColor = if (AlarmState.isAlarmSet() && !AlarmState.isAlarmPaused()) {
                    MaterialTheme.colorScheme.pauseColor
                } else MaterialTheme.colorScheme.startColor,
            ) {
                if (!AlarmState.isAlarmSet()) {
                    viewModel.onEvent(AlarmEvent.SetAlarm(context))
                } else {
                    if (AlarmState.isAlarmPaused()) {
                        viewModel.onEvent(AlarmEvent.ResumeAlarm(context))
                    } else {
                        viewModel.onEvent(AlarmEvent.PauseAlarm(context))
                    }
                }
            }
            Spacer(modifier = Modifier.width(5.dp))
            AlarmStateButton(
                iconId = R.drawable.stop_24,
                surfaceColor = MaterialTheme.colorScheme.stopColor,
                enabled = AlarmState.isStopEnabled()
            ) {
                viewModel.onEvent(AlarmEvent.StopAlarm(context))
            }
        }

    }
}
```

Még létre kell hoznunk az `AlarmApplication` osztályt pusztán a Hilt inicializálásához:

```kotlin
@HiltAndroidApp
class AlarmApplication : Application()
```

És az osztályt regisztráljuk is be a Manifest fájlban:

```xml
    <application
        android:name=".AlarmApplication"

        ...
```

Bár még az alkalmazáslogikánk nem működik ténylegesen, vegyük fel ide a szükséges engedélyeket is. Az alkalmazásunk egy előtérben futó (foreground) Service-t fog használni ahhoz, hogy az időzítés akkor is működjön, ha az alkalmazás `Activity`-je már nem látható. Az ilyen Service-ekhez kötelező, hogy legyen értesítés az értesítési sávon, hogy a felhasználó mindig lássa, hogy milyen alkalmazások futnak a háttérben. (Ennek egy tipikus példája a zenelejátszó alkalmazások esete is.() Ezért az értesítésekhez szükséges engedélyt is fel kell venni. Szükséges még a pontos riasztás, és a pontos riasztás időzítésének engedélye:

```xml
<uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
<uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>

<uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM" />
<uses-permission android:name="android.permission.USE_EXACT_ALARM"/>
```

Még a `MainActivity` elkészítése maradt hátra az alapvető feladatok közül. Ebben elkérjük az engedélyeket, és megjelenítjük a létrehozott képernyőt:

```kotlin
@ExperimentalPermissionsApi
@ExperimentalAnimationApi
@AndroidEntryPoint
class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            AlarmTheme {
                AlarmScreen()
            }
        }

        if (Build.VERSION.SDK_INT == Build.VERSION_CODES.TIRAMISU) {
            requestPermissions(
                Manifest.permission.POST_NOTIFICATIONS
            )
        }
    }

    private fun requestPermissions(vararg permissions: String) {
        val requestPermissionLauncher = registerForActivityResult(
            ActivityResultContracts.RequestMultiplePermissions()
        ) { result ->
            result.entries.forEach {
                Log.d("MainActivity", "${it.key} = ${it.value}")
            }
        }
        requestPermissionLauncher.launch(permissions.asList().toTypedArray())
    }
}
```

Ha a `POST_NOTIFICATIONS` konstansot nem találja a fordító, akkor az `import android.Manifest` sort vegyük fel a fájl elejére.

Most már indítható az alkalmazás, és megjelenik a felhasználói felület, de még nem működik érdemi módon, hiszen a az üzleti logikát végző `Service` nincs kész.

!!!example "BEADANDÓ (1 pont)" 
	Készíts egy **képernyőképet**, amelyen látszik a **futó alkalmazás**, az **általad írt kódrészlet**, valamint a **neptun kódod a kódban valahol kommentként**. 

	A képet a megoldásban a repository-ba f1.png néven töltsd föl.

	A képernyőkép szükséges feltétele a pontszám megszerzésének.

## Az időzítés elkészítése

A `Service` komponensünk a fentiek alapján időzítést és értesítéseket is fog használni, ezért először ezekhez készítünk némi segédlogikát, hogy a `Service` utána könnyebben elkészíthető legyen. 

Az időzítést kiszervezzük egy segédosztályba, hogy a `Service` kódja átláthatóbb maradjon. Először egy `AlarmItem` osztályt készítünk, ez egy időzítendő eseményt jelöl, tulajdonképpen csak az időzítés idejét tartalmazza. Ezt egy új, `scheduler` nevű package-be tegyük:

```kotlin
data class AlarmItem(
    val time: LocalDateTime
)
```

Szintén ebbe a package-be készítünk egy `AlarmReceiver` nevű `BroadcastReceivert`, ezt fogjuk beregisztrálni majd az időzítés leteltére, és az Android időzítő rendszere az idő leteltekor ennek fog értesítést küldeni. Ebben egy `Intent` segítségével a `Service` komponenst fogjuk meghívni, de mivel ez még nincs kész, így a kód jelenleg nem fordul le, majd a `Service` elkészültekor válik az egész működőképessé. Ezt az osztályt is a `scheduler` package-be tegyük:

```kotlin
class AlarmReceiver: BroadcastReceiver() {

    override fun onReceive(context: Context?, intent: Intent?) {
        Intent(context, AlarmService::class.java).apply {
            this.action = AlarmService.ACTION_PLAY
            context?.startService(this)
        }
    }
}
```

Mivel a `BroadcastReceiver` is egy fő komponenstípus Androidon, ezért ezt a Manifest fájlban is szükséges regisztrálni:

```xml
<receiver android:name=".scheduler.AlarmReceiver" />
```

És végül egy `AlarmScheduler` osztályt készítünk:

```kotlin
class AlarmScheduler @Inject constructor(
    @ApplicationContext private val context: Context,
) {
    private val alarmManager = context.getSystemService(AlarmManager::class.java)

    fun schedule(alarmItem: AlarmItem) {
        val intent = Intent(context, AlarmReceiver::class.java).apply {
            putExtra("ALARM_TIME", alarmItem.time)
        }
        alarmManager.setExactAndAllowWhileIdle(
            AlarmManager.RTC_WAKEUP,
            alarmItem.time.atZone(ZoneId.systemDefault()).toEpochSecond() * 1000,
            PendingIntent.getBroadcast(
                context,
                alarmItem.hashCode(),
                intent,
                PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
            )
        )
    }

    fun cancel(item: AlarmItem) {
        alarmManager.cancel(
            PendingIntent.getBroadcast(
                context,
                item.hashCode(),
                Intent(context, AlarmReceiver::class.java),
                PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
            )
        )
    }

}
```

Ebben az osztályban a rendszertől kérünk referenciát az `AlarmManagerre`, amelynek segítségével a rendszerben levő időzítő szolgáltatások elérhetőek. Az osztály két metódust definiál, az egyikkel eseményt időzíthetünk, a másikkal pedig meglévő időzítést törölhetünk.

## Az értesítések küldése

Az értesítések küldését is kiszervezzük, hogy a `Service` osztályunk egyszerűbb legyen. Először is a `notification` package-et hozzuk létre, és ebben készítsük el a `NotificationHelper` osztályt:

```kotlin
class NotificationHelper @Inject constructor(
    val notificationManager: NotificationManager,
    val notificationBuilder: NotificationCompat.Builder
) {
    fun updateNotification(
        hours: Int,
        minutes:Int,
        seconds:Int,
        durationInSeconds: Int,
    ) {
        val progress = durationInSeconds - hours * 60 * 60 - minutes * 60 - seconds
        notificationManager.notify(
            NOTIFICATION_ID,
            notificationBuilder
                .setProgress(durationInSeconds,progress,false)
                .setContentText(
                    String.format("%02d:%02d:%02d",hours,minutes,seconds)
                ).build()
        )
    }

    fun cancelNotification() {
        notificationManager.cancel(NOTIFICATION_ID)
    }

    @SuppressLint("RestrictedApi")
    fun setNotificationButton(vararg actions: NotificationCompat.Action) {
        notificationBuilder.mActions.clear()
        actions.forEachIndexed { index, action ->
            notificationBuilder.mActions.add(index, action)
        }
    }

    fun createNotificationChannel() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val alarmChannel = NotificationChannel(
                NOTIFICATION_CHANNEL_ID,
                NOTIFICATION_CHANNEL_NAME,
                NotificationManager.IMPORTANCE_LOW
            )
            notificationManager.createNotificationChannel(alarmChannel)
        }
    }

    companion object {
        const val NOTIFICATION_ID = 101
        const val NOTIFICATION_CHANNEL_NAME = "ALARM_NOTIFICATION"
        const val NOTIFICATION_CHANNEL_ID = "ALARM_NOTIFICATION_ID"
    }
}
```

!!!example "BEADANDÓ (1 pont)" 
	Készíts egy **képernyőképet**, amelyen látszik az **általad írt kódrészlet**, valamint a **neptun kódod a kódban valahol kommentként**. 

	A képet a megoldásban a repository-ba f2.png néven töltsd föl.

	A képernyőkép szükséges feltétele a pontszám megszerzésének.

Tekintsük át, hogyan épül fel ez az osztály!

- Két függősége van: a `NotificationManager` segítségével kezelhetők az értesítések a rendszeren, viszont az ennek átadott értesítések akár elég összetettek is lehetnek, ezért a builder tervezési minta szerint a `NotificationBuilder` segítségével kell őket felépítenünk.
- Az `updateNotification` létrehozza vagy frissíti az értesítést, és ezen megjelenik a hátralevő idő óra, perc, másodperc bontásban, valamint egy folyamatjelző sáv, "progress bar" is, amelyet a hátralevő idő alapján számítunk. Látható, hogy egy numerikus azonosítót is megadunk az értesítéshez, ez teszi lehetővé, hogy utána majd törölni tudjuk.
- A `cancelNotification` törli a már létrehozott értesítést.
- A `setNotificationButton` segítségével gombokat adhatunk az értesítéshez. Hogy milyen és hány gombot szeretnénk hozzáadni, ez az időzítő állapotától függ, ezért hasznos, hogy külön metódussal adhassuk meg. Például szüneteltetést végző gomb hozzáadásának akkor van értelme, ha az időzítő épp fut.

- A `createNotificationChannel` értesítési csatornát hoz létre. Az Android O óta az értesítéseket csatornákhoz kell rendelni. Ez azért hasznos, mert a felhasználó az alkalmazáson belül csatornánként tudja tiltani vagy engedélyezni a csatornákat. Pl. a fontos értesítések külön csatornára szervezhetők, és az elhanyagolhatóbb jelentőségűeket a felhasználó könnyedén letilthatja. Jelen alkalmazásban ezt a lehetőséget nem használjuk ki, csupán egy csatornánk lesz, de azt akkor is létre kell hozzuk. A csatorna többszöri létrehozása nem okoz problémát, ezért elég minden értesítés előtt meghívni, hogy biztosan létrejöjjön, nem szükséges tesztelni, hogy létezik-e.

Látható, hogy a `NotificationManager` és a `NotificationBuilder` a `NotificationHelper` függőségei, amelyeket a konstruktoron keresztül kap meg a Dagger/Hilt segítségével. Viszont ehhez még konfiguráció szükséges, hogy ezek a komponensek valóban létrejöjjenek. Hozzuk létre a `notification.di` package-et, majd ebben az alábbi modult:

```kotlin
@ExperimentalAnimationApi
@Module
@InstallIn(ServiceComponent::class)
object NotificationModule {

    @Provides
    @ServiceScoped
    fun provideNotificationBuilder(
        @ApplicationContext context: Context
    ) = NotificationCompat.Builder(context,NOTIFICATION_CHANNEL_ID)
        .setSmallIcon(R.drawable.ic_round_alarm_24)
        .setContentTitle(context.getString(R.string.alarm_notification_title))
        .setContentText("00:00")
        .setOngoing(true)
        .addAction(
            R.drawable.ic_round_stop_24,
            context.getString(R.string.alarm_notification_action_cancel),
            null
        )
        .setContentIntent(null)

    @Provides
    @ServiceScoped
    fun provideNotificationManager(
        @ApplicationContext context: Context
    ) = context.getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager

    @Provides
    @ServiceScoped
    fun provideNotificationHelper(
        notificationBuilder: NotificationCompat.Builder,
        notificationManager: NotificationManager
    ) = NotificationHelper(notificationManager, notificationBuilder)
}
```

!!!example "BEADANDÓ (1 pont)" 
	Készíts egy **képernyőképet**, amelyen látszik az **általad írt kódrészlet**, valamint a **neptun kódod a kódban valahol kommentként**. 

	A képet a megoldásban a repository-ba f3.png néven töltsd föl.

	A képernyőkép szükséges feltétele a pontszám megszerzésének.

## A Service elkészítése

Hozzunk létre egy `service` package-et, majd ebbe készítsük el az `AlarmService` osztályt:

```kotlin
import android.app.PendingIntent
import android.app.Service
import android.content.Intent
import android.media.MediaPlayer
import android.media.RingtoneManager
import android.os.Build
import android.os.IBinder
import androidx.core.app.NotificationCompat
import dagger.hilt.android.AndroidEntryPoint
import hu.bme.aut.android.alarm.notification.NotificationHelper
import hu.bme.aut.android.alarm.notification.NotificationHelper.Companion.NOTIFICATION_ID
import hu.bme.aut.android.alarm.scheduler.AlarmItem
import hu.bme.aut.android.alarm.scheduler.AlarmScheduler
import hu.bme.aut.android.alarm.util.AlarmServiceState
import hu.bme.aut.android.alarm.util.AlarmState
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.update
import java.time.LocalDateTime
import java.util.Timer
import javax.inject.Inject
import kotlin.concurrent.fixedRateTimer
import kotlin.time.Duration
import kotlin.time.Duration.Companion.seconds

@AndroidEntryPoint
class AlarmService : Service(), MediaPlayer.OnPreparedListener {

    @Inject
    lateinit var notificationHelper: NotificationHelper

    @Inject
    lateinit var alarmScheduler: AlarmScheduler

    private val _state = AlarmState.state
    private val state = _state.asStateFlow()

    private lateinit var timer: Timer

    private var alarmItem: AlarmItem? = null

    private var mMediaPlayer: MediaPlayer? = null

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        intent?.action.let { action ->
            when (action) {
                ACTION_SET -> {
                    intent?.getStringExtra(ALARM_TIME)?.let { durationString ->
                        Duration.parse(durationString).let { duration ->
                            _state.update { it.copy(
                                alarmState = AlarmServiceState.SET,
                                currentAlarmDuration = duration
                            ) }
                        }
                    }

                    alarmItem = AlarmItem(
                        LocalDateTime.now().plusSeconds(
                            state.value.currentAlarmDuration.inWholeSeconds
                        ))
                    alarmItem?.let(alarmScheduler::schedule)

                    notificationHelper.createNotificationChannel()
                    notificationHelper.setNotificationButton(
                        NotificationCompat.Action(
                            0,
                            "Pause",
                            pauseAlarm()
                        ),
                        NotificationCompat.Action(
                            0,
                            "Cancel",
                            cancelAlarm()
                        )
                    )
                    startForeground(
                        NOTIFICATION_ID,
                        notificationHelper.notificationBuilder.build()
                    )
                    setAlarm { h, m, s ->
                        notificationHelper.updateNotification(
                            h, m, s,
                            state.value.alarmDuration.inWholeSeconds.toInt()
                        )
                    }
                }
                ACTION_PAUSE -> {
                    if (this::timer.isInitialized) {
                        timer.cancel()
                    }
                    notificationHelper.setNotificationButton(
                        NotificationCompat.Action(
                            0,
                            "Resume",
                            resumeAlarm()
                        )
                    )
                    notificationHelper.notificationManager.notify(
                        NOTIFICATION_ID,
                        notificationHelper.notificationBuilder.build()
                    )
                    _state.update { it.copy(alarmState = AlarmServiceState.PAUSE) }

                    alarmItem?.let(alarmScheduler::cancel)
                }
                ACTION_CANCEL -> {
                    if (this::timer.isInitialized) {
                        timer.cancel()
                    }
                    _state.update { it.copy(alarmState = AlarmServiceState.CANCELED) }
                    notificationHelper.cancelNotification()
                    if (Build.VERSION.SDK_INT > Build.VERSION_CODES.N) {
                        stopForeground(STOP_FOREGROUND_REMOVE)
                    } else stopForeground(STOP_FOREGROUND_DETACH)
                    stopSelf()
                    alarmItem?.let(alarmScheduler::cancel)
                    mMediaPlayer?.stop()
                }
                ACTION_PLAY -> {
                    mMediaPlayer = MediaPlayer()
                    mMediaPlayer?.apply {
                        reset()
                        setDataSource(
                            this@AlarmService,
                            RingtoneManager.getDefaultUri(RingtoneManager.TYPE_ALARM)
                        )
                        setOnPreparedListener(this@AlarmService)
                        prepareAsync()
                    }
                }
                else -> Unit
            }
        }

        when(intent?.getStringExtra(ALARM_STATE)) {
            AlarmServiceState.SET.name -> {

                _state.update { it.copy(alarmState = AlarmServiceState.SET) }

                alarmItem = AlarmItem(
                    LocalDateTime.now().plusSeconds(
                        state.value.currentAlarmDuration.inWholeSeconds
                    )
                )

                alarmItem?.let(alarmScheduler::schedule)

                notificationHelper.createNotificationChannel()
                notificationHelper.setNotificationButton(
                    NotificationCompat.Action(
                        0,
                        "Pause",
                        pauseAlarm()
                    ),
                    NotificationCompat.Action(
                        0,
                        "Cancel",
                        cancelAlarm()
                    )
                )
                startForeground(
                    NOTIFICATION_ID,
                    notificationHelper.notificationBuilder.build()
                )
                setAlarm { h, m, s ->
                    notificationHelper.updateNotification(
                        h, m, s,
                        state.value.alarmDuration.inWholeSeconds.toInt()
                    )
                }
            }
            AlarmServiceState.PAUSE.name -> {
                _state.update { it.copy(alarmState = AlarmServiceState.PAUSE) }

                if (this::timer.isInitialized) {
                    timer.cancel()
                }

                notificationHelper.setNotificationButton(
                    NotificationCompat.Action(
                        0,
                        "Resume",
                        resumeAlarm()
                    ),
                    NotificationCompat.Action(
                        0,
                        "Cancel",
                        cancelAlarm()
                    )
                )
                notificationHelper.notificationManager.notify(
                    NOTIFICATION_ID,
                    notificationHelper.notificationBuilder.build()
                )
                alarmItem?.let(alarmScheduler::cancel)
            }
            AlarmServiceState.CANCELED.name -> {
                _state.update { it.copy(
                    currentAlarmDuration = Duration.ZERO,
                    alarmDuration = Duration.ZERO,
                    alarmState = AlarmServiceState.IDLE,
                ) }

                if (this::timer.isInitialized) {
                    timer.cancel()
                }

                notificationHelper.cancelNotification()
                if (Build.VERSION.SDK_INT > Build.VERSION_CODES.N) {
                    stopForeground(STOP_FOREGROUND_REMOVE)
                } else stopForeground(STOP_FOREGROUND_DETACH)
                stopSelf()
                alarmItem?.let(alarmScheduler::cancel)
                mMediaPlayer?.stop()
            }
        }
        return super.onStartCommand(intent, flags, startId)
    }

    private fun setAlarm(onTick: (hours: Int, minutes: Int, seconds: Int) -> Unit) {
        timer = fixedRateTimer(initialDelay = 1000L, period = 1000L) {
            if (state.value.currentAlarmDuration != Duration.ZERO) {
                val newDuration = state.value.currentAlarmDuration - 1.seconds
                _state.update { it.copy(currentAlarmDuration = newDuration) }
                state.value.currentAlarmDuration.toComponents { hours, minutes, seconds, _ ->
                    onTick(hours.toInt(),minutes,seconds)
                }
            }
        }
    }

    private fun cancelAlarm(): PendingIntent {
        val cancelIntent = Intent(this, AlarmService::class.java).apply {
            putExtra(ALARM_STATE, AlarmServiceState.CANCELED.name)
        }
        return PendingIntent.getService(
            this, CANCEL_REQUEST_CODE, cancelIntent, flag
        )
    }

    private fun resumeAlarm(): PendingIntent {
        val resumeIntent = Intent(this, AlarmService::class.java).apply {
            putExtra(ALARM_STATE, AlarmServiceState.SET.name)
        }
        return PendingIntent.getService(
            this, RESUME_REQUEST_CODE, resumeIntent, flag
        )
    }

    private fun pauseAlarm(): PendingIntent {
        val pauseIntent = Intent(applicationContext, AlarmService::class.java).apply {
            putExtra(ALARM_STATE, AlarmServiceState.PAUSE.name)
        }
        return PendingIntent.getService(
            this, PAUSE_REQUEST_CODE, pauseIntent, flag
        )
    }

    override fun onDestroy() {
        super.onDestroy()
        _state.update { it.copy(
            currentAlarmDuration = Duration.ZERO,
            alarmDuration = Duration.ZERO,
            alarmState = AlarmServiceState.CANCELED,
        ) }
        if (this::timer.isInitialized) timer.cancel()
        mMediaPlayer?.release()
        mMediaPlayer = null
    }

    override fun onBind(p0: Intent?): IBinder? = null

    companion object {
        const val ALARM_STATE = "ALARM_STATE"

        const val ACTION_SET = "ALARM_SET"
        const val ACTION_PAUSE = "ALARM_PAUSE"
        const val ACTION_CANCEL = "ALARM_CANCEL"
        const val ACTION_PLAY = "ALARM_PLAY"

        const val ALARM_TIME = "ALARM_TIME"

        private const val flag = PendingIntent.FLAG_IMMUTABLE

        private const val CANCEL_REQUEST_CODE = 102
        private const val RESUME_REQUEST_CODE = 103
        private const val PAUSE_REQUEST_CODE = 104
    }

    override fun onPrepared(mediaPlayer: MediaPlayer) {
        mediaPlayer.start()
    }
}
```

Tekintsük át, hogyan is működik a fenti `Service`! Egy Android `Service` két módon működhet. Lehet "Started Service", ez azt jelenti, hogy indulás után a háttérben (Background Service) teszi a dolgát, amíg az alkalmazás valamilyen komponense látható, vagy akár akkor is működik, amikor az alkalmazás nem látható (Foreground Service), viszont ilyenkor egy értesítést kötelező megjelenítenie, hogy a felhasználó számára ne maradhasson észrevétlen (biztonsági okokból). Az `AlarmService` az alkalmazásunkban egy Foreground Service-ként működik, hogy az időzítő akkor se álljon le, ha az alkalmazás felülete már nincs előtérben. A `Service` másik lehetséges működési módja a "Bound Service". Ez azt jelenti, hogy más komponensek kapcsolódhatnak hozzá, és feladatokat végeztethetnek vele. Addig működik ilyen módon, amíg legalább egy kapcsolódó komponens van. Egy `Service` lehetne egyszerre "Started Service" és "Bound Service" is, de most utóbbi funkcionalitásra nincs szükség az alkalmazásban, mert "Started Service"-ként képes az állapotot frissíteni, aminek változása utána a felhasználói felületen is megjelenik.

Nézzük át előbb az `AlarmService` segédmetódusait:

- `setAlarm`: ez elindítja az időzítést, és ehhez beállít egy másodpercenkénti ütemezőt is, amellyel majd lehet frissíteni a felhasználói felületen, illetve a megjelenített értesítésben kijelzett hátralevő időt. Ez a metódus callbackként kapja meg, hogy mit kell futtatnia, amikor egy másodperc eltelt.
- `cancelAlarm`: ez egy `PendingIntent`et gyárt le, amellyel megállítható az időzítő. A `PendingIntent` magának az `AlarmService`-nek küldi a megfelelő `Intent`et, és ez a `PendingIntent` majd az értesítésben megjelenített akcióhoz rendelhető.
- `resumeAlarm`: az előzőhöz hasonló, de az időzítés újraindításához használandó.
- `pauseAlarm`: mint az előzőek, de pauzálásra.

Az `AlarmService` lényegi logikája az `onStartCommand` metódusban van megvalósítva. Ez az eseménykezelő akkor fut le, amikor a `Service`-t "Started Service"-ként indítják. Ez egy `Intent` küldésével történik, amelyben az action és az extras segítségével utaznak a paraméterek, hogy a `Service`-től mit is kérünk. Nézzük végig az egyes eseteket! Alapvetően mindig az állapot megfelelő beállítása, az időzítő logika meghívása, illetve az értesítés megjelenítése/frissítése történik a korábban áttekintett kódrészletek meghívásával. Különösen fontos részlet, hogy az `ACTION_SET` akció, azaz az időzítő elindítása esetén a `Service` a `startForeground` hívással foreground módba kerül, hogy akkor is működjön továbbra is az időzítés, ha az alkalmazás nem látható. Megfigyelendő még az `ACTION_PLAY` esetében a `MediaPlayer` API használata az ébresztő dallam lejátszásához.

Mivel a `Service` is egy fő komponenstípus Androidon, ezért ezt a Manifest fájlban is szükséges regisztrálni:

```xml
<service
    android:name=".service.AlarmService"
    android:enabled="true"
    android:exported="false"/>
```

Most már összeköthetők a `ViewModel` és a `Service` is az `AlarmViewModel` megfelelő kiegészítésével:

```kotlin
@HiltViewModel
class AlarmViewModel @Inject constructor(): ViewModel() {

    private val _state = AlarmState.state
    val state = _state.asStateFlow()

    fun onEvent(event: AlarmEvent) {
        when(event) {
            is AlarmEvent.SetAlarmDuration -> {
                _state.update { it.copy(
                    currentAlarmDuration = event.duration,
                    alarmDuration = event.duration
                ) }
            }
            is AlarmEvent.SetAlarm -> {
                val context = event.context

                _state.update { it.copy(
                    alarmState = AlarmServiceState.SET,
                ) }

                Intent(context, AlarmService::class.java).apply {
                    this.action = AlarmService.ACTION_SET
                    state.value.currentAlarmDuration.toString()
                    context.startService(this)
                }
            }
            is AlarmEvent.PauseAlarm -> {
                val context = event.context

                _state.update { it.copy(
                    alarmState = AlarmServiceState.PAUSE,
                ) }

                Intent(context, AlarmService::class.java).apply {
                    this.action = AlarmService.ACTION_PAUSE
                    context.startService(this)
                }
            }
            is AlarmEvent.ResumeAlarm -> {
                val context = event.context

                _state.update { it.copy(alarmState = AlarmServiceState.SET) }

                Intent(context, AlarmService::class.java).apply {
                    this.action = AlarmService.ACTION_SET
                    context.startService(this)
                }
            }
            is AlarmEvent.StopAlarm -> {
                val context = event.context

                _state.update { it.copy(
                    currentAlarmDuration = Duration.ZERO,
                    alarmDuration = Duration.ZERO,
                    alarmState = AlarmServiceState.CANCELED,
                ) }

                Intent(context, AlarmService::class.java).apply {
                    this.action = AlarmService.ACTION_CANCEL
                    context.startService(this)
                }
            }
        }
    }
}
```

!!!example "BEADANDÓ (1 pont)" 
	Készíts egy **képernyőképet**, amelyen látszik a **futó alkalmazás**, az **általad írt kódrészlet**, valamint a **neptun kódod a kódban valahol kommentként**. 

	A képet a megoldásban a repository-ba f4.png néven töltsd föl.

	A képernyőkép szükséges feltétele a pontszám megszerzésének.


## Önálló feladat 

Valósítsd meg az ébresztőórákon megszokott "szundi" funkciót! Amikor az időzítő letelik, jeleníts meg az értesítésen egy "Snooze" gombot is, amivel e riasztás abbamarad, és egy újabb 5 perces időzítés indul.

!!!example "BEADANDÓ (1 pont)" 
	Készíts egy **képernyőképet**, amelyen látszik a **futó alkalmazás**, az **általad írt kódrészlet**, valamint a **neptun kódod a kódban valahol kommentként**. 

	A képet a megoldásban a repository-ba f5.png néven töltsd föl.

	A képernyőkép szükséges feltétele a pontszám megszerzésének.
