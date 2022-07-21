# Unity4Lu面试题

## 1.写lua代码时需要注意什么？如何避免

​	1.#table 可以用来获取表的长度，但table中包含nil时，不能使用。
​	2.table.sort()排序时，table必须是连贯的，中间不能有nil。
​	3.尽量使用局部变量，局部变量的存取会更快，且生命周期外就会被释放。
​	4.使用iparis时，table的key必须是连续的。
​	5.next函数常用来进行无序遍历整张表
​	6.调用库函数，如果很频繁，如string.formate("%s",str),math.abs 可以考虑函数局部化，能显著提高性能。
​	7.全局变量不需要声明，如果要删除一个全局变量，只需为其赋值nil
​	8.lua中出了nil和false表示为假，其余值都表示为真（0，空串{}）
​	9.{} 和 nil 表示不一样，{}表示表没有数据，nil表示不存在。
​	10.nil只能和自己相等。
​	11.逻辑运算：A and B A为假返回a，否则返回b。A or B A为真返回A,否则返回B.
​	12.不推荐数组下标为0开始。
​	13.lua 两个table之间不能直接相互运算，可以用元表为其中的table赋予属性。

## 2.Lua内存泄漏是什么？如何处理？
​	1.lua中有垃圾回收机制，理论上不会有内存泄漏。
​	2.内存泄漏指：已经没有被使用了，但外部依然还有引用存在的对象。		

```lua
--函数中应该被申明为local的对象忘记加local
local function test() 
    testTable = {} --这个testTabel会被存放在全局表_G中，GC时由于此对象还有引用存在，所以这里总是会有一个table泄露。 
    local mt = {} --mt加了local修饰，函数调用完后，引用也不复存在了，GC时会被回收。 
    setmetatable(testTable, mt) 
end
```

​		当它进行GC的时候，会从根部开始扫描所有的对象，如果某个地方对这个对象还有引用，就不会把这个对象内存collect，这个对象就没有被GC。

​		编写代码去检测。

## 3.Lua中变量（数据）类型有哪些？简述用途?
nil 空——可以表示无效值，全局变量（默认赋值为nil），赋值nil ，使其被删除
number 整数 ——双精度（double）类型的实浮点数
table 表 —— 数组+哈希表，table的创建通过构造表达式，通常是 { } 创建空表，关联数组：索引可以是数字或者字符串，索引数字一般是从1开始
string 字符——使用一组双引号或者单引号，…字符串拼接，#字符串长度，值类型
userdata 自定义——用户自定义，任意存储在变量中的C数据结构
function 函数——Lua编写函数
bool 布尔——true或者false
thread线程——独立线路，用于执行协同程序

其中：Lua 把 false 和 nil 看作是"假"，其他的都为"真":

##	4. C#与Lua交互过程和原理？

Wrap文件：每一个Wrap文件都是对一个C#类的包装。

C# Call Lua交互过程
C#文件先调用Lua的解析器底层的dll库（C语言编写），再由DLL文件执行相应的Lua文件

Lua Call C# 交互过程
1.Wrap方式：首先生成C#源文件对应的Wrap文件，Lua文件会调用生成的Wrap文件，再由Wrap文件去调用C#文件。
2.反射方式：当索引系统API、DLL库或者第三方库，如果无法将代码具体实现进行代码生成，可通过反射来获取，执行效率较低。

C#与Lua交互原理：虚拟栈！！！
交互通过虚拟栈实现，栈的索引分为正数和负数，如果索引是正数，则1表示栈底，如果索引是负数，则-1表示在栈顶

C# Call Lua交互原理
C#先将数据放入栈中，然后Lua去栈中获取数据，然后返回数据对应的值到栈顶，再由栈顶返回至C#

Lua Call C#交互原理
C#源文件生成Wrap文件、或C#源文件生成C模块，将Wrap文件和C模块注册到Lua的解析器中，最后再由Lua去调用这个模块的函数~

从代码文件方面解释：
lua调用C#过程：
lua->wrap->C#
先生成Wrap文件（中间文件/适配文件），wrap文件把字段方法，注册到lua虚拟机中（解释器luajit），然后lua通过wrap就可以调C#了、或者在config文件中添加相应类型也可以

