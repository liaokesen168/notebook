# <center><font size=8>Qualcomm camera bring up小结</font></center>
-----------------------------------------------------------------------------------
</br>

## 1.1 read sensor id failed
### 1.1.1 上电时序

1.确认sensor上电时序：读取sensor id失败首先要根据sensor的datasheet确认一下sensor的上电时序是否正确。</br>
2.确认sensor上电是否成功：在确保上电时序正确后，再确认sensor是否上电成功：可以在msm&#95;sensor&#95;power&#95;down和msm&#95;camera&#95;power&#95;down两个function的前面加return 0；不让sensor下电，然后再去量ssensor的电是否都有。</br>

### 1.1.2 时钟源

1.配置好上电后就是时钟的配置了，除开camera的dtsi中时钟的GPIO要配置正确，还要在pinctrl中的dtsi配置正确的GPIO。</br>
2.camera中dtsi的时钟源配置，这次camera bring up就遇到原理图中是MCLK1但dtsi中配置成MCLK2才能正常工作的情况，这里不知道是平台端c时钟源的配置有问题还是什么情况，所以以后遇到无法读取id的时候注意下时钟源配置这个坑！！！


### 1.1.3 camera_config.xml编写问题导致camera闪退

1.出现点击camera然后闪退的问题，请注意一下camera&#95;config.xml的编写是否有格式上的错误！

### 1.1.4 在bring up中避免其他camera配置干扰

1.在bring up中可以将camera&#95;config.xml的其他camera配置删掉，和dtsi中其他camera的配置删掉，从而避免这些配置干扰自己的camera bring up配置。