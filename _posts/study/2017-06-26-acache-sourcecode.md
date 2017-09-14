---
layout: post
title: Acache 源码
categories: [study]
description: Acache 源码解读
keywords: Acache, 源码
author:     "wenhao"
catalog:    true
tags:
    - study
---
## [ASimpleCache](https://github.com/yangfuhai/ASimpleCache)  
----

 ASimpleCache 是一个为android制定的 轻量级的 开源缓存框架。轻量到只有一个java文件（由十几个类精简而来）。

## 1、它可以缓存什么东西？
普通的字符串、JsonObject、JsonArray、Bitmap、Drawable、序列化的java对象，和 byte数据。

## 2、它有什么特色？
 *  1：轻，轻到只有一个JAVA文件。
 *  2：可配置，可以配置缓存路径，缓存大小，缓存数量等。
 *  3：可以设置缓存超时时间，缓存超时自动失效，并被删除。
 *  4：支持多进程。

## 3、它在android中可以用在哪些场景？
* 1、替换SharePreference当做配置文件
* 2、可以缓存网络请求数据，比如oschina的android客户端可以缓存http请求的新闻内容，缓存时间假设为1个小时，超时后自动失效，让客户端重新请求新的数据，减少客户端流量，同时减少服务器并发量。
* 3、您来说...

## 4、如何使用：  
```
ACache mCache = ACache.get(this);
//存储数据
mCache.put("test_key1", "test value");
//保存10秒，如果超过10秒去获取这个key，将为null
mCache.put("test_key2", "test value", 10);
//保存两天，如果超过两天去获取这个key，将为null
mCache.put("test_key3", "test value", 2 * ACache.TIME_DAY);
//获取数据
String value = mCache.getAsString("test_key1");
```

---
##  通过源码分析特色  

### Acache缓存各种对象
缓存String对象时是直接把String写入File中，缓存JSON对象是调用JSON的toString方法转化为String后再缓存,读取时先读取为String再转化为JSON
缓存byte[]是直接将btye[]写入File中，缓存Bitmap，Drawable,序列化Java对象是将他们转化为byte[]后再缓存,读取时先读取为byte[]再转化为对应的对象


###  可以配置缓存路径，缓存大小，缓存数量等

调用Acache.get(Context)后默认使用Context.getCacheDir()目录下Acache作为缓存路径，默认大小为50M，默认缓存数量为Integer.MAX_VALUE。

```
private static final int MAX_SIZE = 1000 * 1000 * 50; // 50 mb
private static final int MAX_COUNT = Integer.MAX_VALUE; // 不限制存放数据的数量

public static ACache get(Context ctx) {
  return get(ctx, "ACache");
}

public static ACache get(Context ctx, String cacheName) {
  File f = new File(ctx.getCacheDir(), cacheName);
  return get(f, MAX_SIZE, MAX_COUNT);
}

public static ACache get(Context ctx, long max_zise, int max_count) {
  File f = new File(ctx.getCacheDir(), "ACache");
  return get(f, max_zise, max_count);
}

```

Acache最终会调用get(File,long,int)方法完成初始化，我们也可以自己通过
get(File cacheDir, long max_zise, int max_count) 自己配置缓存路径,缓存大小，缓存数量  
在get(File cacheDir, long max_zise, int max_count)方法中会实例化Acache对象,并生成了ACacheManager对象。
ACacheManager中会调用calculateCacheSizeAndCacheCount方法去计算当前缓存目录下文件数量和文件总大小，
并将文件File作为key,文件上次修改时间作为value存储在线程安全的lastUsageDates中
(Collections.synchronizedMap(Map) 会通过装饰模式对Map的每一个方法进行synchronized以达到线程安全)

