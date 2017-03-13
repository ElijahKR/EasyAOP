# EasyAOP

**EasyAOP**  is a common-component provides a easy way for *aspect-oriented programming (AOP)* in **C#**.

###What is *AOP*?
*AOP* is a programming paradigm that aims to increase modularity by allowing the separation of cross-cutting concerns...

For more details, please see from [wiki](https://en.wikipedia.org/wiki/Aspect-oriented_programming).

###How to implement?

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
            foo.Show();
        }

        public override void ShowOne()
        {
            foo.ShowOne();
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

So previous way is not the best practice, and what changes should we do for this design furter?

We can handle the virtual methods and concrete methods implemented separately.

####Virtual methods

####Concrete methods implemented from interfaces

###How to use?

####1. *EInterceptor*, provides the basic functionalities for *aspect-oriented programming*.
  e.g.
    
        EInterceptor interceptor = new EInterceptor(typeof(Foo));

        var fooBase = (Foo)interceptor.Create();
        fooBase.Print();

        var fooInterface = (IShow)interceptor.CreateProxy();
        fooInterface.Show();
    
####2. *EInterceptorFactory*
  e.g.
  
        var fooBaseCreatedByFactory = EInterceptorFactory.Create<Foo, Foo>();
        fooBaseCreatedByFactory.Print();

        var fooInterfaceCreatedByFactory = EInterceptorFactory.Create<Foo, IShow>();
        fooInterfaceCreatedByFactory.Show();
      
#Reference
[1] Head First Design Pattern
