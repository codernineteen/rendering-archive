
- When you specify `VkDescriptorSetLayout`s while creating an instance of `VkPipelineLayout`, you may create an array of descriptor set layouts and pass its pointer.
  The thing that you should remember is that the order of elements exactly corresponds to the order of set index.
  For example,
```cpp
  std::vector<VkDescriptorSetLayout> descLayouts{set0, set1};

  pipeLayout.setCount = static_cast<uint32_t>(descLayouts.size());
  pipeLayout.pSetLayouts = &descLayouts;
```