<template>
<el-form class='key-content-string'>
  <!-- key content textarea -->
  <el-form-item>
    <FormatViewer
      ref='formatViewer'
      :content='content'
      :binary='binary'
      :redisKey='redisKey'
      float=''
      :textrows=12>
    </FormatViewer>
  </el-form-item>

  <!-- save btn -->
  <el-form-item>
    <el-button ref='saveBtn' type="primary" @click="execSave" title='Ctrl+s'>{{ $t('message.save') }}</el-button>
  </el-form-item>

  <ScrollToTop parentNum='4'></ScrollToTop>
</el-form>
</template>

<script>
import FormatViewer from '@/components/FormatViewer';
import ScrollToTop from '@/components/ScrollToTop';

export default {
  data() {
    return {
      content: Buffer.from(''),
      binary: false,
    };
  },
  props: ['client', 'redisKey', 'hotKeyScope'],
  components: { FormatViewer, ScrollToTop },
  methods: {
    initShow() {
      this.client.getBuffer(this.redisKey).then((reply) => {
        this.content = reply;
        // this.$refs.formatViewer.autoFormat();
      });
    },
    execSave() {
      const content = this.$refs.formatViewer.getContent();

      // viewer check failed, do not save
      if (content === false) {
        return;
      }

      this.client.set(
        this.redisKey,
        content
      ).then((reply) => {
        if (reply === 'OK') {
          // for compatibility, use expire instead of setex
          this.setTTL();
          this.initShow()

          this.$message.success({
            message: this.$t('message.modify_success'),
            duration: 1000,
          });
        }

        else {
          this.$message.error({
            message: this.$t('message.modify_failed'),
            duration: 1000,
          });
        }
      }).catch(e => {
        this.$message.error(e.message);
      });
    },
    setTTL () {
      const ttl = parseInt(this.$parent.$parent.$refs.keyHeader.keyTTL);

      if (ttl > 0) {
        this.client.expire(this.redisKey, ttl).catch(e => {
          this.$message.error('Expire Error: ' + e.message);
        }).then(reply => {});
      }
    },
    initShortcut() {
      this.$shortcut.bind('ctrl+s, ⌘+s', this.hotKeyScope, () => {
        // make input blur to fill the new value
        // this.$refs.saveBtn.$el.focus();
        this.execSave();

        return false;
      });
    },
    dumpCommand() {
      const command = `SET ${this.$util.bufToQuotation(this.redisKey)} ` +
                      this.$util.bufToQuotation(this.content);
      this.$util.copyToClipboard(command);
      this.$message.success({message: this.$t('message.copy_success'), duration: 800});
    },
  },
  mounted() {
    this.initShow();
    this.initShortcut();
  },
};
</script>

<style type="text/css">
  .key-content-string .format-viewer-container {
    min-height: calc(100vh - 253px);
  }

  /*text viewer box*/
  .key-content-string .el-textarea textarea {
    font-size: 14px;
    height: calc(100vh - 286px);
  }
  /*json in monaco editor*/
  .key-content-string #monaco-editor-con {
    height: calc(100vh - 331px);
  }
  /*not text viewer box, such as json*/
  .key-content-string .text-formated-container {
    box-sizing: border-box;
    min-height: calc(100vh - 286px);
  }
</style>
