###Create a rotated 3D cube with WebGL

<img src="https://raw.githubusercontent.com/ultra7677/ADWEBLab2/master/image/pic3.jpg" width="400">  

This article records my learing experiences of WebGL, and I was totally new about both computer graphics and WebGL before.

WebGL enables web content to use an API based on OpenGL ES 2.0 to perform 3D rendering in an HTML canvas in browsers that support it without the use of plug-ins.  

In `index.html` we use those code to create a `canvas` with id "glcanvas".  
<pre><code>&lt;body onload="start()">
    &lt;canvas id="glcanvas" width="640" height="480">
      Your browser doesn't appear to support the HTML5 &lt;canvas&gt; element.
    &lt;/canvas>
  &lt;/body>
</code></pre>

##### Setting up WebGL context  
And `start()` function is used for setting up the WebGL context and start rendering content.    
`webgl-demo.js`
<pre><code>function start() {
  canvas = document.getElementById("glcanvas");
  gl = canvas.getContext("experimental-webgl");
  if (gl) {
    gl.clearColor(0.0, 0.0, 0.0, 1.0);  // Clear to black, fully opaque
    gl.clearDepth(1.0);                 // Clear everything
    gl.enable(gl.DEPTH_TEST);           // Enable depth testing
    gl.depthFunc(gl.LEQUAL);            // Near things obscure far things
    initShaders();                     
    initBuffers();    
    setInterval(drawScene, 15);
  }
}
</code></pre>

`gl` is the reference of initialized webgl context and we set the clear color to black, then we clean the context to that color.  
Function `initShaders` and `initBuffers` will be introduced latter.  

##### Initializing the shaders(着色器)    
Shaders take shape data and turn it into pixels on the screen. There are two types of shader in WebGL, vertex shader(顶点着色器) and fragment shader(片元着色器). 

Each pixel in a polygon is called a fragment in GL lingo. The fragment shader's job is to establish the color for each pixel, and vertex shader defines the position and shape of each vertex.

In our program, the two shaders are defined in `index.html`'s head.    
<pre><code>&lt;script id="shader-fs" type="x-shader/x-fragment"> 
      varying lowp vec4 vColor;
      void main(void) {
        gl_FragColor = vColor;
      }
&lt;/script>
</code></pre>
<pre><code>&lt;script id="shader-vs" type="x-shader/x-vertex">
	attribute vec3 aVertexPosition;
    attribute vec4 aVertexColor;
    uniform mat4 uMVMatrix;
    uniform mat4 uPMatrix;
    varying lowp vec4 vColor;
    void main(void) {
        gl_Position = uPMatrix * uMVMatrix * vec4(aVertexPosition, 1.0);
        vColor = aVertexColor;
    }
&lt;/script>
</code></pre>

And we use function `getShader()` to load these two shader programs by their id. The `getShader()` routine fetches a shader program with the specified name from the DOM, returning the compiled shader program to the caller. the implementation is also in `webgl-demo.js`. Since the whole code is too long so that I only paste key portion of it there.  
In `initShaders()`
<pre><code>  var fragmentShader = getShader(gl, "shader-fs");
  var vertexShader = getShader(gl, "shader-vs");
</code></pre>

Then we create the shader program by calling the WebGL object's `createProgram()` function, attach the two shaders to it, and link the shader program.  

<pre><code>// Create the shader program
  shaderProgram = gl.createProgram();
  gl.attachShader(shaderProgram, vertexShader);
  gl.attachShader(shaderProgram, fragmentShader);
  gl.linkProgram(shaderProgram);
</code></pre>

##### Creating the object  
Before we can render our cube, we need to create the buffer contains its vertices. We will do that using a function `initBuffers()`.  
<pre><code>// Create a buffer for the cube's vertices.
  cubeVerticesBuffer = gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER, cubeVerticesBuffer);
</code></pre>

Then, we create a array of vertices for that cube.(four vertices per side)  
<pre><code>var vertices = [
    // Front face
    -1.0, -1.0,  1.0,
     1.0, -1.0,  1.0,
     1.0,  1.0,  1.0,
    -1.0,  1.0,  1.0,
    // Back face
    -1.0, -1.0, -1.0,
    -1.0,  1.0, -1.0,
     1.0,  1.0, -1.0,
     1.0, -1.0, -1.0,
    // Top face
    -1.0,  1.0, -1.0,
    -1.0,  1.0,  1.0,
     1.0,  1.0,  1.0,
     1.0,  1.0, -1.0,
    // Bottom face
    -1.0, -1.0, -1.0,
     1.0, -1.0, -1.0,
     1.0, -1.0,  1.0,
    -1.0, -1.0,  1.0,
    // Right face
     1.0, -1.0, -1.0,
     1.0,  1.0, -1.0,
     1.0,  1.0,  1.0,
     1.0, -1.0,  1.0,
    // Left face
    -1.0, -1.0, -1.0,
    -1.0, -1.0,  1.0,
    -1.0,  1.0,  1.0,
    -1.0,  1.0, -1.0
  ];
  // Now pass the list of vertices into WebGL to build the shape. 
  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices), gl.STATIC_DRAW);
