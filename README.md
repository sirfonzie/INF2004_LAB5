# LAB 5: FreeRTOS & Sensor Filtering Algorithm

**OBJECTIVES**
- Configure and implement FreeRTOS on RPi Pico.
- Utilize FreeRTOS functionalities such as Task Creation, Message Buffers and Semaphores.
- Implement a filtering algorithm for processing sensor data.


**EQUIPMENT** 
1.	A laptop that has the Pico C/C++ SDK installed
2.	Raspberry Pico W
3.	Micro-USB Cable

> [NOTE]
> Only students wearing fully covered shoes are allowed in the SR6A lab due to safety.

## **INTRODUCTION** 

Real-Time Operating Systems (RTOS), are operating systems designed to meet the requirements of real-time systems, which need to process inputs and provide outputs without buffering delays. FreeRTOS is a popular open-sourced RTOS that offers straightforward functionality to enable easy, robust, and optimizable design in embedded systems like the Raspberry Pi Pico. FreeRTOS offers many functionalities such as Message Buffers to help inter-task communications and data transfer, whereas Semaphores aid in task synchronization and mutual exclusion, ensuring data consistency.

In this lab, we'll focus on implementing FreeRTOS on Raspberry Pi Pico and explore its features by writing tasks that use Message Buffers and Semaphores. We’ll also cover how to process sensor data using a filtering algorithm.

## **SETTING UP FREERTOS ON RPi PICO** 

We will be using the [Ping example that uses FreeRTOS](https://github.com/raspberrypi/pico-examples/blob/master/pico_w/wifi/freertos/ping/picow_freertos_ping.c). However, we will need to take a few steps to enable it. Currently, you should **only be able to see** it in Explorer and **not be able to see** it in CMake. To set up FreeRTOS on Raspberry Pi Pico, download the [FreeRTOS Kernel](https://github.com/FreeRTOS/FreeRTOS-Kernel) and unzip it onto your computer. Do take note of where the folder is located.

> [NOTE]
> Please note that there seems to be an issue with this example in version Pico-v1.5.0. Therefore, we will be using using Pico-v1.5.1. Windows user can just reinstall the SDK and it will create a new folder or you may go a git pull.

<img src="/img/freertosfolder.png" width=100% height=100%>

Then add three items into the Cmake: Environment
- FREERTOS_KERNEL_PATH: C:\FreeRTOS-Kernel-main
- WIFI_SSID: INF2004
- WIFI_PASSWORD: superduperpassword

There are various methods to do it, but in this example, we will include the path into the CMake environment, not the Windows environment. The images below guide you on how you can include the three items.
<img src="/img/CMakeTools.png" width=100% height=100%>
<img src="/img/CmakeEnvironment.png" width=100% height=100%>

Finally, before you can start compiling your code, include `pico_enable_stdio_usb(picow_freertos_ping_sys 1)` into the CMakeLists.txt file for "picow_freertos_ping_sys"
Do remember to re-select the "Pico ARM GCC" compiler to kick-start the configuration process.

At this point, if everything is done correctly, you should be able to see the project under CMake.
<img src="/img/CMakePing.png" width=100% height=100%>

You should try building the project and downloading the code to the RPi Pico. You should be able to see some output on the Serial Monitor.

## **Create FreeRTOS Tasks**

**Create a Blinking LED Task**

Include the following header files to allow printf and to use the GPIO for the LED.
```
#include <stdio.h>
#include "hardware/gpio.h"
```

The following is the function for blinking the LED.
```
void led_task(__unused void *params) {
    while(true) {
        vTaskDelay(5000);
        cyw43_arch_gpio_put(CYW43_WL_GPIO_LED_PIN, 1);
        vTaskDelay(1000);
        cyw43_arch_gpio_put(CYW43_WL_GPIO_LED_PIN, 0);
    }
    cyw43_arch_deinit();
}
```

Include the following in the `vLaunch` function.
```
    TaskHandle_t ledtask;
    xTaskCreate(led_task, "TestLedThread", configMINIMAL_STACK_SIZE, NULL, 5, &ledtask);
```

**Create a Temperature Sensor Task**




**Create an Average Filtering Task**




## **EXERCISE**


