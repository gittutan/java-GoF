# 模块名称

建造者模式 (Builder Pattern)

# 核心概念 (针对Java初学者)

**什么是建造者模式？**

建造者模式（Builder Pattern）是一种创建型设计模式，它的核心思想是**将一个复杂对象的构建过程与其表示分离，使得同样的构建过程可以创建不同的表示。**

想象一下，你要组装一台电脑。这台电脑有很多部件：CPU、内存、硬盘、主板、显卡等。组装的过程（先装CPU到主板，再装内存，再把主板装进机箱等）是相对固定的。但是，你可以选择不同品牌和型号的CPU、内存、硬盘，从而得到不同配置（表示）的电脑。

建造者模式就是用来处理这种场景的：

1.  **构建过程是稳定的**：例如，创建一个文档的步骤可能是：创建标题 -> 创建段落 -> 创建列表项 -> 完成文档。
2.  **对象的表示是多样的**：同样的构建步骤，可以用来创建不同格式的文档，比如一个HTML文档，或者一个纯文本文档。

建造者模式通过引入一个“建造者”接口和多个“具体建造者”类，以及一个“指挥者”类，来达到这个目的。

*   **指挥者 (Director)**: 知道构建一个复杂对象的完整步骤顺序。它不关心每一步具体是怎么做的，只负责按顺序调用建造者的方法。
*   **建造者 (Builder)**: 定义了构建产品各个部件的抽象接口。
*   **具体建造者 (ConcreteBuilder)**: 实现建造者接口，负责实际构建产品的各个部件，并最终提供组装好的产品。
*   **产品 (Product)**: 最终被构建出来的复杂对象。

**类比：快餐店点餐**

*   **你 (Client - 客户端)**: 走到快餐店。
*   **收银员 (Director - 指挥者)**: 你告诉收银员，你要一份“A套餐”。收银员知道“A套餐”包含哪些东西，以及准备的顺序（比如先准备汉堡，再准备薯条，最后准备可乐）。收银员会将这些指令传达给后台厨房。
*   **后台厨师接口 (Builder - 建造者)**: 这是一个抽象的厨师接口，定义了制作套餐中每个单项的方法，如 `做汉堡()`, `做薯条()`, `倒可乐()`。
*   **标准A套餐厨房 (ConcreteBuilder1 - 具体建造者1)**: 这个厨房按照收银员的指令，用标准的食材（牛肉饼、标准面包）制作汉堡，炸标准薯条，倒标准可乐。最后提供一份标准的A套餐。
*   **素食A套餐厨房 (ConcreteBuilder2 - 具体建造者2)**: 这个厨房也接收同样的指令（做汉堡、薯条、可乐），但是它用素食饼、全麦面包制作汉堡，用烤蔬菜条代替炸薯条，提供鲜榨果汁代替可乐。最后提供一份素食的A套餐。
*   **A套餐 (Product - 产品)**: 最终得到的套餐，可以是标准的，也可以是素食的，但都是按照“A套餐”的构建步骤来的。

在这个类比中，顾客（Client）不需要知道汉堡、薯条、可乐具体是怎么做的，也不需要知道标准厨房和素食厨房的区别。顾客只需要告诉收银员（Director）他要什么套餐（构建过程）。收银员（Director）使用同样的指令序列来指挥不同的厨房（Builder），从而得到不同表示的产品。这就是建造者模式的核心思想。

# 模式结构与角色分析

建造者模式主要包含以下角色：

## Builder (抽象建造者 `Builder.java`)

