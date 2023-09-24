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
> Please note that there seems to be an issue with this example in version Pico-v1.5.0. Therefore, we will be using using Pico-v1.5.1. Windows users can just reinstall the SDK and it will create a new folder or you may go a git pull.

<img src="/img/freertosfolder.png" width=100% height=100%>

Then add three items into the Cmake: Environment
- FREERTOS_KERNEL_PATH: C:\FreeRTOS-Kernel-main
- WIFI_SSID: INF2004
- WIFI_PASSWORD: superduperpassword

There are various methods to do it, but in this example, we will include the path into the CMake environment, not the Windows environment. The images below guide you on how you can include the three items.
<img src="/img/CMakeTools.png" width=100% height=100%>
<img src="/img/CmakeEnvironment.png" width=100% height=100%>

Next, include `pico_enable_stdio_usb(picow_freertos_ping_sys 1)` into the CMakeLists.txt file for "picow_freertos_ping_sys" to allow serial monitoring via USB.
Do remember to re-select the "Pico ARM GCC" compiler to kick-start the configuration process.

<img src="/img/NoSMP.PNG" width=100% height=100%>

Finally, before you can start compiling your code, make the following changes (lines #107 & #110) to the FreeRTOS configuration. The configuration file is located at "pico_w\wifi\freertos\ping\FreeRTOSConfig.h". 

```
#if FREE_RTOS_KERNEL_SMP // set by the RP2040 SMP port of FreeRTOS
/* SMP port only */
#define configNUM_CORES                         1
#define configTICK_CORE                         0
#define configRUN_MULTIPLE_PRIORITIES           1
#define configUSE_CORE_AFFINITY                 0
#endif
```

At this point, if everything is done correctly, you should be able to see the project under CMake.
<img src="/img/CMakePing.png" width=100% height=100%>


You should try building the project and downloading the code to the RPi Pico. You should be able to see some output on the Serial Monitor.

## **Create FreeRTOS Tasks**
Currently, we are operating a single task that directs its focus on pinging a specified IP address. Nonetheless, given that we are utilizing a Real-Time Operating System (RTOS), our system is inherently designed to execute multiple tasks concurrently. Consequently, our subsequent task will logically involve constructing an additional task—a task designated to manage the blinking of an LED.

**Create a Blinking LED Task**

Include the following header files to allow printf and to use the GPIO for the LED.
```
#include <stdio.h>
#include "hardware/gpio.h"
```

The following is the function for blinking the LED. Notice that each task is a forever loop. In this example, the LED will turn on for 2 seconds and turn off for 2 seconds.
```
void led_task(__unused void *params) {
    while(true) {
        vTaskDelay(2000);
        cyw43_arch_gpio_put(CYW43_WL_GPIO_LED_PIN, 1);
        vTaskDelay(2000);
        cyw43_arch_gpio_put(CYW43_WL_GPIO_LED_PIN, 0);
    }
}
```

Include the following in the `vLaunch` function. This will create the instance of the task for execution. TaskHandle is used for managing the task, i.e. deleting, changing priority, etc.
```
    TaskHandle_t ledtask;
    xTaskCreate(led_task, "TestLedThread", configMINIMAL_STACK_SIZE, NULL, 5, &ledtask);
```

**Create a Temperature Sensor Task**

The RP2040 microcontroller, embedded in the Raspberry Pi Pico, comes with an inbuilt temperature sensor, allowing developers to easily access and measure the ambient temperature. This sensor is part of the RP2040’s ADC (Analog to Digital Converter) system, which means it converts the analog temperature value to a digital representation that can be processed and read by the MCU. In this section, we’ll discuss how to access this built-in temperature sensor to retrieve temperature data. To access the temperature sensor data, we use the ADC to read the analog voltage and then convert this analog value to a temperature reading in degrees Celsius. Here is a simple step-by-step guide to achieving this.


Include the following header. This is to digitize the data from the inbuilt temperature sensor within RP2040.
```
#include "hardware/adc.h"
```

The following are the functions to obtain the temperature from the RP2040 and print it out via the serial. In the `temp_task` function, we initialize the ADC before reading the sensor data, it is essential to initialize the ADC subsystem. This prepares the ADC to start taking readings. The ADC system can read from multiple channels, so we need to select the temperature sensor as the input channel to the ADC (the temperature sensor is on input 4). Once the temperature sensor is selected, we can read the analog value from the ADC. The adc_read() function returns a 12-bit value representing the analog voltage reading from the temperature sensor. The obtained analog value needs to be converted to a temperature reading. You can use the conversion formula provided in the RP2040 datasheet or SDK to convert the ADC reading to a temperature value in degrees Celsius as shown in `read_onboard_temperature()`.
```
float read_onboard_temperature() {
    
    /* 12-bit conversion, assume max value == ADC_VREF == 3.3 V */
    const float conversionFactor = 3.3f / (1 << 12);

    float adc = (float)adc_read() * conversionFactor;
    float tempC = 27.0f - (adc - 0.706f) / 0.001721f;

    return tempC;
}

void temp_task(__unused void *params) {
    float temperature = 0.0;
    adc_init();
    adc_set_temp_sensor_enabled(true);
    adc_select_input(4);                 // Temperature sensor is on input 4

    while(true) {
        vTaskDelay(1000);
        temperature = read_onboard_temperature();
        printf("Onboard temperature = %.02f C\n", temperature);
    }
}
```

Include the following in the `vLaunch` function.
```
    TaskHandle_t temptask;
    xTaskCreate(temp_task, "TestTempThread", configMINIMAL_STACK_SIZE, NULL, 8, &temptask);
```




**Create an Average Filtering Task**

Create the following variable to manage the message buffer. This is to send data from the temp_task to the avg_task.
```
static MessageBufferHandle_t xControlMessageBuffer;
```

Modify the temp_task to send a message each time it obtains new temperature data from the sensor.
```
void temp_task(__unused void *params) {
    float temperature = 0.0;
    adc_init();
    adc_set_temp_sensor_enabled(true);
    adc_select_input(4);

    while(true) {
        vTaskDelay(1000);
        temperature = read_onboard_temperature();
        printf("Onboard temperature = %.02f C\n", temperature);
        xMessageBufferSend( /* The message buffer to write to. */
            xControlMessageBuffer,
            /* The source of the data to send. */
            (void *) &temperature,
            /* The length of the data to send. */
            sizeof( temperature ),
            /* The block time; 0 = no block */
            0 );
    }
}
```

## **EXERCISE**


