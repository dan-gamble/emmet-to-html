<template>
  <div>
    <textarea name="" id="" cols="30" rows="10" v-model="html"></textarea>
    <pre><code>{{ emmet | json }}</code></pre>
    <pre><code>{{ parsed | json }}</code></pre>

    {{ $data | json }}
  </div>
</template>

<script type="text/ecmascript-6">
import Hello from './components/Hello'
import parser from 'posthtml-parser'
import _ from 'lodash'

export default {
  data () {
    return {
      html: '',
    }
  },

  computed: {
    parsed () {
      return parser(this.html)
    },
    emmet () {
      const html = parser(this.html)
      let emmetString = ''

      for (const el of html) {
        emmetString += el.tag

        for (const key of Object.keys(el.attrs)) {
          if (key === 'class') emmetString += `.${el.attrs[key]}`
          if (key === 'id') emmetString += `#${el.attrs[key]}`

          if (_.has(el, 'content')) {
            const els = _.filter(el.content, (e) => typeof e === 'object')
            console.log(els)
          }
        }
      }

      return emmetString
    }
  }
}
</script>
