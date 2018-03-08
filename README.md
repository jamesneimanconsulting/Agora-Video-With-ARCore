# Agora Video With ARCore

*其他语言版本： [简体中文](README.zhCN.md)*

The Agora-Video-With-ARCore sample app is an open-source demo that will help you get live video chat integrated into your android ARCore applications using the Agora Video SDK.

With this sample app, you can:

- Send captured image from ARCore to live video channel
- Render video frames of remote user to ARCore session

## Running the App
First, create a developer account at [Agora.io](https://dashboard.agora.io/signin/), and obtain an App ID. Update "res/string.xml" with your App ID.

Next, download the **Agora Video SDK** from [Agora.io SDK](https://www.agora.io/en/blog/download/). Unzip the downloaded SDK package and copy the **agora-rtc-sdk.jar** to the "Agora-ARCore/app/libs" and **armeabi-v7a/libagora-rtc-sdk-jni.so** to  "Agora-ARCore/app/src/main/jniLibs/armeabi-v7a" folder in project.

> Custom media device protocols are provided from 2.1.0

Finally, Open Agora-Video-With-ARCore using android studio, connect your android device and run.

## Usage
1. Move device to find a horizontal plane
2. Touch the plane on screen, will add a virtual display screen
3. Run [OpenLive](https://github.com/AgoraIO/OpenLive-Android) with same AppId, join "arcore" as broadcaster
4. Video from remote user will be displayed on virtual screen in ARSession

## Developer Environment Requirements
* Android studio 3.0.1 
* Android SDK Platform version 7.0 (API level 24) or higher.
* [Real devices](https://developers.google.com/ar/discover/) with ARCore supported

## Connect Us

- You can find full API document at [Document Center](https://docs.agora.io/en/)
- You can file bugs about this demo at [issue](https://github.com/AgoraIO/Agora-Video-With-ARCore/issues)

## License

The MIT License (MIT).
