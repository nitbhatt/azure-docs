---
title: Provision a simulated TPM device to Azure IoT Hub using C | Microsoft Docs
description: This quickstart uses individual enrollments. In this quickstart, you create and provision a simulated TPM device using C device SDK for Azure IoT Hub Device Provisioning Service.
author: wesmc7777
ms.author: wesmc
ms.date: 07/13/2018
ms.topic: quickstart
ms.service: iot-dps
services: iot-dps 
manager: timlt
ms.custom: mvc
#Customer intent: As a new IoT developer, I want simulate a TPM device using the C SDK so that I can learn how secure provisioning works.
---

# Quickstart: Provision a simulated TPM device using the Azure IoT C SDK

[!INCLUDE [iot-dps-selector-quick-create-simulated-device-tpm](../../includes/iot-dps-selector-quick-create-simulated-device-tpm.md)]

In this quickstart, you will learn how to create and run a Trusted Platform Module (TPM) device simulator on a Windows development machine. You will connect this simulated device to an IoT hub using a Device Provisioning Service instance. Sample code from the [Azure IoT C SDK](https://github.com/Azure/azure-iot-sdk-c) will be used to help enroll the device with a Device Provisioning Service instance and simulate a boot sequence for the device.

If you're unfamiliar with the process of autoprovisioning, review [Auto-provisioning concepts](concepts-auto-provisioning.md). Also, make sure you've completed the steps in [Set up IoT Hub Device Provisioning Service with the Azure portal](./quick-setup-auto-provision.md) before continuing with this quickstart. 

The Azure IoT Device Provisioning Service supports two types of enrollments:
- [Enrollment groups](concepts-service.md#enrollment-group): Used to enroll multiple related devices.
- [Individual Enrollments](concepts-service.md#individual-enrollment): Used to enroll a single device.

This article will demonstrate individual enrollments.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## Prerequisites

* Visual Studio 2015 or [Visual Studio 2017](https://www.visualstudio.com/vs/) with the ['Desktop development with C++'](https://www.visualstudio.com/vs/support/selecting-workloads-visual-studio-2017/) workload enabled.
* Latest version of [Git](https://git-scm.com/download/) installed.


<a id="setupdevbox"></a>

## Prepare a development environment for the Azure IoT C SDK

In this section, you will prepare a development environment used to build the [Azure IoT C SDK](https://github.com/Azure/azure-iot-sdk-c) and the [TPM](https://docs.microsoft.com/windows/device-security/tpm/trusted-platform-module-overview) device simulator sample.

1. Download the [CMake build system](https://cmake.org/download/). Verify the downloaded binary using the cryptographic hash value that corresponds to the version you download. The cryptographic hash values are also located from the CMake download link already provided.

    The following example used Windows PowerShell to verify the cryptographic hash for version 3.13.4 of the x64 MSI distribution:

    ```PowerShell
    PS C:\Downloads> $hash = get-filehash .\cmake-3.13.4-win64-x64.msi
    PS C:\Downloads> $hash.Hash -eq "64AC7DD5411B48C2717E15738B83EA0D4347CD51B940487DFF7F99A870656C09"
    True
    ```

    The following hash values for version 3.13.4 were listed on the CMake site at the time of this writing:

    ```
    563a39e0a7c7368f81bfa1c3aff8b590a0617cdfe51177ddc808f66cc0866c76  cmake-3.13.4-Linux-x86_64.tar.gz
    7c37235ece6ce85aab2ce169106e0e729504ad64707d56e4dbfc982cb4263847  cmake-3.13.4-win32-x86.msi
    64ac7dd5411b48c2717e15738b83ea0d4347cd51b940487dff7f99a870656c09  cmake-3.13.4-win64-x64.msi
    ```

    It is important that the Visual Studio prerequisites (Visual Studio and the 'Desktop development with C++' workload) are installed on your machine, **before** starting the `CMake` installation. Once the prerequisites are in place, and the download is verified, install the CMake build system.

2. Open a command prompt or Git Bash shell. Execute the following command to clone the [Azure IoT C SDK](https://github.com/Azure/azure-iot-sdk-c) GitHub repository:
    
    ```cmd/sh
    git clone https://github.com/Azure/azure-iot-sdk-c.git --recursive
    ```
    The size of this repository is currently around 220 MB. You should expect this operation to take several minutes to complete.


3. Create a `cmake` subdirectory in the root directory of the git repository, and navigate to that folder. 

    ```cmd/sh
    cd azure-iot-sdk-c
    mkdir cmake
    cd cmake
    ```

## Build the SDK and run the TPM device simulator

In this section, you will build the Azure IoT C SDK, which includes the TPM device simulator sample code. This sample provides a TPM [attestation mechanism](concepts-security.md#attestation-mechanism) via Shared Access Signature (SAS) Token authentication.

1. From the `cmake` subdirectory you created in the azure-iot-sdk-c git repository, run the following command to build the sample. A Visual Studio solution for the simulated device will also be generated by this build command.

    ```cmd/sh
    cmake -Duse_prov_client:BOOL=ON -Duse_tpm_simulator:BOOL=ON ..
    ```

    If `cmake` does not find your C++ compiler, you might get build errors while running the above command. If that happens, try running this command in the [Visual Studio command prompt](https://docs.microsoft.com/dotnet/framework/tools/developer-command-prompt-for-vs). 

    Once the build succeeds, the last few output lines will look similar to the following output:

    ```cmd/sh
    $ cmake -Duse_prov_client:BOOL=ON -Duse_tpm_simulator:BOOL=ON ..
    -- Building for: Visual Studio 15 2017
    -- Selecting Windows SDK version 10.0.16299.0 to target Windows 10.0.17134.
    -- The C compiler identification is MSVC 19.12.25835.0
    -- The CXX compiler identification is MSVC 19.12.25835.0

    ...

    -- Configuring done
    -- Generating done
    -- Build files have been written to: E:/IoT Testing/azure-iot-sdk-c/cmake
    ```

2. Navigate to the root folder of the git repository you cloned, and run the [TPM](https://docs.microsoft.com/windows/device-security/tpm/trusted-platform-module-overview) simulator using the path shown below. This simulator listens over a socket on ports 2321 and 2322. Do not close this command window; you will need to keep this simulator running until the end of this quickstart. 

   If you are in the *cmake* folder, then run the following commands:

    ```cmd/sh
    cd ..
    .\provisioning_client\deps\utpm\tools\tpm_simulator\Simulator.exe
    ```

    You will not see any output from the simulator. Let it continue to run simulating a TPM device.

<a id="simulatetpm"></a>

## Read cryptographic keys from the TPM device

In this section, you will build and execute a sample that will read the endorsement key and registration ID from the TPM simulator you left running, and listening over ports 2321 and 2322. These values will be used for device enrollment with your Device Provisioning Service instance.

1. Launch Visual Studio and open the new solution file named `azure_iot_sdks.sln`. This solution file is located in the `cmake` folder you previously created in the root of the azure-iot-sdk-c git repository.

2. On the Visual Studio menu, select **Build** > **Build Solution** to build all projects in the solution.

3. In Visual Studio's *Solution Explorer* window, navigate to the **Provision\_Tools** folder. Right-click the **tpm_device_provision** project and select **Set as Startup Project**. 

4. On the Visual Studio menu, select **Debug** > **Start without debugging** to run the solution. The app reads and displays a **_Registration ID_** and an **_Endorsement Key_**. Copy these values. They will be used in the next section for device enrollment. 


<a id="portalenrollment"></a>

## Create a device enrollment entry in the portal

1. Sign in to the Azure portal, click on the **All resources** button on the left-hand menu and open your Device Provisioning service.

2. Select the **Manage enrollments** tab, and then click the **Add individual enrollment** button at the top. 

3. On **Add enrollment**, enter the following information, and click the **Save** button.

    - **Mechanism:** Select **TPM** as the identity attestation *Mechanism*.
    - **Endorsement key:** Enter the *Endorsement key* you generated for your TPM device by running the *tpm_device_provision* project.
    - **Registration ID:** Enter the *Registration ID* you generated for your TPM device by running the *tpm_device_provision* project.
    - **IoT Edge device:** Select **Disable**.
    - **IoT Hub Device ID:** Enter **test-docs-device** to give the device an ID.

      ![Enter device enrollment information in the portal](./media/quick-create-simulated-device/enter-device-enrollment.png)  

      On successful enrollment, the *Registration ID* of your device will appear in the list under the *Individual Enrollments* tab. 


<a id="firstbootsequence"></a>

## Simulate first boot sequence for the device

In this section, you will configure sample code to use the [Advanced Message Queuing Protocol (AMQP)](https://wikipedia.org/wiki/Advanced_Message_Queuing_Protocol) to send the device's boot sequence to your Device Provisioning Service instance. This boot sequence will cause the device to be recognized and assigned to an IoT hub linked to the Device Provisioning Service instance.

1. In the Azure portal, select the **Overview** tab for your Device Provisioning service and copy the **_ID Scope_** value.

    ![Extract Device Provisioning Service endpoint information from the portal](./media/quick-create-simulated-device/extract-dps-endpoints.png) 

2. In Visual Studio's *Solution Explorer* window, navigate to the **Provision\_Samples** folder. Expand the sample project named **prov\_dev\_client\_sample**. Expand **Source Files**, and open **prov\_dev\_client\_sample.c**.

3. Near the top of the file, find the `#define` statements for each device protocol as shown below. Make sure only `SAMPLE_AMQP` is uncommented.

    Currently, the [MQTT protocol is not supported for TPM Individual Enrollment](https://github.com/Azure/azure-iot-sdk-c#provisioning-client-sdk).

    ```c
    //
    // The protocol you wish to use should be uncommented
    //
    //#define SAMPLE_MQTT
    //#define SAMPLE_MQTT_OVER_WEBSOCKETS
    #define SAMPLE_AMQP
    //#define SAMPLE_AMQP_OVER_WEBSOCKETS
    //#define SAMPLE_HTTP
    ```

4. Find the `id_scope` constant, and replace the value with your **ID Scope** value that you copied earlier. 

    ```c
    static const char* id_scope = "0ne00002193";
    ```

5. Find the definition for the `main()` function in the same file. Make sure the `hsm_type` variable is set to `SECURE_DEVICE_TYPE_TPM` instead of `SECURE_DEVICE_TYPE_X509` as shown below.

    ```c
    SECURE_DEVICE_TYPE hsm_type;
    hsm_type = SECURE_DEVICE_TYPE_TPM;
    //hsm_type = SECURE_DEVICE_TYPE_X509;
    ```

6. Right-click the **prov\_dev\_client\_sample** project and select **Set as Startup Project**. 

7. On the Visual Studio menu, select **Debug** > **Start without debugging** to run the solution. In the prompt to rebuild the project, click **Yes**, to rebuild the project before running.

    The following output is an example of the provisioning device client sample successfully booting up, and connecting to a Device Provisioning Service instance to get IoT hub information and registering:

    ```cmd
    Provisioning API Version: 1.2.7
    Provisioning Status: PROV_DEVICE_REG_STATUS_CONNECTED

    Registering... Press enter key to interrupt.

    Provisioning Status: PROV_DEVICE_REG_STATUS_CONNECTED
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING

    Registration Information received from service:
    test-docs-hub.azure-devices.net, deviceId: test-docs-device
    ```

8. Once the simulated device is provisioned to the IoT hub by your provisioning service, the device ID appears with the hub's **IoT Devices**. 

    ![Device is registered with the IoT hub](./media/quick-create-simulated-device/hub-registration.png) 


## Clean up resources

If you plan to continue working on and exploring the device client sample, do not clean up the resources created in this Quickstart. If you do not plan to continue, use the following steps to delete all resources created by this Quickstart.

1. Close the device client sample output window on your machine.
2. Close the TPM simulator window on your machine.
3. From the left-hand menu in the Azure portal, click **All resources** and then select your Device Provisioning service. Open **Manage Enrollments** for your service, and then click the **Individual Enrollments** tab. Select the *REGISTRATION ID* of the device you enrolled in this Quickstart, and click the **Delete** button at the top. 
4. From the left-hand menu in the Azure portal, click **All resources** and then select your IoT hub. Open **IoT Devices** for your hub, select the *DEVICE ID* of the device you registered in this Quickstart, and then click **Delete** button at the top.

## Next steps

In this Quickstart, you’ve created a TPM simulated device on your machine and provisioned it to your IoT hub using the IoT Hub Device Provisioning Service. To learn how to enroll your TPM device programmatically, continue to the Quickstart for programmatic enrollment of a TPM device. 

> [!div class="nextstepaction"]
> [Azure Quickstart - Enroll TPM device to Azure IoT Hub Device Provisioning Service](quick-enroll-device-tpm-java.md)

