# Setting Provider 源码分析

## 源文件：

     SettingProvider.java
分为三各类型的数据，分别存在不同的表中：

```java
private static final String TABLE_SYSTEM = "system";
private static final String TABLE_SECURE = "secure";
private static final String TABLE_GLOBAL = "global";
```

为老版本的使用的类型提供兼容：

```java
private static final String TABLE_FAVORITES = "favorites";
private static final String TABLE_OLD_FAVORITES = "old_favorites";
private static final String TABLE_BLUETOOTH_DEVICES = "bluetooth_devices"; 
...
```

用来标注修改的状态：

```java
private static final int MUTATION_OPERATION_INSERT = 1;
private static final int MUTATION_OPERATION_DELETE = 2;
private static final int MUTATION_OPERATION_UPDATE = 3;
```

设置类型的转换：
在  Settings.java 中定义map,并将各种需要转换的类型加入到这里的集合中

```java
// Per user secure settings that moved to the for all users global settings.
static final Set<String> sSecureMovedToGlobalSettings = new ArraySet<>();

static {
     Setting.Secure.getMovedToGlobalSetting(sSecureMovedToGlobalSettings);
}

// Per user system settings that moved to the for all users global settings.
static final Set<String> sSystemMovedToGlobalSettings = new ArraySet<>();

// Per user system settings that moved to the per user secure settings.
static final Set<String> sSystemMovedToSecureSettings = new ArraySet<>();

// Per all users global settings that moved to the per user secure settings.
static final Set<String> sGlobalMovedToSecureSettings = new ArraySet<>();

// Per user secure settings that are cloned for the managed profiles of the user.
private static final Set<String> sSecureCloneToManagedSettings = new ArraySet<>();

// Per user system settings that are cloned for the managed profiles of the user.
private static final Set<String> sSystemCloneToManagedSettings = new ArraySet<>();
```

回到 SettingsProvider.java

用来保持线程同步的对象

```java
private final Object mLock = new Object();
```

通过 SettingRegistry 对象对设置进行操作

```java
@GuardedBy("mLock")
private SettingsRegistry mSettingsRegistry;

//SettingsRegistry 部分方法
final class SettingsRegistry {
     public List<String> getSettingsNamesLocked(int type, int userId) {...}
     public boolean updateSettingLocked(int type, int userId, String name, String value,
          String packageName, boolean forceNotify) {
            //...
     }
     private void migrateAllLegacySettingsIfNeeded() {
           //...
     }
}
```

UserManager 和 PackageManager

```java
// We have to call in the user manager with no lock held
private volatile UserManager mUserManager;

// We have to call in the package manager with no lock held
private volatile IPackageManager mPackageManager;
```

初始化 UserManager、PackageManager 与 HandlerThread，注册广播接收者

```java
@Override
public boolean onCreate() {
     synchronized (mLock) {
        mUserManager = UserManager.get(getContext());
        mPackageManager = AppGlobals.getPackageManager();
        mHandlerThread = new HandlerThread(LOG_TAG,
        Process.THREAD_PRIORITY_BACKGROUND);
        mHandlerThread.start();
        mSettingsRegistry = new SettingsRegistry();
}
registerBroadcastReceivers();
    startWatchingUserRestrictionChanges();
    return true;
}
private void registerBroadcastReceivers() {
     //user status
    IntentFilter userFilter = new IntentFilter();
    userFilter.addAction(Intent.ACTION_USER_REMOVED);
    userFilter.addAction(Intent.ACTION_USER_STOPPED);
    //...

// package changes
monitor.register(getContext(), BackgroundThread.getHandler().getLooper(),UserHandle.ALL, true);
}
//添加 restriction listener - location provider, install unknow source ,debug feature,
//
startWatchingUserRestrictionChanges(){
     // change the settings affected by restrictions to their current value
     //...

}
```

查询：