```
private final Map<File, Long> lastUsageDates = Collections.synchronizedMap(new HashMap<File, Long>());

private void calculateCacheSizeAndCacheCount() {
  new Thread(new Runnable() {
    @Override
    public void run() {
      int size = 0;
      int count = 0;
      File[] cachedFiles = cacheDir.listFiles();
      if (cachedFiles != null) {
        for (File cachedFile : cachedFiles) {
          size += calculateSize(cachedFile);
          count += 1;
          lastUsageDates.put(cachedFile, cachedFile.lastModified());
        }
        cacheSize.set(size);
        cacheCount.set(count);
      }
    }
  }).start();
}

```
至此Acache完成了初始化。
存储数据时调用Acache.put(Key，Value)方法。Acache会把每条存储记录以单独文件的方式存储在缓存目录中。
在put(Key，Value)方法中，Acache首先会调用AcacheManager的newFile()方法以key的Hash值作为文件名称。
```
private File newFile(String key) {
  return new File(cacheDir, key.hashCode() + "");
}
```
Value将以IO的形式存储进文件,存储进文件后将会调用AcacheManager的put(File file)方法
AcacheManager.put(File file)中会检查缓存大小(cacheSize)和缓存数量(cacheCount)值。然后将新文件设置LastModified(最近修改时间)并将File和LastModified存储进lastUsageDates中，注意在Acache中一切对文件的访问都会修改LastModified,包括创建读取等操作
```
private void put(File file) {
    int curCacheCount = cacheCount.get();
    while (curCacheCount + 1 > countLimit) {
        long freedSize = removeNext();
        cacheSize.addAndGet(-freedSize);

        curCacheCount = cacheCount.addAndGet(-1);
    }
    cacheCount.addAndGet(1);

    long valueSize = calculateSize(file);
    long curCacheSize = cacheSize.get();
    while (curCacheSize + valueSize > sizeLimit) {
        long freedSize = removeNext();
        curCacheSize = cacheSize.addAndGet(-freedSize);
    }
    cacheSize.addAndGet(valueSize);

    Long currentTime = System.currentTimeMillis();
    file.setLastModified(currentTime);
    lastUsageDates.put(file, currentTime);
}
```
在上面的put方法中我们可以看到在cacheCount或cacheSize超过设定的值时Acache会循环调用removeNext方法清除部分记录，removeNext()会遍历所有文件的LastModified，找到最久未被访问的文件(LastModified值最小)，将该文件删除并从lastUsageDates中移除。这里其实是LRU算法，(Least recently used最近最少使用算法).
```
private long removeNext() {
  if (lastUsageDates.isEmpty()) {
    return 0;
  }

  Long oldestUsage = null;
  File mostLongUsedFile = null;
  Set<Entry<File, Long>> entries = lastUsageDates.entrySet();
  synchronized (lastUsageDates) {
    for (Entry<File, Long> entry : entries) {
      if (mostLongUsedFile == null) {
        mostLongUsedFile = entry.getKey();
        oldestUsage = entry.getValue();
      } else {
        Long lastValueUsage = entry.getValue();
        if (lastValueUsage < oldestUsage) {
          oldestUsage = lastValueUsage;
          mostLongUsedFile = entry.getKey();
        }
      }
    }
  }

  long fileSize = calculateSize(mostLongUsedFile);
  if (mostLongUsedFile.delete()) {
    lastUsageDates.remove(mostLongUsedFile);
  }
  return fileSize;
}
```
至此Acache实现了可以配置缓存路径，缓存大小，缓存数量


###  可以设置缓存超时时间，缓存超时自动失效，并被删除
使用Acache存储数据时有两种方法put(key,value)和put(key,value,savatime)其中put(key,value,savatime)第三个参数为设置超时时间，单位为秒。超时后数据失效并被删除
```
public void put(String key, String value, int saveTime) {
  put(key, Utils.newStringWithDateInfo(saveTime, value));
}
```
Acache调用了Utils.newStringWithDateInfo()方法对数据进行了封装。newStringWithDateInfo最终调用createDateInfo对当前时间和超时时间进行拼接生成缓存时间信息然后和要存储的信息一起写入缓存文件中
```
public void put(String key, String value, int saveTime) {
  put(key, Utils.newStringWithDateInfo(saveTime, value));
}

private static String newStringWithDateInfo(int second, String strInfo) {
  return createDateInfo(second) + strInfo;
}

private static String createDateInfo(int second) {
  String currentTime = System.currentTimeMillis() + "";
  while (currentTime.length() < 13) {
    currentTime = "0" + currentTime;
  }
  return currentTime + "-" + second + mSeparator;
}
```
在获取数据的时候Acache会调用对应数据格式的get方法，比如读取String记录的时候会调用public String getAsString(String key) 方法。get()通过流的方式读取File中的信息，通过Utils.isDue(readString)判断该记录是否有缓存时间信息，如果有则判断记录是否超时，如果没有则说明该文件不会超时。如果记录未超时则去除缓存信息并返回信息，如果已超时则删除文件并返回null。


###  支持多进程
