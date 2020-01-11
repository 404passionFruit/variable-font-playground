<template>
  <div class="container">
    <FileUploader
      @loading="onLoading"
      @axes="onAxes"
      @font="onFont"
    ></FileUploader>
    <div v-if="fontName && !fontFamily">Loading {{ fontName }}</div>
    <div v-if="fontName && fontFamily">
      {{ fontFamily }}
      <div ref="preview" :style="previewStyle" class="preview">
        {{ previewText }}
      </div>
    </div>
    <form v-if="fontFamily" action="">
      <label v-for="ax in axes" :key="ax.name.en">
        {{ ax.name.en }} {{ ax.value }}
        <input
          type="range"
          :min="ax.minValue"
          :max="ax.maxValue"
          v-model="ax.value"
        />
      </label>
    </form>
    <div v-if="fontFamily">
      <pre>
        <code>
          {{ previewStyle }}
        </code>
      </pre>
    </div>
  </div>
</template>

<script>
import Vue from 'vue'
import FileUploader from '~/components/FileUploader.vue'

export default {
  components: {
    FileUploader
  },
  data() {
    return {
      fontName: '',
      fontFamily: '',
      font: '',
      isPreviewSet: false,
      fallbackFamily: 'sans-serif',
      axes: [],
      previewText: ''
    }
  },
  computed: {
    previewStyle() {
      const axesValues = this.axes.reduce((result, ax) => {
        if (result) {
          result += ', '
        }
        result += `'${ax.tag}' ${ax.value}`
        return result
      }, '')
      return `
      font-family: '${this.fontFamily}', ${this.fallbackFamily};
      font-variation-settings: ${axesValues};
      `
    }
  },
  methods: {
    onAxes(vfont) {
      Vue.set(
        this,
        'axes',
        vfont.getAxes().map((a) => ({
          ...a,
          value: a.defaultValue
        }))
      )
      this.previewText = Object.values(vfont.glyphs.glyphs).reduce(
        (result, g) => {
          if (!g.unicode) return result
          return (result += String.fromCharCode(g.unicode))
        },
        ''
      )

      this.fontFamily = vfont.glyphs.font.names.fontFamily.en
      if (this.font) {
        this.setPreview()
      }
    },
    onFont(font) {
      this.font = font
      if (this.fontFamily) {
        this.setPreview()
      }
    },
    onLoading(fontName) {
      this.fontName = fontName
    },
    setPreview() {
      if (this.isPreviewSet) return
      const st = document.createElement('style')
      st.type = 'text/css'
      st.appendChild(
        document.createTextNode(`
          @font-face {font-family:'${this.fontFamily}'; src: url('${this.font}')
      `)
      )
      document.head.appendChild(st)
    }
  }
}
</script>

<style>
.preview {
  font-size: 64px;
}
</style>
