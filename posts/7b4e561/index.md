# 设计模式与游戏完美开发(2)


&lt;!--more--&gt;

# 第二篇 基础系统

## 第四章 游戏主要类——外观模式（Facade）

### 一、游戏子功能的整合

一个游戏程序常常由内部数个不同的子系统构成（如事件系统，关卡系统，成就系统等等），这些系统支持着游戏基本功能和玩法。这些子系统需要按照一定的顺序进行初始化，某些条件达成时也需要按照一定的流程替他们释放资源。并且这些子系统会彼此使用对方的功能，即不同子系统之间会进行“通信”。

通常来说，我们认为这些子系统的通信及初始化过程应当发生在“内部”，因为当有其他的程序（或者我们自己）来添加一个游戏功能系统时，最好能不用去了解其他子系统之间的相关运行过程，而仅仅通过一些相对“高级”的接口来调用其他子系统的相关功能。

上一章所提到的&#34;战斗状态类(BattleScene)&#34;就是一个必须使用到的游戏系统功能的客户端，根据上一章的说明,战斗状态类(BattleState)主要负责游戏战斗的运行,而《P级阵地》在进行一场战斗时,需要大部分的子系统一起合作完成.在实现时,可以先把这些子系统及相关的执行流程全都放在BattleState类之中一起完成。

*在战斗系统类中实现所有子系统相关操作：*

```c#
public class BattleState: ISceneState {
    //声明所有需要的子系统
    private GameEventSystem m_GameEventSystem = null;
    private CampSystem m_CampSystem = null;
    private StageSystem m_StageSystem = null;
    private CharacterSystem m_CharacterSystem = null;
    private APSystem m_ApSystem = null;
    private AchivementSystem m_AchievementSystem =null;

    public GameState(SceneStateController Controller): base(Controller) {
        this.StateName = &#34;GameState&#34;;
        InitGameSystem();
    }
	//初始化这些子系统
    private void InitGameSystem() {
        m_GameEventySystem = new GameEventSystem();
        ...
    }
	//子系统的更新操作
    private void UpdateGameSystem() {
        m_GameEventSystem.Update();
        ...
    }
}
```

虽然这样的实现方式很简单，但让战斗状态类(BattleState)整个客户端去负责调用所有与游戏玩法相关的系统功能是不好的实现方式，原因是:

- 从让事情**单一化**(单一职责原则)这一点来看,BattleScene类负责的是游戏在&#34;战斗状态&#34;下的功能执行及状态切换，所以不应该负责游戏子系统的初始化,执行操作及相关的整合工作。

- 以&#34;可重用性&#34;来看,这种设计方式会使得BattleState类不容易转换给其他项目使用，因为BattleState类与太多特定的子系统类产生关联，必须将它们删除才能转换给其他项目,因此丧失可重用性

综合上述两个原因,将这些子系统从BattleState类中移出，整合在单一类之下,会是比较好的做法.所以,在《P级阵地》中应用了外观模式(Facade)来整合这些子系统，使它们成为单一界面并提供外界使用。



### 二、外观模式（Facade）的定义

&gt; “为子系统定义一组统一的接口，这个高级的接口会让子系统更容易被使用”。---GoF

其实,外观模式(Facade)是在生活中最容易碰到的模式。当我们能够利用简单的行为来操作一个复杂的系统时，当下所使用的接口,就是以外观模式(Facade)来定义的高级接口。

外观模式(Facade)的重点在于，它能将系统内部的互动细节隐藏起来，并提供一个简单方便的接口.之后客户端只需要通过这个接口，就可以操作一个复杂系统并让它们顺利运行。

![img](https://img2018.cnblogs.com/blog/974944/201904/974944-20190405110218351-676912514.png)

参与者的说明如下:

- client(客户端,用户)：

  从原本需要操作多个子系统的情况,改为只需要面对一个整合后的界面。

- subSystem(子系统)：

  原本会由不同的客户端(非同一系统相关)来操作,改为只会由内部系统之间交互使用。

- Facade(统一对外的界面)：

  整合所有子系统的接口及功能,并提供高级界面(或接口)供客户端使用。

  接收客户端的信息后,将信息传送给负责的子系统。



### 三、使用外观模式实现游戏主程序

PBaseGameDefenseGame类就是&#34;整合所有子系统,并提供高级界面的外观模式类&#34;。

![img](https://img2018.cnblogs.com/blog/974944/201904/974944-20190405125338639-801109717.png)

具体实现：

```c#
//PBaseDefenseGame.cs
public class PBaeDefenseGame 
{
    //声明各个游戏子系统
    ...
    private GameEventSystem m_GameEventSystem =null;
    ...
    //初始化
    public void Initinal() {
        ...
        m_GameEventSystem = new GameEventSystem(this);
        ...
	}
    //更新操作
    public void Update() {
        ...
        m_GameEventSystem.Update();
        ...
    }
    //获取游戏状态、敌人数量等
    ...
}

//BattleState.cs
public class BattleState: ISceneState {
    public override void StateBegin() {
        PBaseDefenseGame.Instance.Initinal();//初始化，隐藏了其它子系统的细节
    }

    public override void StateEnd() {
        PBaseDefenseGame.Instance.Release();
    }

    public override void StateUpdate() {
        ...
        PBaseDefenseGame.Instance.Update();
        ...
        if (PBaseDefenseGame.Instance.ThisGameIsOver()) {
            m_Controller.SetState(new MainMenuState(m_Controller), &#34;MainMenuState&#34;);
        }
    }
}
```



**使用外观模式的优点：**

- 使用外观模式(Facade)可将战斗状态类BattleState单一化，让该类只负责游戏在&#34;战斗状态&#34;下的功能执行及状态切换，不用负责串接各个游戏系统的初始化和功能调用。

- 使用外观模式(Facade)使得战斗状态类BattleScene减少了不必要的类引用及功能整合，因此增加了BattleState类被重复使用的机会。
- 节省时间　　

　　Unity3D本身提供了不少系统的Facade接口，例如物理引擎,渲染系统,动作系统,粒子系统等。

- 易于分工开发　　

　　对于一个既庞大又复杂的子系统而言,若应用外观模式(Facade)，即可成为另一个Facade接口.所以，在工作		的分工配合上，开发者只需要了解对方负责系统的Facade接口类，不必深入了解其中的运行方式。

- 增加系统的安全性

　　隔离客户端对子系统的接触，除了能减少耦合度之外，安全性也是重点之一。



**注意事项：**

由于将所有子系统集中在Facade接口类中，最终会导致Facade接口类过于庞大且难以维护，当发生这种情况时,可以重构Facade接口类,将功能相近的子系统进行整合，以减少内部系统的依赖性，或是整合其他设计模式来减少Facade接口类过度膨胀。



**Facade面对变化时：**

随着开发需求的变更，任何游戏子系统的修改及更换,都被限制在PBaseDefenseGame这个Facade接口类内。所以,，当有新的系统需要增加时，也只会影响PBaseDefenseGame类的定义及增加对外开放的方法，这样就能使项目的变动范围减到最小。


---

> 作者: Azure  
> URL: https://vazurev.github.io/posts/7b4e561/  

