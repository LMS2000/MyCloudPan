# myCloudPan
简易网盘项目，可以上传下载文件修改删除文件,可以增删改查文件夹，也可以通过分享链接给别人下载文件或者文件夹。

需求是每个用户都有自己独立的存储空间可以存储文件下载文件，创建文件夹，修改文件或文件夹名字，删除文件或者文件夹


根据需求设计了初步的表：

DROP TABLE IF EXISTS `User`;
CREATE TABLE User (
  user_id INT(11) PRIMARY KEY AUTO_INCREMENT  COMMENT 'id',
  username VARCHAR(50)  COMMENT '用户名',
  password VARCHAR(50) COMMENT '密码',
  email VARCHAR(50) COMMENT '邮箱',
  is_enable tinyint(1) DEFAULT 0 COMMENT '是否可用',
  use_quota bigint(11) COMMENT '使用情况',
  quota bigint(11) COMMENT '总容量',
  create_time datetime  COMMENT '创建时间',
  update_time datetime  COMMENT '修改时间'
)ENGINE = InnoDB  COMMENT = '用户表' ;
DROP TABLE IF EXISTS `Folder`;
CREATE TABLE Folder (
  folder_id INT(11) PRIMARY KEY AUTO_INCREMENT COMMENT 'id',
  folder_name VARCHAR(50) NOT NULL COMMENT '文件夹名称',
  parent_folder int(11) NOT NULL COMMENT '父级文件夹id',
  user_id int(11) NOT NULL COMMENT '所属用户',
  size bigint(20) DEFAULT COMMENT '文件夹大小',
  create_time datetime  COMMENT '创建时间',
  update_time datetime  COMMENT '修改时间'
) ENGINE = InnoDB  COMMENT = '文件夹表';
DROP TABLE IF EXISTS `File`;
CREATE TABLE File (
  file_id INT(11) PRIMARY KEY AUTO_INCREMENT  COMMENT 'id' ,
  file_name VARCHAR(50)  COMMENT '文件名',
  file_type VARCHAR(50)  COMMENT '文件类型',
  file_path VARCHAR(255)  COMMENT '文件路径',
  file_url VARCHAR(255) COMMENT '文件url',
  size BIGINT(11)  COMMENT '文件大小',
  create_time datetime  COMMENT '创建时间',
  update_time datetime  COMMENT '修改时间',
  user_id INT(11)  COMMENT '所属用户',
  folder_id INT(11)  COMMENT '所属文件夹'
) ENGINE = InnoDB  COMMENT = '文件表';
关于文件夹表和文件表：
文件夹是虚拟路径，没有实际创建
文件存储的实际路径是根据用户bucket桶跟日期存放路径。主要目的是解决上传重复文件的问题
每个用户都有一个默认的路径根目录(/root)




用户注册：前端传递账号密码给后端，后端首先判断用户名是否存在，然后加密密码，记录用户信息，并且创建用户的根目录(/root) 然后分配配额(都是10G)

用户登录: 前端传递账号密码给后端，后端校验成功后将用户信息记录到session中，这样响应体会携带sessionid给前端，前端自动将sessionid设置到cookie中，要访问其他资源的时候都需要携带sessionid(前端配置请求设置携带cookie)



上传文件：
前端上传multifile文件和当前的路径发送给后端
后端获取文件和路径，再根据session获取用户id，判断文件夹是否存在,然后上传文件插入File表，同时修改文件夹大小，修改用户配额。



下载文件：
前端发送文件名称， 后端去查找文件File表，找到文件，根据file_path获取文件，然后将文件流转换为二进制数据传递前端

创建文件夹：
前端发送要创建的文件夹名和当前的路径给后端，后端查看同一路径下有没有重名文件夹，根据当前路径和文件夹名作为文件夹名插入folder表

修改文件名：
前端发送文件id和新文件名给后端，后端根据id和新文件名修改文件

修改文件夹名：
跟修改文件类似。

删除文件：
前端传递文件id给后端，后端删除文件和记录的同时，还要修改用户配额，修改所属文件夹的大小

删除文件夹：前端传递文件夹id,后端首先根据文件夹id查出文件夹树，然后递归删除每一级文件夹所包含的文件和文件夹记录，最后修改用户配额


下载文件夹： 
前端传递文件夹名（路径） ，后端接收然后得到文件夹的得到文件夹树，然后创建zip临时压缩文件，递归文件夹将文件夹添加进去，同时添加所包含的文件


