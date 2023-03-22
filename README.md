
# 需求分析

一个作业从用户提交开始到占有处理机被执行，一般来要由系统三级调度
才能实现，即作业调度、内存调度、进程调度。其中最为重要的：
1. 主要是完成作业从后备状态到可执行状态的转变，即按照某原则，从外存后备队列中挑选作业，将其装入内存并为其创建进程；
2. 主要是完成进程从就绪状态到执行（完成）状态的转变，即按照某策略，从内存就绪队列中选取进程，为其分配处理机使其运行。

# 基本假设
基于设计内容要求及上述理论常识，可以做出如下基本假设：
1. 作业调度和进程调度是互补共存的层级关系。不考虑内存调度的前提下，实现任何算法，至少要实现两级调度：作业调度及进程调度。
2. 作业和进程可以合二为一，统一抽象为任务。不考虑阻塞的前提下，一个任务大致可归为未提交态、收容态、就绪态、运行态及销毁态。
3. 调度是种将何资源分配给何任务的决策行为。从各级调度的交互上说，调度可以分为：本级进行调度、移交上层调度及委托下层调度。

# 概要设计
要模拟批处理系统，就须要模拟时钟、任务及其状态、分级调度。

1. 标定时间流逝的基准，以便执行作业调度、进程调度、性能分析等。每流逝一个单位时间，系统至少要进行一次调度。所以，可以建立“时钟”与“调度器”之间的依赖关系，当时钟发生改变时应该自动通知调度器，调度器接受信息并做出响应。时钟，被称为观察目标；调度器，被称为观察者。一个观察目标可以对应对个观察者，可以根据需要增加和删除观察者。这种设计模式被称为观察者模式或发布-订阅模式。假设，时钟的最小颗粒单位为分钟，所有作业均在23:59 前提交并结束。《任务书》设计描述中，单位并不统一，如，“进程到达时间（单位时间）”、“作业提交时间（时钟时刻）”、“预计运行时间（小时）”。所以需要在正式输入核心部分前，需要进行数据预处理和单位标准化。
2. 每个任务都有提交时间、到达时间等属性；运行中任务的剩余时间随时间流逝而减少（从这种意义说，运行中的进程应当观察时间流逝，并自主改变剩余时间字段）。任务揉杂作业和进程，集成两者的属性，即id、优先级、状态、时钟，提交时间、预计需要时间，到达时间、开始时间、剩余时间、完成时间。在模拟逻辑上，状态包含五种类型：未提交、收容、就绪、运行、结束。未提交，代表用户输入作业序列中的提交时间晚于当前时钟时刻；收容，代表作业已提交，进入外存收容队列（外存一般没有容量限制）；就绪，代表进程已创建，进入内存就绪队列（内存一般有容量限制）；运行，代表进程获得处理机资源，随时间流逝剩余时间递减。
3. 任务状态由调度器进行切换；调度是分层级的。作业提交与调度细节应当是解耦的，同时为较好地实现“分级”调度，可以将调度器链式组织，各司其职，又保证了作业在各级调度中的传递，直到进程销毁并记录日志。这种思想来源于职责链模式。
4. 机器整合时钟、任务、调度，并接受测试样例或用户输入，输出日志。不同的调度策略在各层级上排列组合，可以衍生出多种具体“机器”。为将对象创建和对象使用解耦，可以使用工厂方法模式。


