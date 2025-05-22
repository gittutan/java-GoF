# 模块名称

工厂方法模式 (Factory Method Pattern)

# 核心概念 (针对Java初学者)

**什么是工厂方法模式？**

工厂方法模式（Factory Method Pattern）是一种创建型设计模式。它的核心思想是：**定义一个用于创建对象的接口（我们称之为“工厂方法”），但是让实现这个接口的子类来决定到底实例化哪一个具体的类。** 这样，工厂方法模式将类的实例化操作延迟到了子类中去完成。

简单来说，父类只规定“要生产某种产品”，但不关心“具体生产什么牌子的产品”以及“怎么生产”。这些具体的生产细节都交给了子类工厂去实现。

**类比：汽车工厂**

一个很形象的类比是汽车工厂：

1.  **汽车工厂的蓝图 (Creator - 创建者抽象类)**:
    *   这个蓝图定义了所有汽车工厂都必须具备一个核心的“生产汽车”的方法（这就是工厂方法）。
    *   它可能还定义了一些通用的生产流程，比如“获取零件 -> 组装 -> 喷漆 -> 测试”。
2.  **具体品牌的汽车工厂 (ConcreteCreator - 具体创建者类)**:
    *   **宝马工厂 (ConcreteCreator1)**: 它是“汽车工厂蓝图”的一个具体实现。当调用它的“生产汽车”方法时，它会按照宝马的标准，使用宝马的零件，生产出一辆“宝马车”。
    *   **奔驰工厂 (ConcreteCreator2)**: 它是另一个具体实现。当调用它的“生产汽车”方法时，它会按照奔驰的标准，使用奔驰的零件，生产出一辆“奔驰车”。
3.  **汽车的规范 (Product - 产品抽象类)**:
    *   定义了所有“汽车”都应该具备的基本功能，比如能“跑”、能“开门”。
4.  **具体品牌的汽车 (ConcreteProduct - 具体产品类)**:
    *   **宝马车 (ConcreteProduct1)**: 实现了“汽车规范”，是一辆具体的宝马牌汽车。
    *   **奔驰车 (ConcreteProduct2)**: 实现了“汽车规范”，是一辆具体的奔驰牌汽车。

在这个例子中：
*   客户端（比如一个汽车经销商）想要一辆车，它不需要直接去创建“宝马车”或“奔驰车”对象。
*   经销商只需要找到一个“宝马工厂”的实例，然后调用这个工厂的“生产汽车”方法，就能得到一辆宝马车。如果它找到的是“奔驰工厂”，调用同样的方法，就能得到一辆奔驰车。
*   “生产汽车”这个方法的具体实现（即到底生产什么牌子的车）是由子工厂（宝马工厂、奔驰工厂）决定的，而不是在顶层的“汽车工厂蓝图”中写死的。

工厂方法模式通过这种方式，使得代码更加灵活，易于扩展（比如要增加“奥迪工厂”和“奥迪车”就很容易），并且将对象的创建逻辑与使用逻辑分离开来。

# 模式结构与角色分析

工厂方法模式主要包含以下角色：

## Product (产品抽象类 `framework/Product.java`)

*   **作用**: 定义了工厂方法所创建的对象的共同接口（或抽象行为）。所有具体的产品都必须是这个 `Product` 类型的子类。
*   **代码分析 (`framework/Product.java`)**:
    *   这是一个抽象类，位于 `framework` 包下，表明它是框架层的一部分。
    *   `public abstract void use();`:
        *   定义了一个名为 `use` 的抽象方法。
        *   这个方法规定了所有“产品”都必须具备“使用”这一功能。具体如何“使用”则由其子类（具体产品）来实现。
        *   例如，一张ID卡的使用方式是“刷卡”，一个U盘的使用方式是“读写数据”。

## ConcreteProduct (具体产品类 `idcard/IDCard.java`)

