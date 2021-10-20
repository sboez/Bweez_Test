<template>
	<div class="container">
		<Modal ref="modal" v-show="isModalVisible" @close="closeModal" />
	</div>
</template>

<script>
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
import Modal from './Modal.vue';

export default {
	name: 'Scene',
	components: {
		Modal,
	},
	data() {
		return {
			scene: null,
			camera: null,
			controls: null,
			renderer: null,
			points: [null],
			isModalVisible: false,
			modalDescription: [
				'Je suis le point 1 !',
				'Le spoiler pour aller plus vite !',
				'Les jantes 18"',
			],
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
			this.currentIntersect = null;

			this.setScene();
			await this.loadGltf('./models/street_car.glb');

			document.body.appendChild(this.renderer.domElement);
			this.animate();
		},

		setScene() {
			this.mouse = new THREE.Vector2();

			this.scene = new THREE.Scene();
			this.scene.background = new THREE.Color(0xf1f1f1);
			this.scene.fog = new THREE.FogExp2(0xf1f1f1, 0.0015);

			this.camera = new THREE.PerspectiveCamera(
				45,
				window.innerWidth / window.innerHeight,
				1,
				1000
			);
			this.camera.position.set(0, 80, 180);

			this.hemLight = new THREE.HemisphereLight(0xffffff, 0x404040, 1);
			this.scene.add(this.hemLight);

			this.plane = new THREE.Mesh(
				new THREE.PlaneBufferGeometry(1000, 1000),
				new THREE.MeshPhongMaterial({
					color: 0xcfcfcf,
				})
			);
			this.plane.rotation.x = -Math.PI / 2;
			this.scene.add(this.plane);

			this.setPointsOfInterest();
			this.setRaycaster();
			this.eventListener();
			this.setRenderer();
			this.setControls();
		},

		setPointsOfInterest() {
			this.customPoint = new THREE.Mesh(
				new THREE.SphereGeometry(0.7, 12, 12),
				new THREE.MeshBasicMaterial({ color: '#ff0000' })
			);

			for (let i = 0; i < 3; ++i) {
				this.points[i] = this.customPoint.clone();
				this.scene.add(this.points[i]);
			}

			this.points[0].position.set(0, 15.5, 20);
			this.points[1].position.set(0, 19.5, -29);
			this.points[2].position.set(-15.5, 5, 16);
		},

		setRaycaster() {
			this.raycaster = new THREE.Raycaster();
			this.currentIntersect = null;
			this.rayDirection = new THREE.Vector3(10, 0, 0);
			this.rayDirection.normalize();
		},

		eventListener() {
			window.addEventListener('resize', this.onWindowResize, false);

			window.addEventListener('mousemove', (event) => {
				this.mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
				this.mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
			});

			window.addEventListener('click', () => {
				if (this.currentIntersect) {
					this.showModal();

					switch (this.currentIntersect.object) {
						case this.points[0]:
							this.$refs.modal.showText(this.modalDescription[0]);
							break;

						case this.points[1]:
							this.$refs.modal.showText(this.modalDescription[1]);
							break;

						case this.points[2]:
							this.$refs.modal.showText(this.modalDescription[2]);
							break;
					}
				}
			});
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
			this.controls.maxPolarAngle = Math.PI / 2;
			this.controls.minPolarAngle = Math.PI / 3;
			this.controls.enableDamping = true;
			this.controls.dampingFactor = 0.2;
			this.controls.maxDistance = 300;
			this.controls.minDistance = 40;
			this.controls.target = new THREE.Vector3(0, 15, 0);
		},

		loadGltf(path) {
			return new Promise((resolve) => {
				const loader = new GLTFLoader();
				loader.load(path, (gltf) => {
					this.model = gltf.scene;
					this.model.scale.multiplyScalar(40);
					this.scene.add(this.model);
					resolve(this.model);
				});
			});
		},

		onWindowResize() {
			this.camera.aspect = window.innerWidth / window.innerHeight;
			this.camera.updateProjectionMatrix();
			this.renderer.setSize(window.innerWidth, window.innerHeight);
		},

		animate() {
			this.controls.update();
			this.raycaster.setFromCamera(this.mouse, this.camera);

			const objectsToTest = this.points;
			const intersects = this.raycaster.intersectObjects(objectsToTest);

			if (intersects.length) {
				if (this.currentIntersect)
					document.querySelector('html').style.cursor = 'pointer';
				this.currentIntersect = intersects[0];
			} else {
				document.querySelector('html').style.cursor = 'default';
				this.currentIntersect = null;
			}

			this.renderer.render(this.scene, this.camera);
			requestAnimationFrame(this.animate.bind(this));
		},
	},
	mounted() {
		this.init();
	},
};
</script>
