# Reflector

The `cientos` package provides an abstraction of the Reflector class, which creates a reflector material in real time of your scene `<Reflector >` component that makes a geometry as an argument to create a mirror. This Mesh extends from `Mesh` so all the default props can be passed as well:

## Usage

```html
<Reflector :rotation="[-Math.PI * 0.5, 0, 0]" :position-y="-2" color="#F7F7F7">
  <TresCircleGeometry :args="[10, 10]" />
</Reflector>
```

## Custom mirror effect

For more complex effect you can provide your own shaders, you could do this creating an object and pass the uniforms, vertexShaders or fragmentShaders:

```vue{3}
<script setup lang="ts" >
import vertexShader from "MyCustomvertexShader.glsl"

const customShader = {
  vertexShader
}
<script>
<template>
  <TresCanvas shadows alpha>
    <TresPerspectiveCamera :position="[0, 0, 3]" />
    ...
    <Reflector :rotation="[-Math.PI * 0.5, 0, 0]"
    :position-y="-2"
    color="#fff"
    :shader="customShader"
    >
      <TresCircleGeometry :args="[10, 10]" />
    </Reflector>
...
  </TresCanvas>
</template>
```

| Prop              | Description                                          | Default                   |
| :---------------- | :--------------------------------------------------- | ------------------------- |
| **color**         | The base color that's combine with the mirror effect | '#000000'                 |
| **textureWidth**  | the width of the texture to render on the mirror     | 512                       |
| **textureHeight** | the height of the texture to render on the mirror    | 512                       |
| **clipBias**      | to use the clipBias property                         | 0                         |
| **multisample**   | how many samplers will be render                     | 4                         |
| **shader**        | The texture of the smoke.                            | Reflector.ReflectorShader |


## The Reflector shader use the following configuration by default:

You can extend, modify or just play with them

### name

```js
	name:"ReflectorShader"
```

### Uniforms

```js
	uniforms: {
		color: {
			value: null
		},
		tDiffuse: {
			value: null
		},
		textureMatrix: {
			value: null
		}
	},
```

### VertexShader

```glsl
	vertexShader: /* glsl */`
		uniform mat4 textureMatrix;
		varying vec4 vUv;

		#include <common>
		#include <logdepthbuf_pars_vertex>

		void main() {

			vUv = textureMatrix * vec4( position, 1.0 );

			gl_Position = projectionMatrix * modelViewMatrix * vec4( position, 1.0 );

			#include <logdepthbuf_vertex>

		}`
```

### FragmentShader

```glsl
	fragmentShader: /* glsl */`
		uniform vec3 color;
		uniform sampler2D tDiffuse;
		varying vec4 vUv;

		#include <logdepthbuf_pars_fragment>

		float blendOverlay( float base, float blend ) {

			return( base < 0.5 ? ( 2.0 * base * blend ) : ( 1.0 - 2.0 * ( 1.0 - base ) * ( 1.0 - blend ) ) );

		}

		vec3 blendOverlay( vec3 base, vec3 blend ) {

			return vec3( blendOverlay( base.r, blend.r ), blendOverlay( base.g, blend.g ), blendOverlay( base.b, blend.b ) );

		}

		void main() {

			#include <logdepthbuf_fragment>

			vec4 base = texture2DProj( tDiffuse, vUv );
			gl_FragColor = vec4( blendOverlay( base.rgb, color ), 1.0 );

			#include <tonemapping_fragment>
			#include <colorspace_fragment>

		}`
```