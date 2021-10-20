<template>
	<div class="container"></div>
</template>

<script>
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';

export default {
	name: 'Scene',
	data() {
		return {
			scene: null,
			camera: null,
			controls: null,
			renderer: null,
		};
	},
	methods: {
		init() {
			this.setScene();

			document.body.appendChild(this.renderer.domElement);
			window.addEventListener(
				'resize',
				this.onWindowResize.bind(this),
				false
			);

			this.animate();
		},

		setScene() {
			this.scene = new THREE.Scene();
			this.scene.background = new THREE.Color(0xa0a0a0);

			this.camera = new THREE.PerspectiveCamera(
				45,
				window.innerWidth / window.innerHeight,
				1,
				1000
			);
			this.camera.position.set(0, 100, 200);

			this.plane = new THREE.Mesh(
				new THREE.PlaneBufferGeometry(200, 200),
				new THREE.MeshPhongMaterial({
					color: 0xcfcfcf,
					side: THREE.DoubleSide,
				})
			);
			this.plane.rotation.x = -Math.PI / 2;
			this.plane.receiveShadow = true;
			this.scene.add(this.plane);

			this.setLights();
			this.setRenderer();
			this.setControls();
		},

		setLights() {
			this.light = new THREE.HemisphereLight(0xffffff, 0x404040, 1);
			this.scene.add(this.light);
		},

		setRenderer() {
			this.renderer = new THREE.WebGLRenderer({ antialias: true });
			this.renderer.setSize(window.innerWidth, window.innerHeight);
		},

		setControls() {
			this.controls = new OrbitControls(
				this.camera,
				this.renderer.domElement
			);
			this.controls.update();
		},

		onWindowResize() {
			this.camera.aspect = window.innerWidth / window.innerHeight;
			this.camera.updateProjectionMatrix();
			this.renderer.setSize(window.innerWidth, window.innerHeight);
		},

		animate() {
			this.controls.update();
			// this.addIntersectionWithPoints();
			this.renderer.render(this.scene, this.camera);
			requestAnimationFrame(this.animate.bind(this));
		},
	},
	mounted() {
		this.init();
	},
};
</script>
