# 事件分析

## 事件分析的意义
&emsp;&emsp;在我们跟踪用户的行为操作记录之后，能做什么？用事件分析去分析用户行为事件，找出行为事件的特征。

&emsp;&emsp;在日常工作中，运营、市场、产品、数据分析师根据实际工作情况而关注不同的事件指标。如：
* 最近三个月来自哪个渠道的用户注册量最高？变化趋势如何？
* 各时段的人均浏览量是分别多少？
* 上周来自北京发生过购买行为的独立用户数，按照年龄段的分布情况？

&emsp;&emsp;诸如此类的指标查看的过程中，行为事件分析起到重要作用。

&emsp;&emsp;根据您的产品特性合理配置追踪事件和属性，可以激发出用户行为事件分析的强大潜能，回答关于变化趋势、维度对比的各种细分问题。

## 事件分析的功能介绍
* 「选择用户群」：圈定一群用户去分析他们行为事件指标的情况。
* 「设定分析条件」：选择要分析的行为事件进行统计
* 「分析结果」：观察单行为的指标变化情况
* 「保存常用」：将经常会用到的事件统计保存起来，以便用于日后进行数据分析。

 
## 事件分析的操作介绍
1. 选择用户群

    a)	下拉选择您要分析的用户群，用户群的目的是为了更精细地分析目标用户

    b)	当然，您也可以进入用户分群的界面创建一个用户群进行分析

![](/assets/user/event-1.png)

2.	设定分析条件

    a)	行为事件：指的是在场景数据设置的用户行为事件进行设置，选择要分析的行为事件，常见的行为事件包括用户的点击、浏览、启动和提交订单等。对应的行为指标如下：

        i.总次数： 在选定时间范围内，该事件触发的次数
        ii.总人数： 在选定时间范围内，触发该事件的独立用户数
        iii.平均次数：在选定时间范围内，每次会话触发该事件的次数
        iv.人均次数：在选定时间范围内，每个用户触发该事件的次数
        v.总时长：在选定时间范围内，用户停留的总时间长度
        vi.平均浏览时长：在选定时间范围内，每次会话停留的时间长度
        vii.人均浏览时长：在选定时间范围内，每个用户停留的时间长度

    b)	分组查看：按维度查看数据，可以进行更加精细化的分析。

    c)	过滤条件设置：

        i.时间的过滤：默认选中最近30天进行事件分析。您也可以根据需求调整时间观察，选择过去7天、10天、15天、30天或任意自定义时间。
        例如，如果选择了1月1日至1月9日，那么系统会筛选出1月1日-9日的访问用户，并在这批用户里统计其行为事件的指标情况。
        ii.应用系统的过滤：默认使用全部应用系统进行分析。
        如果您的产品有IOS，WEB，Android的系统类型，可在下拉选项中选择IOS系统的，那么行为指标的结果只有IOS系统的用户统计。
        iii.添加一个过滤条件：结合您的业务情况，点击可自由添加多个的维度值进行过滤。
        如过滤省份为广东省的用户统计情况。注意，不合理的过滤是会被忽略掉的。
    d)	图表类型：支持表格，饼图，条形图，线图四种图表类型。切换后记得点击查询执行结果。

    ![](/assets/user/event-2.png)

3.	保存常用
将常用的分析事件的条件保存起来，便于常用条件的快捷分析。点击右上角的“保存为常用事件”然后输入事件的名称即可在下次进来时在常用事件的下拉中调出保存过的事件即可。

***

# 应用场景
## 场景1：查看最近7天各时段的人均浏览量是分别多少？

```
 TSA设置：
    1、	选择行为事件：浏览事件的人均次数
    2、	选择分组查看：客户端时间时间
    3、	选择过滤时间：最近7天
    4、	选择图表的类型为趋势图进行查询
```
![](/assets/user/event-3.png)

## 场景2：查看最近7天不同来源渠道的用户数分别多少？
```
 TSA设置：
    1、	选择行为事件：全部事件的总人数
    2、	选择分组查看：来源渠道
    3、	选择过滤时间：最近7天
    4、	选择图表的类型为趋势图进行查询
```
![](/assets/user/event-4.png)

关于用户运营的介绍：
  * [用户分群](user-segmentation.md)
  * [用户行为轨迹](user-segmentation.md#behavior-trace)
  * [路径分析](path-analytics.md)
  * [留存分析](retation-analytics.md)
  * [漏斗分析](funnel-analytics.md)