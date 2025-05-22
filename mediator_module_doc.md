# 模块名称

中介者模式 (Mediator Pattern)

# 核心概念 (针对Java初学者)

**什么是中介者模式？**

中介者模式（Mediator Pattern）是一种行为设计模式。它的核心思想是：**用一个中介对象（Mediator）来封装一系列的对象（Colleagues）交互。中介者使各个对象不需要显式地相互引用（即直接通信），从而使其耦合松散，而且可以独立地改变它们之间的交互。**

想象一下，在一个复杂的系统中，有很多个对象，它们之间需要相互通信和协调工作。如果让这些对象直接互相引用和调用，很快就会形成一个非常复杂的“网状”依赖关系。任何一个对象的改变都可能影响到其他很多对象，使得系统难以维护和扩展。

中介者模式通过引入一个中心“协调员”（中介者），将这种网状的交互结构转变为“星形”结构。所有的“同事”（Colleague）对象都只和中介者通信，它们不直接与其他同事通信。当一个同事对象的状态发生改变或需要与其他同事协作时，它会通知中介者。中介者接收到通知后，再根据预设的逻辑去协调和指挥其他相关的同事对象。

**类比：机场的控制塔**

一个非常经典的类比就是机场的控制塔：

1.  **飞机 (Colleague - 同事对象)**:
    *   机场上有很多架飞机，它们需要起飞、降落、滑行等。
2.  **控制塔 (Mediator - 中介者对象)**:
    *   控制塔是机场的指挥中心。
3.  **交互方式**:
    *   **没有控制塔（网状交互）**: 想象一下，如果每架飞机都需要直接和其他所有相关的飞机通信来协调起飞跑道、降落顺序、空中避让等，那将是一片混乱，非常容易发生冲突和事故。飞行员需要时刻关注周围所有飞机的动态。
    *   **有控制塔（星形交互）**: 实际上，每架飞机都只与控制塔通信。
        *   飞机A想要起飞，它向控制塔发出请求。
        *   飞机B准备降落，它也向控制塔报告状态。
        *   控制塔根据整体情况（如跑道是否占用、空中交通状况等）向飞机A发出“允许起飞”的指令，向飞机B发出“在指定高度盘旋等待”的指令。
        *   飞机A和飞机B之间不需要直接对话。控制塔作为中介，确保了所有飞机的行动得到有效协调，避免了冲突。

在这个例子中：
*   控制塔（Mediator）封装了飞机（Colleagues）之间的复杂交互逻辑。
*   飞机（Colleagues）只依赖于控制塔（Mediator），而不相互依赖。
*   如果机场的调度规则改变了（比如引入新的起降流程），主要修改的是控制塔的逻辑，而各个飞机类的逻辑可能不需要大的改动。

中介者模式就是通过这样一个中心协调者，来降低对象之间的耦合度，使得系统更加灵活和易于维护。

# 模式结构与角色分析

中介者模式主要包含以下角色：

## Mediator (中介者接口 `Mediator.java`)

*   **作用**: 定义了一个接口，用于与各个 `Colleague` 对象进行通信。它通常包含一些方法，用于 `Colleague` 对象通知 `Mediator` 自身状态的改变，以及 `Mediator` 控制 `Colleague` 对象的行为。
*   **代码分析 (`Mediator.java`)**:
    ```java
    public interface Mediator {
        // 用于创建并注册所有的 Colleague 对象
        public abstract void createCollegues();

        // 当某个 Colleague 发生变化时被调用，以便 Mediator 进行协调
        public abstract void collegueChanged();
    }
    ```
    *   `public abstract void createCollegues();`:
        *   这个方法通常由具体中介者（`ConcreteMediator`）实现，负责实例化所有的同事（`Colleague`）对象，并将中介者自身注册给这些同事对象。
    *   `public abstract void collegueChanged();`:
        *   当一个同事对象的状态发生改变时，它会调用中介者的这个方法。
        *   中介者接收到通知后，会根据这个改变去协调其他相关的同事对象，比如启用/禁用某些按钮、更新文本域内容等。

