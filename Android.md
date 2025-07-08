### Intro

Level-based architecture:

- **Kernel Based**: Based on Linux, it introduces a **Hardware Abstraction Layer (HAL)** that allows apps to run on many different devices.
- **Libraries**: Native C/C++ code
- **Android Runtime (ART)**: Application execution environment written in Java. Before it was the Dalvik VM.
- **Application framework**: Support through Java objects
- **Application**: Native applications

|DVM|ART|
|---|---|
|Application lagging due to garbage collector pauses and JIT|Reduced application lagging and better user experience|
|App installation time is comparatively lower as the compilation is performed later|App installation time is longer as compilation is done during installation|
|DVM converts bytecode every time you launch a specific app.|ART converts it just once at the time of app installation. That makes CPU execution easier.|
|It is slower than ART.|It is faster.|
|It does not provide optimized battery life as it consumes more power.|It provides optimized battery performance as it consumes less power.|

##### Note on the versions:

- Android 4.4: Introduced ART, then in 5.0 made it the default and only option.
- Android 6.0: Introduced request for user permission after installation/execution
- Android 7.0: Java on OpenJDK
- ...

Over time, the direction is to limit the freedom of the user to enhance the security and stability of the system.

### Kernel Level

The kernel level is based on Linux 3.x merged with the **Hardware Abstraction Layer (HAL)**. Actually, the Linux kernel lacks a native window manager, GNU C support and the standard Linux utilities, and introduces some extensions:

- **Shared memory manager (Ashmem)**: It has reference counting and automatic deallocation.
- **Binder IPC**: Inter-process communication. Allows communication between distinct processes and apps. Apps use this service also to access system services (such as the NotificationManager).
- **WakeLocks**: Battery policy manager.

HAL provides all the support for custom hardware:

- Memory management
- Process management
- Network stack

#### WakeLocks

They are locks to access the functions of the Power Manager. If an app has permission to access them, it can choose the policy for energy consumption. For example: the screen brightness or the CPU usage.

This is the perfect **example of the Cross-layering pattern**: the application level can directly work on the bottom hardware layers to control battery usage.

### Native Libraries and Dalvik VM

Native libraries are designed for heavy tasks like graphics and multimedia services and are accessible through APIs. They mask the lower layer of system services.

The **Dalvik VM** is a JVM machine optimized for mobile devices; for this reason it's register-based instead of the standard stack-based. Due to this particularity, the Dalvik VM requires that the source codes are compiled JIT for the device. The Dalvik VM also comes with its own bytecode (_dex_), but it's actually used only as a universal language between Android devices; it needs to be compiled. The compiled code doesn't get cached, so at each execution the device pays the energy cost to use a JIT compiler.

The new Android Runtime introduced ahead-of-time compiling at installation time, making the stored source code heavier, but saving the energy cost of using the JIT compiler.

### Application Framework

#### Components

Application level:

- **Activity**: Every action a user can make through a GUI. Usually it's a single screen.
- **Service**: It runs in the background and has no interaction with a user. Can work with multiple activities (even when the user is not directly interacting with the application) and can have a dedicated thread or work in the main thread.
- **Intent**: Application interaction enabler. To receive an Intent, a component should have a compatible Intent Filter.
- **Broadcast Receiver**: Receives and responds to compatible Intents. Its life cycle is bound to a single Intent. (An intent can activate several Broadcast Receivers.)

Support Level:

- **Package**: An APK contains the _dex_ executable files, the manifest (an _XML_ file that describes the application), static resources (png, style sheets, etc). All in a well-defined structure.
- **Activity Manager**: Manages the life cycle of activities.
- **Window Manager and View system**: Advanced graphic functions. The view system provides GUI components to interact with users and events. Developer defines a hierarchy of views describing the GUI, then the View system draws them on the screen. It also handles events and touches. The window manager handles the application in windows when not in full screen.
- **Resource manager** and **content provider**: The first handles all the resources (non-code) in the `./res` folder of the project. The latter allows access to structured data through SQLite DB; the data can also be shared with others (like media files).
- **Telephony, Notification and Location Manager**

