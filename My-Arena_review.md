## 一、环境配置

### <1> 外联库
使用 `Qt` 做图形可视化部分 
### <2> IDE
使用 `Clion` 编辑，具体配置过程：
- 首先是工具链，可以用 `Clion` 自带的 `bundled MinGW` 也可以使用自己下载的 `MSVC` 
- `CMake` 方式可以选择 `release` （另一个是 `debug` ，不过过程会更多，稍微慢一点)
- `CMake Options` 中要写出 `Qt` 的路径，这样编译的时候才能链接上，这是传给 `CMake` 构建系统的参数， `CMake` 要知道去哪里找 `Qt` 的头文件。否则，代码根本不能生成
```
  -DQt6_DIR="D:/Users/app_download/qt/6.11.1/msvc2022_64/lib/cmake/Qt6"
```
- 在项目配置中，环境变量中加上 `Qt` 的 `PATH` ，这样在运行过程中才能链接上，如果运行过程中要调用 `Qt` 的动态链接库( `.dll` 文件)。否则，程序根本无法运行
```
PATH=D:/Users/app_download/qt/6.11.1/msvc2022_64/bin
```

> 一定要注意! `Qt` 库关于 `MinGW` 和 `MSVC` 的编译出的二进制结果并不一样，而且不兼容，一定要注意工具链和 `Qt` 的匹配，如果不匹配，可以去运行 `Qt` 安装文件夹中的 `MaintenanceTool.exe` 添加组件（下载相应的版本）：`Qt for Development/Qt/Qt 6.11.1/`
 
 
## 二、游戏设计

### <1> 设计思路
#### 1. 游戏基本功能

游戏以关卡循环方式进行，每回合将有以下状态：

| 游戏状态        | 游戏流程                                     |
| ----------- | ---------------------------------------- |
| 备战（Prepare） | 玩家可以在商店购买英雄，并实现鼠标拖拽功能，实现人物在战场和备战区的移动     |
| 战斗（Combat）  | 当玩家点击开始战斗按钮时，战场上的棋子自动索敌，移动，攻击，直到一方棋子完全阵亡 |
| 结算（Resolve) | 战斗结束后，根据战斗结果进行相应的数据更新                    |
#### 2. 游戏系统配置

| 数据系统 | 具体功能                              |
| ---- | --------------------------------- |
| 金币系统 | 玩家依据自己的金币，进行相关属性的提升和商城的购物         |
| 装备系统 | 战斗胜利有可能获得装备，英雄穿戴装备可使其属性提升或机制改变    |
| 羁绊系统 | 战场上出战英雄满足特定关系触发羁绊，战斗英雄获得属性提升或机制改变 |

#### 3. 英雄基本属性

| 属性分类       | 具体属性                                                  |
| ---------- | ----------------------------------------------------- |
| 棋子固有       | 名称，所属，图像，职业，等级，花费                                     |
| 等级影响的参数    | 生命阈值，法力阈值，攻击伤害，攻击阈值，攻击范围，移动阈值                         |
| 装备和羁绊影响的参数 | 生命阈值buff，法力阈值buff，攻击伤害buff，攻击范围buff，攻击阈值buff，移动阈值buff |
| 战斗动态参数     | 生命值，法力蓄值，行动状态，移动蓄值，攻击蓄值                               |
### <2> 程序通用架构


整体划分成三层模块类：

1. 顶层 `GUI` 层：负责向控制层索取数据，可视化呈现

2. 中层核心控制层 ： 负责向下获取数据，向上传给顶层 `GUI` 层进行渲染。同时控制游戏流程

3. 底层逻辑数据层 ：根本上负责具体的游戏数据

- 顶层 `GUI` :
	
	框架类
	- `GameWindow` 类：负责棋盘绘制
	
	数据类
	- `PieceWidget` 类 ：英雄单元格
	- `ParaWidget` 类：玩家参数表
	- `EquipWidget` 类：装备单元格

