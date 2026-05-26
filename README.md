# BMP280_Driver

This is a BMP280 Driver implentaion in C using arduino CLI and an arduino nano. The BMP280 is a temperature and pressure sensors that communicates with a microcontroller via I2C. Details on how to send data to and retrieve data from the BMP280 are found in the bmp280_datasheet.pdf. 

## Header Content

Users of the driver are provided functions to initialize the module, request and get data from the module and control the state of the module.

```c
void init_bmp280(Bmp280_config_t * cfg);
void request_measurements(void);
BMP280_S32_t bmp280_get_temperature(void);
BMP280_U32_t bmp280_get_pressure(void);
void bmp280_reset(void);
void bmp280_sleep(void);
void bmp280_normal(void);
```