##### Base Applications

Applications preinstalled on the device. The most common are:

- Home: single thread application to launch other apps
- Browser: web-based on WebKit

### Activities Life Cycle

States of an activity:

- `RUNNING` state: only one at a time and it is the one the user is interacting with.
- `PAUSED`: not active but still visible on screen.
- `STOPPED`: not active and not visible
- `KILLED`: not enough resources available

All the resources are usually dedicated to the `RUNNING` activity. The apps that have been deallocated are in the **Bundle** information holder that keeps the state to recover.

Life cycle management through callback methods:

- `OnCreate`
- `OnStart`
- `OnResume`
- `OnDestroy`

### Task

An application can be made of multiple activities. A task models the interaction between Activities and keeps a **LIFO** stack of them. The uppermost is the running one. When we press back we remove the activity from the stack. When we press home we move between different tasks.

### Intent and Intent Filter

Allows communication and exchange of messages between components, like activities and tasks.

- **Explicit**: the class to call is known at compile time `new Intent("ID", MyActivity.class)`
- **Implicit**: the class to call is to be discovered; the user selects the app to call from a menu. The intent has to specify:
    - Action and Category
    - URI of the data to pass
    - MIME Type: specifies the type of the data. 
    The activities are selected with an algorithm of Intent resolution based on the intent filters declared in the manifest.
```Java
<activity android:name="IntentActivity">
	<intent-filter>
		<action android:name="android.intent.action.MAIN" />
		<category android:name="android.intent.category.LAUNCHER" />
		<category android:name="it.mypackage.intent.category.CAT_NAME" />
		<data android:mimeType="vnd.android.cursor.item/vnd.mytype" />
	</intent-filter>
</activity>

protected void onCreate(Bundle savedInstanceState) {
...
	Intent intent = new Intent();
	intent.setAction(MY_ACTION);
	intent.addCategory(Intent.CATEGORY_ALTERNATIVE);
	intent.addCategory(Intent.CATEGORY_BROWSABLE);
	Uri uri = Uri.parse("content://it.mypackage/items/");
	intent.setData(uri);
	intent.setType("vnd.android.cursor.item/vnd.mytype");
	startActivity(intent); //Intent Resolution Algorithm 
...
}
```
### Threading

Every application is associated with a single thread that runs each different activity (**single thread model**). Each thread has an endless loop to handle the message queue: external events (system or user inputs) or internal from the activities.

Options:

- Different Application in a single Dalvik VM (one heavy process)
- Dedicated Dalvik for each application: default option even if started by an Intent with `startActivity(Intent)` or `startService(Intent)`.

> If the caller wants a result it should use `startActivityForResult(Intent,int)`. The int serves as an ID for the asynchronous callback.

If two apps want to use the same VM it must be configured at compile time by setting the _`android:sharedUserId`_ in the manifest.

#### Scheduling

The developer has no direct control over scheduling; it is completely delegated to the Runtime. The priorities are:

- Running Activity
- Visible Activity
- Services
- Background

An app's priority is the same as the highest priority of one of its activities. Typically the CPU is assigned to the running activity for 90% of the time.

## Async Techniques

For the execution of application logic:

- Thread
- Executor
- Handler
- AsyncTask
- Service
- IntentService

### Service

Executes long-term operations in the background. It does not provide any user interface and can't modify the UI. It gets activated by an Intent.

Even though services are split from the UI, they still run on the main thread by default.

Examples of services: Network transactions, Audio streaming, I/O on files, DB interactions.

- **Unbound/Started Service**: Executes indefinitely until it reaches completion. Life cycle managed by the developer. `onStartCommand()`
- **Bound Service**: Client/Server interface. Clients send requests and receive results. Started with `bindService` and terminates when all clients disconnect. 
	- `onBind()`
	- `onUnbind()`
- **Foreground Service**: Needs user acknowledgement, like a music player. Higher priority than background; the user would notice their absence. They have a notification that can't be deleted while the service is working (battery policy).

If a service stops responding it blocks the main thread and also the UI, so it is useful to split into a different thread.

