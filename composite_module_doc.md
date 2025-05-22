# 模块名称

组合模式 (Composite Pattern)

# 核心概念 (针对Java初学者)

**什么是组合模式？**

组合模式（Composite Pattern）是一种结构型设计模式。它的核心思想是：**将对象组合成树形结构以表示“部分-整体”的层次结构。组合模式使得用户对单个对象（叶子节点）和组合对象（容器节点）的使用具有一致性。**

这意味着，无论你操作的是一个简单的独立对象，还是一个包含了很多其他对象的复杂对象集合，你都可以用同样的方式来对待它们。

**类比：公司组织架构**

一个非常形象的例子就是公司的组织架构：

*   **公司 (顶层容器 - Composite)**: 整个公司是一个大的实体。
*   **部门 (中间容器 - Composite)**: 公司内部有多个部门，比如“研发部”、“市场部”、“人力资源部”。每个部门也是一个实体，它可以包含员工，也可以包含更小的子部门或团队。
*   **员工 (叶子节点 - Leaf)**: 部门内部有具体的员工。员工是最小的单元，他们不包含其他员工或部门。

现在，假设我们想执行一些操作，比如：
*   获取名称（公司名、部门名、员工名）。
*   计算某个实体的总薪资（对于员工，是其个人薪资；对于部门，是部门内所有员工和子部门的总薪资；对于公司，是所有部门的总薪资）。
*   打印组织结构列表。

使用组合模式，我们可以定义一个统一的接口（比如叫做“组织单元”），让公司、部门和员工都实现这个接口。这样，当你想获取一个“组织单元”的名称或计算其总薪资时，你不需要关心它具体是一个公司、一个部门，还是一个员工，你都可以用同样的方法去调用。对于“计算总薪资”这样的操作，如果是员工，就直接返回自己的薪资；如果是部门或公司，它就会遍历其内部的所有子单元（其他部门或员工），并汇总它们的结果。

组合模式通过这种方式，简化了客户端代码对复杂树形结构的操作。

# 模式结构与角色分析

组合模式主要包含以下角色：

## Component (组件抽象类 `Entry.java`)

*   **作用**:
    *   为组合中的所有对象（包括叶子节点和容器节点）声明一个统一的接口。
    *   可以实现该接口的一些缺省行为。
    *   可以选择性地声明一个用于访问和管理其子组件的接口（如 `add`, `remove` 等）。如果声明了这些管理子组件的接口，那么叶子节点通常会提供一个默认的空实现或抛出异常，因为它们没有子组件。
*   **代码分析 (`Entry.java`)**:
    *   这是一个抽象类，是 `File`（叶子）和 `Directory`（容器）的共同父类。
    *   `public abstract String getName();`: 抽象方法，要求所有子类必须实现，用于获取条目（文件或目录）的名称。
    *   `public abstract int getSize();`: 抽象方法，要求所有子类必须实现，用于获取条目的大小。
    *   `public Entry add(Entry entry) throws FileTreatmentException`:
        *   这是管理子组件的方法之一。在 `Entry` 基类中，它默认抛出 `FileTreatmentException`。
        *   **设计思想**: 这种设计被称为“透明组合模式”的一种方式。它将管理子节点的方法（如 `add`）放在了组件的基类中，这样客户端可以统一对待所有 `Entry` 对象。但是，对于不能有子节点的 `File` 对象来说，调用 `add` 是没有意义的，所以默认行为是抛出异常。只有 `Directory` 类会重写这个方法以提供具体的添加子条目的实现。
        *   `throw new FileTreatmentException();`
    *   `public void printList()`:
        *   这是一个公共的便利方法，用于启动打印列表的操作。它内部调用了受保护的抽象方法 `printList("")`，并传入一个空的前缀。
    *   `protected abstract void printList(String prefix);`:
        *   抽象方法，要求子类实现如何打印自己的信息，并处理其子条目（如果存在）的打印。`prefix` 参数用于在打印时表示层级结构（如缩进）。
    *   `public String toString()`:
        *   重写了 `Object` 的 `toString` 方法，返回条目的名称和大小，格式为 `"name (size)"`。例如，一个名为 "report.doc" 大小为 1024 的文件，其 `toString()` 会返回 `"report.doc (1024)"`。这个方法被 `printList` 的实现所使用。

## Leaf (叶子构件 `File.java`)

