<template>
	<Wrap :page="page">
		<div class="blog">
			<a :href="feedLink" v-if="feedLink" class="feed">RSS</a>
			<ul class="post-list" v-if="page.posts && page.posts.length > 0">
				<li v-for="post in page.posts" :key="post.permalink">
					<h4 class="post-date">{{ formatDate(post.createdAt) }}</h4>
						<h3>
							<saber-link class="post-link" :to="post.permalink">{{ post.title }}</saber-link>
						</h3>
					</li>
				</ul>

		</div>

		<div class="menu">
			<ul>
				<li v-for="(navItem, index) in $themeConfig.nav" :key="index">
					<saber-link :to="navItem.link">{{ navItem.text }}</saber-link>
				</li>
			</ul>
		</div>

		<div class="scene"></div>
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

import { EffectComposer } from '../../public/three.js/examples/jsm/postprocessing/EffectComposer.js';
import { RenderPass } from '../../public/three.js/examples/jsm/postprocessing/RenderPass.js';
import { FilmPass } from '../../public/three.js/examples/jsm/postprocessing/FilmPass.js';
import { BokehPass } from '../../public/three.js/examples/jsm/postprocessing/BokehPass.js';
import { AfterimagePass } from '../../public/three.js/examples/jsm/postprocessing/AfterimagePass.js';



import Wrap from '../components/Wrap.vue'
import formatDate from '../utils/formatDate'

export default {
  components: {
		Wrap,
  },

  props: ['page'],
	data() {
		return {
			cursorX: null,
			cursorY: null,
		}
	},
  computed: {
    feedLink() {
      return this.$feed && this.$feed.permalink
    }
  },

  methods: {
    formatDate,
  },

	mounted() {

		document.onmousemove = (event) => {
			const x = event.clientX
			const y = event.clientY
			this.cursorX = x
			this.cursorY = y
		}


		var scene = new THREE.Scene()
		scene.background = new THREE.Color(0x2E112D)
		scene.fog = new THREE.Fog(0x2E112D, 0, 9);

		var renderer = new THREE.WebGLRenderer();
		renderer.setSize( window.innerWidth, window.innerHeight );
		document.querySelector('.scene').appendChild( renderer.domElement );



		let camera = new THREE.PerspectiveCamera( 75, window.innerWidth/window.innerHeight, 0.1, 1000 )
		camera.position.x = -2
		// camera.position.z = 5;
		// camera.rotation.x = -0.1
		camera.rotation.y = -0.3
		camera.position.y = 4;


		const composer = new EffectComposer( renderer );
		composer.addPass( new RenderPass( scene, camera ) );

		let afterimagePass = new AfterimagePass();
		afterimagePass.uniforms[ "damp" ].value = 0.98
		composer.addPass( afterimagePass );

		const bokehPass = new BokehPass( scene, camera, {
					focus: 1.0,
					aperture: 0.5,
					maxblur: 0.1,

					width: 10,
					height: 10
				} );
		// composer.addPass(bokehPass)
		// afterimagePass.uniforms[ "damp" ].value = 0.99

		const filmPass = new FilmPass(
			0.4,   // noise intensity
			0,  // scanline intensity
			0,    // scanline count
			false,  // grayscale
		);
		filmPass.renderToScreen = true;
		composer.addPass(filmPass);




		// let light1Col = 0x00ff00
		let light1Col = 0x540032
		let light1 = new THREE.PointLight( light1Col, 1, 40 );
		light1.position.set( -10, 10, 8 );
		scene.add( light1 );

		// let light2Col = 0xff6600
		let light2Col = 0x820333
		let light2 = new THREE.PointLight( light2Col, 1, 60 );
		light2.position.set( 17, 10, -5 );
		scene.add( light2 );




			var terrainGeometry = new THREE.SphereGeometry( 8,10, 40 );
			let terrainMaterial = new THREE.MeshLambertMaterial( { color: 0xffffff, side: THREE.DoubleSide, shading: THREE.SmoothShading } );
			let materialShader

			terrainMaterial.onBeforeCompile = (shader) => {
				shader.uniforms.time = { value: 0}
				shader.uniforms.intensity = { value: 0.5 }
				shader.uniforms.amount = { value: 5 }
				shader.vertexShader = `
					uniform float time;
					uniform float intensity;
					uniform float amount;
					` + shader.vertexShader
				const token = '#include <begin_vertex>'
				const customTransform = `
					vec3 transformed = vec3(position);
					transformed.x = position.x + sin(position.y*amount + time*0.1)*(intensity * 2.0);
					transformed.y = position.y + sin(position.x*amount + time*0.1)*(intensity * 0.1);
					// transformed.y = position.y + sin(position.y*intensity + time*0.5)*0.5;
					transformed.z = position.z - sin(position.z*amount + time*0.2)*(intensity * 0.1);
				`
				shader.vertexShader = shader.vertexShader.replace(token,customTransform)
				materialShader = shader
			}

			let terrain = new THREE.Mesh( terrainGeometry, terrainMaterial );
			terrain.geometry.computeVertexNormals(true);
			terrain.position.z = -3
			scene.add( terrain );
		
		
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


		var animate = (frame) => {
			if(!document.hidden) {
				requestAnimationFrame( animate );
				TWEEN.update();
				composer.render();

				if(materialShader) {
					materialShader.uniforms.time.value = frame/400
					materialShader.uniforms.intensity.value = this.cursorX / 100 + 10
					// materialShader.uniforms.amount.value = this.cursorY / 100
				}

				camera.position.z = this.cursorX / 800 + 9

				// renderer.render( scene, camera );

				const tiltCoords = { x: camera.rotation.x, y: camera.rotation.y }
				const tilt = new TWEEN.Tween(tiltCoords)
						.to({
							y: this.cursorX * -0.00008 - 0.35,
							x: this.cursorY * -0.00005 + 0.07
						}, 800)
						.easing(TWEEN.Easing.Quadratic.Out)
						.onUpdate(() => {
							camera.rotation.x = tiltCoords.x
							camera.rotation.y = tiltCoords.y
						})
						// .start()


				// cubeCamera.update(renderer, scene)
			}
		};

		animate();
	}
}
</script>

<style lang="scss">
.blog {
	margin: 150px;
	max-width: 800px;
	ul {
		margin: 0;
		padding: 0;
		list-style: none;
		li {
			color: white;
			margin: 70px 0;
			.post-link {
				font-size: 18px;
				text-transform: uppercase;
				letter-spacing: 1px;
				color: white;
				font-weight: bold;
				text-decoration: none;
				line-height: 180%;
			}
			.post-date {
				opacity: 0.75;
				margin-top: 20px;
				user-select: none;
			}
		}
	}
}

.feed {
	color: white;
	text-transform: uppercase;
	font-weight: bold;
	text-decoration: none;
	opacity: 0.75;
}

.menu {
	position: fixed;
	bottom: 150px;
	right: 150px;
	color: white;
	ul {
		list-style: none;
		margin: 0;
		padding: 0;
		li {
			text-align: right;
			font-weight: bold;
			margin-top: 30px;
			text-transform: uppercase;
			letter-spacing: 1px;	
			a {
				color: white;
				text-decoration: none;
			}
		}
	}
}

.scene {
	position: fixed;
	width: 100%;
	height: 100%;
	backgroud: #efefef;
	z-index: -1;
	top: 0;
}


@media(max-width: 700px) {
	.blog {
		margin: 40px!important;
		ul {
			li {
				margin: 50px 0;
				.post-link {
					font-size: 16px;
				}
			}
		}
	}
	.menu {
		bottom: 40px!important;
		right: 40px!important;
	}
}
</style>
