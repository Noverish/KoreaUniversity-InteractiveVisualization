
# lecture 07

# I. Shader

## 1. Vertex Shader
- Transform vertices from object space to clip space.
  - Conventionally modelview followed by projection
  - Can define custom transformation to clip space
- Compute other data that are interpolated with vertices.
  - Color, Normals, Texture coordinates, ...

## 2. Fragment Shader
- Compute the color of a fragment (i.e. a pixel).
- Take interpolated data from vertex shaders.
- Can read more data from Textures, or User specified values

## 3. Tessellation
- Perform adaptive subdivision based on a variety of criteria (size, curvature, etc.)
- You can provide coarser models, but have finer ones displayed

## 4. Geometry Shader
- Geometry shader invocations take a single Primitive as input and may output zero or more primitives.
  - Triangles, lines, points, etc.
- A geometry shader is optional and does not have to be used.
- Usually be used to make surface details

## 5. Shader를 사용하는 이유
- Realistic materials, mapping & lighting
- Advanced rendering effects
  - Raytracing, NPR, Global Illuminations,...
- Animation effects
  - Natural phenomena, particle systems,...
- Image processing
  - Filtering, anti-aliasing, matting,...

# II. Shader Programming

## 1. Vertex Processing
- One vertex for Input/Output
- Each vertex is transformed into "screen space" independently.
- Programmable stage.
- S_v0 = M_p * M_vm * v_0

## 2. Primitive Processing

## 3. Rasterization
- Converting vector format into pixel format

## 4. Fragment Processing
- Generated by each primitive
- Input : rasterization of each primitive
- Output : each fragment (pixel) color
- Programmable stage

# III. Shader Programming Start
#### Application
```C++
GLuint program, vertShader, fragShader = 0;

void main(int argc, char *argv[]) {
    /*Create Window*/
    initGL();
    while(1){
        display();
        /*Event Handle*/
    }
}

void initGL() {
    glewInit(); //glew Initialize Function;
    createProgram(); //Create Shader Program
}

void createProgram() {
    char* vert = readShader("Vertex.glsl");
    char* frag = readShader("Fragment.glsl");
    vertShader = createShader(vert, GL_FRAGMENT_SHADER);
    fragShader = createShader(frag, GL_FRAGMENT_SHADER);
    GLuint p = glCreateProgram();
    glAttachShader(p, vertShader);
    glAttachShader(p, fragShader);
    glLinkProgram(p);
    program = p;
}

char* readShader(char *filename) {
    char *buffer = NULL;
    int string_size, read_size;
    FILE *handler = fopen(filename, "r");
    if (handler) {
        fseek(handler, 0, SEEK_END); // Seek the last byte of the file
        string_size = ftell(handler); // filesize
        rewind(handler); // go back to the start of the file
        // Allocate a string that can hold it all
        buffer = (char*) malloc(sizeof(char) * (string_size + 1) );
        // Read it all in one operation
        read_size = fread(buffer, sizeof(char), string_size, handler); 
        // fread doesn't set it so put a \0 in the last position
        buffer[string_size] = '\0';
        if (string_size != read_size){
            // Something went wrong, throw away the memory and set NULL 
            free(buffer);
            buffer = NULL;
        }
        fclose(handler); 
    }
    return buffer; 
}

GLuint createShader(char* src, GLenum type) {
    GLuint shader;
    shader = glCreateShader(type);
    glShaderSource(shader, 1, &src, NULL);
    glCompileShader(shader);
    return shader;
}

void display() {
    glClearColor(0.0f, 0.0f, 0.0f, 1.0f);
    glClear(GL_COLOR_BUFFER_BIT|GL_DEPTH_BUFFER_BIT);
    glUseProgram(program);

    /*Draw Call*/

    glUseProgram(0);
    glXSwapBuffers(dpy, win);
}
```

#### Vertex Shader
```GLSL
#version 130

void main() {
    gl_Position = gl_ModelViewMatrix*gl_Vertex;
    gl_FrontColor = gl_Color;
}
```

#### Fragment Shader
```GLSL
#version 130

void main() {
    gl_FragColor = gl_Color;
}
```

## 1. Input and Output of Vertex Shader
### Input
- Per vertex attributes
  - gl_Vertex , gl_Color , gl_Normal
