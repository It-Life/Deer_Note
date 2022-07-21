# Unity面试总结-功能实现

### 1.从接到任务到完成任务，一共几个过程，每个过程注意什么？

- 需求分析：确保理解和策划想法一致。
- 搭建ui或场景：
- 编写代码：
- 测试功能：
- 解决[bug](https://www.wangt.cc/tag/bug/)：

### 2.设计一个背包系统，格子500. 伪代码实现 增，删，改，查。struct sitem{ string uid，int index, int count, int itemcode }

```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
namespace PaiXu
{
    struct sitem {
       public string uid;     //唯一id
       public int index;      //位置
       public int itemCoad;   //物品id
       public int itemAmount; //物品数量
    }
    class Class2
    {
        public int bagCount = 500;
        //申请一个背包列表
        public List<sitem> bagList = new List<sitem>();
        //初始化单个背包格
        public void InitSimpBagItem(int index)
        {
            bagList[index] = new sitem();
        }
        //初始化整个背包
        public void InitBag() {
            for (int i = 0; i < bagCount; i++) {
                InitSimpBagItem(i);
            }
        }
        //检查格子是否为空
        public bool CheckNull(int index) {
            if (bagList[index].itemAmount == 0)
            {
                return true;
            }
            else {
                return false;
            }
        }
        //检查背包中是否含有这个物品(id)，返回位置
        public int CheckItem(int id) {
            for (int i = 0; i < bagCount; i++) {
                if (bagList[i].itemCoad == id) {
                    return i;
                }
            }
            return -1;
        }
        //获取位置最小的空格子
        public int FindFull() {
            for (int i = 0; i < bagCount; i++) {
                if (bagList[i].itemAmount == 0) {
                    return i;
                }
            }
            return -1;  //满了
        }
        //增
        public void AddItem(sitem sitem) {
            //检查是否存在
            int bagIndex = CheckItem(sitem.itemCoad);
            if (bagIndex == -1)
            {
                int newBagIndex = FindFull();
                if (newBagIndex == -1)
                {
                    return;
                }
                else
                {
                    sitem.index = newBagIndex;
                    sitem.itemAmount = 1;
                    bagList[newBagIndex] = sitem;
                }
            }
            else {
                sitem.itemAmount = bagList[bagIndex].itemAmount + 1;
                bagList[bagIndex] = sitem;
            }
        }
        //删
        public void DelectItem(int id) {
            int bagIndex = CheckItem(id);
            if (bagIndex == -1)
            {
                return;
            }
            else {
                sitem item = bagList[bagIndex];
                item.itemAmount -= 1;
                if (item.itemAmount <= 0) {
                    InitSimpBagItem(bagIndex);
                }
                bagList[bagIndex] = item;
            }
        }
    }
}
```

### 3.描述一下背包如何实现？

使用MVC[设计模式](https://www.wangt.cc/tag/设计模式/)：

modle 和 control 层

- 首先要定义背包的大小
- 再定义格子类，包含的每个格子的数据，如：格子位子，物品id，数量，物品信息
- 再搞一个背包管理类，申请一个 格子类的列表，初始化。
- 包含一些方法，如初始化格子，判断格子是否为空，获取位置最小的空格子，增 删 改 查 等等。

view层

- 获取背包信息，生成相应的格子，形成背包。

### 4.如何在UI上显示模型？

- 使用RenderTexture 和 rawIamge。
- 创建一个专门照射3D模型的相机，为了避免影响主视角，可以把相机位置设置到远一些的地方。
- 创建一个RenderTexture，把它指定给该相机，然后指定给Rawimag
- 该相机下的任何物体都会被渲染纹理记录并显示再RawIamg。
- 注意事项：RenderTexture可以设置分辨率，最好和游戏一样。相机的Clear Flags设置为Solid Color。

### 5.Lua和C#是如何实现通信的？

交互过程：

- C# call Lua ：由C#文件调用Lua解析器底层dll库，由dll文件执行相应的Lua文件；C# -> dl l-> Lua
- Lua call  C# ：首先生成C#源文件所对应的Wrap文件，warp文件把字段方法注册到lua解释器中，就可以通过Warp文件调用C#文件了。 Lua ->Warp->C#

交互原理：

- 主要通过虚拟栈实现。
- C# call Lua：C#将数据放入栈顶，Lua去栈顶获取数据，在lua中做处理，然后把结果放回栈顶，再C#从栈顶取出数据，完成交互.
- Lua call C#：巴拉巴拉巴拉巴拉

### 6.做ui过程中需要注意什么？如何避免

- 画布Canvas是unity UI的基本组件。Canvas上有一个元素或多个元素变化时，会污染整个画布。解决：分离动态元素和静态元素。
- 减少不必要的RayCast Target 。UGUI的点击事件也是一个效率损耗，所以如无必要，建议关掉控件的raycastTarget属性。
- 如无必要关闭ricetext。
- 避免使用Camera.main
- 隐藏画布，禁用Canvas组件。
- 只在频繁的动态元素上使用Animator组件，对于很少变化的元素，可以使用DoTween插件来完成动画。

### 7.伤害飘字怎么实现？

- 首先制作飘字预制体，拥有出现动画，和自动销毁功能，可传出伤害[参数](https://www.wangt.cc/tag/参数/)。
- 获取世界坐标内要显示飘字跟随的目标位置，转换为屏幕坐标。transfrom.position -> anchorPoint.
- 创建一个父级容器，实例化飘字预制体。

### 8.FSM（状态机具体怎么实现）：

初始化各状态机的名称或ID（一般使用枚举），设定每种状态人物对应的行为（编写相应的脚本），状态切换（切换条件，切换回调）

### 9.SDK接入怎么接入的？Plugins目录的作用：

新建一个Android library工程，按照平台给的SDK文档配置AndroidManifest.xml，然后添加SDK的aar或者jar库到libs目录下，不要忘了去Unity安装目录找到classes.jar，放入libs目录下，然后配置gradle。

 接着就开始写SDK代码。写完之后直接生成aar库，直接导入到Unity就可以用了。

Plugins目录：Plugins里存放的SDK,库文件能够优先被编译

Windows     .dll 动态链接库

Linux Mac Android IOS   .so 共享函数库

### 10.地形优化插件

T4M插件

### 11.C#索引器的实现

索引器允许类和结构的实例就像数组一样，直观的引用类中的成员。一索引器可以重载，一个类可以有多个索引器，索引的参数可以是任何类型，而不像整数参数类型只能是整数

```C#
private string[] name = new string[10];

//定义索引器，必须以this关键字定义，设置name字段的索引值为0，address字段的索引值为

public string this[int index]{undefined

get{return name[index];}

set{this.name[index] = value;}

}
```

### 12.矩阵相乘的意义及注意点

用于表示线性的变换：旋转，缩放，投影，平移，仿射

注意矩阵的蠕变：误差的积累