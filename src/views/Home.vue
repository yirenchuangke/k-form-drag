<template>
  <div>
    <a-space>
      <a-button type="primary" @click="visible = true">
        解析JSON
      </a-button>

      <a-button
        v-if="myArray"
        :data-clipboard-text="copyText"
        class="jumpUrl"
        @click="copy('jumpUrl')"
        type="text"
      >
        复制代码
      </a-button>
    </a-space>
    <div class="warp" v-if="myArray">
      <a-card
        class="card"
        size="small"
        :title="element.label"
        v-for="(element, index) in TableList"
        :key="index"
      >
      </a-card>
    </div>

    <draggable
      v-model="myArray"
      chosen-class="chosen"
      force-fallback="true"
      group="people"
      animation="1000"
      @start="onStart"
      @end="onEnd"
    >
      <transition-group>
        <div class="item" v-for="element in myArray" :key="element.id">
          {{ element.name }}
        </div>
      </transition-group>
    </draggable>

    <a-modal v-model="visible" title="解析JSON" @ok="handleOk">
      <a-textarea
        v-model="value"
        placeholder="请输入解析JSON"
        :auto-size="{ minRows: 15, maxRows: 25 }"
      />
    </a-modal>
  </div>
</template>

<script>
import Clipboard from "clipboard"; // 复制
import draggable from "vuedraggable";

export default {
  data() {
    return {
      drag: false,

      value: "",
      copyText: "",
      visible: false,
      myArray: null
    };
  },
  computed: {
    TableList() {
      return this.myArray.list.filter(element => {
        console.log(element.customize.table);
        return element.customize.table;
      });
    }
  },
  components: {
    // eslint-disable-next-line vue/no-unused-components
    draggable
  },
  methods: {
    onStart() {
      this.drag = true;
    },
    onEnd() {
      this.drag = false;
    },
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
    }
  }
};
</script>

<style lang="less" scoped>
.warp {
  display: flex;
  padding: 10px 15px;
}

.card {
  flex: 1;
}

.card:hover {
  cursor: move;
}

.item {
  padding: 6px;
  background-color: #fdfdfd;
  border: solid 1px #eee;
  margin-bottom: 10px;
  cursor: move;
}

.item:hover {
  background-color: #f1f1f1;
  cursor: move;
}

.chosen {
  border: solid 2px #3089dc !important;
}
</style>