With a Broadcast receiver, services and activities can share information.

If an app goes to the background it also blocks the services; a background app can't start a service and gets back an `IllegalStateException`.

#### IntentService

Easier life cycle. Uses a worker thread to handle requests and stops when it is done. Ideal for a long job on a single thread in the background.

Just one request at a time and it can't be stopped.

```java
public class HelloIntentService extends IntentService{....
@Override 
protected void onHandleIntent(Intent intent){...} ...}
```


### Threads

Simple Java threads, but they can't operate directly on the GUI, since **Android GUI libraries are not thread-safe.**

They cannot be stopped with `destroy()` or `stop()`, but you must use:
- `interrupt()`
- `join()`

To use it:
- Write a class that extends Thread and overrides `run()`
- Give a thread instance an Object `Runnable` (implements the interface)
Then just call the method `start()`.
### Handler
New way to communicate between the main thread and new threads. It is a channel to send messages or executable code (an anonymous class that implements `Runnable`) to a thread.
- `post()`
- `postDelayed()`
- `sendMessage()`
- `sendMessageDelayed()`
- `removeCallbacks()`
- `removeMessages()`
```java
    @Override
    public void run() {
        // Prepare the Looper for this thread
        Looper.prepare();
        // Create a Handler associated with this thread's Looper
        this.handler = new Handler(Looper.myLooper()) {
            @Override
            public void handleMessage(Message msg) {
                // Handle the message received from another thread
                String data = (String) msg.obj;
                Log.d("TargetThread", "Received message: " + data);
            }
        };
        // Start the Looper
        Looper.loop();
    }
	public Handler getHandler() {
        return this.handler;
    }
```
### AsyncTask

Services can't update the UI. AsyncTasks run in a background thread and when they end they can update the UI to show results.

- `Params`: parameters
- `Progress`: unit size of the progress bar
- `Result`: result type expected

There are 4 callbacks to handle the life cycle:

- `onPreExecute()`: called from the main thread to show a progress bar
- `doInBackground(Params)`: application logic, works alongside the main thread
- `onProgressUpdate(Progress)`: the background worker called `publishProgress`
- `onPostExecute(Result)`: called from the main thread to show the results on the UI

Since UI handling can become messy and nested code is painful, this solution can ease the work and solve concurrency problems.

### Broadcast

Messages sent from the Android system or other applications. The message is wrapped in an Intent.

- `android.intent.action.HEADSET_PLUG` when headphones are inserted
- `ACTION_BOOT_COMPLETED` when the device is booted
- `ACTION_POWER_CONNECTED` when the device is charging

Useful to be aware of the context of the application, like when WiFi is connected or similar events.

The developer can code custom broadcasts for the application:

- `Ordered Broadcast`: The receivers have a priority number and receive the message sequentially. A receiver can propagate or abort the broadcast.
	- `sendOrderedBroacast(Intent, Permissions)`
- `Normal broadcast`: The receivers don't have a priority; they can't share results with each other. 
	- `sendBroacast(Intent, Permissions)`
- `Local broadcast`: Message sent and received inside the application, created with a `LocalBroadcastManager.getInstance(this).sendBroadcast(Intent)`.

The custom broadcast must have a common name between the Activity and the Broadcast Receiver to narrow the visibility of the intent inside the app. It's important to limit the reception of the broadcast for security reasons; the field `setPackage()` is also available to limit the broadcast to the selected application.

### Broadcast Receiver

They subscribe to receive broadcasts (**of any kind**):

- **Statically**: through the manifest
- **Dynamically**: Using a context or an Activity

The reception of the broadcast has a timeout; if it expires, the application is considered blocked.

To write a broadcast receiver:

- Register it:
	- **Dynamic**: `this.registerReceiver(mReceiver, filter);`
	- **Static:**`<receiver android:name=".MyBroadcastReceiver"> ` 
- Override the `onReceive()` method

In some cases it should specify a permission (just nominal) to allow the receiver to subscribe to the selected broadcast.

### Alarm