C#调用Lua过程：
C#生成Bridge文件，Bridge调dll文件（dll是用C写的库），先调用lua中dll文件，由dll文件执行lua代码
C#->Bridge->dll->Lua 或 C#->dll->Lua

从内存方面解释：说白了就是对栈进行操作
C# Call Lua：C#把请求或数据放在栈顶，然后lua从栈顶取出该数据，在lua中做出相应处理（查询，改变），然后把处理结果放回栈顶，最后C#再从栈顶取出lua处理完的数据，完成交互。

## 5.Lua闭包

闭包=函数+引用环境
子函数可以使用父函数中的局部变量，这种行为可以理解为闭包！

1、闭包的数据隔离
不同实例上的两个不同闭包，闭包中的upvalue变量各自独立，从而实现数据隔离

2、闭包的数据共享
两个闭包共享一份变量upvalue，引用的是更外部函数的局部变量（即Upvlaue）,变量是同一个，引用也指向同一个地方，从而实现对共享数据进行访问和修改。

3、利用闭包实现简单的迭代器
迭代器只是一个生成器，他自己本身不带循环。我们还需要在循环里面去调用它才行。
1）while…do循环，每次调用迭代器都会产生一个新的闭包，闭包内部包括了upvalue(t,i,n)，闭包根据上一次的记录，返回下一个元素，实现迭代
2）for…in循环，只会产生一个闭包函数，后面每一次迭代都是使用该闭包函数。内部保存迭代函数、状态常量、控制变量。

## 6.说明Lua的数据结构和内存占用？

lua的数据结构的内存分配是动态的
常见的数据结构是string和table

## 7.lua是如何实现热更新的

Lua的模块加载机制
热更的核心就是替换Package.loaded表中的模块

导出函数require（mode_name）
查询全局缓存表package.loaded
通过package.searchers查找加载器

package.loaded
存储已经被加载的模块：当require一个mode_name模块得到的结果不为假时，require返回这个存储的值。require从package.loader中获得的值仅仅是对那张表（模块）的引用，改变这个值并不会改变require使用的表（模块）。

package.preload
保存一些特殊模块的加载器：这里面的值仅仅是对那张表（模块）的引用，改变这个值并不会改变require使用的表（模块）。

package.searchers
require查找加载器的表：这个表内的每一项都是一个查找器函数。当加载一个模块时，require按次序调用这些查找器，传入modname作为唯一参数。此方法会返回一个函数（模块的加载器）和一个传给这个加载器的参数。或返回一个描述为什么没有找到这个模块的字符串或者nil。

## 8.简单描述一下Lua的GC原理？并且防止内存泄露 ==

### 1、Lua的GC垃圾回收机制算法

​		`Lua的GC使用了标记清除算法Mark and Sweep`

**标记**：每一次执行GC前，从根节点开始遍历每一个相关节点，进行标记
**清除**：标记完成后，遍历对象链表，然后对需要执行清除标记的对象，进行清除

`	使用三色法：白，灰，黑，作为对象的三种状态`
新白：可以回收的对象；新创建的对象，初始状态是新白，但不会被清除
旧白：可以回收的对象；lua只会清除旧白，GC后，会更新新白
灰色：等待回收的对象：该对象已被GC访问过，但该对象引用的其它对象还未标记
黑色：不可回收的对象

**简单流程**：
1.根对象开始标记，将白色对象重置为灰色对象，加入灰色链表
2.如果灰色链表不为空，取出一个对象，重置为黑色，并遍历相关引用的对象，重置为黑色
3.如果灰色链表为空，清除一次灰色链表
4.根据不同类型对象分布回收，类型的存储表
5.判断是否遍历到链表尾
6.判断对象是否为白色
7.将对象重置为白色
8.释放资源

**总结**

> Lua通过借助**grey链表**，依次利用**reallymarkobject**对对象进行了颜色的标记，之后通过遍历**alloc链表**，依次利用**sweeplist**清除需要回收的对象。

### 	2、**Lua中的GC优化，防止内存泄露**

1.Xlua可以打标签[GCOptimize] ，C#复杂类型struct类型是引用传递到Lua，对满足条件的struct，在Lua一侧做到无GC

