# 模块名称: 工厂方法模式 (Factory Method Pattern)

## 1. 核心概念 (针对Java初学者)

**什么是工厂方法模式？**

工厂方法模式（Factory Method Pattern）是一种创建型设计模式。它的核心思想是：**定义一个用于创建对象的接口（这个接口就是所谓的“工厂方法”），但是让实现这个接口的子类来决定到底实例化哪一个具体的类。** 这也就是说，工厂方法模式将类的实例化操作延迟到了子类中去完成。

客户端代码通常与抽象的创建者（Creator）和抽象的产品（Product）打交道，而不需要关心具体是哪个子类创建者创建了哪个具体的产品。

**生活中的类比：汽车工厂**

*   **汽车 (Product - 产品接口/抽象类)**: 定义了汽车的通用特性或行为，比如 `run()`。
*   **宝马车 (ConcreteProduct1 - 具体产品1)**: 实现了“汽车”接口，是宝马品牌的汽车。
*   **奔驰车 (ConcreteProduct2 - 具体产品2)**: 实现了“汽车”接口，是奔驰品牌的汽车。
*   **汽车工厂 (Creator - 创建者接口/抽象类)**: 定义了一个抽象的“生产汽车”的方法（这就是工厂方法，例如 `createCar()`）。这个工厂本身可能还有一些通用的造车流程（比如车辆登记）。
*   **宝马工厂 (ConcreteCreator1 - 具体创建者1)**: 继承“汽车工厂”，并具体实现“生产汽车”的方法，专门生产“宝马车”。
*   **奔驰工厂 (ConcreteCreator2 - 具体创建者2)**: 继承“汽车工厂”，并具体实现“生产汽车”的方法，专门生产“奔驰车”。

当客户想要一辆宝马车时，他会去找宝马工厂。当想要一辆奔驰车时，他会去找奔驰工厂。客户不需要自己去组装零件，工厂负责生产出完整的汽车。

## 2. 模式结构与角色分析

本项目的 `FactoryMethod/` 目录（包含 `framework/` 和 `idcard/` 子目录）清晰地展示了工厂方法模式的结构。

*   **Product (产品抽象类 `framework/Product.java`)**:
    *   **作用**: 定义了工厂方法所创建的对象的共同接口或抽象父类。所有具体产品都必须是这个类型的子类。
    *   **代码分析 (`framework/Product.java`)**:
        ```java
        package framework;
        public abstract class Product {
            public abstract void use(); // 定义了产品的一个抽象行为：使用
        }
        ```
        `Product` 类非常简单，只定义了一个抽象方法 `use()`，具体的“产品”需要实现这个方法来表明它们如何被使用。

*   **ConcreteProduct (具体产品类 `idcard/IDCard.java`)**:
    *   **作用**: 实现了 `Product` 接口（或继承自 `Product` 抽象类）。这是工厂方法实际创建的对象。
    *   **代码分析 (`idcard/IDCard.java`)**:
        ```java
        package idcard;
        import framework.*; // 导入Product类

        public class IDCard extends Product {
            private String owner;
            private Integer id;

            // 构造函数是包可见性 (package-private)，通常意味着只有同包的工厂类才能创建它
            IDCard(String owner, Integer id) {
                System.out.println("Make " + owner + "'s card with ID: " + id);
                this.owner = owner;
                this.id = id;
            }

            @Override
            public void use() {
                System.out.println("Use " + owner + "'s card (ID:" + id + ")");
            }

            public String getOwner() { return owner; }
            public Integer getId() { return id; }
        }
        ```
        `IDCard` 是一个具体的产品，代表一张身份证。它有所有者（`owner`）和ID号（`id`）。它的 `use()` 方法会打印出使用该卡片的信息。

