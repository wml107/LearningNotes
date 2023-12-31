# 单例模式 Singleton Pattern

> Spring中的Bean默认就采用的是单例模式。

**概念：**Ensure a class has only one instance, and provide a global point of access to it.（确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例。）

**实现：**把一个类的构造函数设置为私有；然后添加一个静态变量存储单例，给他赋值为new构造函数，即在类初始化的时候创建这个单例；最后再添加一个获取这个单例的方法。

**优：**

- 只有一个实例，资源消耗少。

- 可以避免对资源的多重占用。

  > 因为只有一个实例再内存，所以可以避免例如对同一个文件的同时写操作。
  
- 能提供一个全局的共享访问点。

**缺：**

- 没有接口，扩展困难、变更风险大。

  一般情况下，它因为他是自动实例化的，所以创建接口、抽象类这些也就没有任何意义。

- 没有接口，测试不便。

  因为没接口，所以不能使用mock的方式虚拟一个结果。

- 大概率会和单一职责原则冲突。

**适用场景：**有很多，要求一个类有且仅有一个对象，出现多个对象就会出现“不良反应”的场景。

- 要求生成唯一序列号的场景。

- 整个项目需要一个共享访问点或共享数据。

  > 例如Web页面上的计数器，用单例就可以不用把每次刷新结果放在数据库，也能保证计数器的值正确。

- 创建一个对象需要消耗的资源太多，例如访问IO和数据库等资源。

- 需要定义大量静态常量和方法的环境，例如工具类。

  这一条我觉得比较牵强，只能是说可以这样用。但和你直接把所有静态常量和方法都声明成static，用起来没有太大区别。

**注意：**单例模式中，单例的创建最好直接赋值给静态常量，在类初始化的时候创建，不要把实例的创建写在实例获取方法里(具体一点就是判断静态变量是否为空，是就创建后返回，不是就直接返回)。这会导致线程不安全，因为在多线程的情况下，很可能多个线程同时获取这个实例，线程1获取的时候，方法发现没有，决定创建一个，结果在创建前时间片恰好用完，切换到了线程2，线程2也发现没有，决定创建一个，这样就出现了两个该类的实例，破坏了最初单例的预期。当然如果非要这样做也不是不可以，使用synchronized即可。

> 除此之外还要注意复制的情况，Java里对象默认是不可被复制的，但你实现了Cloneable接口和clone方法，就可以复制了，Java的复制是从语言底层实现的，并不是调用构造函数，在构造函数私有的情况下对象也能被复制。这也可能导致单例预期被破坏。

**扩展：多例模式**

使用new关键字正常执行实例化，可以创建无限的实例；使用单例模式只能创造一种实例；显然还有一种情形就是，我们希望创造指定限制个数的实例。

实现方式和单例一样，只需根据规定的数量限制，在类初始化的时候创建相应个数的实例。然后再按照一定的规则获取某个实例即可。

# 工厂方法模式 Factory Method Pattern

要理解工厂方法模式，需要先从**简单工厂模式（Simple Factory Pattern）**开始了解。

工厂模式是用new方法创建对象的替代，使用工厂模式，专门定义一个类，用于创建其他类的实例，这些类往往都有者共同的父类。这样一来创建这些类时就可以直接调用对应的工厂方法，只需输入正确的参数，就能创建正确的类，将具体创建类的操作交给工厂类来完成。

至于为什么要这么做，我到现在也没搞清楚，我觉得网上说的所谓的好处都比较牵强：

- 如果所有对象都用new创建，大量使用此对象时，假如对象的构造方法或者类名等发生了修改，会造成一连串的修改。根据迪米特原则，应当尽量少的和其他对象交互、知道他们的细节。所以如果使用工厂模式，有一天出现了修改，则只需要修改工厂方法，不需要大规模修改。

  但问题是，你参数名修改了怎么办呢？或者再进一步，下面的工厂方法模式，要给工厂传相应的类.class，那你这个类变量，所有调用处不还是得改吗？这也没解决修改扩散的问题啊？通过向工厂方法输入参数创建和直接new这两种方法我没感受到本质区别，都依旧是强耦合。

