---
title:  业务型研发项目总结
showOnHome: false
...

此为业务型研发项目踩坑总结


## 接任前注意事项

-  确认是否胜任
	-  如果同时兼任项目经理， 后端架构，还需要搞定客户大概率 搞不定，特别是需求和研发同时进行的情况。这
	-  BA是否经验足够，如果BA经验不足够，风险更大
- 严控非自己团队项目成员加入，通过面试来保证团队成员适配性
	- 技术能力达标
	- 态度达标
	- 对加班、驻场等态度
- 如果是迭代式的研发， 逐步上线。 需要考虑每次增量发版的问题。






### 后端常识性知识







## 附录

#### 数据字典
 - 避免数据字典的滥用
 - 可以做一个字典来说明数据库表中的字段的值和代表的含义。 比如状态字段。 （需要想个方式来同步不同环境的这个字典的数据）这种方式还有个好处就是可以关联查询统计。


#### nacos 使用规范
https://blog.csdn.net/czpandy/article/details/120514268

#### 增量式发布需要考虑的问题

##### 关于非业务数据同步的问题
比如菜单，新增了菜单如何同步到客户内网或者生产环境。 如果是通过ID关联， ID 可能会不一致， 最好通过唯一的code来关联。