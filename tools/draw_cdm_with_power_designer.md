# 使用Power Designer画CDM

## **概念数据模型概述**

数据模型是现实世界中数据特征的抽象。数据模型应该满足三个方面的要求：

1）能够比较真实地模拟现实世界

2）容易为人所理解

3）便于计算机实现

概念数据模型也称信息模型，它以实体－联系(Entity-RelationShip,简称E-R)理论为基础，并对这一理论进行了扩充。它从用户的观点出发对信息进行建模，主要用于[数据库](http://lib.csdn.net/base/mysql)的概念级设计。

通常人们先将现实世界抽象为概念世界，然后再将概念世界转为机器世界。换句话说，就是先将现实世界中的客观对象抽象为实体(Entity)和联系 (Relationship),它并不依赖于具体的计算机系统或某个DBMS系统，这种模型就是我们所说的CDM;然后再将CDM转换为计算机上某个 DBMS所支持的数据模型，这样的模型就是物理数据模型,即PDM。

CDM是一组严格定义的模型元素的集合，这些模型元素精确地描述了系统的静态特性、动态特性以及完整性约束条件等，其中包括了[数据结构](http://lib.csdn.net/base/datastructure)、数据操作和完整性约束三部分。

1）数据结构表达为实体和属性;

2）数据操作表达为实体中的记录的插入、删除、修改、查询等操作;

3）完整性约束表达为数据的自身完整性约束（如数据类型、检查、规则等）和数据间的参照完整性约束（如联系、继承联系等）;



## **实体、属性及标识符的定义**

实体（Entity），也称为实例，对应现实世界中可区别于其他对象的“事件”或“事物”。例如，学校中的每个学生，医院中的每个手术。

每个实体都有用来描述实体特征的一组性质，称之为属性，一个实体由若干个属性来描述。如学生实体可由学号、姓名、性别、出生年月、所在系别、入学年份等属性组成。

实体集（Entity Set）是具体相同类型及相同性质实体的集合。例如学校所有学生的集合可定义为“学生”实体集，“学生”实体集中的每个实体均具有学号、姓名、性别、出生年月、所在系别、入学年份等性质。

实体类型（Entity Type）是实体集中每个实体所具有的共同性质的集合，例如“患者”实体类型为：患者｛门诊号，姓名，性别，年龄，身份证号.............｝。实体是实体类型的一个实例，在含义明确的情况下，实体、实体类型通常互换使用。

实体类型中的每个实体包含唯一标识它的一个或一组属性，这些属性称为实体类型的标识符（Identifier），如“学号”是学生实体类型的标识符，“姓名”、“出生日期”、“信址”共同组成“公民”实体类型的标识符。

有些实体类型可以有几组属性充当标识符，选定其中一组属性作为实体类型的主标识符，其他的作为次标识符。

##  **实体、属性及标识符的表达**

![image-20190304110727411](/Users/nacht/Library/Application Support/typora-user-images/image-20190304110727411.png)

## 概念数据模型以及实体、属性创建

### **新建概念数据模型**



1）选择File-->New,弹出如图所示对话框，选择CDM模型（即概念数据模型）建立模型。

![image-20190304143105198](/Users/nacht/Library/Application Support/typora-user-images/image-20190304143105198.png)

2）完成概念数据模型的创建。以下图示，对当前的工作空间进行简单介绍。

![image-20190304143126959](/Users/nacht/Library/Application Support/typora-user-images/image-20190304143126959.png)

3）选择新增的CDM模型，右击，在弹出的菜单中选择“Properties”属性项，弹出如图所示对话框。在“General”标签里可以输入所建模型 的名称、代码、描述、创建者、版本以及默认的图表等等信息。在“Notes”标签里可以输入相关描述及说明信息。当然再有更多的标签，可以点击 "More>>"按钮，这里就不再进行详细解释。

![image-20190304143145659](/Users/nacht/Library/Application Support/typora-user-images/image-20190304143145659.png)



### **创建新实体**

1）在CDM的图形窗口中，单击工具选项版上的Entity工具，再单击图形窗口的空白处，在单击的位置就出现一个实体符号。点击Pointer工具或右击鼠标，释放Entitiy工具。如图所示

![image-20190304143553079](/Users/nacht/Library/Application Support/typora-user-images/image-20190304143553079.png)

2）双击刚创建的实体符号，打开下列图标窗口，在此窗口“General”标签中可以输入实体的名称、代码、描述等信息。

![image-20190304143613118](/Users/nacht/Library/Application Support/typora-user-images/image-20190304143613118.png)



### 添加实体属性

1）在上述窗口的“Attribute”选项标签上可以添加属性，如下图所示。

![image-20190304143853592](/Users/nacht/Library/Application Support/typora-user-images/image-20190304143853592.png)

> 注意：数据项中的“添加属性”和“重用已有数据项”这两项功能与模型中Data Item的Unique code 和Allow reuse选项有关。P列表示该属性是否为主标识符;D列表示该属性是否在图形窗口中显示;M列表示该属性是否为强制的，即该列是否为空值。如果一个实体属性为强制的，那么， 这个属性在每条记录中都必须被赋值，不能为空。

2）在上图所示窗口中，点击插入属性按钮，弹出属性对话框，如下图所示。

![image-20190305093201909](/Users/nacht/Library/Application Support/typora-user-images/image-20190305093201909.png)

注意：这里涉及到域的概念，即一种标准的数据结构，它可应用至数据项或实体的属性上



### 定义标识符

一、标识符

标识符是实体中一个或多个属性的集合，可用来唯一标识实体中的一个实例。要强调的是，CDM中的标识符等价于PDM中的主键或候选键。

每个实体都必须至少有一个标识符。如果实体只有一个标识符，则它为实体的主标识符。如果实体有多个标识符，则其中一个被指定为主标识符，其余的标识符就是次标识符了。

二、如果定义主、次标识符

1）选择某个实体双击弹出实体的属性对话框。在Identifiers选项卡上可以进行实体标识符的定义。如下图所示

![image-20190305093609928](/Users/nacht/Library/Application Support/typora-user-images/image-20190305093609928.png)

2）选择第一行“主标识符”，点击属性按钮或双击第一行“主标识符”，弹出属性对话框，如图所示

![image-20190305093626656](/Users/nacht/Library/Application Support/typora-user-images/image-20190305093626656.png)

3）选择"Attributes"选项卡，再点击“Add Attributes”工具，弹出如图所示窗口，选择某个属性作为标识符就行了。

![image-20190305093650815](/Users/nacht/Library/Application Support/typora-user-images/image-20190305093650815.png)



### 数据项

一、数据项

数据项（Data Item）是信息存储的最小单位，它可以附加在实体上作为实体的属性。

注意：模型中允许存在没有附加至任何实体上的数据项。

二、新建数据项

1）使用“Model”---> Data Items 菜单，在打开的窗口中显示已有的数据项的列表，点击 “Add a Row”按钮，创建一个新数据项，如图所示

![image-20190305094021404](/Users/nacht/Library/Application Support/typora-user-images/image-20190305094021404.png)

2）当然您可以继续设置具体数据项的Code、DataType、Length等等信息。这里就不再详细说明了。

三、数据项的唯一性代码选项和重用选项

使用Tools--->Model Options->Model Settings。在Data Item组框中定义数据项的唯一性代码选项(Unique Code)与重用选项（Allow Reuse）。

注意：

如果选择Unique Code复选框 ，每个数据项在同一个命名空间有唯一的代码，而选择Allow reuse ，一个数据项可以充当多个实体的属性。

![image-20190305094055941](/Users/nacht/Library/Application Support/typora-user-images/image-20190305094055941.png)

四、在实体中添加数据项

1）双击一个实体符号，打开该实体的属性窗口。

2）单击Attributes选项卡，打开如下图所示窗口

![image-20190305094113924](/Users/nacht/Library/Application Support/typora-user-images/image-20190305094113924.png)

注意：

Add a DataItem 与 Reuse a DataItem的区别在于

Add a DataItem 情况下，选择一个已经存在的数据项，系统会自动复制所选择的数据项。如果您设置了UniqueCode选项，那系统在复制过程中，新数据项的Code会自动生成一个唯一的号码，否则与所选择的数据项完全一致。

Reuse a DataItem情况下，只引用不新增，就是引用那些已经存在的数据项，作为新实体的数据项



### 联系

联系（Relationship）是指实体集之间或实体集内部实例之间的连接。

实体之间可以通过联系来相互关联。与实体和实体集对应，联系也可以分为联系和联系集，联系集是实体集之间的联系，联系是实体之间的联系，联系是具有方向性的。联系和联系集在含义明确的情况之下均可称为联系。

按照实体类型中实例之间的数量对应关系，通常可将联系分为4类，即一对一（ONE TO ONE）联系、一对多（ONE TO MANY）联系、多对一（MANY TO ONE）联系和多对多联系（MANY TO MANY）。 

**二、 建立联系**

在CDM工具选项板中除了公共的工具外，还包括如下图所示的其它对象产生工具。

![image-20190305094242790](/Users/nacht/Library/Application Support/typora-user-images/image-20190305094242790.png)

在图形窗口中创建两个实体后，单击“实体间建立联系”工具，单击一个实体，在按下鼠标左键的同时把光标拖至别一个实体上并释放鼠标左键，这样就在两个实体间创建了联系，右键单击图形窗口，释放Relationship工具。如下图所示

![image-20190305094259417](/Users/nacht/Library/Application Support/typora-user-images/image-20190305094259417.png)

三、 四种基本的联系

即一对一（ONE TO ONE）联系、一对多（ONE TO MANY）联系、多对一（MANY TO ONE）联系和多对多联系（MANY TO MANY）。如图所示

![image-20190305094344555](/Users/nacht/Library/Application Support/typora-user-images/image-20190305094344555.png)

**四、 其他几类特殊联系**



除了4种基本的联系之外，实体集与实体集之间还存在标定联系（Identify Relationship）、非标定联系（Non-Identify RelationShip）和递归联系（Recursive Relationship）。

标定联系:每个实体类型都有自己的标识符，如果两个实体集之间发生联系，其中一个实体类型的标识符进入另一个实体类型并与该实体类型中的标识符共同组成其标识符时，这种联系则称为标定联系，也叫依赖联系。反之称为非标定联系，也叫非依赖联系。

在非标定联系中，一个实体集中的部分实例依赖于另一个实例集中的实例，在这种依赖联系中，每个实体必须至少有一个标识符。而在标定联系中，一个实体集中的 全部实例完全依赖于另个实体集中的实例，在这种依赖联系中一个实体必须至少有一个标识符，而另一个实体却可以没有自己的标识符。没有标识符的实体用它所依 赖的实体的标识符作为自己的标识符。

换句话来理解，在标定联系中，一个实体（选课）依赖 一个实体（学生），那么（学生）实体必须至少有一个标识符，而（选课）实体可以没有自己的标识符，没有标标识符的实体可以用实体（学生）的标识符作为自己的标识符。

![image-20190305094542240](/Users/nacht/Library/Application Support/typora-user-images/image-20190305094542240.png)

递归联系：递归联系是实体集内部实例之间的一种联系，通常形象地称为自反联系。同一实体类型中不同实体集之间的联系也称为递归联系。



例如：在“职工”实体集中存在很多的职工，这些职工之间必须存在一种领导与被领导的关系。又如“学生”实体信中的实体包含“班长”子实体集与“普通学生” 子实体集，这两个子实体集之间的联系就是一种递归联系。创建递归联系时，只需要单击“实体间建立联系”工具从实体的一部分拖至该实体的别一个部分即可。如 图

![image-20190305094616330](/Users/nacht/Library/Application Support/typora-user-images/image-20190305094616330.png)

**五、 定义联系的特性**

在两个实体间建立了联系后，双击联系线，打开联系特性窗口，如图所示。

![image-20190305094657989](/Users/nacht/Library/Application Support/typora-user-images/image-20190305094657989.png)

**六、 定义联系的角色名**

在联系的两个方向上各自包含有一个分组框，其中的参数只对这个方向起作用，Role Name为角色名，描述该方向联系的作用，一般用一个动词或动宾组表。

如：“学生 to 课目 ” 组框中应该填写“拥有”，而在“课目To 学生”组框中填写“属于”。（在此只是举例说明，可能有些用词不太合理）。

**七、 定义联系的强制性**

Mandatory 表洋这个方向联系的强制关系。选中这个复选框，则在联系线上产生一个联系线垂直的竖线。不选择这个复选框则表示联系这个方向上是可选的，在联系线上产生一个小圆圈。



**八、 有关联系的基数**

联系具有方向性，每个方向上都有一个基数。



举例，

“系”与“学生”两个实体之间的联系是一对多联系，换句话说“学生”和“系”之间的联系是多对一联系。而且一个学生必须属于一个系，并且只能属于一个系， 不能属于零个系，所以从“学生”实体至“系”实体的基数为“1,1”，从联系的另一方向考虑，一个系可以拥有多个学生，也可以没有任何学生，即零个学生， 所以该方向联系的基数就为“0,n”,如图所示

![image-20190305094825545](/Users/nacht/Library/Application Support/typora-user-images/image-20190305094825545.png)



### 定义联系

一. RelationShip(联系)

   先给出PD手册里对联系的定义：“A relationship is a link between entities. For example, in a CDM that manages human resources, the relationship Member links the entities Employee and Team, because employees can be members of teams. This relationship expresses that each employee works in a team and that each team has employees.” 可见，也许联系的概念真的太简单了吧，所以反而不那么好表述，所以PD的文档里也是用一个例子来说明出现了什么样的情况我们就认为两个实体间是有联系的。

   当我们提起实体间联系的时候，最先想到的恐怕是one to one，one to many 和many to many这三种联系类型，这些联系类型也是大家最熟悉的。笔者对[ER图](http://shaozhuqing.com/?tag=er%E5%9B%BE)原本的概念并不精通，但在CDM中，联系还有另外三个可以设置的属性：mandatory（强制性联系）, dependent（依赖性联系/标定关联） 和dominant（统制联系）。这些属性对后面PDM的生成都有比较大的影响，需要我们一一有所了解。它们都是在联系的属性控制面板中设定的，见下图：

![image-20190305094948177](/Users/nacht/Library/Application Support/typora-user-images/image-20190305094948177.png)

1.mandatory

   联系是否具有强制性，指的是实体间是不是一定会出现这种联系；或者换句话说，当我们在谈及一个联系的应用场景的时候，联系对应的那两个实体型的实体实例的 个数可不可能为零。也许这样的解释还是有点抽象，让我们举两个联系的例子，一个是对两边的实体都有强制性的，另一个则不然。

（1）教师--学生 联系

   这个联系首先是一个多对多联系，因为每个老师可以教多个学生，每个学生也都有多个老师来负责他们的学业。同时，这个联系对教师和学生都是强制性的，也就是说，不存在任何一个老师，他不负责任何一个学生的教学；也不存在任何一个学生，他没有任何一个任课老师。

（2）学生--俱乐部 联系

   这个联系也是一个多对多关系，但它对学生这个实体型而言就不是强制的（Optional,可选的）。每个俱乐部都有至少一个学生参加，但并不是每个学生都要去参加俱乐部的活动。完全可以有一些学生，他们什么俱乐部都没参加。

上面的例子主要是从概念的角度来区分了mandatory和optional的区别。实际上如果把这个模型对应到我们最后生成的表，如果A-B间的联系对 A是mandatory的话，那么如果在A里面如果包含B的外键，这个外键不能为空值，反之可以为空值。后面我们谈到PDM和实际数据库的时候，大家会看 到这一点。

2.dependent

   每一个Entity型都有自己的Identifier，如果两个Entity型之间发生关联时，其中一个Entity型的Identifier进入另一个Entity型并与该 Entity型中的Identifier共同组成其Identifier时，这种关联称为标定关联,也叫依赖性关联(dependent relationship)。一个Entity型的Identifier进入另一个Entity型后充当其非Identifier时，这种关联称为非标定关联,也叫非依赖关联。

   概念的定义说起来还是有些拗口，说白了其实就是主-从表关系，从表要依赖于主表。比如在我们系统里要记录教师休假的情况，有一个实体型Holiday，其 属性包括休假的开始时间和天数，每次有教师休假的时候，都要在这个表留下记录。从我们的场景描述中可以看到，实体型假期必须依附于实体型教师，即对于每一 个假期实例，必须指向某一个教师实例。

   对于依赖型联系，必须注意它不可能是一个多对多联系，在这个联系中，必须有一个作为主体的实体型。一个dependent联系的从实体可以没有自己的identifier.

3.dominant

   这个联系属性是最为简单的，它仅作用于一对一联系，并指明这种联系中的主从表关系。在A,B两个实体型的联系中，如果A-->B被指定为 dominant，那么A为这个一对一联系的主表，B为从表，并且在以后生成的PDM中会产生一个引用（如果不指定dominant属性的话会产生两个引 用）。比如老师和班级之间的联系，因为每个班级都有一个老师做班主任，每个老师也最多只能做一个班级的班主任，所以是一个一对一关系。同时，我们可以将老 师作为主表，用老师的工号来唯一确定一个班主任联系。

二.Association（关联）

   先来看一下PD给association的定义：“An association is a connection between entities. In the Merise modeling methodology an association is used to connect several entities that each represents clearly defined objects, but are linked by an event, which may not be so clearly represented by another entity.”。

   在上一小段提到的那些RelationShip，在很多情况下（特别是多对多关系中），我们会把联系专门提出来，作为一个实体型放在两个需要被关联的实体 型中间（在PD中，选中任何一个联系，在右键的弹出菜单中选择“Change to Entity”命令即可完成联系转实体的操作）。但有的时候，把若干个实体型之间的联系抽象为一个实体型可能不太合适，这个时候你可以选择为这些实体型建 立一个association，那么在生成PDM的时候，所有这些相关实体型的identifier都会被加入到association对应生成的表模型 中。所以，说白了，其实association就是实体型的一种特例，用来在建模的时候更确切的表达实体间的关联信息。在PD的文档中举了一个录音带、顾 客、商店三个实体型在租借录音带这个场景上发生关联，然后把租借定义为上述三个实体型之间的association的例子，非常确切。在我们的学校模型 里，我定义了家访做为老师和学生实体型中间的一个association，在接下来产生的PDM中大家就可能看到这种定义所产生的效果。

三.Inheritance（继承）

   这种关系在概念层面是最容易理解的了，本文就不赘述了。