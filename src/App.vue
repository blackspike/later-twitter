
<template lang="pug">
main.wrapper
  h1
    a(href="https://blackspike.com/") By BlackSpike

  .layout

    //- form
    form.input
      .double
        .row
          label(for="intro") Intro
          input#intro(tabindex="1" v-model="intro" placeholder="e.g. Find me at" @blur="updateIntro")
        .row
          label(for="handle") Username e.g. @blackspike
          input#handle(tabindex="2" v-model="handle" placeholder="@username" @blur="updateHandle")
        .row
          label(for="server") Server e.g. @mastodon.social
          input#browser(tabindex="3" v-model="server" placeholder="@mastodon.social" @blur="updateServer")



    //- Fabric
    .fabric
      //- h2 Preview
      //- Controls
      aside.controls
        input#uploader(@change="uploadToCanvas" type="file" hidden)
        .hstack.gap-2
          label.btn(for="uploader") Add image
          button(@click="deleteThing") Delete selected

        .grad-buttons
          button.grad-button(v-for="grad in gradients" :aria-label="grad")
            img.grad-button__image(:src="`/thumbnails/${grad}`" alt="" height="20" width="20")

        button(@click="saveImage") Download Image

      //- Canvas
      .canvas-wrapper
        canvas#canvas



    //- output
    .footer

      .row
        h4
          a(:href="url") {{url}}

</template>

<script setup>
import { fabric } from 'fabric'
import QRCode from 'qrcode'
import { ref, onMounted, computed } from 'vue'

/* Setups
============================= */
const intro = ref('Follow me on Mastodon')
const handle = ref('@gregorysherrow')
const server = ref('@writing.exchange')
const gradients = [
  'top-white-ptarmigan.webp',
  'striped-ivory-alligator.webp',
  'historic-fuchsia-angelfish.webp',
  'irrelevant-amaranth-guanaco.webp',
  'fluttering-gray-vulture.webp',
  'busy-orange-antelope.webp',
  'cold-orange-constrictor.webp',
  'applicable-harlequin-llama.webp',
  'ambitious-rose-orangutan.webp',
  'accused-white-marlin.webp',
]
const randomGrad = gradients[Math.floor(Math.random() * gradients.length)]

const url = computed(() => {
  let urlStr = ''
  if (server.value.startsWith('@')) {
    urlStr = server.value.replace('@', '')
  } else {
    urlStr = server.value
  }
  return `https://${urlStr}/${handle.value}`
})

/* QR
============================= */
const qr = ref('')

const generateQR = async () => {
  try {
    // canvas.remove(oImg)
    const objs = canvas.getObjects()
    const oldQr = objs.find(el => el.name === 'qr')
    if (oldQr) canvas.remove(oldQr)

    // create qr
    qr.value = await QRCode.toDataURL(url.value, {
      width: 180
    })

    // Add qr
    fabric.Image.fromURL(qr.value, function (oImg) {
      oImg.set({ left: 850, top: 154, height: 180, width: 180, name: 'qr' })
      canvas.add(oImg)
    })
  } catch (err) {
    console.error(err)
  }
}

/* Fabric
============================= */

//- Create and add canvas
let canvas = null


const group = new fabric.Group([], {
  left: 150,
  top: 100,
})
const textOptions = {
  fontFamily: 'Arial,sans-serif',
  fill: 'white'
}

// text
let introText = new fabric.Text(intro.value, {
  left: 480, top: 310,
  fontSize: 26,
  ...textOptions
})
let handleText = new fabric.Text(handle.value, {
  left: 480, top: 345,
  fontSize: 40,
  ...textOptions
})
// text
let serverText = new fabric.Text(server.value, {
  left: 480, top: 400,
  fontSize: 30,
  fontWeight: '700',
  ...textOptions
})

const updateIntro = async () => {
  introText.set('text', intro.value)
  await generateQR()
  canvas.renderAll()
}
const updateHandle = async () => {
  handleText.set('text', handle.value)
  await generateQR()
  canvas.renderAll()
}
const updateServer = async () => {
  serverText.set('text', server.value)
  await generateQR()
  canvas.renderAll()
}

