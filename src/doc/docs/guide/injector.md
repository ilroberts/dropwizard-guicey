# Injector instance

## Restrictive options

Guicey itself is compatible with the following guice restrictive options:

```java
public class MyModule extends AbstractModule {
        @Override
        protected void configure() {
            binder().disableCircularProxies();
            binder().requireExactBindingAnnotations();
            binder().requireExplicitBindings();
        }
    }
```

So it is safe to enable them.

## Access injector

In some cases it may be important to get injector instance outside of guice context.

!!! note
    Injector is created on dropwizard run phase. Attempt to obtain injector before it
    will lead to exception.

Injector instance could be resolved with:

* `getInjector()` method on GuiceBundle instance (NPE will be thrown if injector not initialized)
* `InjectorLookup.getInjector(app).get()` static call using application instance (lookup returns `Optional` for null safety).

If you need lazy injector reference, you can use `InjectorProvider` class (it's actually `Provider<Injector>`):

```java
Provider<Injector> provider = new InjectorProvider(app);
// somewhere after run phase
Injector injector = provider.get();
```

When you are inside your application class:

```java
public class MyApplication extends Application<Configuration> {
    
    @Override
    public void run(TestConfiguration configuration, Environment environment) throws Exception {
        InjectorLookup.getInjector(this).get()
                .getInstance(SomeService.class).doSomething();
    }
}
```

!!! tip
    Most likely, requirement for injector instance means integration with some third party library.
    Consider writing custom installer in such cases (it will eliminate need for injector instance).
    
Inside guice context you can simply inject Injector instance:

```java
@Inject Injector injector;
```    

## Injector factory
  
You can control guice injector creation through `ru.vyarus.dropwizard.guice.injector.InjectorFactory`. 

Default implementation is very simple:

```java
public class DefaultInjectorFactory implements InjectorFactory {

    @Override
    public Injector createInjector(final Stage stage, final Iterable<? extends Module> modules) {
        return Guice.createInjector(stage, modules);
    }
}
```

Injector creation customization may be required by some 3rd party library.
For example, [netflix governator](https://github.com/Netflix/governator) 
owns injector creation ([see example](../examples/governator.md)).

Custom injector factory could be registered in guice bundle builder:

```java
bootstrap.addBundle(GuiceBundle.builder()
            .injectorFactory(new CustomInjectorFactory())
            ...
```