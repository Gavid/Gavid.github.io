@[TOC]
# 需求描述
> 今天，在做Element+Vue项目时遇到一个需求：甲方要求在Dialog打开状态下，点击该Dialog以外的区域会导致该Dialog关闭；正确的状态应该是只有在点击关闭按钮，或者是Dialog内的其他操作性按钮才能使得Dialog的状态变为关闭。
# 问题分析
> 之所以会产生这种问题，是因为elementUi在对Dialog组件做初始化的时候，默认让该Dialog在点击组件以外区域会导致该组件关闭，所以elementUI 一定会将该属性暴露出来让开发人员进行配置。
> 通过查询ElementUI的官方文档，发现在Dialog下有个‘==close-on-click-modal==’属性，该属性默认值为‘True’，作用为：是否可以通过点击 modal 关闭 Dialog。
> 所以，通过设置Dialog下的**close-on-click-modal**属性为‘false’，即可解决该问题。
# 问题解决方式
* 解决方式一 ： 将Dialog下的==close-on-click-modal==属性改为‘false’。
	* 需要注意的是： 在使用==close-on-click-modal==属性时，***必须在该属性前加“:”。***
* 解决方式二： 可以通过==before-close==属性，在Dialog关闭时，让用户进行确认是否需要关闭。
	*  before-close 仅当用户通过点击关闭图标或遮罩关闭 Dialog 时起效。如果你在 footer 具名 slot 里添加了用于关闭 Dialog 的按钮，那么可以在按钮的点击回调函数里加入 before-close 的相关逻辑。
# 代码实践
* 解决方式一：
```html
<el-button type="text" @click="dialogVisible = true">点击打开 Dialog</el-button>

<el-dialog
  title="提示"
  :visible.sync="dialogVisible"
  width="30%"
  :close-on-click-modal='false'>
  <span>这是一段信息</span>
  <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
  </span>
</el-dialog>

<script>
  export default {
    data() {
      return {
        dialogVisible: false
      };
    }
  };
</script>
```
* 解决方式二：
```html
<el-button type="text" @click="dialogVisible = true">点击打开 Dialog</el-button>

<el-dialog
  title="提示"
  :visible.sync="dialogVisible"
  width="30%"
  :before-close="handleClose">
  <span>这是一段信息</span>
  <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
  </span>
</el-dialog>

<script>
  export default {
    data() {
      return {
        dialogVisible: false
      };
    },
    methods: {
      handleClose(done) {
        this.$confirm('确认关闭？')
          .then(_ => {
            done();
          })
          .catch(_ => {});
      }
    }
  };
</script>
```
