# Agora Video With ARCore

This tutorial enables you to quickly get started in your development efforts to create an Android app with live video chat using ARCore and the Agora Video SDK. With this sample app you can:

- Send a captured image from ARCore to the live video channel
- Render video frames of a remote user in an ARCore session

## Prerequisites

* Android Studio 3.0.1 or above.
* Android SDK Platform version 7.0 (API level 24) or above.
* Physical [Android device with ARCore support](https://developers.google.com/ar/discover/) (e.g. Nexus 5X). A real device is recommended because some simulators have missing functionality or lack the performance necessary to run the sample.

## Quick Start
This section shows you how to prepare, build, and run the sample application.

### Create an Account and Obtain an App ID
In order to build and run the sample application you must obtain an App ID: 

1. Create a developer account at [agora.io](https://dashboard.agora.io/signin/). Once you finish the signup process, you will be redirected to the Dashboard.
2. Navigate in the Dashboard tree on the left to **Projects** > **Project List**.
3. Locate the file **app/src/main/res/values/strings.xml** and replace <#YOUR APP ID#> with the App ID in the dashboard.

```xml
<string name="private_broadcasting_app_id"><#YOUR APP ID#></string>
```

### Integrate the Agora Video SDK into the sample project
The SDK must be integrated into the sample project before it can opened and built. There are two methods for integrating the Agora Video SDK into the sample project. The first method uses JCenter to automatically integrate the SDK files. The second method requires you to manually copy the SDK files to the project.

**Note:** Custom media device protocols are available in the Agora Video SDK 2.1.0 and above.

#### Method 1 - Integrate the SDK Automatically Using JCenter (Recommended)

1. Clone this repository.
2. Open **app/build.gradle** and add the following line to the `dependencies` list:

```
...
dependencies {
    ...
    compile 'io.agora.rtc:full-sdk:2.1.0' 
}
```

#### Method 2 - Manually copy the SDK files

1. Clone this repository.
2. Download the Agora Video SDK from [Agora.io SDK](https://www.agora.io/en/download/).
3. Unzip the downloaded SDK package.
4. Copy the `agora-rtc-sdk.jar` file from the **libs** folder of the downloaded SDK package to the **/apps/libs** folder of the sample application.
5. Copy the .so files from the **armeabi-v7a** folder of the downloaded SDK package to the **/app/src/main/jniLibs/armeabi-v7a** folder of the sample application.


### Using the Sample Application

This sample application requires two devices, and works in conjunction with the [OpenLive](https://github.com/AgoraIO/OpenLive-Android) sample application. 

1. Run the sample application in Android Studio. Move the device until you find a horizontal surface.
 
2. Touch the plane indicator to add a virtual display screen to your AR session. The virtual display screen streams the video from the remote user.

3. On a different device, launch the [OpenLive](https://github.com/AgoraIO/OpenLive-Android) sample application using the same App ID used in this project, and join the channel `arcore` as a broadcaster.

	The virtual display screen from step 2 displays the video broadcast sent from the remote user's [OpenLive](https://github.com/AgoraIO/OpenLive-Android) application.

## Steps to Create the Sample 

- [Set Permissions](#set-permissions)
- [Create Visual Assets and Design the User Interface](#create-visual-assets-and-design-the-user-interface)
- [Declare the Activity Class and Define Global Variables](#declare-the-activity-class-and-define-global-variables)
- [Create Screen and Permission Override Methods](#create-screen-and-permission-override-methods)
- [Create Interaction and Rendering Override Callbacks](#create-interaction-and-rendering-override-callbacks)
- [Create the initRtcEngine() Method](#create-the-initrtcengine-method)
- [Create Message Handling Methods](#create-message-handling-methods)
- [Create the addRemoteRender() Method](#create-the-addremoterender-method)
- [Set Permission Variables and Methods](#set-permission-variables-and-methods)
- [Add Button Methods](#add-button-methods)
- [Add App Setting Retrieval Methods](#add-app-setting-retrieval-methods)

For details about the APIs used to develop this sample, see the [Agora.io Documentation version 2.2](https://docs.agora.io/en/2.2).

### Set Permissions

In the `AndroidManifest.xml` file, `uses-permissions` settings are added for the camera, Internet, audio recording, audio settings, network state, external storage, and Bluetooth to allow the app to access these features:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          xmlns:tools="http://schemas.android.com/tools"
    package="com.agora.arcore">

    <!-- This tag indicates that this application requires ARCore.  This results in the application
       only being visible in the Google Play Store on devices that support ARCore. -->
    <uses-feature android:name="android.hardware.camera.ar" android:required="true"/>

    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.BLUETOOTH" />

    <application
        android:allowBackup="false"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme"
        android:usesCleartextTraffic="false"
        tools:ignore="GoogleAppIndexingWarning">


        <activity
            android:name=".AgoraARCoreActivity"
            android:label="@string/app_name"
            android:configChanges="orientation|screenSize"
            android:exported="true"
            android:theme="@style/AppTheme"
            android:screenOrientation="locked">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <!-- This tag indicates that this application requires ARCore.  This results in the Google Play
         Store downloading and installing ARCore along with the application. -->
        <meta-data android:name="com.google.ar.core" android:value="required" />
    </application>
</manifest>
```

### Create Visual Assets and Design the User Interface

Add the `ic_launcher.png` icon asset to the */res/drawable* folder. This is a desktop icon for users to invoke the sample application.

The sample contains a single activity called *AgoraARCoreActivity* and its layout is defined in */layout/activity_main.xml*. 

Component|Description
----|----
`surfaceview`|An OpenGL view that handles the AR.
`LinearLayout (unamed)`|A vertical layout that encapsulates five buttons: **AR Mode**, **Show Point**, **Show Plane**, **Zoom In**, and **Zoom Out**. The button details are listed below.
`switch_mode`|A button that handles switching between AR modes.
`show_point_cloud`|A button that handles display of the points on the screen.
`show_plane`|A button that handles display of the plane on the screen.
`zoom_in`|A button that handles zoom in and out.


### Import Agora Resources

The remaining code samples are in the [`AgoraARCoreActivity.java`](Agora-ARCore/app/src/main/java/com/agora/arcore/AgoraARCoreActivity.java) file.

This section defines the key imports for the Agora SDK.

The following imports define the Agora API that provides AR rendering functionality: 

- `import com.agora.arcore.rendering.BackgroundRenderer`
- `import com.agora.arcore.rendering.ObjectRenderer`
- `import com.agora.arcore.rendering.PeerRenderer`
- `import com.agora.arcore.rendering.PlaneRenderer`
- `import com.agora.arcore.rendering.PointCloudRenderer`

The following imports define the Agora API that provides Agora RTC Engine functionality: 

- `import io.agora.rtc.Constants`
- `import io.agora.rtc.IRtcEngineEventHandler`
- `import io.agora.rtc.RtcEngine`
- `import io.agora.rtc.mediaio.MediaIO`


### Declare the Activity Class and Define Global Variables

The `AgoraARCoreActivity` class extends the `AppCompatActivity` class and uses protocol methods from `GLSurfaceView.Renderer`.

The remaining code samples are within the `AgoraARCoreActivity` class.

``` Java
public class AgoraARCoreActivity extends AppCompatActivity implements GLSurfaceView.Renderer {
    
    ...
    
}
```

The `AgoraARCoreActivity` class declares several global variables for use in the sample application.

The following private variables manage Open GL rendering, an installation flag, the AR session, gesture and message handling.

Variable|Description
---|---
`mSurfaceView`|The open GL rendering view
`installRequested`|Boolean flag to determine if an install is required
`mSession`|The AR session
`mGestureDetector`|Gesture detection object
`mMessageSnackbar`|Message container object

``` Java
    private static final String TAG = AgoraARCoreActivity.class.getSimpleName();

    // Rendering. The Renderers are created here, and initialized when the GL surface is created.
    private GLSurfaceView mSurfaceView;

    private boolean installRequested;

    private Session mSession;
    private GestureDetector mGestureDetector;
    private Snackbar mMessageSnackbar;
```

The following private variables help manage AR objects and rendering.

Variable|Description
---|---
`mDisplayRotationHelper`|Helper class to detect AR Rotation
`mBackgroundRenderer`|Handles background rendering
`mVirtualObject`|Renderer for the main AR object
`mVirtualObjectShadow`|Renderer for the AR object shadow
`mPlaneRenderer`|Renderer for the AR plane
`mPointCloud`|Renderer for the point cloud
`mAnchorMatrix`|Help manage per-frame rendering

``` Java
    private DisplayRotationHelper mDisplayRotationHelper;
    
    private final BackgroundRenderer mBackgroundRenderer = new BackgroundRenderer();
    private final ObjectRenderer mVirtualObject = new ObjectRenderer();
    private final ObjectRenderer mVirtualObjectShadow = new ObjectRenderer();
    private final PlaneRenderer mPlaneRenderer = new PlaneRenderer();
    private final PointCloudRenderer mPointCloud = new PointCloudRenderer();
    
    // Temporary matrix allocated here to reduce number of allocations for each frame.
    private final float[] mAnchorMatrix = new float[16];
```

The following private variables are used for gestures, message handling, and video rendering.

Variable|Description
---|---
`queuedSingleTaps`|Single tap queue
`anchors`|Array of anchors
`mRtcEngine`|Agora RTC engine
`mSenderHandler`|Message handler
`SEND_AR_VIEW`|Constant for detecting if AR view should be sent
`mSource`|Agora video source
`mRender`|Agora video renderer
`mSendBuffer`|Buffer for sending data

``` Java
    // Tap handling and UI.
    private final ArrayBlockingQueue<MotionEvent> queuedSingleTaps = new ArrayBlockingQueue<>(16);
    private final ArrayList<Anchor> anchors = new ArrayList<>();

    private RtcEngine mRtcEngine;
    private Handler mSenderHandler;
    private final static int SEND_AR_VIEW = 1;
    private AgoraVideoSource mSource;
    private AgoraVideoRender mRender;
    private ByteBuffer mSendBuffer;
```

The remaining private variables are used to handle what is displayed on screen, event handlers, and peer connections.

Variable|Description
---|---
`mIsARMode`|Boolean to manage if the user is in AR mode
`mHidePoint`|Boolean to manage if a point should be hidden
`mHidePlane`|Boolean to manage if a plane should be hidden
`mScaleFactor`|Float value to manage the factor to scale by
`mRtcEventHandler`|RTC engine event handler
`mPeerObject`|Renderer for the peer
`mRemoteRenders`|Array of Agora Video Renderers for remote users

``` Java
    private boolean mIsARMode;
    private boolean mHidePoint;
    private boolean mHidePlane;
    private float mScaleFactor = 1.0f;

    private IRtcEngineEventHandler mRtcEventHandler;

    private PeerRenderer mPeerObject = new PeerRenderer();

    private List<AgoraVideoRender> mRemoteRenders = new ArrayList<>(20);
```

### Create Screen and Permission Override Methods

The sample application uses superclass override methods, to initialize the application and detect application state changes.

- [Override the onCreate() Method](#override-the-oncreate-method)
- [Override the onResume() Method](#override-the-onresume-method)
- [Override the onPause() Method](#override-the-onpause-method)
- [Override the onDestroy() Method](#override-the-ondestroy-method)
- [Override the onRequestPermissionsResult() Method](#override-the-onrequestpermissionsresult-method)
- [Override the onWindowFocusChanged() Method](#override-the-onwindowfocuschanged-method)

#### Override the onCreate() Method

The onCreate() is invoked once the activity is created. The `R.layout.activity_main` is applied as the screen layout using `setContentView()`.

The `mSurfaceView` variable is with the `surfaceview` object from the UI layout.

``` Java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mSurfaceView = findViewById(R.id.surfaceview);
        mDisplayRotationHelper = new DisplayRotationHelper(/*context=*/ this);
        
        ...
    }
```

Initialize `mGestureDetector` using `new GestureDetector()` and apply event listeners for a single tap up motion and an on tap down motion using `onSingleTapUp()` `onDown()`.

- If the gesture is a single tap up, invoke the `onSingleTap()` method.

- If the gesture is an on down tap gesture, return `true`.

``` Java
        // Set up tap listener.
        mGestureDetector =
                new GestureDetector(
                        this,
                        new GestureDetector.SimpleOnGestureListener() {
                            @Override
                            public boolean onSingleTapUp(MotionEvent e) {
                                onSingleTap(e);
                                return true;
                            }

                            @Override
                            public boolean onDown(MotionEvent e) {
                                return true;
                            }
                        });
```

Apply a touch listener to `mSurfaceView` using  `setOnTouchListener()`.

In the listener object, set the on touch listener callback `onTouch()` returning the gesture's touch event `mGestureDetector.onTouchEvent()`.

``` Java
        mSurfaceView.setOnTouchListener(
                new View.OnTouchListener() {
                    @Override
                    public boolean onTouch(View v, MotionEvent event) {
                        return mGestureDetector.onTouchEvent(event);
                    }
                });
```

Set up `mSurfaceView` settings using the following methods.

Method|Description
---|---
`setPreserveEGLContextOnPause(true)`|Preserves the EGL content when paused
`setEGLContextClientVersion(2)`|Sets the EGL context version to 2
`setEGLConfigChooser(8, 8, 8, 8, 16, 0)`|Configures the EGL plane with an alpha for blending
`setRenderer(this)`|Sets the renderer for the surface view
`setRenderMode(GLSurfaceView.RENDERMODE_CONTINUOUSLY)`|Sets the render mode to continuously render

``` Java
        // Set up renderer.
        mSurfaceView.setPreserveEGLContextOnPause(true);
        mSurfaceView.setEGLContextClientVersion(2);
        mSurfaceView.setEGLConfigChooser(8, 8, 8, 8, 16, 0); // Alpha used for plane blending.
        mSurfaceView.setRenderer(this);
        mSurfaceView.setRenderMode(GLSurfaceView.RENDERMODE_CONTINUOUSLY);
```

The `onCreate()` method completes by setting `installRequested` to false and initializing the Agora RTC engine using `initRtcEngine()`.

``` Java
        installRequested = false;
        initRtcEngine();
```

#### Override the onResume() Method

When the activity resumes, check if `mSession` is `null`.

If the session is `null`:

- Check if the `ArCoreApk` instance needs an install, using `requestInstall()`. If an install is needed, set `installRequested` to `true`.

- Check for camera permissions by calling `CameraPermissionHelper.hasCameraPermission()`. If camera permissions need to be requested, invoke `CameraPermissionHelper.requestCameraPermission()`.

``` Java
    @Override
    protected void onResume() {
        super.onResume();

        if (mSession == null) {
            Exception exception = null;
            String message = null;
            try {
                switch (ArCoreApk.getInstance().requestInstall(this, !installRequested)) {
                    case INSTALL_REQUESTED:
                        installRequested = true;
                        return;
                    case INSTALLED:
                        break;
                }
                // ARCore requires camera permissions to operate. If we did not yet obtain runtime
                // permission on Android M and above, now is a good time to ask the user for it.
                if (!CameraPermissionHelper.hasCameraPermission(this)) {
                    CameraPermissionHelper.requestCameraPermission(this);
                    return;
                }

                mSession = new Session(/* context= */ this);
            
            } ...
            
            ...
            
        }
```

If an exception occurs, display one of the following errors by passing a string to `showSnackbarMessage()` and log the exception.

Exception|Description
---|---
`UnavailableArcoreNotInstalledException`|ARCore is not supported. Tell the user to install ARCore.
`UnavailableApkTooOldException`|The APK is too old. Tell the user to update ARCore.
`UnavailableSdkTooOldException`|The SDK is too old. Tell the user to update the application.
Default exception|Tell the user the device does not support AR.


``` Java
            catch (UnavailableArcoreNotInstalledException
                    | UnavailableUserDeclinedInstallationException e) {
                message = "Please install ARCore";
                exception = e;
            } catch (UnavailableApkTooOldException e) {
                message = "Please update ARCore";
                exception = e;
            } catch (UnavailableSdkTooOldException e) {
                message = "Please update this app";
                exception = e;
            } catch (Exception e) {
                message = "This device does not support AR";
                exception = e;
            }

            if (message != null) {
                showSnackbarMessage(message, true);
                Log.e(TAG, "Exception creating session", exception);
                return;
            }
```

Once the session is created, set the configuration using `mSession.configure()`. If the configuration is not supported, display a `does not support AR` message to the user.

``` Java
            // Create default config and check if supported.
            Config config = new Config(mSession);
            if (!mSession.isSupported(config)) {
                showSnackbarMessage("This device does not support AR", true);
            }
            mSession.configure(config);
```

Complete the `onResume()` method by invoking `showLoadingMessage()` and ensure the session and associated objects are resumed:

- Resume the `mSession` using `resume()`
- Invoke `onResume()` on `mSurfaceView`
- Invoke `onResume()` on `mDisplayRotationHelper`

**Note:** Resuming the objects in this order important, because `mSurfaceView` needs to query `mSession`.

``` Java
        showLoadingMessage();
        // Note that order matters - see the note in onPause(), the reverse applies here.
        mSession.resume();
        mSurfaceView.onResume();
        mDisplayRotationHelper.onResume();
```

#### Override the onPause() Method

When the activity invokes the `onPause()` method, pause the session and associated objects:

- Pause `mDisplayRotationHelper` using `onPause()`
- Pause `mSurfaceView` using `onPause()`
- Pause `mSession` using `pause()`

**Note:** Pausing the objects in this order important, because when `mSurfaceView` is active, it will continue to query `mSession`.

``` Java
    @Override
    public void onPause() {
        super.onPause();
        // Note that the order matters - GLSurfaceView is paused first so that it does not try
        // to query the session. If Session is paused before GLSurfaceView, GLSurfaceView may
        // still call mSession.update() and get a SessionPausedException.
        mDisplayRotationHelper.onPause();
        mSurfaceView.onPause();
        if (mSession != null) {
            mSession.pause();
        }
    }
```

#### Override the onDestroy() Method

When the activity calls the `onDestroy()` method, remove all the renderers from `mRtcEngine` by passing `null` into `mRtcEngine.setRemoteVideoRenderer()`.

Clear all the remove renderers array by using `mRemoteRenders.clear()` and invoke `quit()` on `mSenderHandler.getLooper()`.

Complete the method by destroying the RTC engine object by using `RtcEngine.destroy()`.

``` Java
    @Override
    protected void onDestroy() {
        super.onDestroy();

        mSendBuffer = null;
        for (int i = 0; i < mRemoteRenders.size(); ++i) {
            AgoraVideoRender render = mRemoteRenders.get(i);
            //mRtcEngine.setRemoteVideoRenderer(render.getPeer().uid, null);
        }
        mRemoteRenders.clear();
        mSenderHandler.getLooper().quit();

        RtcEngine.destroy();

    }
```

#### Override the onRequestPermissionsResult() Method

This method is invoked when a user sets device permissions.

Check for camera permissions by using `CameraPermissionHelper.hasCameraPermission()`. If permissions have not been set, ask the user for permissions using `Toast.makeText()` and display the permissions settings window using `CameraPermissionHelper.launchPermissionSettings()`.

``` Java
    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] results) {
        if (!CameraPermissionHelper.hasCameraPermission(this)) {
            Toast.makeText(this,
                "Camera permission is needed to run this application", Toast.LENGTH_LONG).show();
            if (!CameraPermissionHelper.shouldShowRequestPermissionRationale(this)) {
                // Permission denied with checking "Do not ask again".
                CameraPermissionHelper.launchPermissionSettings(this);
            }
            finish();
        }
        
        ...
    }
```

- If the permission requested is `PERMISSION_REQ_ID_RECORD_AUDIO` and `results` is valid, check the app for camera permissions using `checkSelfPermission()`.
- If the permission requested is `PERMISSION_REQ_ID_CAMERA` and `results` is valid, check the app for external storage permissions using `checkSelfPermission()`.

``` Java
        switch (requestCode) {
            case PERMISSION_REQ_ID_RECORD_AUDIO: {
                if (results.length > 0 && results[0] == PackageManager.PERMISSION_GRANTED) {
                    checkSelfPermission(Manifest.permission.CAMERA, PERMISSION_REQ_ID_CAMERA);
                } else {
                    finish();
                }
                break;
            }
            case PERMISSION_REQ_ID_CAMERA: {
                if (results.length > 0 && results[0] == PackageManager.PERMISSION_GRANTED) {
                    checkSelfPermission(Manifest.permission.WRITE_EXTERNAL_STORAGE, PERMISSION_REQ_ID_WRITE_EXTERNAL_STORAGE);
                    //((AGApplication) getApplication()).initWorkerThread();
                } else {
                    finish();
                }
                break;
            }
            case PERMISSION_REQ_ID_WRITE_EXTERNAL_STORAGE: {
                if (results.length > 0 && results[0] == PackageManager.PERMISSION_GRANTED) {
                } else {
                    finish();
                }
                break;
            }
        }
```


#### Override the onWindowFocusChanged() Method

If the window is in focus, set the view to full-screen using `getWindow().getDecorView().setSystemUiVisibility()`.

Ensure the screen stays on by invoking `getWindow().addFlags()`.

``` Java
    @Override
    public void onWindowFocusChanged(boolean hasFocus) {
        super.onWindowFocusChanged(hasFocus);
        if (hasFocus) {
            // Standard Android full-screen functionality.
            getWindow().getDecorView().setSystemUiVisibility(
                View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                    | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                    | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
                    | View.SYSTEM_UI_FLAG_HIDE_NAVIGATION
                    | View.SYSTEM_UI_FLAG_FULLSCREEN
                    | View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY);
            getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);
        }
    }
```
### Create Interaction and Rendering Override Methods

- [Create the onSingleTap() Method](#create-the-onsingletap-method)
- [Override the onSurfaceCreated() Method](#override-the-onsurfacecreated-method)
- [Override the onSurfaceChanged() Method](#override-the-onsurfacechanged-method)
- [Override the onDrawFrame() Method](#override-the-ondrawframe-method)

#### Create the onSingleTap() Method

This method is invoked when a user applies a single tap to the screen. Queue the tap by invoking `queuedSingleTaps.offer()`.

``` Java
    private void onSingleTap(MotionEvent e) {
        // Queue tap if there is space. Tap is lost if queue is full.
        queuedSingleTaps.offer(e);
    }
```

#### Override the onSurfaceCreated() Method

This method is invoked when the surface view is created.

Create the background texture using `mBackgroundRenderer.createOnGlThread()`.

If `mSession` is not `null`, set the `mSession` texture using `mSession.setCameraTextureName()`.

``` Java
    @Override
    public void onSurfaceCreated(GL10 gl, EGLConfig config) {
        GLES20.glClearColor(0.1f, 0.1f, 0.1f, 1.0f);

        // Create the texture and pass it to ARCore session to be filled during update().
        mBackgroundRenderer.createOnGlThread(/*context=*/ this);
        if (mSession != null) {
            mSession.setCameraTextureName(mBackgroundRenderer.getTextureId());
        }
        
        ...
    }
```

Create the rendering objects `mVirtualObject` and `mVirtualObjectShadow` by using `createOnGlThread()` and set the material properties using `setMaterialProperties()`.

- `mVirtualObject` uses the assets `andy.obj` and `andy.png`
- `mVirtualObjectShadow` uses the assets `andy_shadow.obj` and `andy_shadow.png`

If there is an error while creating the objects, log the error using `Log.e()`.

``` Java
        // Prepare the other rendering objects.
        try {
            mVirtualObject.createOnGlThread(/*context=*/this, "andy.obj", "andy.png");
            mVirtualObject.setMaterialProperties(0.0f, 3.5f, 1.0f, 6.0f);

            mVirtualObjectShadow.createOnGlThread(/*context=*/this,
                "andy_shadow.obj", "andy_shadow.png");
            mVirtualObjectShadow.setBlendMode(ObjectRenderer.BlendMode.Shadow);
            mVirtualObjectShadow.setMaterialProperties(1.0f, 0.0f, 0.0f, 1.0f);
        } catch (IOException e) {
            Log.e(TAG, "Failed to read obj file");
        }
```

Create `mPlaneRenderer` with the `trigrid.png` asset using `createOnGlThread()`.

Create the point cloud using `mPointCloud.createOnGlThread()`. 

``` Java
        try {
            mPlaneRenderer.createOnGlThread(/*context=*/this, "trigrid.png");
        } catch (IOException e) {
            Log.e(TAG, "Failed to read plane texture");
        }
        mPointCloud.createOnGlThread(/*context=*/this);
```

Complete the method by creating the peer object using `mPeerObject.createOnGlThread()`.

``` Java
        try {
            mPeerObject.createOnGlThread(this);
        } catch (IOException ex) {
            printLog(ex.toString());
        }
```

#### Override the onSurfaceChanged() Method

The `onSurfaceChanged()` method is invoked each time the surface view is changed.

Invoke the surface changed handler for `mDisplayRotationHelper` using `onSurfaceChanged` and change the viewport settings using `GLES20.glViewport()`.

``` Java
    @Override
    public void onSurfaceChanged(GL10 gl, int width, int height) {
        mDisplayRotationHelper.onSurfaceChanged(width, height);
        GLES20.glViewport(0, 0, width, height);
    }
```

#### Override the onDrawFrame() Method

The `onDrawFrame()` method is invoked when a frame is drawn.

Clear the previous frame using `GLES20.glClear()`. This prevents the driver from loading pixels from the previous frame.

Ensure `mSession` is not `null` before continuing. Update the AR session if needed by using `mDisplayRotationHelper.updateSessionIfNeeded()`.

The remaining code in this section is encapsulated within the `try` statement. If there is an error log the exception using `Log.e()`.

``` Java
    @Override
    public void onDrawFrame(GL10 gl) {
        // Clear screen to notify driver it should not load any pixels from previous frame.
        GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT | GLES20.GL_DEPTH_BUFFER_BIT);

        if (mSession == null) {
            return;
        }
        // Notify ARCore session that the view size changed so that the perspective matrix and
        // the video background can be properly adjusted.
        mDisplayRotationHelper.updateSessionIfNeeded(mSession);
        try {
        
        		...

        } catch (Throwable t) {
            // Avoid crashing the application due to unhandled exceptions.
            Log.e(TAG, "Exception on the OpenGL thread", t);
        }
    }
```

Set the `frame` and `camera` from the session using `mSession.update()` and `frame.getCamera()`.

``` Java
            // Obtain the current frame from ARSession. When the configuration is set to
            // UpdateMode.BLOCKING (it is by default), this will throttle the rendering to the
            // camera framerate.
            Frame frame = mSession.update();
            Camera camera = frame.getCamera();
```

Retrieve the next tap event by using `queuedSingleTaps.poll()`. Ensure that the `tap` is valid and the `camera` is in a `TRACKING` state.

For each tap, perform a hit tests within the frame using `frame.hitTest()`. If any plane or orientation point is hit, ensure the number of `anchors` remains `20` or less by detaching and removing the first object using `detach()` and `remove()`.

Create the new anchor from the `hit` by using `hit.createAnchor()` and add it to the anchor array using `anchors.add()`.

``` Java
            // Handle taps. Handling only one tap per frame, as taps are usually low frequency
            // compared to frame rate.
            MotionEvent tap = queuedSingleTaps.poll();
            if (tap != null && camera.getTrackingState() == TrackingState.TRACKING) {
                for (HitResult hit : frame.hitTest(tap)) {
                    // Check if any plane was hit, and if it was hit inside the plane polygon
                    Trackable trackable = hit.getTrackable();
                    // Creates an anchor if a plane or an oriented point was hit.
                    if ((trackable instanceof Plane && ((Plane) trackable).isPoseInPolygon(hit.getHitPose()))
                            || (trackable instanceof Point
                            && ((Point) trackable).getOrientationMode()
                            == Point.OrientationMode.ESTIMATED_SURFACE_NORMAL)) {
                        // Hits are sorted by depth. Consider only closest hit on a plane or oriented point.
                        // Cap the number of objects created. This avoids overloading both the
                        // rendering system and ARCore.
                        if (anchors.size() >= 20) {
                            anchors.get(0).detach();
                            anchors.remove(0);
                        }
                        // Adding an Anchor tells ARCore that it should track this position in
                        // space. This anchor is created on the Plane to place the 3D model
                        // in the correct position relative both to the world and to the plane.
                        anchors.add(hit.createAnchor());
                        break;
                    }
                }
            }
```

Draw the frame into the background renderer using `mBackgroundRenderer.draw()`.

Check that the camera is tracking, before drawing the 3D objects.

``` Java
            // Draw background.
            mBackgroundRenderer.draw(frame);

            // If not tracking, don't draw 3d objects.
            if (camera.getTrackingState() == TrackingState.PAUSED) {
                return;
            }
```

Get the camera's projection and view matrix by using `camera.getProjectionMatrix()` and `camera.getViewMatrix()`.

Compute the light intensity from the `frame` by using `frame.getLightEstimate().getPixelIntensity()`.

``` Java
            // Get projection matrix.
            float[] projmtx = new float[16];
            camera.getProjectionMatrix(projmtx, 0, 0.1f, 100.0f);

            // Get camera matrix and draw.
            float[] viewmtx = new float[16];
            camera.getViewMatrix(viewmtx, 0);

            // Compute lighting from average intensity of the image.
            final float lightIntensity = frame.getLightEstimate().getPixelIntensity();
```

If the view should show the point cloud, display the tracked points.

- Acquire the point cloud using `frame.acquirePointCloud()`
- Update the point cloud using `mPointCloud.update()`
- Draw the point cloud using `mPointCloud.draw()`

Ensure that the point cloud is released after completion, using `pointCloud.release()`. This aids in memory allocation and garbage collection.

``` Java
            if (isShowPointCloud()) {
                // Visualize tracked points.
                PointCloud pointCloud = frame.acquirePointCloud();
                mPointCloud.update(pointCloud);
                mPointCloud.draw(viewmtx, projmtx);

                // Application is responsible for releasing the point cloud resources after
                // using it.
                pointCloud.release();
            }
```

If a least one plane is detected, hide the loading message using `hideLoadingMessage()`.

If `isShowPlane()` is enabled, draw the planes onto the screen using `mPlaneRenderer.drawPlanes()`.

``` Java
            // Check if we detected at least one plane. If so, hide the loading message.
            if (mMessageSnackbar != null) {
                for (Plane plane : mSession.getAllTrackables(Plane.class)) {
                    if (plane.getType() == Plane.Type.HORIZONTAL_UPWARD_FACING
                            && plane.getTrackingState() == TrackingState.TRACKING) {
                        hideLoadingMessage();
                        break;
                    }
                }
            }

            if (isShowPlane()) {
                // Visualize planes.
                mPlaneRenderer.drawPlanes(
                        mSession.getAllTrackables(Plane.class), camera.getDisplayOrientedPose(), projmtx);
            }
```

Check that each `anchor` is `TRACKING` before displaying the touch anchors.

If AR mode is true:

- Update the model object `mVirtualObject` and its shadow `mVirtualObjectShadow` using `updateModelMatrix()`.
- Draw both `mVirtualObject` and `mVirtualObjectShadow` onto the screen using `draw()`.

If AR mode is false:

- Update the peer's model matrix using `mPeerObject.updateModelMatrix()`
- Render the peer object using `mPeerObject.draw()`

``` Java
            // Visualize anchors created by touch.
            float scaleFactor = 1.0f;

            int i = 0;
            for (Anchor anchor : anchors) {
                if (anchor.getTrackingState() != TrackingState.TRACKING) {
                    continue;
                }
                // Get the current pose of an Anchor in world space. The Anchor pose is updated
                // during calls to session.update() as ARCore refines its estimate of the world.
                anchor.getPose().toMatrix(mAnchorMatrix, 0);

                if (mIsARMode) {
                    // Update and draw the model and its shadow.
                    mVirtualObject.updateModelMatrix(mAnchorMatrix, mScaleFactor);
                    mVirtualObjectShadow.updateModelMatrix(mAnchorMatrix, scaleFactor);
                    mVirtualObject.draw(viewmtx, projmtx, lightIntensity);
                    mVirtualObjectShadow.draw(viewmtx, projmtx, lightIntensity);
                } else {
                    if (mRemoteRenders.size() > i) {
                        AgoraVideoRender render = mRemoteRenders.get(i);
                        ++i;
                        if (render == null) continue;

                        Peer peer = render.getPeer();
                        if (peer.data == null || peer.data.capacity() == 0) continue;

                        mPeerObject.updateModelMatrix(mAnchorMatrix, scaleFactor);
                        mPeerObject.draw(viewmtx, projmtx, peer);
                    }
                }
            }
```

Complete the method by sending an AR view message using `sendARViewMessage()`.

``` Java
            sendARViewMessage(gl);
```

### Create the initRtcEngine() Method

The `initRtcEngine()` method initializes the Agora RTC engine.

Initialize `mIsARMode`, `mHidePoint`, and `mHidePlane` to `true`.

Connect `modeButton` to the UI by using `findViewById()` and add a click event listener using `setOnClickListener()`. When the button is clicked, invoke `switchMode()`.

``` Java
    private void initRtcEngine() {
        mIsARMode = true;
        mHidePoint = true;
        mHidePlane = true;

        Button modeButton = findViewById(R.id.switch_mode);
        modeButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                switchMode((Button)view);
            }
        });
        
        ...
        
    }
```

Connect `hidePointButton` to the UI by using `findViewById()` and add a click event listener using `setOnClickListener()`. When the button is clicked, invoke `showPointCloud()`.


``` Java
        Button hidePointButton = findViewById(R.id.show_point_cloud);
        hidePointButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                showPointCloud((Button)v);
            }
        });
```

Connect `hidePlaneButton` to the UI by using `findViewById()` and add a click event listener using `setOnClickListener()`. When the button is clicked, invoke `showPlane()`.

``` Java
        Button hidePlaneButton = findViewById(R.id.show_plane);
        hidePlaneButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                showPlane((Button)v);
            }
        });
```

Connect `zoomInButton` to the UI by using `findViewById()` and add a click event listener using `setOnClickListener()`. When the button is clicked, increment `mScaleFactor` by `0.2f` if it is less than or equal to `5.0f`.

Connect `zoomOutButton` to the UI by using `findViewById()` and add a click event listener using `setOnClickListener()`. When the button is clicked, decrement `mScaleFactor` by `0.2f` if it is greater than or equal to `0.1f`.

``` Java
        Button zoomInButton = findViewById(R.id.zoom_in);
        zoomInButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (mScaleFactor > 5.0f) {
                    return;
                }
                mScaleFactor += 0.2f;
            }
        });

        Button zoomOutButton = findViewById(R.id.zoom_out);
        zoomOutButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (mScaleFactor < 0.1f) {
                    return;
                }
                mScaleFactor -= 0.2f;
            }
        });