*   **作用**: 为创建一个产品对象的各个部件指定抽象接口。它定义了构建产品的每一个步骤应该做什么，但不涉及具体实现。
*   **代码分析 (`Builder.java`)**:
    *   这是一个抽象类，定义了所有具体建造者必须实现的方法，代表了构建一个文档的各个步骤：
        ```java
        public abstract class Builder {
            //	protected abstract Boolean isCalledTitle; // 源码中注释掉了，但其逻辑在子类中实现
            public abstract void makeTitle(String title);     // 构建标题部分
            public abstract void makeString(String str);      // 构建字符串内容部分
            public abstract void makeItems(String[] items);   // 构建列表项目部分
            public abstract void close();                     // 完成文档构建的收尾工作
        }
        ```
    *   这些方法都是抽象的，由具体的子类来实现。
    *   **关于 `isCalledTitle`**: 虽然在 `Builder.java` 的注释中提到了 `isCalledTitle`，但实际的标志位及其逻辑是在具体建造者 `HTMLBuilder` 和 `TextBuilder` 中实现的。这个标志用于确保 `makeTitle` 方法必须是第一个被调用的构建方法（除了构造函数），否则后续的构建步骤（如 `makeString`, `makeItems`, `close`）会抛出运行时异常。这是一种对构建顺序的隐式要求和校验。

## ConcreteBuilder (具体建造者 `HTMLBuilder.java`, `TextBuilder.java`)

*   **作用**: 实现 `Builder` 接口，负责具体构建和装配产品的各个部件。每个 `ConcreteBuilder` 定义并明确它所创建的产品表示，并通常提供一个获取最终产品的方法。
*   **代码分析 (`HTMLBuilder.java`)**:
    *   此类继承自 `Builder`，用于构建HTML格式的文档。
    *   `private String filename;`: 用于存储生成的HTML文件名。
    *   `private PrintWriter writer;`: 用于向HTML文件写入内容。
    *   `private Boolean isCalledTitle = false;`: 标志位，确保 `makeTitle` 是第一个被调用的构建方法。
    *   `public void makeTitle(String title)`:
        *   检查 `isCalledTitle`，如果为 `true`（意味着 `makeTitle` 已被调用过），则抛出 `RuntimeException`。
        *   创建HTML文件（文件名基于 `title`），初始化 `PrintWriter`。
        *   输出HTML的头部、`<h1>` 标题。
        *   设置 `isCalledTitle = true;`。
    *   `public void makeString(String str)`:
        *   检查 `isCalledTitle`，如果为 `false`，抛出异常。
        *   输出 `<p>` 标签包裹的字符串。
    *   `public void makeItems(String[] items)`:
        *   检查 `isCalledTitle`。
        *   输出 `<ul>` 和 `<li>` 标签包裹的列表项。
    *   `public void close()`:
        *   检查 `isCalledTitle`。
        *   输出 `</body></html>` 并关闭 `writer`。
    *   `public String getResult()`:
        *   返回构建好的HTML文件名。**产品是一个HTML文件。**

*   **代码分析 (`TextBuilder.java`)**:
    *   此类继承自 `Builder`，用于构建纯文本格式的文档。
    *   `private StringBuffer buffer = new StringBuffer();`: 用于存储构建过程中的文本内容。
    *   `private Boolean isCalledTitle = false;`: 同样用于校验 `makeTitle` 的调用顺序。
    *   `public void makeTitle(String title)`:
        *   检查 `isCalledTitle`。
        *   向 `buffer` 追加带分隔线的标题。
        *   设置 `isCalledTitle = true;`。
    *   `public void makeString(String str)`:
        *   检查 `isCalledTitle`。
        *   向 `buffer` 追加带星号前缀的字符串内容。
    *   `public void makeItems(String[] items)`:
        *   检查 `isCalledTitle`。
        *   向 `buffer` 追加带连字符前缀的列表项。
    *   `public void close()`:
        *   检查 `isCalledTitle`。
        *   向 `buffer` 追加底部分隔线。
    *   `public String getResult()`:
        *   返回 `buffer` 中积累的所有文本内容。**产品是一个字符串。**

## Director (指挥者 `Director.java`)