- OpenGL states and user defined uniform variable such as gl_ModelViewMatrix
### Output
- gl_Position : coordinates in the canonical space
- Other values to be interpolated for each fragment during the rasterization
- gl_FrontColor

## 2. Input and Output of Fragment Shader
### Input
- Interpolated values from the rasterizer
- OpenGL states and user defined uniform variables
### Output
- Pixel values to be processed by pixel tests and written to the framebuffer
  - gl_FragColor, glFragDepth

## 3. Striped Cube
#### Vertex Shader
```GLSL
#version 130
varying vec3 pos;
void main() {
    gl_Position = gl_ModelViewMatrix * gl_Vertex;
    gl_FrontColor = gl_Color;
    pos = gl_Vertex.xyz;
}
```
#### Fragment Shader
```GLSL
#version 130
varying vec3 pos;
void main() {
    if(cos(pos.x * 40.0f) > 0.0f && cos(pos.y * 40.0f) > 0.0f)
        gl_FragColor = gl_Color;
    else
        gl_FragColor = gl_Color * 0.3f;
}
```

# IV. GLSL 문법

## 1. GLSL Built-In Data Types
### Scalar types
- bool, int, float
- No implicit conversion between types
- uint and double are supported in GLSL1.3 and 4.0, respectively.
### Vector
- Float: vec2, vec3, vec4
- Int: ivec, Bool: bvec Also (uvec, dvec)
- C++ style constructors
  - `Vec3 a = vec3(1.0, 2.0, 3.0);`
  - `Vec2 b = vec2(a);`
### Matrices
- mat2,mat3,mat4 (dmat2,dmat3,dmat4,matnxm,dmatnxm)
- Stored by columns order (OpenGL convention)
  - `mat2 m = mat2(1.0, 2.0, 3.0, 4.0);`

## 2. GLSL Sampler Types
- Sampler types are used to represent textures, which may be used in both vertex and fragment shaders
  - sampler1D, sampler2D, sampler3D
  - sampler1DShadow, sampler2DShadow
    - depth buffer with comparison
  - samplerCube

## 3. Arrays and Structs
- There are no pointers in GLSL
### Array
- Only one-dimensional array is allowed.
  - `float x[4];`
  - `vec3 colors[4]; colors[0] = vec3(1.0, 1.0, 0.0);`
  - `mat4 matrices[3];`
- Arrays know the number of elements they contain
  - `array.length();`
### Struct
```GLSL
struct light {
    vec4 position;
    vec3 color; 
};
```

## 4. GLSL Operators
- Matrix multiplication is done with * operator
```GLSL
mat4 m4, n4;
vec4 v4;
m4 * v4; // a vec4
v4 * m4; // a vec4
m4 * n4; // a mat4
```
## 5. Selection and Swizzling
- Can refer to vector or matrix elements by using [] operator or selection(.) operator with
  - x, y, z, w
  - r, g, b, a
  - s, t, p, q
  - x[2], x.b, x.z, x.p are the same
- Swizzling operator lets us manipulate components in the following way:
```GLSL
vec4 a;
a.yz = vec2(1.0, 2.0);
a.zw = vec2(2.0, 4.0);
```

## 6. Qualifiers

### const
- a variable whose value cannot be changed.

### varying
- interpolation 하기 위해 vertex shader에서 fragment shader로 값 전달
- Writable in vertex shader, Read-only in fragment shader
- GLSL 1.3에서 deprecated 되었으므로 'in'과 'out'을 사용

### attribute
- vertex 마다 값을 바꿀 수 있는 Global variable
- vertex shader에서만 사용 가능, Read-Only
- application에서 vertex shader에 값을 전달
- 수가 한정되어 있음
- GLSL 1.3에서 deprecated 되었으므로 'in'을 사용

### uniform
- primitive 또는 object 마다 값을 덜 바꾸는 Global variable
- vertex, fragment shader에서 사용 가능, Read-Only
- application에서 shader에 값을 전달
- glBegin, glEnd 블럭 안에서 저의 되나 값을 쓸 수 없음
- 수가 한정되어 있으나 attribute 보다는 많음

## 7. Functions
- No recursive function call
- Parameter passing is call by value-return

### Parameter Qualifiers (used for function calls)
- in, out, inout