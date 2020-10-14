<template>
<div>
  <a-space>
    <a-button type="primary" @click="visible = true">
      解析JSON
    </a-button>

    <a-button :data-clipboard-text="copyText" class="jumpUrl" @click="copy('jumpUrl')" type="text">
      复制代码
    </a-button>
  </a-space>

  <draggable v-model="myArray.list" @change="datadragEnd" class="warp" v-if="myArray" animation="1000">
    <a-card class="card" size="small" :title="element.label" v-for="(element, index) in myArray.list" :key="index">
      <a-form-model layout="horizontal" v-bind="{
            labelCol: { span: 4 },
            wrapperCol: { span: 14 },
          }">
        <a-form-model-item label="列名" v-if="Object.keys(myArray.list[index]).indexOf('label') >= 0">
          <a-input v-model="myArray.list[index].label"></a-input>

        </a-form-model-item>
        <!-- <a-form-model-item label="新增" v-if="Object.keys(myArray.list[index].customize).indexOf('create') >= 0">
          <a-switch checked-children="开" un-checked-children="关" v-model="myArray.list[index].customize.create" default-checked />
        </a-form-model-item>
        <a-form-model-item label="修改" v-if="Object.keys(myArray.list[index].customize).indexOf('edit') >= 0">
          <a-switch checked-children="开" un-checked-children="关" v-model="myArray.list[index].customize.edit" default-checked />
        </a-form-model-item>
        <a-form-model-item label="列表" v-if="Object.keys(myArray.list[index].customize).indexOf('table') >= 0">
          <a-switch checked-children="开" un-checked-children="关" v-model="myArray.list[index].customize.table" default-checked />
        </a-form-model-item>
        <a-form-model-item label="详情" v-if="Object.keys(myArray.list[index].customize).indexOf('detail') >= 0">
          <a-switch checked-children="开" un-checked-children="关" v-model="myArray.list[index].customize.detail" default-checked />
        </a-form-model-item>
        <a-form-model-item label="搜索" v-if="Object.keys(myArray.list[index].customize).indexOf('search') >= 0">
          <a-switch checked-children="开" un-checked-children="关" v-model="myArray.list[index].customize.search" default-checked />
        </a-form-model-item>
        <a-form-model-item label="插槽" v-if="Object.keys(myArray.list[index].customize).indexOf('sort') >= 0">
          <a-switch checked-children="开" un-checked-children="关" v-model="myArray.list[index].customize.sort" default-checked />
        </a-form-model-item>-->
        <a-form-model-item label="类型" v-if="Object.keys(myArray.list[index]).indexOf('type') >= 0">
          <a-select :default-value="myArray.list[index].type">
            <a-select-option value="input">
              单行文本
            </a-select-option>
            <a-select-option value="date">
              日期选择器
            </a-select-option>
            <a-select-option value="textarea">
              多行文本
            </a-select-option>
            <a-select-option value="select">
              下拉选择器
            </a-select-option>
            <a-select-option value="radio">
              单选框
            </a-select-option>
          </a-select>
        </a-form-model-item>
      </a-form-model>
    </a-card>
  </draggable>
  <a-modal v-model="visible" title="解析JSON" @ok="handleOk">
    <a-textarea v-model="value" placeholder="请输入解析JSON" :auto-size="{ minRows: 15, maxRows: 25 }" />
  </a-modal>
</div>
</template>

<script>
import Clipboard from "clipboard"; // 复制
import draggable from "vuedraggable";

export default {
  data() {
    return {
      value: "",
      copyText: "",
      visible: false,
      myArray: null,
    };
  },
  components: {
    // eslint-disable-next-line vue/no-unused-components
    draggable,
  },
  methods: {
    copy(val) {
      this.copyText = JSON.stringify(this.myArray);
      var clipboard = new Clipboard(`.${val}`);
      clipboard.on("success", () => {
        // 释放内存
        this.$message.success("复制成功");
        clipboard.destroy();
        this.copyText = "";
      });
      clipboard.on("error", () => {
        alert("不支持此浏览器");
        // 释放内存
        clipboard.destroy();
      });
    },
    handleOk() {
      let _value = this.value.replace(/\s*/g, "");
      // 去除空格
      // 构建函数字符串，供eval解析
      let fun_str = "(" + _value + ")";
      let result;
      try {
        result = eval(fun_str);
        this.myArray = result;
        this.visible = false;
        this.value = "";
      } catch (err) {
        // 这里可以给用户一些提示，让输入数组或json格式，而不是其它类型的字符串
        this.$message.warning("请输入正确的JSON");
        console.log("错误了");
        console.log(err);
      }
      console.log(result);
      console.log(typeof result);
    },
    datadragEnd() {
      this.value = "";
      console.log(this.myArray);
    },
  },
};
</script>

<style lang="less" scoped>
.warp {
  display: flex;
}

.card {
  flex: 1;
}

.card:hover {
  cursor: move;
}

.list-complete-enter,
.list-complete-leave-active {
  opacity: 0;
  height: 0px;
  margin-top: 0px;
  padding: 0px;
  border: solid 0px;
}

.list-complete-sortable-chosen,
.list-complete-sortable-ghost {
  opacity: 0;
  height: 0px;
  margin-top: 0px;
  padding: 0px;
  border: solid 0px;
}
</style>
