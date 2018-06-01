# <center><font size=8>Qualcomm camera 拍摄RAW图卡死</font></center>
-----------------------------------------------------------------------------------
</br>

## 1.1 分析原因 :

### 1.1.1 MSM8937 Camera Feature

<img src="./msm8937_camera_feature.png" width = "" height = "" alt="camera_dws" align=center />

### 1.1.2 sensor信息
型号：s5k3l6 <br>
尺寸：13M <br>

### 1.1.3 问题现象
项目有高低端两个配置，低配为8M高配为13M，低配8M拍摄raw图无卡死情况，但高配13M拍摄raw图出现卡死情况。

### 1.1.4 总结
通过msm8937 camera feature上看到msm8937用的双ISP都只支持8MP，再通过现象分析低配8M拍摄raw图不会卡死，但高配13M拍摄raw图会卡死，应该是13M超过了单ISP的输出。

## 1.2 高通给出解决方案

### 1.2.1 解决方案
allow raw stream to use dual VFE <br>
1. remove raw existing checking <br>
2. config camif raw split info if necessary

### 1.2.2 解决方案patch文件

	From 4a1711f4d5113a899e7e013912d8361fd47bf989 Mon Sep 17 00:00:00 2001
	From: chiz <chiz@qti.qualcomm.com>
	Date: Wed, 6 Apr 2016 15:09:57 +0800
	Subject: [PATCH] mm-camera2: isp2: allow raw stream to use dual VFE
	
	1. remove raw existing checking
	2. config camif raw split info if necessary
	
	Change-Id: I411253c3d8c4a5e1a9c046b5a1c3871911b7650b
	CRs-fixed: 986086
	---
	 .../media-controller/modules/iface2/iface_util.c   | 34 +++++++++++++++-------
	 .../modules/iface2/ispif/iface_ispif.c             |  2 +-
	 .../modules/isp2/module/isp_module.h               |  1 -
	 .../modules/isp2/module/isp_resource.c             | 16 ++--------
	 .../modules/isp2/module/isp_util.c                 |  9 +-----
	 5 files changed, 28 insertions(+), 34 deletions(-)
	
	diff --git a/mm-camera2/media-controller/modules/iface2/iface_util.c b/mm-camera2/media-controller/modules/iface2/iface_util.c
	index a3960bb..e8c50ff 100644
	--- a/mm-camera2/media-controller/modules/iface2/iface_util.c
	+++ b/mm-camera2/media-controller/modules/iface2/iface_util.c
	@@ -3938,6 +3938,20 @@ int iface_util_set_hw_stream_config_pix(iface_t *iface,
	 
	   /* if isp module is not linked there is not pix streams. YUV sensor case. */
	   if (!isp_resource_request.num_pix_stream) {
	+    /* if no pix stream but need split, config split info for camif stream*/
	+    iface_resource_t *session_resource = NULL;
	+    session_resource = &iface_session->session_resource;
	+    session_resource->isp_id_mask = isp_resource_request.isp_id_mask;
	+    session_resource->num_isps = isp_resource_request.num_isps;
	+    if (isp_resource_request.num_isps > 1) {
	+      sensor_out_info_t *sensor_out_info = NULL;
	+      sensor_out_info = &(iface_sink_port->u.sink_port.sensor_out_info);
	+      session_resource->ispif_split_info.is_split = TRUE;
	+      session_resource->ispif_split_info.overlap = 0;
	+      session_resource->ispif_split_info.right_stripe_offset
	+        = (sensor_out_info->request_crop.last_pixel
	+          - sensor_out_info->request_crop.first_pixel + 1) / 2;
	+    }
	     return 0;
	   }
	   /* store the device type i.e hw version */
	@@ -4368,21 +4382,19 @@ int iface_util_reserve_camif_resource(
	   uint32_t        *isp_id)
	 {
	   int rc = 0;
	+  int i = 0;
	 
	   /*reserve VFE resource, save info in session and user streams*/
	   /* Check if any VFE is already used by the session thenchoose the same */
	-  if ((1 << VFE0) & session->session_resource.isp_id_mask) {
	-    *isp_id = VFE0;
	-  } else if ((1 << VFE1) & session->session_resource.isp_id_mask) {
	-    *isp_id = VFE1;
	-  } else {
	-    *isp_id = VFE0;
	+  /* if split required, reserve both */
	+  for (i = VFE_MAX - 1; i >= 0; i--) {
	+    if ((1 << i) & session->session_resource.isp_id_mask)
	+      *isp_id = i;
	+    session->session_resource.used_resource_mask |=
	+      (1 << (16 * *isp_id + IFACE_INTF_PIX));
	+    session->session_resource.isp_id_mask |= 1 << *isp_id;
	+    session->session_resource.used_wm[*isp_id] += 1;
	   }
	-  session->session_resource.used_resource_mask |=
	-    (1 << (16 * *isp_id + IFACE_INTF_PIX));
	-  session->session_resource.isp_id_mask |= 1 << *isp_id;
	-  session->session_resource.used_wm[*isp_id] += 1;
	-
	   CDBG_HIGH("%s: Camif Choose VFE%d resource ", __func__, *isp_id);
	   return rc;
	 }
	diff --git a/mm-camera2/media-controller/modules/iface2/ispif/iface_ispif.c b/mm-camera2/media-controller/modules/iface2/ispif/iface_ispif.c
	index 88dfeba..94eba9e 100644
	--- a/mm-camera2/media-controller/modules/iface2/ispif/iface_ispif.c
	+++ b/mm-camera2/media-controller/modules/iface2/ispif/iface_ispif.c
	@@ -329,7 +329,7 @@ int iface_ispif_get_cfg_params_from_hw_streams(
	 
	       /*dual vfe configuration and only for pix interface*/
	       if (session->session_resource.ispif_split_info.is_split &&
	-          hw_stream->use_pix == 1) {
	+          hw_stream->isp_split_output_info.is_split) {
	         ispif_out_info_t* split_info = &session->session_resource.ispif_split_info;
	         if (isp_id == VFE0) {
	           /* left half image */
	diff --git a/mm-camera2/media-controller/modules/isp2/module/isp_module.h b/mm-camera2/media-controller/modules/isp2/module/isp_module.h
	index b307a21..b373dc6 100755
	--- a/mm-camera2/media-controller/modules/isp2/module/isp_module.h
	+++ b/mm-camera2/media-controller/modules/isp2/module/isp_module.h
	@@ -270,7 +270,6 @@ typedef struct {
	  *  @streams: handle stream dimension and mapping info
	  **/
	 typedef struct{
	-  boolean raw_stream_exists;
	   uint32_t num_streams;
	   isp_stream_port_map_info_t streams[ISP_MAX_STREAMS];
	 } isp_stream_port_map_t;
	diff --git a/mm-camera2/media-controller/modules/isp2/module/isp_resource.c b/mm-camera2/media-controller/modules/isp2/module/isp_resource.c
	index 6015bf4..bfd9a1d 100644
	--- a/mm-camera2/media-controller/modules/isp2/module/isp_resource.c
	+++ b/mm-camera2/media-controller/modules/isp2/module/isp_resource.c
	@@ -956,7 +956,7 @@ static boolean isp_resource_reserve_isp(isp_resource_t *isp_resource,
	   /*enforce dual vfe enable by set property*/
	   property_get("persist.camera.isp.dualisp", value, "0");
	   dual_vfe_enabled = atoi(value);
	-  if (dual_vfe_enabled == 1 && !session_param->stream_port_map.raw_stream_exists) {
	+  if (dual_vfe_enabled == 1) {
	     ISP_ERR("dual_dbg Dual VFE enforced");
	   } else {
	     for (i = 0; i < isp_resource->num_isp; i++) {
	@@ -966,8 +966,7 @@ static boolean isp_resource_reserve_isp(isp_resource_t *isp_resource,
	       if ((res_alloc->state == ISP_RESOURCE_FREE) &&
	           (res_info->isp_pipeline->max_width >= in_w) &&
	           (res_info->isp_pipeline->max_height >= in_h) &&
	-          ((res_info->isp_pipeline->max_nominal_pix_clk >= pix_clk)||
	-          session_param->stream_port_map.raw_stream_exists)) {
	+          (res_info->isp_pipeline->max_nominal_pix_clk >= pix_clk)) {
	         hw_ids[(*num_isp)++] = hw_id;
	         if(dual_vfe_enabled == 1) {
	           ISP_ERR("dual_dbg Can not enforce Dual VFE, Raw stream present");
	@@ -982,15 +981,6 @@ static boolean isp_resource_reserve_isp(isp_resource_t *isp_resource,
	   ISP_DBG("dual_dbg single vfe reserved %d,request op pix clk %d\n",
	     reserved, pix_clk);
	 
	-  if (!reserved && session_param->stream_port_map.raw_stream_exists
	-      && (session_param->stream_port_map.num_streams == 1)){
	-    hw_id = isp_resource->sorted_hw_ids[isp_resource->num_isp - 1];
	-    hw_ids[(*num_isp)++] = hw_id;
	-    res_alloc->state = ISP_RESOURCE_RESERVED;
	-    res_alloc->session_id = session_param->session_id;
	-    reserved = TRUE;
	-  }
	-
	   /* Try to reserve 2 VFEs in case single VFE nominal clock reserve fails*/
	   if (!reserved) {
	     in_w = ((in_w / 2) * 6) / 5; /* width / 2 + 20% */
	@@ -2594,7 +2584,7 @@ static boolean isp_stream_resource_allocate(isp_session_param_t *session_param,
	   /*enforce dual vfe enable by set property*/
	   property_get("persist.camera.isp.dualisp", value, "0");
	   dual_vfe_enabled = atoi(value);
	-  if (dual_vfe_enabled == 1 && !session_param->stream_port_map.raw_stream_exists) {
	+  if (dual_vfe_enabled == 1) {
	     session_param->num_isp = 2;
	     ISP_ERR("enforce DUAL VFE mode! session_parm num isp = 2");
	   } else if(dual_vfe_enabled == 1) {
	diff --git a/mm-camera2/media-controller/modules/isp2/module/isp_util.c b/mm-camera2/media-controller/modules/isp2/module/isp_util.c
	index 3d758c4..0478103 100644
	--- a/mm-camera2/media-controller/modules/isp2/module/isp_util.c
	+++ b/mm-camera2/media-controller/modules/isp2/module/isp_util.c
	@@ -6676,7 +6676,6 @@ boolean isp_util_handle_stream_info(mct_module_t *module,
	      }
	   }
	 
	-  session_param->stream_port_map.raw_stream_exists = FALSE;
	   if (streams_desc->num_streams > ISP_MAX_STREAMS) {
	       ISP_ERR("failed: Max supported pix supported streams %d, current %d",
	          ISP_MAX_STREAMS, streams_desc->num_streams);
	@@ -6684,13 +6683,7 @@ boolean isp_util_handle_stream_info(mct_module_t *module,
	   }
	   /* Filling number of streams in session info */
	   session_param->stream_port_map.num_streams = streams_desc->num_streams;
	-  /* check if we have a raw stream, we need this info to veto usage of dual VFE*/
	-  for(i = 0; i < streams_desc->num_streams; i++) {
	-      if(streams_desc->type[i] == CAM_STREAM_TYPE_RAW){
	-         session_param->stream_port_map.raw_stream_exists = TRUE;
	-         break;
	-      }
	-  }
	+
	   ret = isp_resource_allocate(module, &isp->isp_resource,
	      session_param->session_id, &session_param->num_isp,
	      session_param->hw_id, 0);
	-- 
	1.8.2.1