```java
@Override
public Cursor query(Uri uri, String[] projection, String where, String[] whereArgs,String order) {
    Arguments args = new Arguments(uri, where, whereArgs, 
    String[] normalizedProjection = normalizeProjection(
    // If a legacy table that is gone, 
    if (REMOVED_LEGACY_TABLES.contains(args.table)) {
        //使用 MatrixCurosr 在没有表的情况下构建一个 Cursor
        return new MatrixCursor(normalizedProjection, 0);
    }
    switch (args.table) {
        case TABLE_GLOBAL: {
            if (args.name != null) {
                Setting setting = getGlobalSetting(args.name);
                return packageSettingForQuery(setting, normalizedProjection);
            } else {
                return getAllGlobalSettings(projection);
            }
        }
        case TABLE_SECURE: {
            final int userId = UserHandle.getCallingUserId();
            if (args.name != null) {
            Setting setting = getSecureSetting(args.name, userId);
                return packageSettingForQuery(setting, normalizedProjection);
            } else {
            return getAllSecureSettings(userId, projection);
            }
        }
    case TABLE_SYSTEM: {
    final int userId = UserHandle.getCallingUserId();
    if (args.name != null) {
    Setting setting = getSystemSetting(args.name, userId);
        return packageSettingForQuery(setting, normalizedProjection);
    } else {
        return getAllSystemSettings(userId, projection);
    }
    }
    default: {
        throw new IllegalArgumentException("Invalid Uri path:" + uri);
        }
    }
}
```

写值：

```java
@Override
public Uri insert(Uri uri, ContentValues values) {
    String table = getValidTableOrThrow(uri);
    // If a legacy table that is gone, done.
    if (REMOVED_LEGACY_TABLES.contains(table)) {
        return null;
    }
    String name = values.getAsString(Settings.Secure.NAME);
    if (!isKeyValid(name)) {
    return null;
    }
    String value = values.getAsString(Settings.Secure.VALUE);
    //对应三各类型的表
    switch (table) {
    case TABLE_GLOBAL: {
    if (insertGlobalSetting(name, value, UserHandle.getCallingUserId(), false)) {
            return Uri.withAppendedPath(Settings.Global.CONTENT_URI, name);
        }
    } break;
    case TABLE_SECURE: {
    if (insertSecureSetting(name, value, UserHandle.getCallingUserId(), false)) {
        return Uri.withAppendedPath(Settings.Secure.CONTENT_URI, name);
    }
    } break;
    case TABLE_SYSTEM: {
    if (insertSystemSetting(name, value, UserHandle.getCallingUserId())) {
        return Uri.withAppendedPath(Settings.System.CONTENT_URI, name);
    }
    } break;
    default: {
        throw new IllegalArgumentException("Bad Uri path:" + uri);
    }
    }
    return null;
}
```

增加现在没有的值：

```java
@Override
public int bulkInsert(Uri uri, ContentValues[] allValues) {
    int insertionCount = 0;
    final int valuesCount = allValues.length;
    for (int i = 0; i < valuesCount; i++) {
    ContentValues values = allValues[i];
    if (insert(uri, values) != null) {
        insertionCount++;
    }
    }
    return insertionCount;
}
```

删除：

```java
@Override
public int delete(Uri uri, String where, String[] whereArgs) {
     ...
}
```

修改：

```java
@Override

public int update(Uri uri, ContentValues values, String where, String[] whereArgs) {
     ...
}
```

根据 uri 获得 电话铃声、通知铃声、闹钟铃声的文件描述对象(FileDescriptor):

```java
public ParcelFileDescriptor openFile(Uri uri, String mode) throws FileNotFoundException {
    final String cacheName;
    if (Settings.System.RINGTONE_CACHE_URI.equals(uri)) {
        cacheName = Settings.System.RINGTONE_CACHE;
    } else if (Settings.System.NOTIFICATION_SOUND_CACHE_URI.equals(uri)) {
        cacheName = Settings.System.NOTIFICATION_SOUND_CACHE;
        } else if (Settings.System.ALARM_ALERT_CACHE_URI.equals(uri)) {
            cacheName = Settings.System.ALARM_ALERT_CACHE;
            } else {
            throw new FileNotFoundException("Direct file access no longer supported; "
            + "ringtone playback is available through android.media.Ringtone");
                }
    final File cacheFile = new File(
    getRingtoneCacheDir(UserHandle.getCallingUserId()), cacheName);
    return ParcelFileDescriptor.open(cacheFile, ParcelFileDescriptor.parseMode(mode));
}
```

   dump?不太清晰什么作用

```java
@Override
public void dump(FileDescriptor fd, PrintWriter pw, String[] args) {
     //...
}
private void dumpSettings(Cursor cursor, PrintWriter pw) {
    //...
}
```