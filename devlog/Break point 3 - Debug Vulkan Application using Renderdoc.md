
 While i've been developing 'Murakano' project, i faced numerous problems.
Most of them were related to application side issues such as wrong specifications for struct. Thererfore, i usually resolved those issues with visual studio debugger by tracking local memory visualizer.

 But a few days ago, i got a problem related to rendering pipeline, so i couldn't see anything in the rendered scene except for background color.

At the first time, i tried to solve the issue in a same manner that i've done so far. However it was not helpful because the buffers are only being stored in GPU device local memory.
After then, i realized this is the right moment to use graphics debugging tools like Renderdoc.
\
In this post, I'm gonna show you how i resolved the problem by using Renderdoc with visualization.

# 1. Pre-Requisites

Before you debug Vulkan application with renderdoc, you need to check few things in advance.

First, Check if there are renderdoc layer in your system's registry. 
You can ensure that you have it by following below instructions.

1. open your system's registry editor and go to a path 'your_pc/HKEY_LOCAL_MACHINE/SOFTWARE/Khronos/Vulkan/ImplicitLayers'

2. If you can find a registry named  'renderdoc.json', everything is fine!![](images/Pasted%20image%2020240715182536.png)

Second, Check if you've ever enabled some extensions that are not supported by renderdoc.
Renderdoc doesn't support hardware ray tracing extensions (acceleration structure features, ray tracing pipeline property and so on.). You need to ensure that you don't have any of them before you launch your application in renderdoc.
- When i forgot to remove the extensions, i got this message in the runtime. ![](images/스크린샷%202024-07-15%20140049.png)

Finally, A few of security programs in your system may block 'process injection' in renderdoc. If renderdoc complains about 'process injection failed', you need to inspect if your system activated any security process that blocks the behavior.


# 2. Let's debug

The problem that i had was related to rendering pipeline. Hence, i started to debug from 'vkCmdDrawIndexed' function call.

If you look into 'Event Browser' tab, you can see the function call lists from capture start to capture end.
When you click a draw command call in your browser tab, you can inspect any pipeline states related to the call.
![](images/스크린샷%202024-07-15%20143710.png)

In 'Vertex Input' stage, i saw there are correct inputs from application in 'Attributes tab'.
Every attributes of input vertex was perfect as i expected.
![](images/스크린샷%202024-07-15%20162612.png)

Also, i could inspect the contents of each buffer in 'Buffers' tab![](images/스크린샷%202024-07-15%20162617.png)

By clicking any of buffers, i could see which values each elements of the buffer has.![](images/스크린샷%202024-07-15%20162625.png)
In this tab, Finally i got to know which part of rendering pipeline caused the problem.
As you can see, All of indices has same value while i tried to copy a set of indices that have different values into index buffer.
At the same time, my vertex buffer had unexpected values, which are ridiculously small.
![](images/스크린샷%202024-07-15%20162633.png)

After reading application source code for hours, i could find there were a typo when i copy values from staging buffer to index buffer.

```cpp
// vertex buffer creation
//..
//..
CopyBufferToBuffer(stagingBuffer/*src*/, _vkVertexBuffer/*dst*/, bufferSize);


// index buffer creation
//..
//..
CopyBufferToBuffer(stagingBuffer/*src*/, _vkVertexBuffer/*dst*/, bufferSize);
```

After i copied a staged vertices data into `_vkVertexBuffer`, i copied a staged indiced data into `_vkVertexBuffer` again.
This made `_vkIndexBuffer` uninitialized and `_vkVertexBuffer` have indices values which have entirely different memory layout.

So, i modified a second `CopyBufferToBuffer` function call as below :
```cpp
CopyBufferToBuffer(stagingBuffer/*src*/, _vkIndexBuffer/*dst*/, bufferSize);
```


After i modified the function call, i launched the application in renderdoc again.

Finally, i could see the correct values in index buffer.
![](images/Pasted%20image%2020240715185821.png)

Also, The mesh viewer showed correct model as i expected to see
![](images/Pasted%20image%2020240715185914.png)

Finally, i solved the problem and could see the correct scene again!
![](images/Pasted%20image%2020240715190015.png)

# 3. Conclusion

This was the first time that i used graphics debugging tool to resolve issue in project.
I realized that the tool is so powerful that it can solve the problem in a sophiscated manner(not relying on heuristics).
