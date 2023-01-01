类图是 UML 中最重要的一种,类图是类与类之间的各种关系:
1. 依赖(dependency),只要在类中用到了对方,他们之间就存在依赖关系,使用虚线箭头表示
2. 泛化(generalization),即继承关系
3. 实现(implementation),一个类实现另一个类的方法,用空心三角形的虚线表示
4. 关联(association)
5. 聚合(aggregation),表示的是整体和部分可以封开(例如 Computer 中的 Mouse/Monitor),用空心棱形的实线来表示
6. 组合(composition),如果整体和部分是不可分开的(成员变量直接实例化 `A 类中 B b = new B();` ),则从聚合关系升级为组合关系.使用实心棱形的实现表示

