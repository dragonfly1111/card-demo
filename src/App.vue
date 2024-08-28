<script setup>
import { onMounted, ref, watch } from 'vue'
import { App, dataType, Image, Line, Text, PointerEvent, UI } from 'leafer-ui'
import { Pane } from 'tweakpane';
import '@leafer-in/editor' // 导入图形编辑器插件
import logoImg from '/src/assets/logo.png'
import templateJson from '/src/template.json'
import templateJson1 from '/src/template1.json'
import config from '/src/compConfig.json'
import { showToast } from 'vant';

// import json from './aaa.json'
let app = null
let app1 = null
let pane = null
const exportSrc = ref('')
const showEditTemplate = ref(false)
const showLoadTmpFlag = ref(false)
const showLoadTmpFlag1 = ref(false)
const showNew = ref(false)
const curTmp = ref('')
const newTmpName = ref('')
const tmpOptions = ref('')
const creatPane = (PARAMS, obj) => {
  if(pane) {
    console.log(pane)
    pane.dispose()
  }
  pane = new Pane();
  for (const paneKey in PARAMS) {
    pane.addBinding(PARAMS, paneKey);
  }

  const btn = pane.addButton({
    title: '保存',
  });
  btn.on('click', function() {
    const state = pane.exportState();
    state.children.forEach(item => {
      console.log(item)
      if(item.title !== '保存') {
        obj.setAttr(item.binding.key, item.binding.value)
      }
    })
    pane.hidden = true
  });
}
const importJson = () => {
  app.tree.set({children: templateJson1})
  app.tree.children.forEach((item) => {
    console.log(item)
    const keyName = item.constructor.name
    const tmpArr = config[keyName]
    const PARAMS = {}
    tmpArr.forEach(key => {
      PARAMS[key] = item[key]
    })
    if(item.editable) {
      item.on(PointerEvent.DOUBLE_CLICK, () => {
        // const PARAMS ={
        //   fill: item.fill,
        //   text: item.text,
        //   fontSize: item.fontSize,
        // };
        creatPane(PARAMS, item)
      })
    }
  })
}
const importJson1 = () => {
  showEditTemplate.value = true
  app.tree.set({children: templateJson})
  app.tree.children.forEach((item) => {
    console.log(item)
    const keyName = item.constructor.name
    const tmpArr = config[keyName]
    const PARAMS = {}
    tmpArr.forEach(key => {
      PARAMS[key] = item[key]
    })
    if(item.editable) {
      item.on(PointerEvent.DOUBLE_CLICK, () => {
        // const PARAMS ={
        //   fill: item.fill,
        //   text: item.text,
        //   fontSize: item.fontSize,
        // };
        creatPane(PARAMS, item)
      })
    }
  })
}

const showLoadTmp = () => {
  showLoadTmpFlag.value = !showLoadTmpFlag.value
}
const showLoadTmp1 = () => {
  showLoadTmpFlag1.value = !showLoadTmpFlag1.value
}

const newTemplate = () => {
  showNew.value = true
}

const saveNewTemplate = () => {
  if(!newTmpName.value) return
  showNew.value = false
  const tmp = JSON.parse(localStorage.getItem("template"))
  tmp[newTmpName.value] = []
  newTmpName.value = ''
  localStorage.setItem("template", JSON.stringify(tmp))
  loadOptions()
}

const loadJSON = (key) => {
  curTmp.value = key
  const localTmp = JSON.parse(localStorage.getItem("template"))[key]
  app.tree.set({children: localTmp})
  app.tree.children.forEach((item) => {
    const keyName = item.constructor.name
    const tmpArr = config[keyName]
    const PARAMS = {}
    tmpArr.forEach(key => {
      PARAMS[key] = item[key]
    })
    item.on(PointerEvent.DOUBLE_CLICK, () => {
      // const PARAMS ={
      //   fill: item.fill,
      //   text: item.text,
      //   fontSize: item.fontSize,
      // };
      creatPane(PARAMS, item)
    })
  })
}

const loadJSON1 = (key) => {
  curTmp.value = key
  const localTmp = JSON.parse(localStorage.getItem("template"))[key]
  localTmp.forEach(item => {
    if(item.userEditable) {
      item.editable = true
    } else {
      item.editable = false
    }
  })
  app1.tree.set({children: localTmp})
  app1.tree.children.forEach((item) => {
    const keyName = item.constructor.name
    const tmpArr = config[keyName]
    const PARAMS = {}
    tmpArr.forEach(key => {
      PARAMS[key] = item[key]
    })
    if(item.userEditable) {
      item.on(PointerEvent.DOUBLE_CLICK, () => {
        // const PARAMS ={
        //   fill: item.fill,
        //   text: item.text,
        //   fontSize: item.fontSize,
        // };
        creatPane(PARAMS, item)
      })
    }
  })
}