*   **作用**: 在组合中表示叶子节点对象。叶子节点没有子节点，它实现了在 `Component` 中定义的行为。
*   **代码分析 (`File.java`)**:
    *   此类继承自 `Entry`，代表文件系统中的一个文件。
    *   `private String name;`: 文件名。
    *   `private int size;`: 文件大小。
    *   `public File(String name, int size)`: 构造函数，初始化文件名和大小。
    *   `public String getName()`: 实现父类的抽象方法，返回文件名。
    *   `public int getSize()`: 实现父类的抽象方法，返回文件大小。
    *   `protected void printList(String prefix)`:
        *   实现了父类的抽象方法，用于打印文件信息。
        *   `System.out.println(prefix + "/" + this);`
        *   它使用传入的 `prefix` 并在后面追加文件名和大小（通过 `this` 隐式调用 `toString()` 方法）。例如，如果 `prefix` 是 "/root/bin"，文件名是 "vi"，则输出 "/root/bin/vi (100000)"。
    *   **关于 `add` 方法**: `File` 类**没有**重写 `Entry` 类中的 `add(Entry entry)` 方法。因此，如果尝试对一个 `File` 对象调用 `add` 方法，它会执行 `Entry` 类中的默认实现，即抛出 `FileTreatmentException`。这符合文件不能包含其他文件或目录的特性。

## Composite (容器构件 `Directory.java`)

*   **作用**:
    *   定义有子部件的那些部件（即容器节点）的行为。
    *   存储其子部件 (`Component` 对象)。
    *   在 `Component` 接口中实现与子部件有关的操作（如 `add`, `remove`），以及其他可能的操作（如 `getSize`，它通常会递归调用其子部件的相应方法）。
*   **代码分析 (`Directory.java`)**:
    *   此类继承自 `Entry`，代表文件系统中的一个目录。
    *   `private String name;`: 目录名。
    *   `private ArrayList directory = new ArrayList();`: **核心字段**，用于存储该目录下的所有子条目（`Entry` 对象，可以是 `File` 或其他 `Directory`）。使用了 `ArrayList` 来实现。
    *   `public Directory(String name)`: 构造函数，初始化目录名。
    *   `public String getName()`: 实现父类的抽象方法，返回目录名。
    *   `public int getSize()`:
        *   实现父类的抽象方法，计算并返回目录的大小。
        *   **递归计算**: 目录的大小是其包含的所有子条目（文件和子目录）大小的总和。
            ```java
            int size = 0;
            Iterator it = directory.iterator();
            while (it.hasNext()) {
                Entry entry = (Entry)it.next();
                size += entry.getSize(); // 递归调用子条目的 getSize() 方法
            }
            return size;
            ```
            这个方法遍历 `directory`列表中的每一个 `Entry`，并调用该 `entry` 的 `getSize()` 方法。如果 `entry` 是一个 `File`，则返回文件的大小；如果 `entry` 是一个 `Directory`，则会再次递归地计算该子目录的大小。
    *   `public Entry add(Entry entry)`:
        *   **重写**了父类 `Entry` 的 `add` 方法。
        *   `directory.add(entry);`: 将传入的 `Entry` 对象（文件或子目录）添加到内部的 `directory` 列表中。
        *   `return this;`: 返回当前 `Directory` 对象，这允许链式调用（虽然在本例的 `Main` 中没有显式使用链式 `add`，但这种返回类型很常见）。
        *   这个方法使得 `Directory` 对象可以包含其他 `Entry` 对象，从而构建树形结构。
    *   `protected void printList(String prefix)`:
        *   实现了父类的抽象方法，用于打印目录及其内容。
        *   `System.out.println(prefix + "/" + this);`: 首先打印当前目录自身的信息（名称和总大小）。
        *   **递归打印**: 然后遍历其所有子条目，并为每个子条目调用 `printList` 方法，同时更新前缀。
            ```java
            Iterator it = directory.iterator();
            while (it.hasNext()) {
                Entry entry = (Entry)it.next();
                entry.printList(prefix + "/" + name); // 为子条目生成新的前缀，并递归调用
            }
            ```
            例如，如果当前目录是 "root"，其子目录是 "bin"，则传递给 "bin" 的前缀将是 "/root/bin"。

## Custom Exception (`FileTreatmentException.java`)

