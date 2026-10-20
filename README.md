<img width="1038" height="326" alt="image" src="https://github.com/user-attachments/assets/062f4c38-8754-43e6-a0ea-9217b1ac0f03" /><img width="1038" height="326" alt="image" src="https://github.com/user-attachments/assets/1043726e-79ec-4cae-81f8-65584c929768" /><img width="1038" height="326" alt="image" src="https://github.com/user-attachments/assets/de328c7a-793f-48bb-9888-a1aa3c1ac78e" /><img width="644" height="576" alt="image" src="https://github.com/user-attachments/assets/7f5ddb47-a07e-44fb-bc7f-82df715ba805" />Тема 1. Сервисы (Services): Started, Bound и Foreground (задание 1-5)
<img width="1911" height="1035" alt="image" src="https://github.com/user-attachments/assets/5da39cef-917d-4adb-ae4e-b87a7aed495e" />
package com.example.layoutmastery;

import android.Manifest;
import android.app.Activity;
import android.content.Intent;
import android.content.pm.PackageManager;
import android.os.Build;
import android.os.Bundle;
import android.widget.Button;
import android.widget.LinearLayout;
import android.widget.ScrollView;
import android.widget.TextView;
import android.widget.Toast;

import com.example.layoutmastery.providers.NotesProvider;
import com.example.layoutmastery.services.CountdownService;
import com.example.layoutmastery.services.DownloadService;
import com.example.layoutmastery.services.MusicForegroundService;
import com.example.layoutmastery.services.ProcessInfoService;

public class MainActivity extends Activity {
    private static final int REQUEST_NOTIFICATIONS = 1001;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        ScrollView scroll = new ScrollView(this);
        LinearLayout page = new LinearLayout(this);
        page.setOrientation(LinearLayout.VERTICAL);
        int pad = dp(18);
        page.setPadding(pad, pad, pad, pad);
        page.setBackgroundColor(0xFFF8FAFC);

        TextView title = new TextView(this);
        title.setText("Layout Mastery\nAndroid system components");
        title.setTextSize(25);
        title.setTextColor(0xFF172554);
        title.setPadding(0, 0, 0, dp(8));
        page.addView(title);

        TextView intro = new TextView(this);
        intro.setText("Практика: Services • Broadcast Receivers • Content Providers\n\nЗапускайте примеры кнопками ниже. Подробности и теория — в README.md.");
        intro.setTextSize(15);
        intro.setTextColor(0xFF334155);
        intro.setPadding(0, 0, 0, dp(16));
        page.addView(intro);

        addButton(page, "1. Запустить таймер 60 секунд", v ->
                startService(new Intent(this, CountdownService.class)));

        addButton(page, "2. Запустить Foreground Service", v -> startMusicService());

        addButton(page, "Остановить Foreground Service", v ->
                stopService(new Intent(this, MusicForegroundService.class)));

        addButton(page, "3. Запустить задачу загрузки (stopSelf)", v ->
                startService(new Intent(this, DownloadService.class)));

        addButton(page, "4. Запустить сервис в отдельном процессе", v ->
                startService(new Intent(this, ProcessInfoService.class)));

        addButton(page, "Broadcast Receiver: отправить событие", v -> {
            Intent intent = new Intent(this, com.example.layoutmastery.receivers.DemoBroadcastReceiver.class);
            intent.setAction("com.example.layoutmastery.ACTION_DEMO");
            intent.putExtra("message", "Событие получено из MainActivity");
            sendBroadcast(intent);
            Toast.makeText(this, "Broadcast отправлен. Проверьте Logcat.", Toast.LENGTH_SHORT).show();
        });

        addButton(page, "Content Provider: добавить тестовую запись", v -> {
            android.content.ContentValues values = new android.content.ContentValues();
            values.put(NotesProvider.COLUMN_TITLE, "Тестовая заметка");
            values.put(NotesProvider.COLUMN_BODY, "Запись создана через ContentResolver");
            android.net.Uri uri = getContentResolver().insert(NotesProvider.CONTENT_URI, values);
            Toast.makeText(this, "Добавлено: " + uri, Toast.LENGTH_LONG).show();
        });

