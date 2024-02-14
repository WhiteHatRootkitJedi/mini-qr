<script setup lang="ts">
import StyledQRCode from '@/components/StyledQRCode.vue'
import {
  copyImageToClipboard,
  downloadPngElement,
  downloadSvgElement,
  IS_COPY_IMAGE_TO_CLIPBOARD_SUPPORTED
} from '@/utils/convertToImage'
import type { CornerDotType, CornerSquareType, DotType } from 'qr-code-styling'
import { computed, onMounted, ref, watch } from 'vue'
import 'vue-i18n'
import { useI18n } from 'vue-i18n'
import { getNumericCSSValue } from './utils/formatting'
import { sortedLocales } from './utils/language'
import { allPresets } from './utils/presets'

const { t, locale } = useI18n()

const defaultPreset = allPresets[0]
const url = ref()
const data = ref()
const image = ref()
const width = ref()
const height = ref()
const margin = ref()
const imageMargin = ref()

const dotsOptionsColor = ref()
const dotsOptionsType = ref()
const cornersSquareOptionsColor = ref()
const cornersSquareOptionsType = ref()
const cornersDotOptionsColor = ref()
const cornersDotOptionsType = ref()
const styleBorderRadius = ref()
const styledBorderRadiusFormatted = computed(() => `${styleBorderRadius.value}px`)
const styleBackground = ref(defaultPreset.style.background)

const dotsOptions = computed(() => ({
  color: dotsOptionsColor.value,
  type: dotsOptionsType.value
}))
const cornersSquareOptions = computed(() => ({
  color: cornersSquareOptionsColor.value,
  type: cornersSquareOptionsType.value
}))
const cornersDotOptions = computed(() => ({
  color: cornersDotOptionsColor.value,
  type: cornersDotOptionsType.value
}))
const style = computed(() => ({
  borderRadius: styledBorderRadiusFormatted.value,
  background: styleBackground.value
}))
const imageOptions = computed(() => ({
  margin: imageMargin.value
}))

const qrCodeProps = computed(() => ({
  url: url.value,
  data: data.value,
  image: image.value,
  width: width.value,
  height: height.value,
  margin: margin.value,
  dotsOptions: dotsOptions.value,
  cornersSquareOptions: cornersSquareOptions.value,
  cornersDotOptions: cornersDotOptions.value,
  imageOptions: imageOptions.value
}))

/* random settings utils */

function createRandomColor() {
  return '#' + Math.floor(Math.random() * 16777215).toString(16)
}

function getRandomItemInArray(array: any[]) {
  return array[Math.floor(Math.random() * array.length)]
}

function randomizeStyleSettings() {
  const dotTypes: DotType[] = [
    'dots',
    'rounded',
    'classy',
    'classy-rounded',
    'square',
    'extra-rounded'
  ]
  const cornerSquareTypes: CornerSquareType[] = ['dot', 'square', 'extra-rounded']
  const cornerDotTypes: CornerDotType[] = ['dot', 'square']

  dotsOptionsType.value = getRandomItemInArray(dotTypes)
  dotsOptionsColor.value = createRandomColor()

  cornersSquareOptionsType.value = getRandomItemInArray(cornerSquareTypes)
  cornersSquareOptionsColor.value = createRandomColor()

  cornersDotOptionsType.value = getRandomItemInArray(cornerDotTypes)
  cornersDotOptionsColor.value = createRandomColor()

  styleBackground.value = createRandomColor()
}

const selectedPreset = ref(defaultPreset)

watch(selectedPreset, () => {
  url.value = selectedPreset.value.url
  data.value = selectedPreset.value.data
  image.value = selectedPreset.value.image
  width.value = selectedPreset.value.width
  height.value = selectedPreset.value.height
  margin.value = selectedPreset.value.margin
  imageMargin.value = selectedPreset.value.imageOptions.margin
  dotsOptionsColor.value = selectedPreset.value.dotsOptions.color
  dotsOptionsType.value = selectedPreset.value.dotsOptions.type
  cornersSquareOptionsColor.value = selectedPreset.value.cornersSquareOptions.color
  cornersSquareOptionsType.value = selectedPreset.value.cornersSquareOptions.type
  cornersDotOptionsColor.value = selectedPreset.value.cornersDotOptions.color
  cornersDotOptionsType.value = selectedPreset.value.cornersDotOptions.type
  styleBorderRadius.value = getNumericCSSValue(selectedPreset.value.style.borderRadius as string)
  styleBackground.value = selectedPreset.value.style.background
})

/* export image utils */
const options = computed(() => ({
  width: width.value,
  height: height.value
}))

async function copyQRToClipboard() {
  console.debug('Copying image to clipboard')
  const qrCode = document.querySelector('#qr-code-container')
  if (qrCode) {
    await copyImageToClipboard(qrCode as HTMLElement, options.value)
  }
}

function downloadQRImageAsPng() {
  console.debug('Downloading image as png')
  const qrCode = document.querySelector('#qr-code-container')
  if (qrCode) {
    downloadPngElement(qrCode as HTMLElement, 'qr-code.png', options.value)
  }
}

function downloadQRImageAsSvg() {
  console.debug('Downloading image as svg')
  const qrCode = document.querySelector('#qr-code-container')
  if (qrCode) {
    downloadSvgElement(qrCode as HTMLElement, 'qr-code.svg', options.value)
  }
}

function uploadImage() {
  console.debug('Uploading image')
  const imageInput = document.createElement('input')
  imageInput.type = 'file'
  imageInput.accept = 'image/*'
  imageInput.onchange = (event: Event) => {
    const target = event.target as HTMLInputElement
    if (target.files) {
      const file = target.files[0]
      const reader = new FileReader()
      reader.onload = (event: ProgressEvent<FileReader>) => {
        const target = event.target as FileReader
        const result = target.result as string
        image.value = result
      }
      reader.readAsDataURL(file)
    }
  }
  imageInput.click()
}

