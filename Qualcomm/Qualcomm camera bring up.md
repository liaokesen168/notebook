# <center><font size=8>Qualcomm camera bring up</font></center>
-----------------------------------------------------------------------------------
</br>

## 1.1 kernel层
### 1.1.1 电源、时钟、RESET管脚等配置（dtsi配置）：
**file path**：kernel/arch/arm/boot/dts/qcom/sdm660-camera-sensor-mtp_gm8plus.dtsi

**example**：

	qcom,camera@2 {
		cell-index = <2>;
		compatible = "qcom,camera";
		reg = <0x02>;
		qcom,csiphy-sd-index = <2>;
		qcom,csid-sd-index = <2>;
		qcom,mount-angle = <270>;
		//qcom,actuator-src = <&actuator2>;
		//qcom,led-flash-src = <&led_flash1>;
		//qcom,eeprom-src = <&eeprom2>;
		cam_vio-supply = <&pm660_l11>;
		cam_vana-supply = <&cam_avdd_gpio_regulator>;
		cam_vdig-supply = <&cam_dvdd_gpio_regulator>;
		qcom,cam-vreg-name = "cam_vio", "cam_vana", "cam_vdig";
		qcom,cam-vreg-min-voltage = <1780000 0 0>;
		qcom,cam-vreg-max-voltage = <1950000 0 0>;
		qcom,cam-vreg-op-mode = <105000 0 0>;
		qcom,gpio-no-mux = <0>;
		pinctrl-names = "cam_default", "cam_suspend";
		pinctrl-0 = <&cam_sensor_mclk1_active
				 &cam_sensor_front_active>;
		pinctrl-1 = <&cam_sensor_mclk1_suspend
				 &cam_sensor_front_suspend>;
		gpios = <&tlmm 33 0>,
			<&tlmm 47 0>;
		qcom,gpio-reset = <1>;
		qcom,gpio-req-tbl-num = <0 1>;
		qcom,gpio-req-tbl-flags = <1 0>;
		qcom,gpio-req-tbl-label = "CAMIF_MCLK1",
					"CAM_RESET1";
		qcom,sensor-position = <1>;
		qcom,sensor-mode = <0>;
		qcom,cci-master = <1>;
		status = "ok";
		clocks = <&clock_mmss MCLK1_CLK_SRC>,
			<&clock_mmss MMSS_CAMSS_MCLK1_CLK>;
		clock-names = "cam_src_clk", "cam_clk";
		qcom,clock-rates = <24000000 0>;
	};

	cam_avdd_gpio_regulator: cam_avdd_fixed_regulator {
		compatible = "regulator-fixed";
		regulator-name = "cam_avdd_gpio_regulator";
		regulator-min-microvolt = <2800000>;
		regulator-max-microvolt = <2800000>;
		enable-active-high;
		gpio = <&tlmm 31 0>;
		vin-supply = <&pm660l_bob>;
	};

	cam_dvdd_gpio_regulator: cam_dvdd_fixed_regulator {
		compatible = "regulator-fixed";
		regulator-name = "cam_dvdd_gpio_regulator";
		regulator-min-microvolt = <1200000>;
		regulator-max-microvolt = <1200000>;
		enable-active-high;
		gpio = <&tlmm 29 0>;
		vin-supply = <&pm660_s5>;
	};

</br>

## 1.2 hal层
### 1.2.1 添加驱动文件：
**file path**：vendor/qcom/proprietary/mm-camera/mm-camera2/media-controller/modules/sensors/sensor/libs/

**example**：</br>
1.创建sensor库文件夹。 </br>
2.添加以下驱动恩建及Makefile: </br>
(1)&lt;sensor>&#95;lib.c </br>
(2)&lt;sensor>&#95;lib.h </br>
(3)Makefile </br>

<font color=ff0000>**key note**</font>：在&lt;sensor>&#95;lib.h文件中注意填写正确的I2C地址与上下电时序。

### 1.2.2 添加效果文件：
**file path**：vendor/qcom/proprietary/mm-camera/mm-camera2/media-controller/modules/sensors/chromatix/0310/

**example**： </br>
1.创建sensor的效果文件夹。 </br>
2.添加效果文件。 </br>

### 1.2.3 添加&lt;sensor>&#95;chromatix.xml配置文件：
1.创建&lt;sensor>&#95;chromatix.xml配置文件。 </br>
**file path**：vendor/qcom-proprietary/mm-camera/mm-camera2/media-controller/modules/sensors/configs/