        addButton(page, "Content Provider: прочитать записи", v -> {
            try (android.database.Cursor cursor = getContentResolver().query(
                    NotesProvider.CONTENT_URI, null, null, null, "_id DESC")) {
                int count = cursor == null ? 0 : cursor.getCount();
                Toast.makeText(this, "Записей в Provider: " + count, Toast.LENGTH_LONG).show();
            }
        });

        TextView footer = new TextView(this);
        footer.setText("\nПодсказка: для просмотра Logcat выберите фильтр по тегам CountdownService, DownloadService, ProcessInfoService или DemoReceiver.");
        footer.setTextSize(13);
        footer.setTextColor(0xFF64748B);
        page.addView(footer);

        scroll.addView(page);
        setContentView(scroll);
    }

    private void startMusicService() {
        if (Build.VERSION.SDK_INT >= 33 &&
                checkSelfPermission(Manifest.permission.POST_NOTIFICATIONS) != PackageManager.PERMISSION_GRANTED) {
            requestPermissions(new String[]{Manifest.permission.POST_NOTIFICATIONS}, REQUEST_NOTIFICATIONS);
            return;
        }
        Intent intent = new Intent(this, MusicForegroundService.class);
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            startForegroundService(intent);
        } else {
            startService(intent);
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if (requestCode == REQUEST_NOTIFICATIONS && grantResults.length > 0 &&
                grantResults[0] == PackageManager.PERMISSION_GRANTED) {
            startMusicService();
        } else if (requestCode == REQUEST_NOTIFICATIONS) {
            Toast.makeText(this, "Разрешение на уведомления не выдано. Попробуйте снова или разрешите его в настройках.", Toast.LENGTH_LONG).show();
        }
    }

    private void addButton(LinearLayout parent, String label, android.view.View.OnClickListener listener) {
        Button button = new Button(this);
        button.setText(label);
        button.setAllCaps(false);
        button.setOnClickListener(listener);
        LinearLayout.LayoutParams lp = new LinearLayout.LayoutParams(
                LinearLayout.LayoutParams.MATCH_PARENT, LinearLayout.LayoutParams.WRAP_CONTENT);
        lp.bottomMargin = dp(8);
        parent.addView(button, lp);
    }

    private int dp(int value) {
        return (int) (value * getResources().getDisplayMetrics().density + 0.5f);
    }
}



Тема 2. Жизненный цикл сервисов и флаги перезапуска (Задание 1-5)
<img width="1909" height="1036" alt="image" src="https://github.com/user-attachments/assets/2ee5767c-35bd-413c-b444-632e76b6a740" />
   <manifest xmlns:android="http://schemas.android.com/apk/res/android">
      <application
        android:allowBackup="true"
        android:label="Layout Mastery — Тема 2"
        android:supportsRtl="true"
        android:theme="@android:style/Theme.Material.Light.NoActionBar">

        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <service android:name=".services.RandomNumberService" android:exported="false" />
        <service android:name=".services.StickyCounterService" android:exported="false" />
        <service android:name=".services.RebindService" android:exported="false" />
        <service android:name=".services.MultiClientService" android:exported="false" />
        <service android:name=".services.CombinedService" android:exported="false" />
    </application>
</manifest>



Тема 3. Широковещательные приемники (Broadcast Receivers)
<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/a0051efe-34b7-44bf-830e-79fd3e9455d4" />

package com.example.layoutmastery;

import android.content.Intent;
import android.content.IntentFilter;
import android.os.Bundle;
import android.graphics.Typeface;
import android.view.View;
import android.widget.Button;
import android.widget.LinearLayout;
import android.widget.ScrollView;
import android.widget.TextView;
import android.widget.Toast;
import android.content.BroadcastReceiver;
import android.content.Context;

import androidx.appcompat.app.AppCompatActivity;
import androidx.core.content.ContextCompat;

