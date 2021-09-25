# Android Handler原理 -Handler

------

​	Handler允许发送和处理与线程的MessageQueue相关联Message和Runnable对象。每一个Handler实例都与一个单独的线程和该线程的消息队列相关联。当创建Handler的时候，它被它创建的线程绑定。它把Message或Runnable对象传递到消息队列，出队的时候执行它们。Handler有两个主要用处：1.调度消息和Runnable在未来的某个时间点执行。2.入队一个操作并在不同的线程上执行。

​	调度消息通过post，postAtTime(Runnable,long),postDelayed,sendEmptyMessage,sendMessage,sendMessageAtTime,sendMessageDelayed方法完成。post版本允许入队Runnable对象，sendMessage版本允许入队Message对象，Message对象包含的数据会被Handler的handleMessage方法处理（需要自己实现Handler的子类）。

​	post或send的时候可以允许item在消息队列准备好的时候立即处理，或者定义一个时间延迟或确切的时间处理。

​	当一个应用的进程创建完，它的主线程专注运行一个消息队列,负责管理高级别的应用对象（Activity，BroadcastReceiver等）和他们创建的window，我们可以创建自己的子线程，并通过Handler和主线程通信。这样是通过在子线程里调用post或sendMessage方法完成。然后Runnable或Message将在Handler的消息队列里调度，在合适的时候处理。

### 构造函数

Handler总共有7个构造函数

~~~
#无参构造函数，内部调用6，callback传null，async传false
1. public Handler() {
        this(null, false);
    }
#设置消息处理回调，内部调用6，async传false
2. public Handler(@Nullable Callback callback) {
        this(callback, false);
    }
  #设置Looper，这样就不用调用Looper.prepare方法，内部调用7
3. public Handler(@NonNull Looper looper) {
        this(looper, null, false);
    }
    #设置Looper和回调，内部调用7
4. public Handler(@NonNull Looper looper, @Nullable Callback callback) {
        this(looper, callback, false);
    }
    #设置消息是否异步，内部调用6
5. public Handler(boolean async) {
        this(null, async);
    }
    #设置回调和是否异步，内部检测looper是否为空，所以所有内部调用这个方法的构造函数都需要先调用Looper.prepare方法初始化Looper和MessageQueue
6. public Handler(@Nullable Callback callback, boolean async) {
        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Handler> klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }

        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread " + Thread.currentThread()
                        + " that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
 #设置Looper，回调，是否异步   
7. public Handler(@NonNull Looper looper, @Nullable Callback callback, boolean async) {
        mLooper = looper;
        mQueue = looper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }


~~~

​	Handler总共有7个构造函数，分别应用在不同场景下，可以设置消息是否异步，设置Looper，回调。

### 消息处理 

消息处理使用dispatchMessage方法，在Looper的loop中，获取到Message以后，通过**msg.target.dispatchMessage**调用，msg.target是Handler实例。

