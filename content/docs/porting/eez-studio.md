---
title: "EEZ-Studio"
description: ""
summary: ""
date: 2024-05-04T22:52:10+08:00
lastmod: 2024-05-04T22:52:10+08:00
draft: false
weight: 1108
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

## 移植流程

点击软件左上角工具栏中的扳手按钮，所需的ui源文件将导出到`${工程根目录}/ui/src/ui`下

打开任一工程，以 `pico_dm_qd3503728_noos` 为例

1. 将EEZ-Studio导出的ui文件夹链接到本工程中，这样比较方便开发，以后在Studio中导出即可
```bash
# 假设位于pico_dm_qd3503728_noos工程根目录, ui_project是你保存eez-studio工程所在文件夹
ln -sf ../ui_project/src/ui ./ui
```

2. 修改`CMakeLists.txt`，引入UI相关文件
```cmake
file(GLOB_RECURSE COMMON_SOURCES
    main.c
    ili9488.c
    ft6236.c
    porting/lv_port_disp_template.c
    porting/lv_port_indev_template.c
    i2c_tools.c
    backlight.c
)

# 在此处定义UI相关文件
file(GLOB_RECURSE UI_SOURCES
    ui/*.c
    ui/*.cpp
)

# rest of your project
add_executable(${PROJECT_NAME} ${COMMON_SOURCES} ${UI_SOURCES}) # 在此处引入UI相关文件
target_link_libraries(${PROJECT_NAME}
        pico_bootsel_via_double_reset
        pico_stdlib hardware_pwm
        hardware_i2c pio_i80
        # factory_test
        lvgl lvgl::demos lvgl::examples
)
target_include_directories(${PROJECT_NAME} PUBLIC .)
```

3. 如果你此前进行过build，则需在build文件夹内执行
```
cmake .. -G Ninja
```

4. 修改 `main.c` ，修改替换 eez-studio 所需要的初始化操作

4.1 文件顶部添加所需头文件
```c
#include "ui/ui.h"
```

4.2 注释所有的 lv_demo，并在此处调用`ui_init`
```c
    // lv_demo_widgets();
    // lv_demo_stress();
    // lv_demo_music();

    /* measure weighted fps and opa speed */
    // Before : Avg.146 256 114 186
    // After  : Avg.177 311 125 216
    // lv_demo_benchmark();
```

4.3 在loop中调用`ui_tick`

```c
    for (;;) {
        // tight_loop_contents();
        // sleep_ms(200);
        lv_timer_handler_run_in_period(1);
        ui_tick();
    }
```

5. 编译工程，并烧录到设备，按住核心板BOOTSEL，并接入USB线连接至电脑
```bash
ninja && cp pico_dm_qd3503728.uf2 /media/${USER}/RPI-RP2
```
eez-studio 设计的UI已成功运行在设备上

## 示例工程

### No-os

[https://github.com/embeddedboys/pico_dm_qd3503728_noos_eez_studio_demo](https://github.com/embeddedboys/pico_dm_qd3503728_noos_eez_studio_demo)

### FreeRTOS

开发中。。。
