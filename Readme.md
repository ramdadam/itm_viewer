# ITM Viewer

ITM Viewer allows you to view the messages sent through the ARM ITM data channels.

*The plugin is at the start of the development and some things may not work as stable as planned, so please use it with patience :)*

![ITM Viewer Plugin Demo](./doc/demo.gif)

## Getting Started
This plugin parses the TCL RPC Server output of openocd. In order to activate it you must activate the tcl server.
### Prerequisites
* openocd
* CLion

Example config for STM32F746NG-Discovery:
```
source [find interface/stlink-v2-1.cfg]
transport select hla_swd
set WORKAREASIZE 0x40000

source [find target/stm32f7x.cfg]

# activate tcl server
tcl_port 6666
# for configuration please see §16.6.3 http://openocd.org/doc/html/Architecture-and-Core-Commands.html 
tpiu config internal - uart off 168000000
# activate ports (configurable)
itm port 24 on
itm port 25 on
itm port 26 on
itm port 27 on
```
And for the corresponding configuration in the CLion plugin config:
![ITM Viewer Plugin Configuration](./doc/itm_viewer_settings.png)

### Example 
A minimal C example to send message via ITM (for STM32F746):
```c
#define LOG_DEBUG_LEVEL 24
#define LOG_INFO_LEVEL 25
#define LOG_WARN_LEVEL 26
#define LOG_ERROR_LEVEL 27

#define LOG_DEBUG(msg) println(msg, LOG_DEBUG_LEVEL)
#define LOG_INFO(msg) println(msg, LOG_INFO_LEVEL)
#define LOG_WARN(msg) println(msg, LOG_WARN_LEVEL)
#define LOG_ERROR(msg) println(msg, LOG_ERROR_LEVEL)

void print(char _char, uint8_t level) {
    while (ITM->PORT[level].u32 == 0UL) {
        __NOP();
    }
    ITM->PORT[level].u8 = (uint8_t) _char;
}

void println(const char* msg, uint8_t level) {
    if(msg == NULL || level > 31) {
        return;
    }
    for(int i = 0; i<strlen(msg); i++) {
        print(msg[i], level);
    }
    print('\n', level);
}

void itm_test(void *pvParameters) {
    while(1) {
        LOG_INFO("Hello World!");
        vTaskDelay(1000); // or HAL_Delay
    }
}
```

### Installing
For now follow the installation path described [here](https://www.jetbrains.com/help/idea/managing-plugins.html#install_plugin_from_disk).
The zip is available under Github Releases. The plugin will be published over the JetBrains Plugins Repository.

## Authors

* **Mohamad Ramadan** - [ramdadam](https://github.com/ramdadam)

Also big thanks to my colleagues at cosee for helping me out :) 

## License

This project is licensed under the MIT License - see the [License](License) file for details
