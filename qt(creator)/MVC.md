- Model 处理数据逻辑和程序运行状态
- View 则只负责显示
- Controller/Delegate 通常负责处理用户交互的部分，从视图读取数据与用户输入，并向模型发送数据；这里顺便提一下，在 Qt 里面我们并没有 Controller 的概念，而是 Delegate（委托），
- 意义很明显：控制器委托模型来处理数据，模型委托控制器来做数据的交互。

- https://blog.51cto.com/quantfabric/1879117