*   **作用**: 构造一个使用 `Builder` 接口的对象。它封装了构建一个特定复杂对象的固定步骤顺序。`Director` 不关心每一步具体是如何实现的，只负责按顺序调用 `Builder` 的方法。
*   **代码分析 (`Director.java`)**:
    *   `private Builder builder;`: 持有一个 `Builder` 类型的引用。这个 `builder` 可以是 `HTMLBuilder` 的实例，也可以是 `TextBuilder` 的实例，或者是任何其他 `Builder` 的子类实例。
    *   `public Director(Builder builder)`: 构造函数，接收一个具体的建造者对象。
    *   `public void construct()`: 这是核心方法，定义了构建“Greeting”文档的固定步骤和顺序：
        ```java
        public void construct() {
            builder.makeTitle("Greeting"); // 1. 制作标题
            builder.makeString("From morning to afternoon"); // 2. 添加一段字符串
            builder.makeItems(new String[] { // 3. 添加一组项目
                    "Good morning",
                    "Good afternoon"
            });
            builder.makeString("At evening"); // 4. 再添加一段字符串
            builder.makeItems(new String[] { // 5. 再添加一组项目
                    "Good evening",
                    "Good night"
            });
            builder.close(); // 6. 完成构建
        }
        ```
        `Director` 通过调用 `builder` 对象的各个 `makeXxx` 方法，按照预设的顺序来构建产品。由于 `builder` 的实际类型不同（`HTMLBuilder` 或 `TextBuilder`），同样的 `construct` 过程会产生不同表示的产品。

## Product (产品)

*   **作用**: 表示被构造出来的复杂对象。`ConcreteBuilder` 在其内部创建并装配该产品的部件。
*   **说明**:
    *   在这个具体的例子中，**并没有一个显式的、独立的 `Product` 类**。
    *   产品是由 `ConcreteBuilder` 直接生成的，并通过 `ConcreteBuilder` 的 `getResult()` 方法返回。
        *   对于 `HTMLBuilder`，产品是最终生成的 **HTML文件**（`getResult()` 返回文件名，如 "Greeting.html"）。
        *   对于 `TextBuilder`，产品是最终生成的 **纯文本字符串**（`getResult()` 返回包含所有文本内容的 `String`）。
    *   在某些建造者模式的实现中，可能会有一个明确的 `Product` 类，`Builder` 负责创建和组装这个 `Product` 类的实例。但在这个例子中，产品是更广义的概念，指代构建的结果。

## Client (客户端 `Main.java`)

*   **作用**:
    1.  创建具体的 `ConcreteBuilder` 对象（如 `TextBuilder` 或 `HTMLBuilder`）。
    2.  将这个 `ConcreteBuilder` 对象传递给 `Director` 对象。
    3.  命令 `Director` 对象执行构建过程（调用 `director.construct()`）。
    4.  从 `ConcreteBuilder` 对象中获取构建好的产品（调用 `builder.getResult()`）。
*   **代码分析 (`Main.java`)**:
    *   `public static void main(String[] args)`:
        *   根据命令行参数 `args[0]` 来决定使用哪种 `Builder`。
        *   **如果 `args[0]` 是 "plain"**:
            ```java
            TextBuilder textbuilder = new TextBuilder();        // 1. 创建 TextBuilder
            Director director = new Director(textbuilder);      // 2. 用 TextBuilder 创建 Director
            director.construct();                               // 3. Director 执行构建过程
            String result = textbuilder.getResult();            // 4. 从 TextBuilder 获取产品 (文本字符串)
            System.out.println(result);                         // 打印产品
            ```
        *   **如果 `args[0]` 是 "html"**:
            ```java
            HTMLBuilder htmlbuilder = new HTMLBuilder();        // 1. 创建 HTMLBuilder
            Director director = new Director(htmlbuilder);      // 2. 用 HTMLBuilder 创建 Director
            director.construct();                               // 3. Director 执行构建过程
            String filename = htmlbuilder.getResult();          // 4. 从 HTMLBuilder 获取产品 (HTML文件名)
            System.out.println(filename);                       // 打印产品 (文件名)
            ```
        *   `usage()` 方法用于在参数不正确时提示用户。
    *   客户端通过选择不同的 `Builder`（`TextBuilder` 或 `HTMLBuilder`）并将其交给同一个 `Director`，就能得到不同格式（纯文本或HTML）的文档，而 `Director` 的构建逻辑 (`construct` 方法) 完全不需要改变。

