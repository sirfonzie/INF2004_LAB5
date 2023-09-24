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

To set up FreeRTOS on Raspberry Pi Pico, follow the instructions below:




[FreeRTOS Kernel](https://github.com/FreeRTOS/FreeRTOS-Kernel)

[Ping example that uses FreeRTOS](https://github.com/raspberrypi/pico-examples/blob/master/pico_w/wifi/freertos/ping/picow_freertos_ping.c)

Add three Items into the Cmake: Environment
- FREERTOS_KERNEL_PATH: C:\FreeRTOS-Kernel-main
- WIFI_SSID: INF2004
- WIFI_PASSWORD: superduperpassword

<img src="/img/CMakeTools.png" width=100% height=100%>

<img src="/img/CmakeEnvironment.png" width=100% height=100%>

Include `pico_enable_stdio_usb(picow_freertos_ping_sys 1)` into the CMakeLists.txt file for "picow_freertos_ping_sys"

Reselect the "Pico ARM GCC" compiler to kick-start the configuration process.

## **Create FreeRTOS Tasks**

**Create a Blinking LED Task**
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

**Create a Temperature Sensor Task**




**Create an Average Filtering Task**




## **EXERCISE**


