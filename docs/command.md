# 模块名称: 命令模式 (Command Pattern)

## 1. 核心概念 (针对Java初学者)

**什么是命令模式？**

命令模式（Command Pattern）是一种行为设计模式，它**将一个请求封装为一个对象**。这样做的好处是：
1.  可以用不同的请求对客户进行**参数化**：例如，一个按钮（客户）可以被配置为执行不同的操作（不同的命令对象）。
2.  可以对请求进行**排队**或**记录请求日志**。
3.  可以支持**可撤销的操作**（Undo/Redo）。

简单来说，命令模式将“行为请求者”和“行为实现者”解耦。请求者只需要知道如何发出命令，而不需要知道命令具体是如何被执行的。命令对象本身知道如何调用正确的接收者来完成操作。

**生活中的类比：餐厅点餐**

*   **顾客 (Client)**: 你来到餐厅，想点菜。
*   **菜单/点餐App (Command Interface/Concrete Commands)**: 菜单上有各种菜品（如“宫保鸡丁”、“鱼香肉丝”）。每一道菜都可以看作一个具体的命令。当你选择一道菜时，就创建了一个“点这道菜”的命令对象。这个命令对象封装了“要做什么菜”以及可能需要的参数（如“宫保鸡丁，少放辣”）。
*   **服务员 (Invoker)**: 你把选好的菜单（或在App上点击下单）交给服务员。服务员不亲自做菜，他只负责接收订单并将其传递给后厨。服务员可以把多个订单排队。
*   **厨师 (Receiver)**: 厨师是实际执行做菜动作的人。他接收到服务员传递过来的具体订单（命令），然后按照订单上的要求（如菜名、口味）去烹饪。

在这个过程中，顾客不需要知道厨师是谁、具体怎么做菜。服务员也不需要知道菜的具体做法。订单（命令对象）在它们之间传递，并最终由厨师执行。

## 2. 模式结构与角色分析

本项目的 `Command/src/` 目录（及其子目录 `command/` 和 `drawer/`）提供了一个图形化界面的画点程序，很好地展示了命令模式的应用。

*   **Command (命令接口 `command/Command.java`)**:
    *   **作用**: 声明所有具体命令类必须实现的接口，通常只包含一个核心的执行方法。
    *   **代码分析 (`command/Command.java`)**:
        ```java
        package command;
        public interface Command {
            public abstract void execute(); // 定义了执行命令的统一方法
        }
        ```

*   **ConcreteCommand (具体命令类)**:
    *   这些类实现了 `Command` 接口，将一个接收者对象与一个或多个动作绑定起来。它们存储了执行动作所需的全部信息，包括接收者对象、方法名以及方法的参数。
    *   **`drawer/DrawCommand.java`**:
        *   **作用**: 封装了在画布上绘制一个点的请求。
        *   **代码分析**:
            ```java
            package drawer;
            import command.Command;
            import java.awt.Point; // 注意这里实际例子中可能没有导入Point，而是直接使用e.getPoint()

            public class DrawCommand implements Command {
                protected Drawable drawable; // 接收者 (DrawCanvas)
                private Point position;    // 请求的参数 (绘制位置)

                public DrawCommand(Drawable drawable, Point position) { // 构造时传入接收者和参数
                    this.drawable = drawable;
                    this.position = position;
                }

                @Override
                public void execute() {
                    drawable.draw(position.x, position.y); // 调用接收者的action方法
                }
            }
            ```
            `DrawCommand`持有一个`Drawable`（画布）的引用和要绘制的`Point`（坐标）。当`execute()`被调用时，它命令画布在指定位置绘制。
    *   **`command/MacroCommand.java`**:
        *   **作用**: 一个复合命令，它可以包含一系列其他的 `Command` 对象。它也实现了 `Command` 接口，所以对于调用者来说，一个宏命令和一个简单命令没有区别。
        *   **代码分析**:
            ```java
            package command;
            import java.util.Iterator;
            import java.util.Stack; // 使用栈来存储命令，方便实现简单的undo

            public class MacroCommand implements Command {
                private Stack<Command> commands = new Stack<>(); // 存储一系列命令

                @Override
                public void execute() { // 执行所有子命令
                    Iterator<Command> it = commands.iterator();
                    while (it.hasNext()) {
                        it.next().execute();
                    }
                }
                public void append(Command cmd) { // 添加子命令
                    if (cmd != this) { commands.push(cmd); } // 防止把自己添加进去
                }
                public void undo() { // 撤销最后一条命令
                    if (!commands.empty()) { commands.pop(); }
                }
                public void clear() { // 清空所有子命令
                    commands.clear();
                }
            }
            ```
            `MacroCommand`允许将多个命令组合起来，当`MacroCommand`的`execute()`被调用时，它会按顺序执行所有内部存储的命令。它还提供了`append`, `undo` (移除最后加入的命令), 和 `clear` 方法来管理这个命令集合。