`Greeting.html` 文件是当使用 `HTMLBuilder` 时，`Director` 的 `construct` 方法执行后生成的一个示例输出文件。

# 建造者模式的优点

1.  **封装性，隐藏内部细节**:
    *   客户端只需要知道 `Builder` 的接口和 `Director` 的存在，而不需要了解产品内部的复杂结构和装配细节。
    *   产品本身的创建细节对客户端是透明的。

2.  **构建过程和表示分离**:
    *   这是建造者模式最核心的优点。`Director` 负责定义和控制构建过程（步骤顺序）。`ConcreteBuilder` 负责实现每一步的具体构建逻辑，从而产生不同的产品表示。
    *   这意味着可以改变产品的内部表示（通过使用不同的 `ConcreteBuilder`）而无需改变构建过程（`Director` 的代码）。反之，也可以改变构建过程而无需改变各个部件的表示。

3.  **更好的控制构建过程**:
    *   `Director` 类封装了复杂的构建逻辑和顺序，将这部分逻辑从客户端代码中分离出来，使得客户端代码更简洁。
    *   建造者模式可以更精细地控制产品的创建过程，例如本例中通过 `isCalledTitle` 标志强制了 `makeTitle` 必须首先被调用。

4.  **易于扩展，提高模块化**:
    *   **增加新的产品表示**: 当需要一种新的产品表示时，只需要添加一个新的 `ConcreteBuilder` 类，实现 `Builder` 接口中定义的各个构建步骤即可。原有的 `Director` 类和 `Builder` 接口通常不需要修改。
    *   **增加新的构建步骤**: 如果构建过程需要增加新的步骤，可能需要修改 `Builder` 接口和所有 `ConcreteBuilder` 子类，以及 `Director`。但如果只是修改现有步骤的顺序或组合，则通常只需要修改 `Director`。

# 与抽象工厂模式的区别 (简要)

建造者模式和抽象工厂模式都是创建型模式，都用于处理对象的创建，但它们的关注点和使用场景有所不同：

*   **建造者模式 (Builder Pattern)**:
    *   **关注点**: **一步一步地构建一个复杂对象**。它强调的是对象的**构建过程**，允许通过相同的构建过程创建出内部结构或表示不同的对象。
    *   **目的**: 将复杂对象的构建过程与其表示分离。
    *   **产出**: 通常最终只生成一个复杂产品。
    *   **例子**: 构建一个文档（如HTML或文本），需要逐步添加标题、段落、列表等。

*   **抽象工厂模式 (Abstract Factory Pattern)**:
    *   **关注点**: **创建一系列相关的产品对象（一个产品族）**，而不需要指定它们的具体类。它不关心单个产品的构建过程，而是强调一次性获取一整套相互匹配的产品。
    *   **目的**: 提供一个接口，用于创建一系列相关或相互依赖的对象。
    *   **产出**: 通常会创建多个属于同一产品族的不同类型的简单或复杂产品。
    *   **例子**: GUI工具包，一个工厂可以创建Windows风格的按钮、文本框、窗口，另一个工厂可以创建MacOS风格的按钮、文本框、窗口。

**简单来说**:
*   **建造者模式**像是“自定义套餐”，你按照一定的步骤（`Director`）选择不同的材料和做法（`ConcreteBuilder`），最终得到一个定制化的复杂产品。
*   **抽象工厂模式**像是“品牌专卖店”，你选择一个品牌（具体工厂），然后这家店能提供该品牌下的一整套产品（按钮、文本框等都是同一风格）。

# 适用场景

建造者模式主要适用于以下情况：

