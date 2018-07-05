#泛型程序设计学习   

泛型就是参数化的类型，在定义类型的时候不指定具体的类型，
而是用一个参数来替代，在使用的时候才传入具体的类型。
这样可以让一部分代码能重复使用，让代码模块化。       

比如在定义一个类的时候不指定它是什么类型，
用一个类型参数T来表示。在对类进行实例化的时候
再指定它是String类型还是Integer类型
```
public class Generic<T> {
private T key;
public Generic(T key) {
this.key = key;
}
}
```
//泛型的类型参数只能是类类型的，不能是简单类型
//传入实参的类型需要和泛型的类型是同一类的
```
Generic<Integer> genericInteger = new Generic<Integer>(123456);
Generic<String> genericString = new Generic<String>("valueOfKey");
```
使用泛型的接口叫泛型接口，接口泛型的定义方法和泛型类的定义方法差不多。
在一个类实现泛型接口的时候可以选择一个具体的类性比如String类型传入。
```
publci interface Generator<T> {
public T next() {
return null;
}
}
```
泛型方法是在调用方法的时候指定参数的具体类型
```
public <T> T getObject(Class<t> c) throws InstantitionException, illegalException {
T t = c.newInstance();
return t;
}
```
和泛型类相比，一般更倾向于使用泛型方法，因为泛型类子啊实例化时候
需要指定具体类型，如果想换一个类就需要重新new一个对象，
使用泛型方法在调用的时候指定类型,比较灵活