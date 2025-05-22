# 模块名称: 适配器模式 (Adapter Pattern)

## 1. 核心概念 (针对Java初学者)

**什么是适配器模式？**

适配器模式（Adapter Pattern）是一种结构型设计模式，它的主要作用是将一个类的接口转换成客户端期望的另一个接口。适配器使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

简单来说，如果你有一个组件，它的功能你很满意，但是它的接口（即调用它的方法的方式）与你当前系统要求的接口不一致，这时候就可以引入一个适配器。适配器包装原有的组件，并提供系统所期望的接口。

**生活中的类比：电源适配器**

想象一下你去国外旅行，你带的笔记本电脑的插头是两孔的（比如中国标准），但酒店墙上的插座是三孔的（比如英国标准）。这时你就需要一个电源适配器。这个适配器一端能插进三孔插座，另一端能让你原来的两孔插头插进去。你的笔记本电脑（客户端）不需要改变，墙上的插座（被适配者）也不需要改变，适配器在中间起到了转换作用。

## 2. 适配器模式的两种实现方式

适配器模式主要有两种实现方式：

1.  **类适配器 (Class Adapter)**: 使用继承的方式来实现。适配器类继承被适配者的类，并且实现目标接口。
2.  **对象适配器 (Object Adapter)**: 使用组合（或称为委托）的方式来实现。适配器类持有一个被适配者对象的引用，并且实现目标接口。

本项目在 `Adaptor/Extends/` 目录下展示了类适配器的实现，在 `Adaptor/Delegate/` 目录下展示了对象适配器的实现。

## 3. 类适配器模式 (Class Adapter)

类适配器模式通过继承被适配者类并实现目标接口的方式工作。

### 代码分析 (`Adaptor/Extends/` 目录下)

*   **`Banner.java` (被适配者 - Adaptee)**:
    *   这是我们已有的一个类，它提供了特定的功能，但其接口与客户端期望的不一致。
    *   `public Banner(String string)`: 构造函数，接收一个字符串。
    *   `public void showWithParen()`: 将字符串用 `()` 包裹并打印。
    *   `public void showWithAster()`: 将字符串用 `*` 包裹并打印。
    *   这个 `Banner` 类就好比我们例子中的“英标插座”，它有自己的工作方式。

*   **`Print.java` (目标接口 - Target)**:
    *   这是客户端期望使用的接口。客户端代码将通过这个接口与适配器进行交互。
    *   `public abstract void printWeek();` (原文为 `printWeak`, 但通常理解为“弱化”打印)
    *   `public abstract void printStrong();`
    *   这个 `Print` 接口就好比“国标插孔”，我们希望最终能调用这两个方法。

*   **`PrintBanner.java` (适配器 - Adapter)**:
    *   这是适配器类，它连接了 `Banner` 和 `Print`。
    *   `public class PrintBanner extends Banner implements Print`:
        *   **`extends Banner`**: 适配器继承了 `Banner` 类。这意味着 `PrintBanner` 直接拥有了 `Banner` 的所有方法（`showWithParen` 和 `showWithAster`）。这是类适配器的核心特征。
        *   **`implements Print`**: 适配器实现了 `Print` 接口，这意味着它必须提供 `printWeek` 和 `printStrong` 方法的具体实现。
    *   `public PrintBanner(String string)`: 构造函数，调用父类 `Banner` 的构造函数。
    *   `public void printWeek()`: 实现了 `Print` 接口的方法。内部调用了继承自 `Banner` 的 `showWithParen()` 方法。
        ```java
        public void printWeek() {
            showWithParen(); // 调用父类 Banner 的方法
        }
        ```
    *   `public void printStrong()`: 实现了 `Print` 接口的方法。内部调用了继承自 `Banner` 的 `showWithAster()` 方法。
        ```java
        public void printStrong() {
            showWithAster(); // 调用父类 Banner 的方法
        }
        ```

*   **`Main.java` (客户端 - Client)**:
    *   客户端代码演示了如何使用适配器。
    *   `Print p = new PrintBanner("Hello");`: 创建 `PrintBanner` 适配器的实例，并将其赋值给 `Print` 类型的引用。客户端只知道它在使用一个 `Print` 对象。
    *   `p.printWeek();`: 调用目标接口的方法。
    *   `p.printStrong();`: 调用目标接口的方法。
    *   客户端并不知道 `Banner` 类的存在，它只与 `Print` 接口打交道。

### 优点

*   适配器可以直接重用被适配者类的部分方法（如果这些方法在目标接口中没有对应，或者适配器需要直接访问被适配者的保护成员）。
*   可以覆盖被适配者类的方法（因为是继承关系）。

### 缺点

*   **Java不支持多重继承**：这是类适配器最大的问题。适配器已经继承了被适配者，所以它不能再继承其他类。这限制了其使用场景。
*   **紧耦合**：适配器和被适配者是继承关系，耦合度较高。
*   **适配难度**：如果被适配者类是 `final` 类，则无法通过继承来创建类适配器。

## 4. 对象适配器模式 (Object Adapter)

对象适配器模式通过在适配器内部持有被适配者对象的实例（组合/委托）来实现。

### 代码分析 (`Adaptor/Delegate/` 目录下)