**example**：

	<ChromatixConfigurationRoot>
	  <CommonChromatixInfo>
	    <ChromatixName>
	      <ISPCommon>s5k3h7_sunny_a8s05a_front_i_common</ISPCommon>
	      <PostProc>s5k3h7_sunny_a8s05a_front_i_postproc</PostProc>
	    </ChromatixName>
	  </CommonChromatixInfo>
	  <ResolutionChromatixInfo>
	    <ChromatixName sensor_resolution_index="0">
	      <ISPPreview>s5k3h7_sunny_a8s05a_front_i_snapshot</ISPPreview>
	      <ISPSnapshot>s5k3h7_sunny_a8s05a_front_i_snapshot</ISPSnapshot>
	      <ISPVideo>s5k3h7_sunny_a8s05a_front_i_video_full</ISPVideo>
	      <CPPVideo>s5k3h7_sunny_a8s05a_front_i_cpp_video_full</CPPVideo>
	      <CPPPreview>s5k3h7_sunny_a8s05a_front_i_cpp_preview</CPPPreview>
	      <CPPSnapshot>s5k3h7_sunny_a8s05a_front_i_cpp_snapshot</CPPSnapshot>
	      <CPPLiveshot>s5k3h7_sunny_a8s05a_front_i_cpp_liveshot</CPPLiveshot>
	      <A3Preview>s5k3h7_sunny_a8s05a_front_i_zsl_preview</A3Preview>
	      <A3Video>s5k3h7_sunny_a8s05a_front_i_zsl_video</A3Video>
	    </ChromatixName>
	    <ChromatixName sensor_resolution_index="1">
	      <ISPPreview>s5k3h7_sunny_a8s05a_front_i_preview</ISPPreview>
	      <ISPSnapshot>s5k3h7_sunny_a8s05a_front_i_preview</ISPSnapshot>
	      <ISPVideo>s5k3h7_sunny_a8s05a_front_i_default_video</ISPVideo>
	      <CPPVideo>s5k3h7_sunny_a8s05a_front_i_cpp_video</CPPVideo>
	      <CPPPreview>s5k3h7_sunny_a8s05a_front_i_cpp_preview</CPPPreview>
	      <CPPSnapshot>s5k3h7_sunny_a8s05a_front_i_cpp_snapshot</CPPSnapshot>
	      <CPPLiveshot>s5k3h7_sunny_a8s05a_front_i_cpp_liveshot</CPPLiveshot>
	      <A3Preview>s5k3h7_sunny_a8s05a_front_i_a3_default_preview</A3Preview>
	      <A3Video>s5k3h7_sunny_a8s05a_front_i_a3_default_video</A3Video>
	    </ChromatixName>
	    <ChromatixName sensor_resolution_index="2">
	      <ISPPreview>s5k3h7_sunny_a8s05a_front_i_hfr_60</ISPPreview>
	      <ISPSnapshot>s5k3h7_sunny_a8s05a_front_i_hfr_60</ISPSnapshot>
	      <ISPVideo>s5k3h7_sunny_a8s05a_front_i_hfr_60</ISPVideo>
	      <CPPVideo>s5k3h7_sunny_a8s05a_front_i_cpp_hfr_60</CPPVideo>
	      <CPPPreview>s5k3h7_sunny_a8s05a_front_i_cpp_hfr_60</CPPPreview>
	      <CPPSnapshot>s5k3h7_sunny_a8s05a_front_i_cpp_hfr_60</CPPSnapshot>
	      <CPPLiveshot>s5k3h7_sunny_a8s05a_front_i_cpp_hfr_60</CPPLiveshot>
	      <A3Preview>s5k3h7_sunny_a8s05a_front_i_a3_hfr_60</A3Preview>
	      <A3Video>s5k3h7_sunny_a8s05a_front_i_a3_hfr_60</A3Video>
	    </ChromatixName>
	    <ChromatixName sensor_resolution_index="3">
	      <ISPPreview>s5k3h7_sunny_a8s05a_front_i_hfr_90</ISPPreview>
	      <ISPSnapshot>s5k3h7_sunny_a8s05a_front_i_hfr_90</ISPSnapshot>
	      <ISPVideo>s5k3h7_sunny_a8s05a_front_i_hfr_90</ISPVideo>
	      <CPPVideo>s5k3h7_sunny_a8s05a_front_i_cpp_hfr_90</CPPVideo>
	      <CPPPreview>s5k3h7_sunny_a8s05a_front_i_cpp_hfr_90</CPPPreview>
	      <CPPSnapshot>s5k3h7_sunny_a8s05a_front_i_cpp_hfr_90</CPPSnapshot>
	      <CPPLiveshot>s5k3h7_sunny_a8s05a_front_i_cpp_hfr_90</CPPLiveshot>
	      <A3Preview>s5k3h7_sunny_a8s05a_front_i_a3_hfr_90</A3Preview>
	      <A3Video>s5k3h7_sunny_a8s05a_front_i_a3_hfr_90</A3Video>
	    </ChromatixName>
	    <ChromatixName sensor_resolution_index="4">
	      <ISPPreview>s5k3h7_sunny_a8s05a_front_i_hfr_120</ISPPreview>
	      <ISPSnapshot>s5k3h7_sunny_a8s05a_front_i_hfr_120</ISPSnapshot>
	      <ISPVideo>s5k3h7_sunny_a8s05a_front_i_hfr_120</ISPVideo>
	      <CPPVideo>s5k3h7_sunny_a8s05a_front_i_cpp_hfr_120</CPPVideo>
	      <CPPPreview>s5k3h7_sunny_a8s05a_front_i_cpp_hfr_120</CPPPreview>
	      <CPPSnapshot>s5k3h7_sunny_a8s05a_front_i_cpp_hfr_120</CPPSnapshot>
	      <CPPLiveshot>s5k3h7_sunny_a8s05a_front_i_cpp_hfr_120</CPPLiveshot>
	      <A3Preview>s5k3h7_sunny_a8s05a_front_i_a3_hfr_120</A3Preview>
	      <A3Video>s5k3h7_sunny_a8s05a_front_i_a3_hfr_120</A3Video>
	    </ChromatixName>
	  </ResolutionChromatixInfo>
	</ChromatixConfigurationRoot>

