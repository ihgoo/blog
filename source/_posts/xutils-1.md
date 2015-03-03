title: xutils框架源码分析(一)
date: 2014-10-05 14:07:59
categories: Android
---
国庆闲来无事，公司放假了，在家随便研究研究点东西。

xutils是github上面功能完善、比较火的开源框架，是基于afinal开发的，比afinal稳定不少，更新也较为频繁。

###目前xutils有四大模块
- BitmapUtils
- DbUtils
- HttpUtils
- ViewUtils

<!--more-->


首先来看看BitmapUtils，使用了这个，无论从网络还是本地上加载图片的时候，都无需考虑oom异常的问题了。

BitmapUtils这个类共有5大块，首先是它的构造函数

```
    public BitmapUtils(Context context) {
        this(context, null);
    }

    public BitmapUtils(Context context, String diskCachePath) {
        if (context == null) {
            throw new IllegalArgumentException("context may not be null");
        }

        this.context = context.getApplicationContext();
        globalConfig = BitmapGlobalConfig.getInstance(this.context, diskCachePath);
        defaultDisplayConfig = new BitmapDisplayConfig();
    }

    public BitmapUtils(Context context, String diskCachePath, int memoryCacheSize) {
        this(context, diskCachePath);
        globalConfig.setMemoryCacheSize(memoryCacheSize);
    }

    public BitmapUtils(Context context, String diskCachePath, int memoryCacheSize, int diskCacheSize) {
        this(context, diskCachePath);
        globalConfig.setMemoryCacheSize(memoryCacheSize);
        globalConfig.setDiskCacheSize(diskCacheSize);
    }

    public BitmapUtils(Context context, String diskCachePath, float memoryCachePercent) {
        this(context, diskCachePath);
        globalConfig.setMemCacheSizePercent(memoryCachePercent);
    }

    public BitmapUtils(Context context, String diskCachePath, float memoryCachePercent, int diskCacheSize) {
        this(context, diskCachePath);
        globalConfig.setMemCacheSizePercent(memoryCachePercent);
        globalConfig.setDiskCacheSize(diskCacheSize);
    }

```
有多种构造函数，可以在创建实体对象时方便设置全局配置globalConfig和defaultDisplayConfig。

第二个是config方法，可以设置加载状态中的默认图片、加载图片超时时间、是否开启内存缓存、disk缓存、配置线程池的数量等等。

