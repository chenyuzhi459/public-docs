# 维度管理
![](/assets/dimension-management/1-1.png)

在SUGO-TSA中，我们把导入的数据字段称为“维度”。例如用户ID、页面名称、商品金额、订单编号，这些字段都是维度。

在维度管理功能中，我们对维度进行创建与维护工作，还可以通过权限管理将维度分配给不同的用户组。

## 创建维度  <span id = "anchor-1"></span>
![](/assets/dimension-management/2-1.png)

在典型的使用场景中，维度是在导入数据文件或接入SDK时自动创建的。
自主创建维度有四种类型：普通维度、计算维度、分组维度、类型转换维度。

### 普通维度
![](/assets/dimension-management/2-2.png)

主要用于创建可视化埋点的上报数据维度，其他情况使用较少。

### 计算维度
![](/assets/dimension-management/2-3.png)

对某个维度的数值进行固定的数学公式计算。仅支持数值型维度计算，且只能在单个维度与常数之间进行计算。

### 分组维度
![](/assets/dimension-management/2-4.png)

当维度值过于松散，或分析时需要将维度值进行分组时，就会用到分组维度。例如，上图中我们将各个省份按照地域进行分组，创建一个别名为“地域”的新维度。

![](/assets/dimension-management/2-4-2.png)

当被分组的维度是数值格式时，分组设置将变为设置数值范围。其中范围区间“左闭右开”，如图中所示，1080P分组的屏幕宽度范围为（720，1080]。

![](/assets/dimension-management/2-4-3.png)

当被分组的维度是时间格式时，如上图将提供时间范围选择进行分组。



### 类型转换维度
![](/assets/dimension-management/2-5.png)
有些时候，我们需要将现有某个字符串维度的数据转换成数值格式，以便进行分段分析。此时可以创建一个类型转换维度，将指定的维度转换格式后，存入新维度。

注意：请检查确认原维度的内容，不能被转换为数值的内容将无法写入新的维度。

***

## 维度标签  <span id = "anchor-2"></span>
![](/assets/dimension-management/3-1.png)

当维度数量众多时，为了使用方便，我们可以将维度赋予标签，进行分类管理。

标签功能支持批量添加/移除标签。将维度进行标签分类之后，可以更快捷的在多维分析和维度管理中查询自己需要的维度。

***

## 维度权限  <span id = "anchor-3"></span>
![](/assets/dimension-management/3-2.png)

在日常运营管理中，我们的数据维度可能并非向全体人员进行开放，此时就需要通过维度权限进行管理分配，让部分维度只对部分用户组可见。

我们可以对每一个维度单独进行用户组的授权管理，并提供有批量管理功能。

***

## 排序与隐藏  <span id = "anchor-4"></span> 
![](/assets/dimension-management/3-3.png)

顾名思义，排序和隐藏功能可以调整维度的展示顺序，及是否展示。

顺序和展示会影响多维分析功能中的维度项。