# 基于FFMPEG 和 DirectX11的流媒体视频

事情起因：因为工作需求，我需要使用ffmpeg进行硬解实时视频并显示到屏幕, 但是性能一直不太理想。偶然发现一篇英文博客与我的需求类似，所以我决定翻译这边英文博客，这是原文地址：https://medium.com/swlh/streaming-video-with-ffmpeg-and-directx-11-7395fcb372c4



在几个月之前，我被分配开发一个低延迟的视频播放器。在之前，我只接触过FFmpeg没接触过DirectX11。但是我认为它应该不难。FFmpeg是非常受欢迎的，DirectX11也出现了很长一段时间了，并且我应该不会用到3D图形或者更复杂的东西。

所以，应该会有一些能借鉴的解码和渲染视频例子，是吧？

但很不幸，没有…， 因此才写这篇文章。

所以下一个对FFmpeg和DirectX11不熟悉的人不必为了显示视频而伤脑筋。

首先，我们需要搞清楚一些重要的事情

① 样例代码需要足够简单。我省略返回码检测的，错误处理等，我的观点是代码例子就只是例子（我会提供更全面得例子，但是你知道，知识产权等等）

② 我不会介绍硬件加速视频解码/渲染原理，因为它有点超出了本文的范围。此外，还有很多其他资源比我能更好地解释它

③ ffmpeg 支持相当多地协议和编码格式。RTSP 和 UDP 的例子都有用它，h264和h265编码同样用到ffmpeg，我相信很多人参考它。

④ 我使用CMake创建此工程，使它不依赖Visual Studio’s 编译平台（因为我们同样需要支持不需要DirectX的渲染）。这让事情变得更加有难度，这就是我为什么提到它



## 第一步：设置源流和视频解码器