function setImageToPABLogo() {
  console.debug('Setting image to our logo')
  image.value = 'data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz4NCjwhLS0gR2VuZXJhdG9yOiBBZG9iZSBJbGx1c3RyYXRvciAxOS4wLjAsIFNWRyBFeHBvcnQgUGx1Zy1JbiAuIFNWRyBWZXJzaW9uOiA2LjAwIEJ1aWxkIDApICAtLT4NCjxzdmcgdmVyc2lvbj0iMS4xIiBpZD0iTGF5ZXJfMSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxuczp4bGluaz0iaHR0cDovL3d3dy53My5vcmcvMTk5OS94bGluayIgeD0iMHB4IiB5PSIwcHgiDQoJIHZpZXdCb3g9IjAgMCAzNjMuMiAzMDguMyIgc3R5bGU9ImVuYWJsZS1iYWNrZ3JvdW5kOm5ldyAwIDAgMzYzLjIgMzA4LjM7IiB4bWw6c3BhY2U9InByZXNlcnZlIj4NCjxzdHlsZSB0eXBlPSJ0ZXh0L2NzcyI+DQoJLnN0MHtmaWxsOiMyMzFGMjA7fQ0KPC9zdHlsZT4NCjxnPg0KCTxnPg0KCQk8cGF0aCBjbGFzcz0ic3QwIiBkPSJNODcuOCwyMTUuMWMwLTAuMywwLjMtMC42LDAuNi0wLjZoNy45YzQuMSwwLDcuMywzLjMsNy4zLDcuM2MwLDQtMy4zLDcuMy03LjMsNy4zaC0zLjR2Ny43DQoJCQljMCwwLjMtMC4zLDAuNi0wLjYsMC42aC0zLjljLTAuMywwLTAuNi0wLjMtMC42LTAuNlYyMTUuMXogTTk2LDIyNC40YzEuNCwwLDIuNS0xLjEsMi41LTIuNXMtMS4xLTIuNS0yLjUtMi41aC0zLjF2NUg5NnoiLz4NCgkJPHBhdGggY2xhc3M9InN0MCIgZD0iTTExMi43LDIzNi41bDEwLjMtMjIuMmMwLjEtMC4yLDAuMy0wLjMsMC42LTAuNGgwLjNjMC4yLDAsMC40LDAuMSwwLjYsMC40bDEwLjMsMjIuMg0KCQkJYzAuMiwwLjMsMC4xLDAuNi0wLjIsMC44Yy0wLjEsMC4xLTAuMiwwLjEtMC4zLDAuMWgtMy43Yy0wLjYsMC0wLjktMC4yLTEuMi0wLjhsLTEuMi0yLjVoLTguOWwtMS4yLDIuNmMtMC4yLDAuNS0wLjcsMC44LTEuMiwwLjgNCgkJCWgtMy42Yy0wLjMsMC4xLTAuNi0wLjEtMC43LTAuNEMxMTIuNSwyMzYuOSwxMTIuNiwyMzYuNywxMTIuNywyMzYuNSBNMTI2LjIsMjI5LjVsLTIuNS01LjRsLTIuNSw1LjRIMTI2LjJ6Ii8+DQoJCTxwYXRoIGNsYXNzPSJzdDAiIGQ9Ik0xNjQuNCwyMTUuMWMwLTAuMywwLjMtMC42LDAuNi0wLjZoNy41YzMuNi0wLjMsNi44LDIuNCw3LjEsNmMtMC4zLDIuMy0xLjcsNC4yLTMuOCw1LjINCgkJCWMyLjQsMC43LDQuMiwyLjksNC40LDUuNGMtMC4yLDMuNy0zLjMsNi41LTcsNi40Yy0wLjEsMC0wLjEsMC0wLjIsMGgtOC4xYy0wLjMsMC0wLjYtMC4zLTAuNi0wLjZWMjE1LjF6IE0xNzIuMiwyMjMuNw0KCQkJYzEuMywwLDIuMy0xLjEsMi4zLTIuNHYwYzAtMS4yLTEtMi4yLTIuMi0yLjJjMCwwLTAuMSwwLTAuMSwwaC0yLjV2NC42SDE3Mi4yeiBNMTcyLjYsMjMyLjdjMS40LDAsMi41LTEuMSwyLjUtMi41DQoJCQljLTAuMi0xLjQtMS4zLTIuNC0yLjctMi4zaC0yLjd2NC44TDE3Mi42LDIzMi43eiIvPg0KCQk8cGF0aCBjbGFzcz0ic3QwIiBkPSJNMTk0LjQsMjE1LjFjMC0wLjMsMC4zLTAuNiwwLjYtMC42aDMuOWMwLjMsMCwwLjYsMC4zLDAuNiwwLjZ2MjEuOGMwLDAuMy0wLjMsMC42LTAuNiwwLjZIMTk1DQoJCQljLTAuMywwLTAuNi0wLjMtMC42LTAuNlYyMTUuMXoiLz4NCgkJPHBhdGggY2xhc3M9InN0MCIgZD0iTTIxNS4xLDIxNS4xYzAtMC4zLDAuMy0wLjYsMC42LTAuNmg3LjVjMy42LTAuMyw2LjgsMi41LDcuMSw2LjFjMCwwLDAsMCwwLDBjLTAuMywyLjMtMS43LDQuMi0zLjgsNS4yDQoJCQljMi40LDAuNyw0LjIsMi45LDQuNCw1LjRjLTAuMiwzLjctMy4zLDYuNS03LDYuNGMtMC4xLDAtMC4xLDAtMC4yLDBoLThjLTAuMywwLTAuNi0wLjMtMC42LTAuNlYyMTUuMXogTTIyMi44LDIyMy44DQoJCQljMS4zLDAsMi4zLTEuMSwyLjMtMi40YzAtMS4yLTEtMi4yLTIuMi0yLjJjMCwwLTAuMSwwLTAuMSwwaC0yLjV2NC42SDIyMi44eiBNMjIzLjIsMjMyLjhjMS4zLDAsMi40LTEuMSwyLjQtMi41DQoJCQljMC0xLjMtMS4xLTIuNC0yLjUtMi40Yy0wLjEsMC0wLjEsMC0wLjIsMGgtMi43djQuN0wyMjMuMiwyMzIuOHoiLz4NCgkJPHBhdGggY2xhc3M9InN0MCIgZD0iTTI0NSwyMTUuMWMwLTAuMywwLjMtMC42LDAuNi0wLjZoMy45YzAuMywwLDAuNiwwLjMsMC42LDAuNnYxNy42aDcuOWMwLjMsMCwwLjYsMC4zLDAuNiwwLjZ2My42DQoJCQljMCwwLjMtMC4zLDAuNi0wLjYsMC42aC0xMi40Yy0wLjMsMC0wLjYtMC4zLTAuNi0wLjZMMjQ1LDIxNS4xeiIvPg0KCQk8cGF0aCBjbGFzcz0ic3QwIiBkPSJNMjcxLjcsMjE1LjFjMC0wLjMsMC4zLTAuNiwwLjYtMC42SDI4NmMwLjMsMCwwLjYsMC4zLDAuNiwwLjZ2My42YzAsMC4zLTAuMywwLjYtMC42LDAuNmgtOS4xdjQuMWg3LjUNCgkJCWMwLjMsMCwwLjYsMC4zLDAuNiwwLjZ2My42YzAsMC4zLTAuMywwLjYtMC42LDAuNmgtNy41djQuNGg5LjFjMC4zLDAsMC42LDAuMywwLjYsMC42djMuNmMwLDAuMy0wLjMsMC42LTAuNiwwLjZoLTEzLjYNCgkJCWMtMC4zLDAtMC42LTAuMy0wLjYtMC42VjIxNS4xeiIvPg0KCTwvZz4NCgk8cGF0aCBjbGFzcz0ic3QwIiBkPSJNMjgyLjQsMTc4LjRjMC0wLjEsMC0wLjMsMC0wLjRjMC0wLjEsMC0wLjItMC4xLTAuM2MwLTAuMSwwLTAuMi0wLjEtMC4zTDI1NywxMDAuOGMtMC40LTEuNi0xLjktMi43LTMuNi0yLjQNCgkJYy0wLjYsMC01LjMtMC41LTE1LjMtNi42Yy0xMC4zLTYuNS0yMC4zLTkuOC0yOS43LTkuOGMtNy45LTAuMi0xNS41LDIuMy0yMS43LDcuMWMtMTYuNC0xNC40LTM3LjUtMjIuNC01OS40LTIyLjQNCgkJYy0xLjgsMC0zLjYsMC4xLTUuNCwwLjJjLTEuMSwwLjEtMi4xLDAuNi0yLjcsMS42Yy0wLjIsMC4zLTE3LjMsMjUuNy00MS40LDYwLjZjLTAuNywxLTAuOCwyLjItMC40LDMuM2MwLjcsMS44LDIuNywyLjcsNC41LDINCgkJYzMtMS4xLDUuOS0yLjIsOC44LTMuMWMtMi44LDQuOS03LjEsMTIuNy0xMy43LDI1LjFjLTAuNiwxLjEtMC42LDIuNSwwLjEsMy41YzEsMS43LDMuMiwyLjIsNC45LDEuMWM2LjgtNC4zLDEzLjMtNy45LDE5LjYtMTAuOA0KCQljLTIuMiw0LjItNC4zLDgtNi40LDExLjdDOTIsMTY3LjksODguNiwxNzQsODUsMTgxLjdjLTAuMiwwLjUtMC40LDEtMC40LDEuNWMwLDEuOSwxLjYsMy41LDMuNSwzLjVjMC4yLDAsMC4zLDAsMC41LDANCgkJYzE3LjItMi4zLDM1LjQtNC4zLDU0LjEtNS45YzE1LTEuMiwyOS43LTIuMiw0My43LTIuN2w5Mi4zLDMuOGMxLjYsMC4xLDMtMSwzLjUtMi40YzAtMC4xLDAtMC4xLDAuMS0wLjJjMC0wLjEsMC0wLjIsMC4xLTAuMw0KCQljMC0wLjEsMC0wLjIsMC4xLTAuNGMwLDAsMCwwLDAtMC4xQzI4Mi40LDE3OC41LDI4Mi40LDE3OC40LDI4Mi40LDE3OC40eiBNMTI0LjEsNzMuOWMxLjEsMCwyLjItMC4xLDMuMy0wLjENCgkJYzIwLjYsMCw0MC41LDcuNyw1NS44LDIxLjVjLTAuMiwyMy42LTAuMyw0MC4yLTAuMyw1MS45Yy0wLjYtMS4yLTEuMy0yLjUtMi0zLjdjLTQuNC03LjYtMTAuNi0xMy45LTE4LjEtMTguNA0KCQljLTEwLTUuNi0yMS4zLTguNC0zMi43LTguMWMtMTMuNywwLjMtMjcuMywyLjctNDAuMyw3LjJDMTA3LjUsOTguNSwxMjAuNSw3OS4yLDEyNC4xLDczLjl6IE05NC41LDE3OC44Yy0wLjEsMC0wLjIsMC0wLjMsMA0KCQljMi41LTQuOCw0LjgtOSw3LjItMTMuNGMzLjEtNS42LDYuNC0xMS41LDEwLTE4LjhjMC4xLTAuMiwwLjEtMC4zLDAuMi0wLjVjNS4yLTEuOSwxMC4zLTMuMywxNS00LjJjOS42LTIuMSwxOS43LTEuNCwyOC45LDINCgkJYzQuOSwxLjksOS4zLDQuOCwxMy4xLDguNGMyLDEuOSwzLjgsNCw1LjUsNi4yQzE1MS4yLDE0OS44LDEwNi42LDE3Mi4zLDk0LjUsMTc4Ljh6IE0xNzMuNywxNDcuM2MtNC41LTQuMi05LjctNy42LTE1LjQtOS45DQoJCWMtMTAuNS0zLjktMjEuOS00LjgtMzIuOC0yLjRjLTEyLjcsMi43LTI0LjksNy4zLTM2LjIsMTMuN2M2LjgtMTIuNiwxMC0xOCwxMS4zLTIwLjJjMCwwLDAsMCwwLTAuMWMxMC44LTIuOCwyMC42LTQuMywyOS40LTQuMw0KCQljMTAuMi0wLjMsMjAuMiwyLjIsMjkuMSw3LjFjNi41LDMuOSwxMS45LDkuNCwxNS42LDE1LjljMC41LDAuOSwxLDEuOCwxLjUsMi44QzE3NS40LDE0OSwxNzQuNSwxNDguMiwxNzMuNywxNDcuM3ogTTI1OC40LDE2Mi44DQoJCWMtNi45LTMuNC0xNC4xLTYuMy0yMS41LTguN2MtNy40LTIuNS0xNS4xLTMuOS0yMi45LTR2MGMtNi44LTAuMi0xMy40LDEuNi0xOS4xLDUuMmMtMS44LDEuMy0zLjUsMi44LTUsNC40bDAuMi02NC40DQoJCWMxLjItMSwyLjYtMiw0LTIuN2M0LjQtMi40LDkuMi0zLjYsMTQuMi0zLjVjOS4zLDAuNCwxOC4zLDMuNCwyNS45LDguN2wwLjEsMGM1LjEsMy40LDEwLjcsNS44LDE2LjYsNy4zbDIxLjUsNjUuMg0KCQlDMjY4LjgsMTY4LjIsMjYzLjksMTY1LjUsMjU4LjQsMTYyLjh6Ii8+DQo8L2c+DQo8L3N2Zz4NCg=='
  imageMargin.value = -12
}

