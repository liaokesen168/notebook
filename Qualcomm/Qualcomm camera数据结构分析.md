# <center><font size=8>Qualcomm camera数据结构分析</font></center>
-----------------------------------------------------------------------------------
</br>

## 1.1 结构体msm&#95;sensor&#95;slave&#95;info

	struct msm_camera_sensor_slave_info {
		char sensor_name[32]; //sensor名称
		char eeprom_name[32]; //eeprom名称
		char actuator_name[32]; //actuator名称
		enum msm_sensor_camera_id_t camera_id; //camera id号
		uint16_t slave_addr; //从设备地址
		enum msm_camera_i2c_reg_addr_type addr_type; //camera i2c寄存器地址类型
		struct msm_sensor_id_info_t sensor_id_info; //sensor 芯片id信息
		struct msm_sensor_power_setting_array power_setting_array; //上电序列
		uint8_t  is_init_params_vaild; //初始化参数是否有效标志位
		struct msm_sensor_init_params sensor_init_params; //sensor初始化参数
	}; 

## 1.2 枚举msm&#95;sensor&#95;camera&#95;id&#95;t
	enum msm_sensor_camera_id_t {
		CAMERA_0, //camera id 号0
		CAMERA_1, //camera id 号1
		CAMERA_2, //camera id 号2
		CAMERA_3, //camera id 号3
		MAX_CAMERAS, //支持的最大id号
	};

## 1.3 枚举msm&#95;camera&#95;i2c&#95;reg&#95;addr&#95;type
	enum msm_camera_i2c_reg_addr_type {
		MSM_CAMERA_I2C_BYTE_ADDR = 1, //1字节型
		MSM_CAMERA_I2C_WORD_ADDR, //2字型
		MSM_CAMERA_I2C_3B_ADDR, //3字节型
	};

## 1.4 结构体msm&#95;sensor&#95;id&#95;info&#95;t
	struct msm_sensor_id_info_t {
		uint16_t sensor_id_reg_addr; //对应sensor id号的寄存器地址
		uint16_t sensor_id; //sensor id号
	};

## 1.5 结构体msm&#95;sensor&#95;power&#95;setting&#95;array
	struct  msm_sensor_power_setting_array {
		struct msm_sensor_power_setting *power_setting; //上电时序
		uint16_t size; //上电时序大小
		struct msm_sensor_power_setting *power_down_setting; //下电时序
		uint16_t size_down; //下电时序大小
	};

结构体msm&#95;sensor&#95;power&#95;setting

	struct msm_sensor_power_setting {     
		enum msm_sensor_power_seq_type seq_type;
		uint16_t seq_val;
		long config_val;
		uint16_t delay;
		void *data[10];
	};

枚举msm&#95;sensor&#95;power&#95;seq&#95;type&#95;t

	enum camera_power_seq_type {
		CAMERA_POW_SEQ_CLK,
		CAMERA_POW_SEQ_GPIO,
		CAMERA_POW_SEQ_VREG,
		CAMERA_POW_I2C_MUX,
		CAMERA_POW_SEQ,
	};

## 1.6 结构体msm&#95;sensor&#95;init&#95;params
	struct  msm_sensor_init_params{
		/* mask of modes supported: 2D, 3D */
		int modes_supported; //支持camera的模式
		/* sensor position: front, back */
		enum camb_position_t position; //sensor的位置
		/* sensor mount angle */
		uint32_t sensor_mount_angle; //sensor安装的角度
	};

枚举camb&#95;position&#95;t

	enum camb_position_t{
		BACK_CAMERA_B, //后摄
		FRONT_CAMERA_B, //前摄
		INVALID_CAMERA_B, //非法
	}

## 2.1 Sensor输出设置
### 2.1.1 sensor输出格式sensor&#95;output&#95;t
	typedef struct{
		sensor_output_format_t output_format; //输出格式
		sensor_connection_mode_t connection_mode; //连接模式
		sensor_raw_output_t raw_output; //raw图格式
	}sensor_output_t;

其中 output&#95;format,connection&#95;mode和raw&#95;output都为枚举类型，定义如下：

