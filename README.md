# Navigation Interface
study https://developer.apple.com/documentation/uikit/uinavigationcontroller 

# UINavigationController 概述
`UINavigationController` 是 `UIKit` 中两大容器视图（另 `UITabBarController`）之一，基于堆栈数据结构，管理一组视图控制器，提供不同视图间的导航能力。系统应用“Settings”就是典型的 `UINavigationController` 应用。


## UINavigationController 的组成
`UINavigationController` 继承自 `UIViewController`，内部通过 `viewControllers` 属性持有一组视图控制器。
通过 `navigationBar` 属性持有导航栏组件，其本身也是该导航栏组件的代理对象。
通过 `toolbar` 属性持有一个可选的工具栏组件。
通过 `delegate` 属性持有其代理对象。
其关系如下图所示：
![source: https://docs-assets.developer.apple.com/published/83ef757907/nav_controllers_objects_a8447aef-d652-4ab9-85d1-1eb8e4876e12.jp](https://docs-assets.developer.apple.com/published/83ef757907/nav_controllers_objects_a8447aef-d652-4ab9-85d1-1eb8e4876e12.jpg)

导航控制器需要至少有一个视图控制器作为其根视图控制器，才能正常显示。要获取导航栏当前显示的视图控制器，有如下两个属性：
1. `topViewController`：`viewControllers` 栈顶视图控制器，因为存在模态视图，该视图控制器不一定是当前正在显示的视图控制器；
2. `visibleViewController`：该视图控制器，可能是 `viewControllers` 栈顶视图控制器，也可能是当前通过模态视图显示的视图控制器。

视图导航，有两个基本操作：
1. push：推入一个新的视图控制器，通过 `pushViewController(_:animated:)` 实现；
注意：待推入的视图控制器，不能是 `UITabBarController` 实例，也不能已经存在于视图堆栈中，否则将抛出异常。
2. pop：弹出栈顶视图控制器或者弹出到指定视图控制器，通过 `popViewController(animated:)`、`popToRootViewController(animated:)`、`popToViewController(_:animated:)` 三个方法实现。


## 协作原理

`UINavigationController` 通过 push 和 pop 两类操作，管理 `viewControllers` 属性中的一组视图控制器。而每个视图控制器，都有一个 `navigationItem` 属性，因此构成了一个 `navigationItems` 数组，`UINavigationBar` 对象，正是类似于 `UINavigationController`，通过 push 和 pop 两类操作，管理着 `items` 属性中的一组对象，导航栏显示内容的数据源，正是这些对象。


## 导航控制器中的 Bar

导航控制器可以通过以下两个方法控制导航栏或工具栏的显示和隐藏：
`func setNavigationBarHidden(_:animated:)`
`func setToolbarHidden(_:animated:)`s

已下属性也可以实现导航栏和者工具栏的隐藏：
`hidesBarsOnTapp`
`hidesBarsOnSwipe`
`hidesBarsWhenVerticallyCompact`
`hidesBarsWhenKeyboardAppears`

由于并非导航控制器管理的所有视图控制器，对导航栏和工具栏都有统一的显示和隐藏需求，因此我们往往需要在导航控制器的代理对象中，通过实现其 `navigationController(_:willShow:animated:)` 代理方法，来实现更加细粒度的控制。

导航栏之上有 4 个标准的控件放置区域，分别是：
1. 顶部居中的 prompt label
2. 左侧按钮区
3. 中间 title view
4. 右侧按钮区

以上 4 个区域有不同的显示规则，具体如下：
1. 顶部居中的 prompt label，只能单行显示，数据来自于视图控制器的 `navigationItem.prompt` 属性。
2. 左侧按钮，除了根视图之外，其余视图控制器导航栏左侧按钮将会导航到上一个视图控制器，其内容由以下规则决定：
    1. 如果顶级视图有自定义的左侧按钮，则显示；
    2. 如果顶级视图没有自定义的左侧按钮，但上一个视图控制器设置了 `backBarButtonItem` 属性，则显示；
    3. 如果任何视图控制器都没有制定左侧按钮，则使用默认后退按钮，使用上一个视图控制器的 `title` 属性作为文本。
3. 中间 title view：
    1. 如果视图控制器提供了 `titleView` 属性，则显示；
    2. 如果视图控制器没有提供 `titleView` 属性，则显示视图控制器的 `navigationItem.title` ?? `title` 属性值。
4. 右侧按钮：
    1. 如果顶级视图提供了 `rightBarButtonItem` 属性实现，则显示；
    2. 如果顶级视图没有提供 `rightBarButtonItem` 属性实现，则什么都不显示。


## 方向支持

导航控制器在决定其所支持的设备方向时，并不会询问其管理的视图控制器，默认情况下，iOS 支持除 portrait upside-down（即设备倒置） 之外的所有方向，iPad 支持所有方向。我们也可以通过实现其代理对象中的 `optional func navigationControllerSupportedInterfaceOrientations(_:)` 方法，来返回特定的方向集合。


* * *

# UINavigationControllerDelegate

导航控制器代理对象，可以在推入视图控制器时，收到通知，我们可以在此针对不同视图控制器做细粒度的配置管理。更常用的，我们可以通过实现相关代理方法，实现自定义的专场动画。

> TODO: 待补充


* * *


# UINavigationBar

`UINavigationBar` 继承自 `UIView`，通常用于导航控制器中，但也可以单独使用，实现自定义的视图导航功能。该组件主要包括顶部居中的 label 区域，左侧按钮区域，中间的 title 区域，以及可选的右侧按钮区域，其构成如图所示：
![https://docs-assets.developer.apple.com/published/dde7452123/3abba22e-4aef-47dd-b4e2-a9965c424338.png](https://docs-assets.developer.apple.com/published/dde7452123/3abba22e-4aef-47dd-b4e2-a9965c424338.png)

如果在导航控制器中使用导航栏，以下步骤是必须的：
1. 创建导航控制器；
2. 配置导航栏的显示效果；
3. 通过实现 `UIViewController` 的 `title`、`navigationItem` 等属性，提供导航栏显示所需要的内容。

如果单独使用导航栏，以下步骤是必须的：
1. 为导航栏设置布局约束，使其正常显示；
2. 设置 root item；
3. 配置代理对象以处理用户事件；
4. 配置导航栏的显示效果；
5. 响应用户事件，在合适的时间 push 或者 pop item 对象。

在导航控制器中使用导航栏，大多数都工作都由导航控制器自动完成，但单独使用导航栏时，我们则需要自己处理 push、pop 等逻辑。和其他 `UIView` 组件不一样，我们通常不会直接添加组件到导航栏视图中，而是通过配置 `UINavigationItem` 实例，来控制导航栏的显示， 其协作示意图如下：
![https://docs-assets.developer.apple.com/published/dde7452123/536711f8-0b4b-4ecd-a086-3b8c6feb1a6c.png](https://docs-assets.developer.apple.com/published/dde7452123/536711f8-0b4b-4ecd-a086-3b8c6feb1a6c.png)

* * *

# UINavigationBarDelegate

在导航控制器中使用导航栏，通常不需要自己实现 `UINavigationBarDelegate` 协议，而单独使用导航栏时，我们需要通过实现该协议，来截获相关操作。该协议定义了一下 4 个方法：
`navigationBar(_:shouldPush:)`
`navigationBar(_:didPush:)`
`navigationBar(_:shouldPop:)`
`navigationBar(_:didPop:)`

* * *

# UINavigationItem

`UINavigationItem` 对象封装了导航栏正常显示所需要的必要信息，每一个被推入导航控制器 `viewControllers` 属性中的视图控制器，都必须有一个 `UINavigationItem` 实例。即使单独使用导航栏组件时，在 push 和 pop 操作时，也需要将相关信息封装到 `UINavigationItem` 对象中。

* * *

# UIBarButtonItem

`UIBarButtonItem` 是对导航栏上按钮控件的封装。

* * *