~~~
   private static void handleCallback(Message message) {
        message.callback.run();
    }
        public void handleMessage(@NonNull Message msg) {
    }
    
    /**
     * Handle system messages here.
     */
    public void dispatchMessage(@NonNull Message msg) {
        if (msg.callback != null) { //优先处理Message的callback
            handleCallback(msg);
        } else {
            if (mCallback != null) {  //其次处理Handler的Callback对象
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            //msg.callback和Handler的Callback对象都是空的，走Handler的handleMessage方法
            handleMessage(msg);
        }
    }
~~~

从上面可以看出消息处理的优先级是**msg.callback>Handler的mCallback>Handler的handleMessage方法**。msg.callback可以在构造Message的时候设置，mCallback可以在上述构造函数中**2，4，6，7**中设置，handleMessage方法可以在我们继承Handler的时候重写该方法。

### 发送消息 

​	文章开头讲过，发送消息主要有两个版本，post和send。post版本会传递一个Runnable对象，封装成Message的callback，然后加入消息队列，这样post版本会在dispatchMessage的时候，优先执行msg.callback。send版本会直接传递一个Message对象。

​	不论是post版本还是send版本，都会调用Handler的**sendMesssageAtTime**方法：

~~~
 public boolean sendMessageAtTime(@NonNull Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
    }

~~~

​	然后再调用Handler的enqueueMessage方法

~~~
    private boolean enqueueMessage(@NonNull MessageQueue queue, @NonNull Message msg,
            long uptimeMillis) {
        msg.target = this;
        msg.workSourceUid = ThreadLocalWorkSource.getUid();

        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }
~~~

最终调用MessageQueue的enqueueMessage方法，把消息插入到消息队列。

### 移除消息

------

Handler移除消息的方法主要有三种**removeCallback**,**removeMessages**和**removeCallbackAndMessages**

​	**removeCallback**有两个方法，内部都调用了MessageQueue的**removeMessages(Handler h, Runnable r, Object object)**方法：

~~~
   #移除消息队列里post的Runnbale对象是r的Message
   public final void removeCallbacks(@NonNull Runnable r) {
        mQueue.removeMessages(this, r, null);
    }
    #移除消息队列里post的Runnable对象是r且obj是token的Message，如果token是null，则移除Runnable对象是r的Message
     public final void removeCallbacks(@NonNull Runnable r, @Nullable Object token) {
        mQueue.removeMessages(this, r, token);
    }
    #MessageQueue的方法
      void removeMessages(Handler h, Runnable r, Object object) {
        if (h == null || r == null) { 
            return;
        }
        synchronized (this) {
            Message p = mMessages;
            // Remove all messages at front.
            #从队首Message开始查找Message的target是h，且callback是r，且object是空或obj等于object的Message对象，
            找到了就移除并回收该Message，p指向下一个节点
            while (p != null && p.target == h && p.callback == r
                   && (object == null || p.obj == object)) {
                Message n = p.next;
                mMessages = n;
                p.recycleUnchecked();
                p = n;
            }

            // Remove all messages after front.
            #经过上面的循环，如果p不是空，则获取p的下个节点，依然做上述条件查找Message对象，找到了就回收该Message，p的下个节点指向回收对象的下个节点，并继续查找，如果找不到则p指向下个节点，继续循环查找，直到p是null
            while (p != null) {
                Message n = p.next;
                if (n != null) {
                    if (n.target == h && n.callback == r
                        && (object == null || n.obj == object)) {
                        Message nn = n.next;
                        n.recycleUnchecked();
                        p.next = nn;
                        continue;
                    }
                }
                p = n;
            }
        }
    }

~~~

​	**removeMessages**方法有两个，内部调用了MessageQueue的**removeMessages(Handler h, int what, Object object) **方法

~~~
   #移除消息队列里Message.what是what的消息
   public final void removeMessages(int what) {
        mQueue.removeMessages(this, what, null);
    }
    #移除消息队列里Message.what,Message.obj是object的消息，如果object是null，则移除消息队列Message.what是what的消息
 public final void removeMessages(int what, @Nullable Object object) {
        mQueue.removeMessages(this, what, object);
    }
      void removeMessages(Handler h, int what, Object object) {
        if (h == null) {
            return;
        }
        synchronized (this) {
            Message p = mMessages;

            // Remove all messages at front.
             #从队首Message开始查找Message的target是h，且what是what，且object是空或obj等于object的Message对象，
            找到了就移除并回收该Message，p指向下一个节点
            while (p != null && p.target == h && p.what == what
                   && (object == null || p.obj == object)) {
                Message n = p.next;
                mMessages = n;
                p.recycleUnchecked();
                p = n;
            }

            // Remove all messages after front.
             #经过上面的循环，如果p不是空，则获取p的下个节点，依然做上述条件查找Message对象，找到了就回收该Message，p的下个节点指向回收对象的下个节点，并继续查找，如果找不到则p指向下个节点，继续循环查找，直到p是null
            while (p != null) {
                Message n = p.next;
                if (n != null) {
                    if (n.target == h && n.what == what
                        && (object == null || n.obj == object)) {
                        Message nn = n.next;
                        n.recycleUnchecked();
                        p.next = nn;
                        continue;
                    }
                }
                p = n;
            }
        }
    }

~~~

### 主线程的Handler

​	主线程Handler是在ActivityThread的main方法里获取的：

~~~
public static void main(String[] args){
   .....
   Looper.prepareMainLooper();
   ActivityThread thread = new ActivityThread();
        thread.attach(false, startSeq);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
       }
   .....
     Looper.loop();
}
~~~

