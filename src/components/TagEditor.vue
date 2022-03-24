<template>
<div>
  <el-dialog id="tag-editor" title="文件标签编辑" :visible.sync="tagEditorShow" >
    <el-tag
      :key="tag"
      v-for="tag in tags"
      closable
      :disable-transitions="false"
      @close="handleDelete(tag)"
      size="medium">
      {{tag}}
    </el-tag>
    <el-select
      class="input-new-tag"
      size="mini"
      v-if="inputVisible"
      v-model="inputValue"
      filterable
      allow-create
      default-first-option
      ref="saveTagInput"
      placeholder="回车确认"
      @keyup.enter.native="handleInputConfirm">
      <el-option
        v-for="item in all_tags"
        :key="item.value"
        :label="item.label"
        :value="item.value">
      </el-option>
    </el-select>
<!--    <el-input-->
<!--      class="input-new-tag"-->
<!--      v-if="inputVisible"-->
<!--      v-model="inputValue"-->
<!--      ref="saveTagInput"-->
<!--      size="mini"-->
<!--      @keyup.enter.native="handleInputConfirm"-->
<!--      @blur="handleInputConfirm"-->
<!--    >-->
<!--    </el-input>-->
    <el-button v-else class="button-new-tag" size="small" @click="showInput">+ 添加标签</el-button>
  </el-dialog>
</div>
</template>

<script>
export default {
  name: "TagEditor.vue",
  data(){
    return {
      tagEditorShow: this.tagEditorShow,
      fileID: this.fileID,
      tags: [],
      all_tags:[],
      inputVisible: false,
      inputValue: ''
    }
  },
  props: {
    tagEditorShow: {
      type: Boolean,
      required: true,
    },
    fileID: {
      type: String,
      required: true,
    },
  },
  created() {
    this.getAllTags()
  },
  mounted() {
    this.getFileTags()
  },
  methods: {
    getAllTags() {
      let _this = this;
      _this.$api.tagEditor.getLabels().then((result) => {
        if (result.code === 200) {
          let _tags = []
          for (const d of result.data) {
            let option = {
              value: d.name,
              label: d.name
            }
            _tags.push(option);
          }
          _this.all_tags = _tags
        }
      }).catch((e) => {
        _this.$message.error("getAllTags: "+e);
      });
    },
    getFileTags() {
      let _this = this;
      let data = {
        fileID: _this.fileID
      }
      _this.$api.tagEditor.getLabels(data).then((result) => {
        if (result.code === 200) {
          let _tags = []
          for (const d of result.data) {
            _tags.push(d.name);
          }
          _this.tags = _tags
        }
      }).catch((e) => {
        _this.$message.error("getFileTags: "+e);
      });
    },
    createTag(tagName) {
      let _this = this;
      let data = {
        fileID: _this.fileID,
        name: tagName
      }
      _this.$api.tagEditor.createLabel(data).then((result) => {
        if (result.code === 200) {

        }
      }).catch((e) => {
        _this.$message.error("getFileTags: "+e);
      });
    },


    handleDelete(tag) {
      let _this = this;
      let data = {
        fileID: _this.fileID,
        name: tag
      }
      _this.$api.tagEditor.deleteFileLabel(data).then((result) => {
        if (result.code === 200) {
          this.tags.splice(this.tags.indexOf(tag), 1);
        }
      }).catch((e) => {
        _this.$message.error("getFileTags: "+e);
      });
    },
    showInput() {
      this.inputVisible = true;
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus();
      });
    },

    handleInputConfirm() {
      let inputValue = this.inputValue;
      if (inputValue) {
        let _this = this;
        let data = {
          fileID: _this.fileID,
          name: inputValue
        }
        _this.$api.tagEditor.createLabel(data).then((result) => {
          if (result.code === 200) {
            _this.tags.push(inputValue);
          }
        }).catch((e) => {
          _this.$message.error("getFileTags: "+e);
        });

      }
      this.inputVisible = false;
      this.inputValue = '';
    }
  },
  watch: {
    tagEditorShow: function (val) {
      this.$emit('tagEditorShow', val)
    },
    fileID: function() {
      this.getFileTags();
    }
  }
}
</script>

<style scoped>
.el-tag {
  margin: 5px;
}
.button-new-tag {
  width: 90px;
  margin: 5px;
  height: 28px;
  line-height: 28px;
  padding-top: 0;
  padding-bottom: 0;
}
.input-new-tag {
  width: 90px;
  margin: 5px;
  vertical-align: bottom;
}

</style>