- 中层核心控制层：

	- `GameManager` 类：负责从底层类中提取数据向上传到 `GUI` 层。控制游戏流程

- 底层逻辑数据层：
	
	框架类：
	- `PreBoard` 类：负责除战斗过程中的棋盘数据存储
	- `ComBoard`类：负责战斗过程中的棋盘数据存储
	- `Player` 类：负责存储玩家等级，金币，积分的数据
	- `Shop` 类：负责商店的数据存储和刷新逻辑
	
	数据类：	
	- `Unit` 类：基类，负责英雄基本属性和方法
	- `Equipment` 类：基类，负责装备的基本作用和方法

### <3> 程序实际实现

#### 第一代版本

虽说是第一代版本，其实也是反复清洗代码之后的完整程序。不过设计不够成熟，讲一讲遇到的问题和困难，以及后续的改进。
##### 1. 对象生命周期管理方式

###### 1）框架类生命周期管理

三层框架类，层级通过组合关系体现，对于框架类使用智能指针（`unique_ptr`）进行上层对下层的管理

```
class GameWindow : public QMainWindow {  
    Q_OBJECT
    //只需要拥有一个管理者，向管理者要数据，管理者拥有所有的数据  
    std::unique_ptr<GameManager> gameManager;
}

class GameManager : public QObject{  
    Q_OBJECT  
    //拥有棋盘数据  
    std::unique_ptr<PreBoard> preBoard;  
    std::unique_ptr<ComBoard> comBoard;  
    //拥有玩家数据  
    std::unique_ptr<Player> player;  
    //拥有商店数据  
    std::unique_ptr<Shop> shop;  
    //拥有装备库  (这里设计有瑕疵，后期修改)
    std::vector<Equipment*> equipment;
```

###### 2）实体单位生命周期管理

第一代，对于装备和战斗单位采用聚合关系，指针管理。依靠 `PreBoard` 和 `Shop` 管理英雄的动态内存。`ComBoard` 类作为临时数据代理，不参与生命周期管理。

##### 2. 需要解决的问题

###### 1）棋盘绘制

这个版本，我采用控件族（`QWidget`及其派生类）结合布局（`Layout`）进行绘图，最大程度实现自定义和可更改。

###### 2）鼠标点击事件

关于鼠标事件，有以下几个：

| 鼠标事件       | 具体功能及实现                          |
| ---------- | -------------------------------- |
| 鼠标按下事件（左键） | 左键落下，负责记录拖拽事件的开始，拖拽可以是拖拽英雄，和拖拽装备 |
| 鼠标松开事件（左键） | 左键松开，负责操作拖拽事件的结束，判定能否成功进行此次操作    |
| 鼠标按下事件（右键） | 右键按下，进行装备的卸下操作                   |
| 鼠标双击事件     | 商店的购买操作                          |
第一代由于 `GameManager` 暴露了很多底层数据接口，所以在此基础上的鼠标事件，都是相当于直接穿透控制层获取底层数据，具体实现如下：
1. 鼠标点击时，我们获取此时最顶层的控件
2. 与`GameWindow` 维护的控件数组进行匹配，匹配成功后，得到具体坐标
3. 穿透控制层，获取底层对应坐标的英雄指针。
4. 将该控件和英雄指针进行绑定，用于 `GUI` 渲染
###### 3）战斗数据管理

对于战斗数据，我们用 `ComBoard` 类持有 `Unit` 单位指针，指向已经上场战斗的英雄，同时更改底层 `Unit` 数据，在每轮战斗之后进行数据恢复，第一代采用 `preBoard` 管理生命周期，所以我们遍历棋盘恢复数据。

###### 4）单位行动决策

战斗过程中如何实现单位行动，这个调度权交给 `ComBoard` ，但是决策权交给 `Unit` ，`ComBoard` 遍历战场，每遍历到一个单位，调用 `Unit->march()` 方法，得到行动指令结构体（行动方式，目标），依照指令 `ComBoard` 进行调度，完成此次行动。

