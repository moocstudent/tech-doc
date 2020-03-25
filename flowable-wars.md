- flowable-idm.war : 该服务主要集成了用户管理,权限管理,组管理,单点登录功能,是modeler等依赖的一个基础用户服务.
- flowable-modeler.war : 核心的业务绘制模块,提供了一个Web化的编辑器,可以在线编辑业务流程,绘制业务表单,编辑决策表,发布应用程序,编写Case模型的功能.
- flowable-admin.war : 管理端的程序,可以查询流程引擎,CMMN引擎,App引擎,表单引擎,DMN引擎,Content引擎的相关信息,并且提供一定的管理能力.
- flowable-task.war : 任务管理程序,提供任务,流程,Case的启动停止能力,并且可以编辑任务的操作步骤.
- flowable-rest.war : 该功能主要提供对flowable的rest接口,rest通过统一的restful接口来服务,主要有部署管理,任务管理,流程管理等功能,可以不通过JAVA API来调用相关接口.

以上的war包都需要通过idm包提供的用户单点登录服务,所以必须启动idm服务.

@ukyo
fork:https://blog.csdn.net/houyj1986/article/details/85240266

启动顺序:
(1)启动flowable-idm.war 8080端口
(2)启动flowable-modeler.war 8888端口
(3)启动flowable-admin.war 9988端口
(4)启动flowable-task.war 9999端口
(5)启动flowable-rest.war 通过指定 --server.port=80 避免与idm冲突


---
flowable-idm登录
启动完成后
通过 http://localhost:8080/flowable-idm 进行登录
默认账户admin/test

Flowable-IDM功能
- 提供用户管理功能: 可以添加用户,编辑用户,删除用户和密码修改功能.
- 提供用户分组功能: 提供用户组的创建,用户组的删除,添加删除用户到组功能,方便统一管理用户权限,是一个简化版的角色处理.
- 提供权限管理功能: 权限简单分为idm/admin/modeler/workflow(task?)/rest的访问权限控制,通过配置用户和组来管理用户的访问权限.
- 提供单点登录管理: modeler、admin等用户需要登录到idm完成用户的验证.

ACT_ID_USER表中如果没有用户,启动会添加默认的管理用户.

功能导航:
# 用户 : 可以配置用户信息,添加和删除用户,修改用户详细,password,delete等 #
# 组 : 用户组管理,用户组列表,添加用户组,可以理解为角色,选中的用户组信息,edit,delete GROUP等 #
# 权限: 用户权限管理,限定访问应用的组,用户等 #

Flowable-IDM特性
IDM除以上功能外还有以下特性:
- IDM是在6.0已经剥离,如果通过集成加入Flowable的流程功能的话不用必须加入IDM.
- IDM的相关表以ACT_ID开头如ACT_ID_USER、ACT_ID_GROUP
- Rest-Api权限需要flowable.rest.app.authentication-demo设置为verify-privilege,默认值也是该值,
如果没权限,则返回403无权限.
- 如果不用自带的用户体系,可以设置flowable.idm.Ldap.enabled=true使用Ldap server来设置用户鉴权,
不过只是用户和组,权限配置还是在Flowable的表中,所以如果使用LDAP鉴权,那么确保Ldap的用户权限在Flowable中
正确配置.
- 如果使用LDAP,那么第一次启动会给配置的flowable.common.app.idm-admin.user用户所有的默认的4个权限,
防止没有一个用户能够登录系统.
- 官方提供的给予Apache Directory Server的配置例子:
------------------------------------------------
#
# LDAP
#
flowable.idm.ldap.enabled=true
flowable.idm.ldap.server=ldap://localhost
flowable.idm.ldap.port=10389
flowable.idm.ldap.user=uid=admin, ou=system
flowable.idm.ldap.password=secret
flowable.idm.ldap.base-dn=o=flowable
flowable.idm.ldap.query.user-by-id=(&(objectClass=inetOrgPerson)(uid={0}))
flowable.idm.ldap.query.user-by-full-name-like=(&(objectClass=inetOrgPerson)(|({0}=*{1}*)({2}=*{3}*)))
flowable.idm.ldap.query.all-users=(objectClass=inetOrgPerson)
flowable.idm.ldap.query.groups-for-user=(&(objectClass=groupOfUniqueNames)(uniqueMember={0}))
flowable.idm.ldap.query.all-groups=(objectClass=groupOfUniqueNames)
flowable.idm.ldap.attribute.user-id=uid
flowable.idm.ldap.attribute.first-name=cn
flowable.idm.ldap.attribute.last-name=sn
flowable.idm.ldap.attribute.group-id=cn
flowable.idm.ldap.attribute.group-name=cn
flowable.idm.ldap.cache.group-size=10000
flowable.idm.ldap.cache.group-expiration=180000
------------------------------------------------
- IDM的常用参数配置
--------------------------------------------------------------------------------------------
										IDM UI 应用配置
--------------------------------------------------------------------------------------------
			属性名           |           老版本名称        |    默认值   |        描述        |
flowable.idm.app.bootstrap   |            xx              |    TRUE    |    是否启动idm     |
flowable.idm.app.rest-enabled|            xx              |    TRUE    |是否启动REST API    |
flowable.idm.app.admin.email |            xx              |     -      |    admin邮箱       |
flowable.idm.app.admin.first-name|        xx              |     -      |    admin姓         |
flowable.idm.app.admin.last-name |        xx              |     -      |    admin名         |
flowable.idm.app.admin.password  |        xx              |     -      |    admin密码       |
flowable.idm.app.admin.user-id   |        xx              |     -      |    admin的id       |
flowable.idm.app.security.remember-me-key|xx              |  testKey   |Spring Security用来hash密码的值,注意不要使用默认值 |
flowable.idm.app.security.user-validity-period|xx         |   30000    |   用户再次校验周期  |
flowable.idm.app.security.cookie.domain|  xx              |     -      |    cookie的域      |
flowable.idm.app.security.cookie.max-age|  xx             |   2678400  |  cookie的最长时间(秒)默认31天 |
flowable.idm.app.security.cookie.refresh-age| xx          |    86400   |  cookie的刷新周期,默认一天|

---

Flowable-Modeler功能
- 提供可视化编辑器,编辑BPMN流程,编辑CASE模型,编辑Form表单,编辑App应用,编辑决策表.
- 提供可视化参数配置:每个流程可以配置详细的参数设置,按照流程对应的规范来设计.
- 提供导入导出功能:方便将流程结果导入到其它应用程序.

功能导航:
# 流程: 绘制BPMN流程 #
# Case模型: 绘制CMMN模型 #
# 表单: 绘制Form表单 #
# 决策表: 绘制DMN的表 #
# 应用程序: 组合生成App #
# 创建流程/导入流程: 创建BPMN流程和导入历史流程 #
Flowable-Modeler没有使用act_re_modeler而使用act_de_modeler数据库表
TODO 案例模型,表单,决策表,应用程序

由于flowable也是在更新,而如果将画图的模块单独拿到自己系统中,在flowable更新后,需要再次更改
如果更新影响不大,那么可以直接将flowable源码进行修改对应.

在将flowable-modeler中的流程分配用户进行指定.在保存右侧的对号验证模型就会验证通过.
https://blog.csdn.net/houyj1986/article/details/85494233





