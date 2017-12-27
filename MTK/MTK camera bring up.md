# <center><font size=8>MTK camera bring up</font></center>
-----------------------------------------------------------------------------------
</br>

## 一、DWS配置：
### 1.根据硬件连接，对照HW interface部分，配置cam各pin：
(1) Serial(DATA/SERIAL data CLOCK) </br>
(2) Mipi(MIPI data和MIPI clock) </br>
(3) MCLK、PCLK </br>
(4) PDN、RST、LDO_ENABLE pin </br>

### 2.example
<img src="./dws.png" width = "" height = "200" alt="camera_dws" align=center />

</br>

## 二、config配置文件：
### 1.Configure camera sensor hal driver in ProjectConfig.mk
**file path**：alps/device/mediatek/$project/ProjectConfig.mk

**example**：

	CUSTOM_HAL_IMGSENSOR = gc0310_yuv imx179_mipi_raw ov5670_mipi_raw hi556_mipi_raw hi846_mipi_raw
	CUSTOM_HAL_MAIN_IMGSENSOR = ov5670_mipi_raw hi846_mipi_raw
	CUSTOM_HAL_SUB_IMGSENSOR = gc0310_yuv hi556_mipi_raw
	CUSTOM_KERNEL_IMGSENSOR = hi556_mipi_raw hi846_mipi_raw
	CUSTOM_KERNEL_MAIN_IMGSENSOR = ov5670_mipi_raw hi846_mipi_raw
	CUSTOM_KERNEL_SUB_IMGSENSOR = hi556_mipi_raw


### 2.Add camera sensor kernel driver project config (&lt;project>_defconfig)
**file path**： alps/<kernel>/arch/arm/configs/&lt;project>_defconfig

**example**：
	CONFIG_CUSTOM_KERNEL_IMGSENSOR="hi556_mipi_raw hi846_mipi_raw"

**key note**：<font color=#ff0000>此处的定义一定要和ProjectConfig.mk文件中的定义一样。</font>

</br>

## 三、Kernel层添加驱动文件、上电时序与头文件相关定义：
### 1.添加驱动文件：
#### （1）Add camera sensor kernel driver code to the corresponding path（kernel层路径）</br>
**file path**：</br>
【By project】：alps/kernel-3.18/drivers/misc/mediatek/mach/mt6580/&lt;project>/imgsensor/ </br>
【By platform】：alps/kernel-3.18/drivers/misc/mediatek/imgsensor/src/&lt;platform>/

#### （2）添加Makefile。

###2.添加上下电时序：
**file path**：alps/kernel-3.18/drivers/misc/mediatek/imgsensor/src/mt6580/camera_hw/kd_camera_hw.c

**example**：
在int kdCISModulePowerOn(CAMERA_DUAL_CAMERA_SENSOR_ENUM SensorIdx, char          	*currSensorName, BOOL On, char *mode_name) function中添加：

	if (currSensorName && (0 == strcmp(SENSOR_DRVNAME_HI556_MIPI_RAW, currSensorName))) {
		//mtkcam_gpio_set(pinSetIdx, CAMLDO, 1);
		/* First Power Pin low and Reset Pin Low */

		if (GPIO_CAMERA_INVALID != pinSet[pinSetIdx][IDX_PS_CMPDN])
			mtkcam_gpio_set(pinSetIdx, CAMPDN, pinSet[pinSetIdx][IDX_PS_CMPDN + IDX_PS_OFF]);

		if (GPIO_CAMERA_INVALID != pinSet[pinSetIdx][IDX_PS_CMRST])
			mtkcam_gpio_set(pinSetIdx, CAMRST, pinSet[pinSetIdx][IDX_PS_CMRST + IDX_PS_OFF]);

		mdelay(10);

		/* VCAM_IO */
		if (TRUE != _hwPowerOn(VCAMIO, VOL_1800)) {
			PK_DBG("[CAMERA SENSOR] Fail to enable IO power (VCAM_IO),power id = %d\n", VCAMIO);
			goto _kdCISModulePowerOn_exit_;
		}

		if (GPIO_CAMERA_INVALID != pinSet[pinSetIdx][IDX_PS_CMRST]) {
			mtkcam_gpio_set(pinSetIdx, CAMRST, pinSet[pinSetIdx][IDX_PS_CMRST + IDX_PS_ON]);
		}
        mdelay(1); //t0

		/* VCAM_A */
		if (TRUE != _hwPowerOn(VCAMA, VOL_2800)) {
			PK_DBG("[CAMERA SENSOR] Fail to enable analog power (VCAM_A),power id = %d\n", VCAMA);
			goto _kdCISModulePowerOn_exit_;
		}

        udelay(1); //t1

        /* VCAM_D */
		if (TRUE != _hwPowerOn(VCAMD, VOL_1200)) {
			PK_DBG("[CAMERA SENSOR] Fail to enable digital power (VCAM_D),power id = %d\n", VCAMD);
			goto _kdCISModulePowerOn_exit_;
		}

        udelay(1); //t2-1

        /* MCLK */
        ISP_MCLK2_EN(1);
        mdelay(1); //t2-2

		/* enable active sensor */
		if (GPIO_CAMERA_INVALID != pinSet[pinSetIdx][IDX_PS_CMPDN]) {
			mtkcam_gpio_set(pinSetIdx, CAMPDN, pinSet[pinSetIdx][IDX_PS_CMPDN + IDX_PS_ON]);
		}
        mdelay(100); //t4
	}

### 3.添加头文件相关定义：
#### (1)kd_imgsensor.h
**file path**：alps/kernel-3.18/drivers/misc/mediatek/imgsensor/inc/kd_imgsensor.h

**example**：

	#define HI556_SENSOR_ID                         0x0556
	......................................................
	#define SENSOR_DRVNAME_HI556_MIPI_RAW           "hi556mipiraw" 

#### (2)kd_sensorlist.h
**file path**：alps/kernel-3.18/drivers/misc/mediatek/imgsensor/src/mt6580/kd_sensorlist.h

**example**：

	UINT32 HI556_MIPI_RAW_SensorInit(PSENSOR_FUNCTION_STRUCT *pfFunc);
 
	.................................................................

	#if defined(HI556_MIPI_RAW)
		{HI556_SENSOR_ID, SENSOR_DRVNAME_HI556_MIPI_RAW,HI556_MIPI_RAW_SensorInit},
	#endif

</br>

## 四、Hal层添加效果驱动文件、sensorlist文件中的定义与相关头文件定义：
### 1.添加效果驱动文件：
**file path**：</br>
【By project】：alps/vendor/mediatek/proprietary/custom/&lt;project>/hal/imgsensor/ </br>
【By platform】：alps/vendor/mediatek/proprietary/custom/&lt;platform>/hal/imgsensor/

### 2.添加sensorlist文件中的定义：
**file path**：alps/vendor/mediatek/proprietary/custom/mt6580/hal/imgsensor_src/sensorlist.cpp

**example**：

	#if defined(HI556_MIPI_RAW)
    	RAW_INFO(HI556_SENSOR_ID, SENSOR_DRVNAME_HI556_MIPI_RAW, NULL), 
	#endif

### 3.添加头文件相关定义：
**file path**：alps/device/mediatek/common/kernel-headers/kd_imgsensor.h

**example**：

	#define HI556_SENSOR_ID                         0x0556
	......................................................
	#define SENSOR_DRVNAME_HI556_MIPI_RAW           "hi556mipiraw"