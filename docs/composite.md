# 模块名称: 组合模式 (Composite Pattern)

## 1. 核心概念 (针对Java初学者)

**什么是组合模式？**

组合模式（Composite Pattern）是一种结构型设计模式，它的核心思想是：**将对象组合成树形结构以表示“部分-整体”的层次结构。组合模式使得用户对单个对象（叶子节点）和组合对象（容器节点/复合节点）的使用具有一致性。**

这意味着，客户端代码可以以相同的方式处理一个单独的组件和一组组件的集合。你不需要在代码中写很多 `if-else` 来区分你正在处理的是一个简单元素还是一个复杂容器。

**生活中的类比：公司组织架构**

*   一个**公司 (Company - 顶层容器/Composite)** 是一个整体。
*   公司可能包含多个**部门 (Department - 容器/Composite)**，比如“研发部”、“市场部”、“人力资源部”。
*   每个部门又可能包含多个**员工 (Employee - 叶子节点/Leaf)**，也可能包含一些**子团队或小组 (SubTeam - 也是容器/Composite)**。

对于这个组织架构，我们可能想执行一些操作，比如：
*   获取名称（可以是公司名、部门名或员工名）。
*   计算总人数（对于员工是1，对于部门是部门内所有员工和子部门人数之和，对于公司则是所有部门人数之和）。
*   显示组织结构。

组合模式允许你定义一个通用的接口（例如 `OrganizationUnit`），让 `Company`, `Department`, `SubTeam`, 和 `Employee` 都实现这个接口。这样，你就可以用同样的方式调用 `unit.getName()` 或 `unit.getEmployeeCount()`，而不用关心 `unit` 到底是整个公司、一个部门还是一个具体的员工。

## 2. 模式结构与角色分析

本项目的 `Composite/` 目录下的文件用一个简化的文件系统（包含文件和文件夹）来演示组合模式。

*   **Component (组件抽象类 `Entry.java`)**:
    *   **作用**: 这是组合中所有对象的抽象基类（或接口）。它声明了叶子节点和容器节点共有的接口和行为。它可以包含添加、删除、获取子组件等方法的默认实现或抽象声明。
    *   **代码分析 (`Entry.java`)**:
        ```java
        public abstract class Entry {
            public abstract String getName(); // 获取名字
            public abstract int getSize();   // 获取大小

            // 添加条目，默认实现是抛出异常，因为叶子节点(File)不支持此操作
            public Entry add(Entry entry) throws FileTreatmentException {
                throw new FileTreatmentException();
            }

            public void printList() { printList(""); } // 公共的打印列表方法
            protected abstract void printList(String prefix); // 带前缀的打印列表，由子类实现

            @Override
            public String toString() { // 方便打印对象信息
                return getName() + " (" + getSize() + ")";
            }
        }
        ```
        `Entry` 定义了所有文件系统条目（文件或目录）的通用操作。注意 `add` 方法，它默认抛出 `FileTreatmentException`。这种设计方式被称为“透明组合模式”的一种实现，即在父类中声明管理子对象的方法，但叶子类不支持这些方法。

*   **Leaf (叶子构件 `File.java`)**:
    *   **作用**: 代表组合中的叶子对象，即那些没有子对象的对象。它实现了 `Component` 接口定义的所有行为。
    *   **代码分析 (`File.java`)**:
        ```java
        public class File extends Entry {
            private String name;
            private int size;

            public File(String name, int size) {
                this.name = name;
                this.size = size;
            }
            public String getName() { return name; }
            public int getSize() { return size; } // 文件大小是固定的

            protected void printList(String prefix) {
                System.out.println(prefix + "/" + this); // 打印自身信息
            }
            // File 类没有重写 add 方法，所以如果调用 file.add()，会执行 Entry 类中的默认实现，抛出异常
        }
        ```
        `File` 类代表一个文件，它有名字和大小。它不能包含其他文件或目录，所以它不重写 `add` 方法。

*   **Composite (容器构件 `Directory.java`)**:
    *   **作用**: 代表组合中的容器对象（即有子对象的节点）。它实现了 `Component` 接口定义的操作，这些操作通常会委托给它的子组件来完成。它还需要存储和管理其子组件。
    *   **代码分析 (`Directory.java`)**:
        ```java
        import java.util.Iterator;
        import java.util.ArrayList;

        public class Directory extends Entry {
            private String name;
            private ArrayList<Entry> directory = new ArrayList<>(); // 存储子条目 (文件或目录)

            public Directory(String name) { this.name = name; }
            public String getName() { return this.name; }

            public int getSize() { // 目录的大小是其所有子条目大小的总和
                int size = 0;
                Iterator<Entry> it = directory.iterator();
                while (it.hasNext()) {
                    Entry entry = it.next();
                    size += entry.getSize(); // 递归调用子条目的getSize()
                }
                return size;
            }

            @Override // 重写add方法以支持添加子条目
            public Entry add(Entry entry) {
                directory.add(entry);
                return this;
            }

            @Override
            protected void printList(String prefix) {
                System.out.println(prefix + "/" + this); // 打印自身信息
                Iterator<Entry> it = directory.iterator();
                while (it.hasNext()) {
                    Entry entry = it.next();
                    // 对每个子条目，递归调用其printList，并增加前缀深度
                    entry.printList(prefix + "/" + name);
                }
            }
        }
        ```
        `Directory` 类代表一个目录。它可以包含其他 `File` 或 `Directory`。
        *   它的 `getSize()` 方法通过累加其所有子条目的 `getSize()` 结果来计算总大小。
        *   它的 `add()` 方法被重写，以允许向目录中添加新的条目。
        *   它的 `printList()` 方法首先打印自身信息，然后递归地调用其所有子条目的 `printList()` 方法。