- 利用工厂方法创建，则可以在创建的过程中定义一系列的判断、以及伴随创建的动作。

  但这个其实new一个对象也可以实现吧？只需要在对应构造器定义判断以及伴随对象实例化要做的事即可。

- 工厂模式可以管理对象的创建、重复利用对象。把创建的对象保存在一个集合内集中管理。在用户利用工厂创建时，根据用户对产品的请求，在集合里按照一定的规则判断是否已经存在一个这样的对象，存在就可以直接返回，否则创建一个新的加入集合并返回给客户端。

  这一点我觉得也有点扯，因为似乎单例模式的扩展——多例模式，一样能做这件事儿......只需要动态更新实例数量限制即可。

不足：

- 简单工厂模式由于使用了静态工厂方法，造成工厂角色无法形成基于继承的等级结构。（Java中类的静态方法无法被继承）

- 如果只是按照上面那种模式，思考这样一种情景。一个类有很多子类，现在在这个类中通过工厂方法，根据参数创建对应类型的子类。但问题是假如调用者需要创造的子类，目前还不存在怎么办？那他是不是就需要去添加子类、修改工厂方法。但这就违反了开闭原则，我们应当对修改关闭。那么我们就要对拓展开放，该如何开放增强扩展性呢？使用通过泛型实现的工厂方法模式。

---

**原话：**Define an interface for creating an object,but let subclasses decide which class to instantiate.Factory Method lets a class defer instantiation to subclasses.（定义一个用于创建对象的接口，让子类决定实例化哪一个类。工厂方法使一个类的实例化延迟到其子类。）

**要点：**这是对上面模式的优化。同样的，某一类各种类型对象不通过new来创建。但也不再利用单一工厂类负责创建：先创建一个抽象的总的工厂类，再定义具体某一种类对象对应的工厂类，这个类继承并实现那个总的抽象工厂类中的方法。这个方法均采用泛型来定义，这样调用的时候直接传入对应类即可轻松实现要创建类的种类扩展。

除此之外，要是后续需要新增的类在创建流程上有特殊之处，和原先的方法不通用，还可以继承这个总的抽象工厂类，去实现一个对应的工厂类即可完成扩展。

**好处：**

- 方便测试：测试一个A类需要把他依赖的B类产生出来，这时候就可以使用工厂方法把B类虚拟出来。目前由于JMock和EasyMock的诞生，该使用场景已经弱化了。

- 可以在不修改原有工厂类的情况下由代码的使用者按需引入新的类，并且依旧按照这种模式来创建实例，更加符合开闭原则。

  但这是当对于简单工厂模式的优点。我依旧不能理解这种模式解决了什么样的问题。

**扩展：**

- 用泛型实现简单工厂模式，但不具备工厂方法模式的接口。
- 使用多个工厂类。哪怕不是对原有代码扩展，也令每一个实现类对应一个扩展类，不要把多个类的创建都写在一个扩展类中，不然代码结构不清晰。

# 抽象工厂模式 Abstract Factory Pattern

**原话：**Provide an interface for creating families of related or dependent objects without specifying their concrete classes.（为创建一组相关或相互依赖的对象提供一个接口，而且无须指定它们的具体类。）

**要点：**它其实依旧是工厂方法模式的一种升级，主要是为了解决一工厂模式构建一组对象的问题。

思考这样一种场景，利用工厂模式构建汽车，汽车有着各种车型，例如SUV、MPV、轿车、皮卡等等，对于这样一种需求，我们使用工厂方法模式就能够很容易构建：抽象类/接口是汽车，实现类就是各种车型，再定义一个抽象工厂类，然后由每种车对应的工厂类去实现这个总的抽象了。

现在我们让这种场景变得复杂一些——现在需要利用工厂模式构建车的各种零件，并且不同车型对某种零件的需求、零件之间搭配的需求也不同，改如何实现呢？

根据这个场景，可以体会到，抽象工厂模式解决的是一种更高维度的问题：

