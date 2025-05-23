<template>
  <a href="/">Main</a>

  <div class="d-flex flex-column" style="height: 90vh">
    <div class="h-50">
      <div v-if="editor">
        <button :disabled="!editor.can().toggleStrong().run()" @click="editor.chain().toggleStrong().run()">toggleStrong</button>
        <button :disabled="!editor.can().setHorizontalRule().run()" @click="editor.chain().setHorizontalRule().run()">setHorizontalRule</button>
        <button :disabled="!editor.can().setHardBreak().run()" @click="editor.chain().setHardBreak().run()">setHardBreak</button>
        <button :disabled="!editor.can().setCodeBlock().run()" @click="editor.chain().setCodeBlock().run()">setCodeBlock</button>
        <button :disabled="!editor.can().insertTable().run()" @click="editor.chain().insertTable().run()">insertTable</button>
        <button @click="loadDoc">Simulate loadDoc</button>
      </div>
      <div class="d-flex">
        <div ref="editor" class="w-50"></div>
        <div class="w-50">
          <h5>Markdown</h5>
          <pre>{{md}}</pre>
        </div>
      </div>
    </div>
    <div class="h-50">
      <div class="d-flex">
        <div class="w-50">
          <h5>Prosemirror JSON</h5>
          <pre>{{ JSON.stringify(lastValue, null, 2) }}</pre>
        </div>
        <div class="w-50">
          <h5>Automerge JSON</h5>
          Spans
          <pre>{{ JSON.stringify(spans, null, 2) }}</pre>
          Marks
          <pre>{{ JSON.stringify(marks, null, 2) }}</pre>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import {CoreEditor} from "@kerebron/editor";
import {ExtensionBasicEditor} from "@kerebron/extension-basic-editor";
import {ExtensionMenu} from "@kerebron/extension-menu";
import {ExtensionAutomerge} from "@kerebron/extension-automerge";

import { BrowserWebSocketClientAdapter } from '@automerge/automerge-repo-network-websocket';

export default {
  name: "EditorAutomerge",
  props: ['modelValue'],
  data() {
    return {
      peerId: '',
      lastValue: null,
      doc: {},
      spans: [],
      marks: [],
      md: '',
      editor: null
    }
  },
  async mounted() {
    const pmDoc = {
      "type": "doc",
      "attrs": {
      },
      "content": [
        {
          "type": "paragraph",
          "attrs": {
          },
          "content": [
            {
              "type": "text",
              "text": "HELLO!"
            }
          ]
        }
      ]
    };

    const protocol = globalThis.location.protocol === 'http:' ? 'ws:' : 'wss:';
    const networkAdapter = new BrowserWebSocketClientAdapter(
      protocol + '//' + globalThis.location.host + '/automerge',
    );

    const extensionAutomerge = new ExtensionAutomerge({
      networkAdapter
    });

    this.editor = new CoreEditor({
      element: this.$refs.editor,
      extensions: [
        new ExtensionBasicEditor(),
        new ExtensionMenu(),
        extensionAutomerge
      ],
      // content: pmDoc
    });

    this.editor.addEventListener('transaction', (ev: CustomEvent) => {
      this.lastValue = ev.detail.transaction.doc;
      // this.md = this.editor.getDocument('text/x-markdown');
      // this.$emit('input', this.lastValue);
    });

    this.editor.addEventListener('automerge:change', (ev: CustomEvent) => {
      this.spans = ev.detail.getSpans();
      this.marks = ev.detail.getMarks();
    });

    this.editor.addEventListener('automerge:url', (ev: CustomEvent) => {
      globalThis.location.hash = ev.detail.url;
    });

    const docUrl = globalThis.location.hash.slice(1);
    if (docUrl.startsWith('automerge:')) {
      extensionAutomerge.loadFromAutoMerge(docUrl);
    } else {
      this.editor.setDocument('# TEST \n\n1.  aaa\n2.  bbb', 'text/x-markdown');
      this.editor.setDocument(pmDoc);
      this.editor.setDocument('# TEST \n\n1.  aaa\n2.  bbb\n\n<table><tr><td>CELL</td></tr></table>', 'text/x-markdown');
      this.editor.setDocument(null);
    }

  },
  watch: {
    value(newValue) {
      if (newValue !== this.lastValue) {
        // this.state = createState(newValue, this.handle);
        // this.view.updateState(this.state);
      }
    }
  },
  methods: {
    loadDoc() {
      this.editor.setDocument('# TEST \n\n1.  aaa **bold**\n2.  bbb\n\n```js\nconsole.log("TEST")\n```\n', 'text/x-markdown');
    }
  }
};
//@import "@kerebron/extension-tables/tables.css";

