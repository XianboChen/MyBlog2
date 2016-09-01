---
title: Java反射浅谈（一）
date: 2016-07-25 22:12:10
tags:
  - java
---
反射常用于各种框架中如:spring中的bean就是利用反射实现以及反编译中。
java.lang.Class是反射的源头。
创建一个类，通过编译（java.exe)，生成对应的.class文件，之后使用java.exe加载（JVM类加载器）此.class文件，此.calss文件加载到内存后，就是一个运行时类，存在缓冲区。这个运行类本身就是一个Class实例，一个运行时类只加载一次。有了Class实例后，我们才可以进行如下操作： 
1、创建对应的运行时类的对象；
2、获取对象运行时类的完整结构（属性、方法、构造器、内部类、父类、所在包、异常、注解...)
3、调用对应运行时类的指定结构（属性、方法、构造器）
4、反射的应用：动态代理
### Java获取Class对象的四种方式
1、若已知具体的类，则首选通过类的class属性获取，该方法最为安全可靠，程序性能最高。
demo：
``` Java
import java.lang.reflect;
import example.Person;

public class  ReflectionTest {
	public static void  main(String args[]) throws Exception  {	
		Class clazz = Person.class; 
		//1.创建clazz对应的运行时类Person类的对象
		Person p = (Person) clazz.newInstance();   
		//2.通过反射调用运行时类的指定属性
		Field f1 = clazz.getField("name");
        f1.set(p, "xianbo");
        //3.通过反射调用运行时类的指定方法
       // p.say();
        Method m1 = clazz.getMethod("say");
        m1.invoke(p);
	   }
	}
```
``` Java
package example;

public class Person {
	public String name = "tom";
	public Person () {
		System.out.println("调用了Person()构造方法");
	}
	public void say () {
		System.out.println("my name is " + name);
	}
}
```
运行结果：

> 调用了Person()构造方法
> my name is xianbo

2、通过运行时类的对象获取
demo:
``` java
import java.lang.reflect;
import example.Person;

public class  ReflectionTest {
	public static void  main(String args[]) throws Exception   {	
		Person p = new Person();
		Class clazz = p.getClass();
		System.out.println(clazz.getName());
	   }
	}
```
运行结果：
> 调用了Person()构造方法
> example.Person

3、已知一个类的全类名，且该类在类路径下，可通过Class类的静态方法forName()获取，可能抛出ClassNotFoundException异常。（反射的动态性）
demo
``` java
import java.lang.reflect;
import example.Person;

public class  ReflectionTest {
	public static void  main(String args[]) throws ClassNotFoundException {	
		String classname = "example.Person";
		Class clazz = Class.forName(classname);
		System.out.println(clazz.getName());
		}
	}
```
运行结果：
> example.Person

4、通过类的加载器
``` java
import java.lang.reflect;
import example.Person;

public class  ReflectionTest {
	public static void  main(String args[]) throws ClassNotFoundException  {	
		ReflectionTest re = new ReflectionTest();
		re.test();
	   }
	public void test() throws ClassNotFoundException {
    String classname = "example.Person";
	ClassLoader classLoader = this.getClass().getClassLoader(); 
	//Cannot use this in a static context
	Class clazz = classLoader.loadClass(classname);
	System.out.println(clazz.getName());
 	}
}
```
运行结果：
> example.Person

### 创建运行时类的对象
在得到Class运行时类后，我们就可以通过newInstance()方法创建运行时类所对应的对象
domo
``` java
import example.Person;

public class TestConstructor {
	public void test1() throws Exception {
		String className = "example.Person";
		Class clazz = Class.forName(className);
		//创建运行时类的对象
		Object obj = clazz.newInstance();
		Person p = (Person)obj;
		System.out.println(p);
	}
}
```
运行结果：
> 调用了Person()构造方法
> example.Person@4d871a69

这里调用newInstance()实际上就是调用了运行时类的空参构造器，所以要想创建对象成功要满足两点：
1、对应的类要有空参的构造器；(没有空参的构造器也能通过其他的方法得到对象）
2、构造器的权限修饰符权限要足够。

### 获取类的属性和方法
在得到类后，可以通过getField（）方法获取public的属性，还可以通过getDecllareFields()方法获取该类的所有属性。
demo
``` java
public static void test1() {
		Class clazz = Person.class;
		Field[] fields = clazz.getDeclaredFields();
		for(Field f : fields) {
			//1.获取每个属性的权限修饰符
			int i = f.getModifiers();//在java源码中，修饰符是一个个static final的数字
			String str1 = Modifier.toString(i);
			System.out.print(i + " "  + str1 + " " );
			//2.获取属性的类型
			Class type = f.getType();
			System.out.print(type + " " + type.getName() + " ");
			//3.获取属性名
			System.out.println(f.getName());
			System.out.println();
		}
		
	}

}
```
``` java
package example;

public class Person {
    String name = "xianbo";
    public int age = 20;
    private float height = 180;
	public Person () {
		System.out.println("调用了Person()构造方法");
	}
	public void say () {
		System.out.println("my name is " + name);
	}
}
```
运行结果：
> 0  class java.lang.String java.lang.String name

> 1 public int int age

> 2 private float float height












