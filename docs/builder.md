# 模块名称: 建造者模式 (Builder Pattern)

## 1. 核心概念 (针对Java初学者)

**什么是建造者模式？**

建造者模式（Builder Pattern）是一种创建型设计模式，它的核心思想是**将一个复杂对象的构建过程与其表示分离，使得同样的构建过程可以创建不同的表示。**

想象一下你要组装一台电脑。电脑是一个复杂对象，包含CPU、内存、硬盘、主板等部件。
*   **构建过程**: 安装CPU -> 安装内存 -> 安装硬盘 -> 连接主板。这个步骤是相对固定的。
*   **表示**:
    *   表示A：高端游戏电脑（Intel i9 CPU, 32GB RAM, 2TB SSD, 高端主板）
    *   表示B：办公电脑（Intel i5 CPU, 16GB RAM, 1TB HDD, 普通主板）

建造者模式允许你使用相同的组装步骤（由`Director`指挥），但通过使用不同的零件供应商和装配工（`ConcreteBuilder`），来得到不同配置的电脑（不同的`Product`表示）。

**生活中的类比：快餐店点餐**

*   **顾客 (Client)**: 你走到快餐店。
*   **收银员 (Director)**: 你告诉收银员，你要一份“A套餐”。收银员知道“A套餐”包含汉堡、薯条、可乐，并且有固定的准备顺序。
*   **后台厨师/配餐员 (Builder)**: 收银员把“A套餐”的指令传给后台。
    *   **标准A套餐厨房 (ConcreteBuilder1)**: 按照汉堡->薯条->可乐的顺序准备标准的A套餐。
    *   **素食A套餐厨房 (ConcreteBuilder2)**: 可能也按照汉堡->薯条->可乐的顺序，但是汉堡是素汉堡，薯条可能是烤的，可乐可能是无糖的。
*   **A套餐 (Product)**: 最后你拿到的那份餐。

同样的点餐指令（“A套餐”），通过不同的后台处理（不同的Builder），可以得到不同内容（表示）的套餐。

## 2. 模式结构与角色分析

本项目的 `Builder/` 目录下的文件清晰地展示了建造者模式的结构。

*   **Builder (抽象建造者 `Builder.java`)**:
    *   **作用**: 为创建一个产品对象的各个部件指定抽象接口。它不关心各个部件是如何表示的，只定义了需要哪些部件以及构建这些部件的抽象方法。
    *   **代码分析 (`Builder.java`)**:
        ```java
        public abstract class Builder {
            // public abstract Boolean isCalledTitle; // 源码中被注释掉了
            public abstract void makeTitle(String title);
            public abstract void makeString(String str);
            public abstract void makeItems(String[] items);
            public abstract void close();
        }
        ```
        定义了构建文档的几个基本步骤：制作标题、制作字符串内容、制作列表项、结束构建。
        *值得注意的是，具体的建造者 (`HTMLBuilder`, `TextBuilder`) 内部都实现了一个 `isCalledTitle` 的布尔标志，用来确保 `makeTitle` 是第一个被调用的构建方法，否则会抛出异常。这暗示了构建步骤之间可能存在的隐式顺序或依赖关系，由具体建造者来管理。*

*   **ConcreteBuilder (具体建造者 `HTMLBuilder.java`, `TextBuilder.java`)**:
    *   **作用**: 实现 `Builder` 接口，负责构造和装配产品的各个部件。每个 `ConcreteBuilder` 定义并明确了它所创建的产品表示，并通常提供一个获取最终产品的方法 (`getResult()`)。
    *   **代码分析 (`HTMLBuilder.java`)**:
        *   负责构建一个HTML格式的文档。
        *   `makeTitle(String title)`: 创建HTML文件的头部、标题 (`<title>`) 和一级标题 (`<h1>`)。它还会初始化一个 `PrintWriter` 将内容写入以 `title` 命名的 `.html` 文件。
        *   `makeString(String str)`: 将字符串包装在 `<p>` 标签中写入文件。
        *   `makeItems(String[] items)`: 将字符串数组包装在 `<ul>` 和 `<li>` 标签中写入文件。
        *   `close()`: 写入 `</body></html>` 并关闭 `PrintWriter`。
        *   `public String getResult()`: 返回生成的HTML文件名。**产品是这个HTML文件本身。**

    *   **代码分析 (`TextBuilder.java`)**:
        *   负责构建一个纯文本格式的文档。
        *   `makeTitle(String title)`: 在 `StringBuffer` 中添加带装饰框的标题。
        *   `makeString(String str)`: 在 `StringBuffer` 中添加带 `*` 前缀的字符串。
        *   `makeItems(String[] items)`: 在 `StringBuffer` 中添加带 ` - ` 前缀的列表项。
        *   `close()`: 在 `StringBuffer` 中添加底部装饰框。
        *   `public String getResult()`: 返回 `StringBuffer` 中积累的完整文本内容。**产品是这个文本字符串。**