## ConcreteMediator (具体中介者类 `LoginFrame.java`)

*   **作用**:
    *   实现 `Mediator` 接口。
    *   它需要知道并维护它所管理的各个 `Colleague` 对象。
    *   它负责具体的协调逻辑，即当某个 `Colleague` 通知它状态改变时，它如何影响其他的 `Colleague`。
*   **代码分析 (`LoginFrame.java`)**:
    *   `LoginFrame` 类在本例中扮演了双重角色：
        1.  它是一个 `java.awt.Frame`，是GUI窗口的容器，负责界面的布局和显示。
        2.  它实现了 `Mediator` 接口，充当了所有GUI组件（同事对象）的中介者。
    *   **成员变量**:
        ```java
        private CollegueCheckBox checkGuest; // “Guest”复选框
        private CollegueCheckBox checkLogin; // “Login”复选框
        private CollegueTextField textUser;  // 用户名输入框
        private CollegueTextField textPass;  // 密码输入框
        private CollegueButton buttonOk;     // “OK”按钮
        private CollegueButton buttonCancel; // “Cancel”按钮
        ```
        这些都是 `Colleague` 类型的对象，`LoginFrame`（作为中介者）持有对它们的引用。
    *   **构造函数 `public LoginFrame(String title)`**:
        *   设置窗口的基本属性（背景色、布局管理器）。
        *   调用 `createCollegues();` 来创建和初始化所有的GUI组件。
        *   将这些组件添加到 `Frame` 中。
        *   调用 `collegueChanged();` 来根据初始状态设置各组件的可用性。
        *   `pack()` 和 `show()` 用于调整窗口大小并显示窗口。
    *   **`@Override public void createCollegues()`**:
        *   这个方法实现了 `Mediator` 接口的 `createCollegues`。
        *   它负责实例化所有的 `CollegueCheckBox`, `CollegueTextField`, `CollegueButton` 对象。
        *   **关键**: 对于每个创建的 `Colleague` 对象，都调用了其 `setMediator(this)` 方法，将 `LoginFrame` 自身（作为 `Mediator`）注册给了这个 `Colleague`。这样，每个 `Colleague` 就都知道了它的中介者是谁。
        *   它还为每个组件添加了相应的事件监听器。例如，`checkGuest.addItemListener(checkGuest)` 表示 `checkGuest` 自己监听自己的 `ItemEvent` 事件。当事件发生时（如复选框被点击），会调用 `checkGuest` 的 `itemStateChanged` 方法，该方法内部会调用 `mediator.collegueChanged()`。
    *   **`@Override public void collegueChanged()`**:
        *   这是中介者模式的核心协调逻辑。当任何一个 `Colleague` 对象的状态发生改变并通知中介者时，这个方法被调用。
        *   **逻辑**:
            1.  `if (checkGuest.getState())`: 如果 "Guest" 复选框被选中：
                *   `textUser.setCollegueEnabled(false);`: 禁用用户名输入框。
                *   `textPass.setCollegueEnabled(false);`: 禁用密码输入框。
                *   `buttonOk.setCollegueEnabled(true);`: 启用 "OK" 按钮（访客模式下通常可以直接确认）。
            2.  `else`: 如果 "Guest" 复选框未被选中（意味着 "Login" 复选框被选中，因为它们在同一个 `CheckboxGroup` 中）：
                *   `textUser.setCollegueEnabled(true);`: 启用用户名输入框。
                *   `userpassChanged();`: 调用另一个私有方法 `userpassChanged()` 来进一步根据用户名和密码输入框的状态调整其他组件。
    *   **`private void userpassChanged()`**:
        *   这是一个辅助方法，在 "Login" 模式下被 `collegueChanged()` 调用，或者当用户名/密码文本框内容改变时也可能被间接调用（通过 `textUser` 或 `textPass` 的事件触发 `mediator.colleagueChanged()`，然后进入 `else` 分支再调用此方法）。
        *   **逻辑**:
            1.  `if (textUser.getText().length() > 0)`: 如果用户名输入框中有内容：
                *   `textPass.setCollegueEnabled(true);`: 启用密码输入框。
                *   `if (textPass.getText().length() > 0)`: 如果密码输入框中也有内容：
                    *   `buttonOk.setCollegueEnabled(true);`: 启用 "OK" 按钮。
                *   `else`: 如果密码输入框为空：
                    *   `buttonOk.setCollegueEnabled(false);`: 禁用 "OK" 按钮。
            2.  `else`: 如果用户名输入框为空：
                *   `textPass.setCollegueEnabled(false);`: 禁用密码输入框。
                *   `buttonOk.setCollegueEnabled(false);`: 禁用 "OK" 按钮。（原代码中这里是 `textUser.setCollegueEnabled(false);`，这似乎是一个小笔误，因为如果用户名为空，通常是密码框和OK按钮受影响，用户名框本身应该保持可用让用户输入。但按原代码逻辑分析，它会再次禁用用户名框，效果上OK按钮也会因用户名框为空而被禁用）。
    *   **`actionPerformed(ActionEvent e)`**:
        *   当 `buttonOk` 或 `buttonCancel` 被点击时调用。
        *   示例代码中只是简单地打印事件信息并退出程序 (`System.exit(0)`)。在实际应用中，这里会处理登录逻辑或取消操作。

