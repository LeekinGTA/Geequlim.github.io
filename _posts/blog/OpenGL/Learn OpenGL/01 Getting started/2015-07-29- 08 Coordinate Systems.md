---
layout: post
title: '[第01章 入门] 08 坐标系统'
category: 'Learn OpenGL'
tags: 'LearnOpenGL'
previousTitle: '[第01章 入门] 07 变换'
previousUrl: '/2015/07/28/%2007%20Transformations.html'
nextTitle: '[第01章 入门] 09 摄像机'
nextUrl: '/2015/07/29/%2009%20Camera.html'
description:
---

原文     | [Coordinate Systems](http://learnopengl.com/#!Getting-started/Coordinate-Systems)
      ---|---
作者     | JoeyDeVries
翻译     | linkoln
校对     | Geequlim

<br/>

在上一个教程中，我们学习了如何有效地利用矩阵变换来对所有顶点进行转换。OpenGL希望在所有顶点着色器运行后，所有我们可见的顶点都在一个规则化的设备范围（NDC）里面。也就是说，每个顶点的x，y，z坐标都应该在-1.0到1.0之间，超出这个坐标范围的顶点都将不可见。我们经常做的是，通过自己定义坐标的范围并且在顶点着色器中将这些坐标转换为标准化设备坐标系(Normalized Device Coordinates，NDC)。然后将这些标准化设备坐标传入光栅器，再将他们转换为屏幕上的二维坐标或像素。

将坐标转换为标准化设备坐标接着转化为屏幕坐标通常是在一个流水线里面实现的，在流水线里面我们在将对象转换到屏幕坐标系之前会先将其转换到多个坐标系统。将对象的坐标转换到几个中间坐标系的优点在于，在这些特定的坐标系统中进行一些操作或运算更加方便和容易，这一点很快将会变得很明显。对我们来说比较重要的总共有5个不同的坐标系统：

 - 局部空间（或者称为对象坐标系）
 - 世界空间
 - 视口坐标系（或者称为视觉坐标系）
 - 裁剪坐标系
 - 屏幕坐标系

这些就是我们将所有顶点转换为图元之前，顶点所处于的不同的状态。
你现在可能对什么是空间或坐标系到底是什么感到困惑，所以接下来我们将会通过展示完整的图片来解释每一个坐标系实际做了什么。

<br/>

### 整体概述

为了将坐标从一个坐标系转换到另一个坐标系，我们需要用到几个转换矩阵，最重要的几个分别是模型矩阵、视图矩阵、投影矩阵。首先，顶点坐标开始于局部空间，称为本地坐标，然后经过世界坐标，视口坐标，裁剪坐标，并最后以屏幕坐标结束。下面的图示显示了整个流程及各个转换过程做了什么：

<img class="post_center_img_noborder" src="/assets/img/blog/LearnOpenGL/01 Getting started/coordinate_systems.png"/>

 1. 本地坐标是对象相对于本地原点的坐标；也是对象开始的坐标。
 2. 将本地坐标转换为世界坐标，世界坐标是作为一个更大空间范围的坐标系统。这些坐标是相对于世界的原点的。
 3. 接下来我们将世界坐标转换为视口坐标，视口坐标是指以摄像机或观察者的角度观察的坐标。
 4. 在将坐标处理到视口空间之后，我们需要将其投影到裁剪坐标。裁剪坐标是处理-1.0到1.0范围内并判断哪些顶点将会出现在屏幕上。
 5. 最后，我们需要将裁剪坐标转换为屏幕坐标，我们将这一过程成为视口变换。视口变换将位于-1.0到1.0范围的坐标转换到由`glViewport`函数所定义的坐标范围内。最后转换的坐标将会送到光栅器，由光栅器将其转化为片段。

你可能了解了每个单独的坐标空间的作用。我们之所以将顶点转换到各个不同的空间的原因是有些操作在特定的坐标系统中才有意义且更方便。例如，当修改对象时，如果在本地空间中则是有意义的；当对对象做相对于其它对象的位置的操作时，在世界坐标系中则是有意义的；等等这些。如果我们愿意，本可以定义一个直接从本地空间到裁剪空间的转换矩阵，但那样会失去灵活性。接下来我们将要更仔细地讨论各个坐标系。

<br/>

#### 本地空间(Local spcae)

本地空间是指对象所在的坐标空间，例如，对象最开始所在的地方。想象你在一个模型建造软件中创建了一个立方体。你创建的立方体的原点有可能位于（0，0，0），即使有可能在最后的应用中位于另外一个位置。有可能你创建的所有模型都以（0，0，0）为初始位置。则你的模型的所有顶点都是在本地空间：他们相对于你的对象来说都是本地的。
<br/>

#### 世界空间(World space)

如果我们想将我们所有的对象传入应用中，它们有可能会在世界的原点（0，0，0）互相重叠，而这并不是我们想见到的。我们想为每一个对象定义一个位置，以使对象位于更大的世界坐标系中。世界空间中的坐标就如它们听起来那样：是指顶点相对于（虚拟）世界的坐标。世界坐标系是将分散的各个对象转换到同一个坐标系中（或者一个更实际的方式）。对象的坐标将会从本地坐标转换到世界坐标；该转换是由模型矩阵实现的。模型矩阵是一种转换矩阵，它能通过对对象进行平移、缩放、旋转来将它置于它本应该在的位置或方向。你可以想象一下，转换一栋房子，通过将它缩小（因为它在本地坐标系中显得太大了），将它往郊区的方向平移且沿着y轴往坐标旋转，经过这样的变换之后，它将恰好能够与邻居的房子重合。你能够想到上一节讲到的利用模型矩阵将各个容器放置到这个屏幕上；我们能够将容器中的本地坐标转换为视口坐标或世界坐标。
<br/>

#### 观察空间(View space)

观察空间经常被人们称之OpenGL的摄像机（所以有时也称为摄像机空间或视觉空间）。观察空间就是将对象的世界空间的坐标转换为观察者视野前面的坐标。因此观察空间就是从摄像机的角度观察到的空间。而这通常是由一系列的平移和旋转的组合来平移和旋转场景以使特定的对象被转换到摄像机前面。这些组合在一起的转换通常存储在一个视图矩阵用来将世界坐标转换到观察空间。在下一个教程我们将广泛讨论如何创建一个这样的视图矩阵来模拟一个摄像机。
<br/>

#### 裁剪空间(Clip space)

在一个顶点着色器运行的最后，OpenGL期望所有的坐标都能落在一个给定的范围内，且任何在这个范围之外的点都应该被裁剪。被裁剪的坐标就变成无效的，所以剩下的坐标就将变为屏幕上可见的片段。这也是裁剪空间名字的由来。

因为将所有可见的坐标都放置在-1.0到1.0的范围内不是很直观，我们会指定自己的坐标集的方式并将它转换回规则化设备坐标系就像OpenGL期望它做的那样。

为了将顶点坐标从观察空间转换到裁剪空间，我们定义一个投影矩阵，它指定了坐标的范围，例如，每个维度都是从-1000到1000。投影矩阵接着会将在它指定的范围内的坐标转换到规则化设备坐标系中（-1.0，1.0）。所有在范围外的坐标在-1.0到1.0之间都不会被绘制出来且会被裁剪。在投影矩阵所指定的范围内，坐标（1250，500，750）将是不可见的，这是由于它的x坐标超出了范围，随后被转化为在规则化设备坐标中坐标值大于1.0的值并且被裁剪掉。

<div class="green_box">
	<p class="green_title">提示</p>
	<p class="box_content">
    如果只是图元的一部分例如三角形，超出了裁剪区域，则OpenGL会重新构建三角形以使一个或多个三角形能适应在裁剪范围内。
	</p>
</div>

由投影矩阵创建的观察区域被称为平截头体（Frustum），且每个出现在平截头体范围内的坐标都会最终出现在用户的屏幕上。将坐标转化到指定的规则化设备坐标系，它可以很容易被映射到2维观察空间的坐标系,这个过程被称之为投影，使用投影矩阵能将3维坐标投影到标准化设备坐标系中。

一旦所有顶点被转换到裁剪空间，一个最终操作——投射除法将会执行，在于我们将位置向量的x，y，z分量分别除以向量的均匀w分量；投射除法是将4维裁剪空间坐标转换为3维规则化设备坐标系。这一步会在每一个顶点着色器运行的最后被自动执行。
在这一阶段之后，坐标经过转换的结果将会被映射到屏幕坐标系（由`glViewport`设置）且被转变成片元。

投影矩阵将观察坐标转换为裁剪坐标的过程采用两种不同的方式，每种方式分别定义自己的平截头体。我们可以创建一个正视投影矩阵或一个透视投影矩阵。
<br/>

### 正视投影（Orthographic projection）

正视投影矩阵定义了一个类似立方体的平截头体，指定了一个裁剪空间，每一个在这空间外面的顶点都会被裁剪。创建一个正视投影矩阵需要指定可见平截头体的宽、高和长度。所有在使用正视投影矩阵转换到裁剪空间后如果还处于这个平截头体里面的坐标就不会被裁剪。它的平截头体看起来像一个容器：

<img class="post_center_img_noborder" src="http://learnopengl.com/img/getting-started/orthographic_frustum.png"/>

上面的平截头体定义了由宽、高、近平面和远平面决定的可视的坐标系。任何出现在近平面前面或远平面后面的坐标都会被裁剪掉。正视平截头体直接将平截头体内部的顶点映射到规则化设备坐标系中，因为每个向量的w分量都是不变的；如果w分量等于1.0，则透视除法不会改变坐标的值。

为了创建一个正视投影矩阵，我们利用GLM的构建函数`glm::ortho`：

{% highlight c++ %}
glm::ortho(0.0f, 800.0f, 0.0f, 600.0f, 0.1f, 100.0f);
{% endhighlight %}

头两个参数指定了平截头体的左右坐标，第三和第四参数指定了平截头体的底部和上部。通过这四个参数我们定义了近平面和远平面的大小，然后第五和第六个参数则定义了近平面和远平面的距离。这个指定的投影矩阵将处于这些x，y，z范围之间的坐标转换到规则化设备坐标系中。

正视投影矩阵直接将坐标映射到屏幕的二维平面内，但实际上一个直接的投影矩阵将会产生不真实的结果，如果这个投影不把透视考虑进去的话。这就是透视投影矩阵为我们解决的。
<br/>

### 透视投影（Perspective projection）

如果你曾经体验过实际生活给你带来的景象，你就会注意到离你越远的东西看起来更小。这个神奇的效果我们称之为透视，它是因为我们是从两个视口看世界的：分别从两只眼睛。透视的效果在我们看一条无限长的高速公路或铁路时尤其明显，正如下面图片显示的那样:

<img class="post_center_img_noborder" src="http://learnopengl.com/img/getting-started/perspective.png"/>


正如你看到的那样，由于透视的原因，平行线似乎在很远的地方看起来会相交。这个投影矩阵将给定的平截头体范围映射到裁剪空间，但每个顶点坐标仍然产生了w值，顶点如果离观察者越远，则w分量越大。被转换到裁剪控件的坐标都会在-w到w的范围之间（任何处于这个范围的对象都会被裁剪）。OpenGL需要可见的坐标都落在-1.0到1.0范围内作为最后的顶点着色器输出，因此一旦坐标在裁剪空间内，透视划分就会被应用到裁剪空间坐标内：

$$out = \begin{pmatrix} x /w \\ y / w \\ z / w \end{pmatrix}$$

每个顶点坐标的分量都会除以它的w分量，得到一个距离观察者的较小的顶点坐标。这是另一个w分量很重要的原因，因为它帮助我们进行透射投影。最后的结果坐标就是处于标准化设备空间内的。如果你对研究正视投影矩阵和透视投影矩阵是如何计算的很感兴趣（且不会对数学感到恐惧）我推荐[这篇由Songho写的文章](http://www.songho.ca/opengl/gl_projectionmatrix.html)。

一个透视投影矩阵可以在GLM中被创建，如下：

{% highlight c++ %}
glm::mat4 proj = glm::perspective(45.0f, (float)width/(float)height, 0.1f, 100.0f);
{% endhighlight %}

`glm::perspective`所做的其实就是再次创建了一个定义了可视空间的大的平截头体，任何在这个平截头体的对象最后都不会出现在裁剪空间体积内并且将会被裁剪。一个透视平截头体可以被可视化为一个不均匀形状的盒子，在这个盒子内部的每个坐标都会被映射到裁剪空间的点。一张透视平截头体的照片如下所示：

<img class="post_center_img_noborder" src="http://learnopengl.com/img/getting-started/perspective_frustum.png"/>


它的第一个参数定义了fov的值，代表了视界范围，且设置了视图空间有多大。对于一个真实的视图，它的值经常设置为45.0，但想要看到更多结果你可以设置一个更大的值。第二个参数设置了宽高比，由视图的高除以宽。第三和第四个参数设置了平截头体的近和远平面。我们经常设置近距离为0.1而远距离设为100.0。所有在近平面和远平面的顶点且处于平截头体内的顶点都会被渲染。

<div class="green_box">
	<p class="green_title">提示</p>
	<p class="box_content">
  当你把透视矩阵的<code>near</code>值设置太大时（如10.0），OpenGL会将靠近摄像机的坐标都裁剪掉（在0.0和10.0之间），这提供了一个相似的视觉效果就如视频游戏中如果你移动得太靠近某个对象时你会把它看穿那样。
	</p>
</div>

当使用正视投影时，每一个顶点坐标都会直接映射到裁剪空间中而不经过任何透视划分（它仍然有进行透视划分，只是w分量没有被操作（它保持为1）因此没有起作用）。因为正视投影没有使用透视，远处的对象不会显得小以产生神奇的视觉输出。由于这个原因，正视投影主要用于二维渲染以及一些建筑或工程的应用,那些我们不需要使用投影来变形顶点的情况下。某些应用如Blender是用来进行三维建模的，有时在建模时会使用正视投影，因为它更准确地藐视了每个对象的各个方面。下面你能够看到在Blender里面使用两种投影方式的对比：

<img class="post_center_img_noborder" src="http://learnopengl.com/img/getting-started/perspective_orthographic.png"/>

你可以看到使用透视投影的话，远处的顶点看起来比较小，而在正视投影中每个顶点距离观察者的距离都是一样的。

<br/>

### 组合

我们为上述的每一个步骤都创建了一个转换矩阵：模型矩阵、视图矩阵和投影矩阵。一个顶点的坐标将会被转换到裁剪坐标，如下：

$$V_{clip} = M_{projection} \cdot M_{view} \cdot M_{model} \cdot V_{local}$$

注意每个矩阵被运算的顺序是相反的（记住我们需要从右往左乘上每个矩阵）。最后的顶点应该被赋予顶点着色器中的`gl_Position`且OpenGL将会自动进行透视划分和裁剪。

<div class="green_box">
	<p class="green_title">然后呢？</p>
	<p class="box_content">
  顶点着色器的输出需要所有的顶点都在裁剪空间内，而这是我们的转换矩阵所做的。OpenGL然后在裁剪空间中执行透视划分从而将它们转换到规则化设备坐标。OpenGL会使用<code>glViewPort</code>内部的参数来将规则化设备坐标映射到屏幕坐标，每个坐标都关联了一个屏幕上的点（在我们的例子中屏幕是<code>800*600</code>）。这个过程称为视口转换。
	</p>
</div>

这一章的主题可能会比较难理解，如果你仍然不确定每个空间的作用的话，你也不必太担心。接下来你会看到我们是怎样好好运用这些坐标空间的并且会有足够的展示例子在接下来的教程中。

<br/>

## 进入三维

既然我们知道了如何将三维坐标转换为二维坐标，我们可以开始将我们的对象展示为三维对象而不是目前我们所展示的差劲的二维图像。

在开始进行三维画图时，我们首先创建一个模型矩阵。这个模型矩阵包含了平移、缩放与旋转，我们将会运用来将对象的顶点转换到全局世界空间。让我们平移一下我们的平面，通过将其绕着x轴旋转使它看起来像位于地板上一样。这个模型矩阵看起来是这样的：

{% highlight glsl %}
glm::mat4 model;
model = glm::rotate(model, -55.0f, glm::vec3(1.0f, 0.0f, 0.0f));
{% endhighlight %}

通过将顶点坐标乘以这个模型矩阵我们将该顶点坐标转换到世界坐标。我们的平面看起来就是在地板上的因此可以代表真实世界的平面。

接下来我们需要创建一个视图矩阵。我们想要在场景里面稍微往后移动以使得对象变成可见的（当在世界空间时，我们位于原点（0,0,0））。在场景里面移动，思考下面的问题：

将摄像机往后移动跟将整个场景往前移是一样的。

这就是视图空间所做的，我们移动整个场景正如我们相反地想让摄像机所做的那样。因为我们想要往后移动且OpenGL是一个右手坐标系所以我们沿着z轴的负方向移动。我们实现这个，通过将场景沿着z轴正方向平移。这个我们的感觉就是我们在往后移动。

<div class="green_box">
	<p class="green_title">右手坐标系</p>
  <div class="box_content">
	<p>
  按照约定，OpenGL是一个右手坐标系。最基本的就是说正x轴在你的右手边，正y轴往上而正z轴是往后的。想象你的屏幕处于三个轴的中心且正z轴穿过你的屏幕朝向你。坐标系画起来如下：
  <br/>
  </p>
  <img class="post_center_img_noborder" src="http://learnopengl.com/img/getting-started/coordinate_systems_right_handed.png"/>
  <br/>
	为了理解为什么被称为右手坐标系，按如下的步骤做：
  <br/><br/>
	· 张开你的右手使正y轴沿着你的手往上。<br/>
	· 使你的大拇指往右。<br/>
	· 使你的食指往上。<br/>
	· 向下90度弯曲你的中指。<br/><br/>
  如果你都正确地做了，那么你的大拇指朝着正x轴方向，食指朝着正y轴方向，中指朝着正z轴方向。如果你用左手来做这些动作，你会发现z轴的方向是相反的。这就是有名的左手坐标系，被DirectX广泛地使用。注意在标准化设备坐标系中OpenGL使用的是左手坐标系（投影矩阵转换了使用右手或左手的习惯）。
  </div>
</div>

接下来我们将会讨论如何利用更多细节来在场景中移动。现在视图矩阵看起来像这样的：

{% highlight glsl %}
glm::mat4 view;
//注意，我们将矩阵向我们要进行移动场景的反向移动。
view = glm::translate(view, glm::vec3(0.0f, 0.0f, -3.0f));
{% endhighlight %}


最后我们需要做的是定义一个投影矩阵。我们想要在我们的场景中使用透视投影所以我们声明的投影矩阵看起来像这样的：

{% highlight glsl %}
glm::mat4 projection;
projection = glm::perspective(45.0f, screenWidth / screenHeight, 0.1f, 100.0f);
{% endhighlight %}

<div class="orange_box">
	<p class="orange_title">注意</p>
	<p class="box_content">
  再重复一次，在glm指定角度的时候要注意。这里我们将参数fov设置为45度，但有些GLM的实现是将fov当成弧度，在这种情况你需要使用<code>glm::radians(45.0)</code>来设置。
	</p>
</div>

既然我们创建了转换矩阵，我们应该将它们传入着色器。首先，让我们在顶点着色器中声明一个单位转换矩阵然后将它乘以顶点坐标：

{% highlight glsl %}
#version 330 core
layout (location = 0) in vec3 position;
...
uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;

void main()
{
    //注意从右向左读
    gl_Position = projection * view * model * vec4(position, 1.0f);
    ...
}
{% endhighlight %}

我们应该将矩阵传入着色器（这通常在每次渲染的时候即转换矩阵将要改变的时候完成）：

{% highlight c++ %}
GLint modelLoc = glGetUniformLocation(ourShader.Program, "model"));
glUniformMatrix4fv(modelLoc, 1, GL_FALSE, glm::value_ptr(model));
... // 视图矩阵和投影矩阵与之类似
{% endhighlight %}

现在我们的顶点坐标通过模型，视图和投影矩阵来转换，最后的对象应该是：

 - 往后向地板倾斜。
 - 离我们有点距离。
 - 由透视展示（顶点越远，变得越小）。

让我们检查一下结果是否满足这些要求：

<img class="post_center_img" src="http://learnopengl.com/img/getting-started/coordinate_systems_result.png"/>

它看起来就像是一个三维的平面，是静止在一些虚构的地板上的。如果你不是得到相同的结果，请检查下完整的[源代码](http://learnopengl.com/code_viewer.php?code=getting-started/cube_vertices) 以及[顶点](http://learnopengl.com/code_viewer.php?code=getting-started/transform&type=vertex) 和[片段](http://learnopengl.com/code_viewer.php?code=getting-started/transform&type=fragment)着色器。

<br/>

## 更加立体

到目前为止，我们在二维平面甚至在三维空间中画图，所以让我们采取大胆的方式来将我们的二维平面扩展为三维立方体。要渲染一个立方体，我们一共需要36个顶点（6个面 \* 每个面有2个三角形组成 \* 每个三角形有3个顶点），这36个顶点的位置你可以[从这里获取](http://learnopengl.com/code_viewer.php?code=getting-started/cube_vertices)。注意，这一次我们省略了颜色值，因为这次我们只在乎顶点的位置和，我们使用纹理贴图。

为了好玩，我们将让立方体随着时间旋转：

{% highlight glsl %}
model = glm::rotate(model, (GLfloat)glfwGetTime() * 50.0f, glm::vec3(0.5f, 1.0f, 0.0f));
{% endhighlight %}

然后我们使用`glDrawArrays`来画立方体，这一次总共有36个顶点。

{% highlight c++ %}
glDrawArrays(GL_TRIANGLES, 0, 36);
{% endhighlight %}

如果一切顺利对的话绘制效果将于下面的类似：

<video class="post_center_img" width="90%" src="http://learnopengl.com/video/getting-started/coordinate_system_no_depth.mp4" controls="controls"></video>

这有点像一个立方体，但有些东西不是。立方体的某些面是通过其它面绘制的。之所以这样是因为OpenGL是通过画一个一个三角形来画你的立方体的，所以它将会覆盖之前已经画在那里的像素。因为这个原因，有些三角形会画在其它三角形上面，虽然它们本不应该是被覆盖的。

幸运的是，OpenGL存储的深度信息在缓冲区里面成为z缓冲区，它允许OpenGL决定何时覆盖一个像素何时不覆盖。通过使用z缓冲区我们可以设置OpenGL来进行深度测试。
<br/>

### z缓冲区


OpenGL存储它的所有深度信息于z缓冲区中，也被成为深度缓冲区。GLFW自动为你生成这样一个缓冲区 （就如它有一个颜色缓冲区来存储输出图像的颜色）。深度存储在每个片段里面（作为片段的z值）当片段像输出它的颜色时，OpenGL会将它的深度值和z缓冲进行比较然后如果当前的片段在其它片段之后它将会被丢弃，然后重写。这个过程称为深度测试且是由OpenGL自动完成的。

然而，如果我们想要确定OpenGL是否真的执行深度测试，首先我们要告诉OpenGL我们想要开启深度测试；而这通常是默认关闭的。我们通过`glEnable`函数来开启深度测试。`glEnable`和`glDisable`函数允许我们开启或关闭某一个OpenGL的功能。该功能会一直是开启或关闭的状态直到另一个调用来关闭或开启它。现在我们想开启深度测试就需要开启`GL_DEPTH_TEST`：

{% highlight c++ %}
glEnable(GL_DEPTH_TEST);
{% endhighlight %}

既然我们使用了深度测试我们也想要在每次重复渲染之前清除深度缓冲区（否则前一个片段的深度信息仍然保存在缓冲区中）。就像清除颜色缓冲区一样，我们可以通过在`glclear`函数中指定`DEPTH_BUFFER_BIT`位来清除深度缓冲区：

{% highlight c++ %}
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
{% endhighlight %}

接着运行你的程序，你将会看到一个开启了深度测试且各个面都是纹理的立方体在旋转。

<video class="post_center_img" width="90%" src="http://learnopengl.com/video/getting-started/coordinate_system_depth.mp4" controls="controls"></video>

检查代码请[点击](http://learnopengl.com/code_viewer.php?code=getting-started/coordinate_systems_with_depth)。

<br/>

### 更多立方体

现在我们想在屏幕上显示10个立方体的话。每个立方体看起来都是一样的，区别在于它们在世界的位置及旋转角度不同。立方体的顶点已经定义好了，所以我们不用改变我们的缓冲数组及属性数组当渲染更多对象的时候，我们唯一需要做的是改变每个对象的模型矩阵来将立方体转换到世界坐标系中。

首先，让我们为每个立方体定义一个转换向量来指定它在世界空间的位置。我们将要在`glm::vec3`数组中定义10个立方体位置向量。

{% highlight glsl %}
glm::vec3 cubePositions[] = {
  glm::vec3( 0.0f,  0.0f,  0.0f),
  glm::vec3( 2.0f,  5.0f, -15.0f),
  glm::vec3(-1.5f, -2.2f, -2.5f),  
  glm::vec3(-3.8f, -2.0f, -12.3f),  
  glm::vec3( 2.4f, -0.4f, -3.5f),  
  glm::vec3(-1.7f,  3.0f, -7.5f),  
  glm::vec3( 1.3f, -2.0f, -2.5f),  
  glm::vec3( 1.5f,  2.0f, -2.5f),
  glm::vec3( 1.5f,  0.2f, -1.5f),
  glm::vec3(-1.3f,  1.0f, -1.5f)  
};
{% endhighlight %}


现在，在循环中，我们调用`glDrawArrays`10次，在我们开始渲染之前每次传入一个不同的模型矩阵到顶点着色器中。我们将会创建一个小的循环来通过一个不同的模型矩阵重复渲染我们的对象10次。注意我们也传入了一个旋转参数到每个容器中：

{% highlight c++ %}
glBindVertexArray(VAO);
for(GLuint i = 0; i < 10; i++)
{
  glm::mat4 model;
  model = glm::translate(model, cubePositions[i]);
  GLfloat angle = 20.0f * i;
  model = glm::rotate(model, angle, glm::vec3(1.0f, 0.3f, 0.5f));
  glUniformMatrix4fv(modelLoc, 1, GL_FALSE, glm::value_ptr(model));

  glDrawArrays(GL_TRIANGLES, 0, 36);
}
glBindVertexArray(0);
{% endhighlight %}

这个代码将会每次都更新模型矩阵然后画出新的立方体，如此总共重复10次。然后我们应该就能看到一个填充着10个古怪的旋转立方体的世界。

<img class="post_center_img" src="http://learnopengl.com/img/getting-started/coordinate_systems_multiple_objects.png"/>

完美！这就像我们的容器找到了志同道合的小伙伴一样。如果你在这里卡住了，你可以检查一下[代码](http://learnopengl.com/code_viewer.php?code=getting-started/coordinate_systems_multiple_objects) 以及[顶点](http://learnopengl.com/code_viewer.php?code=getting-started/coordinate_systems&type=vertex) 和[片段](http://learnopengl.com/code_viewer.php?code=getting-started/coordinate_systems&type=fragment) 。

### 练习


 - 对GLM的投影函数中的FoV和aspect-ratio参数进行试验。看能否研究它们是如何影响透视平截头体的。
 - 将视图矩阵在各个方向上进行平移来看场景是如何改变的。把视图矩阵当成摄像机对象。
 - 只使用模型矩阵每次只让3个立方体旋转（包括第1个）而其它保持静止。[解决方案](http://learnopengl.com/code_viewer.php?code=getting-started/coordinate_systems-exercise3)。