```

The `try` statement initializes the Agora RTC engine, events, and settings. If it false, log the error using `printLog`.

At the end of the method, create and start a new handler `thread` using `new HandlerThread()` and `thread.start()`. When `thread` receives a message, send the AR view using `sendARView` if the message type is `SEND_AR_VIEW`.

The remaining code in this section is encapsulated within the `try` statement.

``` Java
        try {
        
        	...

        } catch (Exception ex) {
            printLog(ex.toString());
        }
        
        HandlerThread thread = new HandlerThread("ArSendThread");
        thread.start();
        mSenderHandler = new Handler(thread.getLooper()) {
            @Override
            public void handleMessage(Message msg) {
                switch (msg.what) {
                    case SEND_AR_VIEW:
                        sendARView((Bitmap)msg.obj);
                }
            }
        };
```

Create a new `IRtcEngineEventHandler` object and add the following event listeners:

Event listener|Description|Breakdown
---|---|---
`onJoinChannelSuccess`|Triggered when a user successfully joins a channel|Log the channel using `printLog()`.
`onFirstRemoteVideoDecoded`|Triggered with the first remote video is decoded|Add a remote renderer for the user `uid` using `addRemoteRender()`.
`onUserOffline`|Triggered with a user goes offline|Removes the user `uid` from `mRemoteRenders` using `remove()` and sets the remove view renderer to null for that user using `mRtcEngine.setRemoteVideoRenderer()`.
`onUserJoined`|Triggers when a user joins a channel|Can be used to add logs or message alerts.
`onError`|Triggers when there is an error|Logs the error using `printLog()`.
`onWarning`|Triggers when there is a warning|Logs the warning using `printLog()`.

``` Java
            mRtcEventHandler = new IRtcEngineEventHandler() {
                @Override
                public void onJoinChannelSuccess(String channel, int uid, int elapsed) {
                    printLog("joined channel " + channel);
                }

                @Override
                public void onFirstRemoteVideoDecoded(int uid, int width, int height, int elapsed) {
                    addRemoteRender(uid);
                }

                @Override
                public void onUserOffline(int uid, int reason) {
                    for (int i = 0; i < mRemoteRenders.size(); ++i) {
                        AgoraVideoRender render = mRemoteRenders.get(i);
                        if (render.getPeer().uid == uid) {
                            mRemoteRenders.remove(uid);
                            mRtcEngine.setRemoteVideoRenderer(uid, null);
                        }
                    }
                }

                @Override
                public void onUserJoined(int uid, int elapsed) {
                }

                @Override
                public void onError(int err) {
                    printLog("Error: " + err);
                }

                @Override
                public void onWarning(int warn) {
                    printLog("Warning: " + warn);
                }
            };
