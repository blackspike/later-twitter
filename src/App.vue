
<template lang="pug">
main.wrapper

  header.header
    h1.header__title 👋 Later Twitter
    .header__intro Generate a Twitter header image with a QR code to your Mastodon page


  .layout

    //- form
    form.input
      .double
        .row
          label(for="handle") Username e.g. @blackspike
          input#handle(tabindex="2" v-model="handle" placeholder="@username" @keyup="updateHandle" @blur="generateQR")
        .row
          label(for="server") Server e.g. @mastodon.social
          input#browser(tabindex="3" v-model="server" placeholder="@mastodon.social" @keyup="updateServer" @blur="generateQR")
        .row.row--max
          label(for="intro") Intro text
          input#intro(tabindex="1" v-model="intro" placeholder="e.g. Find me at" @keyup="updateIntro")

    //- Controls
    aside.controls
      .buttons
        label.btn(for="uploader") Add image
        button(@click="deleteThing") Delete item
        input#uploader(@change="uploadToCanvas" type="file" hidden)

      .grad-buttons
        button.grad-button(v-for="grad in gradients" :aria-label="grad" @click="addGrad(grad, true)")
          img.grad-button__image(:src="`/thumbnails/${grad}`" alt="" height="20" width="20")

      .download
        button.download__btn(@click="saveImage") Download Image

    //- Fabric
    .fabric

      //- Canvas
      .canvas-wrapper
        canvas#canvas

    //- output
    .footer
      a.footer__link(href="https://ingradients.net/") Gradients by ingradients
      a.footer__link(href="https://mastodon.cloud/@felixthehat") App by @felixthehat

</template>

<script setup>
import { fabric } from 'fabric'
import QRCode from 'qrcode'
import { ref, onMounted, computed } from 'vue'

/* Setups
============================= */
const intro = ref('Follow me on Mastodon')
const handle = ref('@username')
const server = ref('@mastodon.social')
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

