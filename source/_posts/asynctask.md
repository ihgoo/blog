title: 浅析AsyncTask
date: 2014-10-10 15:13:35
categories: Android
---
话说今天就要去北京了，现在正坐着高铁。要4个小时才能到，让我们来看下代码打发时间吧。（死电工真是没救了啊）

今天研究的是AsyncTask，这个类在3.0以后源码很大的有改变，所以有必要说一下现在所看的版本，是基于2.3源码来看的。<!--more-->

首先这个类是一个抽象类,要new就需要实现它的抽象方法

```
AsyncTask<Void,Void,Void>.doInBackground(Void...)

```

这个类创建的时候要在UI线程中，也就是主线程，整体工具类的原理是用了Message传递机制。

下面是源码中给出的example：

```
private class DownloadFilesTask extends AsyncTask&lt;URL, Integer, Long&gt; {
      protected Long doInBackground(URL... urls) {
          int count = urls.length;
          long totalSize = 0;
          for (int i = 0; i < count; i++) {
              totalSize += Downloader.downloadFile(urls[i]);
              publishProgress((int) ((i / (float) count)  100));
          }
          return totalSize;
      }
 
      protected void onProgressUpdate(Integer... progress) {
          setProgressPercent(progress[0]);
      }
 
      protected void onPostExecute(Long result) {
          showDialog("Downloaded " + result + " bytes");
      }
}
```

下面是可重载的方法：
onCancelled() -- 在调用cancel方法后会执行
onCancelled(Void result) -- 同上
onPreExecute() -- 在doInBackground之前执行
onPostExecute(Void result) -- 在doInBackground之后执行
onProgressUpdate(Void... values) -- 此方法会更新进度，在asnctask执行后会自动执行。
doInBackground(Void... params) -- 此方法是异步的，耗时的操作都写在此方法中。

整个执行过程都是由一个handler来处理的，message的状态有这么几种：

`
    private static final int MESSAGE_POST_RESULT = 0x1;
`

`    private static final int MESSAGE_POST_PROGRESS = 0x2;
`

`    private static final int MESSAGE_POST_CANCEL = 0x3;
`


asynctask对象创建出来之后还需要执行execute方法才会执行整个过程。在这个方法中，有onPreExecute()方法，是在耗时操作之前执行的。也就是说比odInbackgroud先执行

下面是execute的源码：

```
    public final AsyncTask<Params, Progress, Result> execute(Params... params) {
        if (mStatus != Status.PENDING) {
            switch (mStatus) {
            // 同一个asynctask对象不能被执行两次 执行两次就会抛出以下异常
                case RUNNING:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task is already running.");
                case FINISHED:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task has already been executed "
                            + "(a task can be executed only once)");
            }
        }

		// 将其状态改成running
        mStatus = Status.RUNNING;

		// 执行onPostExecute方法
        onPreExecute();

        mWorker.mParams = params;
        
        // 将任务推到线程池内执行
        sExecutor.execute(mFuture);

        return this;
    }
```

执行sExecutor.execute(mFuture);代码时候， 线程池会执行以下方法(也就是说其中所有代码都在子线程中运行了)

```
 mFuture = new FutureTask<Result>(mWorker) {
            @Override
            protected void done() {
                Message message;
                Result result = null;

                try {
                    result = get();
                } catch (InterruptedException e) {
                    android.util.Log.w(LOG_TAG, e);
                } catch (ExecutionException e) {
                    throw new RuntimeException("An error occured while executing doInBackground()",
                            e.getCause());
                } catch (CancellationException e) {
                    message = sHandler.obtainMessage(MESSAGE_POST_CANCEL,
                            new AsyncTaskResult<Result>(AsyncTask.this, (Result[]) null));
                    message.sendToTarget();
                    return;
                } catch (Throwable t) {
                    throw new RuntimeException("An error occured while executing "
                            + "doInBackground()", t);
                }

                message = sHandler.obtainMessage(MESSAGE_POST_RESULT,
                        new AsyncTaskResult<Result>(AsyncTask.this, result));
                message.sendToTarget();
            }
        };
```

将message发送过去之后，InternalHandler的handlerMessage会进行处理，也就是说odinbackgroud中耗时操作执行完毕后，执行的
`result.mTask.finish(result.mData[0]);`方法，调用了onpostExecute()。代码如下：

```
    private void finish(Result result) {
        if (isCancelled()) result = null;
        onPostExecute(result);
        mStatus = Status.FINISHED;
    }
```

一切变的简单起来，也就是执行顺序如下所示：

onPreExecute -> doInBackground -> onPostExecute

















