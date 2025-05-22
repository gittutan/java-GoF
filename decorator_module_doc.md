# 模块名称

装饰器模式 (Decorator Pattern)

# 核心概念 (针对Java初学者)

**什么是装饰器模式？**

装饰器模式（Decorator Pattern）是一种结构型设计模式。它的主要目的是**动态地给一个对象添加一些额外的职责（功能或行为）**。就增加功能而言，装饰器模式相比于通过创建子类来实现更为灵活。

想象一下，你有一个基础对象，你希望在不修改这个对象本身代码的前提下，给它增加一些新的特性。装饰器模式允许你像“包装”一样，一层一层地给这个对象套上新的功能。

**类比：给人穿衣服**

这个模式最经典的类比就是给人穿衣服：

1.  **人 (ConcreteComponent - 具体组件)**:
    *   这是我们的核心对象，一个“赤裸裸”的人。这个人本身有一些基本行为，比如能说话、能走路。
2.  **衣服 (Decorator - 装饰器)**:
    *   现在我们想给人增加一些“装饰”或“功能”。每一件衣服都可以看作一个装饰器。
    *   **T恤 (DecoratorA)**: 穿上T恤，人增加了“美观”的特性，也可能稍微增加了一点“保暖”功能。T恤本身是穿在人身上的，它包裹着人。
    *   **外套 (DecoratorB)**: 在T恤外面再穿上一件外套。外套进一步增加了“保暖”和“防风”的功能。外套是穿在“已经穿了T恤的人”身上的。
    *   **帽子 (DecoratorC)**: 再戴上一顶帽子。帽子增加了头部的“保暖”或“遮阳”功能。帽子是戴在“已经穿了T恤和外套的人”的头上。

在这个过程中：
*   **动态添加**: 我们可以根据需要选择穿哪些衣服，穿几件。比如天热只穿T恤，天冷就穿T恤+外套+帽子。这些功能是动态添加上去的。
*   **保持接口一致**: 无论穿了多少件衣服，这个人本质上还是“人”，他依然能执行“人”的基本行为（比如走路）。装饰器模式确保被装饰后的对象仍然保持原有对象的接口。
*   **灵活性**: 比起为每一种可能的衣物组合都创建一个新的人的“子类”（比如“穿T恤的人类”、“穿T恤和外套的人类”），用装饰器模式显然更灵活，避免了类的数量爆炸。
*   **透明性**: 对于外部观察者来说，它可能只是在与一个“经过打扮的人”交互，而不一定需要知道这个人具体穿了哪些层次的衣服。

装饰器模式就是用这种“包装”的思想，在运行时透明地给对象添加额外的职责。

# 模式结构与角色分析

装饰器模式主要包含以下角色：

## Component (组件接口/抽象类 `Display.java`)

*   **作用**: 定义一个对象接口，可以给这些对象动态地添加职责。它是被装饰者和所有装饰器的共同父（基）类或接口。
*   **代码分析 (`Display.java`)**:
    *   这是一个抽象类，定义了所有“显示内容”的类的基本规范。
    *   `public abstract int getColumns();`: 抽象方法，用于获取显示内容所需的字符宽度（列数）。
    *   `public abstract int getRows();`: 抽象方法，用于获取显示内容所需的行数。
    *   `public abstract String getRowText(int row);`: 抽象方法，用于获取指定行的文本内容。
    *   `public final void show()`:
        *   这是一个**具体方法**，并且是 `final` 的，意味着子类不能覆盖它。
        *   它定义了将所有行打印到控制台的标准流程：
            ```java
            public final void show() {
                for (int i = 0; i < getRows(); i++) { // 遍历所有行
                    System.out.println(getRowText(i)); // 获取每一行的文本并打印
                }
            }
            ```
        *   这个方法依赖于子类对 `getRows()` 和 `getRowText()` 的具体实现。

## ConcreteComponent (具体组件 `StringDisplay.java`)

*   **作用**: 定义了一个具体的对象，这是我们希望动态添加职责的原始对象。它实现了 `Component` 接口。
*   **代码分析 (`StringDisplay.java`)**:
    *   此类继承自 `Display`，用于显示单行字符串。
    *   `private String string;`: 存储要显示的字符串内容。
    *   `public StringDisplay(String string)`: 构造函数，接收要显示的字符串。
    *   `@Override public int getColumns()`:
        *   返回字符串的字节长度作为列数。注意 `string.getBytes().length` 对于中文字符可能不完全符合视觉宽度，但这是示例中的实现方式。
    *   `@Override public int getRows()`:
        *   因为是单行字符串，所以总是返回 `1`。
    *   `@Override public String getRowText(int row)`:
        *   如果请求的是第 `0` 行（唯一的一行），则返回存储的字符串 `string`。
        *   否则（请求其他行），返回 `null`。