const saveJson = () => {
  if(!curTmp.value) return
  const json = app.tree.toJSON()
  console.log(json)
  const tmp = localStorage.getItem('template')
  const tmpJSON = JSON.parse(tmp)
  tmpJSON[curTmp.value] = json.children
  localStorage.setItem("template", JSON.stringify(tmpJSON))
  alert('模版【' + curTmp.value + '】保存成功');
}
const exportCanvas = () => {

  app1.export('png').then(res => {
    console.log(res)
    exportSrc.value = res.data;
  })
  app1.export('export.png')
  // const app1 = new App({
  //   editor: {},
  //   view: 'leafer-view1',
  //   width: 600, // 不能设置为 0， 否则会变成自动布局
  //   height: 300,
  //   fill: '#ffffff',
  //   type: 'editor',
  //   wheel: { disabled: true },
  //   move: { disabled: true }
  // })
  // console.log(json.children)
  // app1.tree.set({children: json.children})
}

const addImg = () => {
  const image = new Image({
    editable: true,
    x: 30,
    url: logoImg,
    userEditable: true
  })
  app.tree.add(image)
  image.on(PointerEvent.DOUBLE_CLICK, () => {
    const PARAMS ={
      url: image.url,
      userEditable: image.userEditable,
    };
    creatPane(PARAMS, image)
  })
}
const addTxt = () => {
  const text = Text.one({
    editable: true,
    fill: {r: 0, g: 0, b: 0},
    text: '双击设置文字属性',
    fontSize: 32,
    userEditable: true,
  }, 0, 0)
  app.tree.add(text)
  text.on(PointerEvent.DOUBLE_CLICK, () => {
    const PARAMS ={
      fill: text.fill,
      text: text.text,
      fontSize: text.fontSize,
      userEditable: text.userEditable,
    };
    creatPane(PARAMS, text)
  })
}
const addLine = (type) => {
  const line = new Line({
    editable: true,
    width: type === 2 ? 152 : 500,
    strokeWidth: 2,
    x: 38,
    y: type === 2 ? 50 : 238,
    stroke: '#000000',
    rotation: type === 2 ? 90 : 0,
    userEditable: true
  })
  app.tree.add(line)
  line.on(PointerEvent.DOUBLE_CLICK, () => {
    const PARAMS ={
      x: line.x,
      y: line.y,
      strokeWidth: line.strokeWidth,
      userEditable: line.userEditable,
    };
    creatPane(PARAMS, line)
  })

}
const loadOptions = () => {
  tmpOptions.value = Object.keys(JSON.parse(localStorage.getItem("template")))
}
const localStorageJson = localStorage.getItem('template')
if(!localStorageJson) {
  localStorage.setItem('template', JSON.stringify(templateJson))
}

