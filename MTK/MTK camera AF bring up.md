# <center><font size=8>MTK camera AF bring up</font></center>
-----------------------------------------------------------------------------------
</br>

## 一、DWS配置：
<font size=4>1.在DWS里配置好camera AF供电的GPIO。</font></br>
<font size=4>2.配置好camera AF使用的i2c设备。</font>

</br>

## 二、config配置文件：
### 1.Configure camera AF hal driver in ProjectConfig.mk
**file path**：alps/device/mediatek/$project/ProjectConfig.mk

**example**：

	CUSTOM_HAL_MAIN_LENS = dw9714af
	..............................
	CUSTOM_KERNEL_LENS = dw9714af

</br>

## 三、kernel层：
### 1.添加AF上下电时序：
**file path**：alps/<kernel>/drivers/misc/mediatek/imgsensor/src/mt6580/camera_hw/kd_camera_hw.c

**example**：
在int kdCISModulePowerOn(CAMERA_DUAL_CAMERA_SENSOR_ENUM SensorIdx, char          	*currSensorName, BOOL On, char *mode_name) function中添加：

	if (currSensorName && (0 == strcmp(SENSOR_DRVNAME_HI846_MIPI_RAW, currSensorName))) {

		............................................................................................
		/* AF_VCC */
		if (TRUE != _hwPowerOn(VCAMAF, VOL_2800)) {
			PK_DBG("[CAMERA SENSOR] Fail to enable analog power (VCAM_AF),power id = %d\n", VCAMAF);
				goto _kdCISModulePowerOn_exit_;
		}
		mdelay(5);
		............................................................................................
	}

</br>

## 四、hal层：
### 1.添加lenslist中的定义：
**file path**：alps/vendor/mediatek/proprietary/custom/mt6580/hal/lens/lenslist.cpp

**example**：

    #if defined(DW9714AF)
        {HI846_SENSOR_ID, DW9714AF_LENS_ID, "DW9714AF", pDW9714AF_getDefaultData},
    #endif