## Decorator (装饰器抽象类 `Border.java`)

*   **作用**:
    *   继承自 `Component` (在本例中是 `Display`)，这意味着它与被装饰的对象有相同的类型。
    *   持有一个指向 `Component` 对象的引用（即 `display` 字段），这个引用指向它所装饰的“被包装”对象。
    *   通常，`Decorator` 会将所有来自客户端的请求委派给它所包装的 `Component` 对象，并且可以在委派前后添加自己的逻辑。
*   **代码分析 (`Border.java`)**:
    *   这是一个抽象类，作为所有具体边框装饰器的父类。
    *   `public abstract class Border extends Display`: 它继承了 `Display`，所以 `Border` 也是一种 `Display`。
    *   `protected Display display;`: **核心字段**，用于保存被装饰的 `Display` 对象（可以是 `StringDisplay`，也可以是另一个已经装饰过的 `Border` 对象）。
    *   `protected Border(Display display)`:
        *   构造函数，接收一个 `Display` 对象（被装饰者），并将其保存在 `display` 字段中。
        *   `protected` 访问修饰符意味着只有 `Border` 的子类（即具体的边框装饰器）才能调用这个构造函数来创建实例。
    *   **重要**: `Border` 类本身通常不直接实现 `Component` 的所有抽象方法。它可能会将这些方法的实现延迟到其具体的 `ConcreteDecorator` 子类中。如果 `Border` 实现了某些方法，它通常会直接调用 `display` 对象的相应方法。在这个例子中，`Border` 并没有实现 `getColumns`, `getRows`, `getRowText`，这些都由其子类 `SideBorder` 和 `FullBorder` 来实现。

## ConcreteDecorator (具体装饰器类 `SideBorder.java`, `FullBorder.java`)

*   **作用**: 负责给 `Component` 对象“贴上”附加的职责。它继承自 `Decorator` (在本例中是 `Border`)，并重写父类的方法以加入自己的装饰逻辑，同时调用被包装对象（通过 `display` 字段访问）的相应方法。
*   **代码分析 (`SideBorder.java`)**:
    *   此类用于在被装饰的 `Display` 内容的左右两侧添加指定的边框字符。
    *   `private char borderChar;`: 存储用作边框的字符。
    *   `public SideBorder(Display display, char ch)`: 构造函数，接收被装饰的 `Display` 对象和边框字符。
    *   `@Override public int getColumns()`:
        *   计算列数：被装饰对象的列数 `display.getColumns()` 加上左右各一个边框字符，所以是 `1 + display.getColumns() + 1`。
    *   `@Override public int getRows()`:
        *   行数与被装饰对象的行数相同：`display.getRows()`。
    *   `@Override public String getRowText(int row)`:
        *   获取指定行的文本：在被装饰对象对应行的文本 `display.getRowText(row)` 的左右两侧各加上 `borderChar`。
            ```java
            return borderChar + display.getRowText(row) + borderChar;
            ```

*   **代码分析 (`FullBorder.java`)**:
    *   此类用于在被装饰的 `Display` 内容的上下左右都添加边框。
    *   `public FullBorder(Display display)`: 构造函数，接收被装饰的 `Display` 对象。
    *   `@Override public int getColumns()`:
        *   列数：被装饰对象的列数 `display.getColumns()` 加上左右各一个边框字符 `'|'`，所以是 `1 + display.getColumns() + 1`。
    *   `@Override public int getRows()`:
        *   行数：被装饰对象的行数 `display.getRows()` 加上上下各一行边框，所以是 `1 + display.getRows() + 1`。
    *   `@Override public String getRowText(int row)`:
        *   **第0行 (上边框)**: `if (row == 0)`，返回 `"+"` + 由 `display.getColumns()` 个 `'-'` 组成的线条 + `"+"`。
        *   **最后一行 (下边框)**: `else if (row == display.getRows() + 1)`，与上边框相同。
        *   **中间内容行**: `else`，返回 `"|"` + 被装饰对象的前一行内容（因为上下边框各占一行，所以是 `display.getRowText(row - 1)`） + `"|"`。
    *   `private String makeLine(char ch, int count)`:
        *   一个辅助方法，用于生成一个包含 `count` 个 `ch` 字符的字符串。被 `getRowText` 用来创建上下边框的横线。

