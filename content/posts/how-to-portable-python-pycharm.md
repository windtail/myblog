---
title: "How to make a portable Python+Pycharm environment"
date: 2024-04-18T19:52:46+08:00
draft: false
tags:
  - python  
---

## Portable Python

从这里下载 [embeddable package](https://www.python.org/downloads/windows/)，
解压之后，第一个问题就是没有pip， 需要下载 [get-pip.py](https://bootstrap.pypa.io/get-pip.py) 并执行。执行后，虽然pip正确安装到了Scripts文件夹中，
但是调用`pip install`时会提示pip未找到，原因是在[官方文档](https://docs.python.org/3/library/sys_path_init.html#pth-files)中，
`._pth`限制了初始化时的搜索路径，这样可以让所有的路径都是预期的。我们可以通过修改`._pth`文件，将`Lib/site-packages`添加进去，
如果没有要限制搜索路径的需求，也可以简单的删除`._pth`文件，这样python就会按照咱们正常的流程来初始化`sys.path`了。

之后，就可以正常安装各种包了，包都会在site-packages，但是仍然有两个问题
1. 没有tinker，一般了用不上，可以不处理，如果需要写点ui，建议直接安装pyside
2. Scripts中的脚本，在挪动位置后都运行不了，因为他们都指向了生成时的python.exe的路径，如果需要可以使用[这个方法]({{< ref "make-exe-like-pip.md" >}})自己重新生成

## Portable Pycharm

在这里下载 [Pycharm的zip版本](https://www.jetbrains.com/pycharm/download/other.html)，解压后找到bin文件夹中的
pycharm.bat，找个地方设置 `PYCHARM_PROPERTIES`环境变量，指向一个自己创建的properties文件，文件内容如下

```ini
# Use ${idea.home.path} macro to specify location relative to IDE installation home.
# Use ${xxx} where xxx is any Java property (including defined in previous lines of this file) to refer to its value.
# Note for Windows users: please make sure you're using forward slashes: C:/dir1/dir2.

#---------------------------------------------------------------------
# Uncomment this option if you want to customize a path to the settings directory.
#---------------------------------------------------------------------
idea.config.path=${idea.home.path}/../runtime/config

#---------------------------------------------------------------------
# Uncomment this option if you want to customize a path to the caches directory.
#---------------------------------------------------------------------
idea.system.path=${idea.home.path}/../runtime/system

#---------------------------------------------------------------------
# Uncomment this option if you want to customize a path to the user-installed plugins directory.
#---------------------------------------------------------------------
idea.plugins.path=${idea.config.path}/plugins

#---------------------------------------------------------------------
# Uncomment this option if you want to customize a path to the logs directory.
#---------------------------------------------------------------------
idea.log.path=${idea.system.path}/log

#---------------------------------------------------------------------
# Maximum file size (in KiB) IDE should provide code assistance for.
# The larger file is the slower its editor works and higher overall system memory requirements are
# if code assistance is enabled. Remove this property or set to very large number if you need
# code assistance for any files available regardless of their size.
#---------------------------------------------------------------------
idea.max.intellisense.filesize=25000

#---------------------------------------------------------------------
# Maximum file size (in KiB) the IDE is able to open.
#---------------------------------------------------------------------
idea.max.content.load.filesize=20000

#---------------------------------------------------------------------
# This option controls console cyclic buffer: keeps the console output size not higher than the specified buffer size (KiB).
# Older lines are deleted. In order to disable cycle buffer use idea.cycle.buffer.size=disabled
#---------------------------------------------------------------------
idea.cycle.buffer.size=1024

#---------------------------------------------------------------------
# Configure if a special launcher should be used when running processes from within IDE.
# Using Launcher enables "soft exit" and "thread dump" features
#---------------------------------------------------------------------
idea.no.launcher=false

#---------------------------------------------------------------------
# To avoid too long classpath
#---------------------------------------------------------------------
idea.dynamic.classpath=false

#---------------------------------------------------------------------
# There are two possible values of idea.popup.weight property: "heavy" and "medium".
# If you have WM configured as "Focus follows mouse with Auto Raise" then you have to
# set this property to "medium". It prevents problems with popup menus on some
# configurations.
#---------------------------------------------------------------------
idea.popup.weight=heavy

#---------------------------------------------------------------------
# Removing this property may lead to editor performance degradation under Windows.
#---------------------------------------------------------------------
sun.java2d.d3d=false

#---------------------------------------------------------------------
# Removing this property may lead to editor performance degradation on Java 8+.
#---------------------------------------------------------------------
swing.bufferPerWindow=true

#---------------------------------------------------------------------
# Removing this property may lead to editor performance degradation under X Window.
#---------------------------------------------------------------------
sun.java2d.pmoffscreen=false

#---------------------------------------------------------------------
# Enables HiDPI support in JBR
#---------------------------------------------------------------------
sun.java2d.uiScale.enabled=true

#---------------------------------------------------------------------
# Applicable to the Swing text components displaying HTML (except JEditorPane).
# Rebases CSS size map depending on the component's font size to let relative
# font size values (smaller, larger) scale properly. JBR-only.
#---------------------------------------------------------------------
javax.swing.rebaseCssSizeMap=true


#---------------------------------------------------------------------
# Workaround for accessing (in terms of a11y) long VCS logs on macOS. JBR-only.
#---------------------------------------------------------------------
sun.awt.mac.a11y.tableAccessibleRowCountThreshold=1000

#---------------------------------------------------------------------
# Enabling an optimization that excludes traversal of collapsed accessible nodes from the accessible tree. JBR-4167
#---------------------------------------------------------------------
javax.swing.JTree.excludeAccessibleChildrenFromClosedNodes=true

#---------------------------------------------------------------------
# Workaround to avoid long hangs while accessing clipboard under Mac OS X.
#---------------------------------------------------------------------
#ide.mac.useNativeClipboard=True

#---------------------------------------------------------------------
# Maximum size (KiB) the IDE will use to show historical file contents -
# in Show Diff or when calculating Digest Diff
#---------------------------------------------------------------------
#idea.max.vcs.loaded.size.kb=20480

#---------------------------------------------------------------------
# IDEA file chooser peeks inside directories to detect whether they contain a valid project
# (to mark such directories with a corresponding icon).
# Uncommenting the option prevents this behavior outside the user home directory.
#---------------------------------------------------------------------
#idea.chooser.lookup.for.project.dirs=false

#---------------------------------------------------------------------
# In LWCToolkit.invokeAndWait() listens to EDT state and disposes the invocation event
# when EDT becomes free but the invocation event is not yet dispatched (considered lost).
# This prevents a deadlock and makes the invocation return some default result.
#---------------------------------------------------------------------
sun.lwawt.macosx.LWCToolkit.invokeAndWait.disposeOnEDTFree=true

#---------------------------------------------------------------------
# Experimental options that do a number of things to make truly smooth scrolling possible:
#
# * Enables hardware-accelerated scrolling.
#     Blit-acceleration copies as much of the rendered area as possible and then repaints only newly exposed region.
#     This helps to improve scrolling performance and to reduce CPU usage (especially if drawing is compute-intensive).
#
# * Enables "true double buffering".
#     True double buffering is needed to eliminate tearing on blit-accelerated scrolling and to restore
#     frame buffer content without the usual repainting, even when the EDT is blocked.
#
# * Adds "idea.true.smooth.scrolling.debug" option.
#     Checks whether blit-accelerated scrolling is feasible, and if so, checks whether true double buffering is available.
#
# * Enables handling of high-precision mouse wheel events.
#     Although Java 7 introduced MouseWheelEven.getPreciseWheelRotation() method, JScrollPane doesn't use it so far.
#     Depends on the Editor / General / Smooth Scrolling setting, remote desktop detection and power save mode state.
#     Ideally, we need to patch the runtime (on Windows, Linux and macOS) to improve handling of the fine-grained input data.
#     This feature can be toggled via "idea.true.smooth.scrolling.high.precision" option.
#
# * Enables handling of pixel-perfect scrolling events.
#     Currently, this mode is available only under macOS with JetBrains Runtime.
#     This feature can be toggled via "idea.true.smooth.scrolling.pixel.perfect" option.
#
# * Enables interpolation of scrolling input (scrollbar, mouse wheel, touchpad, keys, etc).
#     Smooths input, which lacks both spatial and temporal resolution, performs the rendering asynchronously.
#     Depends on the Editor / General / Smooth Scrolling setting, remote desktop detection and power save mode state.
#     The feature can be tweaked using the following options:
#       "idea.true.smooth.scrolling.interpolation" - the main switch
#       "idea.true.smooth.scrolling.interpolation.scrollbar" - scrollbar interpolation
#       "idea.true.smooth.scrolling.interpolation.scrollbar.delay" - initial delay for scrollbar interpolation (ms)
#       "idea.true.smooth.scrolling.interpolation.mouse.wheel" - mouse wheel / touchpad interpolation
#       "idea.true.smooth.scrolling.interpolation.mouse.wheel.delay.min" - minimum initial delay for mouse wheel interpolation (ms)
#       "idea.true.smooth.scrolling.interpolation.mouse.wheel.delay.max" - maximum initial delay for mouse wheel interpolation (ms)
#       "idea.true.smooth.scrolling.interpolation.precision.touchpad" - touchpad interpolation
#       "idea.true.smooth.scrolling.interpolation.precision.touchpad.delay" - initial delay for touchpad interpolation (ms)
#       "idea.true.smooth.scrolling.interpolation.other" - interpolation of other input sources
#       "idea.true.smooth.scrolling.interpolation.other.delay" - initial delay for other input source interpolation (ms)
#
# * Adds on-demand horizontal scrollbar in editor.
#     The horizontal scrollbar is shown only when it's actually needed for currently visible content.
#     This helps to save editor space and to prevent occasional horizontal "jitter" on vertical touchpad scrolling.
#     This feature can be toggled via "idea.true.smooth.scrolling.dynamic.scrollbars" option.
#---------------------------------------------------------------------
#idea.true.smooth.scrolling=true

#-----------------------------------------------------------------------
# Change to 'enabled' if you want to receive instant visual notifications
# about fatal errors that happen to an IDE or plugins installed.
#-----------------------------------------------------------------------
idea.fatal.error.notification=disabled
```

注意配置`idea.config.path`和`idea.system.path`，使之相对于安装目录。

每次启动时都需要从pycharm.bat启动即可。

为了让pycharm默认使用上面的python，可以在 `idea.config.path` 下面创建 `config/options/jdk.table.xml`，内容类似

```xml
<application>
  <component name="ProjectJdkTable">
    <jdk version="2">
      <name value="Python 3.11" />
      <type value="Python SDK" />
      <version value="Python 3.11.9" />
      <homePath value="$APPLICATION_HOME_DIR$\..\python\python.exe" />
      <roots>
        <classPath>
          <root type="composite">
            <root url="jar://$APPLICATION_HOME_DIR$/../python/python311.zip!/" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/../python" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/python-skeletons" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stdlib" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/gdb" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/six" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/boto" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/cffi" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/mock" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/pika" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/pytz" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/toml" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/tqdm" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/zstd" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/first" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/fpdf2" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/ldap3" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/polib" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/pyjks" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/redis" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/regex" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/retry" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/ujson" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/uWSGI" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/WebOb" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/bleach" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/caldav" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/docopt" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/hdbcli" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/ibm-db" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/invoke" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/passpy" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/peewee" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/Pillow" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/pluggy" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/psutil" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/pyasn1" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/pycurl" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/pynput" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/pysftp" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/PyYAML" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/qrcode" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/stripe" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/zxcvbn" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/boltons" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/chevron" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/inifile" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/libsass" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/netaddr" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/passlib" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/pexpect" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/pyaudio" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/PyMySQL" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/pyvmomi" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/pywin32" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/slumber" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/tzlocal" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/vobject" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/WTForms" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/aiofiles" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/colorama" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/croniter" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/docutils" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/ExifRead" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/greenlet" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/html5lib" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/httplib2" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/jmespath" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/keyboard" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/Markdown" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/oauthlib" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/openpyxl" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/paramiko" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/psycopg2" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/pyflakes" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/Pygments" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/pyserial" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/requests" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/tabulate" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/toposort" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/untangle" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/waitress" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/braintree" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/decorator" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/paho-mqtt" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/playsound" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/PyAutoGUI" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/pyOpenSSL" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/pyRFC3339" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/PyScreeze" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/ttkthemes" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/xmltodict" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/cachetools" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/commonmark" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/dateparser" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/Deprecated" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/Flask-Cors" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/jsonschema" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/pyfarmhash" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/Send2Trash" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/setuptools" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/simplejson" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/SQLAlchemy" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/tensorflow" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/entrypoints" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/flake8-2020" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/JACK-Client" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/mysqlclient" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/opentracing" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/pep8-naming" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/pycocotools" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/pyinstaller" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/python-jose" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/python-nmap" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/python-xlib" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/stdlib-list" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/tree-sitter" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/workalendar" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/aws-xray-sdk" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/console-menu" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/editdistance" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/parsimonious" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/usersettings" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/whatthepatch" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/click-spinner" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/Flask-Migrate" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/humanfriendly" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/python-gflags" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/beautifulsoup4" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/flake8-bugbear" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/Flask-SocketIO" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/python-crontab" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/python-slugify" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/singledispatch" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/flake8-builtins" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/flake8-simplify" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/influxdb-client" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/mypy-extensions" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/python-datemath" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/python-dateutil" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/dockerfile-parse" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/Flask-SQLAlchemy" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/s2clientprotocol" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/flake8-docstrings" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/requests-oauthlib" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/translationstring" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/click-default-group" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/flake8-plugin-utils" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/pytest-lazy-fixture" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/flake8-rst-docstrings" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/flake8-typing-imports" type="simple" />
            <root url="file://$APPLICATION_HOME_DIR$/plugins/python-ce/helpers/typeshed/stubs/tree-sitter-languages" type="simple" />
          </root>
        </classPath>
        <sourcePath>
          <root type="composite" />
        </sourcePath>
      </roots>
      <additional SDK_UUID="250dbd04-fd5f-4252-8f97-5f9517ebe4e7">
        <setting name="FLAVOR_ID" value="WinPythonSdkFlavor" />
        <setting name="FLAVOR_DATA" value="{}" />
      </additional>
    </jdk>
  </component>
</application>
```

注意 `$APPLICATION_HOME_DIR$`指的是pycharm的安装路径

> 参考：[stackoverflow](https://stackoverflow.com/questions/68182223/where-does-pycharm-store-the-python-interpreters)