```

Create the RTC engine using `RtcEngine.create()` and apply the following settings:

Setting|Method
---|---
Set the parameters|`mRtcEngine.setParameters()`
Set the channel profile to live broadcasting|`mRtcEngine.setChannelProfile()`
Enable video|`mRtcEngine.enableVideo()`
Enable dual stream mode|`mRtcEngine.enableDualStreamMode()`
Set the video profile|`mRtcEngine.setVideoProfile()`
Set the client role to live broadcaster|`mRtcEngine.setClientRole()`
Set the video source to `mSource`|`mRtcEngine.setVideoSource()`
Set the local video renderer to `mRender`| `mRtcEngine.setLocalVideoRenderer()`

Complete the method by joining the channel using `mRtcEngine.joinChannel()`.

``` Java
            mRtcEngine = RtcEngine.create(this, getString(R.string.private_broadcasting_app_id), mRtcEventHandler);
            mRtcEngine.setParameters("{\"rtc.log_filter\": 65535}");
            mRtcEngine.setChannelProfile(Constants.CHANNEL_PROFILE_LIVE_BROADCASTING);
            mRtcEngine.enableVideo();
            mRtcEngine.enableDualStreamMode(true);

            mRtcEngine.setVideoProfile(Constants.VIDEO_PROFILE_480P, false);
            mRtcEngine.setClientRole(Constants.CLIENT_ROLE_BROADCASTER);

            mSource = new AgoraVideoSource();
            mRender = new AgoraVideoRender(0, true);
            mRtcEngine.setVideoSource(mSource);
            mRtcEngine.setLocalVideoRenderer(mRender);

            //mRtcEngine.startPreview();

            mRtcEngine.joinChannel(null, "arcore", "ARCore with RtcEngine", 0);
            