## Client (客户端 `Main.java`)

*   **作用**: 创建 `ConcreteComponent` 对象，然后用一个或多个 `ConcreteDecorator` 对象来包装（装饰）它。客户端通过 `Component` 接口与装饰后的对象交互，而不需要关心它具体被装饰了多少层或哪些装饰。
*   **代码分析 (`Main.java`)**:
    *   `public static void main(String[] args)`:
        *   `Display d1 = new StringDisplay("Hello, World");`
            *   创建一个具体的组件对象 `d1`，它是一个 `StringDisplay`，内容是 "Hello, World"。
        *   `Display d2 = new SideBorder(d1, '#');`
            *   创建一个 `SideBorder` 装饰器 `d2`。
            *   **装饰**: `d1` (即 `StringDisplay` 对象) 被作为参数传递给了 `SideBorder` 的构造函数，这意味着 `d2` 装饰了 `d1`。现在 `d2` 的行为会在 `d1` 的行为基础上添加左右 '#' 边框。
        *   `Display d3 = new FullBorder(d2);`
            *   创建一个 `FullBorder` 装饰器 `d3`。
            *   **再次装饰**: `d2` (即已经添加了左右 '#' 边框的 `StringDisplay`) 被作为参数传递给了 `FullBorder` 的构造函数。这意味着 `d3` 装饰了 `d2`。现在 `d3` 的行为会在 `d2` 的行为基础上（即带左右 '#' 边框的 "Hello, World"）再添加上下左右的完整边框。
        *   **调用 `show()` 方法**:
            ```java
            d1.show();
            d2.show();
            d3.show();
            ```
            *   `d1.show()`: 输出原始的 "Hello, World"。
            *   `d2.show()`: 输出 `#Hello, World#`。
            *   `d3.show()`: 输出类似如下的完整边框包裹的内容：
                ```
                +-------------+
                |#Hello, World#|
                +-------------+
                ```
            客户端代码通过 `Display` 接口统一调用 `show()` 方法，而每个对象（原始的、装饰了一层的、装饰了两层的）都会展现出其被装饰后的行为。这种层层嵌套的装饰方式是装饰器模式的典型用法。

# 装饰器模式的优点

1.  **灵活性高，动态添加/删除职责**:
    *   装饰器模式比静态继承更灵活。可以在运行时根据需要给对象添加新的行为，或者移除已添加的行为（如果装饰器支持移除或有反向操作）。
    *   可以对一个对象进行多次装饰，每次装饰都增加新的功能。

2.  **避免类爆炸**:
    *   如果不使用装饰器模式，而是通过继承来为对象添加各种组合的功能，可能会导致子类的数量急剧增加。例如，如果有一个基础组件和三种可选功能，每种功能都可以独立存在或组合存在，那么可能需要 `2^3 = 8` 个子类（包括没有额外功能的基类）。而使用装饰器，只需要1个基础组件类和3个装饰器类。
    *   装饰器模式用较少的类就可以实现多种功能的组合。

3.  **装饰者和被装饰者可以独立发展，耦合度低**:
    *   被装饰的组件（`ConcreteComponent`）不需要知道装饰器（`Decorator`）的存在。
    *   装饰器也只需要知道它所装饰的对象的接口（`Component`），而不需要知道具体的组件类型。
    *   可以独立地增加新的具体组件类和新的具体装饰器类。

4.  **装饰后的对象保持了原对象的接口**:
    *   由于装饰器和被装饰对象都实现了同一个 `Component` 接口，所以对于客户端来说，装饰后的对象和原始对象在使用上没有区别，客户端可以透明地使用它们。

# 装饰器模式的缺点

1.  **多层装饰比较复杂**:
    *   如果一个对象被很多层的装饰器包装，那么这个对象的结构会变得比较复杂。
    *   代码的阅读和调试可能会变得困难，因为一个方法的调用可能会经过多个装饰器的转发。
    *   追踪一个特定行为的来源可能会比较麻烦。