```
    public BitmapUtils configDefaultLoadingImage(Drawable drawable) {
        defaultDisplayConfig.setLoadingDrawable(drawable);
        return this;
    }

    public BitmapUtils configDefaultLoadingImage(int resId) {
        defaultDisplayConfig.setLoadingDrawable(context.getResources().getDrawable(resId));
        return this;
    }

    public BitmapUtils configDefaultLoadingImage(Bitmap bitmap) {
        defaultDisplayConfig.setLoadingDrawable(new BitmapDrawable(context.getResources(), bitmap));
        return this;
    }

    public BitmapUtils configDefaultLoadFailedImage(Drawable drawable) {
        defaultDisplayConfig.setLoadFailedDrawable(drawable);
        return this;
    }

    public BitmapUtils configDefaultLoadFailedImage(int resId) {
        defaultDisplayConfig.setLoadFailedDrawable(context.getResources().getDrawable(resId));
        return this;
    }

    public BitmapUtils configDefaultLoadFailedImage(Bitmap bitmap) {
        defaultDisplayConfig.setLoadFailedDrawable(new BitmapDrawable(context.getResources(), bitmap));
        return this;
    }

    public BitmapUtils configDefaultBitmapMaxSize(int maxWidth, int maxHeight) {
        defaultDisplayConfig.setBitmapMaxSize(new BitmapSize(maxWidth, maxHeight));
        return this;
    }

    public BitmapUtils configDefaultBitmapMaxSize(BitmapSize maxSize) {
        defaultDisplayConfig.setBitmapMaxSize(maxSize);
        return this;
    }

    public BitmapUtils configDefaultImageLoadAnimation(Animation animation) {
        defaultDisplayConfig.setAnimation(animation);
        return this;
    }

    public BitmapUtils configDefaultAutoRotation(boolean autoRotation) {
        defaultDisplayConfig.setAutoRotation(autoRotation);
        return this;
    }

    public BitmapUtils configDefaultShowOriginal(boolean showOriginal) {
        defaultDisplayConfig.setShowOriginal(showOriginal);
        return this;
    }

    public BitmapUtils configDefaultBitmapConfig(Bitmap.Config config) {
        defaultDisplayConfig.setBitmapConfig(config);
        return this;
    }

    public BitmapUtils configDefaultDisplayConfig(BitmapDisplayConfig displayConfig) {
        defaultDisplayConfig = displayConfig;
        return this;
    }

    public BitmapUtils configDownloader(Downloader downloader) {
        globalConfig.setDownloader(downloader);
        return this;
    }

    public BitmapUtils configDefaultCacheExpiry(long defaultExpiry) {
        globalConfig.setDefaultCacheExpiry(defaultExpiry);
        return this;
    }

    public BitmapUtils configDefaultConnectTimeout(int connectTimeout) {
        globalConfig.setDefaultConnectTimeout(connectTimeout);
        return this;
    }

    public BitmapUtils configDefaultReadTimeout(int readTimeout) {
        globalConfig.setDefaultReadTimeout(readTimeout);
        return this;
    }

    public BitmapUtils configThreadPoolSize(int threadPoolSize) {
        globalConfig.setThreadPoolSize(threadPoolSize);
        return this;
    }

    public BitmapUtils configMemoryCacheEnabled(boolean enabled) {
        globalConfig.setMemoryCacheEnabled(enabled);
        return this;
    }

    public BitmapUtils configDiskCacheEnabled(boolean enabled) {
        globalConfig.setDiskCacheEnabled(enabled);
        return this;
    }

    public BitmapUtils configDiskCacheFileNameGenerator(FileNameGenerator fileNameGenerator) {
        globalConfig.setFileNameGenerator(fileNameGenerator);
        return this;
    }

    public BitmapUtils configBitmapCacheListener(BitmapCacheListener listener) {
        globalConfig.setBitmapCacheListener(listener);
        return this;
    }
```

第三个是重头戏，最主要的部分：display，其中有一个方法：


```
 public <T extends View> void display(T container, String uri, BitmapDisplayConfig displayConfig, BitmapLoadCallBack<T> callBack) {
        if (container == null) {
            return;
        }

        // 清除掉动画
        container.clearAnimation();

        // 如果回调函数为null，则创建默认的回调函数
        if (callBack == null) {
            callBack = new DefaultBitmapLoadCallBack<T>();
        }

        // 默认显示配置为null 或者 是使用默认显示配置，就克隆一个新的对象出来
        if (displayConfig == null || displayConfig == defaultDisplayConfig) {
            displayConfig = defaultDisplayConfig.cloneNew();
        }

        // Optimize Max Size
        // 适配到最大的大小
        BitmapSize size = displayConfig.getBitmapMaxSize();
        displayConfig.setBitmapMaxSize(BitmapCommonUtils.optimizeMaxSizeByView(container, size.getWidth(), size.getHeight()));

        // 在图片加载之前进行 回调函数
        callBack.onPreLoad(container, uri, displayConfig);

        // uri为空，执行回调失败函数
        if (TextUtils.isEmpty(uri)) {
            callBack.onLoadFailed(container, uri, displayConfig.getLoadFailedDrawable());
            return;
        }

        // find bitmap from mem cache.
        // 从内存缓存中找bitmap
        Bitmap bitmap = globalConfig.getBitmapCache().getBitmapFromMemCache(uri, displayConfig);

        // 如果在内存缓存中没找到，则重新获取。
        if (bitmap != null) {
            callBack.onLoadStarted(container, uri, displayConfig);
            callBack.onLoadCompleted(
                    container,
                    uri,
                    bitmap,
                    displayConfig,
                    BitmapLoadFrom.MEMORY_CACHE);
        } else if (!bitmapLoadTaskExist(container, uri, callBack)) {

// 创建bitmaploadtask对象
            final BitmapLoadTask<T> loadTask = new BitmapLoadTask<T>(container, uri, displayConfig, callBack);

            
            
            // get executor
            PriorityExecutor executor = globalConfig.getBitmapLoadExecutor();
            // 得到硬盘缓存的File对象
            File diskCacheFile = this.getBitmapFileFromDiskCache(uri);
            boolean diskCacheExist = diskCacheFile != null && diskCacheFile.exists();
            if (diskCacheExist && executor.isBusy()) {
                executor = globalConfig.getDiskCacheExecutor();
            }
            // set loading image
            Drawable loadingDrawable = displayConfig.getLoadingDrawable();
            callBack.setDrawable(container, new AsyncDrawable<T>(loadingDrawable, loadTask));

            loadTask.setPriority(displayConfig.getPriority());
            loadTask.executeOnExecutor(executor);
        }
    }
```