*   **作用**: 这是一个自定义的运行时异常。当尝试对一个不支持特定操作（如对 `File` 对象调用 `add` 方法）的 `Entry` 对象执行该操作时，会抛出此异常。
*   **代码分析 (`FileTreatmentException.java`)**:
    ```java
    public class FileTreatmentException extends RuntimeException {
        public FileTreatmentException() {
        }
        public FileTreatmentException(String msg) {
            super(msg);
        }
    }
    ```
    它继承自 `RuntimeException`，意味着它是一个非检查型异常。`Entry` 类的 `add` 方法默认会抛出这个异常。

## Client (客户端 `Main.java`)

*   **作用**: 通过 `Component` 接口（即 `Entry` 类）来操纵组合部件的对象。客户端代码不需要（也不应该）区分它正在操作的是一个叶子节点（`File`）还是一个容器节点（`Directory`），它统一将它们视为 `Entry` 对象。
*   **代码分析 (`Main.java`)**:
    *   `public static void main(String[] args)`:
        *   **构建树形结构**:
            ```java
            Directory rootdir = new Directory("root");
            Directory bindir  = new Directory("bin");
            Directory tmpdir  = new Directory("tmp");
            Directory usrdir  = new Directory("usr");
            rootdir.add(bindir); // 向 root 添加 bin 目录
            rootdir.add(tmpdir); // 向 root 添加 tmp 目录
            rootdir.add(usrdir); // 向 root 添加 usr 目录

            bindir.add(new File("vi", 100000));   // 向 bin 添加 vi 文件
            bindir.add(new File("ls", 200000));   // 向 bin 添加 ls 文件
            // ... 类似地构建 usr 目录下的结构 ...
            Directory yuki   = new Directory("yuki");
            usrdir.add(yuki);
            yuki.add(new File("diary.html", 1000));
            ```
            客户端代码通过实例化 `Directory` 和 `File` 对象，并使用 `Directory` 的 `add` 方法，逐步构建出一个文件系统的树形层级结构。
        *   **统一操作**:
            ```java
            rootdir.printList();
            ```
            这里，客户端调用了顶层目录 `rootdir` 的 `printList()` 方法。由于组合模式的一致性，这个调用会递归地打印出整个 `rootdir` 及其所有子目录和文件的层级列表和大小信息，而客户端不需要关心 `rootdir` 内部的具体结构。
        *   **异常处理**:
            ```java
            try {
                // ... 构建和操作代码 ...
            } catch (FileTreatmentException e) {
                e.printStackTrace();
            }
            ```
            虽然在这个 `Main` 的示例中，不太可能直接触发 `FileTreatmentException`（因为 `add` 只在 `Directory` 上下文中有意义地调用，而没有尝试对 `File` 调用 `add`），但这种异常处理机制是为了应对可能的误用（例如，如果客户端代码不小心获取了一个 `File` 对象并尝试对其调用 `add`）。

通过这种方式，客户端可以透明地处理复杂的树形结构，而无需编写大量的条件判断代码来区分叶子节点和容器节点。

# 组合模式的优点

1.  **统一处理单个对象和组合对象**:
    *   这是组合模式最核心的优点。客户端代码可以一致地对待单个对象（叶子节点，如 `File`）和对象的组合（容器节点，如 `Directory`）。它们都实现了共同的 `Component` 接口（`Entry`），所以客户端可以用同样的方式调用它们的方法（如 `getName()`, `getSize()`, `printList()`）。
    *   这大大简化了客户端代码，使其不必区分正在处理的是单个元素还是一个集合。

2.  **易于增加新的 Component 类型**:
    *   如果需要添加一种新的叶子节点类型（例如 `SymbolicLinkFile`）或者一种新的容器节点类型（例如 `CompressedDirectory`），只需要让它们继承（或实现）`Component` (`Entry`) 即可。
    *   现有代码（尤其是客户端代码和已有的容器类）通常不需要大的改动，这符合开闭原则。

3.  **可以表示复杂的树形结构**:
    *   组合模式非常适合用来表示具有层级关系（部分-整体）的树形结构，如文件系统、GUI中的控件容器、公司的组织架构等。

4.  **简化了客户端代码**:
    *   由于操作的一致性，客户端不需要编写复杂的 `if-else` 语句来区分叶子节点和容器节点。

# 组合模式的缺点