```

Create Message Handling Methods

- [Create the showSnackbarMessage() Method](#create-the-showsnackbarmessage-method)
- [Show and Hide Loading Message Methods](#show-and-hide-loading-message-methods)
- [Create the printLog() Method](#create-the-printlog-method)
- [Create the sendARViewMessage() Method](#create-the-sendarviewmessage-method)
- [Create the sendARView() Method](#create-the-sendarview-method)

#### Create the showSnackbarMessage() Method

Create `mMessageSnackbar` from the UI the using `Snackbar.make` and `AgoraARCoreActivity.this.findViewById` and set the background color to `0xbf323232`.

If `finishOnDismiss` is true, add an event listener to the `Dismiss` button. This button would invoke `mMessageSnackbar.dismiss()`. Ensure a dismissal callback is applied to `mMessageSnackbar`, which invokes the superclass' dismiss method by calling `super.onDismissed()`.

Complete the method by displaying `mMessageSnackbar` by using `show()`.

``` Java
    private void showSnackbarMessage(String message, boolean finishOnDismiss) {
        mMessageSnackbar = Snackbar.make(
            AgoraARCoreActivity.this.findViewById(android.R.id.content),
            message, Snackbar.LENGTH_INDEFINITE);
        mMessageSnackbar.getView().setBackgroundColor(0xbf323232);
        if (finishOnDismiss) {
            mMessageSnackbar.setAction(
                "Dismiss",
                new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        mMessageSnackbar.dismiss();
                    }
                });
            mMessageSnackbar.addCallback(
                new BaseTransientBottomBar.BaseCallback<Snackbar>() {
                    @Override
                    public void onDismissed(Snackbar transientBottomBar, int event) {
                        super.onDismissed(transientBottomBar, event);
                        finish();
                    }
                });
        }
        mMessageSnackbar.show();