public class MainActivity extends AppCompatActivity {

    private TextView resultText;
    private BroadcastReceiver batteryReceiver;
    private BroadcastReceiver airplaneReceiver;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        ScrollView scrollView = new ScrollView(this);
        LinearLayout page = new LinearLayout(this);
        page.setOrientation(LinearLayout.VERTICAL);
        page.setPadding(20, 20, 20, 20);
        page.setBackgroundColor(0xFFF8FAFC);

        TextView title = new TextView(this);
        title.setText("Broadcast Receivers - Практическая работа");
        title.setTextSize(22);
        title.setTypeface(null, Typeface.BOLD);
        title.setTextColor(0xFF172554);
        page.addView(title);

        TextView description = new TextView(this);
        description.setText(
                "Демонстрация работы широковещательных " +
                "приёмников (Broadcast Receivers)."
        );
        description.setTextSize(14);
        description.setPadding(0, 16, 0, 16);
        page.addView(description);

        addButton(page, "1. Низкий заряд батареи",
                v -> registerBatteryReceiver());

        addButton(page, "2. Режим полета",
                v -> registerAirplaneReceiver());

        addButton(page, "3. После перезагрузки (статический)",
                v -> sendBootTest());

        addButton(page, "4. Кастомный бродкаст",
                v -> sendCustomBroadcast());

        addButton(page, "5. Упорядоченный бродкаст",
                v -> sendOrderedTest());

        TextView resultTitle = new TextView(this);
        resultTitle.setText("Результат:");
        resultTitle.setTextSize(16);
        resultTitle.setTypeface(null, Typeface.BOLD);
        resultTitle.setPadding(0, 20, 0, 8);
        page.addView(resultTitle);

        resultText = new TextView(this);
        resultText.setText(
                "Сообщения от приёмников появятся здесь..."
        );
        resultText.setTextSize(14);
        resultText.setPadding(12, 12, 12, 12);
        resultText.setBackgroundColor(0xFFE5EDFF);
        page.addView(resultText);

