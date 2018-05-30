<template>
  <div class="edit">
    <div ref="textarea" class="text" contenteditable="true" @click="resetRange" @input="resetRange"></div>
    <div class="tools">
      <a href="javascript:;" class="btn" @click.stop="togglePopup">插入链接</a>
    </div>
    <div class="pop" v-show="popup">
      <div class="row">
        <span>标题:</span><input type="text" v-model="title">
      </div>
      <div class="row">
        <span>链接:</span><input type="text" v-model="link">
      </div>
      <div class="row">
        <a href="javascript:;" class="submit btn" @click="addLink">确定</a>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'edit',
    data() {
      return {
        popup: true,
        link: '',
        title: '',
        selection: getSelection(),
        rangeOffset: 0,
        rangeContainer: null,
        targetLink: null
      }
    },
    methods: {
      togglePopup() {
        this.popup = !this.popup;
      },
      addLink() {
        console.group('addLink');
        let a;
        let isEdit = this.isLink() && !this.isAtLinkEnd();
        if(isEdit){
          a = this.rangeContainer.parentNode;
          this.link = a.getAttribute('href');
          this.title = a.innerHTML;
        }else{
          a = document.createElement('a');
        }
        a.href = this.link;
        a.innerHTML = this.title;
        console.log('link: %o', a);
        console.groupEnd();
        !isEdit && this.insert(a);
      },
      isLink(){
        console.group('isLink');
        console.log('node: %o', this.rangeContainer);
        console.log('node.nodeType: %o', this.rangeContainer.nodeType);
        console.log('node.parentNode: %o', this.rangeContainer.parentNode);
        console.groupEnd();
        if(!this.rangeContainer){
          return false;
        }
        let container = this.rangeContainer;
        return container.parentNode && container.parentNode.nodeName === 'A';
      },
      isAtLinkEnd(){
        console.group('isAtLinkEnd');
        if(!this.isLink(this.rangeContainer)){
          return false;
        }
        let offset = this.range.startOffset;
        let text = this.rangeContainer.nodeValue;
        console.log('offset: %o', offset);
        console.log('text: %o', text.length);
        console.groupEnd();
        return text && offset === text.length;
      },
      initRange(context, offset) {
        let range = document.createRange();
        range.setStart(context, offset)
        this.selection.removeAllRanges();
        this.selection.addRange(range);
        this.range = this.selection.getRangeAt(0)
      },
      resetRange() {
        console.group('resetRange');
        this.selection = getSelection();
        this.range = this.selection.getRangeAt(0);
        this.rangeOffset = this.range.startOffset;
        this.rangeContainer = this.range.startContainer;
        let islink = this.isLink();
        let islinkEnd = this.isAtLinkEnd();
        console.log('islink: %o', islink);
        console.log('range: %o', this.range);
        console.log('islinkEnd: %o', islinkEnd);
        console.groupEnd();
      },
      insert(el) {
        console.group('insert');
        // this.resetRange();
        let referenceNode = this.rangeContainer;
        console.log('range: %o', this.range);
        console.log('referenceNode: %o', referenceNode);
        console.log('referenceNode.nodeType: %o', referenceNode.nodeType);
        if (!referenceNode) {
          return;
        }
        let parent = referenceNode.parentNode;
        if (referenceNode.nodeType === 3) {
          if(this.isAtLinkEnd()){
            let link = referenceNode.parentNode;
            this.insertAfter(el, link);
          }else{
            let textLeft = document.createTextNode(referenceNode.nodeValue.slice(0, this.rangeOffset));
            let textRight = document.createTextNode(referenceNode.nodeValue.slice(this.rangeOffset));
            parent.insertBefore(textLeft, referenceNode);
            parent.insertBefore(el, referenceNode);
            parent.insertBefore(textRight, referenceNode);
            this.rangeContainer.remove();
          }
        } else {
          console.log('referenceNode.childNodes: %o', referenceNode.childNodes);
          console.log('rangeOffset: %o', this.rangeOffset);
          if (!referenceNode.childNodes.length) {
            referenceNode.appendChild(el);
          } else {
            let refer = referenceNode.childNodes[this.rangeOffset];
            let index = this.index(refer)
            console.log('refer: %o', refer);
            console.log('index: %o', index);
            this.insertAfter(el, refer);
          }
        }
        console.log('index: %o', this.index(el));
        this.initRange(this.$refs.textarea, this.index(el) + 1);
        console.groupEnd();
      },
      insertAfter(el, referenceNode) {
        console.group('insertAfter');
        console.log('el: %o', el);
        console.log('referenceNode: %o', referenceNode);
        var parent = referenceNode.parentNode;
        if (parent.lastChild == referenceNode) {
          parent.appendChild(el, this.index(el) + 1);
        }
        else {
          parent.insertBefore(el, referenceNode.nextSibling);
        }
        console.groupEnd();
      },
      index(el) {
        let i = 0;
        while (el.nextSibling) {
          el = el.nextSibling;
          i++;
        }
        return i;
      }
    }
  }
</script>

<style>
  
  .text {
    width: 100%;
    height: 300px;
    padding: 10px;
    box-sizing: border-box;
    border: 1px solid #999;
    outline: none;
    text-align: left;
  }
  
  .btn {
    width: 100px;
    text-decoration: none;
    display: block;
    margin: 30px auto;
    padding: 5px 0;
    border-radius: 4px;
    text-align: center;
    color: #fff;
    background-color: olivedrab;
  }
  
  .row {
    margin-bottom: 30px;
  }
  
  .pop {
    width: 600px;
    margin: 50px auto 0;
    padding: 50px 50px 20px 50px;
    border-radius: 6px;
    background-color: #eee;
  }
  
  .pop input {
    height: 30px;
    width: 300px;
  }

</style>