```

#### Show and Hide Loading Message Methods

The `showLoadingMessage()` method runs on the UI thread. Within the `run()` method, invoke `showSnackbarMessage()` to display the loading message.

The `hideLoadingMessage()` method also runs on the UI thread. Within the `run()` method, ensure `mMessageSnackbar` is not `null`. If `mMessageSnackbar` is `null`, dismiss it using `mMessageSnackbar.dismiss()`. Set `mMessageSnackbar` to `null` to complete the `run()` override method.


``` Java
    private void showLoadingMessage() {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                showSnackbarMessage("Searching for surfaces...", false);
            }
        });
    }

    private void hideLoadingMessage() {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                if (mMessageSnackbar != null) {
                    mMessageSnackbar.dismiss();
                }
                mMessageSnackbar = null;
            }
        });
    }
```

#### Create the printLog() Method

The `printLog()` method is used throughout the sample application to log error messages and debug messages by using `Log.e()`.

``` Java
    private void printLog(String message) {
        Log.e("ARCore", message);
    }
```

#### Create the sendARViewMessage() Method

The `sendARViewMessage()` method handles creation and sending of an AR image. 

Begin by retrieving the height and width of the surface view using `mSurfaceView.getWidth()` and `mSurfaceView.getHeight()` and calculate the scene size by multiplying the two results.

If `mSendBuffer` is `null`, create the object using `ByteBuffer.allocateDirect()` and set the order using `mSendBuffer.order()`.

Initialize the buffer position to `0` using `mSendBuffer.position()`.

The remaining code in this section are within the `sendARViewMessage()` method.

``` Java
    private void sendARViewMessage(GL10 gl) {
        int w = mSurfaceView.getWidth();
        int h = mSurfaceView.getHeight();
        int sceneSize = w * h;
        if (mSendBuffer == null) {
            mSendBuffer = ByteBuffer.allocateDirect(sceneSize * 4);
            mSendBuffer.order(ByteOrder.nativeOrder());
        }
        mSendBuffer.position(0);
        
        ...
        
    }
