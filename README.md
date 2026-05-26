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

To set the BMP280 module up you first have to call **init_bmp280()** and pass a pointer of type **Bmp280_config_t**. You create a variable of the apporiate type and intialize the members with values based on your needs. The first member is **.normal_or_forced_mode** which decides the state in which the module will run. From power off mode (meaning the module is off) the BMP280 enters sleep mode once it is supplied with VCC. Then depending on the supplied configuration value it enters Normal or Sleep mode. **Normal Mode** measures periodically while **Forced Mode** measures whenever a measurement request is made and then returns to sleep mode. 

<br>

<img width="1160" height="523" alt="image" src="https://github.com/user-attachments/assets/0da968fb-5890-472f-9c91-5d5cb6869d64" />




