const opts = {
  type: 'image/png',
  width: 180,
  margin: 0,
  color: {
    dark: "#fff",
    light: "#151f2b"
  }
}
const generateQR = async () => {
  try {
    // canvas.remove(oImg)
    const objs = canvas.getObjects()
    const oldQr = objs.find(el => el.name === 'qr')
    if (oldQr) canvas.remove(oldQr)

    // create qr
    qr.value = await QRCode.toDataURL(url.value, opts)

    // Add qr
    fabric.Image.fromURL(qr.value, function (oImg) {
      oImg.set({
        left: 850, top: 154, height: 180, width: 180, name: 'qr', rx: 8,
        ry: 8,
      })
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
const textControlOptions = {
  ml: false,
  mt: false,
  mr: false,
  mb: false,
  mtr: false
}
// text
let introText = new fabric.Text(intro.value, {
  left: 480, top: 300,
  fontSize: 30,
  ...textOptions
})
let handleText = new fabric.Text(handle.value, {
  left: 475, top: 340,
  fontSize: 50,
  ...textOptions
})
// text
let serverText = new fabric.Text(server.value, {
  left: 480, top: 408,
  fontSize: 30,
  fontWeight: '700',
  ...textOptions
})

const updateIntro = async () => {
  introText.set('text', intro.value)
  canvas.renderAll()
}
const updateHandle = async () => {
  handleText.set('text', handle.value)
  canvas.renderAll()
}
const updateServer = async () => {
  serverText.set('text', server.value)
  canvas.renderAll()
}

// Rect
const rect = new fabric.Rect({
  height: 344, width: 608, fill: '#151f2b', left: 446, top: 128, rx: 8, ry: 8
})

/* Mounted
============================= */

onMounted(() => {
  canvas = new fabric.Canvas('canvas', {
    preserveObjectStacking: true,
  })
  canvas.setHeight(500)
  canvas.setWidth(1500)
  canvas.backgroundColor = '#eee'

  canvas.add(rect, introText, handleText, serverText)

  introText.setControlsVisibility(textControlOptions)
  handleText.setControlsVisibility(textControlOptions)
  serverText.setControlsVisibility(textControlOptions)

  addGrad(randomGrad, true)

})

//- Save image

const saveImage = () => {

  let link = document.createElement('a')
  link.download = `later-twitter-${handle.value}.jpg`
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

//- Add gradient

const addGrad = (newGrad, moveable) => {

  const objs = canvas.getObjects()

  const oldGrad = objs.find(el => el.name === 'grad')
  if (oldGrad) canvas.remove(oldGrad)


  fabric.Image.fromURL(`/gradients/${newGrad}`, (grad) => {
    if (moveable) {
      grad.set('selectable', false)
      grad.set('name', 'grad')
      grad.setControlsVisibility({
        tr: false,
        bl: false,
        ml: false,
        mt: false,
        mr: false,
        mb: false,
        mtr: false
      })
    }
    canvas.add(grad)
    canvas.sendToBack(grad)
  })

}

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
      canvas.add(img).renderAll()
      canvas.sendToBack(img)
      canvas.bringForward(img)
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

.wrapper {
  inline-size: min(90vw, 1200px);
  margin-inline: auto;
}

.layout {
  display: grid;
  align-items: start;
  gap: var(--size-4);
  grid-template-areas: 'input' 'controls' 'image' 'footer';
}

.header {
  padding-block: var(--size-4);
  margin-block-end: var(--size-8);
  border-bottom: 1px solid #ddd;
  display: flex;
  gap: var(--size-6);
  justify-content: space-between;
  align-items: center;

  &__title {
    margin: 0;
    font-size: var(--font-size-3);
  }

  &__intro {
    font-size: var(--font-size-1);
    display: none;
  }
}

@media only screen and (min-width: 60rem) {
  .header__intro {
    display: block;
  }

}

.controls {
  grid-area: controls;
  display: grid;
  gap: var(--size-4);
  grid-template-areas: 'download' 'grads' 'buttons';
  width: 100%;
  padding-block: var(--size-5) var(--size-3);
  white-space: nowrap;
}

@media only screen and (min-width: 45rem) {
  .controls {
    grid-template-areas: 'buttons download' 'grads grads';
    grid-template-columns: 1fr 1fr;
  }

}


@media only screen and (min-width: 60rem) {
  .controls {
    gap: var(--size-3);
    grid-template-areas: 'buttons grads download';
    grid-template-columns: 1fr 2fr 1fr;
    gap: var(--size-10);
  }

}

.grad-buttons {
  grid-area: grads;
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
  aspect-ratio: 1/1;

  &__image {
    height: 100%;
    width: 100%;
    object-fit: cover;
  }

}

@media only screen and (min-width: 60rem) {
  .grad-buttons {
    margin-inline: auto;
  }

}

.buttons {
  grid-area: buttons;
  display: grid;
  gap: var(--size-3);
  grid-auto-flow: column;
}

.download {
  display: flex;
  justify-content: flex-end;
  grid-area: download;

  &__btn {
    border-radius: var(--radius-round);
    background-color: var(--brand);
    overflow: hidden;
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

@media only screen and (min-width: 60rem) {

  .row,
  textarea {
    min-height: 100%;
  }
}

.canvas-wrapper {
  width: 100%;
  border-radius: 8px;
  background-color: #ddd;
  padding: .5rem;
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

.preview {
  height: 100%;
  width: 100%;
  border-radius: 8px;
  background-color: #444;
}

.footer {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  gap: var(--size-3);
  padding-block: var(--size-7);

  &__link {
    display: inline-block;
    width: auto;
    color: #444;

    &:hover {
      color: var(--brand);
    }
  }
}

@media only screen and (min-width: 60rem) {

  .footer {
    flex-direction: row;
  }
}
</style>