loadOptions()
onMounted(() => {
  // Debug.enable = true
  app = new App({
    editor: {},
    view: 'leafer-view',
    width: 600, // 不能设置为 0， 否则会变成自动布局
    height: 300,
    fill: '#ffffff',
    type: 'editor',
    wheel: { disabled: true },
    move: { disabled: true }
  })
  app1 = new App({
    editor: {},
    view: 'leafer-view1',
    width: 600, // 不能设置为 0， 否则会变成自动布局
    height: 300,
    fill: '#ffffff',
    type: 'editor',
    wheel: { disabled: true },
    move: { disabled: true }
  })
  Text.addAttr("userEditable", true, dataType)
  Image.addAttr("userEditable", true, dataType)
  Line.addAttr("userEditable", true, dataType)
  // Text.setEditInner('PathEditor')


  // const text = Text.one({
  //   editable: true,
  //   fill: {r: 0, g: 0, b: 0},
  //   text: '标准、规程、规范标识',
  //   fontSize: 32,
  //   test: 3333,
  // }, 130, 90)
  // const image = new Image({
  //   editable: true,
  //   x: 30,
  //   url: logoImg,
  //   draggable: true
  // })
  // const text1 = Text.one({
  //   editable: true,
  //   fill: {r: 0, g: 0, b: 0},
  //   text: '受控编号',
  //   fontSize: 32
  // }, 40, 140)
  // const text11 = Text.one({
  //   editable: true,
  //   fill: {r: 0, g: 0, b: 0},
  //   text: 'ABCD-1234-UIOP',
  //   fontSize: 32
  // }, 250, 140)
  // const text2 = Text.one({
  //   editable: true,
  //   fill: {r: 0, g: 0, b: 0},
  //   text: '受控状态',
  //   fontSize: 32
  // }, 40, 190)
  // const text22 = Text.one({
  //   editable: true,
  //   fill: {r: 0, g: 0, b: 0},
  //   text: '受控',
  //   fontSize: 32
  // }, 250, 190)
  // const text3 = Text.one({
  //   editable: true,
  //   fill: {r: 0, g: 0, b: 0},
  //   text: '存放地点',
  //   fontSize: 32
  // }, 40, 240)
  // const text33 = Text.one({
  //   editable: true,
  //   fill: {r: 0, g: 0, b: 0},
  //   text: '质量管理部',
  //   fontSize: 32
  // }, 250, 240)
  // const line1 = new Line({
  //   editable: true,
  //   width: 500,
  //   strokeWidth: 2,
  //   x: 38,
  //   y: 138,
  //   stroke: '#000000'
  // })
  // const line2 = new Line({
  //   editable: true,
  //   width: 500,
  //   strokeWidth: 2,
  //   x: 38,
  //   y: 188,
  //   stroke: '#000000'
  // })
  // const line3 = new Line({
  //   editable: true,
  //   width: 500,
  //   strokeWidth: 2,
  //   x: 38,
  //   y: 238,
  //   stroke: '#000000'
  // })
  // const line4 = new Line({
  //   editable: true,
  //   width: 500,
  //   strokeWidth: 2,
  //   x: 38,
  //   y: 288,
  //   stroke: '#000000'
  // })
  // const line5 = new Line({
  //   editable: true,
  //   width: 152,
  //   rotation: 90,
  //   strokeWidth: 2,
  //   x: 38,
  //   y: 137,
  //   stroke: '#000000'
  // })
  // const line6 = new Line({
  //   editable: true,
  //   width: 152,
  //   rotation: 90,
  //   strokeWidth: 2,
  //   x: 200,
  //   y: 137,
  //   stroke: '#000000'
  // })
  // const line7 = new Line({
  //   editable: true,
  //   width: 152,
  //   rotation: 90,
  //   strokeWidth: 2,
  //   x: 538,
  //   y: 137,
  //   stroke: '#000000'
  // })
  //
  // app.tree.add(text)
  // app.tree.add(text1)
  // app.tree.add(text11)
  // app.tree.add(text2)
  // app.tree.add(text22)
  // app.tree.add(text3)
  // app.tree.add(text33)
  // app.tree.add(image)
  // app.tree.add(line1)
  // app.tree.add(line2)
  // app.tree.add(line3)
  // app.tree.add(line4)
  // app.tree.add(line5)
  // app.tree.add(line6)
  // app.tree.add(line7)
  // text.on(PointerEvent.DOUBLE_CLICK, () => {
  //   const PARAMS ={
  //     fill: text.fill,
  //     text: text.text,
  //     fontSize: text.fontSize,
  //   };
  //   creatPane(PARAMS, text)
  // })
  // text1.on(PointerEvent.DOUBLE_CLICK, () => {
  //   const PARAMS ={
  //     fill: text1.fill,
  //     text: text1.text,
  //     fontSize: text1.fontSize,
  //   };
  //   creatPane(PARAMS, text1)
  // })
  // image.on(PointerEvent.DOUBLE_CLICK, () => {
  //   const PARAMS ={
  //     url: image.url,
  //   };
  //   creatPane(PARAMS, image)
  // })

  // setTimeout(() => {
  //   console.log(text.setAttr('text', 'cao'))
  // }, 2000)

})
</script>

<template>
  <van-divider>模版编辑 当前模版【{{ curTmp }}】</van-divider>
  <div class="but-wrap">
    <van-button type="success" size="small" @click="showLoadTmp">
      加载模版
    </van-button>
    <van-button type="success" size="small" @click="saveJson">
      保存模版
    </van-button>
  </div>
  <div class="but-wrap" v-if="showLoadTmpFlag">
    <van-button v-for="(item, index) in tmpOptions" type="danger" size="small" @click="loadJSON(item)">
      模版{{index + 1}}: {{ item }}
    </van-button>
    <van-button type="success" size="small" @click="newTemplate">
      新建模版
    </van-button>
  </div>
  <div v-if="showNew" class="but-wrap">
    <div style="width: 60%">
      <van-field v-model="newTmpName" label="模版名称" placeholder="请输入模版名称" />
    </div>
    <van-button type="success" size="small" @click="saveNewTemplate">
      确认
    </van-button>
  </div>
  <div class="but-wrap" v-if="showLoadTmpFlag">
    <van-button type="primary" size="small" @click="addImg">
      图片
    </van-button>
    <van-button type="primary" size="small" @click="addTxt">
      文本
    </van-button>
    <van-button type="primary" size="small" @click="addLine(1)">
      横线
    </van-button>
    <van-button type="primary" size="small" @click="addLine(2)">
      竖线
    </van-button>
  </div>
  <div class="view-wrap">
    <div id="leafer-view"></div>
  </div>
  <van-divider>生成卡片</van-divider>
  <div class="but-wrap">
    <van-button type="success" size="small" @click="showLoadTmp1">
      根据模版创建卡片
    </van-button>
    <van-button type="success" size="small" @click="exportCanvas">
      导出
    </van-button>
  </div>
  <div class="but-wrap" v-if="showLoadTmpFlag1">
    <van-button v-for="(item, index) in tmpOptions" type="danger" size="small" @click="loadJSON1(item)">
      模版{{index + 1}}: {{ item }}
    </van-button>
  </div>
  <div class="view-wrap">
    <div id="leafer-view1"></div>
    <img :src="exportSrc" />
  </div>

</template>

<style scoped>
.but-wrap {
  width: 600px;
  margin-bottom: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 24px;
}
button {
  margin-left: 12px;
}
.view-wrap {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
#leafer-view, #leafer-view1 {
  width: 600px;
  height: 300px;
}
</style>