*   **作用**: 实现了 `Product` 接口（或继承了 `Product` 抽象类），是工厂方法实际创建的对象。
*   **代码分析 (`idcard/IDCard.java`)**:
    *   此类位于 `idcard` 包下，继承自 `framework.Product`。它代表一种具体的产品——ID卡。
    *   `private String owner;`: 存储ID卡持有者的姓名。
    *   `private Integer id;`: 存储ID卡的唯一编号。
    *   `IDCard(String owner, Integer id)`:
        *   构造函数，包级私有（`default` 访问修饰符），意味着只有在同一个包 `idcard` 内的类（即 `IDCardFactory`）才能直接创建 `IDCard` 的实例。这是一种封装，防止外部直接实例化具体产品，强制通过工厂来创建。
        *   `System.out.println("Make " + owner + "'s card");`: 打印一条日志，表示正在制作一张卡。
        *   初始化 `owner` 和 `id` 属性。
    *   `@Override public void use()`:
        *   实现了父类 `Product` 的 `use` 方法。
        *   `System.out.println("Use " + owner + "'s card");`: 打印一条日志，表示正在使用这张ID卡。
    *   `public String getOwner()`: 返回ID卡持有者的姓名。
    *   `public Integer getId()`: 返回ID卡的编号。
    *   这两个 `get` 方法主要用于 `IDCardFactory` 在注册产品时获取信息。

## Creator (创建者抽象类 `framework/Factory.java`)

*   **作用**:
    *   声明了“工厂方法”（在本例中是 `createProduct`），该方法返回一个 `Product` 类型的对象。
    *   `Creator` 类也可以包含一些与产品创建相关的通用逻辑，或者调用工厂方法来创建一个产品对象。
    *   它将具体产品的实例化操作延迟到其子类（`ConcreteCreator`）中。
*   **代码分析 (`framework/Factory.java`)**:
    *   这是一个抽象类，位于 `framework` 包，是所有具体工厂的父类。
    *   `public final Product create(String owner)`:
        *   这是一个**模板方法**（Template Method Pattern 的应用）。它被声明为 `final`，因此子类不能覆盖这个方法的执行流程。
        *   它定义了创建产品的标准流程：
            1.  `Product p = createProduct(owner);`: 调用抽象的“工厂方法” `createProduct` 来实际创建一个具体的产品。传入 `owner` 作为参数。
            2.  `registerProduct(p);`: 调用抽象的 `registerProduct` 方法来注册（或记录）刚刚创建的产品 `p`。
            3.  `return p;`: 返回创建并注册好的产品。
        *   客户端通常调用这个 `create` 方法来获取产品，而不是直接调用 `createProduct`。
    *   `protected abstract Product createProduct(String owner);`:
        *   **核心的工厂方法**。它被声明为 `protected abstract`。
        *   `protected` 意味着它只能被同一个包内的类或其子类访问（在本例中，主要是被子类 `IDCardFactory` 实现）。
        *   `abstract` 意味着这个方法没有具体的实现，必须由子类（具体工厂）来提供实现，以决定到底创建哪种具体类型的 `Product`。
    *   `protected abstract void registerProduct(Product product);`:
        *   另一个抽象方法，用于在产品创建后对其进行某种形式的“注册”或记录。
        *   具体如何注册也由子类（具体工厂）来实现。

## ConcreteCreator (具体创建者类 `idcard/IDCardFactory.java`)

*   **作用**:
    *   继承自 `Creator` (`Factory`)。
    *   重写（实现）父类中声明的工厂方法 `createProduct`，使其返回一个具体的 `ConcreteProduct`（如 `IDCard`）的实例。
    *   实现 `registerProduct` 方法，完成与该具体产品相关的注册逻辑。
