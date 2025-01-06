# 设计模式与游戏完美开发(3)


&lt;!--more--&gt;

# 第二篇 基础系统

## 第五章 获取游戏服务的唯一对象——单例模式（Singleton）

### 游戏实现中的唯一对象

在游戏开发过程中，我们常常希望一些游戏对象（比如某游戏功能管理器）具有两项特性：

- 唯一性：同时间只存在一个对象。
- 便捷性：提供一个快速获取这个对象的方法。

比较直接的想法是使用全局静态对象，但是全局静态对象很难避免产生多个对象，也容易产生全局变量名重复的问题。

所以最好让这个类只产生一个对象，并提供便利的方法来获取这唯一的对象，这就是**单例模式**。

### 单例模式

&gt; 确认类只有一个对象，并提供一个全局的方法来获取这个对象。——GoF

**结构：**

![img](https://img2018.cnblogs.com/blog/974944/201904/974944-20190405141521915-1221955030.png)

参与者如下:

- 能产生唯一对象的类,并且提供&#34;全局方法&#34;让外界可以方便获取唯一的对象

- 通常会把唯一的类对象设置为&#34;静态类属性&#34;

- 习惯上会使用Instance作为全局静态方法的名称,通过这个静态函数可能获取&#34;静态类属性&#34;

**C#实现范例：**

```c#
public class Singleton {
    public string Name { get; set; }

    private static Singleton _instance;
    
    public static Singleton Instance {
        get {
            if (_instance == null) {//保证唯一性，并提供遍历方法供使用
                Debug.Log(&#34;产生Singleton&#34;);
                _instance = new Singleton();
            }
            return _instance;
        }
    }

    private Single() { }//让外部不能new对象，保证唯一性
}

void UnitTest() {
    Singleton.Instance.Name = &#34;Hello&#34;;
    Singleton.Instance.Name = &#34;World&#34;;
    Debug.Log(Singleton.Instance.Name);
}
```



### 使用单例模式获取唯一的游戏服务对象

一般来说，游戏需要一个GameManger类来管理一些功能和全局数据，在《P级阵地》中，PBaseDefenseGame就代表这样一个类，并且应用单例模式设计这个类：

![img](https://img2018.cnblogs.com/blog/974944/201904/974944-20190405142646422-1498182104.png)

参与者说明:

- PBaseDefenseGame
  - 游戏主程序,内部包含了类型为PBaseDefenseGame的静态成员属性_instance,作为该类唯一的对象。
  - 提供使用C# getter实现的静态成员方法Instance,用它来获取唯一的静态成员属性_instance

- BattleScene
  - PBaseDefenseGame类的客户端,使用PBaseDefenseGame.Instance来获取唯一的对象

```c#
PBaseDefenseGame.cs

public class PBaseDefenseGame {
    private static PBaseDefenseGame _instance;

    public static PBaseDefenseGame Instance {
        get {
            if (_instance == null) {
                _instance = new PBaseDefenseGame();
            }
            return _instance;
        }
    }
    ...
    private PBaseDefenseGame() { }
}

BattleState.cs

public class BattleScene: ISceneState {
    ...
    pubic override void StateBegin() {
        PBaseDefenseGame.Instance.Initinal();//通过Instance直接访问PBaseDefenseGame对象
    }
    ...
}

SoldierClickScript.cs

public class SoldierOnClick: MonoBehavior {
    ...
    public void OnClick() {
        PBaseDefenseGame.Instance.ShowSoldierInfo(Solder);
    }
}
```



### 反对使用单例模式的原因

单例模式似乎看起来非常方便，不必为了“安排参数传递”和“设置引用”伤脑筋，你可以在项目的任何地方通过Instance方法使用该类对象。然而，大多数资深设计者都反对滥用单例模式，有以下几个原因：

- 全局变量的过度滥用：实质上，单例模式类也是一种全局变量，然而绝大多数类从设计角度需要保有“适当可视性”，很多类我们并不需要甚至并不愿意它被全局共享，单例模式在这种情况下可以被认为是一种“放弃思考”的暴力求解手段。
- 违反“开-闭原则”：让一个类成为单例类，理论上它就失去了继承能力，就无法对修改关闭。

但是，也有两种方法可以让单例模式返回接口类——即父类为单例模式，并让子类继承实现：

- 子类向父类注册实体对象，让父类的Instance方法返回对象时，按条件查表发挥对应的子类对象。
- 每个子类都实现单例模式，再由父类的Instance去获取这些子类。（《P级阵地》采用类似的方法）。



### 少用单例模式如何方便地引用到单一对象

**让类具有计数功能，即通过一个静态类属性计数器来限制对象的数量。**

**将A类对象设置为B类的成员，B类的方法便可以方便的使用A类对象。这也是“依赖注入”的方式之一。**可以让被引用的对象不必通过参数传递的方式,就能被类的其他方法引用.按照设置的方式又可以分为&#34;**分别设置**&#34;和&#34;**指定类静态成员**&#34;两种。

1. 分别设置

   在初始化的时候给用到的每个B类将A类对象传入

   ```c#
   public class PBaseDefenseGame {
       public void Initinal() {
           m_GameEventSystem = new GameEventSystem(this);//将PBaseDefenseGame类对象传入
           ...
       }
   }
   
   public abstract class IGameSystem {
       protected PBaseDefenseGame m_PBDGame = null;
       public IGameSystem(PBaseDefenseGame PBDGame) {
           m_PBDGame = PBDGame;
       }
   }
   
   public class CampSystem: IGameSystem {
   
       public CampSystem(PBaseDefenseGame PBDGame): base(PBDGame) {
           Initialize();
       }
   
       public void ShowCaptiveCamp() {
           m_PBDGame.ShowGameMsg(&#34;获得俘兵营&#34;);//每个系统都可以通过成员m_PBDGame使用PBaseDefenseGame类对象的方法和属性
       }
   }
   ```

   

2. 指定类的静态成员

   A类的功能若需要使用到B类的方法,并且A类在产生其对象时具有下列几种情况:

   - 产生对象的位置不确定

   - 有多个地方可以产生对象

   - 生成的位置无法引用到

   - 有众多子类

     （实际上与上面相对，1.分别设置中的产生对象位置就是确定的）

​	当满足上述情况之一时,可以直接将B类对象设置为A类中的&#34;静态成员属性&#34;, 让该类的对象都可以直接使用。

```c#
// PBaseDefenseGame.cs

public class PBaseDefenseGame {
    public void Initinal() {
        m_StageSystem = new StageSystem(this);
        // 注入其他系统
        EnemyAI.SetStageSystem(m_StageSystem);
    }
}
//举例来说,敌方单位AI类(EnemyAI), 在运行时需要使用关卡系统(StageSystem)的信息,但EnemyAI对象产生的位置是在敌方单位建造者(EnemyBuilder)之下:
EnemyBuilder.cs

public class EnemyBuilder: ICharacterBuilder {
    public override void AddAI() {
        EnemyAI theAI = new EnemyAI(m_BuildParam.NewCharacter, m_BuildParam.AttackPosition);
        m_BuildParam.NewCharacter.SetAI(theAI);
    }
}
//按照&#34;最少知识原则(LKP)&#34;,会希望敌方单位的建造者(EnemyBuilder)减少对其他无关类的引用.因此,在产生敌方单位AI(EnemyAI)对象时,敌方单位建造者(EnemyBuilder)无法将关卡系统(StageSystem)对象设置给敌方单位AI,这是属于上述&#34;生成的位置无法引用到&#34;的情况.所以,可以在敌方单位AI(EnemyAI)类中,提供一个静态成员属性和静态方法,让关卡系统(StageSystem)对象产生的当下,就设置给敌方单位AI(EnemyAI)类:
public class EnemyAI: ICharacterAI {
    private static StageSystem m_StageSystem = null;

    public static void SetStageSystem(StageSystem StageSystem) {
        m_StageSystem = StageSystem;
    }

    public ovrride bool CanAttackHeart() {
        m_StageSystem.LoseHeart();
        return true;
    }
}
```



3. 直接使用静态类

```c#
public static class PBDFactory {
    private static IAssetFactory m_AssetFactory = null;
    
    public static IAssetFactory GetAssetFactory() {
        if (m_AssetFactory == null) {
            if (m_bLoadFromResource) {
                m_AssetFactory = new ResourceAssetFactory();
            } else {
                m_AssetFactory = new RemoteAssetFactory();
            }
        }
        return m_AssetFactory;
    }
}
```





---

> 作者: Azure  
> URL: https://vazurev.github.io/posts/c5f431c/  