        scrollView.addView(page);
        setContentView(scrollView);
    }

    private void addButton(
            LinearLayout page,
            String text,
            View.OnClickListener listener
    ) {
        Button button = new Button(this);
        button.setText(text);
        button.setAllCaps(false);
        button.setOnClickListener(listener);

        LinearLayout.LayoutParams params =
                new LinearLayout.LayoutParams(
                        LinearLayout.LayoutParams.MATCH_PARENT,
                        LinearLayout.LayoutParams.WRAP_CONTENT
                );

        params.setMargins(0, 4, 0, 4);
        page.addView(button, params);
    }

    private void showResult(String message) {
        resultText.append("\n" + message);
    }

    private void registerBatteryReceiver() {
        if (batteryReceiver != null) {
            showResult("Приёмник батареи уже зарегистрирован.");
            return;
        }

        batteryReceiver = new BroadcastReceiver() {
            @Override
            public void onReceive(Context context, Intent intent) {
                showResult("Низкий заряд батареи!");
                Toast.makeText(
                        context,
                        "Низкий заряд батареи!",



Тема 4. Жизненный цикл Broadcast Receiver и асинхронностьpackage com.example.broadcastapp
<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/951691cf-3f99-4400-ad85-0b9f51f15838" />
import android.content.BroadcastReceiver
import android.content.Context
import android.content.Intent
import android.os.Looper
import android.util.Log
import androidx.work.OneTimeWorkRequestBuilder
import androidx.work.WorkManager
import java.util.concurrent.TimeUnit

class MyReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        // Проверяем, выполняется ли код в главном потоке
        val isMainThread =
            Looper.myLooper() == Looper.getMainLooper()

        Log.d("MyReceiver", "Главный поток: $isMainThread")

        // Передаём фоновую задачу в WorkManager
        val request = OneTimeWorkRequestBuilder<DemoWorker>()
            .setInitialDelay(5, TimeUnit.SECONDS)
            .build()

        WorkManager.getInstance(context).enqueue(request)

        // Разрешаем короткую асинхронную операцию
        val pendingResult = goAsync()

        Thread {
            try {
                val prefs = context.getSharedPreferences(
                    "app_prefs",
                    Context.MODE_PRIVATE
                )

                prefs.edit()
                    .putLong(
                        "last_broadcast_time",
                        System.currentTimeMillis()
                    )
                    .apply()

                Log.d("MyReceiver", "Время события сохранено")
            } finally {
                // Обязательно сообщаем системе о завершении
                pendingResult.finish()
            }
        }.start()
    }
}



Тема 5. Поставщики контента (Content Providers) и UriMatcher
<img width="767" height="484" alt="image" src="https://github.com/user-attachments/assets/1e544363-00c7-47db-baf4-da567c29837f" />
public class NotesProvider extends ContentProvider {

    public static final String AUTHORITY = "com.university.app.provider.notes";
    public static final Uri CONTENT_URI = Uri.parse("content://" + AUTHORITY + "/notes");

    private static final int CODE_NOTES_DIR = 1;
    private static final int CODE_NOTE_ITEM = 2;

    private static final UriMatcher uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);

    static {
        uriMatcher.addURI(AUTHORITY, "notes", CODE_NOTES_DIR);
        uriMatcher.addURI(AUTHORITY, "notes/#", CODE_NOTE_ITEM);
    }

    @Override
    public boolean onCreate() {
        // Инициализация локальной базы данных SQLite / Room
        return true;
    }

    @Override
    public Cursor query(Uri uri, String[] projection, String selection,
                        String[] selectionArgs, String sortOrder) {
        int match = uriMatcher.match(uri);
        switch (match) {
            case CODE_NOTES_DIR:
                // Выборка всей таблицы
                break;
            case CODE_NOTE_ITEM:
                // Извлечение конкретной записи по ID: ContentUris.parseId(uri)
                break;
            default:
                throw new IllegalArgumentException("Неизвестный URI: " + uri);
        }
        return null;
    }

    @Override
    public String getType(Uri uri) {
        return switch (uriMatcher.match(uri)) {
            case CODE_NOTES_DIR -> "vnd.android.cursor.dir/vnd.com.university.note";
            case CODE_NOTE_ITEM -> "vnd.android.cursor.item/vnd.com.university.note";
            default -> throw new IllegalArgumentException("Неизвестный URI: " + uri);
        };
    }

    @Override
    public Uri insert(Uri uri, ContentValues values) {
        // Вставка в БД и уведомление наблюдателей
        getContext().getContentResolver().notifyChange(uri, null);
        return ContentUris.withAppendedId(CONTENT_URI, 1);
    }

    @Override
    public int delete(Uri uri, String selection, String[] selectionArgs) {
        return 0;
    }

    @Override
    public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs) {
        return 0;
    }
}



Тема 6. Жизненный цикл и потокобезопасность Content Provider
<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/39687228-f9b2-4e8c-a765-9f435119a8aa" />
class MyContentProvider : ContentProvider() {

    private lateinit var helper: DatabaseHelper

    override fun onCreate(): Boolean {
        Log.i("InitOrder", "ContentProvider.onCreate()")
        val ctx = context ?: return false
        helper = DatabaseHelper(ctx)
        return true
    }

    override fun query(
        uri: Uri,
        projection: Array<out String>?,
        selection: String?,
        selectionArgs: Array<out String>?,
        sortOrder: String?
    ): Cursor {
        return helper.readableDatabase.query(
            NoteContract.TABLE_NAME,
            projection,
            selection,
            selectionArgs,
            null,
            null,
            sortOrder
        )
    }

    @Synchronized
    override fun insert(
        uri: Uri,
        values: ContentValues?
    ): Uri {
        requireNotNull(values)

        val id = helper.writableDatabase.insertOrThrow(
            NoteContract.TABLE_NAME,
            null,
            values
        )

        return Uri.withAppendedPath(
            NoteContract.CONTENT_URI,
            id.toString()
        )
    }

