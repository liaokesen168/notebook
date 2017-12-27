# <center><font size=8>MTK camera info文件节点添加</font></center>
-----------------------------------------------------------------------------------
</br>

### 1.proc file创建function：

	struct proc_dir_entry *proc_create_data(const char *name, umode_t mode,
											struct proc_dir_entry *parent,
											const struct file_operations *proc_fops, void *data)；

### 2.废话少说，直接上fucking code：

	#include <linux/string.h>
	#include <asm/uaccess.h>
	#include <linux/slab.h>
	#include <linux/module.h>
	#include <linux/proc_fs.h>
	#include <linux/device.h>
	#include <linux/miscdevice.h>
	
	static int front_camera_otp_flag;
	static int rear_camera_otp_flag;
	
	static char front_camera_module_name[15];
	static char rear_camera_module_name[15];
	
	static char front_camera_ic_name[15];
	static char rear_camera_ic_name[15];
	
	void lct_camera_set_proc_otpinfo_flag(int index, int flag)
	{
	    if (index == 0) {
	        rear_camera_otp_flag = flag;
	    } else if (index == 1) {
	        front_camera_otp_flag = flag;
	    }
	}
	
	void lct_camera_set_proc_camera_name(int index, char *module_name, char *ic_name)
	{
	    if (index == 0) {
	        strcpy(rear_camera_module_name, module_name);
	        strcpy(rear_camera_ic_name, ic_name);
	    } else if (index == 1) {
	        strcpy(front_camera_module_name, module_name);
	        strcpy(front_camera_ic_name, ic_name);
	    }
	}
	
	static ssize_t camera_proc_camera_info_read(struct file *file, char __user *buff, size_t size, loff_t *ppos)
	{
	    int cnt = 0;
	    char *page = NULL;
	    page = kzalloc(128, GFP_KERNEL);
	
	    cnt = sprintf(page, "[FRONT_CAM_Module]:%s,[FRONT_CAM_IC]:%s,[REAR_CAM_Module]:%s,[REAR_CAM_IC]:%s\n", (strlen(front_camera_module_name) ? front_camera_module_name : "NULL"), (strlen(front_camera_ic_name) ? front_camera_ic_name : "NULL"), (strlen(rear_camera_module_name) ? rear_camera_module_name : "NULL"), (strlen(rear_camera_ic_name) ? rear_camera_ic_name : "NULL"));
	
	    cnt = simple_read_from_buffer(buff, size, ppos, page, cnt);
	
	    kfree(page);
	
	    return cnt;
	}
	
	static ssize_t camera_proc_camera_otp_info_read(struct file *file, char __user *buff, size_t size, loff_t *ppos)
	{
	    int cnt = 0;
	    char *page = NULL;
	    page = kzalloc(128, GFP_KERNEL);
	
	    cnt = sprintf(page, "sub :%d,main :%d\n", front_camera_otp_flag, rear_camera_otp_flag);
	
	    cnt = simple_read_from_buffer(buff, size, ppos, page, cnt);
	
	    kfree(page);
	
	    return cnt;
	}
	
	static const struct file_operations camera_proc_camera_info_fops = {
	    .read = camera_proc_camera_info_read,
	};
	
	static const struct file_operations camera_proc_camera_otp_info_fops = {
	    .read = camera_proc_camera_otp_info_read,
	};
	
	
	static int camera_create_proc_entry(void)
	{
	    struct proc_dir_entry *proc_entry_camera;
	
	    struct proc_dir_entry *proc_entry_camera_otp;
	
	    //creat camera info proc
	    proc_entry_camera = proc_create_data("camera_info", 0444, NULL, &camera_proc_camera_info_fops, NULL);
	    if (IS_ERR_OR_NULL(proc_entry_camera)) {
	        pr_err("%s:%d: add /proc/camera_info error!\n", __func__, __LINE__);
	    }
	
	    //creat camera otp info proc
	    proc_entry_camera_otp = proc_create_data("otpinfo", 0444, NULL, &camera_proc_camera_otp_info_fops, NULL);
	    if (IS_ERR_OR_NULL(proc_entry_camera_otp)) {
	        pr_err("%s:%d: add /proc/camera_otp_info error!\n", __func__, __LINE__);
	    }
	
	    return 0;
	}
	
	static int __init lct_camera_info_init(void)
	{
	    camera_create_proc_entry();
	
	    return 0;
	}
	
	static void __exit lct_camera_info_exit(void)
	{
	}
	
	module_init(lct_camera_info_init);
	module_exit(lct_camera_info_exit);
	
	MODULE_DESCRIPTION("Lct camera info driver");
	MODULE_AUTHOR("Koson Liao <liaokesen@longcheer.com>");
	MODULE_LICENSE("GPL");
