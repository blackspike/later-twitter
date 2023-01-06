
<template lang="pug">
main.wrapper

  //- Header
  header.header
    h1.header__title 👋 Later Twitter
    .header__intro Generate a Twitter header image with a QR code to your Mastodon page

  //- Layout
  .layout

    //- Form
    form.row

      //- Handle
      .col
        label(for="handle") Username e.g. @blackspike
        input#handle(
          @blur="generateQR"
          @keyup="updateHandle"
          @paste="updateHandle"
          placeholder="@username"
          tabindex="1"
          v-model.trim="handle"
        )

      //- Server
      .col
        label(for="server") Server e.g. @mastodon.social
        input#server(
          @blur="generateQR"
          @keyup="updateServer"
          @paste="updateServer"
          placeholder="@mastodon.social"
          tabindex="2"
          v-model.trim="server"
        )

      //- Intro
      .col
        label(for="intro") Intro text
        input#intro(
          @keyup="updateIntro"
          @paste="updateIntro"
          placeholder="e.g. Find me at"
          tabindex="3"
          v-model.trim="intro"
        )

    //- Controls
    aside.controls

      //- Upload
      .buttons
        label.btn(for="uploader") Add image
        button(@click="deleteThing") Delete item
        input#uploader(@change="uploadToCanvas" type="file" hidden)

      //- Gradients
      .grad-buttons
        button.grad-button(v-for="(grad, index) in gradients"
        :aria-label="`Gradient ${index +1}`"
        @click="addGrad(grad, true)"
        )
          img.grad-button__image(:src="`/thumbnails/${grad}`" alt="" height="20" width="20")

      //- Download
      button.download(@click="saveImage") Download Image

    //- Canvas
    .canvas-wrapper
      canvas#canvas

    //- Footer
    footer.footer
      a.footer__link(href="https://ingradients.net/") Gradients by ingradients
      a.footer__link(href="https://plausible.io/latertwitter.cyou/") Stats
      a.footer__link(href="https://github.com/blackspike/later-twitter") GitHub
      a.footer__link(href="https://mastodon.cloud/@felixthehat") By @felixthehat

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

// Concatenate server and handle
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
  width: 720,
  margin: 0,
  color: {
    dark: "#fff",
    light: "#ffffff00"
  }
}

const generateQR = async () => {
  try {

    // Remove old QR
    const objs = canvas.getObjects()
    const oldQr = objs.find(el => el.name === 'qr')
    if (oldQr) canvas.remove(oldQr)

    // Create QR
    qr.value = await QRCode.toDataURL(url.value, opts)

    // Add QR
    fabric.Image.fromURL(qr.value, function (oImg) {
      oImg.scale(.25)
      oImg.set({
        left: 850, top: 154, name: 'qr',
      })
      canvas.add(oImg)
    })
  } catch (err) {
    console.error(err)
  }
}

/* Fabric.js
============================= */

//- Create and add canvas
let canvas = null

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

// Intro
let introText = new fabric.Text(intro.value, {
  left: 480, top: 300,
  fontSize: 30,
  ...textOptions
})
// Handle
let handleText = new fabric.Text(handle.value, {
  left: 475, top: 340,
  fontSize: 50,
  ...textOptions
})
// Server
let serverText = new fabric.Text(server.value, {
  left: 480, top: 408,
  fontSize: 30,
  fontWeight: '700',
  ...textOptions
})

// Update canvas text
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

// Card background
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

  // Disallow text to be squished
  introText.setControlsVisibility(textControlOptions)
  handleText.setControlsVisibility(textControlOptions)
  serverText.setControlsVisibility(textControlOptions)

  // Add random bg
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

  // Remove old
  const objs = canvas.getObjects()
  const oldGrad = objs.find(el => el.name === 'grad')
  if (oldGrad) canvas.remove(oldGrad)

  // Open gradient from public
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

