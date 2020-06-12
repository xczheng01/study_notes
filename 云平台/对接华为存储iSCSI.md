

>本文说明：
>
>openstack 需要对接 两个磁盘阵列， 华为存储为 iSCSI 组网模式.

### 一、cinder iSCSI Huawei 对接

***

* 拷贝驱动到cinder_voluem

```shell
docker cp Pike cinder_volue:/
```

* /etc/kolla/cinder-volume/cinder.conf 
* [配置多路径](https://docs.openstack.org/cinder/latest/admin/blockstorage-multi-backend.html)

```shell
[DEFAULT]
...
use_multipath_for_image_xfer = True
enforce_multipath_for_image_xfer = True
enabled_backends=huawei_backend_1,huawei_backend_2

...

[huawei_backend_1]
volume_driver = cinder.volume.drivers.huawei.huawei_driver.HuaweiISCSIDriver
cinder_huawei_conf_file = /etc/cinder/cinder_huawei_conf_1.xml
volume_backend_name = huawei_backend_1

[huawei_backend_2]
volume_driver = cinder.volume.drivers.huawei.huawei_driver.HuaweiISCSIDriver
cinder_huawei_conf_file = /etc/cinder/cinder_huawei_conf_2.xml
volume_backend_name = huawei_backend_2
```

* cinder_volume容器中：/etc/cinder/cinder_huawei_conf.xml
* 注意权限： `cinder:cinder cinder_huawei_conf_1.xml `

```xml
<?xml version='1.0' encoding='UTF-8'?>
<config>
  <Storage>
    <Product>V5</Product>
    <Protocol>iSCSI</Protocol>
    <RestURL>https://17.60.4.2:8088/deviceManager/rest/;https://17.60.4.3:8088/deviceManager/rest/</RestURL>
    <UserName>!$$$RlNfYWRtaW4=</UserName>
    <UserPassword>!$$$SHVhd2VpMTIjJA==</UserPassword>
  </Storage>
  <LUN>
    <StoragePool>MEP</StoragePool>
  </LUN>
  <iSCSI>
    <Initiator HostName="*"  TargetIP="172.28.32.1 172.28.32.2 172.28.32.5 172.28.32.6"/>
  </iSCSI>
</config>

```



### 二、创建volumetype

***

* 创建卷类型，<name>表示自定义卷类型名称。

```shell
cinder type-create <name>
```

```shell
cinde type-create mec_1
```

* 设置Thick卷类型，<vtype>表示步骤1的卷类型名称。

```shell
 cinder type-key mec_1 set capabilities:thick_provisioning_support='<is> true'
```

```shell
cinder type-key mec_1 set volume_backend_name=huawei_backend
```

* Openstack cinder 查看

```shell
cinder extra-specs-list
```



### 三、⚠️注意： cinder_volume  搭建后一直重启

***

```shell
1、openstack 部署完，cinder_volume 会不断重启。

2、需要临时增加一个配置：
路径：/etc/kolla/cinder-volume/cinder.conf 

3、追加如下内容：
[DEFAULT]
...
use_multipath_for_image_xfer = True
enforce_multipath_for_image_xfer = True
enabled_backends=huawei_backend_1

[huawei_backend_1]

4、重启容器
docker restart cinder_volume

5、根据具体的存储，按上面的步骤配置
```



* Troubleshooting

对接好存储后，创建虚机失败，在nova-compute日志中报如下错误，排查是由于没有启动multipathd容器

![image-20200609180107369](C:\Users\xczheng\AppData\Roaming\Typora\typora-user-images\image-20200609180107369.png)



打开 /etc/kolla/globals.yml ,将enable_multipathd改为yes

![image-20200609180504410](C:\Users\xczheng\AppData\Roaming\Typora\typora-user-images\image-20200609180504410.png)



上传multipathd容器到仓库