</script>

<style>
table {
  border: 1px solid red;
}
table th, table td {
  border: 1px solid red;
}

.ProseMirror {
  position: relative;
}

.ProseMirror {
  word-wrap: break-word;
  white-space: pre-wrap;
  -webkit-font-variant-ligatures: none;
  font-variant-ligatures: none;
}

.ProseMirror pre {
  white-space: pre-wrap;
}

.ProseMirror li {
  position: relative;
}

.ProseMirror-hideselection *::selection { background: transparent; }
.ProseMirror-hideselection *::-moz-selection { background: transparent; }
.ProseMirror-hideselection { caret-color: transparent; }

.ProseMirror-selectednode {
  outline: 2px solid #8cf;
}

/* Make sure li selections wrap around markers */

li.ProseMirror-selectednode {
  outline: none;
}

li.ProseMirror-selectednode:after {
  content: "";
  position: absolute;
  left: -32px;
  right: -2px; top: -2px; bottom: -2px;
  border: 2px solid #8cf;
  pointer-events: none;
}
.ProseMirror-textblock-dropdown {
  min-width: 3em;
}

.ProseMirror-menu {
  margin: 0 -4px;
  line-height: 1;
}

.ProseMirror-tooltip .ProseMirror-menu {
  width: -webkit-fit-content;
  width: fit-content;
  white-space: pre;
}

.ProseMirror-menuitem {
  margin-right: 3px;
  display: inline-block;
}

.ProseMirror-menuseparator {
  border-right: 1px solid #ddd;
  margin-right: 3px;
}

.ProseMirror-menu-dropdown, .ProseMirror-menu-dropdown-menu {
  font-size: 90%;
  white-space: nowrap;
}

.ProseMirror-menu-dropdown {
  vertical-align: 1px;
  cursor: pointer;
  position: relative;
  padding-right: 15px;
}

.ProseMirror-menu-dropdown-wrap {
  padding: 1px 0 1px 4px;
  display: inline-block;
  position: relative;
}

.ProseMirror-menu-dropdown:after {
  content: "";
  border-left: 4px solid transparent;
  border-right: 4px solid transparent;
  border-top: 4px solid currentColor;
  opacity: .6;
  position: absolute;
  right: 4px;
  top: calc(50% - 2px);
}

.ProseMirror-menu-dropdown-menu, .ProseMirror-menu-submenu {
  position: absolute;
  background: white;
  color: #666;
  border: 1px solid #aaa;
  padding: 2px;
}

.ProseMirror-menu-dropdown-menu {
  z-index: 15;
  min-width: 6em;
}

.ProseMirror-menu-dropdown-item {
  cursor: pointer;
  padding: 2px 8px 2px 4px;
}

.ProseMirror-menu-dropdown-item:hover {
  background: #f2f2f2;
}

.ProseMirror-menu-submenu-wrap {
  position: relative;
  margin-right: -4px;
}

.ProseMirror-menu-submenu-label:after {
  content: "";
  border-top: 4px solid transparent;
  border-bottom: 4px solid transparent;
  border-left: 4px solid currentColor;
  opacity: .6;
  position: absolute;
  right: 4px;
  top: calc(50% - 4px);
}

.ProseMirror-menu-submenu {
  display: none;
  min-width: 4em;
  left: 100%;
  top: -3px;
}

.ProseMirror-menu-active {
  background: #eee;
  border-radius: 4px;
}

.ProseMirror-menu-active {
  background: #eee;
  border-radius: 4px;
}

.ProseMirror-menu-disabled {
  opacity: .3;
}

.ProseMirror-menu-submenu-wrap:hover .ProseMirror-menu-submenu, .ProseMirror-menu-submenu-wrap-active .ProseMirror-menu-submenu {
  display: block;
}