Software component that starts the execution of operations at selected times by sending Intents. It is based on real-time clock and also supports elapsed time. The application can be inactive or the device can be sleeping to receive the alarms.

Alarms are usually used in tandem with Broadcast Receivers that capture the Intents.

1. An activity creates a notification and schedules an Alarm
2. An Alarm triggers and sends the Intent
3. The broadcast receiver captures the Intent
4. The broadcast receiver shows the notification on screen

| |Elapsed Real Time (ERT) - since boot|Real Time Clock (RTC)|
|---|---|---|
|do not wake up|`ELAPSED_REALTIME`|`RTC`|
|wake up|`ELAPSED_REALTIME_WAKEUP`|`RTC_WAKEUP`|

Waking up the CPU with the screen turned off should only be for critical operations; it drains the battery. If the alarm is programmed to not wake up, it rings when the screen is turned on.

Using RTC can also cause a burst of requests to remote servers, so using ERT without waking up can add some randomness that can help the servers.

The Alarm also provides `setInexactRepeating()` that helps to merge a series of upcoming notifications together. This helps because it activates the CPU less frequently.

Exact time notifications are also avoided for real-time clock; now the notifications arrive grouped together and the developer can specify time windows.

### Internet Connections

To use the internet, the app must specify the permissions:

- `<uses-permission android:name="android.permission.INTERNET">`
- `<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE">`

The `ConnectivityManager` answers questions about network connection and its changes through the data structure `NetworkInfo`.

- `AsyncTask`: small downloads
- `AsyncTaskLoader`: larger downloads
- `HttpURLConnection`: to establish a new connection on a new Thread

Usually the results are parsed in `onPostExecute()` using helper classes.

#### Efficient Data Transfers

The radio interface can work at 3 levels:

- **Full power**
- **Low power**: 50% of full power consumption
- **Standby**: no connections available

The Android middleware merges multiple requests in the same transmission to leave the radio interface on standby for as long as possible.

The Battery Manager sends updates on power and battery using intents, so a developer with a Broadcast Receiver can choose the best strategy.

### Job Scheduler

Smart scheduling of heavy background tasks. Condition-based, not time-based. Much more efficient than `AlarmManager`; the scheduler groups tasks together to save battery.

- `JobService`: It starts the task, runs on the main thread
    - `onStartJob(JobParams)`: Called when conditions are reached. It uses child threads to run heavy jobs. Returns `FALSE` if the job ended, `TRUE` if it is running in another thread (it calls `jobFinished()` when done)
    - `onStopJob()`: If the Android system decides to stop the job because the conditions are not met anymore, it plans the job for the future. Calls `jobFinished(Params, boolean)`
- `JobInfo`: Sets the execution task's conditions. Takes the job ID, the service that runs the `JobService` and the `JobService` to run.
- `JobScheduler`: Does the task scheduling. Get one from the system and call `schedule(JobInfo)`

## Security

Each Dalvik VM has its own UID and GID. App permissions come from the UID, GID and from the manifest.

## iOS

It is based on a version of XNU, has Apple's API and full integration with Apple's environment. The SDK provided has all the common services: location manager, events, SQLite DB integrated, threading, etc.

Apple also provides an iPhone simulator for x86 devices.

iOS has many more rules and has a tighter policy for developers to increase the security of the system. The developer can only use Apple's API. The applications can be written in: Objective-C, C, C++ or JavaScript (run on iOS WebKit). The software must be compiled and can't run any interpreted code, plug-ins, downloaded code, etc. Each executable can be downloaded only from the Apple Store and it is signed to run only on the specific device.

A workaround for developers is developing web apps to run on Safari, but this limits the use of sensors and hardware.

### Unlocking and Jailbreaking

- **Unlocking**: Opening the device to work with every telecom provider
- **Jailbreaking**: Breaking the producer policies by gaining root access on the device. This allows running arbitrary code on the system, bypassing the security constraints.

## Cross Platform

_Write once, run everywhere_. Typically it's done by using HTML to develop the UI.

### Xamarin

It allows developing applications for Android, iOS, Windows with native UI using the same application logic. can a device know when the control point leaves, and stop sending event messages?













