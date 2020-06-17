## ShopApplication:
- Main -> Application
- Constants
- Controller -> tool-like controllers
- Filters -> Annotations?
- Init -> Start services
- Mapper
- Model -> not for persistance?
- Populator

## Dependency Injection:
If a class needs dependencies, i.e other objects from other classes, it better not create them by itself. An injector is needed to create and pass the dependencies (objects it needs) to it.
This means Inversion of Control: your dependencies is not under your control. What you can control, is the abstractions of the dependencies, not concrete objects. The DI helps you decide what concrete class you need. Or, the concrete class should be decided on run-time, not compile-time.

## Spring Wiring (Dependency Injection)
### @AutoWired
auto initiated.
```
@Autowired
SomeClass someObject;
```
works as the same as
```
SomeClass someObject = new SomeClass();
```
