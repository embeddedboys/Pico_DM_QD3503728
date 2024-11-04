---
title: "micropython适配"
description: ""
summary: ""
date: 2024-03-23T01:02:33Z
lastmod: 2024-03-23T01:02:33Z
draft: false
weight: 1106
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

源码仓库：
[https://github.com/embeddedboys/lv_micropython](https://github.com/embeddedboys/lv_micropython)

本文将记录micropython的移植过程，由于我们是第一次接触micropython的移植，所以第一版的移植就是瞎搞，然后后面重构，进行了标准化，我们将开发记录保存在此处，供以后参考。

本文以lvgl v8版本为例进行说明，文档中提及的修改可能不够全面，用户可以再翻看下git提交记录。

## 如何编写一个micropython模块

其实整个移植过程主要是为运行的脚本提供对应的模块，我们提供了一个名为 **led** 的模块来帮助理解模块的编写，源码位于：[lv_micropython/ports/rp2/led.c](https://github.com/embeddedboys/lv_micropython/blob/release/v8/ports/rp2/led.c)

```c
#include "py/runtime.h"
#include "py/mphal.h"
#include "py/mperrno.h"

#include "hardware/gpio.h"

void hw_led_init(void)
{
    gpio_init(PICO_DEFAULT_LED_PIN);
    gpio_set_dir(PICO_DEFAULT_LED_PIN, GPIO_OUT);
}

void hw_led_on(void)
{
    gpio_put(PICO_DEFAULT_LED_PIN, 1);
}

void hw_led_off(void)
{
    gpio_put(PICO_DEFAULT_LED_PIN, 0);
}

STATIC mp_obj_t led_init_func(void)
{
    hw_led_init();
    return mp_const_none;
}
STATIC MP_DEFINE_CONST_FUN_OBJ_0(led_init_obj, led_init_func);


STATIC mp_obj_t led_on_func(void)
{
    hw_led_on();
    return mp_const_none;
}
STATIC MP_DEFINE_CONST_FUN_OBJ_0(led_on_obj, led_on_func);


STATIC mp_obj_t led_off_func(void)
{
    hw_led_off();
    return mp_const_none;
}
STATIC MP_DEFINE_CONST_FUN_OBJ_0(led_off_obj, led_off_func);

STATIC const mp_rom_map_elem_t led_module_globals_table[] = {
    { MP_OBJ_NEW_QSTR(MP_QSTR___name__),  MP_OBJ_NEW_QSTR(MP_QSTR_led) },
    { MP_ROM_QSTR(MP_QSTR_init), MP_ROM_PTR(&led_init_obj) },
    { MP_ROM_QSTR(MP_QSTR_on), MP_ROM_PTR(&led_on_obj) },
    { MP_ROM_QSTR(MP_QSTR_off), MP_ROM_PTR(&led_off_obj) },
};
STATIC MP_DEFINE_CONST_DICT(led_module_globals, led_module_globals_table);

const mp_obj_module_t led_module = {
    .base = {&mp_type_module},
    .globals = (mp_obj_dict_t *)&led_module_globals,
};

MP_REGISTER_MODULE(MP_QSTR_led, led_module);
```

它可以在py脚本中这样调用
```python
import led

led.init()

while True:
  led.on()
  time.sleep(1)
  led.off()
  time.sleep(1)
```

## 第一版移植

我们将所有的文件都放到了`ports/rp2`下，具体改动可以查看这一次提交[0f1ab6be40860df717dce0be66e5eb9368b4d1b4](https://github.com/embeddedboys/lv_micropython/commit/0f1ab6be40860df717dce0be66e5eb9368b4d1b4)

这样虽然方便，但是当我们有更多的板子需要适配时，就会变得混乱，而且对rp2的cmakelist改动太多，最主要的原因还是放错了地方，模块的编写应该在`lib/lv_bindings`下

## 标准化

我们对 micropython 和 lv_binding_micropython 都进行了修改，所以在此分开说明。 在查看源码时，先切换到 `release/v8` 分支。

`lv_binding_micropython` 作为git module位于 `micropython` 工程的 lib/lv_bindings

### micropython

[https://github.com/embeddedboys/lv_micropython](https://github.com/embeddedboys/lv_micropython)

##### 1. 修改 mpy工程的 MICROPY_GC_HEAP_SIZE，从预分配改为根据使用情况在ld文件中确定

好像是GC申请的太大，申请显示FB的时候内存爆了，时间过了太久我已经忘记当时的具体原因了。

mpy后面的新版本已经解决了这个问题。

```diff
------------------------------- ports/rp2/main.c -------------------------------
index da21e0b39..676199875 100644
@@ -53,16 +53,8 @@
 #include "lwip/apps/mdns.h"
 #endif
 
-#ifndef MICROPY_GC_HEAP_SIZE
-#if MICROPY_PY_LWIP
-#define MICROPY_GC_HEAP_SIZE 166 * 1024
-#else
-#define MICROPY_GC_HEAP_SIZE 192 * 1024
-#endif
-#endif
-
 extern uint8_t __StackTop, __StackBottom;
-static char gc_heap[MICROPY_GC_HEAP_SIZE];
+extern uint8_t __GcHeapStart, __GcHeapEnd;
 
 // Embed version info in the binary in machine readable form
 bi_decl(bi_program_version_string(MICROPY_GIT_TAG));
@@ -106,7 +98,7 @@ int main(int argc, char **argv) {
     // Initialise stack extents and GC heap.
     mp_stack_set_top(&__StackTop);
     mp_stack_set_limit(&__StackTop - &__StackBottom - 256);
-    gc_init(&gc_heap[0], &gc_heap[MP_ARRAY_SIZE(gc_heap)]);
+    gc_init(&__GcHeapStart, &__GcHeapEnd);
 
     #if MICROPY_PY_LWIP
     // lwIP doesn't allow to reinitialise itself by subsequent calls to this function
```

```diff
---------------------------- ports/rp2/memmap_mp.ld ----------------------------
index 0dc96743e..0757ff377 100644
@@ -236,6 +236,13 @@ SECTIONS
         __flash_binary_end = .;
     } > FLASH
 
+    /* stack limit is poorly named, but historically is maximum heap ptr */
+    __StackLimit = __bss_end__ + __micropy_c_heap_size__;
+
+    /* Define start and end of GC heap */
+    __GcHeapStart = __StackLimit; /* after the C heap (sbrk limit) */
+    __GcHeapEnd = ORIGIN(RAM) + LENGTH(RAM);
+
     /* stack limit is poorly named, but historically is maximum heap ptr */
     __StackLimit = ORIGIN(RAM) + LENGTH(RAM);
     __StackOneTop = ORIGIN(SCRATCH_X) + LENGTH(SCRATCH_X);
@@ -244,8 +251,9 @@ SECTIONS
     __StackBottom = __StackTop - SIZEOF(.stack_dummy);
     PROVIDE(__stack = __StackTop);
 
-    /* Check if data + heap + stack exceeds RAM limit */
-    ASSERT(__StackLimit >= __HeapLimit, "region RAM overflowed")
+    /* Check GC heap is at least 128 KB */
+    /* On a RP2040 using all SRAM this should always be the case. */
+    ASSERT((__GcHeapEnd - __GcHeapStart) > 128*1024, "GcHeap is too small")
 
     ASSERT( __binary_info_header_end - __logical_binary_start <= 256, "Binary info must be in first 256 bytes of the binary")
     /* todo assert on extra code */

```

##### 2. 关闭 ports/rp2/Makefile 中的cmake cache机制

因为我们要给多个板子适配，启用cache机制会导致之前的配置依然生效

```diff
------------------------------ ports/rp2/Makefile ------------------------------
index c2138a340..e3e6185c2 100644
@@ -9,6 +9,7 @@ BUILD ?= build-$(BOARD)
 $(VERBOSE)MAKESILENT = -s
 
 CMAKE_ARGS = -DMICROPY_BOARD=$(BOARD)

 # lv_binding_micropython 板级目录的名称，后面会用到
+CMAKE_ARGS += -DDISP_BOARD=$(DISP_BOARD)
 
 ifdef USER_C_MODULES
 CMAKE_ARGS += -DUSER_C_MODULES=${USER_C_MODULES}
@@ -22,8 +23,9 @@ ifeq ($(DEBUG),1)
 CMAKE_ARGS += -DCMAKE_BUILD_TYPE=Debug
 endif
 
+# [ -e $(BUILD)/CMakeCache.txt ] || cmake -S . -B $(BUILD) -DPICO_BUILD_DOCS=0 ${CMAKE_ARGS}
 all:
-	[ -e $(BUILD)/CMakeCache.txt ] || cmake -S . -B $(BUILD) -DPICO_BUILD_DOCS=0 ${CMAKE_ARGS}
+	cmake -S . -B $(BUILD) -DPICO_BUILD_DOCS=0 ${CMAKE_ARGS}
 	$(MAKE) $(MAKESILENT) -C $(BUILD)
 
 clean:
```

##### 3. 修改 ports/rp2/CMakeLists.txt ，引入我们需要的宏及文件

```diff
--------------------------- ports/rp2/CMakeLists.txt ---------------------------
index 6f209de87..a5a8efacb 100644
@@ -2,6 +2,11 @@ cmake_minimum_required(VERSION 3.12)
 
 set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
 
+if(NOT DISP_BOARD)
+    set(DISP_BOARD "PICO_DM_QD3503728")
+endif()
+set(MICROPY_PY_DISP_PORT_LVGL 1)
+
 # Set build type to reduce firmware size
 if(NOT CMAKE_BUILD_TYPE)
     set(CMAKE_BUILD_TYPE MinSizeRel)
@@ -126,8 +131,6 @@ set(MICROPY_SOURCE_PORT
     mbedtls/mbedtls_port.c
     led.c
     i80.c
-    ili9488.c
-    ft6236.c
 )
 
 set(MICROPY_SOURCE_QSTR
@@ -150,8 +153,6 @@ set(MICROPY_SOURCE_QSTR
     ${PROJECT_SOURCE_DIR}/rp2_flash.c
     ${PROJECT_SOURCE_DIR}/rp2_pio.c
     ${PROJECT_SOURCE_DIR}/led.c
-    ${PROJECT_SOURCE_DIR}/ili9488.c
-    ${PROJECT_SOURCE_DIR}/ft6236.c
 )
 
 set(PICO_SDK_COMPONENTS
@@ -292,6 +293,20 @@ if (MICROPY_PY_NETWORK_WIZNET5K)
     string(CONCAT GIT_SUBMODULES "${GIT_SUBMODULES} " lib/wiznet5k)
 endif()
 
+# Include our board porting files here
+if (MICROPY_PY_DISP_PORT_LVGL)
+    set(LVGL_BINDING_DIR ${LVGL_DIR}/..)
+    set(LVGL_RP2_DRV_DIR ${LVGL_BINDING_DIR}/driver/rp2)
+    set(BOARD_DIR ${LVGL_RP2_DRV_DIR}/${DISP_BOARD})
+    include(${BOARD_DIR}/board.cmake)
+
+    file(GLOB_RECURSE BOARD_FILES ${BOARD_DIR}/*.c)
+    list(APPEND MICROPY_SOURCE_EXTMOD
+        ${BOARD_FILES}
+    )
+endif()
+
+
 # Add qstr sources for extmod and usermod, in case they are modified by components above.
 list(APPEND MICROPY_SOURCE_QSTR
     ${MICROPY_SOURCE_EXTMOD}
```

##### 4. 提供一个build脚本，用来编译固件

因为编译mpy涉及的步骤比较多，所以写到脚本里方便些，在make的最后一步传入 DISP_BOARD 参数即可调整移植的显示拓展板，默认值是 PICO_DM_QD3503728
```bash
#!/bin/bash

git submodule update --init --recursive lib/lv_bindings

# Clean previous build first.
make -C ports/rp2 BOARD=PICO submodules clean
make -j -C mpy-cross clean
make -j -C ports/rp2 BOARD=PICO USER_C_MODULES=../../lib/lv_bindings/bindings.cmake clean

make -C ports/rp2 BOARD=PICO submodules
make -j -C mpy-cross
make -j -C ports/rp2 BOARD=PICO USER_C_MODULES=../../lib/lv_bindings/bindings.cmake
# make -j -C ports/rp2 BOARD=PICO DISP_BOARD=PICO_DM_FPC032MRA003 USER_C_MODULES=../../lib/lv_bindings/bindings.cmake
# make -j -C ports/rp2 BOARD=PICO DISP_BOARD=PICO_DM_GTM0375HI1T02 USER_C_MODULES=../../lib/lv_bindings/bindings.cmake

ln -sf ports/rp2/build-PICO/compile_commands.json .
```

### lv_binding_micropython

[https://github.com/embeddedboys/lv_binding_micropython](https://github.com/embeddedboys/lv_binding_micropython)

##### 1. 修改lv_conf.h配置文件，将 **LV_COLOR_DEPTH** hardcode 为 **16**，虽然可以在CMakeLists.txt中添加宏，但是目前我们没这样做。

```c
lib/lv_bindings/lv_conf.h
#define LV_COLOR_DEPTH 16
```

##### 2. 在 driver/rp2 下创建一个名为 pio 的目录，其中放置 pio 相关的驱动，例如 pio_i80，这些文件都是从 C 工程中直接拿过来的，跟micropython工程没有关系，所以不作说明。

```bash
lv_binding_micropython/driver/rp2/pio

pio_i80.c
pio_i80.pio.h
pio_spi_tx.c
pio_spi_tx.pio.h
```

##### 3. 在 driver/rp2 下创建板级目录 PICO_DM_QD3503728，目录文件如下

- [board.cmake](https://github.com/embeddedboys/lv_binding_micropython/blob/release/v8/driver/rp2/PICO_DM_QD3503728/board.cmake)
- [ft6236.c](https://github.com/embeddedboys/lv_binding_micropython/blob/release/v8/driver/rp2/PICO_DM_QD3503728/ft6236.c)
- [ili9488.c](https://github.com/embeddedboys/lv_binding_micropython/blob/release/v8/driver/rp2/PICO_DM_QD3503728/ili9488.c)

##### 2.1 board.cmake

这个文件里定义了 c 文件中需要的宏，例如引脚定义，是否启用PIO、DMA，分辨率等，需要单独说明的是

```c
list(APPEND MICROPY_SOURCE_EXTMOD
    ${CMAKE_CURRENT_LIST_DIR}/../pio/pio_i80.c
)
```

在这里添加需要的pio文件进编译系统。

##### 2.2 ft6236.c

这个文件实现了一个ft6236的模块，提供了两个函数，一个初始化函数，一个读取触摸函数。
```c
STATIC mp_obj_t ft6236_init_func(void)
{
    ft6236_driver_init();
    return mp_const_none;
}
STATIC MP_DEFINE_CONST_FUN_OBJ_0(ft6236_init_obj, ft6236_init_func);

STATIC bool mp_ft6236_ts_read(struct _lv_indev_drv_t *indev_drv, lv_indev_data_t *data)
{
    static int32_t last_x = 0;
    static int32_t last_y = 0;
    if (ft6236_is_pressed()) {
        data->point.x = last_x = ft6236_read_x();
        data->point.y = last_y = ft6236_read_y();
        data->state = LV_INDEV_STATE_PR;
    } else {
        data->point.x = last_x;
        data->point.y = last_y;
        data->state = LV_INDEV_STATE_REL;
    }
    return false;
}
DEFINE_PTR_OBJ(mp_ft6236_ts_read);
```

2.3 ili9488.c

这个文件实现了一个 ili9488 的驱动，提供初始化、销毁、刷新、fb申请函数

```c
lv_color_t *fb[2] = {NULL, NULL};           // framebuffer pointers

STATIC mp_obj_t ili9488_framebuffer(mp_obj_t n_obj)
{
	int n = mp_obj_get_int(n_obj) -1;
	if (n<0 || n>1){
		return mp_const_none;
	}
	if(fb[n]==NULL){
		fb[n] = m_malloc(sizeof(lv_color_t) * ILI9488_X_RES * 10);
	}
	return mp_obj_new_bytearray_by_ref(sizeof(lv_color_t) * ILI9488_X_RES * 10 , (void *)fb[n]);
}
STATIC MP_DEFINE_CONST_FUN_OBJ_1(ili9488_framebuffer_obj, ili9488_framebuffer);

STATIC mp_obj_t ili9488_init_func(void)
{
    if (fb[0] == NULL) {
        mp_obj_new_exception_msg(&mp_type_RuntimeError, MP_ERROR_TEXT("Failed allocating frame buffer"));
    }
    ili9488_driver_init();
    return mp_const_none;
}
STATIC MP_DEFINE_CONST_FUN_OBJ_0(ili9488_init_obj, ili9488_init_func);

STATIC mp_obj_t ili9488_deinit_func(void)
{
    if (g_priv.buf!=NULL) {
        m_free(g_priv.buf);
        g_priv.buf = NULL;
    }
    if(fb[0]!=NULL){
    	m_free(fb[0]);
    	fb[0]=NULL;
    }
    if(fb[1]!=NULL){
    	m_free(fb[1]);
    	fb[1]=NULL;
    }
    return mp_const_none;
}
STATIC MP_DEFINE_CONST_FUN_OBJ_0(ili9488_deinit_obj, ili9488_deinit_func);

STATIC void ili9488_flush(lv_disp_drv_t *disp_drv, const lv_area_t *area, lv_color_t *color_p)
{
    // pr_debug("%s: xs=%d, ys=%d, xe=%d, ye=%d, len=%d\n", __func__, area->x1, area->y1, area->x2, area->y2, lv_area_get_size(area));
    ili9488_video_sync(&g_priv, area->x1, area->y1, area->x2, area->y2, (void *)color_p, lv_area_get_size(area));
    lv_disp_flush_ready(disp_drv);
}
DEFINE_PTR_OBJ(ili9488_flush);
```

然后就可以像 ili9488_test.py 中那样引用这两个模块，注册到lvgl的回调函数里。