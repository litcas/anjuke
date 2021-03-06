## [SEO底部内链]设计方案


版本 | 日期 | 设计 | 备注
:----------- | :-----------: | :-----------: | -----------:
v0.9   | 2014-06-05      | 李超平 | 初稿


## 目标
* 让百度爬虫更快的收录SEO页面

## 要求
### 运营要求
* 运营要求3天曝光完80万数据量
* 每个单页曝光的数据当天是不变的,给爬虫充分的抓取时间

### 性能要求
* API接口不能影响单页的性能(200ms).


## 数据生成规划

### 数据每天更新一次
* 通过job每天更新一次数据.
### 数据分布

### 缓存设计
* 缓存占用预估 
	* 97万数据,全部缓存占用空间100M
	* 数据按城市分布
	
city_id | num | 二手房数量 | 租房数量
:-----------: | :-----------: | :-----------: | :-----------: 
11 | 199211 | 198429 | 782
14 |  65955 | 56741 | 9214
15 |  44225 | 43447 | 778
13 |  42140 | 41341 | 826
12 |  34436 | 33613 | 823
18 |  34075 | 
20 |  31677 | 
16 |  29049 | 
17 |  28182 | 
26 |  26434 | 
19 |  26241 | 
33 |  24065 | 
31 |  22002 | 
22 |  21704 | 
28 |  21252 | 
30 |  20428 | 
21 |  20265 | 
35 |  17797 | 
46 |  17455 | 
27 |  17328 | 
23 |  16854 | 
39 |  14785 |
40 |  14207 | 
25 |  13134 | 
24 |  12904 | 
41 |  12518 | 
36 |  12514 | 
38 |   8623 | 
50 |   8373 | 
34 |   7692 | 
49 |   5538 | 
32 |   5487 | 
42 |   4874 | 
51 |   4189 | 
48 |   3835 | 
62 |   3563 | 
52 |   3386 | 
56 |   3014 | 
44 |   2930 | 
67 |   2374 | 
71 |   2360 | 
70 |   2171 | 
65 |   1961 | 
69 |   1844 | 
68 |   1770 | 
60 |   1567 | 
74 |   1554 | 
43 |   1550 | 
72 |   1519 | 
58 |   1503 | 
47 |   1459 | 
53 |   1432 | 
37 |   1415 | 
54 |   1400 | 
45 |   1316 | 
64 |   1299 | 
66 |   1125 | 
75 |   1047 | 
73 |    942 | 
59 |    915 | 
63 |    906 | 
57 |    833 | 
55 |    651 | 
61 |    635 | 
29 |     38 | 

	* 数据样本:  110000,110000,110000,110000,110000,110000,110000,110000,110000,110000,110000,110000,110000,110000,110000,110000,110000,110000,110000,110000
![msize](msize.png)
![mcount](mcount.png)	

* memcache缓存实现方式
	*  key设计
		1. 根据`city_id`+类型`type_id`获取总数量 `nums`.
		2. 生成key规则: `$city_id-$type_id-{$nums%20}`
	*  value设计
	   * 存放20个id,以`,`隔开

 

### 底部链接API设计

#### 根据id获取基础关联数据 `getSeoFooterLink($id,$type,$city_id)`
数据包含, id,中文名,拼音,上下文数据,相关趋势,感兴趣小区.

* 方法
> function getSeoFooterLink($id,$type,$city_id)
* 参数及返回约定

参数 | 类型 | 描述
:----------- | :-----------: | :-----------:
$id   | integer       | 单页id
$type   | integer      | 类型:(二手房:1|租房:2)
$city_id   | integer   | city_id
return       | array         | 正常:array(或false)

* 调用
> Ershou_Core_Seo_Service_FooterLink::getSeoFooterLink($id);

### 注意事项
* 代码部署在新框架
* 老框架通过API调用
* 新框架可通过service调用


