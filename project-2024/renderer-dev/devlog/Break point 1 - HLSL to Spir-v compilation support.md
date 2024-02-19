
# Installation & Building source

My renderer engine implementation is based on Vulkan-API currently.
However I wanted to support DX12 backend for my renderer engine after i learned it in the future.
After Then, i thought it would be better to use HLSL for shading language because DX12 uses HLSL as its native and i can compile HLSL to SPIR-V binary using '
DirectXShaderCompiler'.

By following [official wiki](https://github.com/Microsoft/DirectXShaderCompiler/wiki/SPIR%E2%80%90V-CodeGen) in  DirectXShaderCompiler project, i cloned the project to my local and tried to build by following the commands

```terminal
git clone https://github.com/microsoft/DirectXShaderCompiler.git
git submodule update --init # in DirectXShaderCompiler directory

# Run the following if Windows Driver Kit is not installed
python .\utils\hct\hctgettaef.py

.\utils\hct\hctstart.cmd <dxc-src-dir> <dxc-bin-dir> # Set up environment
.\utils\hct\hctbuild.cmd -spirv                      # Configure and build
```

Unfortunately, it was not that trivial to build with those commands.
There were a certain issues about unicode encoding ([compiler warning C4819](https://learn.microsoft.com/en-us/cpp/error-messages/compiler-warnings/compiler-warning-level-1-c4819?view=msvc-170))
This warning was regarded as error, so there is no way to proceed building project without solving this error first.

I only got the error on these two paths.
![](images/Pasted%20image%2020240219164535.png)
![](images/Pasted%20image%2020240219164524.png)

To figure out this error,  I should open the project with visual studio and save them with encoding that can represent the whole possible characters (**Unicode (UTF-8 with signature) - Codepage 65001**).

- In visual studio, Select the file to save as first, 
	Then, File -> Save `<filename>` As menu 
![](images/Pasted%20image%2020240219164719.png)

- In save tab, i needed to click dropdown menu and selected 'Save with Encoding'.
![](images/Pasted%20image%2020240219164737.png)
- Then , i change the previous encoding without signature to UTF-8 'with signature'.
	![](images/Pasted%20image%2020240219164949.png)


With these few modifications, i could successfully build the shader compiler!!
![](images/Pasted%20image%2020240219165110.png)

It tells me that the output binary was made in Debug ouput path as above.
My last step was to implement build pipeline for converting HLSL to SPIR-V and to check if it works with simple triangle drawing.


# Build commands in Cmakelists

I wrote down below cmake commands in my Cmakelists.txt
Although it would be much better to use reusable function about uncessary duplicated part, I didn't want to encounter any other errors that i don't know anymore.

Here the variable i set :
- DXC_PATH : path of directX shader compiler binary
- SHADER_MODEL : specifying a shader model i use for each shader type
- FILE_NAME : an output filename without extension.
- HLSL_SPIRV_BINARY_FILES : output list
```cmake
####################### HLSL Shader build #######################
# get all hlsl files in shaders directory
set(DXC_PATH "C:/VulkanDev/Murakano/thirdparty/hlsl.bin/Debug/bin") # YOU NEED TO CHANGE THIS TO YOUR DXC PATH
find_program(DXC_EXECUTABLE dxc PATHS ${DXC_PATH} NO_DEFAULT_PATH)
message(STATUS ${DXC_EXECUTABLE})

# Define shader model versions as variables
set(VERTEX_SHADER_MODEL vs_6_3)
set(FRAGMENT_SHADER_MODEL ps_6_3)

# Compile Vertex Shader
file(GLOB_RECURSE HLSL_VERTEX "${CMAKE_CURRENT_SOURCE_DIR}/shaders/HLSL/vertex.hlsl")
get_filename_component(FILE_NAME ${HLSL_VERTEX} NAME_WE)
set(HLSL_SPIRV_OUTPUT "${CMAKE_CURRENT_SOURCE_DIR}/shaders/output/spir-v/${FILE_NAME}.spv")
add_custom_command(
    OUTPUT  ${HLSL_SPIRV_OUTPUT}
    COMMAND ${DXC_EXECUTABLE} -spirv -T ${VERTEX_SHADER_MODEL} -E main ${HLSL_VERTEX} -Fo ${HLSL_SPIRV_OUTPUT}
    DEPENDS ${HLSL_VERTEX}
)
list(APPEND HLSL_SPIRV_BINARY_FILES ${HLSL_SPIRV_OUTPUT})

# Compile Fragment Shader
file(GLOB_RECURSE HLSL_FRAG "${CMAKE_CURRENT_SOURCE_DIR}/shaders/HLSL/fragment.hlsl")
get_filename_component(FILE_NAME ${HLSL_FRAG} NAME_WE)
set(HLSL_SPIRV_OUTPUT "${CMAKE_CURRENT_SOURCE_DIR}/shaders/output/spir-v/${FILE_NAME}.spv")
add_custom_command(
    OUTPUT  ${HLSL_SPIRV_OUTPUT}
    COMMAND ${DXC_EXECUTABLE} -spirv -T ${FRAGMENT_SHADER_MODEL} -E main ${HLSL_FRAG} -Fo ${HLSL_SPIRV_OUTPUT}
    DEPENDS ${HLSL_FRAG}
)
list(APPEND HLSL_SPIRV_BINARY_FILES ${HLSL_SPIRV_OUTPUT})
```

Before i was building shader binaries with GLSL , so i just added new target into `Shaders` dependencies

```cmake
# Custom target for Shaders
add_custom_target(
    Shaders
    DEPENDS ${HLSL_SPIRV_BINARY_FILES} ${SPIRV_BINARY_FILES}
)

add_dependencies(${CMAKE_PROJECT_NAME} Shaders) # add shader buld as a dependency of the main project
```


# Writing actual hlsl shader code.

To draw a simple triangle, i was using these vertex and fragment shader codes.
- vertex shader
```glsl
#version 450

vec2 positions[3] = vec2[](
	vec2(0.0, -0.5),
	vec2(0.5, 0.5),
	vec2(-0.5, 0.5)
);

vec3 colors[3] = vec3[](
    vec3(1.0, 0.0, 0.0),
    vec3(0.0, 1.0, 0.0),
    vec3(0.0, 0.0, 1.0)
);

layout(location = 0) out vec3 fragColor;

void main()
{
	gl_Position = vec4(positions[gl_VertexIndex], 0.0, 1.0);
	fragColor = colors[gl_VertexIndex];
}
```

- fragment shader
```glsl
#version 450

layout(location = 0) in vec3 inColor;

layout(location = 0) out vec4 outColor;

void main()
{
	outColor = vec4(inColor, 1.0);
}
```

Then By refering to GLSL mappings with HLSL, i could convert the glsl code into hlsl-style code.

- vertex.hlsl
```hlsl
struct VSInput
{
    uint VertexIndex : SV_VertexID;
};

struct VSOutput
{
    float4 position : SV_POSITION;
    float3 color : COLOR;
};

VSOutput main(VSInput input)
{
    float2 positions[3] = { float2(0.0, -0.5), float2(0.5, 0.5), float2(-0.5, 0.5) };
    float3 colors[3] = { float3(1.0, 0.0, 0.0), float3(0.0, 1.0, 0.0), float3(0.0, 0.0, 1.0) };

    VSOutput output;

    output.position = float4(positions[input.VertexIndex], 0.0, 1.0);
    output.color = colors[input.VertexIndex];

    return output;
}
```

- fragment.hlsl
```hlsl
struct PSInput
{
    float3 color : COLOR;
};

float4 main(PSInput input) : SV_Target
{
    return float4(input.color, 1.0);
}
```


# 4. The result

After i put a lot of efforts for this setup, i got this desirable result !
![](images/Pasted%20image%2020240219201753.png)




P.S) After compiled the project myself, i got to know that Vulkan SDK provides precompiled binary of DirectXShader compiler. 
For everyone who don't want to encounter unexpected error, i want to let you guys that there is a way to use this binary without compilation.