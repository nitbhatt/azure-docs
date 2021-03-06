---
author: ramonarguelles
ms.service: azure-spatial-anchors
ms.topic: include
ms.date: 1/29/2019
ms.author: rgarcia
---
In Unity, open the project in the `Unity` folder.

Open **Build Settings** by selecting **File** > **Build Settings**.

In the **Platform** section, select **Android**. Change the **Build System** to **Gradle** and select **Export Project**.

Select **Switch Platform** to change the platform to **Android**. Unity might prompt you to install Android support components if they're missing.

![Unity Build Settings window](./media/spatial-anchors-unity/unity-android-build-settings.png)

Close the **Build Settings** window.

### Download and import the ARCore SDK for Unity

Download the `unitypackage` file from the [ARCore SDK for Unity releases](https://github.com/google-ar/arcore-unity-sdk/releases/tag/v1.5.0). Back in the Unity project, select **Assets** > **Import Package** > **Custom Package** and then select the `unitypackage` file you previously downloaded. In the **Import Unity Package** dialog box, make sure all files are selected and then select **Import**.