async function getShortLink() {
  // We can use the `Headers` constructor to create headers
  // and assign it as the type of the `headers` variable
  const headers: Headers = new Headers()
  // Add a few headers
  headers.set('Content-Type', 'application/json')
  headers.set('Accept', 'application/json')
  // Add a custom header, which we can use to check
  headers.set('X-Api-Key', 'df82b99c-fce9-4f42-9fd1-1c3105e70bf6')

  const request: RequestInfo = new Request('https://qr.pabibletf.org/rest/v3/short-urls', {
    method: 'POST',
    headers: headers,
    body: '{"longUrl": "' + url.value + '", "findIfExists": true}'
  })

  fetch(request)
    .then(res => res.json())
    .then(res => {
      data.value = res["shortUrl"] as String
    })
}

/* QR Config Utils */

function createQrConfig() {
  return {
    props: qrCodeProps.value,
    style: style.value
  }
}

function downloadQRConfig() {
  console.debug('Downloading QR code config')
  const qrCodeConfig = createQrConfig()
  const qrCodeConfigString = JSON.stringify(qrCodeConfig)
  const qrCodeConfigBlob = new Blob([qrCodeConfigString], { type: 'application/json' })
  const qrCodeConfigUrl = URL.createObjectURL(qrCodeConfigBlob)
  const qrCodeConfigLink = document.createElement('a')
  qrCodeConfigLink.href = qrCodeConfigUrl
  qrCodeConfigLink.download = 'qr-code-config.json'
  qrCodeConfigLink.click()
}

