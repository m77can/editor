<template>
  <el-aside width="200px" class="ew-left-btns">
    <el-card v-for="(com, index) in edComponets" :key="index" class="ew-left-btns_card">
      <div slot="header">
        {{com.kind}}
      </div>
      <el-button v-for="(list, i) in com.list" :key="i"
      class="ew-component--btn" @click="dragItemClick(list.type)"
      :disabled="(list.type === 11 && !$store.state.editor.isPhone) ||
      (list.type === 12 && !$store.state.editor.isSubmit)">
        <i :class="['iconfont', 'ew-component--icon',list.icon]"></i>
        <span>{{list.text}}</span>
      </el-button>
      </el-card>
  </el-aside>
</template>

<script>
import { textActiveOff } from '@/util/tools';
import generate from 'nanoid/generate';

export default {
  name: 'layoutLeft',
  props: {
  },
  data() {
    return {
      edComponets: [
        {
          kind: '基础组件',
          list: [
            { text: '文本', icon: 'iconfont ed-icon-text1', type: 1 },
            { text: '图片', icon: 'iconfont ed-icon-image', type: 2 },
            { text: '热区', icon: 'ed-icon-requm', type: 3 },
            { text: '多图拼接', icon: 'ed-icon-images', type: 4 },
          ],
        },
        {
          kind: '媒体组件',
          list: [
            { text: '视频', icon: 'ed-icon-shipin1', type: 5 },
            { text: '音频', icon: 'ed-icon-yinpin1', type: 6 },
          ],
        },
        {
          kind: '表单组件',
          list: [
            { text: '单行文本', icon: 'ed-icon-wenben', type: 7 },
            { text: '多行文本', icon: 'ed-icon-duohangwenben', type: 8 },
            { text: '单项选择', icon: 'ed-icon-danxiangxuanze', type: 9 },
            { text: '多项选择', icon: 'ed-icon-duoxuan', type: 10 },
            { text: '手机短信', icon: 'ed-icon-shoujiduanxin', type: 11 },
            { text: '提交按钮', icon: 'ed-icon-tijiao1', type: 12 },
            { text: '图片上传', icon: 'ed-icon-tupianshangchuan', type: 13 },
          ],
        },
      ],
    };
  },
  methods: {
    dragItemClick(type) { // 添加组件
      const zIndex = this.$store.state.editor.layoutKey;
      let layerName;
      let num = 0; // 组件类型索引
      let icon = 'ed-icon-text1';
      const updateEditor = {
        isTextSet: false,
        isImgSet: false,
        isLinkSet: false,
        isImgListSet: false,
        isVideoSet: false,
        isAudioSet: false,
        isFTextSet: false,
        isFTextareaSet: false,
        isFRadioSet: false,
        isFCheckboxSet: false,
        isFSmsSet: false,
        isFSubmitSet: false,
      };
      for (const item in this.$store.state.editor.typeCat) {
        const lists = this.$store.state.editor[this.$store.state.editor.typeCat[item][0]];
        if (lists.length) {
          updateEditor[item[0]] = textActiveOff(lists, { index: 0, isAll: true });
        }
      }
      let newEditor = {};
      const top = document.documentElement.scrollTop || document.body.scrollTop;
      const isScroll = top > 56 && this.$store.state.page.phoneHeight
           > this.$store.state.page.screenHeight;
      const top1 = isScroll ?
        (((((this.$store.state.page.phoneHeight - top) + 56) - 90) / 2) +
        top) - 42 : (this.$store.state.page.screenHeight - 90) / 2;
      const top2 = isScroll ? top - 56 - 24 : 0;
      const id = this.getId();
      const info = {
        id,
        isShow: true,
        zIndex: 1000,
        isActive: true,
        dragIndex: zIndex,
      };
      const mediaSize = {
        w: 375,
        h: 300,
      };
      const mediaLocation = {
        x: 0,
        y: top2,
      };
      const formLocation = {
        x: (375 - 325) / 2,
        y: top1,
      };
      switch (type) {
        case 1:
        {
          let drag = this.$store.state.editor.dragTexts;
          num = this.$store.state.editor.dragTexts.length;
          layerName = `文本${!num ? '' : num + 1}`;
          drag = textActiveOff(drag, { index: 0, isAll: true });
          drag.push(Object.assign({}, info, {
            type: 1,
            content: '',
            fontSize: '12px',
            lineHeight: 1.5,
            textAlign: 'left',
            textColor: 'rgba(0, 0, 0, 1)',
            location: {
              x: 0,
              y: top1,
            },
            size: {
              w: 375,
              h: 90,
            },
            position: 'relative',
            icon,
          }));
          newEditor = {
            textSet: true,
            isTextSet: true,
            dragTexts: drag,
            textActive: num,
            layoutKey: zIndex + 1,
          };
          break;
        }
        case 2:
        {
          const drag2 = this.$store.state.editor.dragImages;
          num = drag2.length;
          layerName = `图片${!num ? '' : num + 1}`;
          icon = 'ed-icon-tupian1';
          drag2.push(Object.assign({}, info, {
            type: 2,
            img: {},
            imgList: [],
            location: mediaLocation,
            size: mediaSize,
            isUpload: false,
            position: 'relative',
            icon,
          }));
          newEditor = {
            imgSet: true,
            isImgSet: true,
            dragImages: drag2,
            imgActive: num,
            layoutKey: zIndex + 1,
          };
          break;
        }
        case 3:
        {
          let drag3 = this.$store.state.editor.dragLinks;
          num = drag3.length;
          layerName = `热区${!num ? '' : num + 1}`;
          icon = 'ed-icon-requm';
          drag3 = textActiveOff(drag3, { index: 0, isAll: true });
          drag3.push(Object.assign({}, info, {
            type: 3,
            appLink: '',
            outLink: '',
            location: {
              x: (375 - 100) / 2,
              y: top1,
            },
            size: {
              w: 100,
              h: 30,
            },
            sourceType: '1', // 1.跳转 2.唤起
            awakeLink: '',
            iosLink: '',
            andLink: '',
            yybLink: '',
            position: 'relative',
            icon,
          }));
          newEditor = {
            linkSet: true,
            isLinkSet: true,
            dragLinks: drag3,
            linkActive: num,
            layoutKey: zIndex + 1,
          };
          break;
        }
        case 4:
        {
          const drag4 = this.$store.state.editor.dragImgLists;
          num = drag4.length;
          layerName = `多图拼接${!num ? '' : num + 1}`;
          icon = 'ed-icon-duotu';
          drag4.push(Object.assign({}, info, {
            type: 4,
            isUplaod: false,
            location: {
              x: 0,
              y: 0,
            },
            size: mediaSize,
            imgList: [],
            icon,
          }));
          newEditor = {
            imgListSet: true,
            isImgListSet: true,
            dragImgLists: drag4,
            imgListActive: num,
            layoutKey: zIndex + 1,
          };
          break;
        }
        case 5:
        {
          let drag5 = this.$store.state.editor.dragVideos;
          num = drag5.length;
          layerName = `视频${!num ? '' : num + 1}`;
          icon = 'ed-icon-shipin';
          drag5 = textActiveOff(drag5, { index: 0, isAll: true });
          drag5.push(Object.assign({}, info, {
            type: 5,
            sourceType: '1', // 1.本地视频 2.在线视频
            source: '',
            videoTitle: '',
            loop: true,
            video: {
              position: 'relative',
              accept: '.mp4',
              size: mediaSize,
              location: mediaLocation,
            },
            lineVideo: {
              position: 'relative',
              size: mediaSize,
              location: mediaLocation,
            },
            isUpload: false,
            position: 'relative',
            icon,
          }));
          newEditor = {
            videoSet: true,
            isVideoSet: true,
            dragVideos: drag5,
            videoActive: num,
            layoutKey: zIndex + 1,
          };
          break;
        }
        case 6:
        {
          let drag6 = this.$store.state.editor.dragAudios;
          num = drag6.length;
          layerName = `音频${!num ? '' : num + 1}`;
          icon = 'ed-icon-tubiao-';
          drag6 = textActiveOff(drag6, { index: 0, isAll: true });
          drag6.push(Object.assign({}, info, {
            type: 6,
            sourceType: '1', // 1.本地音频 2.在线音频
            source: '',
            audioTitle: '',
            loop: true,
            isBorder: '2',
            play: {
              title: '',
              isUplaod: false,
              duration: '00:00',
              url: '',
              accept: '.mp3',
              position: 'relative',
              location: mediaLocation,
              size: {
                w: 375,
                h: 82,
              },
            },
            linePlay: {
              title: '',
              isUplaod: false,
              duration: '00:00',
              url: '',
              position: 'relative',
              location: mediaLocation,
              size: {
                w: 375,
                h: 82,
              },
            },
            position: 'relative',
            icon,
          }));
          newEditor = {
            audioSet: true,
            isAudioSet: true,
            dragAudios: drag6,
            audioActive: num,
            layoutKey: zIndex + 1,
          };
          break;
        }
        case 7:
        {
          let drag7 = this.$store.state.editor.dragFormTexts;
          num = drag7.length;
          layerName = `单行文本${!num ? '' : num + 1}`;
          icon = 'ed-icon-text3';
          drag7 = textActiveOff(drag7, { index: 0, isAll: true });
          drag7.push(Object.assign({}, info, {
            type: 7,
            location: formLocation,
            size: {
              w: 325,
              h: 40,
            },
            position: 'relative',
            icon,
            isRequired: false,
            label: '单行文本',
            place: '单行文本',
          }));
          newEditor = {
            fTextSet: true,
            isFTextSet: true,
            dragFormTexts: drag7,
            fTextActive: num,
            layoutKey: zIndex + 1,
          };
          break;
        }
        case 8:
        {
          let drag8 = this.$store.state.editor.dragFormTextareas;
          num = drag8.length;
          layerName = `多行文本${!num ? '' : num + 1}`;
          icon = 'ed-icon-duohangwenben';
          drag8 = textActiveOff(drag8, { index: 0, isAll: true });
          drag8.push(Object.assign({}, info, {
            type: 8,
            location: formLocation,
            size: {
              w: 325,
              h: 140,
            },
            position: 'relative',
            icon,
            isRequired: false,
            label: '多行文本',
          }));
          newEditor = {
            fTextareaSet: true,
            isFTextareaSet: true,
            dragFormTextareas: drag8,
            fTextareaActive: num,
            layoutKey: zIndex + 1,
          };
          break;
        }
        case 9:
        {
          let drag9 = this.$store.state.editor.dragFormRadios;
          num = drag9.length;
          layerName = `单项选择${!num ? '' : num + 1}`;
          icon = 'ed-icon-danxiangxuanze';
          drag9 = textActiveOff(drag9, { index: 0, isAll: true });
          drag9.push(Object.assign({}, info, {
            type: 9,
            location: formLocation,
            size: {
              w: 325,
              h: 80,
            },
            position: 'relative',
            icon,
            isRequired: false,
            label: '单项选择',
            bgColor: '#5AC7F9',
            textColor: '#fff',
            radioType: 1, // '1'：单选，'2'：多选
            list: [{ text: '选项1', label: 1 }],
            optionIndex: 1,
          }));
          newEditor = {
            fRadioSet: true,
            isFRadioSet: true,
            dragFormRadios: drag9,
            fRadioActive: num,
            layoutKey: zIndex + 1,
          };
          break;
        }
        case 10:
        {
          let drag10 = this.$store.state.editor.dragFormCheckboxs;
          num = drag10.length;
          layerName = `多项选择${!num ? '' : num + 1}`;
          icon = 'ed-icon-duoxuan';
          drag10 = textActiveOff(drag10, { index: 0, isAll: true });
          drag10.push(Object.assign({}, info, {
            type: 10,
            location: formLocation,
            size: {
              w: 325,
              h: 80,
            },
            position: 'relative',
            icon,
            isRequired: false,
            label: '多项选择（可多选）',
            bgColor: '#5AC7F9',
            textColor: '#fff',
            radioType: 2, // '1'：单选，'2'：多选
            list: [{ text: '选项1', label: 1 }],
            optionIndex: 1,
          }));
          newEditor = {
            fCheckboxSet: true,
            isFCheckboxSet: true,
            dragFormCheckboxs: drag10,
            fCheckboxActive: num,
            layoutKey: zIndex + 1,
          };
          break;
        }
        case 11:
        {
          let drag11 = this.$store.state.editor.dragFormSmscodes;
          num = drag11.length;
          layerName = `手机短信${!num ? '' : num + 1}`;
          icon = 'ed-icon-shoujiduanxin';
          drag11 = textActiveOff(drag11, { index: 0, isAll: true });
          drag11.push(Object.assign({}, info, {
            type: 11,
            location: formLocation,
            size: {
              w: 325,
              h: 94,
            },
            position: 'relative',
            icon,
            isRequired: false,
            label: '请输入手机号',
            verify: 1,
            bgColor: '#5AC7F9',
            textColor: '#fff',
          }));
          newEditor = {
            fSmsSet: true,
            isFSmsSet: true,
            dragFormSmscodes: drag11,
            fSmsActive: num,
            layoutKey: zIndex + 1,
          };
          this.$store.commit('editor_update', { isPhone: false });
          break;
        }
        case 12:
        {
          let drag12 = this.$store.state.editor.dragFormSubmits;
          num = drag12.length;
          layerName = `提交按钮${!num ? '' : num + 1}`;
          icon = 'ed-icon-tijiao1';
          drag12 = textActiveOff(drag12, { index: 0, isAll: true });
          drag12.push(Object.assign({}, info, {
            type: 12,
            location: {
              x: (375 - 136) / 2,
              y: top1,
            },
            size: {
              w: 136,
              h: 40,
            },
            position: 'relative',
            icon,
            label: '提交',
            bgColor: '#5AC7F9',
            textColor: '#fff',
          }));
          newEditor = {
            fSubmitSet: true,
            isFSubmitSet: true,
            dragFormSubmits: drag12,
            fSubmitActive: num,
            layoutKey: zIndex + 1,
          };
          this.$store.commit('editor_update', { isSubmit: false });
          break;
        }
        case 13:
        {
          let drag13 = this.$store.state.editor.dragFormUploads || [];
          num = drag13.length;
          layerName = `图片上传${!num ? '' : num + 1}`;
          icon = 'ed-icon-tupianshangchuan';
          drag13 = textActiveOff(drag13, { index: 0, isAll: true });
          drag13.push(Object.assign({}, info, {
            type: 13,
            location: {
              x: (375 - 300) / 2,
              y: top1,
            },
            size: {
              w: 300,
              h: 95,
            },
            position: 'relative',
            icon,
            label: '上传图片',
            bgColor: '#5AC7F9',
            textColor: '#fff',
            isRequired: false,
          }));
          newEditor = {
            fUploadSet: true,
            isFUploadSet: true,
            dragFormUploads: drag13,
            fUploadActive: num,
            layoutKey: zIndex + 1,
          };
          break;
        }
        default:
        {
          break;
        }
      }

      const { layerLists } = this.$store.state.editor;
      const { componentIds } = this.$store.state.page;
      layerLists.unshift({
        display: true, // 是否显示
        lock: true, // 是否可以编辑
        name: `${layerName}`, // 图层名
        id,
        type,
        num,
        zIndex,
        editing: false,
        icon,
      });
      componentIds.unshift({
        name: `${layerName}`,
        id,
      });
      this.$store.commit('editor_update', Object.assign({}, updateEditor, newEditor, {
        layerLists,
        layerActive: 0,
      }));
      if (this.$store.state.page.pageSet) {
        this.$store.commit('page_update', { componentIds, pageSet: false });
      }
    },
    getId() {
      return generate('abcdefghijklmnxyz', 10);
    },
  },
};
</script>

<style>
.ew-component--btn.el-button.el-button--text {
  margin-left: 10px;
}
.ew-left-btns_card .el-card__body {
  padding-bottom: 10px !important;
}
.ew-left-btns_card .ed-icon-images {
  font-size: 26px;
}
.ew-left-btns .el-button {
  width: 80px;
  height: 60px;
  margin-top: 10px;
  margin-left: 10px;
  padding: 0;
  border: 1px solid rgba(0, 0, 0, 0);
  background: none;
}
.ew-left-btns .el-button:hover {
  border: 1px solid #409eff;
}
.ew-left-btns .el-button:active {
  background-color: #1593ff;
  color: #fff;
}
.ew-left-btns .el-button.is-disabled:hover, .ew-left-btns .el-button.is-disabled:active {
  border: 1px solid rgba(0, 0, 0, 0);
  background-color: #fff;
  color: #c0c4cc;
}
.ew-component--icon {
  display: block;
  margin-bottom: 5px;
  font-size: 22px;
}
</style>