*   **`Banner.java` (被适配者 - Adaptee)**:
    *   与类适配器中的 `Banner.java` 完全相同。

*   **`Print.java` (目标 - Target)**:
    *   **注意**: 在这个 `Delegate` 的例子中，`Print` 被定义为一个 **抽象类** (`public abstract class Print`)，而不是接口。
    *   `public abstract void printWeek();`
    *   `public abstract void printStrong();`
    *   对于适配器模式而言，目标可以是一个接口，也可以是一个抽象类，甚至是一个具体的类。客户端期望的是这个类型。

*   **`PrintBanner.java` (适配器 - Adapter)**:
    *   `public class PrintBanner extends Print`: 适配器继承了目标抽象类 `Print`（也可以是实现目标接口）。
    *   `private Banner banner;`: **核心区别**！适配器内部持有一个 `Banner` (被适配者) 的私有实例。这是对象适配器的特征——委托。
    *   `public PrintBanner(String string)`: 构造函数。
        ```java
        public PrintBanner(String string) {
            this.banner = new Banner(string); // 创建并持有一个 Banner 实例
        }
        ```
    *   `public void printWeek()`: 实现了 `Print` 的方法。内部将调用委托给了 `banner` 对象的 `showWithParen()` 方法。
        ```java
        public void printWeek() {
            banner.showWithParen(); // 委托给 banner 对象的相应方法
        }
        ```
    *   `public void printStrong()`: 实现了 `Print` 的方法。内部将调用委托给了 `banner` 对象的 `showWithAster()` 方法。
        ```java
        public void printStrong() {
            banner.showWithAster(); // 委托给 banner 对象的相应方法
        }
        ```

*   **`Main.java` (客户端 - Client)**:
    *   与类适配器中的 `Main.java` 用法完全相同。客户端仍然是通过 `Print` 类型来使用 `PrintBanner` 对象，并不知道具体的适配过程。

### 优点

*   **松耦合**：适配器与被适配者是组合关系，耦合度相对较低。可以很容易地替换被适配者的实例或动态改变被适配者。
*   **灵活性高**：由于是组合关系，适配器可以适配一个类的整个继承体系（即 `Adaptee` 的任何子类都可以被适配）。
*   适配器可以同时适配多个被适配者（如果有多个 `Adaptee` 接口）。

### 缺点

*   需要额外编写委托调用的代码，对于被适配者接口方法很多的情况，适配器代码量可能会稍多一些。

## 5. 如何选择？类适配器 vs. 对象适配器

*   **优先使用对象适配器**：通常情况下，对象适配器是更好的选择，因为它更符合“合成复用原则”（Favor Composition over Inheritance），提供了更好的灵活性和松耦合。
*   **当需要适配一个具体类，并且需要在适配器中重写被适配者的方法时**，可以考虑类适配器。但要注意Java单继承的限制。
*   **如果被适配者本身就是一个接口**，那么通常使用对象适配器，适配器实现目标接口并委托给被适配接口的实现。

在Java中，由于单继承的限制，对象适配器的使用场景更为广泛。

## 6. 适用场景

*   **想使用一个已经存在的类，而它的接口不符合你的需求时。**
*   **（类适配器）想创建一个可以复用的类，该类可以与其他不相关的类或不可预见的类（即那些接口可能不一定兼容的类）协同工作。**
*   **（对象适配器）想使用一些已经存在的子类，但是不可能对每一个都进行子类化以匹配它们的接口。对象适配器可以适配这些子类的父类接口。**
*   当你需要统一多个类的接口，或者为旧的接口提供新的接口封装时。
*   在系统重构时，可以用适配器模式来兼容旧的接口，使得新旧代码可以协同工作。

## 7. 给Java初学者的提示

*   **核心作用是“转换”和“兼容”**: 牢记适配器模式就是为了解决接口不匹配的问题，让原本不能一起工作的组件能协同起来。
*   **理解继承和组合**:
    *   类适配器使用 **继承** (`extends Adaptee implements Target`)。
    *   对象适配器使用 **组合/委托** (`private Adaptee adapteeInstance;` 然后在方法中调用 `adapteeInstance.specificMethod()`)。
    *   这是理解两种适配器实现方式的关键。
*   **目标 (Target) 可以是接口也可以是抽象类**:
    *   在 `Adaptor/Extends/` 中, `Print` 是一个接口。这是最常见的情况。
    *   在 `Adaptor/Delegate/` 中, `Print` 是一个抽象类。
    *   对于初学者来说，需要理解：客户端代码 (`Main.java`) 都是面向 `Print` 类型编程的。无论 `Print` 是接口还是抽象类，适配器 (`PrintBanner`) 都需要满足 `Print` 定义的契约（即实现其抽象方法）。使用抽象类作为Target，可能是因为Target除了定义抽象方法外，还想提供一些公共的非抽象方法或属性，供其子类（包括适配器）使用。但在这个具体例子中，`Print` 抽象类和 `Print` 接口的功能是等价的。
*   **适配器不是万能的**: 它只解决接口转换问题，不改变被适配者原有的功能。

适配器模式是一个非常实用且在各种项目中广泛应用的设计模式。理解它的两种形式及其优缺点，能帮助你写出更灵活、更易维护的代码。
