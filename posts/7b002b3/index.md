# 设计模式与游戏完美开发(1)


&lt;!--more--&gt;

# 第二篇 基础系统

## 第三章 游戏场景的转换——状态模式（State）

### 一、游戏场景

&gt; Unity中使用场景（Scene）作为游戏运行时的环境（该环境意指游戏包含游戏物体游戏资源的环境，而非程序上的环境）。其他多数游戏引擎也有类似的概念。

**场景转换**

当游戏比较复杂时，通常会设计成多个场景，让玩家在多个场景之间转换。常见的场景设置：

- 登录场景
- 主画面场景
- 战斗场景
- ……

**场景切分的好处**

- 将不同场景不同功能不同资源明确划分
- 某些场景可以重复利用

**本书范例的场景切分**

- 开始场景（StartScene）：GameLoop游戏对象的所在，游戏启动及相关游戏设置的加载。
- 主画面场景（MainMenuScene）：显示游戏名称和“开始”按钮。
- 战斗场景（BattleScene）：游戏主要执行的场景。

**场景转换的一般实现**

​	在一个SceneManager类中一般可以用两个switch语句，一个用于切换场景，一个在Update函数里用于更新游戏逻辑。

​	缺点：

- 增加场景需要在各处（主要是两个switch语句中）增加对应的代码。

- 每一个与某种场景相关的对象都必须在SceneManager类中持有，此时这些对象同时被所有场景共享（不管它们是否有联系），增加了耦合度，不利于于维护。

  

### 二、状态模式（State）

&gt; 让一个对象的行为随着内部状态的改变而变化，而该对象也像是换了类一样。---GoF

**状态模式的核心**

