# property 属性

```
[default] [final] [required] [readonly] property <propertyType> <propertyName>
```

## 1. default 关键字

在 QML(Qt Modeling Language) 中，`default` 关键字用于将某个属性标记为该组件的**默认属性**。这是一个非常强大的语法糖，它允许开发者以更简洁、更直观的方式定义对象的层级结构。

### 1.1 核心概念
通常，当你向一个 QML 对象赋值时，你需要显式地指定属性名称（例如 `text: "Hello"`）。然而，如果一个属性被标记为 `default`，那么在对象定义的内部嵌套的任何未指定属性名的子对象，都会自动赋值给这个默认属性。

P.S: 如果多个未命名的子对象,自动赋值给这个默认属性的则是最后加载的一个.
P.S：一个组件只能有一个默认属性.

### 1.2 工作原理
假设你定义了一个自定义组件，并且声明了一个名为 `content` 的属性，同时将其标记为 `default`。

```qml
// MyContainer.qml
import QtQuick 2.0

Item {
    // 声明 'content' 属性并标记为 default
    default property alias content: innerItem.children

    Item {
        id: innerItem
    }
}
```

在使用这个组件时，你不需要写 `content: [ Rectangle {} ]`。你可以直接在组件内部书写子组件：

```qml
// 使用 MyContainer
MyContainer {
    // 这个 Rectangle 会自动变成 innerItem 的子对象，
    // 因为 content 是默认属性。
    Rectangle {
        color: "red"
        width: 100; height: 100
    }
}
```

### 1.3 常见应用场景
这种模式在 Qt Quick 的标准库中非常常见。例如，`Item` 或 `Rectangle` 的 `data` 属性通常也被视为默认属性机制的一部分（虽然 `Item` 的默认属性实际上是 `data`，它包括了 `resources` 和 `children`），这就是为什么你可以直接在 `Rectangle` 里面嵌套另一个 `Rectangle` 而不需要显式地写 `children: [ ... ]`。

### 1.4 为什么使用它？
1.  **代码整洁**：减少冗余代码，使 QML 结构描述更像 HTML 或 XML 的树状结构。
2.  **直观性**：让自定义控件（如布局容器、页签视图等）的使用方式看起来更像原生的 QML 容器。

### 1.5 注意事项
*   一个对象定义中只能有一个属性被标记为 `default`。
*   默认属性通常是列表类型的属性（如 `list<Item>`）或者是对象引用，以便它可以接收嵌套的 QML 元素。

### 2. final 关键字
`final` 关键字用于防止属性在子类中被重写。当一个属性被声明为 `final` 时，任何试图在继承该组件的子类中重新定义该属性的行为都会导致编译错误。这对于确保某些关键属性的行为保持一致性非常有用。

final 关键字禁止对属性进行任何遮蔽。它在意义上等价于 Q_PROPERTY 的 FINAL 属性。通过声明你的属性为最终， 你可以帮助 Qt 快速编译器分析你的 QML 代码。这样就能生成更好的代码。

### 3. required 关键字

- `required` 关键字用于强制要求在创建组件实例时必须为该属性赋值。如果一个属性被标记为 `required`，那么在实例化该组件时，如果没有为该属性提供值，QML 引擎将抛出运行时错误。这对于确保组件在使用时具有必要的配置非常有用。

- 在模型-视图-委托（MVD）代码中，必需属性扮演着特殊角色：如果视图的委托具有必需属性，且这些属性的名称与视图模型的角色名称相匹配，则这些属性将被初始化为模型的对应值。

    | 特性 | 普通 Property | Required Property |
    | --- | --- | --- |
    | 初始值 | 可选，通常带默认值 | 禁止设置默认值 |
    | 外部赋值 | 可选 | 必须赋值 |
    | 缺失后果 | 使用默认值，可能导致逻辑空缺 | 抛出错误/警告，组件可能无法正常显示 |
    | Delegate 表现 | 需通过 model.xxx 访问 | 直接映射 Role，性能更优 |

### 3.1 场景：解决作用域歧义
使用 `required` 属性可以极大地解决**作用域歧义**。

在传统的 QML 中，如果你在 `Delegate` 内部又嵌套了一个 `ListView`（嵌套模型），直接访问 `name` 可能会让引擎搞混它是属于父级 `Model` 还是子级 `Model`。
通过 `required` 声明，每个 `Delegate` 明确地“认领”了属于自己的数据。

#### 3.1.1 传统方式：容易混淆的嵌套模型
假设你有一个“省份”列表，每个省份下面又有“城市”列表。如果两个 `Model` 都有一个角色叫 `name`，代码就会变得很危险：

```qml
ListView {
    model: provinceModel // 父模型，角色有：name, cities
    delegate: Column {
        Text { text: "省份: " + name } // 这里访问的是省份的 name

        ListView {
            model: cities // 子模型，角色有：name
            delegate: Text {
                // WARN: 这里的 name 到底是谁的？
                // 虽然通常指向子模型，但在某些加载瞬间或逻辑错误时，
                // 它可能会错误地回退（Fallback）并显示父模型的“省份名”。
                text: "  城市: " + name 
            }
        }
    }
}
```

#### 3.1.2 使用 required 方式：显式且安全的映射
通过 `required` 关键字，我们可以将每一层的数据强行“锁定”在当前的作用域内，避免隐式查找带来的错误。

```qml
ListView {
    model: provinceModel
    delegate: Column {
        id: provinceDelegate
        
        // 显式接收父级 Model 的角色
        required property string name
        required property var cities 

        Text { 
            text: "省份: " + provinceDelegate.name // 明确指向父级
            font.bold: true 
        }

        ListView {
            width: 200; height: 100
            model: provinceDelegate.cities
            delegate: Item {
                height: 20
                // 显式接收子级 Model 的角色
                required property string name 

                Text { 
                    // 这里的 name 绝对不会跳到省份去，因为 required 建立了精确的绑定
                    text: "  城市: " + name 
                }
            }
        }
    }
}
```

### 4. readonly 关键字

`readonly` 关键字用于声明一个只读属性。这意味着该属性的值只能在组件内部设置，外部代码无法修改它。只读属性通常用于表示组件的状态或计算结果，这些值不应被外部直接更改。

只读属性在初始化时必须被赋予一个静态值或绑定表达式。只读属性初始化后，就无法再更改其静态值或绑定表达式。

例如，下面 Component.onCompleted 代码块中的代码无效：
```qml
Item {
    readonly property int someNumber: 10
    Component.onCompleted: someNumber = 20  // TypeError: Cannot assign to read-only property
}
```

**P.S: 只读属性不能同时也是默认属性。**