function saveQRConfigToLocalStorage() {
  const qrCodeConfig = createQrConfig()
  const qrCodeConfigString = JSON.stringify(qrCodeConfig)
  localStorage.setItem('qrCodeConfig', qrCodeConfigString)
}

function loadQRConfig(jsonString: string, name?: string) {
  const qrCodeConfig = JSON.parse(jsonString)
  const qrCodeProps = qrCodeConfig.props
  const qrCodeStyle = qrCodeConfig.style
  const preset = {
    ...qrCodeProps,
    style: qrCodeStyle
  }

  selectedPreset.value = {
    ...preset,
    name: name ?? qrCodeProps.name
  }
}

function loadQRConfigFromLocalStorage() {
  const qrCodeConfigString = localStorage.getItem('qrCodeConfig')
  if (qrCodeConfigString) {
    console.debug('Loading QR code config from local storage')
    loadQRConfig(qrCodeConfigString, t('Last saved locally'))
  } else {
    selectedPreset.value = { ...defaultPreset }
  }
}

function loadQrConfigFromFile() {
  console.debug('Loading QR code config')
  const qrCodeConfigInput = document.createElement('input')
  qrCodeConfigInput.type = 'file'
  qrCodeConfigInput.accept = 'application/json'
  qrCodeConfigInput.onchange = (event: Event) => {
    const target = event.target as HTMLInputElement
    if (target.files) {
      const file = target.files[0]
      const reader = new FileReader()
      reader.onload = (event: ProgressEvent<FileReader>) => {
        const target = event.target as FileReader
        const result = target.result as string
        loadQRConfig(result, t('Loaded from file'))
      }
      reader.readAsText(file)
    }
  }
  qrCodeConfigInput.click()
}

watch(locale, () => {
  selectedPreset.value = {
    ...selectedPreset.value,
    name: t(selectedPreset.value.name)
  }
})

watch(qrCodeProps, () => {
  saveQRConfigToLocalStorage()
})

onMounted(() => {
  loadQRConfigFromLocalStorage()
})
</script>

