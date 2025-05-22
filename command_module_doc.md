# 模块名称

命令模式 (Command Pattern)

# 核心概念 (针对Java初学者)

**什么是命令模式？**

命令模式（Command Pattern）是一种行为设计模式。它的核心思想是：**将一个请求封装为一个对象，从而使你可以用不同的请求对客户进行参数化；对请求排队或记录请求日志，以及支持可撤销的操作。**

简单来说，就是把一个“要做的事情”（请求）打包成一个“命令对象”。这个对象里包含了执行该事情所需的所有信息，比如：
*   要做什么操作？
*   谁来做这个操作（接收者）？
*   操作需要哪些数据（参数）？

一旦打包好了，这个“命令对象”就可以像普通对象一样被传递、存储、排队，甚至可以被撤销（如果设计得当）。

**类比：餐厅点餐**

想象一下你在餐厅点餐的场景：

1.  **你 (Client - 客户端)**: 决定要吃什么，比如“一份宫保鸡丁，米饭，可乐”。
2.  **点菜单 (Command - 命令对象)**: 你把你的需求写在一张点菜单上。这张点菜单就封装了你的请求。它包含了：
    *   动作：制作菜品、准备米饭、倒饮料。
    *   参数：宫保鸡丁的具体做法要求（比如少放辣）、米饭的份量、可乐的品牌和是否加冰。
3.  **服务员 (Invoker - 调用者)**: 你把点菜单交给服务员。服务员不关心具体菜品怎么做，他只负责把点菜单递送到正确的地方。服务员可以把多张点菜单收集起来排队，然后统一交给厨房。
4.  **厨师 (Receiver - 接收者)**: 厨师拿到点菜单后，根据上面的内容（比如“制作宫保鸡丁”），开始实际烹饪。厨师是真正执行操作的人。

在这个过程中：
*   **请求被封装**: 你的点餐需求被封装成了“点菜单”这个对象。
*   **解耦**: 你（客户端）不直接和厨师（接收者）打交道。服务员（调用者）也不需要知道厨师具体怎么做菜。
*   **参数化**: 如果餐厅有不同的厨师（比如川菜厨师、粤菜厨师），服务员可以把同一类型的点菜单（比如“炒一份时蔬”）交给不同的厨师，从而得到不同风味的菜品。
*   **排队/日志**: 服务员可以把点菜单排队处理。餐厅也可以记录下所有点菜单作为销售日志。
*   **可撤销**: 如果你突然不想吃某道菜了，在厨师开始做之前，你可以告诉服务员“撤销那张点菜单上的宫保鸡丁”。

命令模式在软件中就是用类似的方式，将操作请求封装成对象，以实现更灵活和可扩展的系统设计。

# 模式结构与角色分析

命令模式主要包含以下角色：

## Command (命令接口 `command/Command.java`)

*   **作用**: 声明执行操作的统一接口。这个接口通常只包含一个核心方法，用于执行命令。
*   **代码分析 (`command/Command.java`)**:
    ```java
    package command;

    public interface Command {
        public abstract void execute(); // 定义了执行命令的抽象方法
    }
    ```
    接口 `Command` 非常简单，只定义了一个 `execute()` 方法。所有具体的命令类都将实现这个接口，并在这个方法中封装它们要执行的操作。

## ConcreteCommand (具体命令类)

具体命令类实现了 `Command` 接口，并负责将一个动作绑定到一个 `Receiver`（接收者）对象。当 `execute()` 方法被调用时，它会调用 `Receiver` 的相应方法来执行操作。

*   **`drawer/DrawCommand.java` (绘制命令)**:
    *   **作用**: 封装了在画布上绘制一个点的请求。
    *   **代码分析**:
        ```java
        package drawer;

        import command.Command;
        import java.awt.*; // 引入 Point 类

        public class DrawCommand implements Command {
            protected Drawable drawable; // 引用接收者 (Drawable 接口类型)
            private Point position;    // 命令执行所需的参数 (绘制的位置)

            // 构造函数，传入接收者和参数
            public DrawCommand(Drawable drawable, Point position) {
                this.drawable = drawable; // Drawable 在本例中实际是 DrawCanvas 的实例
                this.position = position;
            }

            @Override
            public void execute() {
                // 执行命令时，调用接收者的 draw 方法，并传入参数
                drawable.draw(position.x, position.y);
            }
        }
        ```
        *   `DrawCommand` 持有一个 `Drawable` 类型的引用 `drawable`，这就是命令的接收者（实际执行绘制操作的对象）。
        *   它还持有一个 `Point` 类型的引用 `position`，这是执行绘制操作所需要的参数（在哪里绘制）。
        *   当 `execute()` 方法被调用时，`DrawCommand` 会调用其 `drawable` 对象的 `draw()` 方法，并把 `position` 的x和y坐标传递过去，从而完成在画布上绘制一个点的操作。

