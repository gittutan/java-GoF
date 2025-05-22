# 模块名称: 装饰器模式 (Decorator Pattern)

## 1. 核心概念 (针对Java初学者的)

**什么是装饰器模式？**

装饰器模式（Decorator Pattern）是一种结构型设计模式。它的核心思想是：**动态地给一个对象添加一些额外的职责（功能或行为）。** 就增加功能而言，装饰器模式相比通过继承创建子类更为灵活。

你可以把装饰器看作是一个“包装器”。你有一个核心对象，然后你可以用一个或多个装饰器把它包起来，每个装饰器都会给这个核心对象增加一点新的东西，但不会改变核心对象本身。

**生活中的类比：给人穿衣服**

*   **人 (ConcreteComponent - 具体组件)**: 这是核心，比如一个“小明”。
*   **衣服 (Component - 组件接口)**: 定义了“可以穿戴”的特性。人和装饰性的衣物都“可以穿戴”。
*   **T恤 (ConcreteDecoratorA - 具体装饰器A)**: 给小明穿上一件T恤。T恤增加了“上衣”的功能/外观。
*   **外套 (ConcreteDecoratorB - 具体装饰器_B)**: 在T恤外面再穿上一件外套。外套增加了“保暖”、“口袋”等功能。
*   **帽子 (ConcreteDecoratorC - 具体装饰器C)**: 再戴上一顶帽子。帽子增加了“头部装饰/保暖”的功能。

每一件“衣物”（装饰器）都给“小明”（具体组件）添加了新的职责或特性，而且这些衣物可以自由组合和穿戴（例如，可以只穿T恤，也可以穿T恤+外套，或者T恤+外套+帽子）。如果用继承来实现，你可能需要创建 `穿T恤的小明`、`穿T恤和外套的小明` 等很多子类，非常不灵活。

## 2. 模式结构与角色分析

本项目的 `Decorator/src/` 目录下的文件用一个给文本内容添加不同边框的例子来演示装饰器模式。

*   **Component (组件接口/抽象类 `Display.java`)**:
    *   **作用**: 定义了一个对象的统一接口，这些对象可以被动态地添加职责。它是具体组件和装饰器共同的父类。
    *   **代码分析 (`Display.java`)**:
        ```java
        public abstract class Display {
            public abstract int getColumns(); // 获取内容的宽度（字符数）
            public abstract int getRows();    // 获取内容的行数
            public abstract String getRowText(int row); // 获取指定行的文本内容

            // final确保子类不能覆盖这个显示流程
            public final void show() {
                for (int i = 0; i < getRows(); i++) {
                    System.out.println(getRowText(i));
                }
            }
        }
        ```
        `Display` 类定义了所有“可显示内容”的基本操作。`show()` 方法提供了一个统一的算法来逐行显示内容。

*   **ConcreteComponent (具体组件 `StringDisplay.java`)**:
    *   **作用**: 定义了一个具体的、可以被装饰器装饰的对象。它是装饰链的起点和核心。
    *   **代码分析 (`StringDisplay.java`)**:
        ```java
        public class StringDisplay extends Display {
            private String string; // 要显示的字符串
            public StringDisplay(String string) { this.string = string; }

            @Override
            public int getColumns() { return string.getBytes().length; } // 字符串的字节长度作为宽度
            @Override
            public int getRows() { return 1; } // 单行文本，所以行数总是1
            @Override
            public String getRowText(int row) {
                return (row == 0) ? string : null; // 只在第0行返回字符串
            }
        }
        ```
        `StringDisplay` 是一个具体的组件，它负责显示一个单行的字符串。

*   **Decorator (装饰器抽象类 `Border.java`)**:
    *   **作用**: 继承自 `Component` (这里是 `Display`)，它的作用是作为所有具体装饰器的基类。它内部持有一个指向 `Component` 对象的引用（即被装饰的对象）。
    *   **代码分析 (`Border.java`)**:
        ```java
        public abstract class Border extends Display {
            protected Display display; // 持有被装饰的Display对象 (可以是StringDisplay或其他Border)
            protected Border(Display display) { // 构造时传入被装饰对象
                this.display = display;
            }
            // Border 类通常会将 Component 的方法委托给它所持有的 display 对象来处理
            // 具体如何装饰（即如何修改这些方法的行为）由 ConcreteDecorator 实现
        }
        ```
        `Border` 类本身也是一个 `Display`，所以它可以取代被装饰的 `Display` 对象。它将所有 `Display` 的操作（如 `getColumns`, `getRows`, `getRowText`）的默认行为委托给它所包含的 `display` 对象。