> **条件**：struct允许嵌套其它struct，但它以及它嵌套的struct只能包含这几种基本类型：byte、sbyte、short、ushort、int、uint、long、ulong、float、double；例如UnityEngine定义的大多数值类型：Vector系列，Quaternion，Color。。。均满足条件，或者用户自定义的一些struct

2.游戏逻辑层：实体管理器如果是本身持有了实体，那么就不应该有create/remove接口。所有实体资源,主要是目前的玩家逻辑数据, 必须直接绑定在角色上，确保角色的销毁会引发实体资源的销毁；全局资源性的数据，可以考虑放在weak table中
3.可以对lua中的模块进行分代, 不同的数据使用不同的保存,封装和清除策略,保证在最大效率的情况下准确的完成垃圾收集。
4.Collectgarbage方法就是开放给Lua开发人员的API，用于监听Lua的内存使用情况(collectgarbage(“count”))，或者需要定期调用collectgarbage(“collect”)，又或者collectgarbage(“step”)）进行显式回收。

> 1.如果系统性能还能够承受的话，建议不要直接引用对象，可以多做 一层间接层。2.lua里面的弱引用是非常有用的。3.比较大的物理内存是必要的， 这可以为大家查证问题争取足够多的时间:) 4.可以把查找泄漏的部分写入到关机 逻辑里面，每次关机的时候自动查找泄漏，然后出具报告。Lua的内存监测和回收

## 9.热更新方案有哪些？以及具体热更流程

1、整包：存放在上SteamingAssets里
——策略：完整更新资源放在包里
——优点：首次更新少
——缺点：安装包下载时间长，首次安装久
2、分包
——策略：少部分资源放在包里，大部分更新资源存放在更新资源器中
——优点：安装包小，安装时间短，下载快
——缺点：首次更新下载解压缩包时间旧
3、适用性
——海外游戏大部分是使用分包策略，平台规定
——国内游戏大部分是使用整包策略
4、文件可读写路径
——Application.streamingAssestsPath 只读目录
——Application.persistentDatapath 可读写目录
——资源服务器地址URL
5、【从资源服务器】下载单个文件或多个文件
——NetWorking.UnityWebRequest获取URL , HTTP GET , 连接资源服务器
——获取到downloadHander的文件数据Data，完成后会回调方法，将文件Data作为参数传出
6、检查是否初次安装
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021030500470679.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxNDA3NTIz,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210305010415688.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxNDA3NTIz,size_16,color_FFFFFF,t_70)

## 10.Lua实现面向对象的原理???

`总结：对象标识、状态、类体系、继承、私有性`
1.表table就是一个对象，对象具有了标识self，状态等相关操作
\2. 使用参数self表示方法的该接受者是对象本身，是面向对象的核心点,冒号操作符可以隐藏该self参数
\3. 类（Class）：每个对象都有一个原型，原型(lua类体系)可以组织多个对象间共享行为
\4. setmetatable(A,{__index=B}) 把B设为A的原型
\5. 继承（Inheritance）：Lua中类也是对象，可以从其他类（对象）中获取方法和没有的字段
\6. 继承特性：可以重新定义（修改实现）在基类继承的任意方法
\7. 多重继承：一个函数function用作__Index元方法，实现多重继承，还需要对父类列表进行查找方法，但多继承复杂性，性能不如单继承，优化，将继承的方法赋值到子类当中
\8. 私有性（很少用）基本思想：两个表表示一个对象，第一个表保存对象的状态在方法的闭包中，第二个表用来保存对象的操作（或接口），用来访问对象本身。使第一个表完成内容私有性。

代码举例~~

```lua
--类
A={}
B={}
setmetatable(A,{__index=B})
--单继承
function Account:new(o) -- 传入self为Account
    o=o or {}
    self.__index = self --直接把表Account当做元表
    setmetatable(o,self)
    return o
end
--多重继承
local function serach(k,plist) --在父类列表中查找k方法
    for i = 1, #plsit do
        -- body
        local v = plist[i][k] --第i个方法K
    if v then return v end  --V存在
    end
end
setmetatable(c,{__index=function(t,k)
    local v = search(k,parents)  --serach方法（k是第K个，parents是父类列表）
    t[k]=v --将方法保存进t数组里
    return v
end})
--私有性
function newAccount(initBalance)
    local self = {balance = initBalance} --self表示局部表，第一个表用来保存內部状态，私有性
    local withdraw = function(v)
        self.balance = self.balance - v
    end
    local deposit = function(v)
        self.balance = self.balance + v
    end
    local getbalance = function()
        return self.balance
    end
    return 
    {
        withdraw = withdraw, 
        deposit = deposit, 
        getbalance = getbalance
    }   --返回外部對象，方法名進行映射
end

acc1=newAccount(100)
acc1.deposit(10)
print(acc1.getbalance()) --答案是110
```