```

Read the pixels from `mSendBuffer` using `gl.glReadPixels()`.

Generate `pixelsBuffer[]` and apply it to `mSendBuffer` by using `asIntBuffer().get()`.

Create a `Bitmap` object using `Bitmap.createBitmap()` and set the pixels with `pixelsBuffer` using `bitmap.setPixels()`. Ensure `pixelsBuffer` is set to `null` for garbage collection.

Copy the AR image pixels to `bitmap` using `bitmap.copyPixelsToBuffer()`.

``` Java
        gl.glReadPixels(0, 0, w, h, GL10.GL_RGBA, GL10.GL_UNSIGNED_BYTE, mSendBuffer);
        int pixelsBuffer[] = new int[sceneSize];
        mSendBuffer.asIntBuffer().get(pixelsBuffer);
        Bitmap bitmap = Bitmap.createBitmap(w, h, Bitmap.Config.RGB_565);
        bitmap.setPixels(pixelsBuffer, sceneSize - w, -w, 0, 0, w, h);
        pixelsBuffer = null;

        short sBuffer[] = new short[sceneSize];
        ShortBuffer tmpBuffer = ShortBuffer.wrap(sBuffer);
        bitmap.copyPixelsToBuffer(tmpBuffer);
```

Convert the bitmap from OpenGL format to an Android-compatible bitmap.

Iterate through `sceneSize` and convert the sBuffer[i] based on the original `sBuffer[i]` value.

When the buffer conversion completes, return to the first location in `tmpBuffer` by using `tmpBuffer.rewind()`.

Copy the `tmpBuffer` pixels into `bitmap` using `bitmap.copyPixelsFromBuffer()`.

``` Java
        // Making created bitmap (from OpenGL points) compatible with
        // Android bitmap
        for (int i = 0; i < sceneSize; ++i) {
            short v = sBuffer[i];
            sBuffer[i] = (short) (((v & 0x1f) << 11) | (v & 0x7e0) | ((v & 0xf800) >> 11));
        }
        tmpBuffer.rewind();
        bitmap.copyPixelsFromBuffer(tmpBuffer);