在 `Unit->march()` 方法中，我们需要完成两件事：
1. 索敌：找到最近敌人的位置
2. 行动判定：
	- 如果敌方单位处在攻击范围内，就直接进行攻击；
	- 如果鞭长莫及，那么进行方向确定：
		1. 首先找到新目标位置：离自己最近的可攻击位置
		2. 然后确定移动指令：（为了节省开销，我们不采用寻路算法）
			1. 首先计算横竖距离差，则选差距小的方向
			2. 如果该方向不能行动（有阻挡），那么向另一个差距稍大的方向
			3. 如果依旧阻挡，选择2中反向
			4. 如果依旧阻挡，选择1中反向
			5. 如果依旧阻挡，此轮不动，保留移动蓄值

> 关于如果想要实现搜索算法，详见 *参考资料*


##### 3. 第一代的几个问题

总体评价，这个版本虽然实现了控制层的所有内容。但是呈现出以下几个问题：

- 第一个，关于控制层功能的严重错位，落入上帝类陷阱。

- 第二个，关于模块划分不够彻底和清晰，体现在装备库的处理上，没有单独的设置成一个类进行装备的统一管理

- 第三个，关于英雄羁绊的处理方式不够舒服，关于属性增值和机制改变的架构设计非常草率。

- 最后一个，我觉得是最根本得问题，来自于方法干预权限的问题，什么时候检测，什么时候更改，由谁更改，都不够清晰。

我们来具体的阐述一下这四个问题的详细。

###### 1）控制层的功能错位

什么是上帝类：就是知道的太多，做的太多。过度的暴露了很多底层的 `Get` 操作，比如第一代的 `GameManager` 类提供给 `GameWindow` 的接口：

```
public:  
    GameManager(int r=8, int c=8,int pos=8);  
    //游戏流程  
    GameState getCurrentState() const;  
    void changeStateTo(GameState newGameState);  
    bool isTimerActive() const;  
    void timerStart();  
    void timerStop();  
    bool isComEnd();  
    void resetTheGame();  
    int selfUnitCount() const;  
    void setEnemy();  
    void setEnemyRandomly();  
    void goToNextLevel();  
    void clearEnemyGrid();  
    //棋盘状态  
    int getRow() const;  
    int getCol() const;  
    int getBen() const;  
    Unit* getUnitAtGrid(int r,int c) const;  
    Unit* getUnitAtBench(int pos) const;  
    bool isCellEmptyGrid(int r, int c) const;  
    bool isCellEmptyBench(int pos) const;  
    void removeUnitAtGrid(int r, int c);  
    void removeUnitAtBench(int pos);  
    void placeUnitAtGrid(int r, int c,Unit* unit) ;  
    void placeUnitAtBench(int pos,Unit* unit) ;  
  
    //装备库  
    Equipment* getEquipmentAt(int s) const;  
    bool isEquipmentEmpty(int s) const;  
    void placeEquipmentAt(int s,Equipment* equipment);  
    void removeEquipmentAt(int s);  
    //SHOP  
    bool isCellEmptyShop(int s)const;  
    Unit* getUnitAtShop(int s)const;  
    void placeUnitAtShop(int s,Unit* unit);  
    void removeUnitAtShop(int s);  
    void refreshShop();  
    void autoMergeToBench(int k);  
  
    //PLAYER  
    int getPlayerMoney() const;  
    int getPlayerLevel() const;  
    int getPlayerShopRefreshTimes() const;  
    int getMaxUnit() const;  
    void levelUp();  
    void changePlayerMoney(int num);  
    int getPlayerScore() const;  
    void changeShopRefreshTimes(int num);  
    Player* getPlayer() const;  
  
    //存档系统  
    void saveGame(const QString& path);  
    void loadGame(const QString& path);  
  
    //向外发送的信号参数  
    signals:  
    void shouldUpdate();  
    void enterResolve(ComResult result,int hp);  
    void failToCombat();  
    void traitActivate(bool w,bool m,bool a);  

```