## 11.__index和__newindex元方法的区别

访问不存在的数据，由__index提供最终结果
__index元方法可以是一个函数，Lua语言就会以【表】和【不存在键】为参数调用该函数
__index元方法也可以是一个表，Lua语言就访问这个元表

**Lua实现面向对象：继承**
`元表带有元方法__index,并且可以使用__index来实现单继承`

```lua
prototype={x=0,y=0,width=100,height=200} --元表
local mt = {} --空表
function new(o)
    setmetatable(o,mt) --把mt设为元表
    return o
end
mt.__index=function(_,key) --元表元方法赋值一个函数，不存在键和表
    return prototype[key]
end

w=new{x=10,y=20}
print(w.width)--100

mt.__index=prototype --元表
```

__newindex用于表的更新，__index用于表的查询
对表中不存在的值进行赋值的时候，解释器会查找__newindex
__newindex元方法如果是一个表，Lua语言就对这个元表的字段进行赋值

```
如果绕过元方法可以使用 rawset(t,k,v)等价于t[k]=v
```

参考Lua程序设计第20表相关的元方法

## 12.Lua如何实现C#构造函数new方法

new传进的参数当做一个本地表
元表本身Self作为元表，设置元方法，将其作为参数表o的元表，并返回出去
从而实现C# new构造函数

调用new的时候，其实访问的是元表内容

```lua
function Class:(o) -- 传入self为Account
    o = o or {}
    self.__index = self --直接把表Class当做元表
    setmetatable(o, self)
    return o
end

local object = Class:new(o) --object变量可以访问元表
```

## 13.lua深拷贝和浅拷贝的区别？如何实现深拷贝？

如何实现浅拷贝
使用 = 运算符进行浅拷贝
分2种情况
1.拷贝对象是string、number、bool基本类型。拷贝的过程就是复制黏贴！修改新拷贝出来的对象，不会影响原先对象的值，两者互不干涉
2.拷贝对象是table表，拷贝出来的对象和原先对象时同一个对象，占用同一个对象，只是一个人两个名字，类似C#引用地址，指向同一个堆里的数据~，两者任意改变都会影响对方.

如何实现深拷贝
复制对象的基本类型，也复制源对象中的对象
常常需用对Table表进行深拷贝，赋值一个全新的一模一样的对象，但不是同一个表
Lua没有实现，封装一个函数，递归拷贝table中所有元素，以及设置metetable元表
如果key和value都不包含table属性，那么每次在泛型for内调用的Func就直接由if判断返回具体的key和value
如果有包含多重table属性，那么这段if判断就是用来解开下一层table的，最后层层递归返回。
核心逻辑：使用递归遍历表中的所有元素。
1.先看copy方法中的代码，如果这个类型不是表的话，就没有遍历的必要，可以直接作为返回值赋值；
2.当前传入的变量是表，就新建一个表来存储老表中的数据，下面就是遍历老表，并分别将k,v赋值给新建的这个表，完成赋值后，将老表的元表赋值给新表。
3.在对k,v进行赋值时，同样要调用copy方法来判断一下是不是表，如果是表就要创建一个新表来接收表中的数据，以此类推并接近无限递归。

