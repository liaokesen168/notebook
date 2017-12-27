# <center><font size=8>Qualcomm camera OTP bring up</font></center>
-----------------------------------------------------------------------------------
</br>

## 1.1 kernel层：
### 1.1.1 eeprom读取地址、大小和sensor上电时序等配置（dts配置）：
**file path**：
kernel/arch/arm/boot/dts/qcom/sdm660-camera-sensor-mtp_gm8plus.dtsi

**example**：

	eeprom2: qcom,eeprom@2 {
        cell-index = <2>;
        reg = <0x2>;
        compatible = "qcom,eeprom";

        qcom,eeprom-name = "s5k3h7_sunny_a8s05a_front_i";
        qcom,i2c-freq-mode = <1>;
        qcom,slave-addr = <0x20>;
        qcom,num-blocks = <9>;

        //page 0
        qcom,page0 = <1 0x100 2 0x01 1 1>;
        qcom,poll0 = <0 0x0 2 0 1 1>;
        qcom,mem0 = <0 0x0 2 0 1 0>;

        qcom,page1 = <1 0x0A02 2 0x01 1 1>;
        qcom,poll1 = <0 0x0 2 0 1 1>;
        qcom,mem1 = <0 0x0 2 0 1 0>;

        qcom,page2 = <1 0x0A00 2 0x01 1 100>;
        qcom,poll2 = <0 0x0 2 0 1 1>;
        qcom,mem2 = <64 0x0A04 2 0 1 0>;

        //page 1
        qcom,page3 = <1 0x100 2 0x01 1 1>;
        qcom,poll3 = <0 0x0 2 0 1 1>;
        qcom,mem3 = <0 0x0 2 0 1 0>;

        qcom,page4 = <1 0x0A02 2 0x02 1 1>;
        qcom,poll4 = <0 0x0 2 0 1 1>;
        qcom,mem4 = <0 0x0 2 0 1 0>;

        qcom,page5 = <1 0x0A00 2 0x01 1 100>;
        qcom,poll5 = <0 0x0 2 0 1 1>;
        qcom,mem5 = <64 0x0A04 2 0 1 0>;

        //page 2
        qcom,page6 = <1 0x100 2 0x01 1 1>;
        qcom,poll6 = <0 0x0 2 0 1 1>;
        qcom,mem6 = <0 0x0 2 0 1 0>;

        qcom,page7 = <1 0x0A02 2 0x03 1 1>;
        qcom,poll7 = <0 0x0 2 0 1 1>;
        qcom,mem7 = <0 0x0 2 0 1 0>;

        qcom,page8 = <1 0x0A00 2 0x01 1 100>;
        qcom,poll8 = <0 0x0 2 0 1 1>;
        qcom,mem8 = <64 0x0A04 2 0 1 0>;

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

        qcom,cam-power-seq-type = "sensor_gpio",
                      "sensor_vreg",
                      "sensor_vreg",
                      "sensor_vreg",
                      "sensor_clk",
                      "sensor_gpio";
        qcom,cam-power-seq-val =  "sensor_gpio_reset",
                      "cam_vdig",
                      "cam_vana",
                      "cam_vio",
                      "sensor_cam_mclk",
                      "sensor_gpio_reset";
        qcom,cam-power-seq-cfg-val = <0 1 1 1 24000000 1>;
        qcom,cam-power-seq-delay = <1 1 1 0 1 11>;

        qcom,sensor-position = <1>;
        qcom,sensor-mode = <0>;
        qcom,cci-master = <1>;
        status = "ok";
        clocks = <&clock_mmss MCLK1_CLK_SRC>,
            <&clock_mmss MMSS_CAMSS_MCLK1_CLK>;
        clock-names = "cam_src_clk", "cam_clk";
        qcom,clock-rates = <24000000 0>;
    };

<font color=ff0000>**key note**</font>： </br>
1.eeprom一页的读取示例解析：

	//page 0
    qcom,page0 = <1 0x100 2 0x01 1 1>; //打开stream（解析：向0x100地址的寄存器中写0x01）
    qcom,poll0 = <0 0x0 2 0 1 1>;
    qcom,mem0 = <0 0x0 2 0 1 0>;

    qcom,page1 = <1 0x0A02 2 0x01 1 1>; //设置page
    qcom,poll1 = <0 0x0 2 0 1 1>;
    qcom,mem1 = <0 0x0 2 0 1 0>;

    qcom,page2 = <1 0x0A00 2 0x01 1 100>; //使能读
    qcom,poll2 = <0 0x0 2 0 1 1>;
    qcom,mem2 = <64 0x0A04 2 0 1 0>; //读数据（解析：以0xa04为起始地址，读取64个字节大小的数据）

2.sensor的电源gpio配置与上电时序要写正确。

</br>

## 1.2 hal层：
### 1.2.1 添加eeprom驱动文件：
**file path**：vendor/qcom/proprietary/mm-camera/mm-camera2/media-controller/modules/sensors/eeprom/libs/

**example**： </br>
1.创建sensor的eeprom的文件夹。 </br>
2.添加sensor的eeprom的驱动文件: </br>
(1)&lt;sensor>&#95;eeprom.c </br>
(2)&lt;sensor>&#95;eeprom.h </br>
(3)Makefile </br>

### 1.2.2 配置&lt;project&platform>_camera.xml文件：
**file path**：vendor/qcom-proprietary/mm-camera/mm-camera2/media-controller/modules/sensors/configs/gm8plus_camera.xml

**example**： </br>
添加下述代码中的&lt;EepromName>：

    <CameraModuleConfig>
        <CameraId>2</CameraId>
        <SensorName>s5k3h7_sunny_a8s05a_front_i</SensorName>
        <EepromName>s5k3h7_sunny_a8s05a_front_i</EepromName>
        <FlashName>pmic</FlashName>
        <ChromatixName>s5k3h7_sunny_a8s05a_front_i_chromatix</ChromatixName>
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

### 1.2.3 把前面所有make file中的LOCAL_MODULE添加最终的make file（device-vendor.mk）：
**file path**：vendor/qcom/proprietary/common/config/device-vendor.mk

**example**：

	MM_CAMERA += libmmcamera_s5k3h7_sunny_a8s05a_front_i_eeprom

<font color=ff0000>**key note**</font>：example中的<font color=ff0000>s5k3h7&#95;sunny&#95;a8s05a&#95;front&#95;i&#95;eeprom</font>要与《1.2.1》添加的Makefile文件中的LOCAL_MODULE、《1.2.2》配置的xml文件中的&lt;EepromName>一致。