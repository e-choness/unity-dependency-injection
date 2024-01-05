# Architectures and Patterns for Unity Projects

## Dependency Injection

A light-weight example of dependency injection using `[Provide]` and `[Inject]` attributes.
The system relies on the[attributes and reflection](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/reflection-and-attributes/) in .NET System.

### Attributes

- `ProvideAttribute` Providing dependencies. Any instances marked as `[Provide]` attribute, it is expected to be supplied with an instance of dependency.
- `InjectAttribute` Injecting dependencies. Any field or method marked as `[Inject]` attribute can expect to have the dependency satisfied by the system.

### Services

- `ServiceA` has `Initialze` method, consums and outputs a message on the console.
- `ServiceB` has `Initialze` method, consums and outputs a message on the console.
- `FactoryA` has a field to cache a `ServiceA` instance. `CreateServiceA()` only creats a new instance if the cache is null.

### Interfaces

- `IDependencyProvider` is a contract for identifying a type. It knows which classes in the system can provide dependencies.

### Components

- `ClassA` contains `ServiceA` and has an `Inject` method `Init` that accepts and assigns an instance of `ServiceA` into the field.

- `ClassB` contains both `Injuect` field attribute as for `ServiceA` and `ServiceB`, for variaty purposes. `FactoryA` follows `Inject` method attribute by using `Init` method.

- `Injector` finding and satisfying dependencies. All instances will be Monobehaviours.

  1. `Injector` utilizes reflection's `BindingFlags` to find the desired denpendency providers.
  2. To automatically find all dependency providers, take advantage of Unity's `FindMonoBehaviour` method to find all providers marked as `IDependencyProvider`
  3. `Injector` has a dictionary for registering the dependency providers and their return types. Once the providers are found, they can be invoked with optional parameters.
  4. A helper function `IsInjectable` returns any fields marked as `[Inject]`. Using `FindMonoBehaviour().Where()` to look for any injectable fields inside a MonoBehaviour object.
  5. `IsInjectable` uses `GetType().GetFeilds()` to find field `[Inject]` marks.
  6. `Any()` and `Where()` ([Know more about them](https://stackoverflow.com/questions/3703256/linq-extension-methods-any-vs-where-vs-exists)) are LINQ extention methods.`Any()` return a boolean (wheter or not any items match). `Where()` returns a new sequence of items matching the predicate.
  7. `Inject` method in `Injector` finds `[Inject]` marks in object fields by using `type.GetFields()`, resolve the provider type and assigns the instance to the `[Inject]` field instance.
  8. Similarly, `Inject` finds method `[Inject]` marks by using `type.GetMethods()`. These methods accept injectables as parameters. The query is using LINQ `Select()` to find matching parameter types and return them as an array.
  9. Addtionally, to find injectable properties using `type.GetProperties()'. Injectable properties are referred to as members have getters and setters.
  10. `Resovle` helps determine if the provider instance is registered.
  11. For injectable fields use `SetValue()` to assign instances. For injectable methods use `Invoke()` to call them.
  12. Set `Injector` class Attribute as `[DefaultExecutionOrder(<negative number>)]` will allow injector instantiated before any other objects.

- `Provider` supplies dependencies to the injection system. All instances will be Monobehaviours. Can supply itself as references.

- `Environment` provides itself as a dependency.
  
  1. This goes without letting `Factory` creating new instances. Simply use `Provide` Attribute on `ProvideEnvironmentSystem()` and let it returns itself as an instance.
  2. Best to combine with `Singleton`, that will allow autogenerated instance if one doesn't exist in the project.
  3. `Initialize()` outputs a message.

### Utilities

- `Singleton` inherits from `MonoBehaviour`. It attempts to find existing object that has the same Component. If not create one and attach a new Component of the same Type.

### Code Snippets

- Provide an instance through a provider.

```CSharp
class Provider{
    [Provide]
    public IService ProvideService() {
        return new Service();
    }
}
```

- Self Provide

```CSharp
class SelfProvider: IDependencyProvider, ISelfProvider{
    [Provide]
    public ISelfProvider ProvideSelf(){
        return this;
    }
}
```

`ISelfProvider` can be marked as `[Inject]` in other consumers as fields, method parameters or properties.

- Field Injection

```CSharp
class Consumer{
    // A established provider can 
    [Inject] private IService _service;
}
```

- Method Injection

```CSharp
class Consumer{
    private IService _service;
    [Inject]
    public void Init(IService service){
        _service = service;
    }
}
```

- Property Injection

```CSharp
class Consumer{
    [Inject] public IService Service {get; private set; }
}
```

- Miltiple Injections

```CSharp
class Consumer{
    private IService _service;
    private IFactory _factory;

    [Inject]
    public void Init(IService service, IFactory factory){
        _service = service;
        _factory = factory;
    }
}
```

### In-Editor Tools

- Components(MonoBehaviours) that has injectables will appear in `Component/Scripts/DependencyInjection`
- `Injector` component has two buttons `Validate Dependencies` and `Clear All Injectable Fields`.

## Service Locator

### Context

Inversion of Control is a way to decouple dependencies of services everytime a MonoBehaviour is trying to access them in the context of Unity. No need to manually link game objects together. Leave the automation process to Service Locator.

### Service Locator Core Modules

- `ServiceLocator` finds services for both the scene and global useage.
- `Bootstrapper` initializes services on `Awake()`.
- `ServiceManager` Registers and gets services when needed.

### Mock Serviecs

| Interface | Class | Description |
|--|--|--|
| `ILocalization` | `MockLocalization` | Mocks a method `GetLocalizedWord()` that takes a word and get translation of another language. |
| `ISerializer` | `MockSerializer` | Mocks a method `Serialize()` to serialize objects. |
| `IAuthentication`| `MockAuthentication` | Mocks a method `Login()` to login user. |
| `IGameService`| `MockGameService` | Mocks a method `StartGame()` to run the game. |

## Credits

[Unity Dependency Injection Lite](https://github.com/adammyhre/Unity-Dependency-Injection-Lite/tree/master) from [adammyhre](https://github.com/adammyhre)

[Unity Service Locator](https://github.com/adammyhre/Unity-Service-Locator) from [adammyhre](https://github.com/adammyhre)