```

Create a new `Bitmap` object from `bitmap` using `bitmap.copy()`.

Complete the method by retrieving the `Message` object using `Message.obtain()` and sending the AR image by:

- Setting the message type to `SEND_AR_VIEW`
- Setting the message object to the bitmap `result`
- Send the image message by using `mSenderHandler.sendMessage()`.

``` Java
        Bitmap result = bitmap.copy(Bitmap.Config.ARGB_8888,false);
        
        Message message = Message.obtain();
        message.what = SEND_AR_VIEW;
        message.obj = result;
        mSenderHandler.sendMessage(message);
```

#### Create the sendARView() Method

The `sendARView()` method sends the image snapshot of the AR view specified in the paramater `bitmap`.

Ensure `bitmap` and `mSource.getConsumer()` are not `null` before continuing the send process.

Retrieve the width and height of `bitmap` using `bitmap.getWidth()` and `bitmap.getHeight()`.

Calculate `size` by multiplying `bitmap.getRowBytes()` with `bitmap.getHeight()` and apply it to a `ByteBuffer` object using `ByteBuffer.allocate()`.

Copy the buffer pixels to the bitmap using `bitmap.copyPixelsToBuffer()`.

Complete the method by applying `data` array to `mSource` using `getConsumer().consumeByteArrayFrame()`.

``` Java
    private void sendARView(Bitmap bitmap) {
        if (bitmap == null) return;

        if (mSource.getConsumer() == null) return;

        //Bitmap bitmap = source.copy(Bitmap.Config.ARGB_8888,true);
        int width = bitmap.getWidth();
        int height = bitmap.getHeight();

        int size = bitmap.getRowBytes() * bitmap.getHeight();
        ByteBuffer byteBuffer = ByteBuffer.allocate(size);
        bitmap.copyPixelsToBuffer(byteBuffer);
        byte[] data = byteBuffer.array();

        mSource.getConsumer().consumeByteArrayFrame(data, MediaIO.PixelFormat.RGBA.intValue(), width, height, 270, System.currentTimeMillis());
    }