*   **`command/MacroCommand.java` (宏命令/复合命令)**:
    *   **作用**: 一个复合命令，它可以包含一系列其他命令对象。当宏命令执行时，它会按顺序执行其包含的所有子命令。这体现了组合模式的思想。
    *   **代码分析**:
        ```java
        package command;

        import java.util.Iterator;
        import java.util.Stack; // 使用 Stack 来存储子命令

        public class MacroCommand implements Command {
            private Stack commands = new Stack(); // 用栈来存储一系列命令

            @Override
            public void execute() {
                Iterator it = commands.iterator();
                while (it.hasNext()) {
                    ((Command) it.next()).execute(); // 遍历并执行所有子命令
                }
            }

            // 添加子命令到宏命令中
            public void append(Command cmd) {
                if (cmd != this) { // 防止将自身作为子命令添加，避免无限递归
                    commands.push(cmd);
                }
            }

            // 撤销最后添加的命令 (简单的undo功能)
            public void undo() {
                if (!commands.empty()) {
                    commands.pop(); // 从栈中移除最后一个命令
                }
            }

            // 清空所有子命令
            public void clear() {
                commands.clear();
            }
        }
        ```
        *   `MacroCommand` 内部使用一个 `Stack` 来存储一系列 `Command` 对象。
        *   `execute()` 方法会遍历这个 `Stack`（虽然 `Stack` 的 `iterator()` 顺序是从底部到顶部，但对于本例绘制历史而言，顺序通常是添加的顺序，所以效果上是按添加顺序执行），并调用每个子命令的 `execute()` 方法。
        *   `append(Command cmd)`: 用于向宏命令中添加新的子命令。
        *   `undo()`: 提供了一个简单的撤销功能，即移除最后加入的命令。
        *   `clear()`: 清空所有已存储的命令。
        *   在本例中，`MacroCommand` 被用作 `history`（历史记录），存储用户在画布上的所有绘制操作。

## Receiver (接收者)

接收者是真正知道如何执行请求中指定的操作的对象。任何类都可以作为一个接收者。

*   **`drawer/Drawable.java` (接收者接口)**:
    *   **作用**: 定义了接收者可以执行的操作的接口。
    *   **代码分析**:
        ```java
        package drawer;

        public interface Drawable {
            public abstract void draw(int x, int y); // 定义了一个绘制操作
        }
        ```
        它声明了一个 `draw(int x, int y)` 方法，任何希望能够被绘制的对象都需要实现这个接口。

*   **`drawer/DrawCanvas.java` (具体接收者)**:
    *   **作用**: 实现了 `Drawable` 接口，是真正执行绘制操作的类。它代表了一个可以进行绘制的画布。
    *   **代码分析**:
        ```java
        package drawer;

        import command.MacroCommand;
        import java.awt.*; // 引入 Canvas, Color, Graphics 等 AWT 类

        public class DrawCanvas extends Canvas implements Drawable {
            private Color color = Color.red;    // 默认绘制颜色
            private int radius = 6;             // 默认绘制点的半径
            private MacroCommand history;       // 持有一个 MacroCommand 作为历史记录

            // 构造函数，传入画布大小和历史记录对象
            public DrawCanvas(int width, int height, MacroCommand history) {
                setSize(width, height);
                setBackground(Color.white);
                this.history = history; // 接收外部传入的 history 对象
            }

            // Canvas 的 paint 方法，在需要重绘时被调用
            public void paint(Graphics g) {
                history.execute(); // 通过执行 history 中的所有命令来重绘画布
            }

            @Override
            public void draw(int x, int y) { // 实现 Drawable 接口的 draw 方法
                Graphics g = getGraphics();
                if (g != null) { // 确保 Graphics 对象可用
                    g.setColor(color);
                    g.fillOval(x - radius, y - radius, radius * 2, radius * 2); // 绘制一个实心圆点
                }
            }
        }
        ```
        *   `DrawCanvas` 继承自 `java.awt.Canvas` 并实现了 `Drawable` 接口。
        *   它持有一个 `MacroCommand` 类型的 `history` 对象。这个 `history` 对象（在 `Main` 类中创建并传入）存储了所有的绘制命令。
        *   `draw(int x, int y)` 方法是实际的绘制逻辑，它获取 `Graphics` 对象并在指定位置绘制一个红色的圆点。
        *   **关键点**: `paint(Graphics g)` 方法。当画布需要重绘时（例如窗口大小改变、或者调用了 `repaint()`），这个方法会被AWT框架调用。它的实现是 `history.execute()`，即重新执行历史记录中的所有绘制命令，从而恢复画布上的所有图形。