<template>
  <main class="relative grid place-items-center px-6 py-20 sm:p-8" role="main">
    <div class="absolute end-4 top-4 flex flex-row items-center gap-4">
      <div class="flex flex-row items-center">
        <svg xmlns="http://www.w3.org/2000/svg" width="36" height="36" viewBox="0 0 24 24">
          <g
            fill="none"
            stroke="#abcbca"
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
          >
            <path d="M4 5h7M7 4c0 4.846 0 7 .5 8" />
            <path
              d="M10 8.5c0 2.286-2 4.5-3.5 4.5S4 11.865 4 11c0-2 1-3 3-3s5 .57 5 2.857c0 1.524-.667 2.571-2 3.143m2 6l4-9l4 9m-.9-2h-6.2"
            />
          </g>
        </svg>
        <select
          class="secondary-button cursor-pointer text-center"
          id="locale-select"
          v-model="$i18n.locale"
          :aria-label="t('Change language')"
        >
          <option v-for="(locale, index) in sortedLocales" :key="index" :value="locale">
            {{ t(locale) }}
          </option>
        </select>
      </div>
      <div class="vertical-border"></div>
      <a
        class="icon-button"
        href="https://github.com/lyqht/styled-qr-code-generator"
        :aria-label="t('GitHub repository for this project')"
      >
        <svg xmlns="http://www.w3.org/2000/svg" width="36" height="36" viewBox="0 0 24 24">
          <path
            fill="#abcbca"
            d="M12.001 2c-5.525 0-10 4.475-10 10a9.994 9.994 0 0 0 6.837 9.488c.5.087.688-.213.688-.476c0-.237-.013-1.024-.013-1.862c-2.512.463-3.162-.612-3.362-1.175c-.113-.288-.6-1.175-1.025-1.413c-.35-.187-.85-.65-.013-.662c.788-.013 1.35.725 1.538 1.025c.9 1.512 2.337 1.087 2.912.825c.088-.65.35-1.087.638-1.337c-2.225-.25-4.55-1.113-4.55-4.938c0-1.088.387-1.987 1.025-2.688c-.1-.25-.45-1.275.1-2.65c0 0 .837-.262 2.75 1.026a9.28 9.28 0 0 1 2.5-.338c.85 0 1.7.112 2.5.337c1.913-1.3 2.75-1.024 2.75-1.024c.55 1.375.2 2.4.1 2.65c.637.7 1.025 1.587 1.025 2.687c0 3.838-2.337 4.688-4.563 4.938c.363.312.676.912.676 1.85c0 1.337-.013 2.412-.013 2.75c0 .262.188.574.688.474A10.016 10.016 0 0 0 22 12c0-5.525-4.475-10-10-10Z"
          />
        </svg>
      </a>
    </div>
    <div class="w-full md:w-5/6">
      <div class="mb-8 flex w-full flex-col items-center justify-center">
        <h1 class="text-4xl">{{ t('Mini QR Code Generator') }}</h1>
      </div>
      <div class="flex flex-col-reverse items-start justify-center gap-4 md:flex-row md:gap-12">
        <div
          id="main-content"
          class="sticky top-0 flex w-full shrink-0 flex-col items-center justify-center p-4 md:w-fit"
        >
          <div id="qr-code-container">
            <div
              class="grid place-items-center overflow-hidden"
              :style="[
                style,
                {
                  width: '200px',
                  height: '200px'
                }
              ]"
            >
              <StyledQRCode
                v-if="data"
                v-bind="{ ...qrCodeProps, width: 200, height: 200 }"
                role="img"
                aria-label="QR code"
              />
              <p v-else>{{ t('No data!') }}</p>
            </div>
          </div>
          <div class="mt-4 flex flex-col items-center gap-2">
            <div class="flex flex-col items-center justify-center gap-3">
              <button
                v-if="IS_COPY_IMAGE_TO_CLIPBOARD_SUPPORTED"
                id="copy-qr-image-button"
                class="button flex w-fit flex-row gap-1"
                @click="copyQRToClipboard"
                :aria-label="t('Copy QR Code to clipboard')"
              >
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
                  <g
                    fill="none"
                    stroke="currentColor"
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                  >
                    <path
                      d="M8 10a2 2 0 0 1 2-2h8a2 2 0 0 1 2 2v8a2 2 0 0 1-2 2h-8a2 2 0 0 1-2-2z"
                    />
                    <path d="M16 8V6a2 2 0 0 0-2-2H6a2 2 0 0 0-2 2v8a2 2 0 0 0 2 2h2" />
                  </g>
                </svg>
                <p>{{ t('Copy QR Code to clipboard') }}</p>
              </button>
              <button
                id="save-qr-code-config-button"
                class="button flex w-fit flex-row gap-1"
                @click="downloadQRConfig"
                :aria-label="t('Save QR Code configuration')"
              >
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
                  <g
                    fill="none"
                    stroke="currentColor"
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                  >
                    <path d="M14 3v4a1 1 0 0 0 1 1h4" />
                    <path
                      d="M17 21H7a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h7l5 5v11a2 2 0 0 1-2 2zm-5-4v-6"
                    />
                    <path d="M9.5 14.5L12 17l2.5-2.5" />
                  </g>
                </svg>
                <p>{{ t('Save QR Code configuration') }}</p>
              </button>
              <button
                id="load-qr-code-config-button"
                class="button flex w-fit flex-row gap-1"
                @click="loadQrConfigFromFile"
                :aria-label="t('Load QR Code configuration')"
              >
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
                  <g
                    fill="none"
                    stroke="currentColor"
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                  >
                    <path d="M14 3v4a1 1 0 0 0 1 1h4" />
                    <path
                      d="M17 21H7a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h7l5 5v11a2 2 0 0 1-2 2zm-5-10v6"
                    />
                    <path d="M9.5 13.5L12 11l2.5 2.5" />
                  </g>
                </svg>
                <p>{{ t('Load QR Code configuration') }}</p>
              </button>
            </div>
            <div id="export-options" class="pt-4">
              <p class="pb-2">{{ t('Export as') }}</p>
              <div class="flex flex-row items-center gap-2">
                <button
                  id="download-qr-image-button-png"
                  class="button"
                  @click="downloadQRImageAsPng"
                  :aria-label="t('Download QR Code as PNG')"
                >
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    width="24"
                    height="24"
                    viewBox="0 0 24 24"
                  >
                    <g
                      fill="none"
                      stroke="currentColor"
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                    >
                      <path d="M14 3v4a1 1 0 0 0 1 1h4" />
                      <path
                        d="M5 12V5a2 2 0 0 1 2-2h7l5 5v4m1 3h-1a2 2 0 0 0-2 2v2a2 2 0 0 0 2 2h1v-3M5 18h1.5a1.5 1.5 0 0 0 0-3H5v6m6 0v-6l3 6v-6"
                      />
                    </g>
                  </svg>
                </button>
                <button
                  id="download-qr-image-button-svg"
                  class="button"
                  @click="downloadQRImageAsSvg"
                  :aria-label="t('Download QR Code as SVG')"
                >
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    width="24"
                    height="24"
                    viewBox="0 0 24 24"
                  >
                    <g
                      fill="none"
                      stroke="currentColor"
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                    >
                      <path d="M14 3v4a1 1 0 0 0 1 1h4" />
                      <path
                        d="M5 12V5a2 2 0 0 1 2-2h7l5 5v4M4 20.25c0 .414.336.75.75.75H6a1 1 0 0 0 1-1v-1a1 1 0 0 0-1-1H5a1 1 0 0 1-1-1v-1a1 1 0 0 1 1-1h1.25a.75.75 0 0 1 .75.75m3-.75l2 6l2-6m6 0h-1a2 2 0 0 0-2 2v2a2 2 0 0 0 2 2h1v-3"
                      />
                    </g>
                  </svg>
                </button>
              </div>
            </div>
          </div>
        </div>
        <div id="settings" class="flex w-full grow flex-col items-start gap-8 text-start">
          <div>
            <label for="preset-selector">{{ t('Preset') }}</label>
            <div class="flex flex-row items-center justify-center gap-2">
              <select
                id="preset-selector"
                class="secondary-button cursor-pointer text-start"
                :aria-label="t('QR code preset')"
                v-model="selectedPreset"
              >
                <option
                  v-if="!(selectedPreset.name && selectedPreset.name === defaultPreset.name)"
                  :key="`custom-preset`"
                  :value="selectedPreset"
                  disabled
                >
                  {{ selectedPreset.name }}
                </option>
                <option v-for="(preset, index) in allPresets" :key="index" :value="preset">
                  {{ preset.name }}
                </option>
              </select>
              <button
                class="icon-button"
                @click="randomizeStyleSettings"
                :aria-label="t('Randomize style')"
              >
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  width="40"
                  height="32"
                  viewBox="0 0 640 512"
                >
                  <path
                    fill="#888888"
                    d="M274.9 34.3c-28.1-28.1-73.7-28.1-101.8 0L34.3 173.1c-28.1 28.1-28.1 73.7 0 101.8l138.8 138.8c28.1 28.1 73.7 28.1 101.8 0l138.8-138.8c28.1-28.1 28.1-73.7 0-101.8L274.9 34.3zM200 224a24 24 0 1 1 48 0a24 24 0 1 1-48 0zM96 200a24 24 0 1 1 0 48a24 24 0 1 1 0-48zm128 176a24 24 0 1 1 0-48a24 24 0 1 1 0 48zm128-176a24 24 0 1 1 0 48a24 24 0 1 1 0-48zm-128-80a24 24 0 1 1 0-48a24 24 0 1 1 0 48zm96 328c0 35.3 28.7 64 64 64h192c35.3 0 64-28.7 64-64V256c0-35.3-28.7-64-64-64H461.7c11.6 36 3.1 77-25.4 105.5L320 413.8V448zm160-120a24 24 0 1 1 0 48a24 24 0 1 1 0-48z"
                  />
                </svg>
              </button>
            </div>
          </div>
          <div class="w-full">
            <div class="mb-2 flex flex-row items-center gap-2">
              <label for="url">
                {{ t('URL to link to') }}
              </label>
              <button class="icon-button flex flex-row items-center" @click="getShortLink">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="-139 51 512 512">
                  <g fill="#FFFFFF">
                    <path class="st0" d="M-115.3,136.1c-6.5-35.3,25.7-70.2,61.4-67c33.8,1.1,60.6,35.2,54.4,68.2c16.2,15.9,31.9,32.5,48.4,48
                      c10.5-22.6,21-45.2,31.7-67.7c6.5-12.8,23.9-17.9,36.2-10.3c8,5.8,14.6,13.4,21.5,20.5c9,9.9,8.5,26.1-0.9,35.6
                      c-52.9,53.1-105.9,106.1-159,159c-9.6,9.6-26.3,9.9-36.2,0.5c-6.7-6.9-14.3-13.2-19.9-21.1c-7.5-12.3-2.4-29.7,10.4-36.2
                      c22.5-10.8,45.1-21.2,67.7-31.7c-15.5-16.6-32-32.2-48-48.4C-78.8,191.3-111.6,167.8-115.3,136.1z"/>
                    <path class="st0" d="M66.2,252.2c20.1-19.9,55.7-20.1,75.9-0.4c18.1,17.9,36.2,35.8,54,54.1c21.6,23.1,16.9,63.8-9.1,81.6
                      c-6-6.1-12.3-11.9-17.9-18.3c15.7-8.9,20.6-32.1,8.5-45.9c-14.9-15.4-30.3-30.4-45.4-45.6c-6.8-7.1-14.4-14.8-24.9-15.2
                      c-21.7-2.9-40.8,21.8-31.9,42.1c6.6,13.2,19.7,21.5,29.1,32.7c-3.2,9.7-5.5,19.9-5.3,30.2c-12.7-12.2-25.1-24.7-37.4-37.4
                      C42,308.6,44.1,271.5,66.2,252.2z"/>
                    <path class="st0" d="M134.9,403.1c-21.6-23.1-16.9-63.8,9.1-81.6c6,6.1,12.3,11.9,17.9,18.3c-15.7,8.9-20.6,32.1-8.5,45.9
                      c14.9,15.4,30.3,30.4,45.4,45.6c6.8,7.1,14.5,14.8,24.9,15.2c21.7,2.9,40.8-21.8,31.9-42.1c-6.6-13.2-19.7-21.5-29.1-32.7
                      c3.2-9.7,5.5-19.9,5.3-30.2c12.7,12.2,25.1,24.7,37.4,37.4c19.9,21.5,17.8,58.6-4.3,77.9c-20.1,19.9-55.6,20.1-75.9,0.4
                      C170.8,439.3,152.6,421.4,134.9,403.1z"/>
                    <path class="st0" d="M283.1,454.8c9.9-9.8,19.5-19.8,29.6-29.3c9.2,9.2,20.2,18.2,23.2,31.6c5.3,23.9,9.2,48,13.9,72
                      c1.2,5.3-4.3,10.2-9.4,8.5c-24.7-5.1-49.8-8.5-74.2-14.8c-12-3.9-19.9-14.1-28.6-22.6c9.8-9.8,19.6-19.7,29.5-29.4
                      C277.4,484.5,296.7,465,283.1,454.8z"/>
                  </g>
                </svg>
                <p>{{ t('Get short link') }}</p>
              </button>
            </div>
            <textarea
              name="url"
              class="text-input"
              id="url"
              rows="1"
              :placeholder="t('URL to link to')"
              v-model="url"
            />
          </div>
          <div class="w-full">
            <label for="data">
              {{ t('Data to encode') }}
            </label>
            <textarea
              name="data"
              class="text-input"
              id="data"
              rows="1"
              :placeholder="t('data to encode e.g. a URL or a string')"
              v-model="data"
            />
          </div>
          <div class="w-full">
            <div class="mb-2 flex flex-row items-center gap-2">
              <label for="image-url">
                {{ t('Logo image URL') }}
              </label>
              <button class="icon-button flex flex-row items-center" @click="uploadImage">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
                  <g
                    fill="none"
                    stroke="currentColor"
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                  >
                    <path d="M14 3v4a1 1 0 0 0 1 1h4" />
                    <path
                      d="M17 21H7a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h7l5 5v11a2 2 0 0 1-2 2zm-5-10v6"
                    />
                    <path d="M9.5 13.5L12 11l2.5 2.5" />
                  </g>
                </svg>
                <p>{{ t('Upload image') }}</p>
              </button>
              <button class="icon-button flex flex-row items-center" @click="setImageToPABLogo">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 363.2 308.3">
                  <g fill="#FFFFFF">
                    <g>
                      <path class="st0" d="M87.8,215.1c0-0.3,0.3-0.6,0.6-0.6h7.9c4.1,0,7.3,3.3,7.3,7.3c0,4-3.3,7.3-7.3,7.3h-3.4v7.7
                        c0,0.3-0.3,0.6-0.6,0.6h-3.9c-0.3,0-0.6-0.3-0.6-0.6V215.1z M96,224.4c1.4,0,2.5-1.1,2.5-2.5s-1.1-2.5-2.5-2.5h-3.1v5H96z"/>
                      <path class="st0" d="M112.7,236.5l10.3-22.2c0.1-0.2,0.3-0.3,0.6-0.4h0.3c0.2,0,0.4,0.1,0.6,0.4l10.3,22.2
                        c0.2,0.3,0.1,0.6-0.2,0.8c-0.1,0.1-0.2,0.1-0.3,0.1h-3.7c-0.6,0-0.9-0.2-1.2-0.8l-1.2-2.5h-8.9l-1.2,2.6c-0.2,0.5-0.7,0.8-1.2,0.8
                        h-3.6c-0.3,0.1-0.6-0.1-0.7-0.4C112.5,236.9,112.6,236.7,112.7,236.5 M126.2,229.5l-2.5-5.4l-2.5,5.4H126.2z"/>
                      <path class="st0" d="M164.4,215.1c0-0.3,0.3-0.6,0.6-0.6h7.5c3.6-0.3,6.8,2.4,7.1,6c-0.3,2.3-1.7,4.2-3.8,5.2
                        c2.4,0.7,4.2,2.9,4.4,5.4c-0.2,3.7-3.3,6.5-7,6.4c-0.1,0-0.1,0-0.2,0h-8.1c-0.3,0-0.6-0.3-0.6-0.6V215.1z M172.2,223.7
                        c1.3,0,2.3-1.1,2.3-2.4v0c0-1.2-1-2.2-2.2-2.2c0,0-0.1,0-0.1,0h-2.5v4.6H172.2z M172.6,232.7c1.4,0,2.5-1.1,2.5-2.5
                        c-0.2-1.4-1.3-2.4-2.7-2.3h-2.7v4.8L172.6,232.7z"/>
                      <path class="st0" d="M194.4,215.1c0-0.3,0.3-0.6,0.6-0.6h3.9c0.3,0,0.6,0.3,0.6,0.6v21.8c0,0.3-0.3,0.6-0.6,0.6H195
                        c-0.3,0-0.6-0.3-0.6-0.6V215.1z"/>
                      <path class="st0" d="M215.1,215.1c0-0.3,0.3-0.6,0.6-0.6h7.5c3.6-0.3,6.8,2.5,7.1,6.1c0,0,0,0,0,0c-0.3,2.3-1.7,4.2-3.8,5.2
                        c2.4,0.7,4.2,2.9,4.4,5.4c-0.2,3.7-3.3,6.5-7,6.4c-0.1,0-0.1,0-0.2,0h-8c-0.3,0-0.6-0.3-0.6-0.6V215.1z M222.8,223.8
                        c1.3,0,2.3-1.1,2.3-2.4c0-1.2-1-2.2-2.2-2.2c0,0-0.1,0-0.1,0h-2.5v4.6H222.8z M223.2,232.8c1.3,0,2.4-1.1,2.4-2.5
                        c0-1.3-1.1-2.4-2.5-2.4c-0.1,0-0.1,0-0.2,0h-2.7v4.7L223.2,232.8z"/>
                      <path class="st0" d="M245,215.1c0-0.3,0.3-0.6,0.6-0.6h3.9c0.3,0,0.6,0.3,0.6,0.6v17.6h7.9c0.3,0,0.6,0.3,0.6,0.6v3.6
                        c0,0.3-0.3,0.6-0.6,0.6h-12.4c-0.3,0-0.6-0.3-0.6-0.6L245,215.1z"/>
                      <path class="st0" d="M271.7,215.1c0-0.3,0.3-0.6,0.6-0.6H286c0.3,0,0.6,0.3,0.6,0.6v3.6c0,0.3-0.3,0.6-0.6,0.6h-9.1v4.1h7.5
                        c0.3,0,0.6,0.3,0.6,0.6v3.6c0,0.3-0.3,0.6-0.6,0.6h-7.5v4.4h9.1c0.3,0,0.6,0.3,0.6,0.6v3.6c0,0.3-0.3,0.6-0.6,0.6h-13.6
                        c-0.3,0-0.6-0.3-0.6-0.6V215.1z"/>
                    </g>
                    <path class="st0" d="M282.4,178.4c0-0.1,0-0.3,0-0.4c0-0.1,0-0.2-0.1-0.3c0-0.1,0-0.2-0.1-0.3L257,100.8c-0.4-1.6-1.9-2.7-3.6-2.4
                      c-0.6,0-5.3-0.5-15.3-6.6c-10.3-6.5-20.3-9.8-29.7-9.8c-7.9-0.2-15.5,2.3-21.7,7.1c-16.4-14.4-37.5-22.4-59.4-22.4
                      c-1.8,0-3.6,0.1-5.4,0.2c-1.1,0.1-2.1,0.6-2.7,1.6c-0.2,0.3-17.3,25.7-41.4,60.6c-0.7,1-0.8,2.2-0.4,3.3c0.7,1.8,2.7,2.7,4.5,2
                      c3-1.1,5.9-2.2,8.8-3.1c-2.8,4.9-7.1,12.7-13.7,25.1c-0.6,1.1-0.6,2.5,0.1,3.5c1,1.7,3.2,2.2,4.9,1.1c6.8-4.3,13.3-7.9,19.6-10.8
                      c-2.2,4.2-4.3,8-6.4,11.7C92,167.9,88.6,174,85,181.7c-0.2,0.5-0.4,1-0.4,1.5c0,1.9,1.6,3.5,3.5,3.5c0.2,0,0.3,0,0.5,0
                      c17.2-2.3,35.4-4.3,54.1-5.9c15-1.2,29.7-2.2,43.7-2.7l92.3,3.8c1.6,0.1,3-1,3.5-2.4c0-0.1,0-0.1,0.1-0.2c0-0.1,0-0.2,0.1-0.3
                      c0-0.1,0-0.2,0.1-0.4c0,0,0,0,0-0.1C282.4,178.5,282.4,178.4,282.4,178.4z M124.1,73.9c1.1,0,2.2-0.1,3.3-0.1
                      c20.6,0,40.5,7.7,55.8,21.5c-0.2,23.6-0.3,40.2-0.3,51.9c-0.6-1.2-1.3-2.5-2-3.7c-4.4-7.6-10.6-13.9-18.1-18.4
                      c-10-5.6-21.3-8.4-32.7-8.1c-13.7,0.3-27.3,2.7-40.3,7.2C107.5,98.5,120.5,79.2,124.1,73.9z M94.5,178.8c-0.1,0-0.2,0-0.3,0
                      c2.5-4.8,4.8-9,7.2-13.4c3.1-5.6,6.4-11.5,10-18.8c0.1-0.2,0.1-0.3,0.2-0.5c5.2-1.9,10.3-3.3,15-4.2c9.6-2.1,19.7-1.4,28.9,2
                      c4.9,1.9,9.3,4.8,13.1,8.4c2,1.9,3.8,4,5.5,6.2C151.2,149.8,106.6,172.3,94.5,178.8z M173.7,147.3c-4.5-4.2-9.7-7.6-15.4-9.9
                      c-10.5-3.9-21.9-4.8-32.8-2.4c-12.7,2.7-24.9,7.3-36.2,13.7c6.8-12.6,10-18,11.3-20.2c0,0,0,0,0-0.1c10.8-2.8,20.6-4.3,29.4-4.3
                      c10.2-0.3,20.2,2.2,29.1,7.1c6.5,3.9,11.9,9.4,15.6,15.9c0.5,0.9,1,1.8,1.5,2.8C175.4,149,174.5,148.2,173.7,147.3z M258.4,162.8
                      c-6.9-3.4-14.1-6.3-21.5-8.7c-7.4-2.5-15.1-3.9-22.9-4v0c-6.8-0.2-13.4,1.6-19.1,5.2c-1.8,1.3-3.5,2.8-5,4.4l0.2-64.4
                      c1.2-1,2.6-2,4-2.7c4.4-2.4,9.2-3.6,14.2-3.5c9.3,0.4,18.3,3.4,25.9,8.7l0.1,0c5.1,3.4,10.7,5.8,16.6,7.3l21.5,65.2
                      C268.8,168.2,263.9,165.5,258.4,162.8z"/>
                  </g>
                </svg>
                <p>{{ t('Set image to our logo') }}</p>
              </button>
            </div>
            <textarea
              name="image-url"
              class="text-input"
              id="image-url"
              rows="1"
              :placeholder="t('Logo image URL')"
              v-model="image"
            />
          </div>
          <div id="color-settings" class="flex w-full flex-row flex-wrap gap-4">
            <div class="flex flex-row items-center gap-2">
              <label for="background-color">{{ t('Background color') }}</label>
              <input
                id="background-color"
                type="color"
                class="color-input"
                v-model="styleBackground"
              />
            </div>
            <div class="flex flex-row items-center gap-2">
              <label for="dots-color">{{ t('Dots color') }}</label>
              <input id="dots-color" type="color" class="color-input" v-model="dotsOptionsColor" />
            </div>
            <div class="flex flex-row items-center gap-2">
              <label for="corners-square-color">{{ t('Corners Square color') }}</label>
              <input
                id="corners-square-color"
                type="color"
                class="color-input"
                v-model="cornersSquareOptionsColor"
              />
            </div>
            <div class="flex flex-row items-center gap-2">
              <label for="corners-dot-color">{{ t('Corners Dot color') }}</label>
              <input
                id="corners-dot-color"
                type="color"
                class="color-input"
                v-model="cornersDotOptionsColor"
              />
            </div>
          </div>
          <div class="w-full">
            <label for="width">
              {{ t('Width (px)') }}
            </label>
            <input
              class="text-input"
              id="width"
              type="number"
              placeholder="width in pixels"
              v-model="width"
            />
          </div>
          <div class="w-full">
            <label for="height">
              {{ t('Height (px)') }}
            </label>
            <input
              class="text-input"
              id="height"
              type="number"
              placeholder="height in pixels"
              v-model="height"
            />
          </div>
          <div class="w-full">
            <label for="margin">
              {{ t('Margin (px)') }}
            </label>
            <input class="text-input" id="margin" type="number" placeholder="0" v-model="margin" />
          </div>
          <div class="w-full">
            <label for="image-margin">
              {{ t('Image margin (px)') }}
            </label>
            <input
              class="text-input"
              id="image-margin"
              type="number"
              placeholder="0"
              v-model="imageMargin"
            />
          </div>
          <div class="w-full">
            <label for="border-radius">
              {{ t('Border radius (px)') }}
            </label>
            <input
              class="text-input"
              id="border-radius"
              type="number"
              placeholder="24"
              v-model="styleBorderRadius"
            />
          </div>
          <div
            id="dots-squares-settings"
            class="mb-4 flex w-full flex-col flex-wrap gap-6 md:flex-row"
          >
            <fieldset class="flex-1" role="radiogroup" tabindex="0">
              <legend>{{ t('Dots type') }}</legend>
              <div
                class="radiogroup"
                v-for="type in [
                  'dots',
                  'rounded',
                  'classy',
                  'classy-rounded',
                  'square',
                  'extra-rounded'
                ]"
                :key="type"
              >
                <input
                  :id="'dotsOptionsType-' + type"
                  type="radio"
                  v-model="dotsOptionsType"
                  :value="type"
                />
                <label :for="'dotsOptionsType-' + type">{{ t(type) }}</label>
              </div>
            </fieldset>
            <fieldset class="flex-1" role="radiogroup" tabindex="0">
              <legend>{{ t('Corners Square type') }}</legend>
              <div
                class="radiogroup"
                v-for="type in ['dot', 'square', 'extra-rounded']"
                :key="type"
              >
                <input
                  :id="'cornersSquareOptionsType-' + type"
                  type="radio"
                  v-model="cornersSquareOptionsType"
                  :value="type"
                />
                <label :for="'cornersSquareOptionsType-' + type">{{ t(type) }}</label>
              </div>
            </fieldset>
            <fieldset class="flex-1" role="radiogroup" tabindex="0">
              <legend>{{ t('Corners Dot type') }}</legend>
              <div class="radiogroup" v-for="type in ['dot', 'square']" :key="type">
                <input
                  :id="'cornersDotOptionsType-' + type"
                  type="radio"
                  v-model="cornersDotOptionsType"
                  :value="type"
                />
                <label :for="'cornersDotOptionsType-' + type">{{ t(type) }}</label>
              </div>
            </fieldset>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<style lang="postcss" scoped>
