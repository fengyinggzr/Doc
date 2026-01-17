# QML 对象属性

```mermaid
flowchart LR 
    A[qml 对象属性] 
    A --> B[id] 
    
    A --> C[property/属性]
    A --> D[signal/信号/]
    A --> E[signal handler/信号处理/槽]
    A --> F[method/方法]
    A --> G[attached properties/附加属性]
    A --> H[attached signal handler attributes/附加信号处理/附加槽]
    A --> I[enumeration/枚举]

    B --> B1["id 属性示例 ↓"]

    C --> C1["property/属性 示例： ↓"]

    style A fill:#f9f,stroke:#333,stroke-width:2px,color:#000
    
```

## 1. id 属性示例：
```
import QtQuick

Item {
    id: myItem
    width: 200; height: 200
}
```

## 2. property/属性 示例：

```
[default] [final] [required] [readonly] property <propertyType> <propertyName>
```
