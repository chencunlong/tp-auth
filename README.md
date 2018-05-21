# tp-auth
The ThinkPHP5.1.x Auth Package
(这是Thinkphp5.1.X版本中的auth扩展)
## 安装
> composer require chencunlong/tp-auth

## 配置
### 公共配置
```
// 新建配置文件 Config/当前项目(如admin)/auth.php 
// 配置参数如下

return [
    'auth_on'           => true,                      // 认证开关
    'auth_type'         => 1,                         // 认证方式，1为实时认证；2为登录认证。
    'auth_group'        => 'auth_group',        // 用户组数据表名
    'auth_group_access' => 'auth_group_access', // 用户-用户组关系表
    'auth_rule'         => 'auth_rule',         // 权限规则表
    'auth_user'         => 'admin',             // 用户信息表
    'auth_user_id_field'=> 'id',                // 用户表ID字段名
];
```

### 导入数据表
> `think_` 为自定义的数据表前缀

```
------------------------------
-- ----------------------------
-- think_auth_rule，规则表，
-- id:主键，name：规则唯一标识, title：规则中文名称 status 状态：为1正常，为0禁用，condition：规则表达式，为空表示存在就验证，不为空表示按照条件验证
-- ----------------------------
 DROP TABLE IF EXISTS `think_auth_rule`;
CREATE TABLE `think_auth_rule` (
    `id` mediumint(8) unsigned NOT NULL AUTO_INCREMENT,
    `name` char(80) NOT NULL DEFAULT '',
    `title` char(20) NOT NULL DEFAULT '',
    `type` tinyint(1) NOT NULL DEFAULT '1',
    `status` tinyint(1) NOT NULL DEFAULT '1',
    `condition` char(100) NOT NULL DEFAULT '',  # 规则附件条件,满足附加条件的规则,才认为是有效的规则
    PRIMARY KEY (`id`),
    UNIQUE KEY `name` (`name`)
) ENGINE=MyISAM  DEFAULT CHARSET=utf8;
-- ----------------------------
-- think_auth_group 用户组表，
-- id：主键， title:用户组中文名称， rules：用户组拥有的规则id， 多个规则","隔开，status 状态：为1正常，为0禁用
-- ----------------------------
 DROP TABLE IF EXISTS `think_auth_group`;
CREATE TABLE `think_auth_group` (
    `id` mediumint(8) unsigned NOT NULL AUTO_INCREMENT,
    `title` char(100) NOT NULL DEFAULT '',
    `status` tinyint(1) NOT NULL DEFAULT '1',
    `rules` char(80) NOT NULL DEFAULT '',
    PRIMARY KEY (`id`)
) ENGINE=MyISAM  DEFAULT CHARSET=utf8;
-- ----------------------------
-- think_auth_group_access 用户组明细表
-- uid:用户id，group_id：用户组id
-- ----------------------------
DROP TABLE IF EXISTS `think_auth_group_access`;
CREATE TABLE `think_auth_group_access` (
    `uid` mediumint(8) unsigned NOT NULL,
    `group_id` mediumint(8) unsigned NOT NULL,
    UNIQUE KEY `uid_group_id` (`uid`,`group_id`),
    KEY `uid` (`uid`),
    KEY `group_id` (`group_id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;
```

## 原理
Auth权限认证是按规则进行认证。
在数据库中我们有 

- 规则表（think_auth_rule） 
- 用户组表(think_auth_group) 
- 用户组明显表（think_auth_group_access）

我们在规则表中定义权限规则， 在用户组表中定义每个用户组有哪些权限规则，在用户组明显表中定义用户所属的用户组。 

下面举例说明：

我们要判断用户是否有显示一个操作权限， 首先定义一个规则， 在规则表中添加一个规则。 然后在用户组表添加一个用户组，定义这个用户组有权限规则（think_auth_group表中rules字段存得时规则ID，多个以逗号隔开）， 然后在用户组明细表定义 UID 为1 的用户 属于刚才这个的这个用户组。 

## 使用
判断权限方法
```
// 引入类库
use think\auth\Auth;

// 获取auth实例
$auth = new Auth();

// 检测权限
if($auth->check(module().'/'.controller().'/'.action(),1)){//规则名称,用户UID
	//有操作权限
}else{
	//没有操作权限
}
		    	
```