其中BitmapDisplayConfig参数并不是必须有的，代码检测到displayconfig为null的话就会创建一个默认的displayconfig。
有自己的BitmapLoadCallBack参数，如果不传入也会创建一个新的默认回调的参数供其回调使用。

```
// 默认的bitmapLoadCallBack类,重载了onLoadCompleted和onLoadFailed方法
public class DefaultBitmapLoadCallBack<T extends View> extends BitmapLoadCallBack<T> {

// 加载完成时候显示图片和动画
    @Override
    public void onLoadCompleted(T container, String uri, Bitmap bitmap, BitmapDisplayConfig config, BitmapLoadFrom from) {
        this.setBitmap(container, bitmap);
        Animation animation = config.getAnimation();
        if (animation != null) {
            animationDisplay(container, animation);
        }
    }

// 当加载图片失败时候显示失败图片
    @Override
    public void onLoadFailed(T container, String uri, Drawable drawable) {
        this.setDrawable(container, drawable);
    }
    
    private void animationDisplay(T container, Animation animation) {
        try {
            Method cloneMethod = Animation.class.getDeclaredMethod("clone");
            cloneMethod.setAccessible(true);
            container.startAnimation((Animation) cloneMethod.invoke(animation));
        } catch (Throwable e) {
            container.startAnimation(animation);
        }
    }
}
```

先从内存缓存中找bitmamp对象，如果找不到就重新加载获取，其中有一个PriorityAsnctask类,是图片异步任务的管理者，管理着图片加载的顺序。
在内部类BitmapLoadTask的doInbackground中，重新获取了bitmap对象。代码如下所示：

```
 @Override
        protected Bitmap doInBackground(Object... params) {

            synchronized (pauseTaskLock) {
                while (pauseTask && !this.isCancelled()) {
                    try {
                        pauseTaskLock.wait();
                        if (cancelAllTask) {
                            return null;
                        }
                    } catch (Throwable e) {
                    }
                }
            }

            Bitmap bitmap = null;

            // get cache from disk cache
            // 从disk cache寻找对应的bitmap对象
            if (!this.isCancelled() && this.getTargetContainer() != null) {
                this.publishProgress(PROGRESS_LOAD_STARTED);
                bitmap = globalConfig.getBitmapCache().getBitmapFromDiskCache(uri, displayConfig);
            }
            
			// 如果上一步找不到则下载image
            // download image
            if (bitmap == null && !this.isCancelled() && this.getTargetContainer() != null) {
            // 是由bitmapCache 中的方法控制下载bitmap对象
                bitmap = globalConfig.getBitmapCache().downloadBitmap(uri, displayConfig, this);
                from = BitmapLoadFrom.URI;
            }

            return bitmap;
        }

```


代码中的注释已经说了，是由bitmapCache类中方法下载的bitmap对象，让我们到这个类里面看看是如何下载管理的

