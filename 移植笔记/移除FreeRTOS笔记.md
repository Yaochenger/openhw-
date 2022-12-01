### 移除FreeRTOS源代码笔记

#### APP下Makefile注释以下文件：

```makefile
#USER_SRCS += barrMemTest.c
#USER_SRCS += cam_tests.c
#USER_SRCS += camera_task.c
#USER_SRCS += camera.c
#USER_SRCS += cli_adv_timer_unit_tests.c
#USER_SRCS += cli_efpga_tests.c
#USER_SRCS += cli_efpgaio_tests.c
#USER_SRCS += cli_fcb_tests.c
#ssUSER_SRCS += cli_gpio_tests.c
#USER_SRCS += cli_i2c_tests.c
#USER_SRCS += cli_i2cs_tests.c
#USER_SRCS += cli_interrupt_tests.c
#USER_SRCS += cli.c
#USER_SRCS += gpio_map.c
#USER_SRCS += i2c_task.c
#USER_SRCS += sdio_tests.c
#USER_SRCS += programFPGA.c
```

#### 处理注释USER_SRCS += qspi_tests.c报错

- 注释system_init函数中关于qspi初始化部分

  ```c
  for (i = 0; i < N_QSPIM; i++)
  	{
  		setQspimPinMux(i);
  		udma_qspim_open(i, 2500000);
  		udma_qspim_control((uint8_t)i, (udma_qspim_control_type_t)kQSPImReset, (void *)0);
  
  		lFlashID = udma_flash_readid(i, 0);
  		if ((lFlashID == 0xFFFFFFFF) || (lFlashID == 0))
  		{
  			gQSPIFlashPresentFlg[i] = 0;
  		}
  		else
  		{
  			gQSPIFlashPresentFlg[i] = 1;
  			if ((lFlashID & 0xFF) == 0x20)
  			{
  				gMicronFlashDetectedFlg[i] = 1;
  				gQSPIIdNum = 0;
  			}
  			else
  				gMicronFlashDetectedFlg[i] = 0;
  		}
  		restoreQspimPinMux(i);
  	}
  	#endif
  ```

- 验证工程是否可以正常执行

#### 移除main.c中的vApplicationTickHook函数

```c
void vApplicationTickHook( void )
{
	/* The tests in the full demo expect some interaction with interrupts. */
	#if( mainCREATE_SIMPLE_BLINKY_DEMO_ONLY != 1 )
	{
		extern void vFullDemoTickHook( void );
		vFullDemoTickHook();
	}
	#endif
}
```

- 在tasks.c中找到xTaskIncrementTick函数

  ```c
  		#if ( configUSE_TICK_HOOK == 1 )
  		{
  			if( xPendedTicks == ( TickType_t ) 0 )
  			{
  				vApplicationTickHook();
  			}
  			else
  			{
  				mtCOVERAGE_TEST_MARKER();
  			}
  		}
  		#endif /* configUSE_TICK_HOOK */
  ```

- 关闭FreeRTOSConfig.h中configUSE_TICK_HOOK宏

  ```c#
  #define configUSE_TICK_HOOK				0
  ```

- 运行测试与当前存储分配

  >   text	   data	    bss	    dec	    hex	filename
  >   96097	  68152	  80792	 245041	  3bd31	cli_test

- 移除main.c中vApplicationMallocFailedHook函数

#### 移除xilinx文件夹

#### 移除gvsim文件夹

#### 注释metal_srcs.mk以下内容

```makefile
#n25q driver
#dir := $(FREERTOS_PROJ_ROOT)/app/N25Q_16Mb-1Gb_Device_Driver
#include $(dir)/makefile.mk
```

- 注释core-v-mcu.c以下内容

  ```c
  FLASH_DEVICE_OBJECT gFlashDeviceObject[N_QSPIM];
  ```

- 移除N25Q_16Mb-1Gb_Device_Driver文件夹

- 运行测试与当前存储分配

  >    text	   data	    bss	    dec	    hex	filename
  >   96073	  68152	  80408	 244633	  3bb99	cli_test

- 注释以下FreeRTOS内核相关内容

  ```makefile
  #SRCS += $(dir)/event_groups.c
  #SRCS += $(dir)/list.c
  #SRCS += $(dir)/queue.c
  #SRCS += $(dir)/stream_buffer.c
  #SRCS += $(dir)/tasks.c
  #SRCS += $(dir)/timers.c
  ```

- 部分错误如下

  >bin/../lib/gcc/riscv32-unknown-elf/7.3.0/../../../../riscv32-unknown-elf/bin/ld: /home/wangshun/plct_cli/cli_test/cli_test/drivers/source/udma_uart_driver.c:118: undefined reference to `xQueueGenericSend'

#### 在udma_i2cm_driver.c开始处定义unuse_freertos_in_i2cm，用于屏蔽该文件中的freertos中的代码

- 屏蔽freertos中信号量相关代码

```c++
    #ifndef unuse_freertos_in_i2cm
    SemaphoreHandle_t shSemaphoreHandle;			//创建信号量句柄
    shSemaphoreHandle = xSemaphoreCreateBinary();	//创建信号量
    configASSERT(shSemaphoreHandle);
    xSemaphoreGive(shSemaphoreHandle);				//释放一个信号量
    i2cm_semaphores_rx[i2cm_id] = shSemaphoreHandle;

    shSemaphoreHandle = xSemaphoreCreateBinary();
    configASSERT(shSemaphoreHandle);
    xSemaphoreGive(shSemaphoreHandle);
    i2cm_semaphores_tx[i2cm_id] = shSemaphoreHandle;
    #endif
```

#### 在udma_uart_driver.c开始处定义unuse_freertos_in_uart，用于屏蔽该文件中的freertos中的代码

```c
	#ifndef  unuse_freertos_in_uart

	/* Set semaphore */
	unuse_freertos_in_uart
	SemaphoreHandle_t shSemaphoreHandle;		
	shSemaphoreHandle = xSemaphoreCreateBinary();
	configASSERT(shSemaphoreHandle);
	xSemaphoreGive(shSemaphoreHandle);
	uart_semaphores_rx[uart_id] = shSemaphoreHandle;

	shSemaphoreHandle = xSemaphoreCreateBinary();
	configASSERT(shSemaphoreHandle);
	xSemaphoreGive(shSemaphoreHandle);
	uart_semaphores_tx[uart_id] = shSemaphoreHandle;
	#endif
```

### 移除外部中断中有关FreeRTOS信号量

- 定义SOC_EU_NB_FC_EVENTS个外部中断

  ```
  static SemaphoreHandle_t  fc_event_semaphores[SOC_EU_NB_FC_EVENTS];
  ```

- ***重点***

  >(1)注意中断嵌套的次数
  >
  >(2)系统启动前不开启中断

- FreeRTOS创建信号量，会执行关中断、

- 疑问：udma_uart_driver.c中包含以下代码可以正常运行，去除则不能执行

  ```C
  	SemaphoreHandle_t shSemaphoreHandle;		
  	shSemaphoreHandle = xSemaphoreCreateBinary(); //创建二值信号量
  ```

- 解决办法:在串口（外设）初始化前可别开中断，将以下语句注释

  ```c
  	irq_clint_enable();
  ```

  

