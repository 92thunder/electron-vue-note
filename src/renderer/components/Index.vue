<template>
  <div class="grid-container">
    <div class="sidebar">
      <div class="memos-title">Memos</div>
      <div class="memos">
        <div v-for="(memo, memoIndex) in memos" :key="memoIndex" @click="changeIndex(memoIndex)"
              :class="['memo', index === memoIndex ? 'active' : '']">
          {{ memo.filePath && memo.filePath.split('/').reverse()[0] || memo.text.split('\n')[0] || "Empty" }}
        </div>
      </div>
    </div>
    <div class="action">
      <div class="action-title">Action</div>
      <div @click="addNew" class="action-item">Add New</div>
      <div @click="importMemo" class="action-item">Import</div>
    </div>
    <div class="editor">
      <textarea ref="editor" v-model="memos[index].text" placeholder="Input free memo." class="textarea"
         @keydown.meta.83="save" @keyup.ctrl.83="save" autofocus></textarea>
    </div>
  </div>
</template>

<script>
  const {dialog} = require('electron').remote
  const fs = require('fs')

  const emptyMemo = {'text': '', filePath: ''}

  export default {
    data: () => {
      return {
        memos: [{'text': 'sample memo.', filePath: ''}],
        index: 0
      }
    },
    methods: {
      addNew () {
        this.memos.push(Object.assign({}, emptyMemo))
        this.changeIndex(this.memos.length - 1)
      },
      changeIndex (index) {
        this.index = index
        this.$refs.editor.focus()
      },
      importMemo () {
        const filePath = dialog.showOpenDialog({
          properties: ['openFile', 'createDirectory']
        })[0]
        fs.readFile(filePath, 'utf8', (error, text) => {
          if (error) {
            console.error('error: ', error)
          } else {
            this.memos.push({
              text: text,
              filePath: filePath
            })
            this.changeIndex(this.memos.length - 1)
          }
        })
      },
      save () {
        const memo = this.memos[this.index]
        if (!memo.filePath) {
          const filePath = dialog.showSaveDialog({title: 'Save as'})
          memo.filePath = filePath
        }
        fs.writeFileSync(memo.filePath, memo.text)
      }
    }
  }
</script>

<style scoped>
  .grid-container {
    display: grid;
    grid-template-columns: 220px 1fr;
    grid-template-rows: 1fr 100px;
    grid-template-areas: "sidebar editor"
                         "action editor";
  }

  .sidebar {
    grid-area: sidebar;
    background-color: #4863ad;
    color: #FFFFFF;
    max-height: calc(100vh - 100px);
    overflow: auto;
  }

  .action {
    grid-area: action;
    background-color: #4863ad;
    color: #FFFFFF;
  }

  .editor {
    grid-area: editor;
    height: 100vh;
  }

  .textarea {
    padding: 8px 8px;
    width: 100%;
    height: 100vh;
    resize: none;
    border: none;
    font-size: 18px;
    line-height: 1.2em;
  }

  .memos {
    overflow-y: auto;
  }

  .memos-title {
    padding: 8px 16px;
    color: #FFFFFF;
  }

  .memo {
    padding: 4px 24px;
    color: #FFFFFF;
    cursor: pointer;
  }
  .memo:hover {
    background-color: #1A3682;
  }
  .memo.active {
    background-color: #9AADDF;
  }

  .action-title {
    padding: 8px 16px;
    color: #FFFFFF;
  }

  .action-item {
    padding: 4px 24px;
    cursor: pointer;
  }
  .action-item:hover {
    background-color: #1A3682;
  }
</style>