可以发现，在 `GameManager` 类的 `public` 方法中，暴露了非常多的底层数据接口，导致代码的臃肿，以及，你会发现，如果我的渲染层需要一个数据，这个控制层完全充当了一个数据传输者的角色（`Data Proxy`），而我们的解决方法则是迁移Web中后端传给前端渲染的 `DTO(Data Transfer Object(数据传输对象）` 或者 `ViewModel（视图模型）` ，保持 `MVC` 架构核心，不完全参考 `MVVM架构` ，吸收其思想即可。

> 关于 `MVC` , `MVP` 以及 `MVVM` 三种架构，详见 *参考资料*

具体实现方式，由于C++没有一个方便的字典类。所以，我们在控制层定义结构体进行强替代，由控制层像数据层索取数据，封装成结构体列表，向上传。
这个时候不免担心，性能开销会不会很大，由于返回对象是我们在函数中创建的临时变量，所以每一次返回的时候将会被动的触发移动语义，而不会触发昂贵的拷贝开销。

> 关于移动语义的触发，详见 *参考资料*

具体更改结果我们放到第二版时候进行实际修改。

###### 2）模块划分不清晰

注意到，关于商店和装备栏，我们采用两种所有方式：
```
    //拥有商店数据  
    std::unique_ptr<Shop> shop;  
    //拥有装备库  
    std::vector<Equipment*> equipment;  
```

`Shop` 类作为一个单独的类构造，用智能指针链接。
武器库却直接用一个指针数组直接存储在控制层，这实际上严重违背了我们控制层数据层解耦的思想，所以第二版的重要改进方向，就是处理好装备库的数据封装问题。

###### 3）属性和机制实现方式不够优雅

对英雄属性的buff，来自两个部分，一个是装备，一个是羁绊。第一版的实现方式是对buff进行额外定义为成员，方便更改。
但是机制改变，只能是写死在英雄中，我增加一个羁绊触发标志，然后依赖标志去在英雄的攻击过程中进行调整

```
//装备buff加成
Equipment* equipment;  
int equipHpBuff;  
int equipAttBuff;  
int equipAttSpeedBuff;  
int equipAttAreaBuff;  
int equipMoveSpeedBuff;  
int equipManaBuff;  
//羁绊 buff 加成  
int traitHpBuff;  
int traitAttBuff;  
int traitAttSpeedBuff;  
int traitAttAreaBuff;  
int traitMoveSpeedBuff;  
int traitManaBuff;
```

这个设计让我觉得不够优雅。后期进行修改。


#### 第二代版本

##### 1. 更新概览

###### 1）实体单位生命周期管理

本次我们不采用 `PreBoard` 和 `Shop` 管理英雄单位，管理方式分开：

- 已购买的单位，我们在 `GameManager` 类中维护一个指针容器，负责管理已购买单位的生命，统一释放，这样后续甚至可以拓展售卖英雄的操作

- 商店单位，由于体量不大，我们就不用 *英雄库预加载* 或者 *重载new操作符* 的方式进行管理，就是直接 `new` 和 `delete` 。同时，各种具体英雄的头文件，就交给 `shop` 来持有。

这样，对于实体英雄单位的生命周期管理就更清晰了，棋盘只负责持有指针，不负责 `delete` 释放空间。

###### 2）关于 `GameManager` 的优化

之前由于我们暴露很多底层数据接口，导致体量很大，很多其实本质就是数据传递的功能。

更新的核心思想就是重构 `GUI` 层和 `GameManager` 的交流方式，重构 `GameManager` 为两类方法：

- 状态查询类：
	- `bool` 类，负责审核
	- `DTO` 类，负责数据包传递
- 行为操作类：
	- `void` 类，负责响应玩家操作，主动地更改数据状态


###### 3）重构装备库和商店类

