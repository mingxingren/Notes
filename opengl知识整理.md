## OpenGL Sharders（着色器）

### GPU显示数据流程

![image-01](https://raw.githubusercontent.com/mingxingren/Notes/master/resource/photo/image-2021123001.png)



### 顶点着色器的坐标系统

顶点着色器的坐标很有意思,  它使用 -1 和 1表示坐标系轴方向上的负边界和正边界.  仔细想想确实应该如此，假使我们想渲染的一块区域的大小是 10x10大小的矩形，那么边界就是 -5 ~ 5，当区域是20x20时，边界便改为 -10~10. 我猜如果以确定边界绘制左边代码便不具有普适性，这个矩形尺寸调整一下，那个矩形尺寸调整一下. 所以统一用 -1 和 1表示边界值, 中间值乘以系数表示其他坐标点，这样就不用受到渲染矩形区域大小的影响.

![image-01](https://github.com/mingxingren/Notes/raw/master/resource/photo/image-2021120501.png)



着色器（**Shader**）是运行在GPU上的小程序，这些小程序为图形渲染管线的某个特定部分而运行。使用一种叫 **GLSL** 的类C语言写成，一个典型的着色器有下面的结构

```glsl
#version version_number
in type in_variable_name;
in type in_variable_name;

out type out_variable_name;

uniform type uniform_name;

int main()
{
  // 处理输入并进行一些图形操作
  ...
  // 输出处理过的结果到输出变量
  out_variable_name = weird_stuff_we_processed;
}
```



### 着色器数据输入

CPU数据通过OpenGL缓冲区发送到GPU

![image-02](https://github.com/mingxingren/Notes/raw/master/resource/photo/image-2021123002.png)





## 双缓冲

应用程序使用单缓冲绘图时可能会存在图像闪烁的问题。 这是因为生成的图像不是一下子被绘制出来的，而是按照从左到右，由上而下逐像素地绘制而成的。最终图像不是在瞬间显示给用户，而是通过一步一步生成的，这会导致渲染的结果很不真实。为了规避这些问题，我们应用双缓冲渲染窗口应用程序。**前**缓冲保存着最终输出的图像，它会在屏幕上显示；而所有的的渲染指令都会在**后**缓冲上绘制。当所有的渲染指令执行完毕后，我们**交换**(Swap)前缓冲和后缓冲，这样图像就立即呈显出来，之前提到的不真实感就消除了。



## OpenGL 调用流程

1. 着色器创建流程

```flow
start=>start: Start
op_1=>operation: glCreateShader 创建着器句柄
op_2=>operation: glShaderSource 加载着色器脚本
op_3=>operation: glCompileShader 编译着色器脚本
op_4=>operation: glGetShaderiv 判断着色器编译结果是否成功
op_5=>operation: glGetShaderInfoLog 获取着色器信息日志
cond=>condition: 是否成功
end=>end
start->op_1->op_2->op_3->op_4->cond
cond(yes)->end
cond(no)->op_5->end
```

2. 创建 OpenGL 工程对象

```flow
st=>start: Start
op_1=>operation: glCreateProgram 创建工程对象
op_2=>operation: glAttachShader 加载顶点着色器,片段着色器
op_3=>operation: glLinkProgram 构建工程
op_4=>operation: glGetProgramiv 获取工程构建状态
op_5=>operation: glDeleteShader 删除着色器
op_6=>operation: glGetProgramInfoLog 获取工程日志
cond=>condition: 是否成功
e=>end
st->op_1->op_2->op_3->op_4->cond
cond(yes)->op_5->e
cond(no)->op_6->e
```

3. 申请顶点队列对象(VAO)、顶点缓冲对象(VBO)、索引缓冲对象(EBO)

   ```flow
   st=>start: Start
   op_1=>operation: glGenVertexArrays 申请顶点队列对象(VAO)
   op_2=>operation: glGetBuffer 创建VBO(顶点缓冲对象) EBO(索引缓冲对象)等缓冲对象
   op_3=>operation: glBindVertexArray 将当前执行对象绑定到VAO
   op_4=>operation: glBindData 将VBO绑定到VAO上,并将当前操作对象指向VBO
   op_5=>operation: glBufferData 对VBO EBO填入数据
   op_6=>operation: glVertexAttribPointer 将location id 绑定到当前的 VBO 缓存
   op_7=>operation: glDrawElements 根据 EBO 画出图形
   e=>end: End
   st->op_1->op_2->op_3->op_4->op_5->op_6->op_7->e
   ```

   4. 使用 texture(纹理) 流程
   
   ```flow
   st=>start: Start
   op_1=>operation: glGenTextures 申请 texture 对象
   op_2=>operation: glBindTexture 绑定当前需要操作的 texture
   op_3=>operation: glTexParameteri 设置 texture 属性?
   op_4=>operation: glTexImage2D 对 texture 填入数据
   op_5=>operation: glGenerateMipmap 生成 mipmap https://zh.wikipedia.org/wiki/Mipmap
   op_6=>operation: glUniform1i 特殊用法: 将 片段着色器中 texture变量 指定为对应的纹理单元
   op_7=>operation: glActiveTexture 激活纹理单元例如:GL_TEXTURE0, 并设置成当前操作的纹理单元
   op_8=>operation: glBindTexture 将我们申请的texture对象绑定到当前的纹理单元中
   e=>end: End
   st->op_1->op_2->op_3->op_4->op_5->op_6->op_7->op_8->e
   ```

片段着色器中的变量和我们程序申请的texture资源是通过纹理单元进行传值的. 其关系大致为：

```sequence
fs texture变量->texture unit: 绑定对应的纹理单元
程序 texture对象资源->texture unit: 程序申请的texture资源输入到纹理单元
texture unit->fs texture变量: 映射
```



## opengl 参考资料

https://antongerdelan.net/opengl/index.html#onlinetuts