1. 此时要利用工厂模式生产的产品，由一维变成了二维
2. 原先只有一个抽象类，差异主要是在这个抽象类的多个实现类种产生的。此时除了实现类，抽象类也有多个
3. 需求是同时构建这些不同抽象类对应的实现类的实例，并且不同的抽象类对应的实例之间存在约束关系。

自然而然想到的一种解决方法就是：

1. 利用一段代码，根据更高一级实体之间的约束关系，集中创建多个抽象类对应的实例。

2. 同时应当考虑扩展，未来约束发生了变化怎么办？未来高一级的实体之间产生了新的关系怎么办（用前面的场景举例就是车的种类未来有新增怎么办）？
3. 那么利用怎样一段代码也就十分清楚了——使用接口。
4. 也就是把多个类的构造都塞到一个工厂类的实现类中，根据某一种约束关系，这个实现类对于每一种产品类，只能生产特定的型号。多种约束关系对应了多种工厂实现类。

这就是工厂模式的核心要点：

1. 产品类仍然由一个抽象类和多个实现类构成，但有多组。

   按照上面的情景来举例就是，每种零件对应一个抽象类，然后多个实现类是这一种类下的不同产品（例如该种零件的不同品牌、型号）。

2. 抽象工厂类所约定的创建的产品，是这些多个抽象类实例的组合，确切的说，是每一种抽象类对应的实例创建方法的组合。

3. 实现工厂类继承前面的工厂类，实现每个抽象类实例化的方法，但这些方法之间存在约束，只能创造某个抽象产品类对应的某些特定的实现类的实例。

可以通过下面实例类图，和代码片段，来验证你的理解：

<img src="./pic/抽象工厂模式的通用源码类图.png" style="zoom:40%;">

```
//对应的抽象工厂类
public abstract class AbstractCreator {
	//创建A类产品
	public abstract AbstractProductA createProductA();
	//创建B类产品
	public abstract AbstractProductB createProductB();
}
```

```
//对应的两种可能的实现工厂类

public class Creator1 extends AbstractCreator {
	//生产特定种类的A类产品
	public AbstractProductA createProductA() {
		return new ProductA1();
	}
	//生产特定种类的B类产品
	public AbstractProductB createProductB() {
		return new ProductB1();
	}
}

public class Creator2 extends AbstractCreator {
	//生产特定种类的A类产品
	public AbstractProductA createProductA() {
		return new ProductA2();
	}
	//生产特定种类的B类产品
	public AbstractProductB createProductB() {
		return new ProductB2();
	}
}
```

**工厂方法模式是抽象工厂模式的极度退化情形：**

你看到这一定会发觉，这不就是工厂方法模式嘛！是的，他们在大的结构，抽象工厂类、实现工厂类、产品类上，基本是一摸一样的，唯一的区别就是抽象工厂模式的工厂类里面装了创建好几种类的方法，而工厂方法模式一般只有一种。

**坏处与特点：**这一模式除了解决了一种额外的需求之外，有没有什么副作用呢？

显而易见地，抽象工厂模式下，产品类的种类扩展十分困难。就比如上面的实例图和代码，有A、B两个类，但假如有一天我像添加一个类C，这是几乎不可能地。因为这导致了现有所有接口的变更，而接口是不宜变更了，一大堆使用该接口的调用处都得跟着变。

但对于某一抽象类下的实现类，或者是各种类之间的约束关系/工厂类，还是可以随意扩展的，只需新增对应的产品实现类、工厂实现类即可。

# 建造者模式 Builder Pattern

也叫生成器模式。

**原话：**Separate the construction of a complex object from its representation so that the same construction process can create different representations.（将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。）

**用途：**用于解决复杂对象的构造，同时将对象子组之间的构造和装配分离。

这种复杂对象：

1. 通常有着诸多复杂的子组件；
2. 并且这些子组件的构建顺序还会对结果产生影响；
3. 除此之外，这些对象内部还存在一个方法对其他诸多内部方法的调用，但调用的顺序会随着实际情形而变化。

**要点/详解/实现：**通过以下几种情形来逐步说明，建造者模式是一种怎样的模式，具体解决了一种什么样的情形。

