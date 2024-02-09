
- a class which has a member of an object of window class
```cpp
class MKRenderer
{
public:
	MKRenderer();
	~MKRenderer();
	void Render();

private:
	MKWindow window;
};

```

- a window class
```cpp
class MKWindow
{
public:
	MKWindow() = default;
	MKWindow(bool isResizable);
	~MKWindow();
	inline void PollEvents() {	glfwPollEvents(); }
	inline bool ShouldClose() { return glfwWindowShouldClose(window); }

private:
	static void framebufferResizeCallback(GLFWwindow* window, int width, int height);

private:
	GLFWwindow* window;
	bool framebufferResized = false;
};
```

- constructor with assignment
	1. initialize 'MKWindow' object
	2. assign the object into 'window' memeber
	3. MKWindow object is destructed when it goes out of constructor scope.
```cpp
MKRenderer::MKRenderer() { 
	window = MKWindow(false);
}
```
- constructor with initialzier list
	1. Initialize 'window' member variable by calling its constructor.
	Hence, the member 'window' is never destructed before the renderer object is destructed.
```cpp
MKRenderer::MKRenderer() : window(false) {}
```