2.  **可能产生许多小对象**:
    *   使用装饰器模式通常会产生很多细粒度的小对象（即各种装饰器实例）。如果过度使用，可能会导致系统中存在大量的小对象，增加系统的复杂度。

3.  **具体组件类和装饰类之间可能存在接口不一致的问题（如果装饰器增加了新的公共方法）**:
    *   装饰器模式的核心是保持 `Component` 接口的一致性。如果一个具体的装饰器类添加了新的、不属于 `Component` 接口的 `public` 方法，那么客户端只有在知道它正在使用的是这个特定的装饰器类型时才能调用这些新方法。如果客户端仍然通过 `Component` 接口来引用对象，那么这些新方法是不可见的。
    *   这在某种程度上破坏了透明性。如果确实需要添加新接口，可能需要考虑其他模式（如适配器模式）或重新设计。

# 适用场景

装饰器模式主要适用于以下情况：

1.  **在不影响其他对象的情况下，以动态、透明的方式给单个对象添加职责。**
    *   当你希望在运行时根据条件给对象添加功能，或者这些功能可以动态撤销时。
2.  **需要扩展一个类的功能，但通过继承并不合适或不灵活时。**
    *   当子类的数量会因为功能的各种可能组合而爆炸式增长时。
    *   当一个类的定义可能是隐藏的，或者由于其他原因不能通过子类化来扩展时。
3.  **当一个对象的某些职责可以在运行时动态地添加或删除。**
    *   虽然本例主要展示添加职责，但如果设计得当（例如，装饰器链可以被修改），也可以实现职责的动态移除。
4.  **需要为一批相关的兄弟类进行统一的改装或加装功能。**
    *   例如，为一组不同的UI控件（按钮、文本框、列表）添加统一的边框或滚动条功能。

典型的应用场景包括：
*   Java I/O 类库（如 `FileInputStream` 被 `BufferedInputStream` 装饰，再被 `DataInputStream` 装饰）。
*   GUI框架中为组件添加边框、滚动条、背景等。
*   在不修改原有代码的情况下，为类增加日志记录、权限校验、性能监控等横切关注点。

# 给Java初学者的提示

*   **核心在于“包装”和“接口一致性”**:
    *   装饰器类 (`Border`) 和被装饰的组件类 (`StringDisplay`) 都实现了同一个接口 (`Display`) 或继承自同一个抽象类。这是确保客户端可以透明对待它们的关键。
    *   装饰器类内部**一定**会持有一个被装饰组件的引用（在本例中是 `Border` 类中的 `protected Display display;` 字段）。

*   **装饰器方法的典型实现模式**:
    *   装饰器的方法（如 `getColumns()`, `getRows()`, `getRowText()`）通常会：
        1.  调用它所持有的被装饰组件的对应方法（例如 `display.getColumns()`）。
        2.  然后在其结果基础上加入自己的“装饰”逻辑（例如，列数加2，行文本两边加字符）。

*   **可以像套娃一样层层嵌套装饰器**:
    *   `new FullBorder(new SideBorder(new StringDisplay("Hello")))` 就是一个很好的例子。
    *   `StringDisplay` 是最内层的“娃”（具体组件）。
    *   `SideBorder` 是包裹 `StringDisplay` 的第一层“娃”（装饰器）。
    *   `FullBorder` 是包裹 `SideBorder` 的第二层“娃”（也是装饰器）。
    *   你可以根据需要继续添加更多的装饰层。

*   **与继承的区别**:
    *   **继承**是在编译时静态地为类添加功能，并且会创建一个新的子类。如果功能组合很多，会导致子类数量爆炸。
    *   **装饰器模式**是在运行时动态地为对象添加功能，不需要创建很多子类，更加灵活。

*   **Java I/O 中的经典应用**:
    *   当你使用 `new BufferedReader(new FileReader("file.txt"))` 时：
        *   `FileReader` (类似于 `StringDisplay`) 是一个具体的组件，提供了基本的从文件读取字符的功能。
        *   `BufferedReader` (类似于 `SideBorder` 或 `FullBorder`) 是一个装饰器，它“包装”了 `FileReader`，并为其添加了缓冲功能，提高了读取效率。
    *   你可以进一步嵌套，如 `new LineNumberReader(new BufferedReader(new FileReader(...)))`，`LineNumberReader` 又为 `BufferedReader` 增加了行号追踪的功能。

理解了这种“包装”和“递归组合”的思想，就能很好地掌握装饰器模式。