<font color=ff0000>**key note**</font>：文件中一系列<font color=ff0000>"s5k3h7&#95;sunny&#95;a8s05a&#95;front_i&#95;***"</font> name一定要与1.2.2中添加的效果文件中的Makefile里的<font color=ff0000>LOCAL&#95;MODULE</font>名字一样。

2.配置make file文件： </br>
**file path**：/vendor/qcom-proprietary/mm-camera/mm-camera2/media-controller/modules/sensors/configs/Android.mk

**example**：
	
	include $(CLEAR_VARS)
	LOCAL_MODULE:= gm8plus_s5k3h7_sunny_front_i_chromatix.xml
	LOCAL_MODULE_CLASS := EXECUTABLES
	LOCAL_SRC_FILES := gm8plus_s5k3h7_sunny_front_i_chromatix.xml
	LOCAL_MODULE_TAGS := optional
	LOCAL_MODULE_PATH := $(TARGET_OUT_VENDOR)/etc/camera
	LOCAL_MODULE_OWNER := qti
	include $(BUILD_PREBUILT)

### 1.2.4 配置&lt;project&platform>_camera.xml文件：
**file path**：vendor/qcom/proprietary/mm-camera/mm-camera2/media-controller/modules/sensors/configs/gm8plus_camera.xml

**example**：

	<CameraModuleConfig>
        <CameraId>2</CameraId>
        <SensorName>gm8plus_s5k3h7_sunny_front_i</SensorName>
        <!--EepromName>gm8plus_s5k3h7_sunny_front_i_eeprom</EepromName-->
        <FlashName>pmic</FlashName>
        <ChromatixName>gm8plus_s5k3h7_sunny_front_i_chromatix</ChromatixName>
        <ModesSupported>1</ModesSupported>
        <Position>FRONT</Position>
        <MountAngle>270</MountAngle>
        <CSIInfo>
            <CSIDCore>2</CSIDCore>
            <LaneMask>0x1F</LaneMask>
            <LaneAssign>0x4320</LaneAssign>
            <ComboMode>0</ComboMode>
        </CSIInfo>
        <LensInfo>
            <FocalLength>1.97</FocalLength>
            <FNumber>2.0</FNumber>
            <TotalFocusDistance>1.9</TotalFocusDistance>
            <HorizontalViewAngle>84.0</HorizontalViewAngle>
            <VerticalViewAngle>63.0</VerticalViewAngle>
            <MinFocusDistance>0.1</MinFocusDistance>
        </LensInfo>
    </CameraModuleConfig>

**key note**：上述代码中的SensorName、EepromName和ChromatixName一定要填写正确。

### 1.2.5 把前面所有make file中的LOCAL_MODULE添加最终的make file（device-vendor.mk）：

**file path**：vendor/qcom/proprietary/common/config/device-vendor.mk

**example**：
	
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_common
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_postproc
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_cpp_preview
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_cpp_snapshot
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_cpp_liveshot
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_cpp_us_chromatix
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_cpp_ds_chromatix
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_cpp_video_full
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_cpp_video
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_cpp_hfr_120
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_cpp_hfr_90
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_cpp_hfr_60
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_a3_default_preview
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_a3_default_video
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_a3_hfr_60
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_a3_hfr_90
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_a3_hfr_120
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_zsl_preview
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_zsl_video
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_snapshot
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_video_full
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_preview
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_liveshot
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_default_video
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_hfr_60
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_hfr_90
	MM_CAMERA += libchromatix_gm8plus_s5k3h7_sunny_front_i_hfr_120
	
	MM_CAMERA += libmmcamera_gm8plus_s5k3h7_sunny_front_i
	MM_CAMERA += gm8plus_s5k3h7_sunny_front_i_chromatix.xml