枚举sensor&#95;output&#95;format

	typedef enum {
		SENSOR_BAYER, //Bayer格式
		SENSOR_YCBCR  //YUV格式（Y,Cb,Cr）
	} sensor_output_format_t;

枚举sensor&#95;connection&#95;mode&#95;t

	typedef enum {
		SENSOR_PARALLEL, //并行
		SENSOR_MIPI_CSI, //MIPI CSI
		SENSOR_MIPI_CSI_1, //CSI1
		SENSOR_MIPI_CSI_2, //CSI2
	} sensor_connection_mode_t;

枚举sensor&#95;raw&#95;output&#95;t

	typedef enum {
		SENSOR_8_BIT_DIRECT, //8-bit
		SENSOR_10_BIT_DIRECT, //10-bit
		SENSOR_12_BIT_DIRECT, //12-bit
	} sensor_raw_output_t;
<font color=ff0000>**key note**</font>：10-bit RAW图数据是通过数据包的格式进行传输的，打包之后的数据格式为8-bit。（<font color=ff0000>**！补表补表！**</font>）下表是对RAW10数据包格式限制条件的说明，每一个数据包的长度必须是表中数值的整数倍，bit位传输顺序服从CSI-2规则，LSB优先。

## 2.2 Sensor像素格式信息
### 2.2.1 结构体sensor&#95;pix&#95;fmt&#95;info&#95;t

	struct sensor_pix_fmt_info_t {
		uint32_t fourcc;
	};