main方法里先调用Looper.prepareMainLooper方法，再实例化一个ActivityThread，再通过getHandler方法给主线程Handler赋值，再调用Looper.loop方式开启主线程的消息循环。

getHandler方法获取了AcitivityThread的mH字段 

~~~
 final H mH = new H(); 
 final Handler getHandler() {
       return mH;
    }
~~~

H类是ActivityThread的内部类，主要用来处理系统消息比如应用的绑定和退出，Activity打开和生命周期，Service的创建停止等。

~~~
    private class H extends Handler {
        public static final int LAUNCH_ACTIVITY         = 100;
        public static final int PAUSE_ACTIVITY          = 101;
        public static final int PAUSE_ACTIVITY_FINISHING= 102;
        public static final int STOP_ACTIVITY_SHOW      = 103;
        public static final int STOP_ACTIVITY_HIDE      = 104;
        public static final int SHOW_WINDOW             = 105;
        public static final int HIDE_WINDOW             = 106;
        public static final int RESUME_ACTIVITY         = 107;
        public static final int SEND_RESULT             = 108;
        public static final int DESTROY_ACTIVITY        = 109;
        public static final int BIND_APPLICATION        = 110;
        public static final int EXIT_APPLICATION        = 111;
        public static final int NEW_INTENT              = 112;
        public static final int RECEIVER                = 113;
        public static final int CREATE_SERVICE          = 114;
        public static final int SERVICE_ARGS            = 115;
        public static final int STOP_SERVICE            = 116;

        public static final int CONFIGURATION_CHANGED   = 118;
        public static final int CLEAN_UP_CONTEXT        = 119;
        public static final int GC_WHEN_IDLE            = 120;
        public static final int BIND_SERVICE            = 121;
        public static final int UNBIND_SERVICE          = 122;
        public static final int DUMP_SERVICE            = 123;
        public static final int LOW_MEMORY              = 124;
        public static final int ACTIVITY_CONFIGURATION_CHANGED = 125;
        public static final int RELAUNCH_ACTIVITY       = 126;
        public static final int PROFILER_CONTROL        = 127;
        public static final int CREATE_BACKUP_AGENT     = 128;
        public static final int DESTROY_BACKUP_AGENT    = 129;
        public static final int SUICIDE                 = 130;
        public static final int REMOVE_PROVIDER         = 131;
        public static final int ENABLE_JIT              = 132;
        public static final int DISPATCH_PACKAGE_BROADCAST = 133;
        public static final int SCHEDULE_CRASH          = 134;
        public static final int DUMP_HEAP               = 135;
        public static final int DUMP_ACTIVITY           = 136;
        public static final int SLEEPING                = 137;
        public static final int SET_CORE_SETTINGS       = 138;
        public static final int UPDATE_PACKAGE_COMPATIBILITY_INFO = 139;
        public static final int TRIM_MEMORY             = 140;
        public static final int DUMP_PROVIDER           = 141;
        public static final int UNSTABLE_PROVIDER_DIED  = 142;
        public static final int REQUEST_ASSIST_CONTEXT_EXTRAS = 143;
        public static final int TRANSLUCENT_CONVERSION_COMPLETE = 144;
        public static final int INSTALL_PROVIDER        = 145;
        public static final int ON_NEW_ACTIVITY_OPTIONS = 146;
        public static final int ENTER_ANIMATION_COMPLETE = 149;
        public static final int START_BINDER_TRACKING = 150;
        public static final int STOP_BINDER_TRACKING_AND_DUMP = 151;
        public static final int MULTI_WINDOW_MODE_CHANGED = 152;
        public static final int PICTURE_IN_PICTURE_MODE_CHANGED = 153;
        public static final int LOCAL_VOICE_INTERACTION_STARTED = 154;
        public static final int ATTACH_AGENT = 155;
        public static final int APPLICATION_INFO_CHANGED = 156;
        public static final int ACTIVITY_MOVED_TO_DISPLAY = 157;
        ...
        public void handleMessage(Message msg) {
            if (DEBUG_MESSAGES) Slog.v(TAG, ">>> handling: " + codeToString(msg.what));
            switch (msg.what) {
                case LAUNCH_ACTIVITY: {
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityStart");
                    final ActivityClientRecord r = (ActivityClientRecord) msg.obj;

                    r.packageInfo = getPackageInfoNoCheck(
                            r.activityInfo.applicationInfo, r.compatInfo);
                    handleLaunchActivity(r, null, "LAUNCH_ACTIVITY");
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                } break;
                case RELAUNCH_ACTIVITY: {
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityRestart");
                    ActivityClientRecord r = (ActivityClientRecord)msg.obj;
                    handleRelaunchActivity(r);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                } break;
                case PAUSE_ACTIVITY: {
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityPause");
                    SomeArgs args = (SomeArgs) msg.obj;
                    handlePauseActivity((IBinder) args.arg1, false,
                            (args.argi1 & USER_LEAVING) != 0, args.argi2,
                            (args.argi1 & DONT_REPORT) != 0, args.argi3);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                } break;
                case PAUSE_ACTIVITY_FINISHING: {
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityPause");
                    SomeArgs args = (SomeArgs) msg.obj;
                    handlePauseActivity((IBinder) args.arg1, true, (args.argi1 & USER_LEAVING) != 0,
                            args.argi2, (args.argi1 & DONT_REPORT) != 0, args.argi3);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                } break;
                case STOP_ACTIVITY_SHOW: {
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityStop");
                    SomeArgs args = (SomeArgs) msg.obj;
                    handleStopActivity((IBinder) args.arg1, true, args.argi2, args.argi3);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                } break;
                case STOP_ACTIVITY_HIDE: {
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityStop");
                    SomeArgs args = (SomeArgs) msg.obj;
                    handleStopActivity((IBinder) args.arg1, false, args.argi2, args.argi3);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                } break;
                case SHOW_WINDOW:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityShowWindow");
                    handleWindowVisibility((IBinder)msg.obj, true);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case HIDE_WINDOW:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityHideWindow");
                    handleWindowVisibility((IBinder)msg.obj, false);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case RESUME_ACTIVITY:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityResume");
                    SomeArgs args = (SomeArgs) msg.obj;
                    handleResumeActivity((IBinder) args.arg1, true, args.argi1 != 0, true,
                            args.argi3, "RESUME_ACTIVITY");
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case SEND_RESULT:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityDeliverResult");
                    handleSendResult((ResultData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case DESTROY_ACTIVITY:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityDestroy");
                    handleDestroyActivity((IBinder)msg.obj, msg.arg1 != 0,
                            msg.arg2, false);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case BIND_APPLICATION:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "bindApplication");
                    AppBindData data = (AppBindData)msg.obj;
                    handleBindApplication(data);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case EXIT_APPLICATION:
                    if (mInitialApplication != null) {
                        mInitialApplication.onTerminate();
                    }
                    Looper.myLooper().quit();
                    break;
                case NEW_INTENT:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityNewIntent");
                    handleNewIntent((NewIntentData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case RECEIVER:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "broadcastReceiveComp");
                    handleReceiver((ReceiverData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case CREATE_SERVICE:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, ("serviceCreate: " + String.valueOf(msg.obj)));
                    handleCreateService((CreateServiceData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case BIND_SERVICE:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "serviceBind");
                    handleBindService((BindServiceData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case UNBIND_SERVICE:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "serviceUnbind");
                    handleUnbindService((BindServiceData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case SERVICE_ARGS:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, ("serviceStart: " + String.valueOf(msg.obj)));
                    handleServiceArgs((ServiceArgsData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case STOP_SERVICE:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "serviceStop");
                    handleStopService((IBinder)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case CONFIGURATION_CHANGED:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "configChanged");
                    mCurDefaultDisplayDpi = ((Configuration)msg.obj).densityDpi;
                    mUpdatingSystemConfig = true;
                    try {
                        handleConfigurationChanged((Configuration) msg.obj, null);
                    } finally {
                        mUpdatingSystemConfig = false;
                    }
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case CLEAN_UP_CONTEXT:
                    ContextCleanupInfo cci = (ContextCleanupInfo)msg.obj;
                    cci.context.performFinalCleanup(cci.who, cci.what);
                    break;
                case GC_WHEN_IDLE:
                    scheduleGcIdler();
                    break;
                case DUMP_SERVICE:
                    handleDumpService((DumpComponentInfo)msg.obj);
                    break;
                case LOW_MEMORY:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "lowMemory");
                    handleLowMemory();
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case ACTIVITY_CONFIGURATION_CHANGED:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityConfigChanged");
                    handleActivityConfigurationChanged((ActivityConfigChangeData) msg.obj,
                            INVALID_DISPLAY);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case ACTIVITY_MOVED_TO_DISPLAY:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityMovedToDisplay");
                    handleActivityConfigurationChanged((ActivityConfigChangeData) msg.obj,
                            msg.arg1 /* displayId */);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case PROFILER_CONTROL:
                    handleProfilerControl(msg.arg1 != 0, (ProfilerInfo)msg.obj, msg.arg2);
                    break;
                case CREATE_BACKUP_AGENT:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "backupCreateAgent");
                    handleCreateBackupAgent((CreateBackupAgentData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case DESTROY_BACKUP_AGENT:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "backupDestroyAgent");
                    handleDestroyBackupAgent((CreateBackupAgentData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case SUICIDE:
                    Process.killProcess(Process.myPid());
                    break;
                case REMOVE_PROVIDER:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "providerRemove");
                    completeRemoveProvider((ProviderRefCount)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case ENABLE_JIT:
                    ensureJitEnabled();
                    break;
                case DISPATCH_PACKAGE_BROADCAST:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "broadcastPackage");
                    handleDispatchPackageBroadcast(msg.arg1, (String[])msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case SCHEDULE_CRASH:
                    throw new RemoteServiceException((String)msg.obj);
                case DUMP_HEAP:
                    handleDumpHeap((DumpHeapData) msg.obj);
                    break;
                case DUMP_ACTIVITY:
                    handleDumpActivity((DumpComponentInfo)msg.obj);
                    break;
                case DUMP_PROVIDER:
                    handleDumpProvider((DumpComponentInfo)msg.obj);
                    break;
                case SLEEPING:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "sleeping");
                    handleSleeping((IBinder)msg.obj, msg.arg1 != 0);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case SET_CORE_SETTINGS:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "setCoreSettings");
                    handleSetCoreSettings((Bundle) msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case UPDATE_PACKAGE_COMPATIBILITY_INFO:
                    handleUpdatePackageCompatibilityInfo((UpdateCompatibilityData)msg.obj);
                    break;
                case TRIM_MEMORY:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "trimMemory");
                    handleTrimMemory(msg.arg1);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case UNSTABLE_PROVIDER_DIED:
                    handleUnstableProviderDied((IBinder)msg.obj, false);
                    break;
                case REQUEST_ASSIST_CONTEXT_EXTRAS:
                    handleRequestAssistContextExtras((RequestAssistContextExtras)msg.obj);
                    break;
                case TRANSLUCENT_CONVERSION_COMPLETE:
                    handleTranslucentConversionComplete((IBinder)msg.obj, msg.arg1 == 1);
                    break;
                case INSTALL_PROVIDER:
                    handleInstallProvider((ProviderInfo) msg.obj);
                    break;
                case ON_NEW_ACTIVITY_OPTIONS:
                    Pair<IBinder, ActivityOptions> pair = (Pair<IBinder, ActivityOptions>) msg.obj;
                    onNewActivityOptions(pair.first, pair.second);
                    break;
                case ENTER_ANIMATION_COMPLETE:
                    handleEnterAnimationComplete((IBinder) msg.obj);
                    break;
                case START_BINDER_TRACKING:
                    handleStartBinderTracking();
                    break;
                case STOP_BINDER_TRACKING_AND_DUMP:
                    handleStopBinderTrackingAndDump((ParcelFileDescriptor) msg.obj);
                    break;
                case MULTI_WINDOW_MODE_CHANGED:
                    handleMultiWindowModeChanged((IBinder) ((SomeArgs) msg.obj).arg1,
                            ((SomeArgs) msg.obj).argi1 == 1,
                            (Configuration) ((SomeArgs) msg.obj).arg2);
                    break;
                case PICTURE_IN_PICTURE_MODE_CHANGED:
                    handlePictureInPictureModeChanged((IBinder) ((SomeArgs) msg.obj).arg1,
                            ((SomeArgs) msg.obj).argi1 == 1,
                            (Configuration) ((SomeArgs) msg.obj).arg2);
                    break;
                case LOCAL_VOICE_INTERACTION_STARTED:
                    handleLocalVoiceInteractionStarted((IBinder) ((SomeArgs) msg.obj).arg1,
                            (IVoiceInteractor) ((SomeArgs) msg.obj).arg2);
                    break;
                case ATTACH_AGENT:
                    handleAttachAgent((String) msg.obj);
                    break;
                case APPLICATION_INFO_CHANGED:
                    mUpdatingSystemConfig = true;
                    try {
                        handleApplicationInfoChanged((ApplicationInfo) msg.obj);
                    } finally {
                        mUpdatingSystemConfig = false;
                    }
                    break;
            }
            Object obj = msg.obj;
            if (obj instanceof SomeArgs) {
                ((SomeArgs) obj).recycle();
            }
            if (DEBUG_MESSAGES) Slog.v(TAG, "<<< done: " + codeToString(msg.what));
        }
    }
~~~

### HandlerThread

​	主线程的Handler创建的时候，ActivityThread的main方法里自动帮我们调用了Looper.prepareMainLooper和Looper.loop方法，而我们在子线程创建Handler的时候，首先要调用Looper .prepare方法创建looper对象和MessageQueue对象，再创建Handler对象，再调用Looper.loop开启消息循环，否则Hanlder实例化的时候会直接抛出异常。

​	HandlerThread继承自Thread，内部持有Looper和Handler实例，在run方法里自动调用Looper.prepare方法和Looper.loop方法自动准备好消息队列和循环，当首次调用getThreadHandler方法是，才会初始化Handler，使用HanlderThread能够简化在子线程创建Hanlder。免去调用Looper.prepare和Looper.loop方法。



### 总结 

Handler机制是安卓消息循环机制的基石，Handler是Handler机制的指挥者

Handler发送消息方式有两个版本，post和send，post版本发送一个runnable对象，send直接发送Message对象，post和send最终都调用了MessageQueue的enqueueMessage方法。

Handler处理消息的方法是dispatchMessge方法，优先处理msg.callback,其次是Handler的mCallback对象，最后才是handleMessage方法。

Handler移除消息的方法最终都是调用MessageQueue的移除方法。

HandlerThread继承自Thread，简化了在子线程创建Handler的过程，run方法自动调用Looper.prepare和Looper.loop,内部持有的Hanlder对象是懒加载的，首次调用getThreadHandler方法才会去实例化。