<template>
	<div class="container">
		<div class="point point-1" @click="showModal">
			<div class="point__label">1</div>
		</div>
		<div class="point point-2" @click="showModal">
			<div class="point__label">2</div>
		</div>
		<Modal v-show="isModalVisible" @close="closeModal" />
	</div>
</template>

<script>
import Modal from './Modal.vue';
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';

export default {
	name: 'Scene',
	components: {
		Modal,
	},
	data() {
		return {
			isModalVisible: false,
			scene: null,
			camera: null,
			controls: null,
			renderer: null,
		};
	},
	methods: {
		showModal() {
			this.isModalVisible = true;
		},
		closeModal() {
			this.isModalVisible = false;
		},
		async init() {
			this.setScene();
			await this.loadGltf('./models/street_car.glb');

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

		loadGltf(path) {
			return new Promise((resolve) => {
				const loader = new GLTFLoader();
				loader.load(path, (gltf) => {
					this.model = gltf.scene;
					this.model.traverse((object) => {
						if (object.isMesh) {
							object.castShadow = true;
							object.receiveShadow = true;
						}
					});
					this.model.scale.multiplyScalar(40);
					this.scene.add(this.model);
					resolve(this.model);
				});
			});
		},

		clickEvent(text) {
			alert(text);
		},

		onWindowResize() {
			this.camera.aspect = window.innerWidth / window.innerHeight;
			this.camera.updateProjectionMatrix();
			this.renderer.setSize(window.innerWidth, window.innerHeight);
		},

		animate() {
			this.controls.update();
			this.renderer.render(this.scene, this.camera);
			requestAnimationFrame(this.animate.bind(this));
		},
	},
	mounted() {
		this.init();
	},
};
</script>

<style lang="scss">
.point {
	position: absolute;
	&-1 {
		top: 50%;
		left: 50%;
	}
	&-2 {
		top: 55%;
		left: 45%;
	}
	&__label {
		position: absolute;
		top: -20px;
		left: -20px;
		width: 20px;
		height: 20px;
		border-radius: 50%;
		background: #00000077;
		border: 1px solid #ffffff77;
		color: #ffffff;
		font-family: Helvetica, Arial, sans-serif;
		text-align: center;
		line-height: 20px;
		font-weight: 50;
		font-size: 7px;
		cursor: help;
	}
}
</style>