```

### Create the addRemoteRender() Method

The `addRemoteRender()` method adds a new Agora video renderer to the Agora RTC engine.

- Create a new `AgoraVideoRender` object applying the user's `uid`.
- Add `render` to the list of remote renderers using `mRemoteRenders.add()`.
- Set the remote video renderer for `uid` using `mRtcEngine.setRemoteVideoRenderer()`.

``` Java
    private void addRemoteRender(int uid) {
        AgoraVideoRender render = new AgoraVideoRender(uid, false);
        mRemoteRenders.add(render);
        mRtcEngine.setRemoteVideoRenderer(uid, render);
    }
```

### Set Permission Variables and Methods

The following private static variables are used as constants for the permission methods.

Variable|Value|Description
---|---|---
`BASE_VALUE_PERMISSION`|`0X0001`|Base value used for permission calculations
`PERMISSION_REQ_ID_RECORD_AUDIO`|`BASE_VALUE_PERMISSION + 1`|ID value used for audio recording permissions
`PERMISSION_REQ_ID_CAMERA`|`BASE_VALUE_PERMISSION + 2`|ID value used for camera permissions
`PERMISSION_REQ_ID_WRITE_EXTERNAL_STORAGE`|`BASE_VALUE_PERMISSION + 3`|ID value used for external storage permissions


``` Java
    private static final int BASE_VALUE_PERMISSION = 0X0001;
    private static final int PERMISSION_REQ_ID_RECORD_AUDIO = BASE_VALUE_PERMISSION + 1;
    private static final int PERMISSION_REQ_ID_CAMERA = BASE_VALUE_PERMISSION + 2;
    private static final int PERMISSION_REQ_ID_WRITE_EXTERNAL_STORAGE = BASE_VALUE_PERMISSION + 3;
```

The `checkSelfPermissions()` methods provide options to check all device permissions or to check permissions for a specific permission type.

If no permission type is specified, check all the permission types are enabled and return the result.

If a permission type is specified, check for granted device permissions using `ContextCompat.checkSelfPermission(this, permission) != PackageManager.PERMISSION_GRANTED`. If permissions have not been granted yet, request permissions using `ActivityCompat.requestPermissions()`. 

``` Java
    private boolean checkSelfPermissions() {
        return checkSelfPermission(Manifest.permission.RECORD_AUDIO, PERMISSION_REQ_ID_RECORD_AUDIO) &&
                checkSelfPermission(Manifest.permission.CAMERA, PERMISSION_REQ_ID_CAMERA) &&
                checkSelfPermission(Manifest.permission.WRITE_EXTERNAL_STORAGE, PERMISSION_REQ_ID_WRITE_EXTERNAL_STORAGE);
    }

    public boolean checkSelfPermission(String permission, int requestCode) {
        //log.debug("checkSelfPermission " + permission + " " + requestCode);
        if (ContextCompat.checkSelfPermission(this, permission) != PackageManager.PERMISSION_GRANTED) {

            ActivityCompat.requestPermissions(this, new String[]{permission}, requestCode);
            return false;
        }

        return true;
    }
```

### Add Button Methods

The following methods are applied to buttons in the UI layout.

The `switchMode()` method toggles AR mode on / off.

Update the `button` text using `button.setText()`. `R.string.ar_mode` indicates AR Mode, and `R.string.agora_mode` indicates Agora mode.

``` Java
    private void switchMode(Button button) {
        button.setText((mIsARMode = !mIsARMode) ? getString(R.string.ar_mode) :
                getString(R.string.agora_mode));
    }
```

The `showPointCloud()` method displays / hides the cloud points.

Update the `button` text using `button.setText()`. `R.string.show_point` is the text indicator to show points, and `R.string.hide_point` is the text indicator to hide points.

``` Java
    private void showPointCloud(Button button) {
        button.setText((mHidePoint = !mHidePoint) ? getString(R.string.show_point) :
                getString(R.string.hide_point));
    }
```

The `showPlane()` method displays / hides the plane.

Update the `button` text using `button.setText()`. `R.string.show_plane` is the text indicator to show points, and `R.string.hide_plane` is the text indicator to hide points.

``` Java
    private void showPlane(Button button) {
        button.setText((mHidePlane = !mHidePlane) ? getString(R.string.show_plane) :
                getString(R.string.hide_plane));
    }
```

### Add App Setting Retrieval Methods

The following methods are used to retrieve settings applied to the app.

The `isARMode()` method returns `mIsARMode` and indicates if the app is in AR mode.

``` Java
    private boolean isARMode() {
        return mIsARMode;
    }
```

The `isShowPointCloud()` method returns `!mHidePoint` and indicates if the app should show the point cloud.

``` Java
    private boolean isShowPointCloud() {
        return !mHidePoint;
    }
```

The `isShowPlane()` method returns `!mHidePlane` and indicates if the app should show the plane.


``` Java
    private boolean isShowPlane() {
        return !mHidePlane;
    }
```

## Resources
* You can find full API documentation at the [Document Center](https://docs.agora.io/en/).
* You can file bugs about this sample [here](https://github.com/AgoraIO/Agora-Video-With-ARCore/issues).


## License
This software is under the MIT License (MIT). [View the license](LICENSE.md).