## Invoker (调用者)

调用者负责发起请求。它持有一个命令对象，并在需要时调用命令对象的 `execute()` 方法。调用者不直接知道接收者是谁，也不知道具体的操作是什么，它只与命令对象交互。

*   **作用**: 要求命令执行请求。
*   **说明**: 在本例中，并没有一个单独的 `Invoker` 类。调用者的角色是由 `Main.java` 中的**事件监听器方法**扮演的：
    *   `mouseDragged(MouseEvent e)`: 当用户在画布上拖动鼠标时，这个方法被调用。它会创建一个新的 `DrawCommand`，将其添加到 `history`（一个 `MacroCommand` 实例）中，并立即执行该 `DrawCommand`（使得新点被画出）。
    *   `actionPerformed(ActionEvent e)`: 当用户点击 "Clear" 按钮时，这个方法被调用。它会清空 `history` 中的所有命令，并调用 `canvas.repaint()` 来刷新画布（此时 `canvas.paint()` 会执行空的 `history`，从而清空画布）。

    这些方法根据用户的交互创建命令对象，并决定何时执行它们（或将它们存储起来供后续执行，如 `history`）。

## Client (客户端 `Main.java`)

客户端负责创建整个命令模式的组件：创建具体命令对象，设置其接收者，创建调用者（如果需要），并将命令对象传递给调用者。

*   **作用**: 组装命令模式的各个部分。
*   **代码分析 (`Main.java`)**:
    *   **创建 Receiver**:
        ```java
        private MacroCommand history = new MacroCommand(); // 这个 history 同时也是一个 Command (MacroCommand)
        private DrawCanvas canvas = new DrawCanvas(400, 400, history); // 创建 Receiver (DrawCanvas)，并将 history 传给它
        ```
        这里，`Main` 类创建了 `DrawCanvas`（接收者）的实例。`DrawCanvas` 在构造时接收了 `history`（一个 `MacroCommand` 实例）的引用。这个 `history` 用于记录所有绘制命令，并在画布重绘时使用。
    *   **创建 ConcreteCommand 并设置 Invoker (通过事件监听)**:
        在 `mouseDragged` 方法中：
        ```java
        public void mouseDragged(MouseEvent e) {
            Command cmd = new DrawCommand(canvas, e.getPoint()); // 1. 创建具体命令 DrawCommand
                                                                //    将 canvas (Receiver) 和鼠标位置 (参数) 传给它
            history.append(cmd);                                 // 2. 将命令添加到 history (MacroCommand) 中
            cmd.execute();                                       // 3. 立即执行该命令 (Invoker 的一部分职责)
        }
        ```
        当鼠标拖动事件发生时，客户端代码（`Main` 类）创建了一个 `DrawCommand`。它将 `canvas`（接收者）和当前鼠标位置 `e.getPoint()`（参数）传递给 `DrawCommand` 的构造函数。然后，这个命令被添加到 `history` 中，并被立即执行。
    *   **处理其他调用**:
        在 `actionPerformed` 方法中（当 "Clear" 按钮被点击时）：
        ```java
        public void actionPerformed(ActionEvent e) {
            if (e.getSource() == clearButton) {
                history.clear();        // 调用 MacroCommand 的 clear 方法
                canvas.repaint();       // 请求重绘画布
            }
        }
        ```
        这里，客户端直接调用 `history`（一个 `MacroCommand`）的 `clear()` 方法，然后通过 `canvas.repaint()` 触发画布重绘。画布的 `paint` 方法会执行 `history.execute()`，由于 `history` 已被清空，画布也会变为空白。

`Main` 类作为客户端，初始化了所有对象，包括 `DrawCanvas` (Receiver)，`MacroCommand` (history，用于存储命令)，并将它们连接起来。它还通过事件处理机制充当了部分 Invoker 的角色，根据用户操作创建并执行（或存储）命令。

# 命令模式的优点

1.  **解耦请求的发送者和接收者**:
    *   发送者（Invoker，如 `Main` 中的事件处理器）只需要知道如何调用命令的 `execute()` 方法，而不需要知道命令具体做了什么，也不需要知道是谁（Receiver）在做。
    *   接收者（Receiver，如 `DrawCanvas`）只负责执行具体的动作，它不需要知道是谁（Invoker）发起了这个动作。
    *   命令对象（Command，如 `DrawCommand`）充当了它们之间的桥梁。

2.  **可扩展性高**:
    *   **易于增加新的命令**: 如果需要新的操作，只需要创建一个新的 `ConcreteCommand` 类实现 `Command` 接口即可。例如，可以添加一个 `ChangeColorCommand` 来改变画笔颜色。
    *   **易于增加新的接收者**: 也可以方便地添加新的 `Receiver` 类。

