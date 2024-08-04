<template>
	<main class="center h-[80dvh]">
		<section id="container" class="relative w-full max-w-[920px]">
			<img id="img" src="/certificate.jpeg" class="w-10/12 object-contain bg-cover bg-center bg-no-repeat" :style="{ backgroundImage: 'url(/certificate.svg)' }">

			<div
				id="input-container"
				class="flex absolute"
				:style="{ top: position.y + 'px', left: position.x + 'px' }"
				@mousedown="startDrag"
			>
				<input type="text" class="input-field md:min-w-[360px]">
				<button class="btn py-[11px] px-3 bg-dark text-light absolute right-0 cursor-move">
					Drag me
				</button>
			</div>
		</section>

		<button class="btn mt-4 px-4 py-2  rounded" @click="savePosition">
			Save Position
		</button>
	</main>
</template>

<script setup lang="ts">


const position = ref({ x: 0, y: 0 })
const isDragging = ref(false)
const offset = ref({ x: 0, y: 0 })
const containerRect = ref<DOMRect | null>(null)

onMounted(() => {
  const container = document.getElementById('img')
  if (container) {
    containerRect.value = container.getBoundingClientRect()
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
  if (!isDragging.value || !containerRect.value) return

  let newX = event.clientX - offset.value.x
  let newY = event.clientY - offset.value.y

  // Constrain movement within the container
  newX = Math.max(0, Math.min(newX, containerRect.value.width - 100)) // 100 is an approximation of the input container width
  newY = Math.max(0, Math.min(newY, containerRect.value.height - 40)) // 40 is an approximation of the input container height

  position.value = { x: newX, y: newY }
}

function stopDrag() {
  isDragging.value = false
}

function savePosition() {
  const container = document.getElementById('container')
  const inputContainer = document.getElementById('input-container')

  if (container && inputContainer) {
    const containerRect = container.getBoundingClientRect()
    const inputRect = inputContainer.getBoundingClientRect()

    const relativePosition = {
      x: inputRect.left - containerRect.left,
      y: inputRect.top - containerRect.top
    }

    console.log('Saved position:', relativePosition)
    // Here you can send this data to a server or store it locally
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
</style>
