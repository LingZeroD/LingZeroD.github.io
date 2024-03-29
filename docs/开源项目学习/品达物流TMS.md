## 项目概述

### 简介

运输管理系统，是对运输作业从运力资源准备到最终货物抵达目的地的全流程管理。TMS系统适用于运输公司、各企业下面的运输队等，它主要包括订单管理、配载作业、调度分配、行车管理、GPS车辆定位系统、车辆管理、线路管理、车次管理、人员管理、数据报表、基本信息维护等模块。该系统对车辆、驾驶员、线路等进行全面详细的统计考核，能大大提高运作效率，降低运输成本，使公司能够在激烈的市场竞争中处于领先地位。



### 软件架构

#### 系统架构

![image-20200605150332201](https://picture.lingzero.cn/img/202209072131655.png)

#### 技术架构

![image-20200803090944749](品达物流TMS.assets/image-20200803090944749.png)

### 整体业务流程

寄件人下单到最终收件人签收的整个流程：

![image-20200605152114503](品达物流TMS.assets/image-20200605152114503.png)

### 开发过程

通过通用权限系统进行菜单的配置、权限的配置、用户的配置、认证和鉴权等。

客户端App是通过注册登录服务来完成C端用户的注册和登录功能。

快递员端App是通过文件服务来完成附件的上传操