</code></pre>
<img src="https://raw.githubusercontent.com/ultra7677/ADWEBLab2/master/image/pic1.jpg" width="400">  
And we need to build an array of colors for each of 24 vertices. And we need to make sure that vertuces of each side has the same color and add these color data to buffer.  
<pre><code>  // Now set up the colors for the faces.
  var colors = [
    [0,  0,  1,  0.5],   
    [0,  1,  1,  0.5],   
    [1,  0,  1,  0.5],   
    [1,  1,  0,  0.5],   
    [0,  1,  0,  0.5],    
    [1,  1,  1,  0.5]     
  ]; 
  // Convert the array of colors into a table for all the vertices.
  var generatedColors = [];
  for (j=0; j<6; j++) {
    var c = colors[j];
    for (var i=0; i<4; i++) {
      generatedColors = generatedColors.concat(c);
    }
  } 
  cubeVerticesColorBuffer = gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER, cubeVerticesColorBuffer);
  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(generatedColors), gl.STATIC_DRAW);
</code></pre>
Then we build the element array buffer, this specifies the indices into the vertex array for each face's vertices.  
<pre><code>  cubeVerticesIndexBuffer = gl.createBuffer();
  gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, cubeVerticesIndexBuffer);
</code></pre>
This array defines each face as two triangles, using the indices into the vertex array to specify each triangle's position.  
<pre><code>var cubeVertexIndices = [
    0,  1,  2,      0,  2,  3,    // front
    4,  5,  6,      4,  6,  7,    // back
    8,  9,  10,     8,  10, 11,   // top
    12, 13, 14,     12, 14, 15,   // bottom
    16, 17, 18,     16, 18, 19,   // right
    20, 21, 22,     20, 22, 23    // left
  ]
  // Now send the element array to GL
  gl.bufferData(gl.ELEMENT_ARRAY_BUFFER,
  new Uint16Array(cubeVertexIndices), gl.STATIC_DRAW);
</code></pre>  
<img src="https://raw.githubusercontent.com/ultra7677/ADWEBLab2/master/image/pic2.jpg" width="400" height="500">  
##### Drawing the cube
The last part is about function `drawScene()`. We first draw the cube and then update the rotate of it.

First, we need to clear the canvas before we start drawing on it.  
`gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);`  

Second,we stablish the perspective with which we want to view the scene.  
`perspectiveMatrix = makePerspective(45, 640.0/480.0, 0.1, 100.0);`

Then we set the drawing position to the "identity" point, which is the center of the scene.  
`loadIdentity();`  

Now move the drawing position a bit to where we want to start drawing the cube.  
`mvTranslate([0.0, 0.0, -7.0]);`

And this is the key part of drawing.  
First we save the current matrix, then rotate before we draw.
and the cube is rotate around X-axis and Y-axis.  
<pre><code>  mvPushMatrix();
  mvRotate(cubeRotation, [1, 1, 0]);
</code></pre>
Then we draw the cube by binding different buffer, setting those attributes, and pushing it to GL. And use `drawElemnts()` to compose the cube by corresponding triangles.   
<pre><code>  gl.bindBuffer(gl.ARRAY_BUFFER, cubeVerticesBuffer);
  gl.vertexAttribPointer(vertexPositionAttribute, 3, gl.FLOAT, false, 0, 0);
  gl.bindBuffer(gl.ARRAY_BUFFER, cubeVerticesColorBuffer);
  gl.vertexAttribPointer(vertexColorAttribute, 4, gl.FLOAT, false, 0, 0);
  gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, cubeVerticesIndexBuffer);
  setMatrixUniforms();
  // Draw the cube.
  gl.drawElements(gl.TRIANGLES, 36, gl.UNSIGNED_SHORT, 0);
  // Restore the original matrix
  mvPopMatrix();  
</code></pre>

Finally, because we draw the cube every 15 milliseconds, we need to update the rotation for the next draw.
<pre><code>  var currentTime = (new Date).getTime();
  if (lastCubeUpdateTime) {
    var delta = currentTime - lastCubeUpdateTime;
    cubeRotation += (180 * delta) / 1000.0;
  }
  lastCubeUpdateTime = currentTime;
}
</code></pre>

And most of the matrix operation function is implemente by the demo code. It is in the finally part of `webgl-demo.js`. 

---
References:  
[WebGL Fundamentals](http://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html)  
[Creating 3D objects using WebGL](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API/Tutorial/Creating_3D_objects_using_WebGL)  