*   **Receiver (接收者)**:
    *   **作用**: 知道如何实施与执行一个请求相关的操作。任何类都可能作为一个接收者。
    *   **`drawer/Drawable.java` (接收者接口)**:
        ```java
        package drawer;
        public interface Drawable {
            public abstract void draw(int x, int y); // 定义了接收者可以执行的画点动作
        }
        ```
    *   **`drawer/DrawCanvas.java` (具体接收者)**:
        *   **作用**: 实现了 `Drawable` 接口，是真正执行画点操作的对象。
        *   **代码分析**:
            ```java
            package drawer;
            import command.MacroCommand;
            import java.awt.*; // AWT相关的类

            public class DrawCanvas extends Canvas implements Drawable {
                private Color color = Color.red;
                private int radius = 6;
                private MacroCommand history; // 持有一个MacroCommand作为历史记录

                public DrawCanvas(int width, int height, MacroCommand history) {
                    setSize(width, height);
                    setBackground(Color.white);
                    this.history = history; // 接收传递过来的历史命令集合
                }

                // 当Canvas需要重绘时 (例如窗口大小改变，或调用repaint())，此方法会被AWT调用
                public void paint(Graphics g) {
                    history.execute(); // 执行历史记录中的所有命令来重绘画布内容
                }

                @Override
                public void draw(int x, int y) { // 实现Drawable接口的画点方法
                    Graphics g = getGraphics(); // 获取画笔
                    g.setColor(color);
                    g.fillOval(x - radius, y - radius, radius * 2, radius * 2); // 画一个红点
                }
            }
            ```
            `DrawCanvas` 不仅知道如何画一个点 (`draw`方法)，还持有一个 `MacroCommand` 类型的 `history` 对象。这个 `history` 对象存储了所有已经执行过的绘制命令。当画布需要重绘时（`paint`方法被调用），它会重新执行 `history` 中的所有命令，从而恢复画布上的图像。

*   **Invoker (调用者)**:
    *   **作用**: 要求命令对象执行其请求。调用者不直接与接收者交互，而是通过命令对象。
    *   **说明**: 在本例中，`Main.java` 类中的**事件监听器方法**扮演了调用者的角色。
        *   `mouseDragged(MouseEvent e)`: 当用户在画布上拖动鼠标时，这个方法被触发。它创建一个 `DrawCommand`，将其添加到 `history`，并立即执行该命令。
        *   `actionPerformed(ActionEvent e)`: 当用户点击 "Clear" 按钮时，这个方法被触发。它调用 `history.clear()`（这也是一个命令的执行，只不过是针对`MacroCommand`的内部管理），然后重绘画布。

*   **Client (客户端 `Main.java`)**:
    *   **作用**: 负责创建具体命令对象，并设置其接收者。客户端还可能决定何时以及如何将命令对象传递给调用者（或者客户端本身就包含调用者逻辑）。
    *   **代码分析 (`Main.java`)**:
        ```java
        // ... import ...
        public class Main extends JFrame implements ActionListener, MouseMotionListener, WindowListener {
            private MacroCommand history = new MacroCommand(); // 历史命令 (也是一个复合命令)
            private DrawCanvas canvas = new DrawCanvas(400, 400, history); // 接收者，并传入history
            private JButton clearButton = new JButton("Clear"); // 一个触发命令的UI元素

            public Main(String title) { /* ...设置GUI，添加监听器... */ }

            // ActionListener for clearButton (Invoker part)
            @Override
            public void actionPerformed(ActionEvent e) {
                if (e.getSource() == clearButton) {
                    history.clear(); // 清空历史命令
                    canvas.repaint(); // 请求重绘，会间接执行空的history
                }
            }

            // MouseMotionListener for canvas (Invoker part)
            @Override
            public void mouseDragged(MouseEvent e) {
                // 创建具体命令，设置接收者(canvas)和参数(e.getPoint())
                Command cmd = new DrawCommand(canvas, e.getPoint());
                history.append(cmd); // 将命令添加到历史记录
                cmd.execute();       // 立即执行命令以在屏幕上画点
            }
            // ...其他WindowListener方法和main方法...
        }
        ```
        `Main` 类初始化了 `DrawCanvas`（接收者）和 `MacroCommand`（用于存储历史记录）。当用户与GUI交互时（拖动鼠标或点击按钮），相应的监听器方法会创建 `DrawCommand` 或调用 `MacroCommand` 的方法，从而触发命令的执行和记录。

