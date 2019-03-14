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

OpenGL是一个开放的三维图形软件包，它独立于窗口系统和操作系统。OpenGL使用简便，效率高。它具有七大功能
建模，变换，颜色模式设置，光照和材质设置，纹理映射，位图显示和图象增强图象功能，双缓存动画等



1.音视频  可视电话
imediaDataSource---自定义数据源，也是只能从源头或数据，不适用。
先调研软解码的效率，软件编码效率（android avframe压缩录像）。


如何和厂家联系沟通

GL_TEXTURE_EXTERNAL_OES， 该Target也是主要用于从EGLImage中产生纹理的情景
updateTexImage()方法会将ImageStream的图片数据更新到GL_OES_EGL_image_external类型的纹理中。
每当使用该类纹理对纹理对象进行绑定时，需使用GL_TEXTURE_EXTERNAL_OES而不是GL_TEXTURE_2D。

mEgl.eglGetError();获取上次egl执行函数的错误码，通过比较错误码，可以找到错误问题点

eglMakeCurrent()
在完成EGL的初始化之后，需要通过eglMakeCurrent()函数来将当前的上下文切换，这样opengl的函数才能启动作用。
boolean eglMakeCurrent(EGLDisplay display, EGLSurface draw, EGLSurface read, EGLContext context)
该接口将申请到的display，draw（surface）和 context进行了绑定。也就是说，在context下的OpenGLAPI指令将draw（surface）作为其渲染最终目的地。
而display作为draw（surface）的前端显示。调用后，当前线程使用的EGLContex为context。


eglPresentationTimeANDROID 设置时间戳，单位是纳秒

eglSwapBuffers  https://blog.csdn.net/qiuyun0214/article/details/54614892


Semaphore信号量。
semaphore.acquire();从信号量中获取一个许可
semaphore.release();释放一个许可(在释放许可之前，必须先获获得许可。)
所有函数https://blog.csdn.net/yuruixin_china/article/details/82084946
new Semaphore(0)---暂时理解，不允许有线程能同时访问




中间层的思想：
OpenGL ES 定义了一个渲染图形的 API。它没有定义窗口系统。
在android中， EGL就是创建OpenGL ES可用的“绘图表面”，同步不同类别的API之间的渲染(OpenGL和本地窗口的绘图命令之间),
和本地窗口系统（native windowing system）通讯
帮助实现openggl独立性，保证了OpenGL ES的平台独立性.

EGLSurfaces https://blog.csdn.net/tq08g2z/article/details/77311887
EGLSurface 可以是一个 EGL 分配的离屏缓冲区 (称为 "pbuffer") 或由操作系统分配的窗口。
EGL 窗口 surfaces 由 eglCreateWindowSurface() 调用创建。它接收一个 "窗口对象" 作为参数，
在 Android 上它可以是一个 SurfaceView，SurfaceTexture，SurfaceHolder 或 Surface -- 所有在底层具有 BufferQueue 的东西。
当你执行这个调用时，EGL 创建一个新的 EGLSurface 对象，将它与窗口对象的 BufferQueue 的生产者接口连接。从那时起，
向那个 EGLSurface 渲染将使得一个缓冲区被取出，向其中渲染，并放回以由消费者使用。
同一时间只有一个 EGLSurface 可以与一个 Surface 关联，销毁了 EGLSurface 它将从 BufferQueue 断开，并允许其它东西连接。
通过修改什么是当前的 EGLSurface 在多个 EGLSurface 之间切换。同一时间一个 EGLSurface 只能是一个线程的当前 EGLSurface。
EGLSurface 有关的最常见的错误是假设它只是 Surface 的另一方面（像 SurfaceHolder 一样）。它是相关但独立的概念。
你可以在一个后端没有 Surface 支持的 EGLSurface 上绘制，且你可以不通过 EGL 使用 Surface。EGLSurface 仅仅给了 GLES 一个绘制的平面。


eglSwapBuffers(EGLDisplay dpy, EGLContext ctx)
EGLDisplay 是一个关联系统物理屏幕的通用数据类型,表示显示设备句柄,也可以认为是一个前端显示窗。
其实就是交换缓冲区啦，用了这种技术的话，你所有的绘制都是绘制到一个后台的缓冲区里面的，
如果不交换缓冲区，就看不到绘制的东西了。 surface有一个BufferQueue，采用生产者消费者用的形式。

OpenGL 绘图的机制是: 先用 OpenGL 的绘图上下文 Rendering Context (简称为 RC )把图画好，
再把所绘结果通过 SwapBuffer() 函数传给 Window 的 绘图上下文。


