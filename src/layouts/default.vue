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