```lua
local numTest1=5
local numTest2=numTest1 --使用 == 进行浅拷贝
local numTest2=10 --修改numTest2，不会改变numTest1
print(numTest1) 
--答案 5
print(numTest2) 
--答案 10

local tab={}
tab["好好学习"]="游戏开发"
tab["热更"]="Xlua"
for key, value in pairs(tab) do
    print(key.."对应"..value)
end

local temp = tab
tab["好好学习"]="热更"
tab["热更"]="好好学习"
for key, value in pairs(tab) do
    print(key.."对应"..value)
end
--输出答案，tab和temp都发生了改变
--热更对应Xlua
--好好学习对应游戏开发

t={name="asd",hp=100,table1={table={na="aaaaaaaa"}}};

--实现深拷贝的函数
function copy_Table(obj)

		function copy(obj)
			if type(obj)  ~= "table" then						--对应代码梳理“1”  （代码梳理在下面）
				return obj;
			end
			local newTable={};									--对应代码梳理“2”

			for k,v in pairs(obj) do
				newTable[copy(k)]=copy(v);						--对应代码梳理“3”
			end
			return setmetatable(newTable,getmetatable(obj));
		end
		
	return copy(obj)
end

a=copy_Table(t);

for k,v in pairs(a) do
	print(k,v);
end

--1.先看copy方法中的代码，如果这个类型不是表的话，就没有遍历的必要，可以直接作为返回值赋值；
--2.当前传入的变量是表，就新建一个表来存储老表中的数据，下面就是遍历老表，并分别将k,v赋值给新建的这个表，完成赋值后，将老表的元表赋值给新表。
--3.在对k,v进行赋值时，同样要调用copy方法来判断一下是不是表，如果是表就要创建一个新表来接收表中的数据，以此类推并接近无限递归。

```

## 14.lua中ipairs和pairs的区别？

第一种情况：Table的组成：哈希表（键值对，链表解决冲突），数组（数字，表，）
1、根据元素列表可以分为：哈希表{[1]=1,[3]=3,[5]=5.[6]=6} , 数组{2,4}
2、将数组中的元素放入哈希表，将会发生变化 [1]=2,[2]=4,[1]的键值对发生冲突，重新匹配,得到新的哈希表元素{[1]=2,[2]=4,[3]=3,[5]=5,[6]=6}

第二种情况：数字和表混合
先哈希表（键值对，然后在数值的方式进行匹配）
table在存储值的时候是按照顺序的，但是在存储键值对的时候是按照键的哈希值存储的，为nil也会分配一个key

```lua
local t = {[1]=1,2,[3]=3,4,[5]=5,[6]=6}
print('ipairs')
for index, value in ipairs(t) do
    print(value)
end
print('pairs')
for key, value in pairs(t) do
    print(value)
end
--答案是ipairs [2 4 3] , pairs [2 4 3 6 5] 无序 

local testTab ={1,2,3,4,5};-- '纯表'
local testTab1 = {a = 1, b = 2, c =3};
local testTab2 = {"zi",a = 5,b = 10, c = 15,"miao","chumo"};-- '杂表1'
local testTab3 = {"zi",a = 5,b = 10, c = 15,"miao",nil,"chumo"};-- '杂表2'

--testTab3答案是ipairs [zi,miao] , pairs [zi,miao,chumo,5,10,15] 

```

总结：

所以当ipairs遍历table时，从键值对索引值[1]开始连续递增，当键值对索引值[ ]断开或遇到nil时退出，所以上面的例子中ipairs遍历出的结果是2，4，3。

而pairs遍历时，会遍历表中的所有键值对，先按照索引值输出数组，在输出其它键值对，且元素是根据哈希算法来排序的，得到的不一定是连续的，所以pairs遍历出的结果是2，4，3，6，5。

如何解决既要顺序遍历，也要遇到nil也能遍历完全？？？
1.现获取table的长度，包含其中的nil，maxSize

```lua
for idx=1, maxSize do
     if pTable[idx] ~= nil then --不等于
          -- 做相应的处理...
     end
end
```

## 15.如何解析版本文件？如何加载AB包资源？具体流程是怎么样的？

1.解析版本文件列表
——File.ReadAllLines(读取文件列表资源路径URL)
——获取资源名称，获取AB包名称，获取依赖项，字典容器存储
——获取Lua文件
2.加载资源
——异步加载资源AB包，AssetBundleRequest请求，AssetBundle.LoadFromFileAsync
——先检查依赖项，再异步加载AB包依赖项
——加载成功后都有对应的回调方法，将资源作为参数传入

