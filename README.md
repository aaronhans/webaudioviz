web audio, waveforms, webgl tutorial
===========

![alt tag](https://raw.github.com/aaronhans/webaudioviz/master/examples/images/pattern.png)

# Access files via ajax

In order to load the audio file into the webpage you need to run some kind of server locally to serve the static files or launch your browser with security disabled

### Launch browser with security disabled

Launch Chrome with security disabled on Mac:

```
open -a Google\ Chrome --args --disable-web-security
```

This will allow you to just open an html file locally with the file open command and make ajax requests successfully. To do this on other OS see [stack overflow disable web security](http://stackoverflow.com/questions/3102819/disable-same-origin-policy-in-chrome)

### Or run local server

Install http://nodejs.org

Checkout this repository, from the root directory run

```
npm install
```

Then launch the server with 

```
npm start
```

Then you can browse everything in the examples directory at http://localhost:1337/ like http://localhost:1337/images/pattern.png

# Load Audio

This is a vanilla ajax request with responseType is set to arraybuffer and when data is received the global AudioContext 

```
function loadSound(url, callback) {
  var request = new XMLHttpRequest();
  request.open('GET', url, true);
  request.responseType = 'arraybuffer';

  request.onload = function() {
    context.decodeAudioData(request.response, function(buffer) {
      callback(buffer);
    }, onError);
  }
  request.send();

  function onError() {
  	console.log('error')
  }
}
```

# Play Audio

Create a web audio analyser node and use this to process the audio signal which will give us access to frequency data described in the [MDN Analyser Node docs](https://developer.mozilla.org/en-US/docs/Web/API/AnalyserNode)

```
function playSound(buffer) {
  source = context.createBufferSource(); // creates a sound source
  source.buffer = buffer;                // tell the source which sound to play
  source.connect(context.destination);   // connect the source to the context's destination (the speakers)

	analyser = context.createAnalyser();
	source.connect(analyser);
	analyser.connect(context.destination);

  source.start(0);
}
```

# Three.js setup

The following are the basic requirements for a three.js to begin drawing

```
    var scene, camera, renderer, line;
    var geometry, material, mesh;

    function init() {

        scene = new THREE.Scene();

        camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 1, 10000 );
        camera.position.z = 1000;

        renderer = new THREE.WebGLRenderer();
        renderer.setSize( window.innerWidth, window.innerHeight );

        document.body.appendChild( renderer.domElement );

    }
```


# Animation

Three.js includes the request animation frame shim so you can access the browsers native loop function tied to the screen refresh rate.

```
    function animate() {

      requestAnimationFrame( animate );
      newLine();

      renderer.render( scene, camera );

    }
```

# Waveform

Draw a line representing the audio volume over the full frequency range

```
    function newLine() {

    	scene.remove(line);

			var material = new THREE.LineBasicMaterial({
				color: 0x0000ff
			});

			var geometry = new THREE.Geometry();

			array = new Uint8Array(analyser.frequencyBinCount);
			analyser.getByteFrequencyData(array)

			var Ypoints = array;
			var xPoint = -2048;
			for(var i=0;i<Ypoints.length;i++) {
				geometry.vertices.push( new THREE.Vector3( xPoint, Ypoints[i], 0 ) );
				xPoint = xPoint + 4;
			}

			line = new THREE.Line( geometry, material );
			scene.add( line );
    }
```

Add calls to init and animate to your volume load function to begin drawing lines once you have audio data

```
  init();
  animate();
```

This should give you an animated view of the signal strength over the full spectrum seen in [freq.html](https://github.com/aaronhans/webaudioviz/blob/master/examples/freq.html)

![alt tag](https://raw.github.com/aaronhans/webaudioviz/master/examples/images/freq.png)

We can hear between 20 and 20,000 Hz. The web audio api provides a sample rate of 44100 Hz. You can define the fftSize which affects the number of slices of spectrum you receive in frequencyBinCount. The default fftSize is 1024 so it is slicing the spectrum into approximately 43Hz chunks. Here is a chart identifying the frequencies [corresponding to audible octaves](http://en.wikipedia.org/wiki/Audio_frequency).


Audio file used is: [Paper Planes by Virtual riot](https://soundcloud.com/virtual-riot/liquid-dnb-fun-test)

# Particle Spheres

Using a predefined geometry in three.js allows for quick particle placement. This example uses a particle sphere and also includes the mouse movement detection orbit controls three.js addon which allows you to move the camera around the page

```
var geometry = new THREE.SphereGeometry( 200, 22, 22 );
var particleMaterial = new THREE.ParticleSystemMaterial({color: 0xff0000});
```

[sphere.html](https://github.com/aaronhans/webaudioviz/blob/master/examples/sphere.html)

![alt tag](https://raw.github.com/aaronhans/webaudioviz/master/examples/images/sphereinside.png)

Using a predefined shape allows you to easily create, rotate or scale the entire geometry but will limit you if you want to rearrange that specific set of particles into a different shape. If that is what you are after start with a basic geometry and place the particles yourself: [particles.html](https://github.com/aaronhans/webaudioviz/blob/master/examples/particles.html)

# Shaders

These examples begin to use shaders which opens up a crazy world of pure GPU powered graphics written in OpenGL Shading Language (GLSL). See [shadertoy.com](http://www.shadertoy.com). 

A vertex shader which responsible for particle placement

```
<script type="x-shader/x-vertex" id="vertexshader">
uniform float time;
attribute float customFrequency;
attribute vec3 customColor;
varying vec3 vColor;
void main() 
{
	vColor = customColor; // set color associated to vertex; use later in fragment shader
	vec4 mvPosition = modelViewMatrix * vec4( position, 1.0 );

	gl_PointSize = (1.0 + sin( customFrequency * time )) * 8.0 * ( 300.0 / length( mvPosition.xyz ) );
  gl_Position = projectionMatrix * mvPosition;
}
</script>
```

And a fragment shader which must define the color

```
<script type="x-shader/x-fragment" id="fragmentshader">
uniform sampler2D texture;
varying vec3 vColor; // colors associated to vertices; assigned by vertex shader
void main() 
{
	gl_FragColor = vec4( vColor, 1.0 );
	gl_FragColor = gl_FragColor * texture2D( texture, gl_PointCoord );
}
</script>
```

Defining a shader material accessing and defining these GLSL properties in your javascript:

```
var customAttributes = 
{
	customColor:	 { type: "c", value: [] },
}

var sphereColor = Math.random();
for( var v = 0; v < geometry.vertices.length; v++ ) 
{
	customAttributes.customColor.value[ v ] = new THREE.Color( 0xffffff * sphereColor );
}

var shaderMaterial = new THREE.ShaderMaterial( 
{
	uniforms: {
		texture: { type: "t", value: discTexture } 
	},
	attributes:		customAttributes,
	vertexShader:   document.getElementById( 'vertexshader' ).textContent,
	fragmentShader: document.getElementById( 'fragmentshader' ).textContent,
	transparent: true, alphaTest: 0.5
});
```

Apply the shader material to your geometry:

```
particleSphere = new THREE.ParticleSystem( geometry, shaderMaterial );
```

# Respond to music

The picture at the top of this document is a screenshot of the final demo which is a few particle spheres with their color changing in response to any major sound file changes [gears.html](https://github.com/aaronhans/webaudioviz/blob/master/examples/gears.html)


