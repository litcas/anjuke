# 房源前后台拆分概要设计

基本方案参考[房源数据前后台拆分](http://git.corp.anjuke.com/_sketch/sketch/issues/34)

本次项目只涉及房源基本表和扩展表(ajk\_propertys,ajk\_propertysale)


## 总体流程

![](http://pic1.ajkimg.com/display/origin/76454de152e3646623a36af45a982690.jpg)

## 前后台接口

客户线在插入/修改/删除ajk\_propertys和ajk\_propertysale两个表的数据时，发送消息到消息中间件，通知客户线数据发生变更。

api

```
http://nydus.dev.anjuke.com/publish?tunnel=esf_property_event
```

消息格式

```javascript
{
    "city_id":11,
    "pro_id":1234,
    "type": 1,
    "update_time":'xxxx-xx-xx',
    "flag":1
}
```

其中 

* type 用来区分房源的类型: 1表示经纪人房源，2表示个人房源，目前只有type=1
* flag 的值分别为：1表示新增，2表示修改，3表示删除

## 消息读取job

由用户端负责，实时读取房源变更的消息，再从目前已有的房源api获取房源信息，写入用户端房源表 (后续会再发送消息给用户端下游的应用）。

job可以按城市id和房源类型来分成多个 job，提高吞吐量

## 用户端房源表设计

这里主要是分表/拆表方案，具体表的字段设计暂时不涉及

### 房源id

当前的不同类型的房源是在同一个solr，为了使房源数据能够共存，只能通过id来区分的，0-10亿经纪人和抓取房源，10-20亿是个人房源，大于20亿是用户端抓取房源。长久来看id的增长肯定会遇到问题，因此打算重新设计房源id。

将不同类型的房源统一，前台页面展示使用新的房源id。房源 id 规则采用类型+id的方式组合。

* 经纪人房源     A+id
* 个人房源       B+id
* 客户线抓取房源  C+id
* 用户线抓取房源  D+id

实际存储的时候每个类型使用单独的表，包含类型和原始id两个字段，原始id作为表的主键

solr 仍然提供按经纪人房源id搜索的方式。solr存储时，主键使用上面描述的字母+id的方式，用户和客户相关的接口(例如soj, ppc)仍然传递经纪人房源的id。同时用户页面的房源单页使用新的url

影响范围
* 产品和SEO权重
* BI数据分析
* 移动app
* ppc

ppc的接口和soj会继续传递经纪人房源id，需要确认是否会有其他的影响

现有的房源单页url仍然接收经纪人房源id，之后会301跳转到新的房源单页url，对经纪人房源链接的使用来说应该是没有太大影响。

如果有依赖房源单页access log的数据分析，会有较大的影响


### 分表

当前经纪人房源表的数据量如下

* 北京 61万
* 上海 174万
* city_id<=40  372万
* city_id>40   435万 (其中厦门170万，也存在单个城市<2000套房源)

前台页面上显示的在线房源

* 北京 8.5万
* 上海 53万
* 深圳 10万
* 成都 11万
* 厦门 5万

按现在的情况考虑，单个城市超过1000万房源的情况近几年内应该不太会发生，同时考虑以后小区，经纪人等信息的前后台拆分，因此逻辑上每个城市一个单独的db。将来小区信息和经纪人信息也按城市，和房源一样。

物理上，多个城市可能使用同一个mysql实例和database，使用相同的表名，数据会在同一个表里面。与目前经纪人房源表的分表情况类似。

代码结构上通过Dao层来屏蔽具体的细节。

房源表分为基本表和扩展表

房源基本表只存储房源本身的属性字段，例如broker\_id, comm\_id等等，跟小区相关的字段不在冗余存储(areacode, comm\_name...)

扩展表存储text,varchar以及很少用到的字段

具体的表设计在之后的详细设计中说明