*   **代码分析 (`idcard/IDCardFactory.java`)**:
    *   此类位于 `idcard` 包，继承自 `framework.Factory`。它是专门用于生产 `IDCard` 产品的具体工厂。
    *   `private List owners = new ArrayList();`: 用于存储所有已创建ID卡的持有者姓名。
    *   `private Map<String, Integer> idList = new HashMap<String, Integer>();`: 用于存储每个持有者姓名与其ID卡编号的映射。
    *   `private Integer lastId = 0;`: 用于生成递增的ID卡编号。
    *   `@Override protected Product createProduct(String owner)`:
        *   **实现了核心的工厂方法**。
        *   `return new IDCard(owner, lastId++);`:
            *   当被调用时，它会创建一个新的 `IDCard` 实例。
            *   传入 `owner` 和一个通过 `lastId++` 生成的唯一ID。
            *   这里决定了该工厂生产的具体产品是 `IDCard`。
    *   `@Override protected void registerProduct(Product product)`:
        *   **实现了产品注册方法**。
        *   `owners.add(((IDCard)product).getOwner());`: 将刚创建的 `IDCard` 的持有者姓名添加到 `owners` 列表中。
        *   `idList.put(((IDCard)product).getOwner(), ((IDCard)product).getId());`: 将持有者姓名和ID编号存入 `idList` 这个Map中。
        *   这里需要将 `Product` 类型的 `product` 向下转型为 `IDCard` 类型，以便调用 `IDCard` 特有的 `getOwner()` 和 `getId()` 方法。这暗示了 `IDCardFactory` 知道它处理的是 `IDCard` 这种具体产品。
    *   `public List getOwners()`: 返回所有ID卡持有者的列表。
    *   `public Map getIdList()`: 返回持有者与ID的映射表。
    *   这两个 `get` 方法是 `IDCardFactory` 特有的，用于获取其内部管理的产品信息，不属于 `Factory` 的通用接口。

## Client (客户端 `Main.java`)

*   **作用**: 客户端代码通常只与 `Product` 的抽象接口和 `Creator` (`Factory`) 的抽象接口交互。它通过 `Creator` 的实例来请求创建 `Product` 对象，然后使用 `Product` 对象。客户端不需要知道具体是哪个 `ConcreteProduct` 被创建，也不需要知道是哪个 `ConcreteCreator` 在负责创建。
*   **代码分析 (`Main.java`)**:
    *   `Factory factory = new IDCardFactory();`:
        *   客户端创建了一个具体的工厂实例 `IDCardFactory`，并将其赋值给 `Factory` 类型的引用 `factory`。
        *   这里，客户端虽然实例化了具体的工厂，但后续的操作都是通过 `Factory` 接口进行的。
    *   `Product card1 = factory.create("Nobita");`:
        *   客户端调用 `factory` 的 `create` 方法来创建一个产品。传入所有者姓名 "Nobita"。
        *   由于 `factory` 的实际类型是 `IDCardFactory`，所以 `factory.create()` 内部会调用 `IDCardFactory` 实现的 `createProduct("Nobita")`（返回一个 `IDCard` 对象）和 `registerProduct(card)`。
        *   返回的 `Product` 对象（实际上是 `IDCard` 的实例）被赋值给 `Product` 类型的引用 `card1`。
    *   `Product card2 = factory.create("Takeshi");`
    *   `Product card3 = factory.create("Shizuka");`
        *   类似地创建另外两张卡。
    *   `card1.use();`
    *   `card2.use();`
    *   `card3.use();`
        *   客户端通过 `Product` 接口的 `use()` 方法来使用这些产品，而不需要知道它们具体是 `IDCard`。

通过这种方式，如果将来需要生产另一种类型的产品（比如“学生证” `StudentCard`），只需要：
1.  创建一个 `StudentCard` 类继承 `Product`。
2.  创建一个 `StudentCardFactory` 类继承 `Factory`，并实现 `createProduct` 和 `registerProduct` 来生产和注册 `StudentCard`。
3.  客户端代码在创建工厂时，将 `new IDCardFactory()` 改为 `new StudentCardFactory()` 即可，其余使用产品的代码（如 `product.use()`）通常不需要改变。

# 工厂方法模式的优点

1.  **良好的封装性，代码结构清晰**:
    *   将对象的创建过程封装在具体的工厂子类中，与产品的使用代码分离。
    *   客户端只需要知道抽象工厂和抽象产品的接口，而不需要关心具体的创建细节。

2.  **扩展性好**:
    *   当需要引入新的产品类型时，只需要添加一个新的具体产品类和一个与之对应的具体工厂类即可。
    *   不需要修改已有的工厂类或产品类，也不需要修改客户端调用工厂的代码（除非客户端需要显式选择新工厂）。这符合**开闭原则**（对扩展开放，对修改关闭）。

3.  **灵活性高**:
    *   `Creator` (父工厂) 的子类 (`ConcreteCreator`) 可以自由地选择创建何种具体产品，以及如何创建。
    *   例如，一个工厂方法可以根据参数或配置文件的不同，返回不同的具体产品实例。

4.  **更松散的耦合**:
    *   产品创建的逻辑被隔离在具体的工厂实现中，而不是直接硬编码在客户端或产品使用的地方。

