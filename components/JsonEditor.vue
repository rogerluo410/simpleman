<template>
  <div class="json-editor">
    <textarea ref="textarea" />
  </div>
</template>

<script>
import CodeMirror from 'codemirror'
import 'codemirror/lib/codemirror.css'
import 'codemirror/mode/javascript/javascript'
import 'codemirror/addon/lint/lint.css'
import 'codemirror/addon/lint/lint'
import 'codemirror/addon/lint/json-lint'

// 主题样式
import 'codemirror/theme/rubyblue.css'
// 括号显示匹配
import 'codemirror/addon/edit/matchbrackets'
import 'codemirror/addon/selection/active-line'
// 括号、引号编辑和删除时成对出现
import 'codemirror/addon/edit/closebrackets'
// 折叠代码要用到一些玩意
import 'codemirror/addon/fold/foldgutter.css'
import 'codemirror/addon/fold/foldgutter'
import 'codemirror/addon/fold/xml-fold'
import 'codemirror/addon/fold/foldcode'
import 'codemirror/addon/fold/brace-fold'
import 'codemirror/addon/fold/indent-fold.js'
import 'codemirror/addon/fold/markdown-fold.js'
import 'codemirror/addon/fold/comment-fold.js' // 代码高亮必须引入

// 代码错误检查
// eslint-disable-next-line import/no-webpack-loader-syntax
require('script-loader!jsonlint')

export default {
  name: 'CodeMirror',
  components: {},
  props: {
    jsonCode: {
      type: String,
      default: '',
    },
    readonly: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      jsonEditor: '',
    }
  },
  watch: {
    jsonCode(newVal) {
      const editorValue = this.jsonEditor.getValue()
      if (newVal !== editorValue) {
        this.jsonEditor.setValue(
          // 这里是json格式化
          JSON.stringify(JSON.parse(this.jsonCode), null, 2)
        )
        setTimeout(() => {
          this.jsonEditor.refresh()
        }, 1)
      }
    },
    immediate: true,
    deep: true,
  },
  mounted() {
    this.jsonEditor = CodeMirror.fromTextArea(this.$refs.textarea, {
      mode: 'application/json',
      theme: 'base16-light', // 主题样式
      lint: true,
      tabSize: 2,
      smartIndent: true, // 是否智能缩进
      styleActiveLine: true, // 当前行高亮
      lineNumbers: true, // 显示行号
      gutters: [
        'CodeMirror-linenumbers',
        'CodeMirror-foldgutter',
        'CodeMirror-lint-markers',
      ],
      lineWrapping: true, // 自动换行
      matchBrackets: true, // 括号匹配显示
      autoCloseBrackets: true, // 输入和退格时成对
      readOnly: this.readonly, // 只读
      foldGutter: true,
    })
    // 这里是json格式化
    this.jsonEditor.setValue(JSON.stringify(JSON.parse(this.jsonCode), null, 2))
    this.jsonEditor.on('change', (cm) => {
      this.$emit('change', cm.getValue())
    })
  },
  methods: {
    // 这玩意就是为了刷新编辑器的
    refresh() {
      /*
       * refresh: Fires when the editor is refreshed or resized.
       * Mostly useful to invalidate cached values that depend on the editor or character size.
       */
      this.jsonEditor && this.jsonEditor.refresh()
    },
  },
}
</script>

<style>
.json-editor {
  height: 100%;
  position: relative;
}
/* 高度自适应 */
.json-editor .CodeMirror {
  height: auto;
}
.json-editor .CodeMirror-scroll {
  height: auto;
  overflow-y: hidden;
  overflow-x: auto;
}

.CodeMirror-foldgutter-folded.CodeMirror-guttermarker-subtle {
  color: black !important;
  font-size: 14px;
}
.CodeMirror-foldgutter-open:after {
  color: black !important;
  font-size: 14px;
}
.json-editor .cm-string {
  color: coral !important;
}
</style>