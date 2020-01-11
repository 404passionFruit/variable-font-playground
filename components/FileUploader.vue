<template>
  <div>
    <form action @submit.prevent>
      <input type="file" @change="onFontUpload" />
    </form>
  </div>
</template>

<script>
import * as opentype from 'opentype.js'
import VariableFont from 'variablefont.js'

export default {
  methods: {
    onFontUpload(e) {
      const file = e.target.files[0]
      if (file) {
        if (typeof window.FileReader !== 'function') {
          console.log("Can't load file")
          return
        }
        this.$emit('loading', e.target.files[0].name)
        this.readBuffer(file)
        this.readFont(file)
      }
    },
    readBuffer(file) {
      const reader = new FileReader()
      reader.onload = () => this.onBuffer(reader.result)
      reader.readAsArrayBuffer(file)
    },
    readFont(file) {
      const reader = new FileReader()
      reader.onload = () => this.onDataUrl(reader.result)
      reader.readAsDataURL(file)
    },
    onBuffer(buffer) {
      const font = opentype.parse(buffer)
      const vf = new VariableFont(font)
      this.$emit('axes', vf)
    },
    onDataUrl(font) {
      this.$emit('font', font)
    }
  }
}
</script>