1.  **当创建一个复杂对象的算法（构建过程）应该独立于该对象的组成部分以及它们的装配方式时。**
    *   例如，本例中构建文档的算法（先标题，后字符串，再列表项等）由 `Director` 定义，这与具体是生成HTML文档还是纯文本文档（由 `ConcreteBuilder` 决定）是分离的。
2.  **当构造过程必须允许被构造的对象有不同的表示时。**
    *   这是建造者模式的核心优势。同样的 `Director` 和 `construct()` 过程，配合不同的 `ConcreteBuilder`（`HTMLBuilder`, `TextBuilder`），可以得到不同格式的产品。
3.  **需要生成的产品对象有复杂的内部结构，这些产品对象通常包含多个成员属性或部件，并且这些部件的创建顺序会影响最终产品的形态或行为。**
    *   例如，一个汽车对象，需要按顺序安装引擎、底盘、车轮、车身等。
4.  **当一个对象的构建过程涉及多个步骤，并且有些步骤是可选的或者可以有不同的实现方式时。**
    *   `Builder` 接口可以定义所有可能的步骤，而 `ConcreteBuilder` 可以选择性地实现或提供不同的实现。

# 给Java初学者的提示

*   **理解Director的作用是封装“构建顺序”**:
    *   `Director` 类中的 `construct()` 方法是关键。它定义了“先做什么，后做什么”的稳定流程。这个流程对于所有类型的 `Builder` 都是一样的。
    *   如果没有 `Director`，客户端代码就需要自己负责调用 `Builder` 的各个方法，并且要保证调用顺序的正确性。`Director` 将这个顺序封装起来，简化了客户端。

*   **理解Builder的作用是“实现每一步的具体构建逻辑”和“提供最终产品”**:
    *   抽象 `Builder` 定义了“有哪些构建步骤”（如 `makeTitle`, `makeString`）。
    *   具体 `ConcreteBuilder` (如 `HTMLBuilder`, `TextBuilder`) 负责实现这些步骤具体怎么做（例如，`HTMLBuilder` 的 `makeTitle` 是生成 `<h1>` 标签，而 `TextBuilder` 的 `makeTitle` 是添加文本分隔符和标题内容）。
    *   `ConcreteBuilder` 还负责提供获取最终构建结果的方法（`getResult()`）。

*   **注意本例中 `HTMLBuilder` 和 `TextBuilder` 内部的状态管理 (`isCalledTitle`)**:
    *   这是一个很好的实践，用于确保构建步骤按照预期的依赖关系进行。例如，必须先调用 `makeTitle` 才能调用 `makeString` 或 `makeItems`。
    *   这种状态管理使得 `Builder` 的使用更加健壮，可以防止因错误的调用顺序导致构建出不正确的产品或程序异常。

*   **思考如果没有Director，客户端将如何直接使用Builder**:
    *   在某些情况下，如果构建一个特定产品的步骤不是非常固定，或者客户端更愿意灵活地控制构建的每一步，那么可以不使用 `Director`。
    *   客户端可以直接实例化一个 `ConcreteBuilder`，然后根据需要调用其 `makeXxx()` 方法，最后调用 `getResult()` 获取产品。
        ```java
        // 不使用 Director 的示例
        TextBuilder textBuilder = new TextBuilder();
        textBuilder.makeTitle("My Custom Document");
        textBuilder.makeString("This is the first paragraph.");
        // ... 可能根据条件决定是否调用 makeItems ...
        // textBuilder.makeItems(new String[]{"Item A", "Item B"});
        textBuilder.makeString("Another paragraph.");
        textBuilder.close();
        String customText = textBuilder.getResult();
        System.out.println(customText);
        ```
    *   这种方式牺牲了构建过程的封装性，但增加了灵活性。建造者模式的核心仍然是 `Builder` 和 `ConcreteBuilder` 对构建步骤和产品表示的分离。`Director` 只是一个可选的辅助角色，用于封装固定的构建流程。

通过理解这些角色如何协作，以及它们各自的职责，Java初学者可以更好地掌握建造者模式，并将其应用于需要灵活构建复杂对象的场景。