*   **Director (指挥者 `Director.java`)**:
    *   **作用**: 构造一个使用 `Builder` 接口的对象。`Director` 封装了构建一个复杂对象的固定步骤或算法顺序。客户端通常只需要将 `ConcreteBuilder` 传递给 `Director`，然后由 `Director` 来按顺序调用 `Builder` 的方法。
    *   **代码分析 (`Director.java`)**:
        ```java
        public class Director {
            private Builder builder; // 持有对Builder接口的引用
            public Director(Builder builder) { // 接收一个具体的Builder实例
                this.builder = builder;
            }
            public void construct() { // 定义了构建产品的固定顺序
                builder.makeTitle("Greeting");
                builder.makeString("From morning to afternoon");
                builder.makeItems(new String[] {
                        "Good morning",
                        "Good afternoon"
                });
                builder.makeString("At evening");
                builder.makeItems(new String[] {
                        "Good evening",
                        "Good night"
                });
                builder.close();
            }
        }
        ```
        `construct` 方法精确地控制了文档的构建流程：先标题，再问候语，再列表项等等。

*   **Product (产品)**:
    *   **作用**: 表示被构造的复杂对象。`ConcreteBuilder` 创建该产品的内部表示并定义它的装配过程。
    *   **说明**: 在这个具体的例子中，并没有一个显式的、单独的 `Product` 类（例如 `Document` 类）。
        *   对于 `HTMLBuilder`，产品就是它最终生成的那个 **HTML文件**。`getResult()` 返回的是该文件名。
        *   对于 `TextBuilder`，产品就是它构建的那个**纯文本字符串**。`getResult()` 返回的是这个字符串。
        这种方式也是建造者模式的一种常见实现，产品由Builder直接生成并返回。

*   **Client (客户端 `Main.java`)**:
    *   **作用**:
        1.  创建具体的 `ConcreteBuilder` 对象。
        2.  创建 `Director` 对象，并将 `ConcreteBuilder` 实例传递给它。
        3.  调用 `Director` 的构建方法（例如 `construct()`）来启动构建过程。
        4.  从 `ConcreteBuilder` 中获取构建完成的产品。
    *   **代码分析 (`Main.java`)**:
        ```java
        public class Main {
            public static void main(String[] args) {
                if (args.length != 1) { /* ... usage ... */ }

                if (args[0].equals("plain")) {
                    TextBuilder textbuilder = new TextBuilder();        // 1. 创建TextBuilder
                    Director director = new Director(textbuilder);    // 2. Director用TextBuilder配置
                    director.construct();                             // 3. 指挥构建
                    String result = textbuilder.getResult();          // 4. 从Builder获取产品
                    System.out.println(result);
                } else if (args[0].equals("html")) {
                    HTMLBuilder htmlbuilder = new HTMLBuilder();      // 1. 创建HTMLBuilder
                    Director director = new Director(htmlbuilder);  // 2. Director用HTMLBuilder配置
                    director.construct();                           // 3. 指挥构建
                    String filename = htmlbuilder.getResult();        // 4. 从Builder获取产品(文件名)
                    System.out.println(filename);                   // 打印文件名
                } else { /* ... usage ... */ }
            }
            // ... usage() method ...
        }
        ```
        客户端根据命令行参数选择使用 `TextBuilder` 还是 `HTMLBuilder`，然后让 `Director` 使用选定的 `Builder` 来构建文档，最后从 `Builder` 获取结果。

## 3. 建造者模式的优点

