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