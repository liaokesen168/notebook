# <center><font size=8>Qualcomm camera 差值添加</font></center>
-----------------------------------------------------------------------------------
</br>

## 1.1 hal层：
### 1.1.1 Driver changes reference
**file path**：
vendor/qcom/proprietary/mm-camera/mm-camera2/media-controller/modules/sensors/sensor/libs/

**example**：

	#ifndef __S5K3P3SM_LIB_H__
	#define __S5K3P3SM_LIB_H__
	...
	#define s5k3p3sm_MAX_DGAIN (s5k3p3sm_MAX_DGAIN_REG_VAL / 256)
	+typedef struct {
	+ int32_t width;
	+ int32_t height;
	+} msm_sensor_dimension_t;
	...
	+static msm_sensor_dimension_t scale_size_tbl[] = {
	+ {4160, 3120},//add scale size here, you must keep the ratio the same as oringle sizes
	+};
	…
	/* Sensor Handler */
	static sensor_lib_t sensor_lib_ptr =KBA-170619003303
	Confidential and Proprietary – Qualcomm Technologies, Inc. 2
	MAY CONTAIN U.S. AND INTERNATIONAL EXPORT CONTROLLED INFORMATION
	{
	...
	.adc_readout_time = 0,
	.sensor_num_fast_aec_frame_skip = 0,
	.noise_coeff = {
	.gradient_S = 3.738032e-06,
	.offset_S = 3.651935e-04,
	.gradient_O = 4.499952e-07,
	.offset_O = -2.968624e-04,
	},
	…
	+ /* scale size table count*/
	+ .scale_tbl_cnt = ARRAY_SIZE(scale_size_tbl),
	+ /*function to get scale size tbl */
	+ .get_scale_tbl = get_scale_tbl,
	};
	…
	+static int32_t get_scale_tbl(msm_sensor_dimension_t* tbl)
	+{
	+ int i;
	+ if(sensor_lib_ptr.scale_tbl_cnt == 0)
	+ return -1;
	+ for(i = 0; i < sensor_lib_ptr.scale_tbl_cnt; i++){
	+ tbl[i] = scale_size_tbl[i];
	+ }
	+ return 0;
	+}
	#endif /* __s5k3p3sm_LIB_H__ */

**key note**：<font color=#ff0000>添加的差值要按配置的最大picture size的比例来添加，具体算法可以查看1.1.4中的算法分析。</font>

### 1.1.2 Mm-camera changes reference
**file path**：
vendor/qcom/proprietary/mm-camerasdk/sensor/includes/sensor_lib.h

**example**：

	#ifndef __SENSOR_LIB_H_
	#define __SENSOR_LIB_H_
	…
	sensorlib_pdaf_apis_t sensorlib_pdaf_api;
	pdaf_lib_t pdaf_config;
	… +
	/* scale size tbl count*/
	+ uint8_t scale_tbl_cnt;
	+ /* function to get scale size tbl*/
	+ int32_t (*get_scale_tbl)(msm_sensor_dimension_t *);KBA-170619003303
	Confidential and Proprietary – Qualcomm Technologies, Inc. 3
	MAY CONTAIN U.S. AND INTERNATIONAL EXPORT CONTROLLED INFORMATION
	} sensor_lib_t;
	#endif /* __SENSOR_LIB_H_ */

**file path**：
vendor/qcom/proprietary/mm-camera/mm-camera2/media-controller/modules/sensors/sensor/module/sensor.c

