# 附录 A.与 Pygame 集成

本附录显示了如何在 OpenCV 应用程序中设置 Pygame 库以及如何使用 Pygame 进行窗口管理。 此外，附录还概述了 Pygame 的其他功能以及一些学习 Pygame 的资源。

### 注意

[可以从我的网站下载本章的所有完成代码](http://nummist.com/opencv/3923_06.zip)。

# 安装 Pygame

假设我们已经根据第 1 章，“设置 OpenCV”中描述的方法之一设置了 Python。 根据我们现有的设置，我们可以通过以下方式之一安装 Pygame：

*   带有 32 位 Python 的 Windows：从[这个页面](http://pygame.org/ftp/pygame-1.9.1.win32-py2.7.msi)下载并安装 Pygame 1.9.1。
*   带有 64 位 Python 的 Windows：从[这个页面](http://www.lfd.uci.edu/~gohlke/pythonlibs/2k2kdosm/pygame-1.9.2pre.win-amd64-py2.7.exe)。
*   带有 Macports 的 Mac：打开**终端**并运行以下命令：

    ```py
    $ sudo port install py27-game

    ```

*   带有 Homebrew 的 Mac：打开**终端**并运行以下命令来安装 Pygame 的依赖项，然后安装 Pygame 本身：

    ```py
    $ brew install sdl sdl_image sdl_mixer sdl_ttf smpeg portmidi
    $ /usr/local/share/python/pip install \
    > hg+http://bitbucket.org/pygame/pygame

    ```

*   Ubuntu 及其衍生版本：打开**终端**并运行以下命令：

    ```py
    $ sudo apt-get install python-pygame

    ```

*   其他类似 Unix 的系统：Pygame 在许多系统的标准存储库中可用。 典型的包装名称包括`pygame`，`pygame27`，`py-game`，`py27-game`，`python-pygame,`和`python27-pygame`。

现在，Pygame 应该可以使用了。

# 文档和教程

Pygame 的 API 文档和一些教程可以在[这个页面](http://www.pygame.org/docs/)上在线找到。

Al Sweigart 的《使用 Python 和 Pygame 制作游戏》是一本烹饪手册，用于在 Pygame 1.9.1 中重新创建几个经典游戏。 可在[这个页面](http://inventwithpython.com/pygame/chapters/)上在线获得免费的电子版本，或在[这个页面](http://inventwithpython.com/makinggames.pdf)上下载 PDF 文件。 。

# 子类管理器。

如第 2 章，“处理相机，文件和 GUI”中所述，我们的面向对象设计使我们可以轻松地将 OpenCV 的 HighGUI 窗口管理器替换为另一个窗口管理器，例如 Pygame 。 为此，我们只需要将`managers.WindowManager`类子类化，并覆盖四种方法：`createWindow()`，`show()`，`destroyWindow()`和`processEvents()`。 另外，我们需要导入一些新的依赖项。

要继续，我们需要第 2 章，“处理摄像机，文件和 GUI”的`managers.py`文件和第 4 章“使用 Haar 级联跟踪人脸”的`utils.py`文件。 从`utils.py`中，我们只需要一个函数`isGray()`，我们在第 4 章“用 Haar 级联跟踪人脸”中实现了该功能。 让我们编辑`managers.py`以添加以下导入：

```py
import pygame
import utils
```

同样在`managers.py`中，在实现`WindowManager`之后的某个地方，我们想添加一个名为`PygameWindowManager`的新子类：

```py
class PygameWindowManager(WindowManager):
    def createWindow(self):
        pygame.display.init()
        pygame.display.set_caption(self._windowName)
        self._isWindowCreated = True
    def show(self, frame):
        # Find the frame's dimensions in (w, h) format.
        frameSize = frame.shape[1::-1]
        # Convert the frame to RGB, which Pygame requires.
        if utils.isGray(frame):
            conversionType = cv2.COLOR_GRAY2RGB
        else:
            conversionType = cv2.COLOR_BGR2RGB
        rgbFrame = cv2.cvtColor(frame, conversionType)
        # Convert the frame to Pygame's Surface type.
        pygameFrame = pygame.image.frombuffer(
            rgbFrame.tostring(), frameSize, 'RGB')
        # Resize the window to match the frame.
        displaySurface = pygame.display.set_mode(frameSize)
        # Blit and display the frame.
        displaySurface.blit(pygameFrame, (0, 0))
        pygame.display.flip()
    def destroyWindow(self):
        pygame.display.quit()
        self._isWindowCreated = False
    def processEvents(self):
        for event in pygame.event.get():
            if event.type == pygame.KEYDOWN and \
                    self.keypressCallback is not None:
                self.keypressCallback(event.key)
            elif event.type == pygame.QUIT:
                self.destroyWindow()
                return
```

请注意，我们正在使用两个 Pygame 模块：`pygame.display`和`pygame.event`。

通过调用`pygame.display.init()`创建窗口，并通过调用`pygame.display.quit()`销毁窗口。 重复调用`display.init()`不起作用，因为 Pygame 仅适用于单窗口应用程序。 Pygame 窗口具有`pygame.Surface`类型的绘图表面。 要获得对此`Surface`的引用，我们可以调用`pygame.display.get_surface()`或`pygame.display.set_mode()`。 后一个函数在返回`Surface`实体的属性之前对其进行修改。 `Surface`实体具有`blit()`方法，即，该方法将另一个`Surface`和一个坐标对作为参数，其中后一个`Surface`应该被“涂抹”（绘制）到第一个坐标上。 完成当前帧的窗口`Surface`的更新后，我们应调用`pygame.display.flip()`来显示它。

通过调用`pygame.event.get()`来轮询诸如`keypresses`之类的事件，该事件将返回自上次调用以来发生的所有事件的列表。 每个事件的类型均为`pygame.event.Event`，并具有`type`属性，该属性指示事件的类别，例如，单击的`pygame.KEYDOWN`和单击窗口的**关闭**按钮的`pygame.QUIT` 。 根据`type`的值，`Event`实体可能具有其他属性，例如`KEYDOWN` 事件的`key`（ASCII 密钥代码）。

相对于使用 HighGUI 的基础`WindowManager`，`PygameWindowManager`通过在每帧 OpenCV 的图像格式和 Pygame 的`Surface`格式之间进行转换会产生一些间接费用。 但是，`PygameWindowManager`提供正常的窗口关闭行为，而基本的`WindowManager`不提供。

# 修改应用程序

让我们将`cameo.py`文件修改为使用`PygameWindowManager`而不是`WindowManager`。 在`cameo.py`中找到以下行：

```py
from managers import WindowManager, CaptureManager
```

替换为：

```py
from managers import PygameWindowManager as WindowManager, \
                     CaptureManager
```

就这样！ 现在，`cameo.py`使用一个 Pygame 窗口，当单击标准**关闭**按钮时，该窗口应该关闭。

# Pygame 的进一步使用

我们仅使用了`pygame.display`和`pygame.event`模块的一些基本功能。 Pygame 提供了更多功能，包括：

*   绘制 2D 几何
*   绘图文字
*   管理可绘制 AI 实体（精灵）的组
*   捕获与窗口，键盘，鼠标和操纵杆/游戏手柄相关的各种输入事件
*   创建自定义事件
*   播放和合成声音和音乐

例如，Pygame 可能是适合使用计算机视觉的游戏的后端，而 HighGUI 则不是。

# 总结

到现在为止，我们应该有一个应用程序，该应用程序使用 OpenCV 捕获（并可能操纵）图像，同时使用 Pygame 显示图像和捕获事件。 从这个基本的集成示例开始，您可能想要扩展`PygameWindowManager`以包装其他 Pygame 功能，或者您可能想要创建另一个`WindowManager`子类来包装另一个库。