这是ffmpeg这块的，只是设置格式上下文，解码上下文，和所有ffmpeg需要的结构体。对于此处，我参考了[这个例子](https://ffmpeg.org/doxygen/3.4/hw__decode_8c_source.html)和[MoonLight](https://github.com/moonlight-stream/moonlight-qt/blob/master/app/streaming/video/ffmpeg.cpp)的源码。

注意你需要以某种方式向 AVCodecContext 提供硬件设备类型。我选择与ffmpeg相同的方式执行此操作。

```cpp
// initialize stream
const std::string hw_device_name = "d3d11va";
AVHWDeviceType device_type = av_hwdevice_find_type_by_name(hw_device_name.c_str());
// set up codec context
AVBufferRef* hw_device_ctx;
av_hwdevice_ctx_create(&hw_device_ctx, device_type, nullptr, nullptr, 0);
codec_ctx->hw_device_ctx = av_buffer_ref(hw_device_ctx);
// open stream
```

一旦设置完成，解码是相当简单的；它仅仅是从源流种获取AVPacket并且将它们解码成AVFrames这些事情。

```cpp
AVPacket* packet = av_packet_alloc();
av_read_frame(format_ctx, packet);
avcodec_send_packet(codec_ctx, packet);
AVFrame* frame = av_frame_alloc();
avcodec_receive_frame(codec_ctx, frame);
```

这些都是简化的，但组在一起也不需要太麻烦。虽然我还不能将它显示到屏幕上，我想验证是否生成有效的帧，所以我认为我只需将它们写入bitmap来验证。

这里有个小问题



## 第二步：转换 NV12 为 RGBA

创建一个 bitmap(并且当它转换成功后，使用DX11 swapchain呈现)，我需要RGBA格式的帧。然后这个解码器解出来的是 NV12格式，所以我使用 FFmpeg的[swscale](https://ffmpeg.org/doxygen/0.5/swscale-example_8c-source.html)将 AV_PIX_FMT_NV12 转换成 AV_PIX_FMT_RGBA。

设置 SwsContext就像调用单个函数一样简单

```cpp
SwsContext* conversion_ctx = sws_getContext(
        SRC_WIDTH, SRC_HEIGHT, AV_PIX_FMT_NV12,
        DST_WIDTH, DST_HEIGHT, AV_PIX_FMT_RGBA,
        SWS_BICUBLIN | SWS_BITEXACT, nullptr, nullptr, nullptr);
```

当然，为了使用sws_scale(), 我们需要将帧数据从GPU转换到CPU. 利用ffmpeg的av_hwframe_transfer_data() 。有很多这样的[例子](https://ffmpeg.org/doxygen/3.4/hw__decode_8c_source.html)

```cpp
// decode frame
AVFrame* sw_frame = av_frame_alloc();
av_hwframe_transfer_data(sw_frame, frame, 0);
sws_scale(conversion_ctx, sw_frame->data, sw_frame->linesize, 
          0, sw_frame->height, dst_data, dst_linesize);
sw_frame->data = dst_data
sw_frame->linesize = dst_linesize
sw_frame->pix_fmt = AV_PIX_FMT_RGBA
sw_frame->width = DST_WIDTH
sw_frame->height = DST_HEIGHT
```

这样能正常工作，但作为一个长期解决方案有两个问题.

1. 我想从 AVFrame 得到是一个简单字节数组；使用"d3d11va" 给我们帧数据并不是简单字节数组。因此我将硬件名称改成 "dxva". 通过这种方式，frame->data 就是 uint8_t* 形式的位图。目前它是有效的，但是作为一个长期解决方案，不使用 "d3d11va" 基本没抓住重点
2. 当调用 sws_scale() 将帧转换成RGBA，我们需要将帧从GPU移动到CPU。确实，现在是有效的，但是很明显我们将这步移除



所以无论如何都不是完美，但至少我们解出了帧，并以bitmap形式呈现到我们眼前。



## 第三步：设置 DirectX11 渲染

如果你还不知道，这是对你的忠告：DirectX11 是完全不同于 DirectX9，完全不同！

经过很多失败的尝试，我复制并粘贴了这个[示例](directxtutorial.com/Lesson.aspx?lessonid=11-4-5)，这样我可以开始写代码了。之后将三角形编程了正方形的异常复杂的任务。

此外，我没有在运行时编译着色器，而是选择在编译时编译它们。有那么一瞬间，我想我必须包含一个第三方库才能做到这点。但是它只需要在CMakeList.txt文件中的几行代码。找到 fxc.exe 可执行文件，并使用适当的选项执行命令来编译着色器。（我使用 /Fh 将它们编译成自动生成的头文件）



## 第四步：交换纹理颜色

当我得到一个彩虹方块，只需在定义的输入布局中为 TEXCOORD 切换成 COLOR即可。显然，这意味着要改变一些事情：

1. 顶点结构现在有 `XMFLOAT2` (*x, y*)  对应纹理坐标映射值为 `XMFLOAT4` (*r*, *g*, *b*, *a*)
2. 片段着色器需要从纹理中采样而不是我们直接提供颜色，这意味着我们需要一个 sampler
3. 另外，[纹理坐标系和位置坐标系是不同的](http://www.rastertek.com/dx11s2tut05.html)

当我能渲染一张基础的静态图时，我知道我接近了，剩下的就是将实际位图从帧传输到共享纹理。



## 第五步：渲染实际的帧

因为我们的帧仍然是简单的RGBA字节数组，并且我们的 ID3D11Texture2D 是 DXGI_FORMAT_R8G8B8A8_UNORM 的格式。使用 memcpy 就可以了，我们需要根据计算得出来的数组长度：`width_in_pixels * height_in_pixels * bytes_per_pixel`.

注意我们同样需要调用device context’s `Map()` 获取能访问纹理底层数据的权限

```cpp
// decode and convert frame
static constexpr int BYTES_IN_RGBA_PIXEL = 4;
D3D11_MAPPED_SUBRESOURCE ms;
device_context->Map(m_texture.Get(), 0, D3D11_MAP_WRITE_DISCARD, 0, &ms);
memcpy(ms.pData, frame->data[0], frame->width * frame->height * BYTES_IN_RGBA_PIXEL);
device_context->Unmap(m_texture.Get(), 0);
// clear the render target view, draw the indices, present the swapchain
```



## 第六步：渲染实际帧… 可能是正确的

从我的研究开始，我就知道使用 “d3d11va” 硬件加速解出来 AVFrame 可以使用 DirectX11 进行渲染。哪要怎么做呢？

我需要初始化 **d3d11va hardware device context.** 基本上，FFmpeg解码器需要了解它正在使用D3D11设备。

```cpp
AVBufferRef* hw_device_ctx = av_hwdevice_ctx_alloc(AV_HWDEVICE_TYPE_D3D11VA);
AVHWDeviceContext* device_ctx = reinterpret_cast<AVHWDeviceContext*>(hw_device_ctx->data);
AVD3D11VADeviceContext* d3d11va_device_ctx = reinterpret_cast<AVD3D11VADeviceContext*>(device_ctx->hwctx);
// m_device is our ComPtr<ID3D11Device>
d3d11va_device_ctx->device = m_device.Get();
// codec_ctx is a pointer to our FFmpeg AVCodecContext
codec_ctx->hw_device_ctx = av_buffer_ref(hw_device_ctx);
av_hwdevice_ctx_init(codec_ctx->hw_device_ctx);
```

所以现在，当我把解出来的帧传到渲染器时，不需要将他们传输到CPU上，并且也不要将它们转成RGBA。我们可以简单地这样做：

```cpp
ComPtr<ID3D11Texture2D> texture = (ID3D11Texture2D*)frame->data[0];
```

但是我们完成了么，并没有，还差很多。

我们需要将像素格式转成GPU。我们的交换链并没有神奇地开始渲染NV12帧，这意味着从NV12到RGBA转换需要在某个地方进行。现在，它将在GPU中进行，具体来说是在着色器中进行。这是合乎逻辑的；我们不仅是对纹理进行采样，因为纹理格式不是RGBA。为了让我们的着色器为每个像素返回正确的RGBA值，它需要根据纹理的YUV值来计算。

加入另一个着色器资源视图。当这个RGBA片段着色器将单个着色器资源视图作为输入，但是NV12片段着色器实际上需要两个：色度和亮度。因此，我们因此将一个纹理拆分成两个着色器资源视图。（在此之前，我不明白为什么DirectX需要区分纹理和着色器资源视图。天哪，我很高兴他们这样做了）

```cpp
// DXGI_FORMAT_R8_UNORM for NV12 luminance channel
D3D11_SHADER_RESOURCE_VIEW_DESC luminance_desc = CD3D11_SHADER_RESOURCE_VIEW_DESC(m_texture, D3D11_SRV_DIMENSION_TEXTURE2D, DXGI_FORMAT_R8_UNORM);
m_device->CreateShaderResourceView(m_texture, &luminance_desc,  &m_luminance_shader_resource_view); 
// DXGI_FORMAT_R8G8_UNORM for NV12 chrominance channel
D3D11_SHADER_RESOURCE_VIEW_DESC chrominance_desc = CD3D11_SHADER_RESOURCE_VIEW_DESC(texture,  D3D11_SRV_DIMENSION_TEXTURE2D, DXGI_FORMAT_R8G8_UNORM);
m_device->CreateShaderResourceView(m_texture, &chrominance_desc, &m_chrominance_shader_resource_view);
```

当然，我们还需要确保允许我们的片段着色器访问这些色度和亮度通道。

```cpp
m_device_context->PSSetShaderResources(0, 1, m_luminance_shader_resource_view.GetAddressOf());
m_device_context->PSSetShaderResources(1, 1, m_chrominance_shader_resource_view.GetAddressOf());
```

我们需要将纹理作为共享资源打开，这个ID3D11Texture2D 对象作为渲染器资源视图和FFmpeg 解出来帧的桥梁。我们将新解出来的帧复制到 ID3D11Texture2D 并且从中提取着色器资源视图。它是一种共享资源。

```cpp
ComPtr<IDXGIResource> dxgi_resource;
m_texture->QueryInterface(__uuidof(IDXGIResource), reinterpret_cast<void**>(dxgi_resource.GetAddressOf()));
dxgi_resource->GetSharedHandle(&m_shared_handle);
m_device->OpenSharedResource(m_shared_handle, __uuidof(ID3D11Texture2D), reinterpret_cast<void**>(m_texture.GetAddressOf()));
```

我们需要改变我们接收复制纹理的方式。很显然每次渲染一帧时创建新的着色器资源视图代价是高昂。而且 `memcpy` 不再是一个选择因为我们无法简单访问纹理数据。我认为通过调用像 `CopySubresourceRegion` DirectX  函数来复制解出来的帧到纹理资源

```cpp
ComPtr<ID3D11Texture2D> new_texture = (ID3D11Texture2D*)frame->data[0];
const int texture_index = frame->data[1];
m_device_context->CopySubresourceRegion(
        m_texture.Get(), 0, 0, 0, 0, 
        new_texture.Get(), texture_index, nullptr);
```

通过这些更改，我可以放心的告别那些 `av_hwframe_transfer_data()`和 `sws_scale()`这些函数，并且终于，向完全集成的FFmpeg-DirectX11视频播放器问好。



注: 翻译这篇博文是因为它有一个很好处理思路和完整的参考代码实例。但是可能并没有到极致，因为 ffmpeg 解码出来的纹理并不能直接用于管线，需要经过 `ID3D11DeviceContext::CopySubresourceRegion` 进行拷贝。我认为可以通过自定义 `AVD3D11VAFramesContext` 和 `AVHWFramesContext`，可以改变纹理属性从而直接用于管线。