# 与抽象工厂模式的区别 (简要)

*   **工厂方法模式 (Factory Method Pattern)**:
    *   **关注点**: 主要关注**单个产品**的创建。它提供一个工厂方法来创建一个类型的对象。
    *   **结构**: 通常是一个抽象工厂类对应多个具体工厂子类，每个具体工厂子类负责创建一个具体产品。存在一个与产品平行的工厂层次结构。
    *   **目的**: 将一个产品的实例化延迟到子类。

*   **抽象工厂模式 (Abstract Factory Pattern)**:
    *   **关注点**: 主要关注创建**一系列相关或相互依赖的产品对象（一个产品族）**。
    *   **结构**: 通常是一个抽象工厂接口定义了创建多个不同类型产品的方法（如 `createProductA()`, `createProductB()`）。每个具体工厂实现这个接口，以创建特定“风格”或“系列”的一整套产品。
    *   **目的**: 提供一个接口，用于创建一系列相关或相互依赖的对象，而无需指定它们具体的类。

简单来说：
*   **工厂方法**：“我需要一个X产品，请某个能生产X的工厂给我造一个。” （一个工厂通常只负责一种主要产品）
*   **抽象工厂**：“我需要一套Y风格的配件（比如Y风格的鼠标、Y风格的键盘、Y风格的显示器），请Y风格的工厂把这些都给我造出来。” （一个工厂负责生产一整套相关的产品）

# 适用场景

工厂方法模式主要适用于以下情况：

1.  **当一个类不知道它所必须创建的对象的类的时候。**
    *   类希望通过其操作创建对象，但希望将选择具体对象类型的工作推迟到其子类。
2.  **当一个类希望由它的子类来指定它所创建的对象的时候。**
    *   父类只定义创建对象的接口（工厂方法），具体创建哪个对象由子类决定。
3.  **当类将创建对象的职责委托给多个帮助子类中的某一个，并且你希望将哪一个帮助子类是代理者这一信息局部化的时候。**
    *   例如，根据不同的配置或环境，选择不同的具体工厂来创建对象。
4.  **需要封装对象的创建过程，使得系统在产品类型扩展时具有良好的灵活性和可维护性。**

# 给Java初学者的提示

*   **核心在于“延迟实例化到子类”**:
    *   父类 `Factory` (Creator) 定义了创建产品的大致流程（在 `create` 方法中）和创建产品的抽象接口（`createProduct` 方法）。
    *   具体的实例化逻辑（即 `new ConcreteProduct()`）是在子类 `IDCardFactory` (ConcreteCreator) 的 `createProduct` 方法中实现的。

*   **`Factory.create` 方法中的 `createProduct` 和 `registerProduct` 调用展示了模板方法模式的应用**:
    *   `Factory.create()` 方法定义了一个算法的骨架（先创建产品，再注册产品）。
    *   这个算法中的某些步骤（`createProduct` 和 `registerProduct`）被声明为抽象的，由子类去实现。
    *   这是模板方法模式的一个典型应用场景，父类定义流程，子类实现细节。

*   **注意区分“工厂方法”（`createProduct`）和调用它的公共方法（`create`）**:
    *   **工厂方法 (`createProduct`)**: 通常是 `protected` 和 `abstract` (或可被覆盖的 `protected` 方法)。它是模式的核心，由子类实现以创建具体产品。
    *   **调用工厂方法的公共方法 (`create`)**: 通常是 `public` 和 `final`。这是客户端实际调用的方法，它封装了调用工厂方法以及可能进行的其他通用处理（如本例中的 `registerProduct`）。

*   **客户端通常只与 `Factory` (Creator) 和 `Product` 的抽象层面交互**:
    *   在 `Main.java` 中，虽然 `new IDCardFactory()` 实例化了具体工厂，但之后它被赋值给 `Factory factory` 类型的引用。
    *   同样，`factory.create()` 返回的产品也被赋值给 `Product card1` 类型的引用。
    *   这使得客户端代码不直接依赖于具体的产品类和具体工厂类（除了初始化的那一行），从而提高了灵活性。

工厂方法模式通过将对象的创建过程推迟到子类，提供了一种优雅的方式来创建对象，同时保持了代码的灵活性和可扩展性。
