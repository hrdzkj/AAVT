通俗上讲，OpenGL是一个操作GPU的API，它通过驱动向GPU发送相关指令，控制图形渲染管线状态机的运行状态。但OpenGL需要本地视窗系统进行交互，这就需要一个中间控制层，最好与平台无关。EGL 是 OpenGL ES（嵌入式）和底层 Native 平台视窗系统之间的接口。

EGLContext ——OpenGL ES 图形上下文，它代表了OpenGL状态机；如果没有它，OpenGL指令就没有执行的环境。
EGLConfig ——Surface的EGL配置. ???是EGLSurface的配置吗？

标准 EGL 数据类型如下所示：
EGLBoolean ——EGL_TRUE =1, EGL_FALSE=0
EGLint ——int 数据类型
EGLDisplay，EGLConfig，EGLSurface，EGLContext。

EGL函数API文档
https://www.zybuluo.com/cxm-2016/note/572030#egl%E5%87%BD%E6%95%B0api%E6%96%87%E6%A1%A3

OpenGL ES是一个平台中立的图形库，在它能够工作之前，需要与一个实际的窗口系统关联起来，这与OpenGL是一样的。
OpenGL ES的初始化过程如下图所示意：Display → Config → Surface

SurfaceTexture
https://source.android.google.cn/devices/graphics/arch-st