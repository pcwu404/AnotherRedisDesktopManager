<template>
  <div>
    <!-- </textarea> -->
    <el-input ref='textInput' :disabled='disabled' type='textarea' :rows='textrows' :value='contentDisplay'></el-input>
  </div>
</template>

<script type="text/javascript">
export default {
  props: ['content', 'contentVisible', 'textrows', 'disabled'],
  computed: {
    contentDisplay() {
      return this.$util.bufToBinary(this.content);
    },
  },
  watch: {
    content() {
      // refresh
      this.$nextTick(() => {
        this.$refs.textInput.$refs.textarea.value = this.contentDisplay;
      });
    },
  },
  methods: {
    getContent() {
      const content = this.$refs.textInput.$refs.textarea.value;
      return this.$util.binaryStringToBuffer(content);
    },
  },
}
</script>
