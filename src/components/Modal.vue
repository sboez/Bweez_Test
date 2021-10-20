<script>
export default {
	name: 'Modal',
	data() {
		return {
			text: null,
		};
	},
	methods: {
		close() {
			this.$emit('close');
		},
		showText(description) {
			this.text = description;
		},
	},
};
</script>

<template>
	<transition name="modal-fade">
		<div class="modal-backdrop">
			<div
				class="modal"
				role="dialog"
				aria-labelledby="modalTitle"
				aria-describedby="modalDescription"
			>
				<section class="modal-body" id="modalDescription">
					<slot name="body">
						{{ text }}
					</slot>
				</section>

				<footer class="modal-footer">
					<button
						type="button"
						class="btn-green"
						@click="close"
						aria-label="Close modal"
					>
						Close me!
					</button>
				</footer>
			</div>
		</div>
	</transition>
</template>

<style>
.modal-backdrop {
	position: fixed;
	top: 0;
	bottom: 0;
	left: 0;
	right: 0;
	background-color: rgba(0, 0, 0, 0.3);
	display: flex;
	justify-content: center;
	align-items: center;
}

.modal {
	background: #ffffff;
	box-shadow: 2px 2px 20px 1px;
	overflow-x: auto;
	display: flex;
	flex-direction: column;
}

.modal-body {
	position: relative;
	padding: 20px 10px;
}

.modal-footer {
	border-top: 1px solid #eeeeee;
	flex-direction: column;
	padding: 15px;
	display: flex;
}

.btn-green {
	color: white;
	background: #4aae9b;
	border: 1px solid #4aae9b;
	border-radius: 2px;
	cursor: pointer;
}

.modal-fade-enter,
.modal-fade-leave-to {
	opacity: 0;
}

.modal-fade-enter-active,
.modal-fade-leave-active {
	transition: opacity 0.5s ease;
}
</style>
