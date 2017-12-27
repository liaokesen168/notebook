# <center><font size=8>MTK camera flashlight CTS test failed</font></center>
-----------------------------------------------------------------------------------
</br>

## 一、CTS test failed的log抓取方法：
<font size=4>1.安装CTS Verifier  apk，打开MTKlog，找到对应模块的Fail测试。</font></br>
<font size=4>2.输入Mtklog暗码：*#*#3646633#*#*。</font>

</br>

## 二、修改metedata中的配置文件：
**file path**：alps/vendor/mediatek/proprietary/custom/mt6580/hal/imgsensor_metadata/

**key note**：<font color=ff0000>此文件夹下如果没有指定camera的配置文件夹，代码会走common中的配置。</font>

<font size=4>1.创建指定camera的配置文件夹（复制其他camera的配置文件然后修改名称）。</font></br>
<font size=4>2.打开相关文件进行配置。</font>

</br>

## 三、front camera flashlight test failed:
**file path**：alps/vendor/mediatek/proprietary/custom/mt6580/hal/imgsensor_metadata/hi556_mipi_raw/config_static_metadata.project.flashlight.hi556mipiraw.h

**example**：

    if (0 == uFacing)
    {
        CONFIG_ENTRY_VALUE(MTK_FLASH_INFO_AVAILABLE_TRUE, MUINT8)
    }
    else
    {
        CONFIG_ENTRY_VALUE(MTK_FLASH_INFO_AVAILABLE_TRUE, MUINT8)
    }

**key note**：<font color=ff0000>把MTK&#95;FLASH&#95;INFO&#95;AVAILABLE&#95;FALSE改成MTK&#95;FLASH&#95;INFO&#95;AVAILABLE&#95;TRUE</font>