*   **Creator (创建者抽象类 `framework/Factory.java`)**:
    *   **作用**: 声明了“工厂方法”（`createProduct`），该方法返回一个 `Product` 类型的对象。`Creator` 类也可以包含其他依赖于由工厂方法创建的产品的操作。通常，`Creator` 中的工厂方法是由其子类来实现的。
    *   **代码分析 (`framework/Factory.java`)**:
        ```java
        package framework;
        public abstract class Factory {
            // 这是客户端主要调用的方法，它定义了创建产品的完整流程/骨架
            // 注意它是 final 的，子类不能修改这个流程
            public final Product create(String owner) {
                Product p = createProduct(owner); // 1. 调用工厂方法创建产品
                registerProduct(p);             // 2. 注册产品
                return p;                       // 3. 返回产品
            }

            // 工厂方法：由子类实现，负责具体创建哪个Product对象
            protected abstract Product createProduct(String owner);

            // 注册产品的方法：也由子类实现，负责如何处理创建出来的Product
            protected abstract void registerProduct(Product product);
        }
        ```
        `Factory` 类定义了一个公共的 `create` 方法。这个方法不是工厂方法本身，但它调用了两个抽象方法：
        1.  `createProduct(String owner)`: **这才是真正的工厂方法**。它被声明为 `protected abstract`，意味着具体的实例化逻辑被推迟到子类中。
        2.  `registerProduct(Product product)`: 这是一个辅助方法，也由子类实现，用于在产品创建后对其进行某种形式的注册或记录。
        这种结构（一个公共的final方法调用若干抽象的protected方法）是**模板方法模式**的一个体现。

*   **ConcreteCreator (具体创建者类 `idcard/IDCardFactory.java`)**:
    *   **作用**: 继承自 `Factory` 类，并重写（实现）其抽象的工厂方法 `createProduct`，以返回一个具体的 `ConcreteProduct` 实例。它也需要实现 `registerProduct` 方法。
    *   **代码分析 (`idcard/IDCardFactory.java`)**:
        ```java
        package idcard;
        import framework.*;
        import java.util.*;

        public class IDCardFactory extends Factory {
            private List<String> owners = new ArrayList<>(); // 用于记录所有者的列表
            private Map<String, Integer> idList = new HashMap<>(); // 用于记录所有者和对应ID的映射
            private Integer lastId = 0; // 用于生成唯一的ID

            @Override
            protected Product createProduct(String owner) {
                // 实际创建 IDCard 对象，并分配一个新的ID
                return new IDCard(owner, lastId++);
            }

            @Override
            protected void registerProduct(Product product) {
                // 将产品转换为 IDCard 类型（因为本工厂只生产IDCard）
                IDCard card = (IDCard) product;
                owners.add(card.getOwner());
                idList.put(card.getOwner(), card.getId());
                System.out.println("Register " + card.getOwner() + "'s card (ID:" + card.getId() + ")");
            }

            public List<String> getOwners() { return owners; }
            public Map<String, Integer> getIdList() { return idList; }
        }
        ```
        `IDCardFactory` 是一个具体的工厂，它知道如何创建 `IDCard`。
        *   `createProduct` 方法实例化并返回一个新的 `IDCard`。
        *   `registerProduct` 方法将创建的 `IDCard` 的所有者和ID记录在内部的列表中。

*   **Client (客户端 `Main.java`)**:
    *   **作用**: 客户端代码通常只与 `Product` 和 `Factory` 的抽象接口打交道。它通过调用 `Factory` 实例的 `create` 方法来获取 `Product` 对象，而不需要知道具体是哪个 `ConcreteProduct` 被创建了，也不需要知道是哪个 `ConcreteFactory` 创建了它（尽管在本例中客户端直接实例化了 `IDCardFactory`）。
    *   **代码分析 (`Main.java`)**:
        ```java
        import framework.*;
        import idcard.*; // 导入具体工厂和产品 (虽然理想中客户端可能只知道framework)
        // ...
        public class Main {
            public static void main(String[] args) {
                // 客户端创建一个具体的工厂实例
                Factory factory = new IDCardFactory();

                // 通过工厂的create方法创建产品，客户端面向Product接口编程
                Product card1 = factory.create("Nobita");
                Product card2 = factory.create("Takeshi");
                Product card3 = factory.create("Shizuka");

                // 使用产品
                card1.use();
                card2.use();
                card3.use();

                // 可以通过具体工厂类型获取额外信息 (如果需要的话)
                if (factory instanceof IDCardFactory) {
                    IDCardFactory idFactory = (IDCardFactory) factory;
                    System.out.println("Registered owners: " + idFactory.getOwners());
                    System.out.println("ID List: " + idFactory.getIdList());
                }
            }
        }
        ```
        `Main` 类首先创建了一个 `IDCardFactory` 的实例，然后调用其 `create` 方法来生产多张 `IDCard`。客户端通过 `Product` 接口来使用这些卡片。

