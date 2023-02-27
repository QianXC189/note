# Event
### 发布者
```C#
using System;
using UnityEngine;

public class TestingEvents:MonoBehaviour{
    public event EventHandler<MoreDataEventArgs> OnSpacePressed;
    public class MoreDataEventArgs:EventArgs{
        public int spaceCount;
    }
    private int spaceCount;
    private void Start(){
    }
    //使用委托
    public delegate void TestEventDelegate(float f);
    public event TestEventDelegate OnFloatEvent;
    //使用Action
    public event Action<bool,int> OnActionEvent;
    //使用UnityEvent
    public UnityEvent OnUnityEvent;

    private void Update(){
        if(Input.GetKeyDown(keyCode.Space)){
            //Space pressed
            spaceCount++;
            OnSpacePressed?.Invoke(this,new MoreDataEventArgs{spaceCount=spaceCount});

            OnFloatEvent?.Invoke(5.5f);

            OnActionEvent?.Invoke(true,56);
        }
    }
}
```
>*EventArgs* 对于EventArgs类型，有2个作用。当不需要使用事件传递参数时，此变量传递null即可；当需要使用事件传递参数时，该类型当作基类使用，可传递其的子类(存储数据)，用于传递数据。

### 订阅者
```C#
using System;
using UnityEngine;
public class TestingEventSubscriber:MonoBehaviour{
    private void Start(){
        TestingEvents testingEvents=GetComponent<TestingEvents>();
        testingEvents.OnSpacePressed+=TestingEvents_OnSpacePressed;
        testingEvents.OnFloatEvent+=TestingEvents_OnFloatEvent;
        testingEvents.OnActionEvent+=TestingEvents_OnActionEvent;
    }
    private void TestingEvents_OnSpacePressed(object sender,TestingEvents.MoreDataEventArgs e){
        Debug.log("Space!"+e.spaceCount);
    }

    private void TestingEvents_OnFloatEvent(float f){
        Debug.log("Float:"+f);
    }
    
    private void TestingEvents_OnActionEvent(bool arg1,int arg2){
        Debug.log(arg1+":"+arg2);
    }

    public void TestingUnityEvent(){
        Debug.log("TestingUnityEvent");
    }
}
```
# Delegate
>store a function in a variable  
>上述 EventHandler 
>```
>public delegate void EventHandle(object sender,EventArgs e);
>```
## delegate,action,func
```c#
using System;

public class Testing : MonoBehaviour{
    public delegate void TestDelegate();
    public delegate bool TestBoolDelegate(int i);//a
    
    private TestDelegate testDelegateFunction;
    private TestBoolDelegate testBoolDelegate;

    private Action testAction;
    private Action<int,float> testIntFloatAction;
    private Func<bool> testFunc;
    private Func<int,bool> testBoolDelegateFunction;//作用与a相同,语法糖

    private void Start(){
        /*
        testDelegateFunction=MyTestDelegateFunction1;

        testDelegateFunction();

        testDelegateFunction=MyTestDelegateFunction2;

        testDelegateFunction();
        */
        testDelegateFunction+=MyTestDelegateFunction1;
        testDelegateFunction+=MyTestDelegateFunction2;

        testDelegateFunction= delegate(){Debug.Log("aaa");};//测试
        testDelegateFunction=()=>{Debug.Log("aaa");};//lambda

        testDelegateFunction();

        testBoolDelegate+=MyTestBoolDelegateFunction;
        testBoolDelegate=(int i)=>i<3;//lamb写法
        Debug.Log(testBoolDelegate(1));

        testIntFloatAction=(int i,float f)=>{Debug.Log("int float Action");};

        testFunc=()=>false;

        testBoolDelegateFunction=(int i)=> i<5;//语法糖
    }

    private void MyTestDelegateFunction1(){
        Debug.log("1");
    }

    private void MyTestDelegateFunction2(){
        Debug.log("2");
    }

    private bool MyTestBoolDelegateFunction(int i){
        return i<3;
    }
}
```
>testBoolDelegate+=(int i)=>i<3;//将无法-=操作
## 计时器案例
>脏代码
```c#
public class ActionOnTimer:MonoBehaviour{
    private float timer;
    public void SetTimer(float timer){
        this.timer=timer;
    }
    private void Update(){
        timer-=time.deltaTime;
    }
    public bool IsTimerComplete(){
        return timer<=0f;
    }
}
```
```c#
public class Testing :MonoBehaviour{
    [SerializeField] private ActionOnTimer actionOntimer;
    bool hasTimeElapsed;

    private void Start(){
        actionOnTimer.SetTimer(1f);
    }

    private void Update(){
        if(!hasTimeElapsed && actionOnTimer.IsTimerComplete()){
            Debug.log("Time Out");
            hasTimeElapsed=true;
        }
    }
}
```
>优化
```c#
public class ActionOnTimer:MonoBehaviour{

    private Action timerCallback;
    private float timer;
    public void SetTimer(float timer,Action timerCallback){
        this.timer=timer;
        this.timerCallback=timerCallback
    }
    private void Update(){
        if(time>0){
            timer-=time.deltaTime;

            if(IsTimerComplete()){
                timerCallback();
            }
        }
       
    }
    public bool IsTimerComplete(){
        return timer<=0f;
    }
}
```
```c#
public class Testing :MonoBehaviour{
    [SerializeField] private ActionOnTimer actionOntimer;
    private void Start(){
        actionOnTimer.SetTimer(1f,()=>{Debug.Log("Timer complete!");});
    }

}
```
# Loop
```c#
public class Testing: MonoBehaviour{
    private void Start(){
        int i=0;
        while(i<3){
            i++;
            Debug.Log(i);
        } 

        do{
            i++;
            Debug.Log(i);
        }while(i<3);

        List<char> charList=new List<char>{'a','b','c'};
        for(int i=0;i<charList.Length;i++){
            Debug.Log(i+":"+char[i]);
            if(charList[i]=='b'){
                charList.RemoveAt(i);
                i--;
            }
        }

        foreach(char c in charList){
            Debug.Log(c);
            //charList.Remove(c);//非法
            if(c=='b')continue;
        }

        int width=2;
        int height=3;
        for(int i=0;i<width;i++){
            for(int j=0;j<height;j++){
                if(j==1)break;
                Debug.Log(x+","+y);
            }
        }
    }
}
```
# Interface
```c#
public interface IMyInterface{
    
    event EventHandler OnMyEvent;

    int MyInt{get;set;}

    void TestFunction();
}

public interface IMySecondInterface{
    void SecondInterfaceFunction(int i);
}

public class MyClass:IMyInterface,IMySecondInterface{
    public void TestFunction(){
        Debug.Log("MyClass.TestFunction()");
    }

    public void SecondInterfaceFunction(int i){

    }
}

public class MySecondClass:IMyInterface{
    public void TestFunction(){
        Debug.log("MySecondClass.TestFunction()");
    }
    
}

public class Testing:MonoBehaviour{
    private void Start(){
        MyClass myClass=new MyClass();
        MySecondClass mySClass=new MySecondClass();
        TestInterface(myClass);
        TestInterface(mySClass);
    }

    private void TestInterface(IMyInterface myInterface){
        myInterface.TestFunction();
    }
}
```
>接口也可以继承接口
## 案例
>脏代码
```c#
public class Bullet:MonoBehaviour{
    private void OnTriggerEnter2D(Colider2D colider){
        Enemy enemy=colider.GetComponent<Enemy>();
        if(enemy!=null){
            //hit a enemy
            enemy.Damage();
        }
        
        Crate crate=colider.GetComponent<crate>();
        if(crate!=null){
            //hit a crate
            crate.Damage();
        }
    }
}
```
>优化
```c#
public interface IDamageable{
    void Damage();
}

public class Enemy:MonoBehaviour,IDamageable{}

public class Crate:MonoBehaviour,IDamageable{}
```
```c#
public class Bullet:MonoBehaviour{
    private void OnTriggerEnter2D(Colider2D colider){
        IDamageable damageable=colider.GetComponent<IDamageable>();
        if(damageable!=null){
            //hit a damageable obj
            damageable.Damage();
        }
    }
}
```
# Generics
>work with multiple types
```c#
public class TestingGenerics:MonoBehaviour{

    private delegate void MyDelegate<T1,T2>(T1 t1,T2 t2);
    private Func<int,bool> func;
    private Action<int,string> action

     private delegate TResult MyDelegate<T1,T2>(T1 t1,T2 t2);

    private void Start(){
        int[] intArray=CreateArray<int>(5,6);
        Debug.Log(intArray.Length+""+intArray[0]+""+intArray[1]);

        CreateArray<String>("5","6");
        TestMultiGenerics("2",1);

        /*
        MyClass<int> myClass=new MyClass<int>();
        myClass.value=1;
        */T被限制继承自IEnemy

        MyClass<E1> e1=new MyClass<E1>(new E1());

        MyClass<E2> e1=new MyClass<E2>(new E2());
    }

    /*
    private int[] CreateArray(int firstElement,int secondElement){
        return new int[]{firstElement,secondElement}
    }
    */

    private T[] CreateArray<T>(T firstElement,T secondElement){
        return new T[]{firstElement,secondElement}
    }

    private void TestMultiGenerics<T1,T2>(T1 t1,T2 t2){
        Debug.Log(t1.GetType());
        Debug.Log(t2.GetType());
    }
}

public class MyClass<T>where T:class,IEnemy<int>{
    public T vaule;

    public MyClass(T value){
        value.Damage(0);
    }

    private T[] CreateArray<T>(T firstElement,T secondElement){
        return new T[]{firstElement,secondElement}
    }
}

public interface IEnemy<T>{
    void Damage(T value);
}

public class E1:IEnemy<int>{
    public void Damage(int i){
        Debug.Log("E1");
    }
}

public class E2:IEnemy<int>{
    public void Damage(int i){
        Debug.Log("E2");
    }
}
```