## Colleague (同事接口 `Collegue.java`)

*   **作用**: 定义了所有同事对象的通用接口。每个同事对象都需要知道它的中介者（`Mediator`），并且提供一个方法让中介者可以控制它的状态（如启用/禁用）。
*   **代码分析 (`Collegue.java`)**:
    ```java
    public interface Collegue {
        // 设置该同事对象所关联的中介者
        public abstract void setMediator(Mediator mediator);

        // 允许中介者控制该同事对象的启用/禁用状态
        public abstract void setCollegueEnabled(boolean enabled);
    }
    ```
    *   `public abstract void setMediator(Mediator mediator);`:
        *   用于将一个中介者对象注册给当前同事对象。同事对象内部通常会保存这个 `mediator` 的引用。
    *   `public abstract void setCollegueEnabled(boolean enabled);`:
        *   这个方法允许中介者根据协调逻辑来改变当前同事对象的状态，例如将其设置为可用或不可用。

## ConcreteColleague (具体同事类)

这些类实现了 `Collegue` 接口，并且它们通常是GUI中的具体组件（如按钮、文本框、复选框）。

*   **`CollegueButton.java`**:
    *   继承自 `java.awt.Button` 并实现 `Collegue` 接口。
    *   `private Mediator mediator;`: 保存中介者的引用。
    *   `public void setMediator(Mediator mediator)`: 实现接口方法，设置中介者。
    *   `public void setCollegueEnabled(boolean enabled)`: 实现接口方法，调用AWT组件的 `setEnabled(enabled)` 方法来实际启用或禁用按钮。
    *   **注意**: `CollegueButton` 自身不直接监听事件来通知中介者。它的事件（`ActionEvent`）是在 `LoginFrame` 中被监听和处理的。当 `LoginFrame` 的 `actionPerformed` 被调用时，它知道是哪个按钮触发了事件。