*   **Custom Exception (`FileTreatmentException.java`)**:
    *   **作用**: 这是一个自定义的运行时异常，当尝试对 `File` 对象执行 `add` 操作时，由 `Entry` 类的默认 `add` 方法抛出。
        ```java
        public class FileTreatmentException extends RuntimeException {
            public FileTreatmentException() { }
            public FileTreatmentException(String msg) { super(msg); }
        }
        ```

*   **Client (客户端 `Main.java`)**:
    *   **作用**: 通过 `Component` 接口（即 `Entry` 类）来操作组合部件中的对象，而无需关心它具体是叶子节点还是容器节点。
    *   **代码分析 (`Main.java`)**:
        ```java
        public class Main {
            public static void main(String[] args) {
                try {
                    System.out.println("Making root entries...");
                    Directory rootdir = new Directory("root");
                    Directory bindir  = new Directory("bin");
                    // ... (创建其他目录和文件) ...
                    rootdir.add(bindir);
                    // ... (添加其他条目到目录中) ...
                    bindir.add(new File("vi", 100000));
                    bindir.add(new File("ls", 200000));
                    // ...
                    rootdir.printList(); // 统一调用printList，无论是目录还是文件都能正确响应

                    // ... (创建更多用户目录和文件) ...
                    usrdir.add(yuki);
                    yuki.add(new File("diary.html", 1000));
                    // ...
                    rootdir.printList(); // 再次打印整个树形结构
                } catch (FileTreatmentException e) {
                    e.printStackTrace();
                }
            }
        }
        ```
        客户端代码创建了一个文件系统的树形结构，比如 `rootdir` 包含 `bindir`，`bindir` 包含 `File` 对象 `vi` 和 `ls`。然后，客户端调用 `rootdir.printList()`。这个调用会递归地遍历整个树结构，每个 `Directory` 和 `File` 对象都会以适当的格式打印出自己的信息。客户端与 `Entry` 接口交互，不需要区分 `File` 和 `Directory` 来调用 `printList` 或 `getSize`。

## 3. 组合模式的优点

*   **统一样式处理，简化客户端代码**: 客户端可以一致地对待单个对象（叶子）和组合对象（容器）。它不需要写 `if-else` 判断来区分是叶子还是容器，使得代码更简洁、更易于理解。
*   **易于增加新的Component种类**: 如果需要增加新的叶子类型（比如 `SymbolicLinkFile`）或新的容器类型（比如 `CompressedFolder`），只需要让它们继承自 `Entry` 并实现相应方法即可。这符合开闭原则。
*   **可以表示复杂的树形结构**: 非常适合用来表示具有层级关系的对象结构，如GUI的窗口和控件、文件系统、公司的组织结构等。

## 4. 组合模式的缺点

*   **“透明方式”的缺点**: 如果在 `Component` 接口中定义了管理子组件的方法（如 `add`, `remove`），那么 `Leaf` 类也会继承这些方法。但 `Leaf` 本身不能有子组件，所以它对这些方法的实现通常是抛出异常或空操作。这可能会让客户端在调用这些方法时感到困惑，因为编译时不会报错，但运行时可能会出错。本例通过在`Entry.add`中默认抛出`FileTreatmentException`来处理。
    *   另一种方式是“安全方式”，即只在 `Composite` 类中声明管理子组件的方法，但这样客户端就需要区分 `Leaf` 和 `Composite` 对象，失去了一部分透明性。
*   **设计可能变得更加抽象**: 如果系统中的对象类型很多，或者树的结构非常复杂，组合模式的设计可能会变得比较抽象，增加理解和维护的难度。
*   **类型检查的限制**: 有时可能需要在运行时检查组件的实际类型，以便执行特定于该类型的操作，这可能违背了透明性的初衷。

## 5. 适用场景

*   **当你想表示对象的部分-整体层次结构（树形结构）时。** 这是组合模式最典型的应用场景。
*   **当希望用户忽略组合对象与单个对象的不同，用户将统一地使用组合结构中的所有对象时。**
*   **需求中体现了部分与整体的关系，并且这种关系可以有任意的深度。** 例如，GUI中的容器可以包含其他容器或基本控件；多级菜单；公司的组织架构等。

## 6. 给Java初学者的提示

*   **核心在于一个统一的 `Component` 接口/抽象类**: 这个接口定义了所有子件（无论是叶子还是容器）共有的行为。
*   **`Composite` 类是关键**: 它持有对其子 `Component` 的引用集合，并且通常将其自身的操作（如 `getSize`, `printList`）委托或汇总其子组件的相应操作。
*   **递归是常用技巧**: 许多在 `Composite` 对象上的操作（如计算大小、打印树、查找等）都是通过递归遍历其子组件来实现的。
*   **理解“透明性”与“安全性”的权衡**:
    *   **透明性**: 在 `Component` 中声明所有管理子节点的方法（如 `add`, `remove`）。优点是客户端完全统一处理所有组件。缺点是 `Leaf` 节点需要提供这些方法的空实现或抛出异常。本例属于这种。
    *   **安全性**: 只在 `Composite` 中声明管理子节点的方法。优点是 `Leaf` 接口更干净，不会有不支持的操作。缺点是客户端需要区分 `Leaf` 和 `Composite` 才能调用管理子节点的方法。
*   **叶子节点和容器节点的区分**: 虽然客户端可以统一处理它们，但在实现时，叶子节点（如 `File`）通常执行具体工作，而容器节点（如 `Directory`）则主要负责管理子节点并将工作分派给它们。

组合模式通过将对象组织成树形结构，并提供统一的接口来访问这些对象，极大地简化了对复杂层次结构的处理。
