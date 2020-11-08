<template>
  <Wrap :page="page">
    <article class="post lab">
      <div class="post-content">
				<h1 class="post-title">{{ page.title}}</h1>
        <slot name="default"></slot>
      </div>
    </article>

		<transition name="fade">
			<div class="gradient" v-show="gradientShow"></div>
		</transition>

		<!-- <div class="scene"></div> -->
  </Wrap>
</template>

<script type="x-shader/x-vertex" id="vertexShader">
varying vec2 vUv;

void main() {

  vUv = uv;
  gl_Position = projectionMatrix * modelViewMatrix * vec4( position, 1.0 );

}
</script>

<script type="x-shader/x-vertex" id="fragmentShader">
varying vec2 vUv;

void main() {

  // colour is RGBA: u, v, 0, 1
  gl_FragColor = vec4( vec3( vUv, 0. ), 1. );

}
</script>



<script>
import * as THREE from 'three'
import * as TWEEN from 'tween.js'

import Wrap from '../components/Wrap.vue'

export default {
  components: {
		Wrap,
  },

  props: ['page'],
	data() {
		return {
			cursorX: null,
			cursorY: null,
			gradientShow: false,
		}
	},
  computed: {
    feedLink() {
      return this.$feed && this.$feed.permalink
    }
  },

  methods: {
  },

	mounted() {

		document.onmousemove = (event) => {
			const x = event.clientX
			const y = event.clientY
			this.cursorX = x
			this.cursorY = y
		}

		this.gradientShow = true


		var scene = new THREE.Scene()
		// scene.background = new THREE.Color(0xeeeeee)

		var renderer = new THREE.WebGLRenderer();
		renderer.setSize( window.innerWidth, window.innerHeight );
		document.querySelector('.scene').appendChild( renderer.domElement );


		let camera = new THREE.PerspectiveCamera( 75, window.innerWidth/window.innerHeight, 0.1, 1000 )
		camera.position.x = 3;
		camera.position.z = 0;
		// camera.rotation.x = -0.1
		camera.rotation.y = -0.4
		camera.position.y = 5;




		// let light1Col = 0x00ff00
		let light1Col = 0x0c7ff9
		let light1 = new THREE.PointLight( light1Col, 1, 40 );
		light1.position.set( -17, 5, -4 );
		scene.add( light1 );

		// let light2Col = 0xff6600
		let light2Col = 0xf90794
		let light2 = new THREE.PointLight( light2Col, 1, 60 );
		light2.position.set( 17, 5, -6 );
		scene.add( light2 );

		
		let width = 40
		let height = 45
		let depth = 20

		var roomMaterial = new THREE.MeshPhysicalMaterial( {
			color: 0x3E38F2,
			side: THREE.DoubleSide,
			roughness: 0.8,
			reflectivity: 1
		} );

    //
		// let cubeCamera = new THREE.CubeCamera(1, 100, 256);
		// cubeCamera.position.set(0,1,10)
		// scene.add(cubeCamera);
    //
		// let material = new THREE.MeshPhysicalMaterial( {
		// 	envMap: cubeCamera.renderTarget.texture,
		// 	clearcoat: 1,
		// 	reflectivity: 1,
		// 	// roughness: 1:
		// } );
    //


		let groundGeometry = new THREE.PlaneGeometry( width, depth, 1 );
		let ground = new THREE.Mesh(groundGeometry, roomMaterial )
		// let ground = new THREE.Mesh(wallGeometry, material )
		ground.rotateX( - Math.PI / 2 );
		scene.add( ground );


		let wallGeometry = new THREE.PlaneGeometry( depth, height, 1 );
		let wall = new THREE.Mesh(wallGeometry, roomMaterial )
		// let ground = new THREE.Mesh(wallGeometry, material )
		wall.rotateY( - Math.PI / 2 );
		wall.position.x = 20
		wall.position.y = 5
		scene.add( wall );


		function vertexShader() {
			return `
				varying vec2 vUv;

				void main() {

					vUv = uv;
					gl_Position = projectionMatrix * modelViewMatrix * vec4( position, 1.0 );

				}
			`
		}

		function fragmentShader() { 
			return `
				varying vec2 vUv;

				void main() {

					gl_FragColor = vec4( vec3( vUv, 1.0 ), 1.0 );

				}
			`
		}


		var portalGeometry = new THREE.PlaneGeometry( width, height, 1 );
		let portalMaterial = new THREE.ShaderMaterial( {
			vertexShader: vertexShader(),
			fragmentShader: fragmentShader()
		} );
		// var portalMaterial = new THREE.MeshBasicMaterial( {color: 0xffffff, side: THREE.DoubleSide} );
		var portal = new THREE.Mesh( portalGeometry, portalMaterial );
		portal.position.z = -5
		portal.position.y = 5
		scene.add( portal );

		
		var animate = (frame) => {
			requestAnimationFrame( animate );
			TWEEN.update();
			renderer.render( scene, camera );

			const tiltCoords = { x: camera.rotation.x, y: camera.rotation.y }
			const tilt = new TWEEN.Tween(tiltCoords)
					.to({
						y: this.cursorX * -0.00008 + 0.1,
						x: this.cursorY * -0.00005 + 0.07
					}, 800)
					.easing(TWEEN.Easing.Quadratic.Out)
					.onUpdate(() => {
						camera.rotation.x = tiltCoords.x
						camera.rotation.y = tiltCoords.y
					})
					.start()


			// cubeCamera.update(renderer, scene)
		};

		// animate();
	}
}
</script>

<style lang="scss">
.lab {
	.post-content {
		ul {
			margin: 0;
			padding: 0;
			list-style: none;
			li {
				font-size: 18px;
			}
		}
	}
}

.gradient {
	position: fixed;
	width: 100%;
	height: 100%;
	//background-image: linear-gradient(to bottom right, rgba(0,0,0,0.4), transparent);
	background-image: linear-gradient(to top left, rgba(#111111,1), transparent);
	top: 0;
	z-index: -1;
}
.scene {
	position: fixed;
	width: 100%;
	height: 100%;
	backgroud: #efefef;
	z-index: -2;
	top: 0;
}

.fade-enter-active, .fade-leave-active {
  transition: opacity .7s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
</style>
