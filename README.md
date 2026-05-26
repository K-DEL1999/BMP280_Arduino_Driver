# BMP280_Driver

This is a BMP280 Driver implentaion in C using arduino CLI and an arduino nano. The BMP280 is a temperature and pressure sensors that communicates with a microcontroller via I2C. Details on how to send data to and retrieve data from the BMP280 are found in the bmp280_datasheet.pdf. 

## Provided Functions, Unions/Structs and Types

Users of the driver are provided functions to initialize the module, request and get data from the module and control the state of the module. They are also given a union definition - Bmp280_config_t - to configure the sensor and also supplied aliases which are integral to communicating properly with the module and creating more readable code.

```c
// Aliases
typedef signed long BMP280_S32_t;
typedef unsigned long BMP280_U32_t;
typedef signed long long BMP280_S64_t;

// BMP280 configuration union
typedef union {
    struct {
        unsigned int normal_or_forced_mode : 1; // 0 : normal, 1 : forced 
        unsigned int temp_measurement : 3;
        unsigned int press_measurement : 3;
        unsigned int iir_filter : 3;
        unsigned int standby_time : 3;
    };
    unsigned short all_flags;
} Bmp280_config_t;

// Available functions
void init_bmp280(Bmp280_config_t * cfg);
void request_measurements(void);
BMP280_S32_t bmp280_get_temperature(void);
BMP280_U32_t bmp280_get_pressure(void);
void bmp280_reset(void);
void bmp280_sleep(void);
void bmp280_normal(void);
```

## Intializing BMP280 Module
### Initialization Function
To set the BMP280 module up you first have to call **init_bmp280()** and pass a pointer of type **Bmp280_config_t**. You create a variable of the apporiate type and intialize the members with values based on your needs. The first member is **.normal_or_forced_mode** which decides the state in which the module will run. From power off mode (meaning the module is off) the BMP280 enters sleep mode once it is supplied with VCC. Then depending on the supplied configuration value it enters Normal or Sleep mode. **Normal Mode** measures periodically while **Forced Mode** measures whenever a measurement request is made and then returns to sleep mode. Set **.normal_or_forced_mode** to 0 for **Normal Mode** and 1 for **Forced Mode**.
<br>
<br>
<img width="1160" height="523" alt="image" src="https://github.com/user-attachments/assets/0da968fb-5890-472f-9c91-5d5cb6869d64" />
<br>
<br>
### Quick Note On Measurements
You can select the resolution of each measurement by setting its **sampling rate** to a rate which is much higher than the **Nyquist Rate**. This tecnique is known as **oversampling** and its used by this module to set the resolution and reduce the noise in a reading. Naturally as you set this value higher it will take more time to perform the read. 
<br>
<br>

### Setting Measurement Resolution
After determining the mode of operation you can decide the resolution of the temperature measurements you want or you can choose to disable a measurement. You set the **.temp_measurement** to a value inbetween 0 and 7 and you do the same with **.press_measurement**. Setting these members to 0 will disable the senor from collecting the sample for said data while values between 1 and 7 will set the resolution.
<br>
<br>
<div align="center">
    <img width="45%" height="508" alt="temperature_oversampling_settings" src="https://github.com/user-attachments/assets/6ef3d7f6-9159-47e1-9530-2a9b181a140b" />
    <img width="45%" height="508" alt="pressure_oversampling_settings" src="https://github.com/user-attachments/assets/5ef75c8b-5363-4dae-b7e6-fad40e4369c7" />
</div>
<br>
<br>

### Setting IIR Filter Coefficient 

IIR Filters (Infinite Impulse Response Filters) are digital filters that use feedback which means that the output depends on both the current input, previous input and the past output. The BMP280 module has an IIR Filter due to the sensors readings being affected by short term changes caused by slamming doors, slamming windows, air etc. Basically any external disturbance may affect the module performing a measurement. The IIR Filter mitigates these disturbances and uses the following formula to calculate its filtered value. 
<br>
<br>
<img width="1160" height="157" alt="image" src="https://github.com/user-attachments/assets/cff1f4bd-5efb-4242-8f35-2ddec1082c5b" />
<br>
<br>
You can select your filter coefficient from the following table...
<br>
<br>
<img width="718" height="378" alt="image" src="https://github.com/user-attachments/assets/1119f055-388c-48bf-b17e-fd8f184fcfd7" />
<br>
<br>
You set the filter coefficient by assigning the **.iir_filter** member with a value between 0-7. Setting the coefficient equal to 0 will disable the filter.

### Setting Standby Time

This setting is only set when the module operates in **Normal Mode**



















