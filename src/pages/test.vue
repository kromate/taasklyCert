<template>
	<main class="center h-[80dvh]">
		<section id="container" class="relative w-full max-w-[920px]">
			<div id="cert" class="relative w-[800px] h-[600px] bg-cover bg-center bg-no-repeat" :style="{ backgroundImage: `url(${backgroundImage})` }">
				<div
					id="input-container"
					class="flex absolute"
					:style="{ top: position.y + 'px', left: position.x + 'px' }"
					@mousedown="startDrag"
				>
					<input type="text" :class="['input-field md:min-w-[360px] text-3xl font-medium', { 'no-border': isDownloading }]">
					<button :class="['btn py-[11px] px-3 bg-dark text-light absolute right-0 cursor-move', { 'hidden': isDownloading }]">
						Drag me
					</button>
				</div>
			</div>
		</section>

		<div class="flex gap-4 mt-4">
			<label class="btn px-4 py-2 rounded cursor-pointer">
					Upload Image
					<input type="file" @change="handleImageUpload" accept="image/*" class="hidden">
			</label>
			<button class="btn px-4 py-2 rounded" @click="savePosition">
				Save Position
			</button>
			<button class="btn px-4 py-2 rounded" @click="downloadCertificate">
				Download Certificate
			</button>
		</div>
	</main>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import html2canvas from 'html2canvas'

const position = ref({ x: 0, y: 0 })
const isDragging = ref(false)
const offset = ref({ x: 0, y: 0 })
const certRect = ref<DOMRect | null>(null)
const backgroundImage = ref('')
const isDownloading = ref(false)

onMounted(() => {
	const cert = document.getElementById('cert')
	if (cert) {
		certRect.value = cert.getBoundingClientRect()
	}

	window.addEventListener('mousemove', drag)
	window.addEventListener('mouseup', stopDrag)
})

onUnmounted(() => {
	window.removeEventListener('mousemove', drag)
	window.removeEventListener('mouseup', stopDrag)
})

function startDrag(event: MouseEvent) {
	isDragging.value = true
	offset.value = {
		x: event.clientX - position.value.x,
		y: event.clientY - position.value.y
	}
}

function drag(event: MouseEvent) {
	if (!isDragging.value || !certRect.value) return

	let newX = event.clientX - offset.value.x
	let newY = event.clientY - offset.value.y

	// Constrain movement within the cert div
	newX = Math.max(0, Math.min(newX, certRect.value.width - 100)) // 100 is an approximation of the input container width
	newY = Math.max(0, Math.min(newY, certRect.value.height - 40)) // 40 is an approximation of the input container height

	position.value = { x: newX, y: newY }
}

function stopDrag() {
	isDragging.value = false
}

function savePosition() {
	const cert = document.getElementById('cert')
	const inputContainer = document.getElementById('input-container')

	if (cert && inputContainer) {
		const certRect = cert.getBoundingClientRect()
		const inputRect = inputContainer.getBoundingClientRect()

		const relativePosition = {
			x: inputRect.left - certRect.left,
			y: inputRect.top - certRect.top
		}

		console.log('Saved position:', relativePosition)
		// Here you can send this data to a server or store it locally
	}
}

function handleImageUpload(event: Event) {
	const file = (event.target as HTMLInputElement).files?.[0]
	if (file) {
		const reader = new FileReader()
		reader.onload = (e) => {
			backgroundImage.value = e.target?.result as string
		}
		reader.readAsDataURL(file)
	}
}

function downloadCertificate() {
	const cert = document.getElementById('cert')
	if (cert) {
		isDownloading.value = true
		setTimeout(() => {
			html2canvas(cert).then((canvas) => {
				const link = document.createElement('a')
				link.download = 'certificate.png'
				link.href = canvas.toDataURL()
				link.click()
				isDownloading.value = false
			})
		}, 100) // Small delay to ensure CSS changes are applied before capture
	}
}
</script>

<style scoped>
.center {
	display: flex;
	justify-content: center;
	align-items: center;
	flex-direction: column;
}

.cursor-move {
	cursor: move;
}

.hidden {
	display: none;
}

.no-border {
	border: none;
	outline: none;
	background: transparent;
}
</style>
