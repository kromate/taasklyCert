<template>
	<section class="relative flex flex-col items-center justify-center h-screen gap-6 px-4 py-12 md:p-12 bg">
		<div ref="pdfSection" class="holder w-[750px] max-w-[98vw] relative bg-white">
			<Gdsc_certificate :name="name" />
		</div>

		<div class="relative field max-w-[365px] bg-transparent">
			<input
				id="email"
				v-model="name"
				type="email"
				placeholder="Enter the name of your team member"
				name="email"
				class="input-field"
				required
			>
		</div>
		<button class="bg-[#283ba4] text-white px-4 py-2 rounded-md" @click="capture">
			Generate certificate
		</button>
	</section>
</template>

<script setup lang="ts">
import html2canvas from 'html2canvas'
import { currentUser } from '@/composable/base'
import Gdsc_certificate from '@/components/gdsc_certificate.vue'

const name = ref('')
const pdfSection = ref<HTMLElement | any>(null)
const capture = async () => {
	const canvas = await html2canvas(pdfSection.value)
	DownloadCanvasAsImage(canvas, name.value || 'certificate')
}

const DownloadCanvasAsImage = (canvas:HTMLCanvasElement, name:string) => {
    const downloadLink = document.createElement('a')
    downloadLink.setAttribute('download', `${name}.png`)
    const dataURL = canvas.toDataURL('image/png')
    const url = dataURL.replace(/^data:image\/png/, 'data:application/octet-stream')
    downloadLink.setAttribute('href', url)
    downloadLink.click()
}

definePageMeta({
    middleware: [() => {

    }]
})
</script>

<style scoped>

</style>