![img](https://img-blog.csdnimg.cn/20210304235842841.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxNDA3NTIz,size_16,color_FFFFFF,t_70)



## 16.Lua的特性

轻量级: 它用标准C语言编写并以源代码形式开放，编译后仅仅一百余K，可以很方便的嵌入别的程序里。

可扩展: Lua提供了非常易于使用的扩展接口和机制：由宿主语言(通常是C或C++)提供这些功能，Lua可以使用它们，就像是本来就内置的功能一样。

其它特性:

支持面向过程(procedure-oriented)编程和函数式编程(functional programming)；

自动内存管理；只提供了一种通用类型的表（table），用它可以实现数组，哈希表，集合，对象；

语言内置模式匹配；闭包(closure)；函数也可以看做一个值；提供多线程（协同进程，并非操作系统所支持的线程）支持；

通过闭包和table可以很方便地支持面向对象编程所需要的一些关键机制，比如数据抽象，虚函数，继承和重载等。

## 17.Lua原表

在 Lua table 中我们可以访问对应的key来得到value值，但是却无法对两个 table 进行操作。

因此 Lua 提供了元表(Metatable)，允许我们改变table的行为，每个行为关联了对应的元方法。

例如，使用元表我们可以定义Lua如何计算两个table的相加操作a+b。

当Lua试图对两个表进行相加时，先检查两者之一是否有元表，之后检查是否有一个叫"__add"的字段，若找到，则调用对应的值。"__add"等即时字段，其对应的值（往往是一个函数或是table）就是"元方法"。

有两个很重要的函数来处理元表：

**setmetatable(table,metatable):**对指定table设置元表(metatable)，如果元表(metatable)中存在__metatable键值，setmetatable会失败 。

**getmetatable(table):**返回对象的元表(metatable)。

**__index 元方法**

这是 metatable 最常用的键。

当你通过键来访问 table 的时候，如果这个键没有值，那么Lua就会寻找该table的metatable（假定有metatable）中的__index 键。如果__index包含一个表格，Lua会在表格中查找相应的键。

Lua查找一个表元素时的规则，其实就是如下3个步骤:

1.在表中查找，如果找到，返回该元素，找不到则继续

2.判断该表是否有元表，如果没有元表，返回nil，有元表则继续。

3.判断元表有没有__index方法，如果__index方法为nil，则返回nil；如果__index方法是一个表，则重复1、2、3；如果__index方法是一个函数，则返回该函数的返回值。

**__newindex 元方法**

__newindex 元方法用来对表更新，__index则用来对表访问 。

当你给表的一个缺少的索引赋值，解释器就会查找__newindex 元方法：如果存在则调用这个函数而不进行赋值操作。

**__call 元方法**

__call 元方法在 Lua 调用一个值时调用。

**__tostring 元方法**

__tostring 元方法用于修改表的输出行为。

## 18.Lua 协同程序(coroutine)

**什么是协同(coroutine)？**

Lua 协同程序(coroutine)与线程比较类似：拥有独立的堆栈，独立的局部变量，独立的指令指针，同时又与其它协同程序共享全局变量和其它大部分东西。

协同是非常强大的功能，但是用起来也很复杂。

**线程和协同程序区别**

线程与协同程序的主要区别在于，一个具有多个线程的程序可以同时运行几个线程，而协同程序却需要彼此协作的运行。

在任一指定时刻只有一个协同程序在运行，并且这个正在运行的协同程序只有在明确的被要求挂起的时候才会被挂起。

协同程序有点类似同步的多线程，在等待同一个线程锁的几个线程有点类似协同。

### 19.Lua如何在项目中使用：

通过一个C#脚本调用 Lua入口文件（Main.lua），加载全局模块，比如UI模块，事件模块，table工具类模块，class类模块，单例管理类模块等等，部分模块用C#实现，Lua调用C#

### 20.Lua实现单例模式：

```lua
Singleton={}
function Singleton:new(o)
    o=o or {}
    setmetatable(o,self)
    self.__index=self
    return o
end
 
function Singleton:Instance()
    if self.instance == nil then
    self.instance = self:new()
    end
    return self.instance
end
 
--[[
    优点：确保所有对象都访问唯一实例
    缺点：每次对象请求引用时都要检查是否存在类的实例。  必须记住自己不能使用new关键字实例化对象。
--]]

```

