# EasyAOP

**EasyAOP**  is a common-component provides a easy way for *aspect-oriented programming (AOP)* in **C#**.

### What is *AOP*?
*AOP* is a programming paradigm that aims to increase modularity by allowing the separation of cross-cutting concerns[2].

For more details, please see from [wiki](https://en.wikipedia.org/wiki/Aspect-oriented_programming).

### How to implement?

Let's provide the some code firstly, as shown below.

    public interface IShow
    {
        void Show();
    }

    public class Foo : IShow
    {
        public void Show()
        {
            Console.WriteLine("show...");
        }

        public virtual void Print()
        {
            Console.WriteLine("print...");
        }
    }
    
In a specific scenario, we want to log all the function calls before invoking and after invoking.
What should we do?

Let's see...

We want to add additional responsibilities to Foo's functions. Maybe we can use *Decorator Pattern*.

Let's have a try, the code is shown below.

    public class Stupid : Foo
    {
        Foo foo;

        public Stupid(Foo foo)
        {
            this.foo = foo;
        }

        public new void Show()
        {
            Console.WriteLine("before showing...");
            foo.Show();
            Console.WriteLine("after showing...");
        }

        public override void Print()
        {
            Console.WriteLine("before printing...");
            foo.Print();
            Console.WriteLine("after printing...");
        }
    }
    
Class *Stupid* is derived from class *Foo*, the diagram is shown below.

![Alt text](https://github.com/ElijahKR/EasyAOP/blob/master/imgs/diagram%20decorator.png "Decorator")

    Foo foo = new Foo();

    Stupid stupid = new Stupid(foo);
    stupid.Show();
    stupid.Print();
    
Looks great!

A well-architected should strive for loosely coupled designs between object that intercact[1]. So we usually coding like this.

    Foo foo = new Foo();

    Foo stupid = new Stupid(foo);
    stupid.Show();
    stupid.Print();
    
or:

    IShow stupid = new Stupid(foo);
    stupid.Show();
    
And the result is beyond our expectations, because function *Show* is a concrete method, not a *hook*(a method given an empty or default implementation).

A well-architected also should follow these principles.

1.  Program to interface, not implementation.[1]
2.  Depend on abstractions. Dot depend on concrete class.[1]

So previous way is not the best practice, and what should we do to optimize this design further?

We can handle the virtual methods and concrete methods implemented separately.

#### *Virtual methods*

For virtual methods, we can derived from the its base class to implement *AOP*.

e.g.

    Foo stupid = new Stupid();
    stupid.Print();
    
    public class Stupid : Foo
    {
        public override void Print()
        {
            Console.WriteLine("before printing...");
            base.Print();
            Console.WriteLine("after printing...");
        }
    }
    
The diagram is shown below.

![Alt text](https://github.com/ElijahKR/EasyAOP/blob/master/imgs/diagram%20inheritance.png "Inheritance")

#### *Concrete implementation methods*

For concrete implementation methods, we can use proxy that implements the same interface to relize *AOP*.

e.g.

    Foo foo = new Foo();

    IShow proxy = new Proxy(foo);
    proxy.Show();
    
    public class Proxy : IShow
    {
        IShow foo = null;

        public Proxy(IShow foo)
        {
            this.foo = foo;
        }

        public void Show()
        {
            Console.WriteLine("before showing...");
            foo.Show();
            Console.WriteLine("after showing...");
        }
    }
    
The diagram is shown below.

![Alt text](https://github.com/ElijahKR/EasyAOP/blob/master/imgs/diagram%20proxy.png "Proxy")

**Finally**, we combine these two into one compound pattern, as shown below.

![Alt text](https://github.com/ElijahKR/EasyAOP/blob/master/imgs/diagram%20compound.png "Compound")

This is what we want, it's perfect. And this is **EasyAOP**'s design idea.

### How to use?

First, add the attributes **EAtttributeAspect** to the methods that need to intercept.

e.g.

    public class Foo : IShow
    {
        [EAtttributeAspect(typeof(EAspectLogging), EEnumJoinpoints.Before | EEnumJoinpoints.After)]
        public void Show()
        {
            Console.WriteLine("show...");
        }

        [EAtttributeAspect(typeof(EAspectLogging), EEnumJoinpoints.Before)]
        public virtual void Print()
        {
            Console.WriteLine("print...");
        }
    }
    
Following is the properties of **EAtttributeAspect**

    public Type AspectType { get; set; }            // specify the type of concrete aspect
    public EEnumJoinpoints Joinpoint { get; set; }  // join points
    public bool IsAsync { get; set; }               // if true, the method invoke will be executed asynchronously
    public object Data { get; set; }                // data transmission

And then, use following two ways to write your own code.

#### 1. *EInterceptor*, provides the basic functionalities for *aspect-oriented programming*.
  e.g.
    
        EInterceptor interceptor = new EInterceptor(typeof(Foo));

        Foo fooBase = (Foo)interceptor.Create();
        fooBase.Print();

        IShow fooInterface = (IShow)interceptor.CreateProxy();
        fooInterface.Show();
    
#### 2. *EInterceptorFactory*(preffered), provides the unified interface to create interceptors. And it will retrun the specific object according to the generic return type.
  e.g.
  
        Foo fooBaseCreatedByFactory = EInterceptorFactory.Create<Foo, Foo>();
        fooBaseCreatedByFactory.Print();

        IShow fooInterfaceCreatedByFactory = EInterceptorFactory.Create<Foo, IShow>();
        fooInterfaceCreatedByFactory.Show();
        
Following is the overloads of method *Create*.

        /// <summary>
        /// create object with interceptor
        /// </summary>
        /// <typeparam name="T">the retrun type</typeparam>
        /// <param name="typeTarget">target type that will be intecepted</param>
        /// <returns></returns>
        public static T Create<T>(Type typeTarget = null){ }
        
        /// <summary>
        /// create object with interceptor
        /// </summary>
        /// <typeparam name="T">target type that will be intecepted</typeparam>
        /// <typeparam name="R">the retrun type</typeparam>
        /// <returns></returns>
        public static R Create<T, R>(){ }
        
*EInterceptorFactory* will cache the interceptors that ever been created, but it's not thread safe, please note.
        
#### Checkpoint, always follow the **Object-Oriented** principles.

### How to debug?

Because *proxy* is created dynamicly, so it hard to debug while error occurs.
But we can use *IEInterceptor.GetSourceCode()* and *EInterceptorFactory.GetSourceCode(Type typeTarget)* to generate the source code, it's very intuitive and convient to debug. 

 e.g.
 
        EInterceptor interceptor = new EInterceptor(typeof(Foo));

        string source = AppDomain.CurrentDomain.BaseDirectory + "code.cs";
        using (StreamWriter sw = new StreamWriter(source, false))
        {
            sw.Write(interceptor.GetSourceCode());
        }
        
Generate the source code, and append to your project to debug.

### How to customize your own aspect?

To relize this, you should implement the interface *IEAspect*.

e.g.

    public class EAspectAuthentication : IEAspect
    {
        public virtual void Invoke(object obj, EEventArgs arg)
        {
            IEAttributeAspect attrAspect = (IEAttributeAspect)obj;
            User data = Newtonsoft.Json.JsonConvert.DeserializeObject<User>(attrAspect.Data.ToString());

            if (data != null && data.UserName.Equals("Elijah") && data.Password.Equals("EasyAOP"))
            {
                Console.WriteLine("success to authenticate...");
            }
            else
            {
                Console.WriteLine("user name or password does not match...");
            }
        }
    }

    public class User
    {
        public string UserName { get; set; }
        public string Password { get; set; }
    }
    
How to use our customize aspect? The code is shown below.

    IShow fooInterfaceCreatedByFactory = EInterceptorFactory.Create<Foo, IShow>();
    fooInterfaceCreatedByFactory.Show();
                
    public class Foo : IShow
    {
        [EAtttributeAspect(typeof(EAspectAuthentication), 
            Data = "{\"UserName\":\"Elijah\",\"Password\":\"EasyAOP\"}")]
        public void Show()
        {
            Console.WriteLine("show...");
        }

        public virtual void Print()
        {
            Console.WriteLine("print...");
        }
    }
    
It's very easy to customize your own logic.
      
# Reference

[1] Head First Design Pattern, Eric Freeman, Elisabeth Robson, Bert Bates,Kathy Sierra.

[2] https://en.wikipedia.org/wiki/Aspect-oriented_programming

[3] https://msdn.microsoft.com/en-us/library/hh156524(v=vs.110).aspx

[4] https://msdn.microsoft.com/en-us/library/650ax5cx(v=vs.110).aspx

[5] http://blogs.msmvps.com/ricardoperes/2016/10/23/interception-in-net-part-4-an-interception-framework/?utm_source=tuicool&utm_medium=referral

<br />

If you have any question, please contact Elijah_K@163.com.