3.  **可实现复杂操作的组合 (`MacroCommand`)**:
    *   可以将多个简单的命令组合成一个复杂的宏命令。`MacroCommand` 本身也是一个 `Command`，可以像单个命令一样被执行，这体现了组合模式的应用。
    *   本例中的 `history` 就是一个 `MacroCommand`，它存储了一系列的 `DrawCommand`。

4.  **方便实现请求的排队、日志、撤销/重做等功能**:
    *   **排队/日志**: 命令对象可以被存储起来（如在队列或日志文件中），然后在未来的某个时间点被执行。本例中的 `history` 就是一种日志/存储机制。
    *   **撤销/重做**:
        *   `MacroCommand` 中的 `undo()` 方法提供了一个非常基础的撤销（移除最后一条命令）的示例。
        *   要实现更完善的撤销/重做，每个 `ConcreteCommand` 通常需要额外实现 `unexecute()` 方法（执行与 `execute()` 相反的操作），并且需要一个更复杂的历史管理器来维护可撤销和可重做的命令栈。

# 适用场景

命令模式在以下情况下非常有用：

1.  **需要将请求的调用者和执行者解耦时**: 当一个对象需要向另一个对象发出请求，但你不想让这两个对象直接耦合。命令模式可以将请求的发起者和执行者分离开。
2.  **需要支持撤销/重做操作时**: 命令对象可以存储足够的信息（包括如何撤销自身），使得操作可以被回滚。
3.  **需要将操作封装成对象，以便进行参数化、排队、记录日志或通过网络传输时**:
    *   **参数化**: 将命令作为方法的参数。
    *   **排队**: 将待执行的命令放入队列中，由工作线程逐个取出并执行。
    *   **记录日志**: 将执行过的命令序列化到日志中，用于故障恢复或审计。
4.  **需要实现回调机制时**: 命令对象可以看作是一种回调的实现，在某个事件发生时被调用执行。
5.  **需要将一组操作组合成一个更高级的操作（宏命令）时。**

例如：
*   GUI按钮的点击事件处理。
*   文本编辑器的操作（剪切、复制、粘贴、撤销、重做）。
*   多级菜单的实现。
*   游戏中的玩家动作指令。

# 给Java初学者的提示

*   **命令对象的核心：“封装了一个动作及其参数和执行者”**:
    *   `Command` 接口通常只有一个 `execute()` 方法。
    *   `ConcreteCommand` (如 `DrawCommand`) 内部持有对“谁来做”（`Receiver`，即 `DrawCanvas`）的引用，以及“做什么需要的额外信息”（参数，即 `Point position`）。当 `execute()` 被调用时，它就调用 `Receiver` 的方法并传入参数。

*   **`MacroCommand` 展示了组合模式在命令模式中的应用**:
    *   `MacroCommand` 既是一个 `Command`（实现了 `Command` 接口），又可以包含其他 `Command` 对象（包括其他 `MacroCommand`）。
    *   这使得你可以像对待单个命令一样对待一组命令，非常灵活。例如，`history.execute()` 会执行所有历史绘制命令。

*   **`DrawCanvas` 的 `paint` 方法通过执行 `history` 中的命令来实现重绘**:
    *   这是命令模式的一个巧妙应用。当窗口需要重绘时（例如，被其他窗口遮挡后重新显示），`paint` 方法不是直接去画什么，而是简单地重新执行 `history` 这个 `MacroCommand`。
    *   `history` 中保存了用户之前的所有 `DrawCommand`。依次执行这些 `DrawCommand` 就能恢复画布之前的状态。这实际上是用命令序列来记录和恢复状态。

*   **注意区分 Invoker（何时执行）和 Client（如何组装命令和接收者）**:
    *   **Client (`Main` 类的主体部分)**: 负责创建 `DrawCommand`，并将 `DrawCanvas` 设置为其 `Receiver`。它还创建了 `MacroCommand` (history)。
    *   **Invoker (`Main` 类的事件处理方法，如 `mouseDragged` 和 `actionPerformed`)**: 负责在特定事件发生时（如鼠标拖动、按钮点击）决定是否执行一个命令（`cmd.execute()`）或如何操作命令集合（`history.append(cmd)`，`history.clear()`）。
    *   有时候，Client 和 Invoker 的角色可能在同一个类中，但理解它们概念上的区别很重要。Client 负责“组装”，Invoker 负责“触发”。

命令模式通过引入命令对象，极大地增加了系统的灵活性，使得请求的发送和执行过程更加松耦合和可控。
