# 安信可ESP32系列模组对接腾讯物联开发平台指南
# 目录
0. [介绍](#Introduction)
1. [硬件准备](#hardwareprepare)  
2. [IDF环境搭建](#compileprepare)  
3. [构建&烧录&运行工程](#makefile)  
4. [调试工具](#debugtool)
5. [相关资源](#faq)

# <span id = "Introduction">0. 介绍</span>
`Ai-Thinker-Open-esp32-qcloud` 是由 [安信可科技](https://www.ai-thinker.com) 推出接入 [腾讯物联网开发平台 (IoT Explorer)](https://console.cloud.tencent.com/iotexplorer) 的开发套件 。该套件依据 `IoT Explorer` 通信协议而设计，相对于 [腾讯云物联 IoT C-SDK](https://github.com/tencentyun/qcloud-iot-sdk-embedded-c)，该套件将云平台配置、配网操作封装成接口，简化了整体流程，只需完成简单的调用，即可完成配网、连云操作。同时，该套件提供了丰富的调试工具、示例代码、量产工具供你快速完成产品开发。

| 支持模组 |    要求  |   购买链接  |
| -------- | -------- | ---- |
| 安信可 ESP32 系列模组 | 4MB FLash | https://item.taobao.com/item.htm?id=548905088891 |
| 安信可 ESP32S2 系列模组 | 4MB FLash | https://item.taobao.com/item.htm?id=621845112434 |
| 安信可 ESP32-C3 系列模组 | 4MB FLash | https://item.taobao.com/item.htm?id=646269575479 |

- **配网方式**
- [x] softap
    - [x] airkiss
    - [x] esp-touch v1
    - [x] ble
    - [x] provisioning softap
    - [ ] esp-touch v2
    
- **认证方式**

    - [x] 密钥认证
    - [ ] 证书认证
    - [ ] 动态注册

- **业务功能**

    - [x] 状态上报与下发
    - [x] OTA升级
    - [x] 事件上报
    - [ ] 网关

- **调试功能**

    - [x] 日志上报云平台
    - [x] 日志本地存储
    - [x] 串口调试

- **生产工具**

    - [x] 单一/批量 bin 生成
    - [ ] 加密

# <span id = "hardwareprepare">1.**模组**开发板准备</span>
- ESP32 开发板：https://item.taobao.com/item.htm?id=548905088891
- ESP32-S2 开发板：https://item.taobao.com/item.htm?id=621845112434
- ESP32-C3 开发板（内置4MB版本）：https://item.taobao.com/item.htm?id=646269575479

# <span id = "compileprepare">2. IDF 环境搭建</span>

- 可以参考 [ESP-IDF编程指南-快速入门](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32/get-started/index.html#get-started-setup-toolchain) 快速完成工具链与环境的搭建
- 使用 `ESP32`、`ESP32-S2` 构建工程之前需将 `ESP-IDF` 切换到 `release/v4.2 分支`，使用 `ESP32-C3` 则需要切换到 `master`。

    > `ESP32-C3` 暂无 `release` 分支，目前可用的commitID：8e3e65a47b7d9b5dc4f52eb56660a748fda1884e

    ```shell
    cd  $IDF_PATH 

    # esp32 or esp32s2
    git checkout release/v4.2 
    # esp32c3
    git checkout 8e3e65a47b7d9b5dc4f52eb56660a748fda1884e 

    git submodule update --init --recursive
    ```

# <span id = "makefile">3. 构建&烧录&运行工程</span> 

0. **项目文件**

     你可以直接在 `main` 目录进行开发。

     ```shell
     #Delete unnecessary files
     cd main
     idf.py fullclean
     ```


1. **配置芯片**

    不同的芯片会加载不同的 `sdkconfig` 默认文件，位于 [config](./config/sdkconfig_defaults) 目录下。默认的 `sdkconfig` 文件会加载 `4MB` 的 `分区表 (partitions)` 。如果芯片 `flash` <b>小于</b> `4MB` ，需要将其更改为 `2MB` 的分区表。下述几个命令将帮助你快速配置芯片。

    - **查看当前芯片信息**

        ```shell
        $IDF_PATH/components/esptool_py/esptool/esptool.py flash_id
        ```

    - **设置编译时目标芯片**

        ```shell
        #choose esp32
        idf.py set-target esp32 

        #choose esp32s2
        idf.py set-target esp32s2

        #choose esp32c3
        idf.py set-target esp32c3
        ```

    - **更改 `Partition`**

        1. **进入 `menuconfig` 配置界面**

            ```shell
            idf.py menuconfig
            ```

        2. **选择 `Partition Table`**

            ```shell
            Partition Table (Custom partition table CSV)  --->
            (${QCLOUD_PATH}/config/partition_table/partitions_4MB.csv) Custom partition CSV file
            (0xc000) Offset of partition table
            [*] Generate an MD5 checksum for the partition table
            ```

            - `Custom partition CSV file` 中即可编辑 `CSV` 文件。


2. **烧录认证信息[可选]**

   当使用云平台时，你需要在 [IoT Explorer](https://console.cloud.tencent.com/iotexplorer) 获取认证信息，认证信息通常为 `产品 ID (PRODUCT_ID)` 、 `设备名称 (DEVICE_NAME) `、 `设备密钥 (DEVICE_SECRET) `，可参考 [智能灯文档](./examples/led_light/README.md) 中 `烧录认证信息` 或 [IoT Explorer 官方文档](https://cloud.tencent.com/document/product/1081/34739) 进行平台参数设置、获取。获取到认证信息时，可以选择下述任意一种方式烧录。

   - **通过 `menuconfig` 配置**

        1. **进入 `menuconfig` 配置界面**

            ```shell
            idf.py menuconfig
            ```

        2. **选择 `ESP QCloud Config` 选项**

            ```shell
            [ ] ESP QCloud Mass Manufacture
            (PRODUCT_ID) Product ID
            (DEVICE_NAME) Device Name
            (DEVICE_SECRET) Device Secret
            ESP QCloud OTA Config  --->
            ESP QCloud utils  --->
            ESP QCloud Log Config  --->
            UART for console input (UART0)  --->
            ```

            - <b>不开启</b> `ESP QCloud Mass Manufacture` 选项。

        3. **填入你的信息**

            填写 `产品 ID (PRODUCT_ID) `、 `设备名称 (DEVICE_NAME) `、 `设备密钥 (DEVICE_SECRET) `。
        
 - **通过量产工具配置**
   
        请参考量产工具目录下的 [文档](./config/mass_mfg/README.md) ，另外需要 <b>开启</b> `ESP QCloud Mass Manufacture` 选项。


3. **构建项目**
   
    当完成配置，下述命令将帮助快速构建。

    - **构建工程**

    ```shell
    idf.py build
    ```

    - **清理工程**

    ```shell
    idf.py fullclean
    ```

    - **性能选择**

        通过在编译时传递 `PERF` 定义来决定开启的性能。未指定参数时将使用 `defaults` 配置，关于性能的说明，请参考 [性能说明文档](./docs/ESP32PerformanceOptions.md)。

    ```shell
    #high performance
    idf.py menuconfig -D PERF="hight_perf"

    #low performance
    idf.py menuconfig -D PERF="low_perf"
    ```

3. **运行项目**

    - **下载、运行工程**

    ```shell
    idf.py flash monitor
    ```
    
    - **擦除 Flash**

    ```shell
    idf.py erase_flash
    ```

# <span id = "debugtool">4. 调试工具</span> 

- **串口调试**

    串口调试允许通过串口查看当前设备信息，开启方法如下：

    1. **进入 `menuconfig` 配置界面**

        ```shell
        idf.py menuconfig
        ```

    2. **选择 `ESP QCloud Example Configuration`**

        ```shell
        Light development board selection   --->
        Light provisioning network selection  --->
        (5) More than this number of continuous uninterrupted restarts triggers a reset of the device
        [*] The device will be in debug mode
        ```

        - <b>开启</b> `The device will be in debug mode` 选项。

    3. **查看调试信息**

        - 下述为 `led_light` 中的打印信息。

        ```
        I (31091) esp_qcloud_utils: System information sta_mac: 24:6f:28:80:3f:14, channel: [10/2], rssi: -40, free_heap: 181128, minimum_heap: 161540
        ```

        - 通过 `shell` 输入指令可查看相关信息，例如执行 `heap` 指令，将打印系统运行状态，更多指令可通过 `help` 查询。

        ```shell
        esp32> heap
        I (15360) esp_qcloud_mem: (714) <esp_qcloud_log: 159> ptr: 0x3ffbd1c4, size: 16
        I (15361) esp_qcloud_mem: (716) <esp_qcloud_log_flash: 157> ptr: 0x3ffbdecc, size: 24
        I (15372) esp_qcloud_mem: (1093) <esp_qcloud_device: 85> ptr: 0x3ffbe8e4, size: 20
        I (15383) esp_qcloud_mem: (1093) <esp_qcloud_device: 218> ptr: 0x3ffc0d3c, size: 20
        I (15384) esp_qcloud_mem: (1093) <esp_qcloud_device: 218> ptr: 0x3ffc0d68, size: 20
        I (15395) esp_qcloud_mem: (1094) <esp_qcloud_device: 218> ptr: 0x3ffc0d80, size: 20
        I (15406) esp_qcloud_mem: (1094) <esp_qcloud_device: 218> ptr: 0x3ffc0da8, size: 20
        I (15417) esp_qcloud_mem: (3608) <esp_qcloud_iothub: 213> ptr: 0x3ffca3d4, size: 128
        I (15418) esp_qcloud_mem: Memory record, num: 8, size: 268
        I (15429) esp_qcloud_mem: Free heap, current: 181096, minimum: 161540
        I (15441) esp_qcloud_mem: ---------------- The State Of Tasks ----------------
        I (15441) esp_qcloud_mem: - HWM   : usage high water mark (Byte)
        I (15452) esp_qcloud_mem: - Status: blocked ('B'), ready ('R'), deleted ('D') or suspended ('S')

        I (15463) esp_qcloud_mem: TaskName              Status  Prio    HWM     TaskNum CoreID  RunTimeCounter  Percentage
        I (15464) esp_qcloud_mem: console_handle        R       1       1952    8       -1      26908           <1
        I (15475) esp_qcloud_mem: IDLE0                 R       0       1008    4       0       13232798        89
        I (15486) esp_qcloud_mem: mqtt_task             B       5       3472    12      -1      978102          6
        I (15487) esp_qcloud_mem: Tmr Svc               B       1       792     5       0       1851            <1
        I (15498) esp_qcloud_mem: qcloud_log_send       B       4       2580    7       0       3451            <1
        I (15509) esp_qcloud_mem: tiT                   B       18      2112    9       -1      26425           <1
        I (15510) esp_qcloud_mem: esp_timer             B       22      3348    1       0       38541           <1
        I (15521) esp_qcloud_mem: wifi                  B       23      3944    11      0       234073          1
        I (15532) esp_qcloud_mem: sys_evt               B       20      596     10      0       2884            <1
        ```

- **保存调试日志**

    该功能可将日志保存在 `Flash`。你需要构造 `esp_qcloud_log_config_t` 结构体，通过 `esp_qcloud_log_init()` 设置日志等级。

    ```c
    esp_qcloud_log_config_t log_config = {
    .log_level_flash = ESP_LOG_INFO,
    };
    ```