1. 有这样一种情形，一个对象，这个对象拥有一大堆的属性（10个以上），例如用户注册场景下新增一个用户，用构造函数创建，那自然要往这个构造函数里传一大堆的参数。这样做非常的墨迹：调用者要对着文档一个个看是哪个参数，然后参数漏了也不行，哪天参数改了，这就更麻烦了，这样写出来也不易读。

   这还不是最墨迹的，假如要创建的这个对象的内部对象还是对象，且这些对象一样复杂，你用构造函数传是不是就更墨迹了。

   在此基础上，假如对象的构建存在某种次序，亦或者是某些情形只需要极个别对象。假设属性个数为n，那你最坏情况就得把这个对象的构造函数重载n的立方次！

   所以就产生了setter这种东西，直接new一个空对象，通过相应的`setter()`方法设置某一个值，这样就彻底解决了前面的问题，使对象的构架变得灵活多了。

   但现在依旧无法解决全部问题，首先setter一般是用来简单的对对象的值进行设置，那假如这个类的某个子模块也是个复杂对象呢？我们需要一个代码段执行对子模块的构建。除此之外，假如有十几个setter，一个个调用，看起来似乎也很乱吧？除此之外，有时我们还希望指定这个类的对象的某个方法内部的执行顺序。

   所以我们通过**Builder建造器**替代setter，创建一个Builder类。

   - 这个类具有一个私有属性——建造器要创建的产品类的实例，每次建造器被创建的时候，也同时调用构造函数创建一个相应的产品类实例。
   - 这个类下面有诸多建造方法，负责指定产品类的每个子模块的构建，在这个私有属性的基础上进行构建。
   - 除此之外还有一类建造方法，用来指定产品类的某个方法内部的执行次序。（一种可供参考的实现方法：产品类内维护一个数组，用来记录某个方法内部的执行流程，调用这个方法就是遍历这个数组，依次执行相应流程。）
   - 每个建造方法都返回this，也就是建造器实例本身，这样就可以链式调用各种方法，比setter更简洁。

   思考或查阅Java中StringBuilder的用法，可以验证你对上面内容的理解。

2. 但仅有构造器，仍然无法满足我们的需求：

   1. 产品类有可能是一个产品族，是一个抽象类对应多个实现类的那种结构。
   2. 我们希望建造器具体一定的可扩展性。

   所以需要实现多个建造器，此时的结构就变为了，一个**Builder接口或抽象类**，然后对应了多个**concreteBuilder实现类**。

   此时构造这个复杂对象的流程就变为了：用concreteBuilder创建一个Builder→按顺序调用所需要的建造方法→获取所创建的产品类，完成创建。

3. 此时我们只是解决了，要创建的产品类过于复杂，以及利用建造方法对内部子模块构建的封装。但构建过程依旧复杂且手动。你可以思考以下在“2”最后提到的流程中，创建一个类仍然需要链式调用一大串。

   现在需要对这一组装过程进行封装，避免需要创建复杂对象的高层模块，涉及过多构建该对象的内容，侵入到建造者内部的实现类。我们希望这种复杂对象的创建，也能像直接new一个那样简单，让高层模块和建造者有一条边界。

   所以引入一个新的角色**Director导演类**，由它将各种典型的创建流程进行封装。这个类内部由各种各样的getProduct()方法来获取各种产品类，这些方法通过“2”中的那个流程，操纵建造器来实现。这样我们就完成了对组装的封装。

   至此，我们就实现了完整的建造者模式。

4. 总结：

   - 有了上面做的这些工作，现在我们想要创建某个类的特定实例，只需要创建一个Director实例，然后调用相应的getProduct方法，就能完成对这个复杂对象的构建。**过程极其简单，根本无需了解、参与构建的细节。**
   - 同时这种模式，还实现了对**复杂对象的拆解，复杂对象的表示放在相应的产品类中，复杂对象内部成员的构建放在了Builder中，复杂对象的组装放在了Director中，结构清晰明了。**

**通用类图：**