## 3. 工厂方法模式的优点

*   **良好的封装性，代码结构清晰**: 创建对象的代码（在具体工厂中）与使用对象的代码（在客户端中）分离。客户端不需要知道对象是如何被创建的。
*   **优秀的扩展性**: 当需要增加新的产品时，只需要创建一个新的具体产品类和对应的具体工厂类即可。原有的工厂类和产品类不需要修改，这符合“开闭原则”（对扩展开放，对修改关闭）。
*   **高度的灵活性**: 父类 `Creator` 只是给出了创建产品的接口（工厂方法），而具体创建什么产品、如何创建，则由子类 `ConcreteCreator` 决定。这使得系统更加灵活。
*   **解耦**: 将产品创建的责任从客户端代码中移除，降低了客户端与具体产品实现之间的耦合。

## 4. 与抽象工厂模式的区别 (简要)

虽然都是创建型模式，但它们的意图和结构有所不同：

*   **工厂方法模式 (Factory Method)**:
    *   **目的**: 定义一个创建对象的接口，但让子类决定实例化哪个类。它主要解决单个对象的创建问题。
    *   **结构**: 通常涉及一个产品接口/抽象类，多个具体产品类；一个创建者接口/抽象类（包含工厂方法），多个具体创建者类。每个具体创建者通常只负责创建一个特定的具体产品。
    *   **层级**: 主要处理一个“产品等级结构”。

*   **抽象工厂模式 (Abstract Factory)**:
    *   **目的**: 提供一个接口，用于创建一系列相关或相互依赖的对象（即一个“产品族”），而无需指定它们具体的类。
    *   **结构**: 通常涉及多个产品接口/抽象类（形成产品族），每个接口有多个具体实现；一个抽象工厂接口（包含多个创建不同产品的方法），多个具体工厂类。每个具体工厂负责创建一整套特定风格（族）的产品。
    *   **层级**: 主要处理多个“产品族等级结构”。

简单来说：工厂方法是“我要一个X产品，具体是X的哪种子类型由子工厂决定”，而抽象工厂是“我要一套Y风格的组件，其中包含A、B、C等多种产品，具体是Y风格的A、B、C由具体工厂决定”。

## 5. 适用场景

*   **当一个类不知道它所必须创建的对象的类的时候。** (例如，一个框架需要创建一些对象，但具体对象的类型是在框架被使用时才确定的)。
*   **当一个类希望由它的子类来指定它所创建的对象的时候。** (这是工厂方法的核心思想)。
*   **当类将创建对象的职责委托给多个帮助子类中的某一个，并且你希望将哪一个帮助子类是代理者这一信息局部化的时候。** (客户端不需要知道是哪个子类工厂在工作)。
*   需要解耦对象的创建和使用时。

## 6. 给Java初学者的提示

*   **核心在于“延迟实例化到子类”**: 这是工厂方法模式的精髓。父类 `Factory` 定义了创建的规范（通过抽象的 `createProduct` 方法），而具体的创建工作由子类 `IDCardFactory` 完成。
*   **模板方法模式的应用**: 注意 `Factory` 类中的 `create` 方法。它是一个 `final` 方法，定义了创建和注册产品的步骤顺序。而具体的步骤 `createProduct` 和 `registerProduct` 则是抽象的，由子类实现。这种结构本身就是模板方法模式的一个应用。
*   **区分“工厂方法”和“调用工厂方法的方法”**:
    *   `createProduct(String owner)` 是真正的工厂方法，它通常是 `protected` 和 `abstract` (或可被覆盖的)。
    *   `create(String owner)` 是一个公共接口，客户端通过它来获取产品，它内部调用了工厂方法。
*   **面向接口编程**: 客户端 (`Main`) 通常应该依赖于抽象的 `Factory` 和 `Product` 接口，而不是具体的实现类。这样可以更容易地替换具体工厂或具体产品。

工厂方法模式通过将实例化逻辑封装在子类中，提高了系统的灵活性和可扩展性，是面向对象设计中非常重要的一个模式。