*   **ConcreteDecorator (具体装饰器类 `SideBorder.java`, `FullBorder.java`)**:
    *   **作用**: 负责给 `Component` 对象“贴上”具体的附加职责（装饰）。它们会重写父类（`Decorator` 或 `Component`）的方法，在调用被装饰对象（`this.display`）的相应方法之前或之后，加上自己的装饰行为。
    *   **代码分析 (`SideBorder.java`)**:
        *   给文本左右两边添加指定的单个字符边框。
        ```java
        public class SideBorder extends Border {
            private char borderChar; // 边框字符
            public SideBorder(Display display, char ch) {
                super(display); // 调用父类Border的构造，保存被装饰对象
                this.borderChar = ch;
            }
            @Override
            public int getColumns() { // 宽度增加2 (左右各一个边框字符)
                return 1 + display.getColumns() + 1;
            }
            @Override
            public int getRows() { //行数不变
                return display.getRows();
            }
            @Override
            public String getRowText(int row) { // 在被装饰内容的左右添加边框字符
                return borderChar + display.getRowText(row) + borderChar;
            }
        }
        ```
    *   **代码分析 (`FullBorder.java`)**:
        *   给文本上下左右都添加边框。
        ```java
        public class FullBorder extends Border {
            public FullBorder(Display display) { super(display); }

            @Override
            public int getColumns() { // 宽度增加2 (左右边框'|')
                return 1 + display.getColumns() + 1;
            }
            @Override
            public int getRows() { // 行数增加2 (上下边框)
                return 1 + display.getRows() + 1;
            }
            @Override
            public String getRowText(int row) {
                if (row == 0) { // 上边框
                    return "+" + makeLine('-', display.getColumns()) + "+";
                } else if (row == display.getRows() + 1) { // 下边框 (注意这里的行号计算)
                    return "+" + makeLine('-', display.getColumns()) + "+";
                } else { // 中间内容行 (左右加 '|')
                    return "|" + display.getRowText(row - 1) + "|";
                }
            }
            private String makeLine(char ch, int count) { // 辅助方法：生成一行重复字符
                StringBuffer buf = new StringBuffer();
                for (int i = 0; i < count; i++) { buf.append(ch); }
                return buf.toString();
            }
        }
        ```

*   **Client (客户端 `Main.java`)**:
    *   **作用**: 负责创建具体组件（`ConcreteComponent`），然后根据需要用一个或多个具体装饰器（`ConcreteDecorator`）来包装（装饰）它。
    *   **代码分析 (`Main.java`)**:
        ```java
        public class Main {
            public static void main(String[] args) {
                // 1. 创建一个具体的被装饰对象 (一个只显示 "Hello, World" 的 StringDisplay)
                Display d1 = new StringDisplay("Hello, World");

                // 2. 用 SideBorder 装饰 d1 (给 "Hello, World" 两边加上 '#')
                //    d2 仍然是一个 Display 类型
                Display d2 = new SideBorder(d1, '#');

                // 3. 用 FullBorder 装饰 d2 (给已经被 SideBorder 装饰过的结果再加上完整边框)
                //    d3 也是一个 Display 类型
                Display d3 = new FullBorder(d2);

                System.out.println("d1:");
                d1.show(); // 显示原始的 StringDisplay

                System.out.println("
d2:");
                d2.show(); // 显示被 SideBorder 装饰后的结果

                System.out.println("
d3:");
                d3.show(); // 显示被 FullBorder (内含SideBorder) 装饰后的结果

                // 也可以一步到位地进行多重装饰
                Display d4 = new FullBorder(
                                new SideBorder(
                                    new StringDisplay("Hi, Decorator!"),
                                    '*'
                                )
                             );
                System.out.println("
d4:");
                d4.show();

                Display d5 = new SideBorder(
                                new FullBorder(
                                    new SideBorder(
                                        new StringDisplay("Test"),
                                        '!'
                                    )
                                ),
                                '/'
                             );
                System.out.println("
d5:");
                d5.show();
            }
        }
        ```
        客户端代码展示了如何像“套娃”一样，一层一层地用装饰器包装原始的 `StringDisplay` 对象。每次调用 `show()` 方法时，由于Java的多态性，会正确地调用到最外层装饰器的 `show()`（最终是 `Display.show()`），该方法内部又会调用 `getRowText` 等，这些调用会沿着装饰链向内传递，每一层装饰器都会加上自己的行为。

