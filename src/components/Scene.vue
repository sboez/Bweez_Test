<template>
	<div class="container">
		<Modal ref="modal" v-show="isModalVisible" @close="closeModal" />
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
			this.scene.background = new THREE.Color(0xa0a0a0);

			this.camera = new THREE.PerspectiveCamera(
				45,
				window.innerWidth / window.innerHeight,
				1,
				1000
			);
			this.camera.position.set(0, 100, 200);

			this.light = new THREE.HemisphereLight(0xffffff, 0x404040, 1);
			this.scene.add(this.light);

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

			this.set3DPoints();
			this.setRaycaster();
			this.eventListener();
			this.setRenderer();
			this.setControls();
		},

		set3DPoints() {
			this.point1 = new THREE.Mesh(
				new THREE.SphereGeometry(0.5, 6, 6),
				new THREE.MeshBasicMaterial({ color: '#ff0000' })
			);
			this.point1.position.set(0, 15.5, 20);

			this.point2 = new THREE.Mesh(
				new THREE.SphereGeometry(0.5, 6, 6),
				new THREE.MeshBasicMaterial({ color: '#ff0000' })
			);
			this.point2.position.set(0, 19.5, -29);

			this.point3 = new THREE.Mesh(
				new THREE.SphereGeometry(0.5, 6, 6),
				new THREE.MeshBasicMaterial({ color: '#ff0000' })
			);
			this.point3.position.set(-15.5, 5, 16);

			this.scene.add(this.point1, this.point2, this.point3);
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
						case this.point1:
							this.$refs.modal.showText(this.modalDescription[0]);
							break;

						case this.point2:
							// this.showModal();
							this.$refs.modal.showText(this.modalDescription[1]);
							break;

						case this.point3:
							// this.showModal();
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

		onWindowResize() {
			this.camera.aspect = window.innerWidth / window.innerHeight;
			this.camera.updateProjectionMatrix();
			this.renderer.setSize(window.innerWidth, window.innerHeight);
		},

		animate() {
			this.controls.update();
			this.raycaster.setFromCamera(this.mouse, this.camera);

			const objectsToTest = [this.point1, this.point2, this.point3];
			const intersects = this.raycaster.intersectObjects(objectsToTest);

			intersects.length
				? (this.currentIntersect = intersects[0])
				: (this.currentIntersect = null);

			this.renderer.render(this.scene, this.camera);
			requestAnimationFrame(this.animate.bind(this));
		},
	},
	mounted() {
		this.init();
	},
};
</script>