**example**：

	…
	/*==========================================================
	* FUNCTION - sensor_get_capabilities -
	*
	* DESCRIPTION:
	*==========================================================*/
	static int32_t sensor_get_capabilities(void *slib, void *data)
	{
	int32_t rc = SENSOR_SUCCESS;
	sensor_lib_params_t *lib = (sensor_lib_params_t *)slib;
	mct_pipeline_sensor_cap_t *sensor_cap = (mct_pipeline_sensor_cap_t *)data;
	uint32_t i = 0, size = 0, j=0;
	struct sensor_lib_out_info_t *out_info = NULL;
	sensor_dimension_t scale_tbl[MAX_SCALE_SIZES_CNT];
	struct sensor_crop_parms_t *crop_params = NULL;
	uint32_t pix_fmt_fourcc = 0;
	sensor_optical_black_region_t *optical_black_region_ptr = NULL;
	...
	sensor_cap->sensor_physical_size[0] =
	(float)sensor_cap->pixel_array_size.width *
	lib->sensor_lib_ptr->sensor_property.pix_size / 1000;
	sensor_cap->sensor_physical_size[1] =
	(float)sensor_cap->pixel_array_size.height *
	lib->sensor_lib_ptr->sensor_property.pix_size / 1000;
	+/* Fill scale size table*/
	+if(lib->sensor_lib_ptr->get_scale_tbl &&
	+ lib->sensor_lib_ptr->scale_tbl_cnt > 0){
	+ if(lib->sensor_lib_ptr->scale_tbl_cnt > MAX_SCALE_SIZES_CNT){
	+ lib->sensor_lib_ptr->scale_tbl_cnt = MAX_SCALE_SIZES_CNT;
	+ }
	+ rc = lib->sensor_lib_ptr->get_scale_tbl(scale_tbl);KBA-170619003303
	Confidential and Proprietary – Qualcomm Technologies, Inc. 4
	MAY CONTAIN U.S. AND INTERNATIONAL EXPORT CONTROLLED INFORMATION
	+ if(rc){
	+ return rc;
	+ }
	+
	+ sensor_cap->scale_picture_sizes_cnt = lib->sensor_lib_ptr->scale_tbl_cnt;
	+ for(i = 0; i < lib->sensor_lib_ptr->scale_tbl_cnt; i++){
	+ sensor_cap->scale_picture_sizes[i].width = scale_tbl[i].width;
	+ sensor_cap->scale_picture_sizes[i].height= scale_tbl[i].height;
	+ SERR("scale size width=%d, height=%d."
	+ sensor_cap->scale_picture_sizes[i].width, sensor_cap->scale_picture_sizes[i].height);
	+ }
	+}
	/*raw fmts*/
	sensor_cap->supported_raw_fmts_cnt = 0;
	for ( j = 0; j < lib->sensor_lib_ptr->sensor_stream_info_array.size; j++ )
	{
	…

### 1.1.3 Check points/Debug points
**file path**：
vendor/qcom/proprietary/mm-camera/mm-camera2/media-controller/mct/pipeline/mct_pipeline.c

**example**：

	static boolean mct_pipeline_fill_caps_sensor(mct_pipeline_t *pipeline)
	{
	uint32_t i = 0, j = 0, k = 0;
	if (!pipeline) {
	CLOGE(CAM_MCT_MODULE, "Invalid pipeline ptr");
	return FALSE;
	}
	mct_pipeline_cap_t *local_data = &pipeline->query_data;
	cam_capability_t *hal_data = pipeline->query_buf;
	if (!hal_data) {
	CLOGE(CAM_MCT_MODULE, "NULL query_buf!");
	return FALSE;
	}
	hal_data->modes_supported = local_data->sensor_cap.modes_supported;
	hal_data->sensor_mount_angle = local_data->sensor_cap.sensor_mount_angle;
	hal_data->focal_length = local_data->sensor_cap.focal_length;KBA-170619003303
	Confidential and Proprietary – Qualcomm Technologies, Inc. 5
	MAY CONTAIN U.S. AND INTERNATIONAL EXPORT CONTROLLED INFORMATION
	hal_data->hor_view_angle = local_data->sensor_cap.hor_view_angle;
	hal_data->ver_view_angle = local_data->sensor_cap.ver_view_angle;
	hal_data->max_roll_degrees = local_data->sensor_cap.max_roll_degree;
	hal_data->max_pitch_degrees = local_data->sensor_cap.max_pitch_degree;
	hal_data->max_yaw_degrees = local_data->sensor_cap.max_yaw_degree;
	hal_data->pixel_pitch_um = local_data->sensor_cap.pix_size;

	+/* Set scale picture size */
	+size_t scale_picture_sizes_cnt =
	+MIN(local_data->sensor_cap.scale_picture_sizes_cnt, MAX_SCALE_SIZES_CNT);
	+hal_data->scale_picture_sizes_cnt = scale_picture_sizes_cnt;
	+memcpy(hal_data->scale_picture_sizes,
	+local_data->sensor_cap.scale_picture_sizes,
	+scale_picture_sizes_cnt * sizeof (cam_dimension_t));

	/* Set RAW formats */
	…

**file path**：
hardware/qcom/camera/QCamera2/HAL/QCameraParameters.cpp

**example**：

	int32_t QCameraParameters::initDefaultParameters()
	{
	if(initBatchUpdate() < 0 ) {
	…
	// Set supported picture sizes
	if (m_pCapability->picture_sizes_tbl_cnt > 0 &&
	m_pCapability->picture_sizes_tbl_cnt <= MAX_SIZES_CNT) {
	String8 pictureSizeValues = createSizesString(
	m_pCapability->picture_sizes_tbl, m_pCapability->picture_sizes_tbl_cnt);
	set(KEY_SUPPORTED_PICTURE_SIZES, pictureSizeValues.string());
	LOGH("supported pic sizes: %s", pictureSizeValues.string());
	// Set default picture size to the smallest resolution
	CameraParameters::setPictureSize(
	m_pCapability->picture_sizes_tbl[m_pCapability->picture_sizes_tbl_cnt-1].width,
	m_pCapability->picture_sizes_tbl[m_pCapability->picture_sizes_tbl_cnt-1].height);
	} else {
	LOGW("supported picture sizes cnt is 0 or exceeds max!!!");
	}

	+// Need check if scale should be enabled
	+if (m_pCapability->scale_picture_sizes_cnt > 0 &&KBA-170619003303
	+Confidential and Proprietary – Qualcomm Technologies, Inc. 6
	+MAY CONTAIN U.S. AND INTERNATIONAL EXPORT CONTROLLED INFORMATION
	+m_pCapability->scale_picture_sizes_cnt <= MAX_SCALE_SIZES_CNT){
	+//get scale size, enable scaling. And re-set picture size table with scale sizes
	+m_reprocScaleParam.setScaleEnable(true);
	+int rc_s = m_reprocScaleParam.setScaleSizeTbl(
	+m_pCapability->scale_picture_sizes_cnt, m_pCapability->scale_picture_sizes,
	+m_pCapability->picture_sizes_tbl_cnt, m_pCapability->picture_sizes_tbl);
	+if(rc_s == NO_ERROR){
	+cam_dimension_t *totalSizeTbl = m_reprocScaleParam.getTotalSizeTbl();
	+size_t totalSizeCnt = m_reprocScaleParam.getTotalSizeTblCnt();
	+String8 pictureSizeValues = createSizesString(totalSizeTbl, totalSizeCnt);
	+set(KEY_SUPPORTED_PICTURE_SIZES, pictureSizeValues.string());
	+LOGH("scaled supported pic sizes: %s", pictureSizeValues.string());
	+}else{
	+m_reprocScaleParam.setScaleEnable(false);
	+LOGW("reset scaled picture size table failed.");
	+}
	+}else{
	+m_reprocScaleParam.setScaleEnable(false);
	+}
	…


### 1.1.4 Set Scale Size Table算法分析

**file path**：
hardware/qcom/camera/QCamera2/HAL/QCameraParameters.cpp

**function**：setScaleSizeTbl

	int32_t QCameraParameters::QCameraReprocScaleParam::setScaleSizeTbl(size_t scale_cnt,
	        cam_dimension_t *scale_tbl, size_t org_cnt, cam_dimension_t *org_tbl)
	{
	    int32_t rc = NO_ERROR;
	    size_t i;
	    mNeedScaleCnt = 0;
	
	    if(!mScaleEnabled || scale_cnt <=0 || scale_tbl == NULL || org_cnt <=0 || org_tbl == NULL){
	        return BAD_VALUE;    // Do not need scale, so also need not reset picture size table
	    }
	
	    mSensorSizeTblCnt = org_cnt;
	    mSensorSizeTbl = org_tbl;
	    mNeedScaleCnt = checkScaleSizeTable(scale_cnt, scale_tbl, org_cnt, org_tbl);
	    if(mNeedScaleCnt <= 0){
	        LOGE("do not have picture sizes need scaling.");
	        return BAD_VALUE;
	    }
	
	    if(mNeedScaleCnt + org_cnt > MAX_SIZES_CNT){
	        LOGE("picture size list exceed the max count.");
	        return BAD_VALUE;
	    }
	
	    //get the total picture size table
	    mTotalSizeTblCnt = mNeedScaleCnt + org_cnt;
	
	    if (mNeedScaleCnt > MAX_SCALE_SIZES_CNT) {
	        LOGE("Error!! mNeedScaleCnt (%d) is more than MAX_SCALE_SIZES_CNT",
	                 mNeedScaleCnt);
	        return BAD_VALUE;
	    }
	
	    for(i = 0; i < mNeedScaleCnt; i++){
	        mTotalSizeTbl[i].width = mNeedScaledSizeTbl[i].width;
	        mTotalSizeTbl[i].height = mNeedScaledSizeTbl[i].height;
	        LOGH("scale picture size: i =%d, width=%d, height=%d.",
	            i, mTotalSizeTbl[i].width, mTotalSizeTbl[i].height);
	    }
	    for(; i < mTotalSizeTblCnt; i++){
	        mTotalSizeTbl[i].width = org_tbl[i-mNeedScaleCnt].width;
	        mTotalSizeTbl[i].height = org_tbl[i-mNeedScaleCnt].height;
	        LOGH("sensor supportted picture size: i =%d, width=%d, height=%d.",
	            i, mTotalSizeTbl[i].width, mTotalSizeTbl[i].height);
	    }
	    return rc;
	}

**function**：checkScaleSizeTable

	size_t QCameraParameters::QCameraReprocScaleParam::checkScaleSizeTable(size_t scale_cnt,
	        cam_dimension_t *scale_tbl, size_t org_cnt, cam_dimension_t *org_tbl)
	{
	    size_t stbl_cnt = 0;
	    size_t temp_cnt = 0;
	    ssize_t i = 0;
	    if(scale_cnt <=0 || scale_tbl == NULL || org_tbl == NULL || org_cnt <= 0)
	        return stbl_cnt;
	
	    //get validate scale size table. Currently we only support:
	    // 1. upscale. The scale size must larger than max sensor supported size
	    // 2. Scale dimension ratio must be same as the max sensor supported size.
	    temp_cnt = scale_cnt;
	    for (i = (ssize_t)(scale_cnt - 1); i >= 0; i--) {
	        if (scale_tbl[i].width > org_tbl[0].width ||
	                (scale_tbl[i].width == org_tbl[0].width &&
	                    scale_tbl[i].height > org_tbl[0].height)) {
	            //get the smallest scale size
	            break;
	        }
	        temp_cnt--;
	    }
	
	    //check dimension ratio
	    double supported_ratio = (double)org_tbl[0].width / (double)org_tbl[0].height;
	    for (i = 0; i < (ssize_t)temp_cnt; i++) {
	        double cur_ratio = (double)scale_tbl[i].width / (double)scale_tbl[i].height;
	        if (fabs(supported_ratio - cur_ratio) > ASPECT_TOLERANCE) {
	            continue;
	        }
	        mNeedScaledSizeTbl[stbl_cnt].width = scale_tbl[i].width;
	        mNeedScaledSizeTbl[stbl_cnt].height= scale_tbl[i].height;
	        stbl_cnt++;
	    }
	
	    return stbl_cnt;
	}
**key note**：<font color=#ff0000>1. upscale. The scale size must larger than max sensor supported size. 2. Scale dimension ratio must be same as the max sensor supported size.</font>