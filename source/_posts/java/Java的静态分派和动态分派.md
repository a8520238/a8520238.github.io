# Java的静态分派和动态分派

由来：Class文件就在编译过程中，一切方法都只是符号引用，而不是实际地址（直接引用）。直到类加载时期，甚至到运行期间才能确定目标方法的直接引用。这个特性给Java带来了强大的动态扩展能力。

## 1 解析

- 解析阶段将符号引用转换为直接引用
- 解析能成立的前提：方法在程序真正执行之前就有一个可确定的调用版本，并且这个调用版本在运行期是不可变的。也就是调用目标在编译期就要确定下来。
- Java语言中符合"编译期可知，运行期不可变"，可以分为静态方法和私有方法两大类。前者与类型直接关联，后者在外部不可被访问，这两种方法各自的特点决定了他们不可能通过继承或别的方式重写其他版本，因此他们适合在类加载阶段进行解析。
- **静态方法、私有方法、实例构造器、父类方法。这些方法称为非虚方法，它们在类加载的时候就会把符号引用解析为该方法的直接引用。与之相反，其他方法称为虚方法（除去final方法）。**

## 2 分派

### 2.1 静态分派

可以将父类成为变量的静态类型，继承的子类为变量的实际类型。

静态类型和实际类型在程序中都可以发生一些变化。

区别：
- 静态类型的变化仅仅在使用时发生，变量本身的静态类型不会被改变，并且最终的静态类型在编译器可知。
- 实际类型变化的结果在运行期才确定，编译器在编译期并不知道一个对象的实际类型是什么。

```
public class StaticDispatch {
	static abstract class Human{
	}
	static class Man extends Human{
	}
	static class Woman extends Human{
	}
	public static void sayHello(Human guy){
		System.out.println("hello,guy!");
	}
	public static void sayHello(Man guy){
		System.out.println("hello,gentlemen!");
	}
	public static void sayHello(Woman guy){
		System.out.println("hello,lady!");
	}
	
	public static void main(String[] args) {
		Human man=new Man();
		Human woman=new Woman();
		sayHello(man);
		sayHello(woman);
	}

输出：
hello,guy!
hello,guy!

```
上面的例子说明：**编译器在重载时是通过参数的静态类型而不是实际类型作为判定的依据**。并且静态类型在编译期可知，因此，编译阶段，Javac编译器会根据参数的静态类型决定使用哪个重载版本。**换句话说因为该方法是重载。** 

> 所有依赖静态类型来定位方法执行版本的分派动作称为静态分派。静态分派的典型应用就是方法重载。

### 2.2 动态分派

```
public class DynamicDispatch {
	static abstract class Human{
		protected abstract void sayHello();
	}
	static class Man extends Human{ 
		@Override
		protected void sayHello() { 
			System.out.println("man say hello!");
		}
	}
	static class Woman extends Human{ 
		@Override
		protected void sayHello() { 
			System.out.println("woman say hello!");
		}
	} 
	public static void main(String[] args) {
		
		Human man=new Man();
		Human woman=new Woman();
		man.sayHello();
		woman.sayHello();
		man=new Woman();
		man.sayHello(); 
	}
输出：
man say hello!
woman say hello!
woman say hello!

```

下面描述Java虚拟机是如何根据实际类型来分派方法执行版本:

1. invokevirtual指令的多态查找过程
- 找到操作数栈顶的第一个元素所指向的对象的实际类型，记作C。
- 如果在类型C中找到与常量中的描述符和简单名称相符合的方法，然后进行访问权限验证，如果验证通过则返回这个方法的直接引用，查找过程结束；如果验证不通过，则抛出java.lang.IllegalAccessError异常。
- 否则未找到，就按照继承关系从下往上依次对类型C的各个父类进行第2步的搜索和验证过程。
- 如果始终没有找到合适的方法，则跑出java.lang.AbstractMethodError异常。
2. 本质
- 由于invokevirtual指令执行的第一步就是在运行期确定接收者的实际类型，**所以两次调用中的invokevirtual指令把常量池中的类方法符号引用解析到了不同的直接引用上**，这个过程就是Java语言方法重写的本质。我们把这种在运行期根据实际类型确定方法执行版本的分派过程称为动态分派。
3. 实际操作
- 为了保证性能，实际操作中不会出现对方法进行频繁的搜索。
- 对这种情况，最常用的”稳定优化“手段就是为类在方法区中建立一个虚方法表（Virtual Method Table，也称为vtable），使用虚方法表索引来代替元数据查找以提高性能。
- 虚方法表：
    + 虚方法表中存放着各个方法的实际入口地址。如果某个方法在子类中没有被重写，那子类的虚方法表里面的地址入口和父类相同方法的地址入口是一致的，都是指向父类的实际入口。如果子类中重写了这个方法，子类方法表中的地址将会替换为指向子类实际版本的入口地址。
    + 为了程序实现上的方便，具有相同签名的方法，在父类、子类的虚方法表中具有一样的索引序号，这样当类型变换时，仅仅需要变更查找的方法表，就可以从不同的虚方法表中按索引转换出所需要的入口地址。
    + 方法表一般在类加载阶段的连接阶段进行初始化，准备了类的变量初始值后，虚拟机会把该类的方法表也初始化完毕。
- 虚方法表中的实际指向是在运行时赋值是给出的，因为加载父类时，不知道真实指向。