*   **封装性好，构建和表示分离**: 客户端不需要知道产品内部的构建细节和装配方式。构建产品的具体过程被封装在 `ConcreteBuilder` 中，而构建步骤的顺序则被封装在 `Director` 中。客户端只需要指定产品的类型（通过选择不同的 `ConcreteBuilder`）就可以得到产品。
*   **更好的控制构建过程**: `Director` 负责具体的构建顺序，使得构建过程更加规范和可控。
*   **易于扩展，可以创建不同的产品表示**: 增加新的产品表示只需要增加一个新的 `ConcreteBuilder` 类，实现 `Builder` 接口中定义的方法即可。原有的 `Director` 类不需要修改，甚至客户端代码的改动也很小。
*   **逐步构建，精细控制**: 建造者模式允许一步一步地构建对象，可以在每一步进行更精细的控制，或者在构建过程中加入条件判断。

## 4. 与抽象工厂模式的区别 (简要)

虽然两者都是创建型模式，但它们的关注点不同：

*   **建造者模式 (Builder)**:
    *   关注的是一个**复杂对象**的**分步骤构建过程**。
    *   强调的是“如何一步步把这个东西造出来”，并且允许同样的构建过程（Director）产生不同的最终形态（不同的ConcreteBuilder产出不同的Product表示）。
    *   通常客户端最后只需要从Builder获取一个构建好的完整产品。
*   **抽象工厂模式 (Abstract Factory)**:
    *   关注的是创建**一系列相关的产品对象（产品族）**。
    *   强调的是“一次性拿到一整套风格匹配的东西”，不关心这些东西内部是如何被创建的。
    *   客户端通过抽象工厂获取多个不同类型但属于同一族的产品。

简单来说：建造者是“我要一个这样流程造出来的东西”，抽象工厂是“我要一套这样风格的东西”。

## 5. 适用场景

*   **当创建复杂对象的算法应该独立于该对象的组成部分以及它们的装配方式时。** (例如，`Director` 定义了文档的通用结构，而 `HTMLBuilder` 和 `TextBuilder` 决定了如何表示这些结构)。
*   **当构造过程必须允许被构造的对象有不同的表示时。** (本例中，同样的 `Director` 构建流程可以产生HTML文档或纯文本文档)。
*   **需要生成的产品对象有复杂的内部结构，这些产品对象通常具备共有的创建步骤。**
*   当一个对象的构建需要多个步骤，并且这些步骤的顺序很重要时。

## 6. 给Java初学者的提示

*   **理解 `Director` 的核心作用**: `Director` 封装了构建复杂对象的“稳定”部分——即构建的顺序和步骤。如果构建顺序经常变化，或者非常简单，有时可以省略 `Director`，由客户端直接按需调用 `Builder` 的方法。
*   **理解 `Builder` 的核心作用**: `Builder` 接口定义了“需要哪些构建步骤”，而 `ConcreteBuilder` 则负责“如何实现每一个具体的构建步骤”以及“最终提供构建好的产品”。
*   **产品可以由Builder直接返回**: 像本例一样，产品可以直接是 `ConcreteBuilder` 的 `getResult()` 方法的返回值（比如一个字符串或文件名），而不必是一个独立定义的 `Product` 类。
*   **状态管理**: 注意到 `HTMLBuilder` 和 `TextBuilder` 内部都有一个 `isCalledTitle` 状态变量，用于确保 `makeTitle` 方法在其他构建方法之前被调用。这是 `ConcreteBuilder` 内部管理构建过程一致性的一种方式。
*   **链式调用 (Fluent Interface)**: 很多现代API中的Builder模式（尤其是在Java中，如 `StringBuilder`, `Stream.Builder`, Lombok的 `@Builder`）会把 `makeXXX` 方法的返回类型设为 `Builder` 本身 (即 `return this;`)，这样可以实现链式调用，例如：
    ```java
    // 假设 TextBuilder 的方法返回 this
    // String result = new TextBuilder()
    //                  .makeTitle("My Doc")
    //                  .makeString("Intro")
    //                  .makeItems(new String[]{"Item 1"})
    //                  .close()
    //                  .getResult();
    ```
    本项目中的例子没有使用这种风格，但这是值得了解的一个常见变体。

建造者模式通过将复杂的构建逻辑从业务代码中分离出来，使得代码更加清晰，也更容易适应变化。