EGL 是一层接口，上层跟 OpenGL 对接，下层跟本地窗口系统对接，负责隔离 OpenGL 与本地窗口的依赖.
OpenGL 本身只专注于渲染流程，核心就是 Pipeline(管线)的处理.
如果没有 EGL 提供的渲染上下文，则 OpenGL 无法执行.
GLSurfaceView已经包含了EGL这一块


共享texture:surfaceView预览和glSurfaceView共享texture 。本地窗口和opengl渲染共享纹理texture

SurfaceView=Surface+View 
              
             mCamera.setSurface()

1.）如何归一坐标转换到屏幕坐标：
投影变换和视口变换合起来才决定场景如何映射到屏幕上
Matrix.orthoM
正交投影(Orthographic Projection)：
	/**
	 * 第三步 ： 根据屏幕的width 和 height 创建投影矩阵 https://blog.csdn.net/b1480521874/article/details/54292754
	 * @param width
	 * @param height
	 */
	 public void projectionMatrix(int width,int height){
		 final float aspectRatio = width > height ?
				 (float) width / (float) height :
			     (float) height / (float) width;
		 if(width > height){
			 Matrix.orthoM(projectionMatrix, 0, -aspectRatio, aspectRatio, -1f, 1f, -1f, 1f);
		 }else{
			 Matrix.orthoM(projectionMatrix, 0, -1f, 1f, -aspectRatio, aspectRatio, -1f, 1f);
		 }
	}
有两个函数可以生成透视投影矩阵frustumM和perspectiveM	
	
将摄像机矩阵, 投影矩阵, 着色矩阵相乘, 就是最终矩阵;Matrix.multiplyMM


2.）流程：
应用程序会先创建一个SurfaceTexture，然后将SurfaceTexture传递给图形生产者对象（比如Camera，通过调用setPreviewTexture传递），图形生产者对象生产一帧数据后，会回调onFrameAvailable通知应用程序有新的图像数据可以使用，
思路：
c层实现：雷神和其他 1.完成后再进行转码添加水印  2.压缩也转移到获取avpacket的地方。
java层实现：湖广午王 https://blog.csdn.net/junzia/article/details/77924629
             https://blog.csdn.net/u010302327/article/details/79259661
           https://blog.csdn.net/junzia/article/details/78154648
           https://blog.csdn.net/junzia/article/details/54018671
          https://blog.csdn.net/STN_LCD/article/details/74926376
共享纹理：https://blog.csdn.net/cmshao/article/details/80060546
      https://cloud.tencent.com/developer/article/1369883
      
绘制三角形https://github.com/tong123/OpenGLTriangle  熟悉glsl 已经fork
     解释https://www.cnblogs.com/designyourdream/p/6739413.html

应用程序就可以调用updateTexImage将图像数据先送到Texture，之后就可以调用opengl接口做些具体的业务了。
Camera-->SurfaceTexture--->OnFrameAvailableListener--->updateTexImage

OnDrawFrame方法中将更新后的纹理渲染到屏幕。

SurfaceTexture，和SurfaceView不同的是，它对图像流的处理并不直接显示，而是转为GL外部纹理，因此可用于图像流数据的二次处理（如Camera滤镜，桌面特效等），和OpenGL ES一起使用可以创造出无限可能。
SurfaceTexture的updateTexImage方法会更新接收到的预览数据到其绑定的OpenGL纹理中。此方法只能在生成该纹理的OpenGL线程中调用。
SurfaceTexture.setOnFrameAvailableListener
SurfaceView.getHolder().addCallback
getTransformMatrix
当从OpenGL ES的纹理对象取样时，首先应该调用getTransformMatrix()来转换纹理坐标。
每次updateTexImage()被调用时，纹理矩阵都可能发生变化。所以，每次texture image被更新时，getTransformMatrix ()也应该被调用。getTransformMatrix()得到的矩阵，将传统的形如(s,t,0,1的)OpenGL ES 二维纹理坐标列向量转换为纹理流中正确的采样位置。

SurfaceTexture的getTransformMatrix方法可以获取到图像数据流的坐标变换矩阵。


色器程序控制渲染流程：坐标变换(顶点着色器）--->图元装配--->构造出新图元(几何着色器)
--->映射像素(光栅化)--->裁切(片段着色器)--->测试和混合
在现代OpenGL中，我们必须定义至少一个顶点着色器和一个片段着色器（因为GPU中没有默认的顶点/片段着色器）。 

void glBindFramebuffer(GLenum target, GLuint id)
第一个参数target应该是GL_FRAMEBUFFER，第二个参数是FBO的ID号。一旦FBO被绑定，之后的所有的OpenGL操作都会对当前所绑定的FBO造成影响。ID号为0表示缺省帧缓存，即默认的window提供的帧缓存。因此，在glBindFramebuffer()中将ID号设置为0可以解绑定当前FBO。
