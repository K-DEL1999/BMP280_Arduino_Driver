# BMP280_Driver

This is a BMP280 Driver implentaion in C using arduino CLI and an arduino nano. The BMP280 is a temperature and pressure sensors that communicates with a microcontroller via I2C. Details on how to send data to and retrieve data from the BMP280 are found in the bmp280_datasheet.pdf. 

## Pin Set Up
<br>
<br>
<div align="center">
<img width="756" height="164" alt="image" src="https://github.com/user-attachments/assets/a7aa70bb-41ba-4f72-966f-1d98fb26f9ab" />
</div>
<br>
<br>

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
<div align="center">
<img width="718" height="378" alt="image" src="https://github.com/user-attachments/assets/1119f055-388c-48bf-b17e-fd8f184fcfd7" />
</div>
<br>
<br>

You set the filter coefficient by assigning the **.iir_filter** member with a value between 0-7. Setting the coefficient equal to 0 will disable the filter. A value of 1 will correspond to a filter coefficient of 2, 2 to 4, 3 to 8, 4,5,6,7 to 16.

### Setting Standby Time

This setting is only set when the module operates in **Normal Mode**. It determines the time inbetween each measurement made. Values can still be read from the modules memory even while a measurement is taking place. To set the time inbetween each measurement you set **.standby_time** equal to a value inbetween 0-7. The image below maps a standby time to each value within said range.
<br>
<br>
<div align="center">
<img width="508" height="541" alt="standby_settings" src="https://github.com/user-attachments/assets/ea500307-36e9-47fc-8173-72f59b70d94f" />
</div>
<br>
<br>

### Example Intialization
```c
.
.
.
static Bmp280_config_t cfg; // Made static to ensure all undefined members are 0
int i = 0;

void setup() {
    cfg = { 
        {
        .normal_or_forced_mode = 0x00,
        .temp_measurement = 0x01,
        .press_measurement = 0x01, 
        .iir_filter = 0x01,
        .standby_time = 0x01
        }
    };
    
    init_bmp280(&cfg);

}
.
.
.
```
<br>
When deciding what values to use when intializing the Bmp280 please defer to the following charts supplied by the manufacturers. This chart shows the expected performance for specific configurations. Other configurations have not been tested as thoroughly as the ones below.
<br>
<br>
<img width="1274" height="733" alt="recommended_settings" src="https://github.com/user-attachments/assets/ec91d026-9b16-4879-8e8c-2a6b5f7929c8" />

## Using BMP280 Module

### Forced Mode 

When using module in **Force Mode** you must request a measurement first and then get the data.
```c
.
.
.
void loop() {
    request_measurements();
    
    char buffer[64];
    sprintf(buffer, "temp: %ld C \tpress: %lu hPa\n", bmp280_get_temperature(), bmp280_get_pressure());
    Serial.print(buffer);
    
    delay(1000);
}
```

### Normal Mode

When using module in **Normal Mode** you can get data whenever. There is no need to call **request_measurements()**.

```c
.
.
.
void loop() {    
    char buffer[64];
    sprintf(buffer, "temp: %ld C \tpress: %lu hPa\n", bmp280_get_temperature(), bmp280_get_pressure());
    Serial.print(buffer);

    // Ensure the make the time greater than the standby time to ensure you read a new measurement each time
    delay(TIME_GREATER_THAN_STANBY_TIME);
}
```

## Source Details

### Helper Functions

```c
static void bmp280_burst_read_reg(unsigned char * cmds, unsigned char * data, uint8_t data_size);
static void bmp280_burst_write_reg(unsigned char * cmds, uint8_t num_of_cmds, unsigned char * data);

// Load calibration values into compensation_words array for later use in calibrating adc values
static void bmp280_get_calibration_values(void);

// Assembles bytes --- data sheet specifices that 
static BMP280_S32_t assemble_measurement(unsigned char MSB, unsigned BYTE1, unsigned char LSB);

// Compensation temperature and pressure functions provided by data sheet
static BMP280_S32_t bmp280_compensate_T_int32(BMP280_S32_t adc_T);
static BMP280_U32_t bmp280_compensate_P_int64(BMP280_S32_t adc_P);

// I2C Functions
static void bmp_i2c_init(void);
static void bmp280_i2c_transmit(unsigned char * cmds, uint8_t num_of_cmds, unsigned char * data);
static void bmp280_i2c_receive(unsigned char * cmds, unsigned char * data, uint8_t data_size);
```
Implementations can be found in BMP280_I2C_Driver_Source.cpp!
<br>