    @Synchronized
    override fun update(
        uri: Uri,
        values: ContentValues?,
        selection: String?,
        selectionArgs: Array<out String>?
    ): Int {
        return helper.writableDatabase.update(
            NoteContract.TABLE_NAME,
            values,
            selection,
            selectionArgs
        )
    }

    @Synchronized
    override fun delete(
        uri: Uri,
        selection: String?,
        selectionArgs: Array<out String>?
    ): Int {
        return helper.writableDatabase.delete(
            NoteContract.TABLE_NAME,
            selection,
            selectionArgs
        )
    }

    override fun getType(uri: Uri): String =
        "vnd.android.cursor.dir/vnd.notes"
}



50 СМЕШАННЫХ ЗАДАНИЙ


<img width="1388" height="569" alt="image" src="https://github.com/user-attachments/assets/288cf6c2-2fe0-495e-8bf1-f51e6e16d766" />
class StepService : Service(), SensorEventListener {

    private lateinit var sensorManager: SensorManager
    private var steps = 0
    private val handler = Handler(Looper.getMainLooper())

    override fun onCreate() {
        super.onCreate()

        sensorManager = getSystemService(SENSOR_SERVICE)
                as SensorManager

        val sensor = sensorManager.getDefaultSensor(
            Sensor.TYPE_STEP_COUNTER
        )

        sensor?.let {
            sensorManager.registerListener(
                this, it, SensorManager.SENSOR_DELAY_NORMAL
            )
        }

        startForeground(1, createNotification())
        handler.post(updateRunnable)
    }

    private val updateRunnable = object : Runnable {
        override fun run() {
            updateNotification()
            handler.postDelayed(this, 10_000)
        }
    }

    override fun onSensorChanged(event: SensorEvent) {
        steps = event.values[0].toInt()
    }

    override fun onAccuracyChanged(sensor: Sensor?, accuracy: Int) {}

    private fun createNotification(): Notification {
        val channelId = "steps"
        if (Build.VERSION.SDK_INT >= 26) {
            val channel = NotificationChannel(
                channelId, "Шагомер",
                NotificationManager.IMPORTANCE_LOW
            )
            getSystemService(NotificationManager::class.java)
                .createNotificationChannel(channel)
        }

        return NotificationCompat.Builder(this, channelId)
            .setContentTitle("Шагомер")
            .setContentText("Шаги: $steps")
            .setSmallIcon(android.R.drawable.ic_menu_compass)
            .build()
    }

    private fun updateNotification() {
        getSystemService(NotificationManager::class.java)
            .notify(1, createNotification())
    }

    override fun onDestroy() {
        handler.removeCallbacks(updateRunnable)
        sensorManager.unregisterListener(this)
        super.onDestroy()
    }

    override fun onBind(intent: Intent?): IBinder? = null
}

![Uploading image.png…]()


val playIntent = Intent(this, PlayerService::class.java)
    .setAction("PLAY")

val playPendingIntent = PendingIntent.getService(
    this, 1, playIntent,
    PendingIntent.FLAG_UPDATE_CURRENT or
        PendingIntent.FLAG_IMMUTABLE
)

val notification = NotificationCompat.Builder(this, "music")
    .setContentTitle("Музыкальный плеер")
    .setContentText("Воспроизведение")
    .setSmallIcon(android.R.drawable.ic_media_play)
    .addAction(0, "Play", playPendingIntent)
    .build()
<img width="271" height="352" alt="image" src="https://github.com/user-attachments/assets/7e551fee-163e-414e-bb20-0f604b2330b0" />



class DownloadService : Service() {

    override fun onStartCommand(
        intent: Intent?, flags: Int, startId: Int
    ): Int {
        Thread {
            for (i in 1..100) {
                // Здесь выполняется загрузка файла
                val progress = i

                Log.d("DownloadService", "$progress%")
            }
            stopSelf(startId)
        }.start()

        return START_REDELIVER_INTENT
    }