<img src=".\pic\建造者模式通用类图.PNG" style="zoom:50%;" />

- Product产品类

  这是要创建的复杂对象，它可以是有着抽象类和继承抽象类的实现类结构的一类产品，由多个类。

- Builder抽象建造者

  规范产品的组建，一般是由子类实现。

- ConcreteBuilder具体建造者

  实现抽象类定义的所有方法，负责构建各个零件，并返回一个组建好的对象。

- Director导演类

  负责安排已有模块的顺序，调用ConcreteBuilder完成构建，完成组装，最后调用ConcreteBuilder获取组件好的结果。

  起到封装的作用，让高层模块和建造者内部有一条明确的边界，不要和Builder产生太多的纠葛，导致结构不清晰。实现能像new一个对象那样只通过一行代码就实现复杂对象的创建。

  建造者模式比较庞大时，导演类也可以有多个。

**优点：**

- 封装性

  在没有这种东西之前，我们创建一个复杂的对象，可能也没有使用new或者setter，但是就直接写在了高层的业务代码中。现在使用这种模式，可以让使用者不必知道产品内部组成的细节。

- 易于扩展

  独立的建造者实现类、和接口，使得扩展非常容易。

- 便于控制细节风险

  具体的建造者是独立的，因此可以对建造过程逐步细化，而不对其他模块产生任何影响。

# 原型模式 Prototype Pattern

**原话：**Specify the kinds of objects to create using a prototypical instance,and create new objects by copying this prototype.（用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。）

**通俗的说：**原型模式就是先生产出一个类的一些包含大量共有信息的实例，然后根据已有的实例，通过拷贝副本、修正细节信息，来建立新对象。

**原型模式的通用类图：**

<img src=".\pic\原型模式通用类图.PNG" style="zoom: 25%;" />

**实现要点：**原型模式的核心就是实现一个clone方法，通过该方法进行拷贝，基于一个已经有的实例来创建其他的该类对象。

- 深拷贝与浅拷贝：Java的clone只会执行浅拷贝，所以如果要进行深拷贝，在覆盖的过程中除了调用`super.clone()`，还需要额外对新拷贝的对象，将哪些需要深拷贝的成员额外赋值，递归调用这些对象的clone方法。除此之外也可以通过二进制流操作对象，实现深拷贝。

  > ps：深拷贝的实现是递归的，对于当前对象的深拷贝，只能完成到第二层，也就是确保要拷贝对象的引用类型的成员被复制，但如果这个成员内部还有引用类型的成员，是一堆对象嵌套的情形，那就需要在相应的对象里实现cloneable接口。

- clone与final：如果你想实现深拷贝，那么clone和final类型的引用类型成员就是冲突的，会报错。因为你深拷贝是通过先执行浅拷贝，然后对引用类型的成员重新赋值，调用它自己的clone方法实现的。但它要是被申明成final，还怎么重新赋值？

**特点/用途：**

- 性能优良

  以Java语言为例，通过覆盖cloneable接口实现clone方法，该方法是直接在内存的二进制流中拷贝，比直接new一个对象性能好很多，特别是在一个循环体内产生大量对象的时候，这种方法优势明显。

  适用于一些创建的时候需要消耗很多资源、需要做非常繁琐的数据准备、访问权限鉴定的场景。

- 逃避构造函数的约束

  是优点也是缺点，直接在内存中拷贝，构造函数是不会执行的。好处就是减少了约束，坏处也是减少的这些约束在实践中可能会带来问题，要结合实际情况考虑。

- 一个类，多个原型

  如果说只是根据原型模板创建的话，似乎直接设置默认值然后new就可以了，但上面两点使得原型模式、用clone创建对象成为了必要。

  但除此之外，原型模式还有一功能，使之区别于new。假如有多套需要共有的信息呢？使用原型模式，可以基于多种典型值，创建一个类的多个原型，然后再在这些原型的基础上去创建对象，形成多个分支。

**和其他模式组合使用：**实践中，原型模式很少单独使用，一般是和工厂方法模式一起出现，工厂方法通过clone生产对象。