### How Communication Was Established
The BMP280 uses I2C to communicate with microcontrollers. To write you send the **SLAVE_ADDRESS** in write mode then follow it with a **control byte** that holds the address of the register you wish to write to and a **data byte** that holds that data that will be written. After the address you can send continous pairs of **control and data bytes**.
<br>
<br>
<img width="1235" height="387" alt="image" src="https://github.com/user-attachments/assets/899bbdaf-aafa-4b27-b2d3-e95b68550e58" />
<br>
<br>
To read from the device you send the **SLAVE_ADDRESS** in write mode followed by the desired register to read from. Then either a stop or repeated start condition is sent with the **SLAVE_ADDRESS** in read mode. Afterwards the slave starts sending out data from auto-incremented register addresses until a NOACK and a stop condition occurs.
<br>
<br>
<img width="1235" height="387" alt="image" src="https://github.com/user-attachments/assets/7f7e0207-10a8-45f1-a77b-1b39c49322bf" />
<br>
<br>
The module 7 bit address is 0b111011X where X is determined by whether the SD0 pin is GND or VDD. 
<br>
**If SD0 is set to VDD then address is 0b1110111**
<br>
**If SD0 is set to GND then address is 0b1110110**
<br>
### Example Read From Module Captured With Logic Analyzer
<br>
<img width="2151" height="525" alt="typical_measurement" src="https://github.com/user-attachments/assets/0a8fdc1c-b6bd-495b-884f-6601af1291c0" />
<br>
<br>
Here is an example of reading from register F3 which is the status register and then performing a burst read from the temperature and pressure register. The read starts from address F7 and auto increments as each piece of data is sent. The read concludes with a NOACK and stop condition.

### Example Initialization of Module Captured with Logic Analyzer

<img width="2774" height="634" alt="startup_sequence_verification" src="https://github.com/user-attachments/assets/1eb1f398-2729-4ffb-94f5-d70b591e6bb8" />
<br>
<br>
First address F4 (CTRL_MEAS) and F5 (CONFIG) where written to with the appropriate values - these configure the measurement parameters and the overall module configuration. Afterwards address F3 (STATUS) is polled until a flag is set indicating that the callibration values are ready for read - once the STATUS changes from OD to OC. Finally All calibration values are read starting from address 0x88 until 0x9F. Only the first address is needed because the auto incrementing nature of the registers will handle the request. **That is why the enum only contains CALIB00**. A burst read just has to be initiated from register 0x88 and the module will send data until it reaches register 0xFA. There are 24 calibration values. The datasheet provides methods on how to prepare the calibration values and functions for using the calibration values to adjust the incoming data from the module - this information can be found on page 21 and 22 of the datasheet. An implementation of how to retrieve and assemble these values is shown below along with the calibration functions.
<br>
<br>

