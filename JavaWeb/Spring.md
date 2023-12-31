# 一些前置概念

> 不理解也没关系，有个印象就好。

**Plain Old Java Object（POJO）**

- 简单的Java对象。
- 定义：没有继承任何类、没有实现任何接口、没有被框架侵入、没有业务方法、不携带connection之类的方法。
- 举例：JavaBeans。

**JavaBean**

- 定义：是一种Java语言写成的可重用的组件。他是一种规范。

- 符合如下约定：

  - 所有属性为private。
  - 提供无参数构造器。
  - 属性使用getter和setter方法访问。
  - 可序列化，实现serializable接口。

- 区别：算是遵循特定命名约定的POJO。

  | POJO                                           | JavaBean                                             |
  | ---------------------------------------------- | ---------------------------------------------------- |
  | 除了Java强加的限制以外没有其他特殊限制。       | 是一个特殊的POJO，有一些额外的限制。                 |
  | 不做要求。                                     | 提供对成员的完全控制(getter和setter)。               |
  | 不做要求。                                     | 要求实现可序列化接口。                               |
  | 不做要求。                                     | 字段只能是private，然后用方法访问。                  |
  | 不做要求。                                     | 必须有无参构造函数。                                 |
  | 当你不想限制成员并让用户访问你的实体时使用它。 | 当你要向用户提供你的实体，但仅提供实体的一部分服务。 |

**SpringBean**

- 定义：受Spring管理的对象，通过配置由Spring自动实例化，用完后自动销毁的对象。
- 和JavaBean区别：
  1. 用途：JavaBean更多用于值传递参数；SpringBean更为广泛。
  2. 结构：JavaBean作为值对象，要求每个属性都由getter和setter；但SpringBean只需为接受值注入的属性提供setter方法。
  3. 生命周期：JavaBean作为值对象传递，不接受任何容器管理其生命周期；SpringBean由Spring管理其生命周期行为。

**EntityBean**

- 定义：领域模型对象，用于实现O/R映射，负责将数据库中的表记录映射为内存中的Entity对象。

  > 如何理解这种映射呢，事实上创建一个EntityBean对象相当于创建一条数据库记录，删除则相当于删除记录，修改一个EntityBean时，容器会自动将EntityBean的状态和数据库同步。

# IOC容器

将对象放入，用于管理对象的生命周期、对象和对象之间的依赖关系。

为什么要把实例交给Spring，交给它的IOC容器？

- Inversion of Control（IOC）控制反转

**S1：源数据**

将配方告诉Spring，然后Spring就会按照你给的这个配方创建并管理Bean。