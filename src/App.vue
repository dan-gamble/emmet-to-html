<template>
  <div>
    <textarea name="" id="" cols="30" rows="10" v-model="html" debounce="300"></textarea>
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
        const parsed = parser(this.html)

        return createString(parsed)
      }
    }
  }

  function createString (parsed) {
    let emmetString = ''
    const els = _.filter(parsed, (e) => typeof e === 'object')

    for (const el of els) {
      if (isAdjacent(el, els)) emmetString += '+'

      emmetString += parseObject(el, emmetString)
    }

    return emmetString
  }

  function parseObject(obj, str='') {
    // Restart the string if it's a new object
    if (str) str = ''
    str += obj.tag

    if (_.has(obj, 'attrs')) {
      for (const attr of Object.keys(obj.attrs)) {
        // If the attribute is empty, skip it
        // if (!obj.attrs[attr]) continue

        if (attr === 'class') {
          // If it's a class it needs to be prepended by a `.`
          str += `.${obj.attrs[attr].split(' ').join('.')}`
        } else if (attr === 'id' && obj.attrs[attr]) {
          // If it's an id it needs to be prepended by a `#`
          str += `#${obj.attrs[attr]}`
        } else if (attr !== 'class' && attr !== 'id') {
          str += `[${attr}="${obj.attrs[attr]}"]`
        }
      }
    }

    if (_.has(obj, 'content')) {
      const textContent = _.filter(obj.content, item => typeof item === 'string' && _.trim(item).length)
      const trimmedText = _.map(textContent, item => _.trim(item))[0]

      if (trimmedText) str += `{${trimmedText}}`

      const els = _.filter(obj.content, (e) => typeof e === 'object')

      for (const el of els) {
        isAdjacent(el, els) ? str += '+' : str += '>'

        str += parseObject(el)
      }
    }

    return str
  }

  function isAdjacent(item, arr) {
    return arr.indexOf(item) !== 0 && arr.indexOf(item) > 0
  }
</script>