label,
legend {
  @apply text-gray-700 dark:text-gray-100 text-lg font-semibold;
}

.text-input {
  @apply bg-zinc-50 dark:bg-zinc-800 text-zinc-900 dark:text-zinc-100;
  @apply shadow hover:shadow-md transition-shadow rounded-lg;
  @apply outline-none focus-visible:ring-1 focus-visible:ring-zinc-700 dark:focus-visible:ring-zinc-200;
  @apply resize-none appearance-none ms-1 p-4 rounded w-full;
}

input[type='color'] {
  @apply outline-none focus-visible:ring-1 focus-visible:ring-zinc-700 dark:focus-visible:ring-zinc-200;
  @apply bg-transparent shadow p-0 border rounded box-border text-zinc-700 dark:text-zinc-100 focus-visible:shadow;
}

input[type='radio'] {
  @apply outline-none focus-visible:ring-1 focus-visible:ring-zinc-700 dark:focus-visible:ring-zinc-200;
  @apply m-3;
}

.button {
  @apply bg-zinc-100 dark:bg-zinc-700 text-zinc-900 dark:text-zinc-200;
  @apply shadow-sm hover:shadow p-2 focus-visible:shadow-md rounded-lg;
  @apply outline-none focus-visible:ring-1 focus-visible:ring-zinc-700 dark:focus-visible:ring-zinc-200;
}

.secondary-button {
  @apply bg-zinc-50 dark:bg-zinc-800 text-zinc-900 dark:text-zinc-100;
  @apply shadow hover:shadow-md transition-shadow rounded-lg;
  @apply outline-none focus-visible:ring-1 focus-visible:ring-zinc-700 dark:focus-visible:ring-zinc-200;
  @apply outline-none p-1.5;
}

.icon-button {
  @apply outline-none focus-visible:ring-1 focus-visible:ring-zinc-700 dark:focus-visible:ring-zinc-200;
  @apply outline-none hover:shadow rounded-sm;
}

.vertical-border {
  @apply h-6 bg-slate-300 dark:bg-slate-700 w-1;
}

.radiogroup {
  @apply flex flex-row items-center gap-1;
}

.radiogroup > label {
  @apply font-normal;
}
</style>