### 2.2.2 像素格式的值V4L2&#95;PIX&#95;FMT&#95;SRGGB10是宏定义在linux/videodev2.h中，如下：

	#define  v4l2_fourcc(a,b,c,d) \
	((__u32)(a) | ((__u32)(b) << 8) | ((__u32)(c) << 16) | ((__u32)(d) << 24)
	
	#define V4L2_PIX_FMT_SBGGR8
	v4l2_fourcc('B','A','8','1') //BGGR 8bit
	#define V4L2_PIX_FMT_SGBRG8
	v4l2_fourcc('G','B','R','G') //GBRG 8bit
	#define V4L2_PIX_FMT_SGRBG8
	v4l2_fourcc('G','R','B','G') //GRBG 8bit
	#define V4L2_PIX_FMT_SRGGB8
	v4l2_fourcc('R','G','G','B') //RGGB 8bit
	#define V4L2_PIX_FMT_SBGGR10
	v4l2_fourcc('B','G','1','0') //BGGR 10bit
	#define V4L2_PIX_FMT_SGBRG10
	v4l2_fourcc('G','B','1','0') //GBRG 10bit
	#define V4L2_PIX_FMT_SGRBG10
	v4l2_fourcc('B','A','1','0') //GRBG 10bit
	#define V4L2_PIX_FMT_SRGGB10
	v4l2_fourcc('R','G','1','0') //RGGB 10bit
	#define V4L2_PIX_FMT_SBGGR12
	v4l2_fourcc('B','G','1','2') //BGGR 12bit
	#define V4L2_PIX_FMT_SGBRG12
	v4l2_fourcc('G','B','1','2') //GBRG 12bit
	#define V4L2_PIX_FMT_SGRBG12
	v4l2_fourcc('B','A','1','2') //GRBG 12bit
	#define V4L2_PIX_FMT_SRGGB12
	v4l2_fourcc('R','G','1','2') //RGGB 12bit

### 2.2.3 素格式的值MSM_V4L2_PIX_FMT_META是宏定义在media/msm_cam_sensor.h中：
	
	#define MSM_V4L2_PIX_FMT_META v4l2_fourcc('M','E','T','A')

## 2.3 Sensor输出尺寸设置
结构体sensor&#95;lib&#95;out&#95;info&#95;t用于保存sensor所支持的不同分辨率的信息。

### 2.3.1 各参数含义解析
结构体sensor&#95;lib&#95;out&#95;info&#95;t

	struct sensor_lib_out_info_t {
		uint16_t x_output; //sensor输出宽度（pixels）
		uint16_t y_output; //sensor输出高度（pixels）
		uint16_t line_length_pclk;   //每一帧每一行多少个pixels
		uint16_t frame_length_lines; //每一帧多少行
		uint32_t vt_pixel_clk;       //sensor扫描速率（pixels per sec）
		uint32_t op_pixel_clk;       //sensor实际输出速率（pixels per sec）
		uint16_t bining_factor; /*?: 1 if average is taken, >1 if sum is taken(applies only for if this resolution has binnig) */
		float min_fps; //sensor支持的最小帧率
		float max_fps; //sensor支持的最大帧率
		uint32_t mode; //分辨率所对应的模式
	};
<font color=ff0000>**key note**</font>：<br>
1.使用Chromatix软件进行tuning设置Image Width和Image Height的值时分别参考此处x&#95;output和y&#95;output。<br>
2.line&#95;length&#95;pckl 和frame&#95;length&#95;lines 是指包含blanking的宽度值和高度值。<br>
3.line&#95;lenth&#95;pclk和frame&#95;length&#95;lines决定帧的大小。

### 2.3.2 什么是blanking
每一帧图像的每一行输出是遵循CSI2的通用帧格式。每一行的行尾（Packet Footer，PF）到下一行行头（Packet Header，PH）的期间称为“line blanking”。同样的，每一帧的帧尾（Frame End，FE）到下一帧帧头（Frame Start，FS）的期间称为“frame blanking”。

### 2.3.3 vt&#95;pixel&#95;clk和op&#95;pixel&#95;clk
1.vt&#95;pixel&#95;clk时钟用于内部图像处理，计算曝光时间和帧率等。<br>
2.帧率：frame rate =  vt&#95;pixel&#95;clk / (line&#95;length&#95;pclk * frame&#95;length&#95;lines);<br>
3.op&#95;pixel&#95;clk = (sensor输出实际比特率) / bits-per-pixel;<br>

<font color=ff0000>**举个栗子**</font>：
如果 MIPI DDR 时钟值 (sensor MIPI 的时钟 lane 频率) 为 300Mhz, 同时 sensor 使用4 个 lane 传输数据, 每一个 lane 的数据率是 300 * 2 = 600Mhz. 因此, 总数据率为 600 * 4 = 2400Mhz. 对于 10bit 的 bayer sensor, op&#95;pixel&#95;clk 值可设置为 2400/10 = 240Mhz.这些值可以从 sensor 的寄存器设置中计算出来。

### 2.3.4 参数mode，分辨率所对应的模式，其中mode值得宏定义如下：

	/* HFR模式不用于常规的camera，camcorder */
	#define SENSOR_DEFAULT_MODE (1 << 0) //默认模式
	#define SENSOR_HFR_MODE (1 << 1) //高帧率模式，用于捕捉慢动作视频
	#define SENSOR_HDR_MODE (1 << 2) //高动态范围图像模式 

### 2.3.5 x&#95;output & y&#95;output参数设置

x&#95;output和y&#95;output是sensor输出图像的重要参数，分别代表了图像的宽度和高度，单位是pixel。上层camera app最终就是从这里获取的sensor输出图像的宽度和高度信息，然后根据此信息裁剪出各种尺寸的图片。<br>
<font color=ff0000>**key note**</font>：Camera app照相所支持的图片尺寸在mct_pipeline.c文件中定义。

### 2.3.6 结构体sensor&#95;lib&#95;out&#95;info&#95;array

	struct sensor_lib_out_info_array {
		struct sensor_lib_out_info_t *out_info; //指向sensor_lib_out_info_t结构体的指针
		uint16_t size; //sensor_lib_out_info_t结构体数组长度
	}；

ARRAY_SIZE的宏定义如下：

	#define ARRAY_SIZE(x) (sizeof(x) / sizeof((x)[0])) //获取数组长度。

## 2.4 Sensor输出寄存器地址设置
结构体msm&#95;sensor&#95;output&#95;reg&#95;addr

	struct msm_sensor_output_reg_addr_t {
		uint16_t x_output; //寄存器X_OUT_SIZE地址
		uint16_t y_output; //寄存器Y_OUT_SIZE地址
		uint16_t line_length_pclk;   //寄存器LIN_LENGTH_PCK地址
		uint16_t frame_length_lines; //寄存器FRM_LENGTH_LINES地址
	};

## 2.5 图像裁剪设置
图像裁剪设置主要用到的结构体为sensor&#95;crop&#95;parms&#95;t和sensor&#95;crop&#95;params&#95;arry, sensor&#95;crop&#95;params&#95;t用于保存裁剪的位置信息。定义在sensor&#95;lib.h中：

	struct sensor_crop_parms_t {
		uint16_t  top_crop;    //距离顶部的距离
		uint16_t  bottom_crop; //距离底部的距离
		uint16_t  left_crop;   //距离左侧的距离
		uint16_t  right_crop;  //距离右侧的距离
	} ;

结构体sensor&#95;lib&#95;crop&#95;params&#95;array

	struct sensor_lib_crop_params_array{
		struct sensor_crop_parms_t *crop_params; //结构体指针
	  	uint16_t size;                           //结构数组长度
	};

## 2.6 分辨率切换设置
枚举类型sensor&#95;res&#95;cfg&#95;type&#95;t说明了进行分辨率切换时所需要进行的操作，在sensor&#95;lib.h中定义如下：

	typedef enum {
		SENSOR_SET_STOP_STREAM,    //停止数据传输
	  	SENSOR_SET_START_STREAM,   //开始数据传输
	  	SENSOR_SET_NEW_RESOLUTION, //设置新的分辨率
	  	SENSOR_SEND_EVENT,         //发送事件
	  	SENSOR_SET_CSIPHY_CFG,     //CSIPHY参数设置
	  	SENSOR_SET_CSID_CFG,       //CSID参数设置
	  	SENSOR_LOAD_CHROMATIX,     //加载chromatix参数
	} sensor_res_cfg_type_t;

切换分辨率的操作顺序：停止数据传输 ----> 设置新的分辨率 ----> CSIPHY参数设置 ----> CSID参数设置 ----> 加载chromatix参数 ----> 发送事件 ----> 开始数据传输。

## 3.1 Camera I2C寄存器设置
I2C寄存器的设置都会用到这两种结构体：msm&#95;camera&#95;i2c&#95;reg&#95;array 和msm&#95;camera&#95;i2c&#95;reg&#95;setting。其定义在media/msm_camera.h中：

	struct msm_camera_i2c_reg_array {
		uint16_t reg_addr; //寄存器地址
		uint16_t reg_data; //寄存器数据
	};

	struct msm_camera_i2c_reg_setting {
		struct msm_camera_i2c_reg_array *reg_setting; //结构体指针
		uint16_t size;                                //结构数组长度
		enum msm_camera_i2c_reg_addr_type addr_type;  //地址类型
		enum msm_camera_i2c_data_type data_type;      //数据类型
		uint16_t  dalay;                              //延时
	};

枚举msm&#95;camera&#95;i2c&#95;reg&#95;addr&#95;type

	enum msm_camera_i2c_reg_addr_type {
		MSM_CAMERA_I2C_BYTE_ADDR = 1, //1字节型
		MSM_CAMERA_I2C_WORD_ADDR, //2字型
		MSM_CAMERA_I2C_3B_ADDR, //3字节型

枚举msm&#95;camera&#95;i2c&#95;data&#95;type

	enum msm_camera_i2c_data_type {
		MSM_CAMERA_I2C_BYTE_DATA = 1,
		MSM_CAMERA_I2C_WORD_DATA,
		MSM_CAMERA_I2C_SET_BYTE_MASK,
		MSM_CAMERA_I2C_UNSET_BYTE_MASK,
		MSM_CAMERA_I2C_SET_WORD_MASK,
		MSM_CAMERA_I2C_UNSET_WORD_MASK,
		MSM_CAMERA_I2C_SET_BYTE_WRITE_MASK_DATA,
	};

### 3.1.1 寄存器初始化设置
表现为在相机启动时一组一次性写入的寄存器。init&#95;reg&#95;array[],res0&#95;reg&#95;array[]和res1&#95;reg&#95;arry[]定义在头文imx230_lib.h中。分别对应excel表RegisterSetting中的全局设置和不同分辨率设置的数据。

-寄存器初始化流程为：<br>
上电 —> 外部时钟输入 —> XCLR关闭 —> 外部时钟寄存器设置 —> 全局寄存器设置 —> Load Setting <br>
-之后寄存器设置根据不同分辨率具有不同的设置：<br>
Load Setting —> 模式设置 —> 输出格式设置 —> 时钟设置 —> Data rate设置 —> 曝光时间设置 —> Gain值设置 —> HDR设置 —> DPC2D设置 —> LSC设值 —> Stats 设置

### 3.1.2 Grouphold on设置 
sensor工作时更新曝光设定需要操作许多寄存器（曝光时间，每帧行数，增益），这些必须在同一帧完成更新。这些寄存器都有双buffer，并具有按组更新的功能。表现为所有相关寄存器一起完成更新。<br>
例：地址0x0104就是寄存器GRP&#95;PARAM&#95;HOLD的地址，当其寄存器的值设为1时，写入的寄存器数据被暂存的buffer寄存器中。

### 3.1.3 Grouphold off设置
当寄存器GRP_PARAM_HOLD的值为0时，所需要寄存器的值会被同时更新，参数的变化会在同一帧生效。

### 3.1.4 启动输出设置
MIPI数据包必须以在SoT(Start of Transmission)和EoT(End of Transmission)之间发送。根据参考手册7.1，在正确的时间设定寄存器MODE&#95;SEL（地址0x0100）为1时，开始进行数据输出。

启动数据输出流程分为两种情况：<br>
情况1：

	在上电之后：
	(1)准备上电序列时序
	(2)PLL锁相环参数设置
	(3)初始化设置
	(4)设置读取模式(起始/结束位置，大小，曝光时间，gain值)
	(5)设置MIPI接口参数
	(6)设置寄存器MODE_SEL的值为1，准备数据输出
	在经过MIPI唤醒时间和初始化时间之后，开始输出第一帧图像数据。

情况2：

	在经过一次数据输出之后：
	(1)设置寄存器MODE_SEL的值为0，进入待命状态
	(2)等待MIPI的FE package
	(3)设置下一次数据输出模式
	(4)设置寄存器MODE_SEL的值为1，准备数据输出
	在经过MIPI唤醒时间和初始化时间之后，开始输出第一帧图像数据。

### 3.1.5 停止输出设置
在正确的时间设定MODE&#95;SEL为0时，结束数据传输。

## 4.1 曝光设置
### 4.1.1 曝光寄存器地址
结构体msm&#95;sensor&#95;exp&#95;gain&#95;info&#95;t

	struct msm_sensor_exp_gain_info_t {
		uint16_t coarse_int_time_addr; //粗曝光时间寄存器地址
		uint16_t global_gain_addr;     //模拟增益寄存器地址
		uint16_t vert_offset;          //曝光行偏置
	};

粗曝光时间单位为lines，用于计算曝光时间，计算关系如下：

	Tsh = Tline * (COARSE_INTEG_TIME + FINE_INTEG_TIME / LINE_LENGTH_PCK);

曝光行偏置用于设定以下关系：

	COARSE_INTEG_TIME ≤ frame_length_lines – vert_offset;

### 4.1.2 AEC参数设置
结构体sensor&#95;aec&#95;data&#95;t

	typedef struct {
		sensor_mode_t op_mode;     //sensor 模式
	  	uint32_t pixels_per_line;  //每一帧每一行多少个pixels
	   	uint32_t lines_per_frame;  //每一帧多少行
	  	uint32_t pclk;             //vt_pixel_clk
	   	uint32_t max_fps;          //最大帧率
	  	float digital_gain;        //数字增益
	  	float stored_digital_gain;
	   	float max_gain;            //最大数字增益
	  	uint32_t max_linecount;    //最大曝光行数
	} sensor_aec_data_t;

枚举sensor&#95;mode&#95;t

	typedef enum {
		SENSOR_MODE_SNAPSHOT,     //快照模式
	  	SENSOR_MODE_RAW_SNAPSHOT, //raw图快照模式
	  	SENSOR_MODE_PREVIEW,      //预览模式
	  	SENSOR_MODE_VIDEO,        //视频录像模式
	  	SENSOR_MODE_VIDEO_HD,     //高清视频录像模式
	  	SENSOR_MODE_HFR_60FPS,    //60帧率HFR模式
	  	SENSOR_MODE_HFR_90FPS,    //90帧率HFR模式
	  	SENSOR_MODE_HFR_120FPS,   //120帧率HFR模式
	  	SENSOR_MODE_HFR_150FPS,   //150帧率HFR模式
	  	SENSOR_MODE_ZSL,          //零秒快拍
	  	SENSOR_MODE_INVALID,      //非法
	} sensor_mode_t;

### 4.1.3 曝光增益gain值设置
AEC算法中模拟增益gain用于曝光计算，实际上必须把gain转换成寄存器gain去设置sensor。以下是imx230的gain转换函数：<br>
模拟增益real&#95;gain值的范围是1至8， 对应到reg&#95;gain的范围为0到448。real&#95;gain与reg&#95;gain的关系为：<br>

	real_gain = 512 / (512 - reg_gain);

结构体sensor&#95;exposure&#95;info&#95;t

	typedef struct {
		uint16_t reg_gain;         //寄存器gain值
		uint16_t line_count;       //曝光行数
		float digital_gain;
		float sensor_real_gain;    //sensor的模拟gain值
		float sensor_digital_gain; //sensor的数字gain值
	} sensor_exposure_info_t;

## 5.1 镜头参数设置
### 5.1.1 结构体sensor&#95;lens&#95;info&#95;t

	typedef struct {
		float focal_length; //焦距
		float pix_size; //像素大小
		float f_number; //光圈
		float total_f_dist;
		float hor_view_angle; //水平视角
		float ver_view_angle; //垂直视角
	} sensor_lens_info_t;

## 6.1 Chromatix参数
每一种分辨率都必须有对应的chromatix库文件。这里对应2种分辨率，设置的是相应的库文件名称。

结构体sensor&#95;lib&#95;chromatix&#95;t

	struct sensor_lib_chromatix_t {
		char *common_chromatix;
	   	char *camera_preview_chromatix;
	   	char *camera_snapshot_chromatix;
	   	char *camcorder_chromatix;
	   	char *liveshot_chromatix;
	};

<font color=ff0000>**key note**</font>：其数据成员都是字符型指针，用来记录不同分辨率下不同模式的库文件名称。

## 7.1 MIPI接收器配置
### 7.1.1 CSI lane参数配置
结构体csi&#95;lane&#95;params&#95;t

	struct csi_lane_params_t {
		uint16_t csi_lane_assign;  //端口映射设置
		uint8_t csi_lane_mask;     //标识哪一个lane被使用
		uint8_t csi_if;            //未使用
		uint8_t csid_core[2];      //csid硬件选择
		uint8_t csi_phy_sel;       //csi-phy设备选择
	};

### 7.1.2 csi&#95;lane&#95;assign 
有时候用户的MIPI lanes可能使用不同与MSM参考设置的端口映射。比如，sensor的lane0连接到MSM的数据lane4等。对于这种情况，csi&#95;lane&#95;assign参数能设置正确的端口映射。csi&#95;lane&#95;assign是一个16bit的值，每位的含义参见下表。lane1用于MIPI时钟，客户不可用它来映射到任何数据lane。

### 7.1.3 csi&#95;lane&#95;mask
用于表示哪些lane被使用，这是一个8位值，每一位含义如下：

	Bit position Represents
	7:5 保留
	4 数据lane4是否使用：
	- 0 ：不
	- 1 ：是
	3 数据lane3是否使用：
	- 0 ：不
	- 1 ：是
	2 数据lane2是否使用：
	- 0 ：不
	- 1 ：是
	1 数据lane1是否使用：(注意：该位必须设置为1)
	- 0 ：不
	- 1 ：是 
	0 数据lane0是否使用：
	- 0 ：不
	- 1 ：是

<font color=ff0000>**举个栗子**</font>：比如0x1F表示4条数据lane和时钟都被使用。

### 7.1.4 csi&#95;if
暂不使用

### 7.1.5 csid&#95;core
设置哪个CSID硬件被该sensor使用。两个并发的sensor不能使用同一个CSID硬件。

### 7.1.6 csi&#95;phy&#95;sel
设置哪个CSI-PHY硬件被该sensor使用。对于每一个sensor来说必须是独一无二的，除非有额外的MIPI桥连接两个sensor到同一个PHY接口上。

## 7.2 虚拟通道设置
CSI2传输的数据包包头部分的起始1byte为数据标志符（Data Identifier, DI），由VC[7:6]（Virtual Channel）和DT[5;0]（Data Type）组成。通过不同的VC和DT值来标志不同的数据流，占2个bit位的虚拟通道VC允许最多4个数据流交叉传输，其取值范围为0~3.

结构体msm&#95;camera&#95;csid&#95;vc&#95;cfg用于保存虚拟通道的设置信息：

	struct msm_camera_csid_vc_cfg {
		uint8_t cid;           //通道号
		uint8_t dt;            //数据类型
		uint8_t decode_format; //解码格式
	};

其数据类型和解码格式的值是宏定义的，其中数据类型的宏定义是根据上述DT表（<font color=ff0000>**！补表补表！**</font>）得来的。如下：

	#define CSI_EMBED_DATA 0x12
	#define CSI_RESERVED_DATA_0 0x13
	#define CSI_YUV422_8 0x1E
	#define CSI_RAW8 0x2A
	#define CSI_RAW10 0x2B
	#define CSI_RAW12 0x2C
	
	#define CSI_DECODE_6BIT	        0
	#define CSI_DECODE_8BIT         1
	#define CSI_DECODE_10BIT        2
	#define CSI_DECODE_DPCM_10_8_10 5

## 7.3 数据流设置

### 7.3.1 结构体sensor&#95;stream&#95;info&#95;t
	
	typedef struct _sensor_stream_info_t {
		uint16_t vc_cfg_size;
	   	struct msm_camera_csid_vc_cfg *vc_cfg; //虚拟通道设置
	  	struct sensor_pix_fmt_info_t *pix_fmt_fourcc; //像素格式
	} sensor_stream_info_t;

### 7.3.2 结构体sensor&#95;stream&#95;info&#95;array&#95;t

	typedef struct _sensor_stream_info_array_t {
		sensor_stream_info_t *sensor_stream_info;
	   	uint16_t size;
	} sensor_stream_info_array_t;

## 7.4 CSID和CSI-PHY参数设置
### 7.4.1 结构体msm&#95;camera&#95;csid&#95;lut&#95;params

	struct msm_camera_csid_lut_params {
		uint8_t num_cid;                       //虚拟通道个数
		struct msm_camera_csid_vc_cfg *vc_cfg; //虚拟通道参数
	};

### 7.4.2 结构体msm&#95;camera&#95;csid&#95;params

	struct msm_camera_csid_params {
		uint8_t lane_cnt; //使用lane的数目
		uint16_t lane_assign;
		uint8_t phy_sel;
		struct msm_camera_csid_lut_params lut_params;
	};

### 7.4.3 结构体msm&#95;camera&#95;csiphy&#95;params

	struct msm_camera_csiphy_params {
		uint8_t lane_cnt;
		uint8_t settle_cnt;
		uint16_t lane_mask;
		uint8_t combo_mode;
		uint8_t csid_core;
	};

<font color=ff0000>**key note**</font>：<br>
1.lane&#95;cnt —— 有多少数据 lane 用于数据传输. 该值必须在 sensor 最大能力范围内,而且sensor 寄存器设置必须与该 lane 数匹配。<br>
2.settle&#95;cnt —— 该值须和 sensor 的特性匹配, 保证 sensor 的 MIPI 传输和 MSM 的 MIPI 接收能同步。

### 7.4.4 结构体msm&#95;camera&#95;csi2&#95;params

	struct msm_camera_csi2_params {
		struct msm_camera_csid_params csid_params;     //CSID参数
		struct msm_camera_csiphy_params csiphy_params; //CSI-PHY参数
	};

## 8.1 结构体sensor&#95;lib&#95;t

	typedef struct {
	  /* sensor slave info */
	  struct camera_sensor_slave_info sensor_slave_info;
	
	  /* sensor output settings */
	  sensor_output_t sensor_output;
	
	  /* sensor output register address */
	  struct sensor_output_reg_addr_t output_reg_addr;
	
	  /* sensor exposure gain register address */
	  struct sensor_exp_gain_info_t exp_gain_info;
	
	  /* sensor aec info */
	  sensor_aec_data_t aec_info;
	
	  /* number of frames to skip after start stream info */
	  unsigned short sensor_num_frame_skip;
	
	  /* number of frames to skip after start HDR stream info */
	  unsigned short sensor_num_HDR_frame_skip;
	
	  /* sensor pipeline delay */
	  unsigned int sensor_max_pipeline_frame_delay;
	
	  /* sensor lens info */
	  sensor_property_t sensor_property;
	
	  /* imaging pixel array size info */
	  sensor_imaging_pixel_array_size pixel_array_size_info;
	
	  /* Sensor color level information */
	  sensor_color_level_info color_level_info;
	
	  /* sensor port info that consists of cid mask and fourcc mapaping */
	  sensor_stream_info_array_t sensor_stream_info_array;
	
	  /* Sensor Settings */
	  struct camera_i2c_reg_setting_array start_settings;
	  struct camera_i2c_reg_setting_array stop_settings;
	  struct camera_i2c_reg_setting_array groupon_settings;
	  struct camera_i2c_reg_setting_array groupoff_settings;
	  struct camera_i2c_reg_setting_array embedded_data_enable_settings;
	  struct camera_i2c_reg_setting_array embedded_data_disable_settings;
	  struct camera_i2c_reg_setting_array aec_enable_settings;
	  struct camera_i2c_reg_setting_array aec_disable_settings;
	  struct camera_i2c_reg_setting_array dualcam_settings;
	
	  /* sensor test pattern info */
	  sensor_test_info test_pattern_info;
	  /* sensor effects info */
	  struct sensor_effect_info effect_info;
	
	  /* Sensor Settings Array */
	  struct sensor_lib_reg_settings_array init_settings_array;
	  struct sensor_lib_reg_settings_array res_settings_array;
	
	  struct sensor_lib_out_info_array     out_info_array;
	  struct sensor_csi_params             csi_params;
	  struct sensor_csid_lut_params_array  csid_lut_params_array;
	  struct sensor_lib_crop_params_array  crop_params_array;
	
	  /* Exposure Info */
	  sensor_exposure_table_t exposure_func_table;
	
	  /* video_hdr mode info*/
	  struct sensor_lib_meta_data_info_array meta_data_out_info_array;
	
	  /* sensor optical black regions */
	  sensor_optical_black_region_t optical_black_region_info;
	
	  /* sensor_capability */
	  sensor_capability_t sensor_capability;
	
	  /* sensor_awb_table_t */
	  sensor_awb_table_t awb_func_table;
	
	  /* Parse RDI stats callback function */
	  sensor_RDI_parser_stats_t parse_RDI_stats;
	
	  /* full size info */
	  sensor_rolloff_config rolloff_config;
	
	  /* analog-digital conversion time */
	  long long adc_readout_time;
	
	  /* number of frames to skip for fast AEC use case */
	  unsigned short sensor_num_fast_aec_frame_skip;
	
	  /* add soft delay for sensor settings like exposure, gain ...*/
	  unsigned char app_delay[SENSOR_DELAY_MAX];
	
	  /* for noise profile calculation
	     Tuning team must update with proper values. */
	  struct sensor_noise_coefficient_t noise_coeff;
	
	  /* Flag to be set if any external library are to be loaded */
	  unsigned char external_library;
	
	  sensorlib_pdaf_apis_t sensorlib_pdaf_api;
	
	  pdaf_lib_t            pdaf_config;
	} sensor_lib_t;