1.  **“透明方式”下，使设计更加抽象，有时会限制组件的类型**:
    *   **透明方式**: 指的是在 `Component` 基类中声明所有管理子对象的方法（如 `add`, `remove`, `getChild`）。这样做的好处是客户端可以完全忽略 `Leaf` 和 `Composite` 的区别。
    *   **缺点**: `Leaf` 类也继承了这些管理子对象的方法，但它不能有子对象，所以这些方法的实现通常是抛出异常（如本例中的 `Entry.add()` 抛出 `FileTreatmentException`）或返回 `null`。这在编译时无法检查，可能会导致运行时错误。
    *   如果某些组件的类型非常特定，不适合拥有 `add/remove` 等方法，这种设计可能会让人困惑。

2.  **“安全方式”下，客户端代码需要区分 Leaf 和 Composite**:
    *   **安全方式**: 指的是只在 `Composite` 类中声明管理子对象的方法，而不在 `Component` 基类中声明。
    *   **优点**: `Leaf` 类就不会有那些它不支持的方法，类型上更安全。
    *   **缺点**: 客户端代码在需要管理子对象时，必须先判断当前 `Component` 对象是否是 `Composite` 类型（例如通过 `instanceof`），然后才能调用相应的管理方法。这牺牲了一部分透明性。
    *   本项目采用的是偏向“透明方式”的设计，因为 `Entry` 中定义了 `add` 方法。

3.  **如果树结构过于复杂，系统的理解和维护可能有一定难度。**
    *   大量的对象和层级关系可能会使得调试和追踪变得复杂。

# 适用场景

组合模式主要适用于以下情况：

1.  **想表示对象的部分-整体层次结构（树形结构）**。
    *   例如：文件系统中的文件和文件夹、GUI中的窗口和控件、公司的部门和员工、XML或HTML文档的节点结构。
2.  **希望用户忽略组合对象与单个对象的不同，用户将统一地使用组合结构中的所有对象。**
    *   客户端代码可以通过共同的接口与所有对象交互，无需关心它是叶子还是容器。
3.  **需求中体现了部分与整体的关系，并且这种关系可以有任意深度。**
    *   例如，文件夹可以包含文件和其他文件夹，这些子文件夹又可以包含更多的文件和文件夹。
4.  **当对象的某些操作对于叶子节点和容器节点有不同的实现，但希望客户端能够以相同的方式调用它们时。**
    *   例如，计算大小：文件直接返回自身大小，目录返回其所有子项大小的总和。但客户端都只调用 `getSize()`。

# 给Java初学者的提示

*   **核心在于定义一个统一的 Component 接口/抽象类**:
    *   这是组合模式的基石。在本例中，`Entry` 就是这个角色。它定义了所有“条目”（无论是文件还是目录）共有的行为，如 `getName()`, `getSize()`, `printList()`。

*   **Composite 类通常会包含一个子 Component 的集合**:
    *   `Directory` 类内部有一个 `ArrayList directory`，用于存储它包含的 `Entry` 对象（可以是 `File` 或其他 `Directory`）。
    *   `Composite` 的很多方法（如 `getSize()`, `printList()`）会遍历这个集合，并将操作委托（或递归调用）给它的子组件。

*   **理解 `Entry` 类中 `add` 方法默认抛出异常，而在 `Directory` 类中重写该方法的意义**:
    *   这是组合模式中处理“透明性”与“安全性”权衡的一种常见方式。
    *   在 `Entry` （组件基类）中声明 `add` 方法，使得客户端可以对任何 `Entry` 对象尝试调用 `add`（透明性）。
    *   对于 `File`（叶子节点），它不应该有 `add` 操作，所以它继承了 `Entry` 中默认抛出 `FileTreatmentException` 的行为。
    *   对于 `Directory`（容器节点），它需要 `add` 操作来添加子项，所以它重写了 `add` 方法以提供具体实现。
    *   这种设计让客户端在大多数情况下可以统一处理，但在不支持的操作上通过异常来反馈。

*   **递归是组合模式中常见操作的实现方式**:
    *   很多在容器节点上的操作，如计算总大小 (`Directory.getSize()`)、打印整个树形结构 (`Directory.printList()`)，都是通过递归地调用其子组件的相应方法来实现的。
    *   例如，`Directory.getSize()` 会调用其所有子 `Entry` 的 `getSize()` 方法，如果子 `Entry` 还是一个 `Directory`，则会进一步递归下去。

组合模式通过将叶子节点和容器节点视为同一种类型，极大地简化了对复杂树形结构的处理，使得代码更加清晰和易于维护。