![img](https://img2018.cnblogs.com/blog/974944/201904/974944-20190403160826003-362391640.png)

- Context（状态拥有者，状态机）：持有状态对象，并且包含状态转换，状态添加方法等。Context在同一时间只会处于一种状态中。
- State（状态基类）：制定状态的接口。
- ConcreteState（状态子类）：继承自状态基类，表示一个具体状态。

**状态模式实现范例**

```cc
public class Context {
    State m_state = null;//当前状态，同一时间只表现为一种状态
    
    public void Request(int Value) {//接收外界参数，进行状态转换
        m_State.Handle(Value);
    }

    public void SetState(State theState) {
        Debug.Log(&#34;Context.SetState:&#34; &#43; theState);
        m_state = theState;
    }
}

public abstract class State {
    protected Context m_Context = null;
    
    public State(Context theContext) {
        m_Context = theContext;
    }

    public abstract void Handle(int Value);//各种状态的基本行为
}

public class ConcreteStateA: State {
    public ConcreteStateA(Context theContext): base(theContext) {
        }
    
    public override void Handle(int Value) {
        Debug.Log(&#34;ConcreteStateA.Handle&#34;);
        if (Value &gt; 10) {
            m_Context.SetState(new ConcreteStateB(m_Context));
        }
    }
}

public class ConcreteStateB: State {
    public ConcreteStateB(Context theContext): base(theContext) {
        }
    
    public override void Handle(int Value) {
        Debug.Log(&#34;ConcreteStateB.Handle&#34;);
        if (Value &gt; 20) {
            m_Context.SetState(new ConcreteStateC(m_Context));
        }
    }
}

public class ConcreteStateC: State {
    public ConcreteStateC(Context theContext): base(theContext) {
        }

    public override void Handle(int Value) {
        Debug.Log(&#34;ConcreteStateC.Handle&#34;);
        if (Value &gt; 30) {
            m_Context.SetState(new ConcreteStateA(m_Context));
        }
    }
}

//测试范例
void UnitTest() {
    Context theContext = new Context();
    theContext.SetState(new ConcreteStateA(theContext));
    theContext.Request(5);
    theContext.Request(15);
    theContext.Request(25);
    theContext.Request(35);
}

Context.SetState:DesignPattern_State.ConcreteStateA
ConcreteStateA.Handle
ConcreteStateA.Handle
Context.SetState:DesignPattern_State.ConcreteStateB
Context.SetState:DesignPattern_State.ConcreteStateC
Context.SetState:DesignPattern_State.ConcreteStateA
```

除了上述三个元素，状态转换也是重要的一环。主要有两种方式：

- 交由Context类本身,按条件在各状态之间转换。*具体实现需要在Context类里写各个状态类的转换方法。*
- 产生Context类对象时, 马上指定初始状态给Context对象,而在后续执行过程中的状态转换则交由State对象负责,Context对象不再介入。*这种方式更自然，多数人也愿意用该种方式。*



### 三、使用状态模式实现游戏场景的转换

&gt; 在U3D中，游戏同一时间只会在一个场景中运行，所以我们可以将不同的场景做成不同的状态类，然后使用一个SceneStateController来控制场景（状态）转换。

![img](https://img2018.cnblogs.com/blog/974944/201904/974944-20190403191604688-44781122.png)

**具体实现**

- ISceneState: 场景类的接口, 定义《P级阵地》种场景转换和执行时需要调用的方法

- StartState, MainMenuState, BattleState: 分别对应范例中的开始场景(StartScene), 主画面场景(MainMenuScene)及战斗场景(BattleScene),作为这些场景执行时的操作类

- SceneStateController: 场景状态的拥有者(Context), 保持当前游戏场景状态, 并作为与GameLoop类互动的接口. 除此之外, 也是执行&#34;Unity3D场景转换&#34;的地方

- GameLoop:: 游戏主循环类作为Unity3D与《P级阵地》的互动接口,包含了初始化游戏和定期调用更新操作

**代码**

```c#
//ISceneState.cs
public class ISceneState {
    //靠状态名来区分状态
    private string m_StateName = &#34;ISceneState&#34;;
    public string StateName {
        get { return m_StateName; }
        set { m_StateName = value; }
    }
	//持有状态机
    protected SceneStateController m_Controller = null;

    public ISceneState(SceneStateController Controller) {
        m_Controller = Controller;
    }
    //状态开始的行为
    public virtual void StateBegin() { }
	//状态结束的行为
    public virtual void StateEnd() { }
	//状态更新的行为
    public virtual void StateUpdate() { }

    public override string ToString() {
        return string.Format(&#34;[I_SceneState: StateName = {0}]&#34;, StateName);
    }
}
//StartScene.cs
//该类仅作为加载场景出现，所以进入该状态直接加载MainMenuScene场景
public class StartScene: ISceneState {
    public StartScene(SceneStateController Controller): base(Controller) {
        this.StateName = &#34;StartState&#34;;
    }

    public override void StateBegin() {

    }

    public override void StateUpdate() {
        m_Controller.SetState(new MainMenuState(m_Controller), &#34;MainMenuScene&#34;);    
    }
}
//MainMenuState.cs
//开始界面，只需注册一个按钮事件用于开始游戏（即转换到战斗场景BattleScene）
public class MainMenuState: ISceneState {
    public MainMenuState(SceneStateController Controller): base(Controller) {
        this.StateName = &#34;MainMenuState&#34;;
    }

    public override void StateBegin() {
        Button tmpBtn = UITool.GetUIComponent&lt;Button&gt;(&#34;StartGameBtn&#34;);
        if (tempBtn != null) {
            tmpBtn.onClick.AddListener(()=&gt;OnStartGameBtnClick(tmpBtn));
     }

    public void OnStartGameClick(Button theButton) {
        m_Controller.SetState(new BattleState(m_Controller), &#34;BattleScene&#34;);
    }
}
//BattleScene.cs
//战斗场景，游戏主要玩法逻辑所在
public class BattleScene: ISceneState {
    public BattleScene(SceneStateController Controller): base(Controller) {
        this.StateName = &#34;BattleState&#34;;
    }

    public overrride void StateBegin() {
        PBaseDefenseGame.Instance.Initial();
    }

    public ovrride void StateEnd() {
        PBaseDefenseGame.Instance.Update();
    }

    public ovrride void StateUpdate() {
        InputProcess();
        PBaseDefenseGame.Instance.Update();
        if (PBaseDefenseGame.Instance.ThisGameIsOver()) {
            m_Controller.SetState(new MainMenuState(m_Controller), &#34;MainMenuScene&#34;);
        }
    }

    private void InputProcess() {
        //...
    }
}
//SceneStateController.cs
//
public class SceneStateController {
    private ISceneState m_State;
    private bool m_bRunBegin = false;//用于标识当前状态的开始行为函数有没有执行，没有执行则需要执行，已经执行过就不需要再执行。
    
    public SceneStateController() { }
    //设置状态并且转换到对应场景
    public void SetState(ISceneState State, string LoadSceneName) {
         m_bRunBegin = false;

        LoadScene(LoadSceneName);

        if (m_State != null) {
            m_State.StateEnd();
        }

        m_State = State;
    }

    private void LoadScene(string LoadSceneName) {
        if (LoadSceneName == null || LoadSceneName.Length == 0) {
            return;
        }

        Application.LoadLevel(LoadSceneName);
    }

    public void StateUpdate() {
        //场景是否加载完成，如果没加载完就不执行下面的逻辑
        if (Application.isLoadingLevel) {
            return;
        }

        if (m_state != null &amp;&amp; m_bRunBegin == false) {
            m_State.StateBegin();
            m_bRunBegin = true;
        }

        if (m_State != null) {
            m_State.StateUpdate();
        }
    }
}
//GameLoop.cs
//游戏主循环，通过SceneStateController控制游戏进程
public class GameLoop: MonoBehavior {
    SceneStateController m_SceneStateController = new SceneStateController();

    void Awake() {
        GameObject.DontDestroyOnLoad(this.gameObject);
        UnityEngine.Random.seed = (int)DateTime.Now.Ticks;
    }

    void Start() {
        m_SceneStateController.SetState(new StartState(m_SceneStateController), &#34;&#34;);
    }

    void Update() {
        m_SceneStateController.StateUpdate();
    }
}
```



### 四、总结

**状态模式的优点：**

- 减少错误的发生并降低维护难度
- 状态执行环境单一化
- 项目之间可以共享场景

**状态模式的缺点：**

- 游戏状态过多时容易“状态爆炸”。

**其他应用方式：**

- 角色控制、角色AI。
- 游戏服务器连线状态。
- 关卡进行状态。





---

> 作者: Azure  
> URL: https://vazurev.github.io/posts/7b002b3/  