## 3. 命令模式的优点

*   **解耦请求者和实现者**: 调用者（Invoker）不需要知道接收者（Receiver）是谁，也不需要知道操作是如何执行的。它只需要知道如何执行一个命令。同样，接收者也不知道调用者的存在。
*   **可扩展性高**: 增加新的命令非常容易，只需要创建一个新的 `ConcreteCommand` 类即可。
*   **支持复合命令**: 可以将多个简单的命令组合成一个复杂的命令（如本例中的 `MacroCommand`），使得可以像处理单个命令一样处理一组命令。
*   **方便实现请求的排队、日志、撤销/重做**:
    *   **排队/日志**: 命令对象本身可以被存储起来，用于后续执行或记录。`history` 对象就扮演了这个角色。
    *   **撤销/重做**: 如果命令接口包含 `unexecute()` 或 `undo()` 方法，并且具体命令类实现了它（例如，保存操作前的状态），那么就可以实现撤销和重做功能。`MacroCommand` 中的 `undo()` 提供了一个非常基础的撤销（移除最后一条命令）。

## 4. 适用场景

*   **当需要将请求的调用者和执行者解耦时。** (例如，GUI按钮点击后执行某个操作，按钮本身不关心操作如何实现)。
*   **当需要支持撤销/重做操作时。** (命令对象可以保存执行操作所需的状态，以便恢复)。
*   **当需要将操作封装成对象，以便进行参数化、排队、记录日志或通过网络传输时。**
*   **当需要实现回调机制时，即一个对象调用另一个对象的方法，但被调用方法的具体实现在运行时才确定。**
*   **当需要将一组操作组合成一个更高级别的操作时（宏命令）。**

## 5. 给Java初学者的提示

*   **核心思想是“封装请求”**: 命令对象就像一个包含了“要做什么”、“谁来做”、“需要什么参数”的小包裹。
*   **`MacroCommand` 是组合模式的应用**: `MacroCommand` 同时是 `Command` (所以可以被执行和嵌套) 并且包含其他 `Command` (所以能组合)。
*   **巧妙的重绘机制**: `DrawCanvas` 的 `paint()` 方法通过执行 `history` 中的所有命令来重绘整个画布，这是一个非常优雅的实现。它确保了即使窗口被遮挡后重新显示，之前绘制的内容也能恢复。这体现了命令模式在状态管理和恢复方面的能力。
*   **区分Invoker和Client**:
    *   **Client**: 负责创建命令对象，并设置好命令的接收者和参数。在本例中，`Main` 类在初始化和事件处理方法中都扮演了部分Client的角色。
    *   **Invoker**: 负责触发命令的执行 (`command.execute()`)。在本例中，是 `Main` 类的事件监听器方法（如 `mouseDragged` 直接调用 `cmd.execute()`）和AWT的绘图机制（调用 `canvas.paint()` 间接触发 `history.execute()`）。
*   **Undo/Redo的实现**: 要实现更完善的Undo/Redo，`DrawCommand` 需要能够撤销其效果（例如，知道如何擦除一个点，或者保存画布之前的某个区域状态）。`MacroCommand` 的 `undo` 也需要更复杂，比如能够调用子命令的 `unexecute` 方法。

命令模式通过引入命令对象，极大地提高了系统的灵活性和可扩展性，是许多复杂系统中常用的设计模式。