.ProseMirror-menubar {
  border-top-left-radius: inherit;
  border-top-right-radius: inherit;
  position: relative;
  min-height: 1em;
  color: #666;
  padding: 1px 6px;
  top: 0; left: 0; right: 0;
  border-bottom: 1px solid silver;
  background: white;
  z-index: 10;
  -moz-box-sizing: border-box;
  box-sizing: border-box;
  overflow: visible;
}

.ProseMirror-icon {
  display: inline-block;
  line-height: .8;
  vertical-align: -2px; /* Compensate for padding */
  padding: 2px 8px;
  cursor: pointer;
}

.ProseMirror-menu-disabled.ProseMirror-icon {
  cursor: default;
}

.ProseMirror-icon svg {
  fill: currentColor;
  height: 1em;
}

.ProseMirror-icon span {
  vertical-align: text-top;
}
.ProseMirror-gapcursor {
  display: none;
  pointer-events: none;
  position: absolute;
}

.ProseMirror-gapcursor:after {
  content: "";
  display: block;
  position: absolute;
  top: -2px;
  width: 20px;
  border-top: 1px solid black;
  animation: ProseMirror-cursor-blink 1.1s steps(2, start) infinite;
}

@keyframes ProseMirror-cursor-blink {
  to {
    visibility: hidden;
  }
}

.ProseMirror-focused .ProseMirror-gapcursor {
  display: block;
}
/* Add space around the hr to make clicking it easier */

.ProseMirror-example-setup-style hr {
  padding: 2px 10px;
  border: none;
  margin: 1em 0;
}

.ProseMirror-example-setup-style hr:after {
  content: "";
  display: block;
  height: 1px;
  background-color: silver;
  line-height: 2px;
}

.ProseMirror ul, .ProseMirror ol {
  padding-left: 30px;
}

.ProseMirror blockquote {
  padding-left: 1em;
  border-left: 3px solid #eee;
  margin-left: 0; margin-right: 0;
}

.ProseMirror-example-setup-style img {
  cursor: default;
}

.ProseMirror-prompt {
  background: white;
  padding: 5px 10px 5px 15px;
  border: 1px solid silver;
  position: fixed;
  border-radius: 3px;
  z-index: 11;
  box-shadow: -.5px 2px 5px rgba(0, 0, 0, .2);
}

.ProseMirror-prompt h5 {
  margin: 0;
  font-weight: normal;
  font-size: 100%;
  color: #444;
}

.ProseMirror-prompt input[type="text"],
.ProseMirror-prompt textarea {
  background: #eee;
  border: none;
  outline: none;
}

.ProseMirror-prompt input[type="text"] {
  padding: 0 4px;
}

.ProseMirror-prompt-close {
  position: absolute;
  left: 2px; top: 1px;
  color: #666;
  border: none; background: transparent; padding: 0;
}

.ProseMirror-prompt-close:after {
  content: "âœ•";
  font-size: 12px;
}

.ProseMirror-invalid {
  background: #ffc;
  border: 1px solid #cc7;
  border-radius: 4px;
  padding: 5px 10px;
  position: absolute;
  min-width: 10em;
}

.ProseMirror-prompt-buttons {
  margin-top: 5px;
  display: none;
}
#editor, .editor {
  background: white;
  color: black;
  background-clip: padding-box;
  border-radius: 4px;
  border: 2px solid rgba(0, 0, 0, 0.2);
  padding: 5px 0;
  margin-bottom: 23px;
}

.ProseMirror p:first-child,
.ProseMirror h1:first-child,
.ProseMirror h2:first-child,
.ProseMirror h3:first-child,
.ProseMirror h4:first-child,
.ProseMirror h5:first-child,
.ProseMirror h6:first-child {
  margin-top: 10px;
}

.ProseMirror {
  padding: 4px 8px 4px 14px;
  line-height: 1.2;
  outline: none;
}

.ProseMirror p { margin-bottom: 1em }

.codeblock-select {
  position: absolute;
  right: 0;
  z-index: 100;
  opacity: 0;
  transition: all 0.3s ease;
  margin: 6px 14px;
}

.codeblock-root {
  position: relative;
}

.codeblock-root:hover .codeblock-select {
  opacity: 1;
}
</style>