// Rect
const rect = new fabric.Rect({
  height: 344, width: 608, fill: '#151f2b', left: 446, top: 128, rx: 8, ry: 8
})

// 446
onMounted(() => {
  canvas = new fabric.Canvas('canvas', {
    preserveObjectStacking: true,
  })
  canvas.setHeight(500)
  canvas.setWidth(1500)
  canvas.backgroundColor = '#eee'

  canvas.add(rect)
  canvas.add(introText)
  canvas.add(handleText)
  canvas.add(serverText)
})
//- Save image

const saveImage = () => {

  let link = document.createElement('a')
  link.download = `verified-${handle.value}.jpg`
  link.href = canvas.toDataURL({
    format: 'jpeg',
    quality: 1,
    multiplier: 2
  })
  link.click()
}

//- Delete element

const deleteThing = () => {
  const activeObject = canvas.getActiveObjects()
  canvas.discardActiveObject()
  canvas.remove(...activeObject)
}

//- Add paw of approval

const addGrad = () => {
  fabric.Image.fromURL(`/gradients/${randomGrad}`, (grad) => {
    grad.set('selectable', false)
    grad.setControlsVisibility({
      tr: false,
      bl: false,
      ml: false,
      mt: false,
      mr: false,
      mb: false,
      mtr: false
    })
    canvas.add(grad)
    canvas.sendToBack(grad)
  })

}

addGrad()

//- Upload and add image to canvas

const uploadToCanvas = async (e) => {
  const reader = new FileReader()
  reader.onload = function (e) {
    let image = new Image()
    image.src = e.target.result
    image.onload = function () {
      let img = new fabric.Image(image)
      img.set({
        left: 0,
        top: 0
      })
      // img.scaleToHeight(500)
      canvas.add(img).setActiveObject(img).renderAll()
    }
  }
  reader.readAsDataURL(e.target.files[0])
}

// Upload to cloudflare


</script>

<style lang="scss">
@import "https://unpkg.com/open-props";


#app {
  width: 100%;
}

.layout {
  display: flex;
  flex-direction: column;
  gap: var(--size-4);
  align-items: start;
}

@media only screen and (min-width: 64rem) {
  .layout {
    display: grid;
    align-items: start;
    gap: var(--size-4);
    grid-template-areas: 'input' 'image' 'footer';
  }
}

.fabric {
  grid-area: image;
}

.footer {
  grid-area: footer;
}

.input {
  grid-area: input;
  display: flex;
  flex-direction: column;
  gap: var(--size-5);
}

.row {
  display: flex;
  flex-direction: column;
  gap: var(--size-2);
  width: 100%;
  flex: 1;
}

.double {
  display: flex;
  flex-wrap: wrap;
  gap: var(--size-5);
  width: 100%;
}

@media only screen and (min-width: 45rem) {

  .double {
    flex-wrap: nowrap;
  }

  .row,
  textarea {
    min-height: 100%;
  }
}

.button-list {
  display: flex;
  flex-wrap: wrap;
  gap: var(--size-2);
  width: 100%;
}

.canvas-wrapper {
  width: 100%;
  border-radius: 8px;
  background-color: #ddd;
  padding: 1rem;
  display: flex;
  justify-content: center;

  .canvas-container,
  canvas {
    max-width: calc(100vw - calc(var(--size-7)));
    aspect-ratio: 3/1;
    width: 100% !important;
    height: auto !important;
  }
}

.controls {
  display: flex;
  width: 100%;
  justify-content: space-between;
  padding-block: var(--size-5) var(--size-3);
}

.preview {
  height: 100%;
  width: 100%;
  border-radius: 8px;
  background-color: #444;
}

.grad-buttons {
  align-items: center;
  grid-auto-flow: column;
  display: grid;
  gap: var(--size-2);
}

.grad-button {
  padding: 0;
  height: 100%;
  width: 100%;
  border: 1px solid black;
  overflow: hidden;

  &__image {
    max-width: 32px;
    height: 100%;
    width: 100%;
    object-fit: cover;
  }
}
</style>