    override fun onBind(intent: Intent?): IBinder? = null
}
<img width="199" height="255" alt="image" src="https://github.com/user-attachments/assets/3bd0b2f7-4107-41fc-9875-216bf86ec4b0" />



class SocketService : Service() {

    private val handler = Handler(Looper.getMainLooper())

    private fun reconnect() {
        handler.postDelayed({
            connectSocket()
        }, 5000)
    }

    private fun connectSocket() {
        // Создать WebSocket-клиент и подключиться
        Log.d("SocketService", "Подключение к серверу")
    }

    override fun onCreate() {
        super.onCreate()
        connectSocket()
    }

    override fun onBind(intent: Intent?): IBinder? = null
}
<img width="195" height="253" alt="image" src="https://github.com/user-attachments/assets/5cb0a805-0947-43a5-99d3-902504c05bbf" />


class SocketService : Service() {

    private val handler = Handler(Looper.getMainLooper())

    private fun reconnect() {
        handler.postDelayed({
            connectSocket()
        }, 5000)
    }

    private fun connectSocket() {
        // Создать WebSocket-клиент и подключиться
        Log.d("SocketService", "Подключение к серверу")
    }

    override fun onCreate() {
        super.onCreate()
        connectSocket()
    }

    override fun onBind(intent: Intent?): IBinder? = null
}<img width="205" height="247" alt="image" src="https://github.com/user-attachments/assets/9f9626e1-c075-4e63-8c45-15036f4beb16" />



class VideoService : Service() {

    private val executor =
        Executors.newSingleThreadExecutor()

    fun compressVideo(path: String) {
        executor.execute {
            Log.d("VideoService", "Обработка: $path")
            // Передать видео в библиотеку перекодирования
        }
    }

    override fun onDestroy() {
        executor.shutdown()
        super.onDestroy()
    }

    override fun onBind(intent: Intent?): IBinder? = null
}
<img width="202" height="246" alt="image" src="https://github.com/user-attachments/assets/45bd0db3-29c9-429f-9a16-0812096b420c" />

class SocketService : Service() {

    private val handler = Handler(Looper.getMainLooper())

    private fun reconnect() {
        handler.postDelayed({
            connectSocket()
        }, 5000)
    }

    private fun connectSocket() {
        // Создать WebSocket-клиент и подключиться
        Log.d("SocketService", "Подключение к серверу")
    }

    override fun onCreate() {
        super.onCreate()
        connectSocket()
    }

    override fun onBind(intent: Intent?): IBinder? = null
}



    class VideoService : Service() {

    private val executor =
        Executors.newSingleThreadExecutor()

    fun compressVideo(path: String) {
        executor.execute {
            Log.d("VideoService", "Обработка: $path")
            // Передать видео в библиотеку перекодирования
        }
    }

    override fun onDestroy() {
        executor.shutdown()
        super.onDestroy()
    }

    override fun onBind(intent: Intent?): IBinder? = null
}



<img width="1038" height="326" alt="image" src="https://github.com/user-attachments/assets/0175c84b-851b-4344-854e-a47b12b39803" />


public class LogProvider extends ContentProvider {

    public boolean onCreate() {
        return true;
    }

    
    public Cursor query(Uri uri, String[] projection,
                        String selection, String[] args,
                        String sortOrder) {
        MatrixCursor cursor =
            new MatrixCursor(new String[]{"id", "log"});

        cursor.addRow(new Object[]{1, "Encrypted log"});
        return cursor;




    public String getType(Uri uri) {
        return "vnd.android.cursor.dir/vnd.example.logs";
    }

    
    public Uri insert(Uri uri, ContentValues values) {
        return null;
    }

    
    public int delete(Uri uri, String s, String[] args) {
        return 0;
    }

    
    public int update(Uri uri, ContentValues v,
                      String s, String[] args) {
        return 0;
    }
}


<img width="420" height="224" alt="image" src="https://github.com/user-attachments/assets/b7a0b5aa-9289-46c2-976f-faa4c1cbf638" />

public class VCardProvider extends ContentProvider {

    
    public ParcelFileDescriptor openFile(
            Uri uri, String mode)
            throws FileNotFoundException {

        File file = new File(
            getContext().getCacheDir(),
            "contact.vcf"
        );

        return ParcelFileDescriptor.open(
            file,
            ParcelFileDescriptor.MODE_READ_ONLY
        );
    }