*   **`CollegueCheckBox.java`**:
    *   继承自 `java.awt.Checkbox`，实现 `Collegue` 接口，并且还实现了 `java.awt.event.ItemListener` 接口来监听自身的选中状态变化。
    *   `private Mediator mediator;`: 保存中介者的引用。
    *   `public void setMediator(Mediator mediator)`: 实现接口方法。
    *   `public void setCollegueEnabled(boolean enabled)`: 实现接口方法，调用 `setEnabled(enabled)`。
    *   `@Override public void itemStateChanged(ItemEvent e)`:
        *   **核心**: 当复选框的选中状态发生改变时，这个方法会被AWT框架调用。
        *   `mediator.collegueChanged();`: 它立即通知中介者 (`Mediator`) 它的状态发生了变化。中介者的 `collegueChanged()` 方法随后会执行协调逻辑。

*   **`CollegueTextField.java`**:
    *   继承自 `java.awt.TextField`，实现 `Collegue` 接口，并且还实现了 `java.awt.event.TextListener` 接口来监听自身文本内容的变化。
    *   `private Mediator mediator;`: 保存中介者的引用。
    *   `public void setMediator(Mediator mediator)`: 实现接口方法。
    *   `public void setCollegueEnabled(boolean enabled)`:
        *   实现接口方法，调用 `setEnabled(enabled)`。
        *   额外地，它还会根据 `enabled` 状态改变文本框的背景色（可用时白色，不可用时浅灰色），以提供视觉反馈。
    *   `@Override public void textValueChanged(TextEvent e)`:
        *   **核心**: 当文本框中的文本内容发生改变时，这个方法会被AWT框架调用。
        *   `mediator.collegueChanged();`: 它立即通知中介者它的状态（文本内容）发生了变化。

## Client (客户端 `Main.java`)

*   **作用**: 客户端代码负责创建 `ConcreteMediator` 对象。通常，`ConcreteMediator` 在其内部会负责创建和配置所有的 `Colleague` 对象。
*   **代码分析 (`Main.java`)**:
    ```java
    public class Main {
        public static void main(String[] args) {
            new LoginFrame("Mediator Sample"); // 创建并显示 LoginFrame
        }
    }
    ```
    *   客户端 `Main` 的职责非常简单：它只创建了一个 `LoginFrame`（具体中介者）的实例。
    *   `LoginFrame` 的构造函数负责了所有同事对象的创建、注册中介者给同事、以及初始状态的设置。
    *   一旦 `LoginFrame` 被创建并显示出来，用户与GUI组件的交互就会通过 `Colleague` -> `Mediator` -> 其他 `Colleague` 的方式进行协调。

通过这种方式，`CollegueCheckBox`, `CollegueTextField`, `CollegueButton` 这些GUI组件之间不需要直接相互引用或调用。例如，当用户点击 `checkGuest` 复选框时，`checkGuest` 只通知 `LoginFrame`（中介者）它的状态变了。然后 `LoginFrame` 根据这个变化，决定去启用或禁用 `textUser`, `textPass`, `buttonOk` 这些其他的组件。这种交互逻辑被集中在了 `LoginFrame` 的 `collegueChanged()` 和 `userpassChanged()` 方法中。

# 中介者模式的优点

1.  **降低了类间的耦合（松耦合）**:
    *   中介者模式将原来多对多（网状）的复杂交互关系转变为一对多（星形）的交互关系（各个 `Colleague` 都只与 `Mediator` 通信）。
    *   `Colleague` 对象之间不再直接相互依赖，它们只依赖于 `Mediator`。这大大减少了类之间的依赖数量。

2.  **集中控制交互**:
    *   对象之间的交互行为被封装在中介者对象 (`ConcreteMediator`) 内部。这使得交互逻辑更加集中，易于理解、维护和修改。
    *   如果需要改变对象间的交互方式，通常只需要修改中介者类的代码，而不需要修改各个 `Colleague` 类。

3.  **符合迪米特法则（最少知识原则）**:
    *   每个 `Colleague` 对象只需要知道 `Mediator`，而不需要知道其他大量的 `Colleague` 对象。它只需要和直接的朋友（`Mediator`）通信。

4.  **提高了组件的可复用性**:
    *   由于 `Colleague` 对象只与 `Mediator` 交互，不依赖于其他 `Colleague`，因此它们更容易被复用到其他不同的中介者环境中。