```c
.
.
.
//Compensation Values
static unsigned long DIG_T1;
static signed long DIG_T2;
static signed long DIG_T3;
static unsigned long DIG_P1;
static signed long  DIG_P2;
static signed long  DIG_P3;
static signed long  DIG_P4;
static signed long  DIG_P5;
static signed long  DIG_P6;
static signed long  DIG_P7;
static signed long  DIG_P8;
static signed long  DIG_P9;
.
.
.

static void bmp280_get_calibration_values(void){
    // gets calibration values from bmp280 
    // uses bmp280 built in auto increment for reading all values continously starting from CALIB00 
    unsigned char calibration_values[24];
    cmds[0] = CALIB00;
    bmp280_burst_read_reg(cmds, calibration_values, 24);

    DIG_T1 = (calibration_values[1] << 8) | calibration_values[0];
    DIG_T2 = (calibration_values[3] << 8) | calibration_values[2];
    DIG_T3 = (calibration_values[5] << 8) | calibration_values[4];
    DIG_P1 = ((calibration_values[7] << 8) | calibration_values[6]);
    DIG_P2 = (calibration_values[9] << 8) | calibration_values[8];
    DIG_P3 = (calibration_values[11] << 8) | calibration_values[10];
    DIG_P4 = (calibration_values[13] << 8) | calibration_values[12];
    DIG_P5 = (calibration_values[15] << 8) | calibration_values[14];
    DIG_P6 = (calibration_values[17] << 8) | calibration_values[16];
    DIG_P7 = (calibration_values[19] << 8) | calibration_values[18];
    DIG_P8 = (calibration_values[21] << 8) | calibration_values[20];
    DIG_P9 = (calibration_values[23] << 8) | calibration_values[22];
}

// The datasheet expects the conversion from unsigned 20 bit int to signed long. The 8 bytes are assembled into a unsigned 24 
//  bit value which is then bitmasked to 20, since only 20 bits contain data. The value is then explicitly converted into a 
//  signed 32 long which is the type expected by the compensation functions.
static BMP280_S32_t assemble_measurement(unsigned char MSB, unsigned BYTE1, unsigned char LSB){ // big endian -- MSB to LSB
    return ((BMP280_S32_t)MSB << 12) | ((BMP280_S32_t)BYTE1 << 4) | ((BMP280_S32_t)LSB >> 4);
    
    // BUG !!!! 
    //return (MSB << 12 | BYTE1 << 4 | LSB >> 4);
}

// ======================================================== //
// ===== Compensation functions provided by datasheet ===== //
// ======================================================== //
static BMP280_S32_t t_fine;
static BMP280_S32_t bmp280_compensate_T_int32(BMP280_S32_t adc_T){
        BMP280_S32_t var1, var2, T;
    var1 = ((((adc_T>>3) - ((BMP280_S32_t)DIG_T1<<1))) * ((BMP280_S32_t)DIG_T2)) >> 11;
    var2 = (((((adc_T>>4) - ((BMP280_S32_t)DIG_T1)) * ((adc_T>>4) - ((BMP280_S32_t)DIG_T1)))>> 12) *((BMP280_S32_t)DIG_T3)) >> 14;
    t_fine = var1 + var2;
    T = (t_fine * 5 + 128) >> 8;
    return T;
}

static BMP280_U32_t bmp280_compensate_P_int64(BMP280_S32_t adc_P){
    BMP280_S64_t var1, var2, p;
    var1 = ((BMP280_S64_t)t_fine) - 128000;
    var2 = var1 * var1 * (BMP280_S64_t)DIG_P6;
    var2 = var2 + ((var1*(BMP280_S64_t)DIG_P5)<<17);
    var2 = var2 + (((BMP280_S64_t)DIG_P4)<<35);
    var1 = ((var1 * var1 * (BMP280_S64_t)DIG_P3)>>8) + ((var1 * (BMP280_S64_t)DIG_P2)<<12);
    var1 = (((((BMP280_S64_t)1)<<47)+var1))*((BMP280_S64_t)DIG_P1)>>33;
    
    if (var1 == 0){
        return 0; // avoid exception caused by division by zero
    }

    p = 1048576-adc_P;
    p = (((p<<31)-var2)*3125)/var1;
    var1 = (((BMP280_S64_t)DIG_P9) * (p>>13) * (p>>13)) >> 25;
    var2 = (((BMP280_S64_t)DIG_P8) * p) >> 19;
    p = ((p + var1 + var2) >> 8) + (((BMP280_S64_t)DIG_P7)<<4);

    return (BMP280_U32_t)p;
}
// ======================================================== //
// ======================================================== //

.
.
.

```

<br>

### Memory Map
This is the memory map provided by the datasheet. Each register address was saved in an **enum** at the top of the source file. What each register does and its corresponding address and size can be found in the datasheet in this directory - bmp280_datasheet.pdf - on pages 24, 25, 26 and 27.
```c
.
.
.
//BMP280_Registers
enum {
    TEMP_XLSB = 0xFC,
    TEMP_LSB = 0xFB,
    TEMP_MSB = 0xFA,
    PRESS_XLSB = 0xF9,
    PRESS_LSB = 0xF8,
    PRESS_MSB = 0xF7,
    CONFIG = 0xF5,
    CTRL_MEAS = 0xF4,
    STATUS = 0xF3,
    RESET = 0xE0,
    ID = 0xD0,
    CALIB00 = 0x88
};
.
.
.
```
<br>
<br>
<img width="1274" height="608" alt="memory_map" src="https://github.com/user-attachments/assets/04e74236-d98c-4790-9a4b-cd8652135e54" />
<br>
<br>

## FINAL OUTPUTS
<br>
<img width="532" height="634" alt="example_outputs" src="https://github.com/user-attachments/assets/4be1b9d1-9997-4b0b-b44a-1063ec50873b" />
<br>
<br>
Here you can see the calibration values for this particular module and a couple of measurements taken!