## 3. 装饰器模式的优点

*   **高度灵活性，动态添加功能**: 装饰器模式比继承更灵活。它可以在运行时动态地给对象添加或移除功能，而不需要修改对象的代码或创建大量的子类。
*   **避免类爆炸**: 如果使用继承来为对象添加多种组合功能，可能会导致子类的数量急剧增加（例如，一个“带左边框的显示”、“带右边框的显示”、“带左右边框的显示”、“带上下边框的显示”...）。装饰器模式通过组合不同的装饰器来达到同样的效果，但类的数量要少得多。
*   **装饰者和被装饰者可以独立发展**: 只要组件接口不变，可以独立地增加新的具体组件类和新的具体装饰器类，它们之间耦合度较低。
*   **装饰器可以被多次装饰**: 一个对象可以被一个或多个装饰器重复装饰。

## 4. 装饰器模式的缺点

*   **多层装饰导致对象复杂**: 过多层次的装饰可能会使得对象结构变得复杂，调试时追踪问题也可能比较困难（需要深入很多层包装）。
*   **具体组件类和装饰类之间可能存在接口不一致的问题**: 装饰器模式要求装饰器和被装饰对象实现相同的接口。如果一个具体装饰器添加了新的公共方法（而不是仅仅修改已有接口方法的行为），那么这些新方法只有通过具体装饰器类型的引用才能访问，通过组件接口类型的引用是访问不到的，这在一定程度上破坏了透明性。

## 5. 适用场景

*   **在不影响其他对象的情况下，以动态、透明的方式给单个对象添加职责。**
*   **当不能用继承或不希望用继承来扩展类的功能时。** 例如，类可能被标记为 `final`，或者扩展功能会导致子类数量失控。
*   **需要为一批相关的兄弟类进行改装或加装功能时。**
*   当对象的职责可以动态地撤销时。

## 6. 给Java初学者的提示

*   **核心关系**: 装饰器类（`Border`）和被装饰的组件类（`StringDisplay`）都必须实现同一个接口（或继承自同一个抽象父类，如本例中的 `Display`）。这是能够透明替换和嵌套的前提。
*   **“HAS-A”关系**: 装饰器类内部**必须**持有一个它所装饰的组件的引用（`protected Display display;`）。
*   **委托与增强**: 装饰器的方法在执行时，通常会调用被装饰组件的对应方法（委托），然后在其返回结果的基础上加入自己的“装饰”逻辑（增强）。
*   **可以像“套娃”一样层层嵌套**: `new DecoratorA(new DecoratorB(new ConcreteComponent()))`。最外层的对象类型仍然是它们共同的组件类型。
*   **Java I/O 类库是经典范例**: Java 的 `java.io` 包中的 `InputStream`, `OutputStream`, `Reader`, `Writer` 体系就广泛应用了装饰器模式。例如：
    *   `FileInputStream` 是一个具体的组件，可以直接读取文件。
    *   `BufferedInputStream` 是一个装饰器，它接收一个 `InputStream` 对象，并为其添加缓冲功能以提高读写效率。
    *   `DataInputStream` 也是一个装饰器，它接收一个 `InputStream`，并为其添加读取Java基本数据类型的方法（如 `readInt()`, `readDouble()`）。
    你可以这样用：`new DataInputStream(new BufferedInputStream(new FileInputStream("file.txt")))`。

装饰器模式是一种非常强大的模式，用于在不改变原有对象结构的情况下，动态地为其添加新的行为。
