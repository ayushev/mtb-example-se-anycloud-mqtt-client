# AnyCloud Connectivity using a Secure Element: </br> MQTT Client

This code example demonstrates implementing an MQTT Client using the [AnyCloud MQTT Client library](https://github.com/cypresssemiconductorco/mqtt). The library uses the AWS IoT Device SDK MQTT Client library that includes an MQTT 3.1.1 Client and OPTIGA Trust M Secure Element

In this example, the MQTT Client RTOS task reads out a pre-provisioned X.509 certificate out of the secure element and populates internal MQTT Client configuration to establishe a connection with the configured MQTT Broker, and creates two tasks - Publisher and Subscriber. The Publisher task publishes messages on a topic when the user button is pressed on the kit. The Subscriber task subscribes to the same topic and controls the user LED based on the messages received from the MQTT Broker. In case of unexpected disconnection of MQTT or Wi-Fi connection, the application executes a reconnection mechanism to restore the connection. In addtion all ECDSA and ECDHE related operations performed as part of this demo; i.e. as part of the TLS channel establishment, automatically accelerated on the Secure Element.


**Sequence of operation**

1. The user button is pressed.

2. The GPIO interrupt service routine (ISR) notifies the Publisher task.

3. The Publisher task publishes a message on a topic.

4. The MQTT Broker sends back the message to the MQTT Client because it is also subscribed to the same topic.

5. When the message is received, the Subscriber task turns the LED ON or OFF. As a result, the user LED toggles every time the user presses the button.

[Provide feedback on this code example.](https://cypress.co1.qualtrics.com/jfe/form/SV_1NTns53sK2yiljn?Q_EED=eyJVbmlxdWUgRG9jIElkIjoiQ0UyMjk4ODkiLCJTcGVjIE51bWJlciI6IjAwMi0yOTg4OSIsIkRvYyBUaXRsZSI6IkFueUNsb3VkOiBNUVRUIENsaWVudCIsInJpZCI6InNtcngiLCJEb2MgdmVyc2lvbiI6IjMuMC4wIiwiRG9jIExhbmd1YWdlIjoiRW5nbGlzaCIsIkRvYyBEaXZpc2lvbiI6Ik1DRCIsIkRvYyBCVSI6IklDVyIsIkRvYyBGYW1pbHkiOiJQU09DIn0=)


## Requirements

- [ModusToolbox&trade; software](https://www.cypress.com/products/modustoolbox-software-environment) v2.3 or later

    **Note:** This code example version requires ModusToolbox software version 2.3 or later and is not backward compatible with v2.2 or older versions.

- Board support package (BSP) minimum required version: 2.0.0
- Programming language: C
- Associated parts: All [PSoC&trade; 6 MCU](http://www.cypress.com/PSoC6) parts with SDIO, [PSoC™ 6 MCU with AIROC™ CYW43012 Wi-Fi & Bluetooth® combo chip](https://www.cypress.com/documentation/datasheets/cyw43012-single-chip-ultra-low-power-ieee-80211n-compliant-ieee-80211ac), [PSoC™ 6 MCU with AIROC™ CYW4343W Wi-Fi & Bluetooth® combo chip](https://www.cypress.com/documentation/datasheets/cyw4343w-single-chip-80211-bgn-macbasebandradio-bluetooth-51)


## Supported toolchains (make variable 'TOOLCHAIN')

- GNU Arm® embedded compiler v9.3.1 (`GCC_ARM`) - Default value of `TOOLCHAIN`
- Arm compiler v6.13 (`ARM`)
- IAR C/C++ compiler v8.42.2 (`IAR`)


## Supported kits (make variable 'TARGET')

- Rapid IoT Connect Developer Kit (`CYSBSYSKIT-DEV-01`) - Default value of TARGET

## Hardware setup

This example uses the board's default configuration. See the kit user guide to ensure that the board is configured correctly.

## Software setup

Install a terminal emulator if you don't have one. Instructions in this document use [Tera Term](https://ttssh2.osdn.jp/index.html.en).

This code example implements a generic MQTT Client that can connect to various MQTT Brokers. In this document, the instructions to set up and run the MQTT Client have been provided for the AWS IoT and Mosquitto MQTT Brokers for reference. If you are using this code example with Mosquitto broker running locally on your computer, you need to download and install Mosquitto Broker from https://mosquitto.org/download.

This example requires no additional software or tools if you are using the MQTT Client with a publicly hosted MQTT Broker.

## Using the code example

Create the project and open it using one of the following:

<details><summary><b>In Eclipse IDE for ModusToolbox</b></summary>

1. Click the **New Application** link in the **Quick Panel** (or, use **File** > **New** > **ModusToolbox Application**). This launches the [Project Creator](http://www.cypress.com/ModusToolboxProjectCreator) tool.

2. Pick a kit supported by the code example from the list shown in the **Project Creator - Choose Board Support Package (BSP)** dialog.

   When you select a supported kit, the example is reconfigured automatically to work with the kit. To work with a different supported kit later, use the [Library Manager](https://www.cypress.com/ModusToolboxLibraryManager) to choose the BSP for the supported kit. You can use the Library Manager to select or update the BSP and firmware libraries used in this application. To access the Library Manager, click the link from the **Quick Panel**.

   You can also just start the application creation process again and select a different kit.

   If you want to use the application for a kit not listed here, you may need to update the source files. If the kit does not have the required resources, the application may not work.

3. In the **Project Creator - Select Application** dialog, choose the example by enabling the checkbox.

4. Optionally, change the suggested **New Application Name**.

5. Enter the local path in the **Application(s) Root Path** field to indicate where the application needs to be created.

   Applications that can share libraries can be placed in the same root path.

6. Click **Create** to complete the application creation process.

For more details, see the [Eclipse IDE for ModusToolbox User Guide](https://www.cypress.com/MTBEclipseIDEUserGuide) (locally available at *{ModusToolbox install directory}/ide_{version}/docs/mt_ide_user_guide.pdf*).

</details>

<details><summary><b>In command-line interface (CLI)</b></summary>

ModusToolbox provides the Project Creator as both a GUI tool and a command line tool to easily create one or more ModusToolbox applications. See the "Project Creator Tools" section of the [ModusToolbox User Guide](https://www.cypress.com/ModusToolboxUserGuide) for more details.

Alternatively, you can manually create the application using the following steps:

1. Download and unzip this repository onto your local machine, or clone the repository.

2. Open a CLI terminal and navigate to the application folder.

   On Windows, use the command line `modus-shell` program provided in the ModusToolbox installation instead of a standard Windows command line application. This shell provides access to all ModusToolbox tools. You can access it by typing `modus-shell` in the search box in the Windows menu.

   In Linux and macOS, you can use any terminal application.

   **Note:** The cloned application contains a default BSP file (*TARGET_xxx.mtb*) in the *deps* folder. Use the [Library Manager](https://www.cypress.com/ModusToolboxLibraryManager) (`make modlibs` command) to select and download a different BSP file, if required. If the selected kit does not have the required resources or is not [supported](#supported-kits-make-variable-target), the application may not work.

3. Import the required libraries by executing the `make getlibs` command.

Various CLI tools include a `-h` option that prints help information to the terminal screen about that tool. For more details, see the [ModusToolbox User Guide](https://www.cypress.com/ModusToolboxUserGuide) (locally available at *{ModusToolbox install directory}/docs_{version}/mtb_user_guide.pdf*).

</details>

<details><summary><b>In Third-party IDEs</b></summary>

1. Follow the instructions from the **In command-line interface (CLI)** section to create the application, and import the libraries using the `make getlibs` command.

2. Export the application to a supported IDE using the `make <ide>` command.

   For a list of supported IDEs and more details, see the "Exporting to IDEs" section of the [ModusToolbox User Guide](https://www.cypress.com/ModusToolboxUserGuide) (locally available at *{ModusToolbox install directory}/docs_{version}/mtb_user_guide.pdf*).

3. Follow the instructions displayed in the terminal to create or import the application as an IDE project.

</details>

## Operation

1. Connect the board to your PC using the provided USB cable through the KitProg3 USB connector.

2. Modify the user configuration files in the *configs* directory as follows:

    1. **Wi-Fi Configuration:** Set the Wi-Fi credentials in *configs/wifi_config.h*: Modify the macros `WIFI_SSID`, `WIFI_PASSWORD`, and `WIFI_SECURITY` to match with that of the Wi-Fi network that you want to connect.

    2. **MQTT Configuration:** Set up the MQTT Client and configure the credentials in *configs/mqtt_client_config.h*. Some of the important configuration macros are as follows:
      
        * **AWS IoT MQTT:**  Set up the MQTT device (also known as a *Thing*) in the AWS IoT Core as described in the [Getting started with AWS IoT tutorial](https://docs.aws.amazon.com/iot/latest/developerguide/iot-gs.html). Please don't create neither a client certificate not a corresponding private key as they will be provided by the Secure Element

            **Note:** While setting up your device, ensure that the policy associated with this device permits all MQTT operations (*iot:Connect*, *iot:Publish*, *iot:Receive*, and *iot:Subscribe*) for the resource used by this device. For testing purposes, it is recommended to have the following policy document which allows all *MQTT Policy Actions* on all *Amazon Resource Names (ARNs)*.
            ```
            {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Effect": "Allow",
                        "Action": "iot:*",
                        "Resource": "*"
                    }
                ]
            }
            ```
   
        * Program the board using one of the following (you don't need any extra configuration for the first part):

            <details><summary><b>Using Eclipse IDE for ModusToolbox</b></summary>

                  1. Select the application project in the Project Explorer.

                  2. In the **Quick Panel**, scroll down, and click **\<Application Name> Program (KitProg3_MiniProg4)**.
            </details>

            <details><summary><b>Using CLI</b></summary>

                 From the terminal, execute the `make program` command to build and program the application using the default toolchain to the default target. You can specify a target and toolchain manually:
                  ```
                  make program TARGET=<BSP> TOOLCHAIN=<toolchain>
                  ```

                  Example:
                  ```
                  make program TARGET=CYSBSYSKIT-DEV-01 TOOLCHAIN=GCC_ARM
                  ```
               </details>

               After programming, the application starts automatically. Observe the messages on the UART terminal, and wait for the device to make all the required connections.

               **Figure 1. Application initialization status**

               ![](images/application_initialization.png)

        * Open a terminal program and select the KitProg3 COM port. Set the serial port parameters to 8N1 and 115200 baud.
        
        * You should see a similar output.
          <details><summary>TeraTerm output</summary>
             <img src="https://github.com/ayushev/mtb-example-se-anycloud-mqtt-client/raw/master/images/application_initialization.png">
          </details>
            This is a base64 encoded pre-provisioned certificate (starting from `-----BEGIN CERTIFICATE-----` till `-----END CERTIFICATE-----`) which belongs to your board. Please follow the steps mentioned in the terminal output after the certificate output.

        * In the *configs/mqtt_client_config.h* file, set `MQTT_BROKER_ADDRESS` and `MQTT_SNI_HOSTNAME` to your custom endpoint on the **Settings** page of the AWS IoT Console. This has the format `ABCDEFG1234567-ats.iot.<region>.amazonaws.com`. To get you endpoint:

            - Browse to the [AWS IoT console](https://console.aws.amazon.com).
            - In the navigation pane, choose Settings.
            - Your AWS IoT endpoint is displayed in Endpoint. It should look like 1234567890123-ats.iot.us-east-1.amazonaws.com. Make a note of this endpoint.

        * Set the macros `MQTT_PORT` to `8883` and `MQTT_SECURE_CONNECTION` to `1` in the *configs/mqtt_client_config.h* file.

        * Download the following certificate(-s):
            - Root CA "ECC 256 bit key: Amazon Root CA 3" for AWS IoT from [CA Certificates for Server Authentication](https://docs.aws.amazon.com/iot/latest/developerguide/server-authentication.html#server-authentication-certs). In case the ECDHE_ECDSA based TLS ciphersuite is selected should be the [Amazon Root CA 3](https://www.amazontrust.com/repository/AmazonRootCA3.pem) (Default), in case the ECDHE_RSA based TLS ciphersuite is in use another CA should be selected: [Amazon Root CA 1](https://www.amazontrust.com/repository/AmazonRootCA1.pem)

        * Using these certificates, enter the following parameters in *mqtt_client_config.h* in Privacy-Enhanced Mail (PEM) format:
            - `ROOT_CA_CERTIFICATE` - Root CA certificate

              You can either convert the values to strings manually following the format shown in *mqtt_client_config.h* or you can use the HTML utility available [here](https://github.com/cypresssemiconductorco/amazon-freertos/blob/master/tools/certificate_configuration/PEMfileToCString.html) to convert the certificates and keys from PEM format to C string format. You need to clone the repository from GitHub to use the utility.

         See [Setting up the MQTT Broker](#setting-up-the-mqtt-broker) to learn how to configure these macros for AWS IoT.

         For a full list of configuration macros used in this code example, see [Wi-Fi and MQTT configuration macros](#wi-fi-and-mqtt-configuration-macros).

      3. Other configuration files: You can optionally modify the configuration macros in the following files according to your application:
         - *configs/core_mqtt_config.h* used by the [MQTT library](https://github.com/cypresssemiconductorco/mqtt)

         - *configs/FreeRTOSConfig.h* used by the [FreeRTOS library](https://github.com/cypresssemiconductorco/freertos)

5. Once the initialization is complete, confirm that the message *"Press the user button (SW2) to publish "TURN ON"/"TURN OFF" on the topic 'ledstatus'..."* is printed on the UART terminal. This message may vary depending on the MQTT topic and publish messages that are configured in the *mqtt_client_config.h* file.

6. Press the user button (SW2) on the kit to toggle the LED state.

7. Confirm that the user LED state is toggled and the messages received on the subscribed topic are printed on the UART terminal.

   **Figure 2. Publisher and Subscriber logs**

   ![](images/publish_subscribe_messages.png)

This example can be programmed on multiple kits (*Only when `GENERATE_UNIQUE_CLIENT_ID` is set to `1`*); the user LEDs on all the kits will synchronously toggle with button presses on any kit.

Alternatively, the publish and subscribe functionalities of the MQTT Client can be individually verified if the MQTT Broker supports a Test MQTT Client like the AWS IoT.

- *To verify the subscribe functionality*: Using the Test MQTT Client, publish messages such as "TURN ON" and "TURN OFF" on the topic specified by the `MQTT_PUB_TOPIC` macro in *mqtt_client_config.h* to control the LED state on the kit.

- *To verify the publish functionality*: From the Test MQTT Client, subscribe to the MQTT topic specified by the `MQTT_SUB_TOPIC` macro and confirm that the messages published by the kit (when the user button is pressed) are displayed on the Test MQTT Client's console.

## Debugging

You can debug the example to step through the code. In the IDE, use the **\<Application Name> Debug (KitProg3_MiniProg4)** configuration in the **Quick Panel**. For more details, see the "Program and debug" section in the [Eclipse IDE for ModusToolbox User Guide](https://www.cypress.com/MTBEclipseIDEUserGuide).

**Note:** **(Only while debugging)** On the CM4 CPU, some code in `main()` may execute before the debugger halts at the beginning of `main()`. This means that some code executes twice - once before the debugger stops execution, and again after the debugger resets the program counter to the beginning of `main()`. See [KBA231071](https://community.cypress.com/docs/DOC-21143) to learn about this and for the workaround.

## Design and implementation

This example implements four RTOS tasks: OPTIGA Trust, MQTT Client, Publisher, and Subscriber. The main function initializes the BSP and the retarget-io library, and creates the OPTIGA Trust task.

The OPTIGA Trust task initializes the secure element, extracts pre-provisioned public key certificate out of ther chip and populates with it internal configuration for the secure communication. Afterwards it initializes the MQTT Client task.
               
The MQTT Client task initializes the Wi-Fi connection manager (WCM) and connects to a Wi-Fi access point (AP) using the Wi-Fi network credentials that are configured in *wifi_config.h*. Upon a successful Wi-Fi connection, the task initializes the MQTT library and establishes a connection with the MQTT Broker/Server.

The MQTT connection is configured to be secure by default; the secure connection requires a client certificate, a private key, and the Root CA certificate of the MQTT Broker that are configured in *mqtt_client_config.h*.

After a successful MQTT connection, the Subscriber and Publisher tasks are created. The MQTT Client task then waits for commands from the other two tasks and callbacks to handle events like unexpected disconnections.

The Subscriber task initializes the user LED GPIO and subscribes to messages on the topic specified by the `MQTT_SUB_TOPIC` macro that can be configured in *mqtt_client_config.h*. When the Subscriber task receives a message from the Broker, it turns the user LED ON or OFF depending on whether the received message is "TURN ON" or "TURN OFF" (configured using the `MQTT_DEVICE_ON_MESSAGE` and `MQTT_DEVICE_OFF_MESSAGE` macros).

The Publisher task sets up the user button GPIO and configures an interrupt for the button. The ISR notifies the Publisher task upon a button press. The Publisher task then publishes messages (*TURN ON* / *TURN OFF*) on the topic specified by the `MQTT_PUB_TOPIC` macro. When the publish operation fails, a message is sent over a queue to the MQTT Client task.

An MQTT event callback function `mqtt_event_callback()` invoked by the MQTT library for events like MQTT disconnection and incoming MQTT subscription messages from the MQTT broker. In the case of an MQTT disconnection, the MQTT client task is informed about the disconnection using a message queue. When an MQTT subscription message is received, the subscriber callback function implemented in *subscriber_task.c* is invoked to handle the incoming MQTT message.

The MQTT client task handles unexpected disconnections in the MQTT or Wi-Fi connections by initiating reconnection to restore the Wi-Fi and/or MQTT connections. Upon failure, the Publisher and Subscriber tasks are deleted, cleanup operations of various libraries are performed, and then the MQTT client task is terminated.
               
## How to enable your Anycloud based example to use the OPTIGA Trust library
               
There are several key points which should be considered:
   
   1. Hardware configuration
       * I2C Pins: supported boards define `CYBSP_I2C_SCL_OPTIGA` and `CYBSP_I2C_SDA_OPTIGA` in their BSP, alternativly you can define in the [`optiga_lib_mtb_config.h`](https://github.com/ayushev/mtb-example-se-anycloud-mqtt-client/blob/8149c37ae4026f28037116370580b766ec5f58b8/source/optiga_lib_config_mtb.h#L161-L162) file your own definitions for them.
       * Reset and Power Control Pins: the OPTIGA Trust secure element can be controlled via a deddicated reset and a power control line, for instance the latter is used for entering a so-called hibernation mode on the chip. Define `OPTIGA_TRUSTM_VDD` and `OPTIGA_TRUSTM_RST` with corresponding GPIOs in your `optiga_lib_config_mtb.h` file. In addition to this change the reset type also in the same [configuration file](https://github.com/ayushev/mtb-example-se-anycloud-mqtt-client/blob/master/source/optiga_lib_config_mtb.h#L114) to 0 if you have both defined, in case you have only the reset line connected this value should be 2.
   2. Software configuration
       * **Initialisation in a FreeRTOS task**
           * If the [Makefile](https://github.com/ayushev/mtb-example-se-anycloud-mqtt-client/blob/987b08e691fa3d1a66c0f8324c4fa5c2a00dce9e/Makefile#L81) defines `PSOC_FREERTOS` this means that the OPTIGA Trust library will use a FreeRTOS based Platform Abstraction Layer for the communication, this leads to the following guidelines.
           * Always initialise the Secure Element from a task
           * Give enough Stack to the OPTIGA Trust task, depending on the configuration it needs not more than 3072 bytes
           * Don't start any MQTT related tasks before the Secure Element will be initialised
       * **Certificate extraction and MQTT connection configuration**
           * Right after the Secure Element will be initialised you might need to extract the certificate out of the chip and assign it to the internal MQTT client configuration. An example of this can be found [here](https://github.com/ayushev/mtb-example-se-anycloud-mqtt-client/blob/987b08e691fa3d1a66c0f8324c4fa5c2a00dce9e/source/mqtt_client_config.c#L171). 
       * mbedTLS configuration
           * Handshake methods
               * For a successfull TLS communication make sure only supported handshake methods are selected in you mbedtls configuration file. For this **either undefine** the following
                   * `MBEDTLS_KEY_EXCHANGE_DHE_PSK_ENABLED`
                   * `MBEDTLS_KEY_EXCHANGE_ECDHE_PSK_ENABLED`
                   * `MBEDTLS_KEY_EXCHANGE_RSA_PSK_ENABLED`
                   * `MBEDTLS_KEY_EXCHANGE_RSA_ENABLED`
                   * `MBEDTLS_KEY_EXCHANGE_DHE_RSA_ENABLED`
                   * `MBEDTLS_KEY_EXCHANGE_ECDH_RSA_ENABLED`
                   * `MBEDTLS_KEY_EXCHANGE_ECDHE_RSA_ENABLED`, **or define** the following
                   * `MBEDTLS_KEY_EXCHANGE_ECDHE_ECDSA_ENABLED`
               * In case of a WiFi online provisioning your example will try to esatblish a conneciton with several clouds, thus restricts usage of many handshake methods. For instance the `MBEDTLS_KEY_EXCHANGE_ECDHE_ECDSA_ENABLED` method might be not available. You can opt to the `MBEDTLS_KEY_EXCHANGE_ECDHE_RSA_ENABLED` and change the Root CA configuration `ROOT_CA_CERTIFICATE` to use the Amazon Root CA 1 (RSA 2048 based), instead of the default value of Amazon Root CA 3 (ECC 256 based)
           * Cryptography (ECDSA, ECDHE, Random) functions call routing
               * Your mbedtls configuration file should have the following macros defined
                   * `MBEDTLS_ECDH_GEN_PUBLIC_ALT`
                   * `MBEDTLS_ECDSA_SIGN_ALT`
                   * `MBEDTLS_ECDSA_VERIFY_ALT`
                   * `MBEDTLS_ECDH_COMPUTE_SHARED_ALT`
                   * `MBEDTLS_ECDSA_GENKEY_ALT`
               * In addtion to above macros make sure your build includes `$(optiga-trust-m)/examples/mbedtls_port` files

### Configuring the MQTT Client
      
#### Wi-Fi and MQTT configuration macros

| Macro                               |  Description              |
| :---------------------------------- | :------------------------ |
| **Wi-Fi Connection Configurations**  |  In *configs/wifi_config.h*  |
| `WIFI_SSID`       | SSID of the Wi-Fi AP to which the MQTT Client connects   |
| `WIFI_PASSWORD`   | Passkey/password for the Wi-Fi SSID specified above                 |
| `WIFI_SECURITY`   | Security type of the Wi-Fi AP. See `cy_wcm_security_t` structure in *cy_wcm.h* file for more details.      |
| `MAX_WIFI_CONN_RETRIES`   | Maximum number of retries for Wi-Fi connection   |
| `WIFI_CONN_RETRY_INTERVAL_MS`   | Time interval in milliseconds in between successive Wi-Fi connection retries   |
| **MQTT Connection Configurations**  |  In *configs/mqtt_client_config.h*  |
| `MQTT_BROKER_ADDRESS`      | Hostname of the MQTT Broker        |
| `MQTT_PORT`                | Port number to be used for the MQTT connection. As specified by IANA (Internet Assigned Numbers Authority), port numbers assigned for MQTT protocol are *1883* for non-secure connections and *8883* for secure connections. However, MQTT Brokers may use other ports. Configure this macro as specified by the MQTT Broker.                      |
| `MQTT_SECURE_CONNECTION`   | Set this macro to `1` if a secure (TLS) connection to the MQTT Broker is  required to be established; else `0`.         |
| `MQTT_USERNAME` <br> `MQTT_PASSWORD`   | User name and password for client authentication and authorization, if required by the MQTT Broker. However, note that this information is generally not encrypted and the password is sent in plain text. Therefore, this is not a recommended method of client authentication. |
| **MQTT Client Certificate Configurations**  |  In *configs/mqtt_client_config.h*  |
| `CLIENT_CERTIFICATE` <br> `CLIENT_PRIVATE_KEY`  | Certificate and private key of the MQTT Client used for client authentication. Note that these macros are applicable only when `MQTT_SECURE_CONNECTION` is set to `1`.      |
| `ROOT_CA_CERTIFICATE`      |  Root CA certificate of the MQTT Broker |
| **MQTT Message Configurations**    |  In *configs/mqtt_client_config.h*  |
| `MQTT_PUB_TOPIC`           | MQTT topic to which the messages are published by the Publisher task to the MQTT Broker    |
| `MQTT_SUB_TOPIC`           | MQTT topic to which the Subscriber task subscribes to. The MQTT Broker sends the messages to the Subscriber that are published in this topic (or equivalent topic).  |
| `MQTT_MESSAGES_QOS`        | The Quality of Service (QoS) level to be used by the Publisher and Subscriber. Valid choices are `0`, `1`, and `2`.  |
| `ENABLE_LWT_MESSAGE`       | Set this macro to `1` if you want to use the 'Last Will and Testament (LWT)' option; else `0`. LWT is an MQTT message that will be published by the MQTT Broker on the specified topic if the MQTT connection is unexpectedly closed. This configuration is sent to the MQTT Broker during MQTT connect operation; the MQTT Broker will publish the Will message on the Will topic when it recognizes an unexpected disconnection from the client. |
| `MQTT_WILL_TOPIC_NAME` <br> `MQTT_WILL_MESSAGE`   | The MQTT topic and message for the LWT option described above. These configurations are applicable only when `ENABLE_LWT_MESSAGE` is set to `1`.  |
| `MQTT_DEVICE_ON_MESSAGE` <br> `MQTT_DEVICE_OFF_MESSAGE`  | The MQTT messages that control the device (LED) state in this code example.  |
| **Other MQTT Client Configurations**    |  In *configs/mqtt_client_config.h*  |
| `GENERATE_UNIQUE_CLIENT_ID`   | Every active MQTT connection must have a unique client identifier. If this macro is set to `1`, the device will generate a unique client identifier by appending a timestamp to the string specified by the `MQTT_CLIENT_IDENTIFIER` macro. This feature is useful if you are using the same code on multiple kits simultaneously.   |
| `MQTT_CLIENT_IDENTIFIER`     | The client identifier (client ID) string to be used during MQTT connection. If `GENERATE_UNIQUE_CLIENT_ID` is set to `1`, a timestamp is appended to this macro value and used as the client ID; else, the value specified for this macro is directly used as the client ID.   |
| `MQTT_CLIENT_IDENTIFIER_MAX_LEN`   | The longest client identifier that an MQTT server must accept (as defined by the MQTT 3.1.1 spec) is 23 characters. However, some MQTT Brokers support longer client IDs. Configure this macro as per the MQTT Broker specification.  |
| `MQTT_TIMEOUT_MS`            | Timeout in milliseconds for MQTT operations in this example   |
| `MQTT_KEEP_ALIVE_SECONDS`    | The keepalive interval in seconds used for MQTT ping request   |
| `MQTT_ALPN_PROTOCOL_NAME`   | The Application Layer Protocol Negotiation (ALPN) protocol name to be used that is supported by the MQTT Broker in use. Note that this is an optional macro for most of the use cases. <br>Per IANA, the port numbers assigned for MQTT protocol are 1883 for non-secure connections and 8883 for secure connections. In some cases, there is a need to use other ports for MQTT like port 443 (which is reserved for HTTPS). ALPN is an extension to TLS that allows many protocols to be used over a secure connection.     |
| `MQTT_SNI_HOSTNAME`   | The Server Name Indication (SNI) host name to be used during the Transport Layer Security (TLS) connection as specified by the MQTT Broker. <br>SNI is extension to the TLS protocol. As required by some MQTT Brokers, SNI typically includes the hostname in the "Client Hello" message sent during TLS handshake.     |
| `MQTT_NETWORK_BUFFER_SIZE`   | A network buffer is allocated for sending and receiving MQTT packets over the network. Specify the size of this buffer using this macro. Note that the minimum buffer size is defined by `CY_MQTT_MIN_NETWORK_BUFFER_SIZE` macro in the MQTT library.  |
| `MAX_MQTT_CONN_RETRIES`   | Maximum number of retries for MQTT connection   |
| `MQTT_CONN_RETRY_INTERVAL_MS`   | Time interval in milliseconds in between successive MQTT connection retries   |

### Resources and settings

**Table 1. Application resources**

| Resource  |  Alias/Object     |    Purpose     |
| :------- | :------------    | :------------ |
| UART (HAL)|cy_retarget_io_uart_obj| UART HAL object used by Retarget-IO for Debug UART port  |
| GPIO (HAL)    | CYBSP_USER_LED         | User LED controlled by the Subscriber based on incoming MQTT messages  |
| GPIO (HAL)    | CYBSP_USER_BTN         | User Button used to notify the Publisher to publish MQTT messages |

## Related resources

| Application notes                                            |                                                              |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [AN228571](https://www.cypress.com/AN228571) – Getting started with PSoC 6 MCU on ModusToolbox | Describes PSoC 6 MCU devices and how to build your first application with ModusToolbox |
| [AN221774](https://www.cypress.com/AN221774) – Getting started with PSoC 6 MCU on PSoC Creator | Describes PSoC 6 MCU devices and how to build your first application with PSoC Creator |
| [AN210781](https://www.cypress.com/AN210781) – Getting started with PSoC 6 MCU with Bluetooth Low Energy connectivity on PSoC Creator | Describes PSoC 6 MCU with Bluetooth LE connectivity devices and how to build your first application with PSoC Creator |
| [AN215656](https://www.cypress.com/AN215656) – PSoC 6 MCU: dual-CPU system design | Describes the dual-CPU architecture in PSoC 6 MCU, and shows how to build a simple dual-CPU design |
| **Code examples**                                            |                                                              |
| [Using ModusToolbox](https://github.com/cypresssemiconductorco/Code-Examples-for-ModusToolbox-Software) | [Using PSoC Creator](https://www.cypress.com/documentation/code-examples/psoc-6-mcu-code-examples) |
| **Device documentation**                                     |                                                              |
| [PSoC 6 MCU datasheets](https://www.cypress.com/search/all?f[0]=meta_type%3Atechnical_documents&f[1]=resource_meta_type%3A575&f[2]=field_related_products%3A114026) | [PSoC 6 technical reference manuals](https://www.cypress.com/search/all/PSoC%206%20Technical%20Reference%20Manual?f[0]=meta_type%3Atechnical_documents&f[1]=resource_meta_type%3A583) |
| **Development kits**                                         | Buy at www.cypress.com                                       |
| [CYSBSYSKIT-DEV-01](https://github.com/cypresssemiconductorco/TARGET_CYSBSYSKIT-DEV-01) Rapid IoT Connect Developer Kit |  |
| **Libraries**                                                |                                                              |
| PSoC 6 peripheral driver library (PDL) and docs  | [mtb-pdl-cat1](https://github.com/cypresssemiconductorco/mtb-pdl-cat1) on GitHub |
| Hardware abstraction layer (HAL) Library and docs    | [mtb-hal-cat1](https://github.com/cypresssemiconductorco/mtb-hal-cat1) on GitHub |
| Retarget IO - A utility library to retarget the standard input/output (STDIO) messages to a UART port | [retarget-io](https://github.com/cypresssemiconductorco/retarget-io) on GitHub |
| **Middleware**                                               |                                                              |
| MQTT Client library and documents                            | [mqtt](https://github.com/infineon/mqtt) on GitHub |
| Wi-Fi connection manager (WCM) library and documents         | [wifi-connection-manager](https://github.com/infineon/wifi-connection-manager) on GitHub |
| Wi-Fi middleware core library and documents                  | [wifi-mw-core](https://github.com/infineon/wifi-mw-core) on GitHub|
| FreeRTOS library and documents                               | [freeRTOS](https://github.com/infineon/freertos) on GitHub |
| CapSense&trade; library and documents                        | [capsense](https://github.com/infineon/capsense) on GitHub |
| OPTIGA™ Trust M library                                      | [optiga-trust-m](https://github.com/infineon/optiga-trust-m) on GitHub |
| Links to all PSoC 6 MCU middleware                           | [psoc6-middleware](https://github.com/infineon/psoc6-middleware) on GitHub |
| **Tools**                                                    |                                                              |
| [Eclipse IDE for ModusToolbox](https://www.cypress.com/modustoolbox) | The cross-platform, Eclipse-based IDE for IoT designers that supports application configuration and development targeting converged MCU and wireless systems. |
| [PSoC Creator™](https://www.cypress.com/products/psoc-creator-integrated-design-environment-ide) | The legacy Cypress IDE for PSoC and FM0+ MCU development. |

## Other resources

Cypress provides a wealth of data at www.cypress.com to help you select the right device, and quickly and effectively integrate it into your design.

For PSoC 6 MCU devices, see [How to design with PSoC 6 MCU - KBA223067](https://community.cypress.com/docs/DOC-14644) in the Cypress community.

## Document history

Document title: *CE229889* - *AnyCloud: MQTT Client*

| Version | Description of Change |
| ------- | --------------------- |
| 1.2.0   | New code example      |
------

All other trademarks or registered trademarks referenced herein are the property of their respective owners.

![banner](images/ifx-cy-banner.png)

-------------------------------------------------------------------------------

© Cypress Semiconductor Corporation, 2016-2021. This document is the property of Cypress Semiconductor Corporation, an Infineon Technologies company, and its affiliates ("Cypress").  This document, including any software or firmware included or referenced in this document ("Software"), is owned by Cypress under the intellectual property laws and treaties of the United States and other countries worldwide.  Cypress reserves all rights under such laws and treaties and does not, except as specifically stated in this paragraph, grant any license under its patents, copyrights, trademarks, or other intellectual property rights.  If the Software is not accompanied by a license agreement and you do not otherwise have a written agreement with Cypress governing the use of the Software, then Cypress hereby grants you a personal, non-exclusive, nontransferable license (without the right to sublicense) (1) under its copyright rights in the Software (a) for Software provided in source code form, to modify and reproduce the Software solely for use with Cypress hardware products, only internally within your organization, and (b) to distribute the Software in binary code form externally to end users (either directly or indirectly through resellers and distributors), solely for use on Cypress hardware product units, and (2) under those claims of Cypress’s patents that are infringed by the Software (as provided by Cypress, unmodified) to make, use, distribute, and import the Software solely for use with Cypress hardware products.  Any other use, reproduction, modification, translation, or compilation of the Software is prohibited.
<br>
TO THE EXTENT PERMITTED BY APPLICABLE LAW, CYPRESS MAKES NO WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, WITH REGARD TO THIS DOCUMENT OR ANY SOFTWARE OR ACCOMPANYING HARDWARE, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE.  No computing device can be absolutely secure.  Therefore, despite security measures implemented in Cypress hardware or software products, Cypress shall have no liability arising out of any security breach, such as unauthorized access to or use of a Cypress product.  CYPRESS DOES NOT REPRESENT, WARRANT, OR GUARANTEE THAT CYPRESS PRODUCTS, OR SYSTEMS CREATED USING CYPRESS PRODUCTS, WILL BE FREE FROM CORRUPTION, ATTACK, VIRUSES, INTERFERENCE, HACKING, DATA LOSS OR THEFT, OR OTHER SECURITY INTRUSION (collectively, "Security Breach").  Cypress disclaims any liability relating to any Security Breach, and you shall and hereby do release Cypress from any claim, damage, or other liability arising from any Security Breach.  In addition, the products described in these materials may contain design defects or errors known as errata which may cause the product to deviate from published specifications.  To the extent permitted by applicable law, Cypress reserves the right to make changes to this document without further notice. Cypress does not assume any liability arising out of the application or use of any product or circuit described in this document.  Any information provided in this document, including any sample design information or programming code, is provided only for reference purposes.  It is the responsibility of the user of this document to properly design, program, and test the functionality and safety of any application made of this information and any resulting product.  "High-Risk Device" means any device or system whose failure could cause personal injury, death, or property damage.  Examples of High-Risk Devices are weapons, nuclear installations, surgical implants, and other medical devices.  "Critical Component" means any component of a High-Risk Device whose failure to perform can be reasonably expected to cause, directly or indirectly, the failure of the High-Risk Device, or to affect its safety or effectiveness.  Cypress is not liable, in whole or in part, and you shall and hereby do release Cypress from any claim, damage, or other liability arising from any use of a Cypress product as a Critical Component in a High-Risk Device.  You shall indemnify and hold Cypress, including its affiliates, and its directors, officers, employees, agents, distributors, and assigns harmless from and against all claims, costs, damages, and expenses, arising out of any claim, including claims for product liability, personal injury or death, or property damage arising from any use of a Cypress product as a Critical Component in a High-Risk Device.  Cypress products are not intended or authorized for use as a Critical Component in any High-Risk Device except to the limited extent that (i) Cypress’s published data sheet for the product explicitly states Cypress has qualified the product for use in a specific High-Risk Device, or (ii) Cypress has given you advance written authorization to use the product as a Critical Component in the specific High-Risk Device and you have signed a separate indemnification agreement.
<br>
Cypress, the Cypress logo, and combinations thereof, WICED, ModusToolBox, PSoC, CapSense, EZ-USB, F-RAM, and Traveo are trademarks or registered trademarks of Cypress or a subsidiary of Cypress in the United States or in other countries.  For a more complete list of Cypress trademarks, visit cypress.com.  Other names and brands may be claimed as property of their respective owners.