//- Upload  image to canvas

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
      canvas.add(img).renderAll()
      canvas.sendToBack(img)
      canvas.bringForward(img)
    }
  }
  reader.readAsDataURL(e.target.files[0])
}

</script>

<style lang="scss">
@import "https://unpkg.com/open-props";

#app {
  inline-size: 100%;
}

.wrapper {
  inline-size: min(90vw, 1200px);
  margin-inline: auto;
}

.layout {
  align-items: start;
  display: grid;
  gap: var(--size-4);
  grid-template-areas: 'input' 'controls' 'image' 'footer';
}

// Header

.header {
  align-items: center;
  border-bottom: 1px solid var(--gray-3);
  display: flex;
  gap: var(--size-6);
  justify-content: space-between;
  margin-block-end: var(--size-8);
  padding-block: var(--size-4);

  &__title {
    font-size: var(--font-size-4);
    letter-spacing: var(--font-letterspacing-0);
    margin: 0;
  }

  &__intro {
    color: var(--gray-6);
    display: none;
    font-size: var(--font-size-1);
  }
}

@media only screen and (min-width: 60rem) {
  .header__intro {
    display: block;
  }
}

// Controls

.controls {
  display: grid;
  gap: var(--size-4);
  grid-area: controls;
  grid-template-areas: 'download' 'grads' 'buttons';
  inline-size: 100%;
  padding-block: var(--size-5) var(--size-3);
}

@media only screen and (min-width: 45rem) {
  .controls {
    grid-template-areas: 'buttons download' 'grads grads';
    grid-template-columns: 1fr 1fr;
  }
}

@media only screen and (min-width: 60rem) {
  .controls {
    gap: var(--size-7);
    grid-template-areas: 'buttons grads download';
    grid-template-columns: 1fr 2fr 1fr;
  }
}

// Grad buttons

.grad-buttons {
  align-items: center;
  display: grid;
  gap: var(--size-2);
  grid-area: grads;
  grid-auto-flow: column;
}

.grad-button {
  aspect-ratio: 1/1;
  border: 2px solid var(--gray-9);
  block-size: 100%;
  inline-size: 100%;
  overflow: hidden;
  padding: 0;

  &__image {
    height: 100%;
    inline-size: 100%;
    object-fit: cover;
  }

  &:hover {
    border-color: var(--brand);
  }
}


@media only screen and (min-width: 60rem) {
  .grad-buttons {
    margin-inline: auto;
  }
}

.buttons {
  display: grid;
  gap: var(--size-3);
  grid-area: buttons;
  grid-auto-flow: column;
}

.download {
  background-color: var(--brand);
  grid-area: download;

  &:hover {
    background-color: var(--gray-9);
  }
}


// Rows n cols

.col {
  display: flex;
  flex-direction: column;
  flex: 1;
  gap: var(--size-2);
  inline-size: 100%;
}

.row {
  display: flex;
  flex-wrap: wrap;
  gap: var(--size-5);
  inline-size: 100%;
}

@media only screen and (min-width: 60rem) {
  .row {
    min-height: 100%;
  }
}

// Canvas

.canvas-wrapper {
  background-color: var(--gray-4);
  border-radius: var(--radius-2);
  display: flex;
  grid-area: image;
  inline-size: 100%;
  justify-content: center;
  padding: var(--size-2);

  .canvas-container,
  canvas {
    aspect-ratio: 3/1;
    block-size: auto !important;
    max-width: calc(100vw - calc(var(--size-7)));
    inline-size: 100% !important;
  }
}

// Footer

.footer {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  grid-area: footer;
  gap: var(--size-3);
  padding-block: var(--size-7);

  &__link {
    color: var(--gray-6);
    display: inline-block;
    inline-size: auto;


    &:hover {
      color: var(--brand);
    }
  }
}

@media only screen and (min-width: 60rem) {
  .footer {
    flex-direction: row;

    &__link:last-of-type {
      flex: 2;
      text-align: end;
    }
  }
}
</style>