```
    public Bitmap downloadBitmap(String uri, BitmapDisplayConfig config, final BitmapUtils.BitmapLoadTask<?> task) {

        BitmapMeta bitmapMeta = new BitmapMeta();

        OutputStream outputStream = null;
        LruDiskCache.Snapshot snapshot = null;

        try {
            Bitmap bitmap = null;
            // try download to disk
            if (globalConfig.isDiskCacheEnabled()) {
                synchronized (mDiskCacheLock) {
                    // Wait for disk cache to initialize
                    while (!isDiskCacheReady) {
                        try {
                            mDiskCacheLock.wait();
                        } catch (Throwable e) {
                            break;
                        }
                    }
                }

                if (mDiskLruCache != null) {
                    try {
                    	// 取diskLru缓存中的快照，如果取不到，则重新下载至diskLru中，重新再取
                        snapshot = mDiskLruCache.get(uri);
                        if (snapshot == null) {
                            LruDiskCache.Editor editor = mDiskLruCache.edit(uri);
                            if (editor != null) {
                                outputStream = editor.newOutputStream(DISK_CACHE_INDEX);
                                bitmapMeta.expiryTimestamp = globalConfig.getDownloader().downloadToStream(uri, outputStream, task);
                                if (bitmapMeta.expiryTimestamp < 0) {
                                    editor.abort();
                                    return null;
                                } else {
                                    editor.setEntryExpiryTimestamp(bitmapMeta.expiryTimestamp);
                                    editor.commit();
                                }
                                snapshot = mDiskLruCache.get(uri);
                            }
                        }
                        // 取出快照后，decode转成bitmap对象
                        if (snapshot != null) {
                            bitmapMeta.inputStream = snapshot.getInputStream(DISK_CACHE_INDEX);
                            bitmap = decodeBitmapMeta(bitmapMeta, config);
                            if (bitmap == null) {
                                bitmapMeta.inputStream = null;
                                mDiskLruCache.remove(uri);
                            }
                        }
                    } catch (Throwable e) {
                        LogUtils.e(e.getMessage(), e);
                    }
                }
            }

            // 这一步是如果diskLru缓存方式没开启，就下载到内存中
            // try download to memory stream
            if (bitmap == null) {
                outputStream = new ByteArrayOutputStream();
                bitmapMeta.expiryTimestamp = globalConfig.getDownloader().downloadToStream(uri, outputStream, task);
                if (bitmapMeta.expiryTimestamp < 0) {
                    return null;
                } else {
                    bitmapMeta.data = ((ByteArrayOutputStream) outputStream).toByteArray();
                    bitmap = decodeBitmapMeta(bitmapMeta, config);
                }
            }

            if (bitmap != null) {
                bitmap = rotateBitmapIfNeeded(uri, config, bitmap);
                // 加到内存缓存中
                bitmap = addBitmapToMemoryCache(uri, config, bitmap, bitmapMeta.expiryTimestamp);
            }
            return bitmap;
        } catch (Throwable e) {
            LogUtils.e(e.getMessage(), e);
        } finally {
            IOUtils.closeQuietly(outputStream);
            IOUtils.closeQuietly(snapshot);
        }

        return null;
    }
 ```
 
 作者用了双缓存（diskLru和内存缓存）来管理的图片，对图片的大小也统一进行decode，避免了大图片加载时出现OOM异常的问题。


下面是一段为什么加载大图片会出现OOM异常的解答


>一张图片如果是1280*800像素的，颜色深度按ARGB8888来算，一个像素点大概会占用8字节的内存，那么1280*800*8就是8192000字节，也就是8000K，也就是7.8M，这张图片读到内存就会占用这么大,如果内存仅有16MB，加载3张这样的图片就会OOM异常了。

>这个大小是硬盘占用空间，和读到内存里的大小是两回事。
不同颜色深度占用的字节数也不一样的，ARGB8888是占用的最多的，如果你想让图片小一点，可以用RGB256这样的，一个像素只占4个字节，但颜色就不如前者要好。

>所以一张图片占用多少内存也要看它在界面上显示的效果怎么样，我们平时压缩图片其实就是牺牲了效果，换来了更小的内存。