我们维护两个类，一个是 `EquipmentRepo` 一个是 `Shop` 。
他们内部维护两个指针数组：
- 一个是展示用：负责传递给GUI展示
- 一个是存储用：用于内部维护，管理生命周期

##### 2. 程序具体实现


## 三、重写过程的小Tips

第二个版本是我从空文件开始重新搭框架，一次重写，以更流畅的方式回顾这个项目。

这一次，我们脑海中已经有了很多内容，我们知晓了整体的样子，知道了大体的责任分工和实现方式，那么，我也将更注重其中的细节和知识，一下仅作 `tips` 记录，具体详情，见最后的参考资料。

### <1> `Q_OBJECT` 宏 (Macro Definition)：

为了使用 **信号与槽函数** 的功能，我们要让 `GameManager` 类继承 `QObject`,同时，将 `Q_OBJECT` 宏写在第一行

```
class GameManager: public QObject {  
    Q_OBJECT  
    std::unique_ptr<PreBoard> preBoard;  
    std::unique_ptr<ComBoard> comBoard;  
    std::unique_ptr<Player> player;  
    std::unique_ptr<Shop> shop;  
    std::unique_ptr<EquipmentRepo> equipmentRepo;  
public:  
    GameManager();  
};
```

> 关于宏的概念，详见*参考资料*

### <2> `explicit` 说明符 (Specifier)

一般情况下，我们只有在单参数构造函数实现类型转换的时候，为了防止隐式类型转换才会加上这个说明符。

### <3> `delete nullptr`

类中成员的生命周期由该类保管的时候，我们需要写析构函数，其中 `delete` 操作，无需检验空指针， `delete nullptr` 无事发生，也不会报错。

### <4> 智能指针和裸指针

智能指针和裸指针是不同的，如果我们使用 `std::make_unique<>` 创建智能指针，想要拿到对象的裸指针，需要使用 `.get()` 方法

### <5> `QLabel`  和 `QPixmap`

`QLabel` 是一个控件，可以当容器和相框，可被加入布局，设置大小等等。

`QPixmap` 是一个图像数据，相当于一张图片

```
//一般要展示照片，需要二者合作

// 1. 创建 QPixmap（把照片洗出来） 
QPixmap heroIcon(":/images/hero_jinx.png"); 

// 2. 创建 QLabel（拿一个相框） 
QLabel *iconLabel = new QLabel(this); 

// 3. 把照片放进相框里 
iconLabel->setPixmap(heroIcon); 
```

### <6> `QWidget` 和布局系统

- 每 `new` 一个 `QWidget` 出来之后都需要考虑设置以下（3+4）：

```
QWidget* part = new QWidget(parent);
part->setFixedSize(200,100);
part->setStyleSheet("border:1px solid red;background:green);
```

- 每 `new` 一个 `Layout` 出来之后都需要考虑设置以下：

```
QVBoxLayout* partLayout = new QVBoxLayout(part);
partLayout->setContentsMargins(5,5,5,5);
partLayout->setSpacing(5);
partLayout->setAlignment(Qt::AlignCenter);
```

### <7> 布局系统的对齐方式

布局设置的对齐，和添加控件的对齐，两者不太一样。

布局设置的对齐，针对的是布局方向上的对齐，即主轴方向；

添加控件的对齐，是控制另一个方向的对齐，即交叉轴方向；

### <8> 按钮和`QMessageBox::question`

代码示例：

```
connect(shopButton,&QPushButton::clicked,this,[=]() {  
    shopButton->setFixedSize(CELL_SIZE-5,25);  
    gameManager->sleepMs(50);  
    shopButton->setFixedSize(CELL_SIZE,30);  
    int ans = QMessageBox::question(  
        this,  
        "Sure?",  
        "Spend 5 coins to refresh?",  
        QMessageBox::Yes|QMessageBox::No,  
        QMessageBox::Yes);  
    if (ans == QMessageBox::Yes) {  
        // gameManager->requestShopRefresh();  
  
    }
```

