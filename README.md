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

        public virtual void ShowOne()
        {
            Console.WriteLine("show one...");
        }
    }
    


###How to use?

####1. *EInterceptor*, provides the basic functionalities for *aspect-oriented programming*.
  e.g.
    
      EInterceptor interceptor = new EInterceptor(typeof(Foo));

      var fooBase = (Foo)interceptor.Create();
      fooBase.ShowOne();

      var fooInterface = (IShow)interceptor.CreateProxy();
      fooInterface.Show();
    
####2. *EInterceptorFactory*
  e.g.
  
      var fooBaseCreatedByFactory = EInterceptorFactory.Create<Foo, Foo>();
      fooBaseCreatedByFactory.ShowOne();

      var fooInterfaceCreatedByFactory = EInterceptorFactory.Create<Foo, IShow>();
      fooInterfaceCreatedByFactory.Show();
      
  
      