# 中介者模式的缺点

1.  **中介者职责过重，可能变成“上帝类” (God Object)**:
    *   如果系统中的 `Colleague` 对象非常多，或者它们之间的交互逻辑非常复杂，那么所有的这些交互逻辑都会集中到 `Mediator` 类中。
    *   这可能导致 `Mediator` 类变得异常庞大和复杂，承担了过多的职责，从而难以理解和维护。它本身可能成为一个新的瓶颈或复杂点。
    *   需要仔细设计中介者的职责范围，避免其过于臃肿。

# 适用场景

中介者模式主要适用于以下情况：

1.  **当一组对象以定义良好但复杂的方式进行通信，导致了过多的相互依赖和直接通信。**
    *   如果系统中对象之间存在复杂的网状依赖关系，使得系统难以修改和维护，中介者模式可以帮助理清这些关系。
2.  **想自定义一个分布在多个类中的行为，而又不想生成太多的子类时。**
    *   例如，一个GUI界面中，多个控件（按钮、文本框、复选框等）之间的行为是相互关联和影响的。如果不用中介者，可能需要在每个控件中编写大量的逻辑来处理与其他控件的交互。
3.  **常用于GUI中各组件之间的协调。**
    *   这是中介者模式的一个非常经典的应用场景。例如，对话框中的各个控件（`Colleague`）的行为通常由对话框本身（`Mediator`）来协调。本例中的 `LoginFrame` 就是一个很好的体现。
4.  **当你想为一个已存在的类创建一个可复用的组件，但又不能修改那个类，可以通过创建一个中介者来与那个类进行交互。**

# 给Java初学者的提示

*   **核心在于“解除Colleague之间的直接依赖”，所有通信都通过Mediator。**
    *   想象一下，如果没有 `LoginFrame` 作为中介者，那么 `checkGuest` 复选框可能需要直接持有 `textUser`, `textPass`, `buttonOk` 的引用，并在其状态改变时逐个去设置它们的 `enabled` 状态。同样，`textUser` 文本框也可能需要知道 `textPass` 和 `buttonOk`。这将导致组件间代码的混乱和高度耦合。
    *   中介者模式将这些依赖关系都转移到了中介者身上。

*   **Mediator了解所有的Colleague，但Colleague只了解Mediator。**
    *   `LoginFrame` (Mediator) 拥有对所有 `CollegueCheckBox`, `CollegueTextField`, `CollegueButton` 实例的引用。
    *   而每个 `Collegue` 对象（如 `CollegueCheckBox`）只拥有一个对 `Mediator` 接口的引用。它不知道其他具体 `Colleague` 的存在。

*   **当一个Colleague发生变化时，它通知Mediator，Mediator再根据预设逻辑通知其他相关的Colleague。**
    *   例如，`CollegueCheckBox.itemStateChanged()` 方法中调用 `mediator.collegueChanged()`。
    *   然后 `LoginFrame.collegueChanged()` 方法会根据是哪个 `Colleague` 变化了（或者根据所有 `Colleague` 的当前状态组合）来决定如何设置其他 `Colleague` 的 `enabled` 状态（通过调用 `collegue.setCollegueEnabled(...)`）。

*   **`LoginFrame`既是Mediator又是GUI的容器，这在GUI应用中很常见。**
    *   在很多GUI框架中，窗口或对话框（如 `JFrame`, `JDialog`, `Frame`）本身就经常扮演其内部组件的中介者角色。这是因为窗口天然地拥有对其所有子组件的引用，并且负责响应用户与这些组件的交互。
    *   这种将UI容器和中介者角色合二为一的方式可以简化设计，但也可能使得容器类职责过重，需要权衡。

中介者模式通过引入一个中心协调点，有效地简化了对象间的交互，降低了系统的耦合度，使得系统更加灵活和易于维护。