    @Override
    public boolean onCreate() {
        return true;
    }
}



<img width="752" height="268" alt="image" src="https://github.com/user-attachments/assets/b37500c9-4c4e-4245-886d-a11dc190b628" />

public int bulkInsert(Uri uri, ContentValues[] values) {
    SQLiteDatabase db = dbHelper.getWritableDatabase();
    int count = 0;

    db.beginTransaction();
    try {
        for (ContentValues value : values) {
            long id = db.insert(
                "products", null, value
            );

            if (id != -1) count++;
        }

        db.setTransactionSuccessful();
    } finally {
        db.endTransaction();
    }

    return count;


<img width="1038" height="326" alt="image" src="https://github.com/user-attachments/assets/3206d7de-ffc1-4fb9-a383-a2a7d4fe6730" />

public int bulkInsert(Uri uri, ContentValues[] values) {
    SQLiteDatabase db = dbHelper.getWritableDatabase();
    int count = 0;

    db.beginTransaction();
    try {
        for (ContentValues value : values) {
            long id = db.insert(
                "products", null, value
            );

            if (id != -1) count++;
        }

        db.setTransactionSuccessful();
    } finally {
        db.endTransaction();
    }

    return count;


<img width="420" height="224" alt="image" src="https://github.com/user-attachments/assets/c532612a-74f4-475c-b141-ba2ad2dd75e1" />

public int bulkInsert(Uri uri, ContentValues[] values) {
    SQLiteDatabase db = dbHelper.getWritableDatabase();
    int count = 0;

    db.beginTransaction();
    try {
        for (ContentValues value : values) {
            long id = db.insert(
                "products", null, value
            );

            if (id != -1) count++;
        }

        db.setTransactionSuccessful();
    } finally {
        db.endTransaction();
    }

    return count;
}


<img width="644" height="576" alt="image" src="https://github.com/user-attachments/assets/e6376ae4-b2bd-49bb-99a2-639bbdadbd63" />
ContentValues[] products = new ContentValues[10000];

for (int i = 0; i < products.length; i++) {
    ContentValues item = new ContentValues();
    item.put("name", "Product " + i);
    item.put("price", i + 10);
    products[i] = item;
}

int inserted = getContentResolver().bulkInsert(
    Uri.parse("content://com.example.productprovider/products"),
    products
);

Log.d("ProductProvider", "Добавлено строк: " + inserted);


<img width="648" height="552" alt="image" src="https://github.com/user-attachments/assets/b90cb46c-c314-407f-83af-a31b861b971d" />

ContentValues[] products = new ContentValues[10000];

for (int i = 0; i < products.length; i++) {
    ContentValues item = new ContentValues();
    item.put("name", "Product " + i);
    item.put("price", i + 10);
    products[i] = item;
}

int inserted = getContentResolver().bulkInsert(
    Uri.parse("content://com.example.productprovider/products"),
    products
);

Log.d("ProductProvider", "Добавлено строк: " + inserted);



<img width="644" height="576" alt="image" src="https://github.com/user-attachments/assets/13f07461-7b46-4632-a8e9-441bfd90d281" />


class TimedService : Service() {

    private val handler = Handler(Looper.getMainLooper())

    private val stopRunnable = Runnable {
        stopForeground(STOP_FOREGROUND_REMOVE)
        stopSelf()
    }

    override fun onCreate() {
        super.onCreate()

        // Уведомление должно быть создано заранее
        // и передано в startForeground().
        handler.postDelayed(
            stopRunnable,
            30 * 60 * 1000L
        )
    }

    override fun onDestroy() {
        handler.removeCallbacks(stopRunnable)
        super.onDestroy()
    }

    override fun onBind(intent: Intent?): IBinder? = null
}

