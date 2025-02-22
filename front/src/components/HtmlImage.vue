<template>
  <Card dis-hover :padding="0" class="img">
    <template v-if="imageStatus == 0">
      <div class="img-error img-box">
        <Badge class="user-select-none">
          <Icon type="md-refresh" class="spin-icon-load" slot="count" size="20"/>
          <Icon type="md-images" size="50"/>
        </Badge>
        <img class="user-select-none" style="display: none" :src="src" @error="onError" @load="onLoad"/>
        <p class="img-box-url">{{ src }}</p>
      </div>
    </template>
    <template v-else-if="imageStatus == 1">
      <div class="img-box">
        <div class="img-toolbar">
          <Row>
            <Col>
              <a class="user-select-none" size="small" type="dashed" href="javascript:void(0)" @click="onRotating(-90)">
                <Icon type="md-redo" size="15" style="transform: rotate(-180deg)"/>
              </a>
              <Divider type="vertical"></Divider>
              <a class="user-select-none" size="small" type="dashed" href="javascript:void(0)" @click="onRotating(90)">
                <Icon type="md-redo" size="15"/>
              </a>
              <Divider type="vertical" v-if="rotateValue != 0"></Divider>
              <a v-if="rotateValue != 0" class="user-select-none" size="small" type="dashed" href="javascript:void(0)" @click="onRotating(0)">
                <Icon type="md-refresh" size="15"/>
              </a>
            </Col>
            <Col flex="1" class="img-title">
              <span>{{ src }}</span>
            </Col>
            <Col v-if="src" class="user-select-none">
              <a size="small" type="dashed" :href="src" target="_new">
                <Icon type="md-link"/>
              </a>
            </Col>
          </Row>
        </div>

        <img :src="src" class="img-tag user-select-none" :style="`transform: rotate(${rotateValue}deg)`"/>
        <div class="img-hover ivu-card user-select-none" @click="show">
          <Icon type="ios-search" size="50"/>
        </div>
      </div>
    </template>
    <template v-else-if="imageStatus == -1">
      <div class="img-error img-box" @click="openUrl">
        <Badge class="user-select-none">
          <Icon type="md-alert" slot="count" size="20"/>
          <Icon type="md-images" size="50"/>
        </Badge>
        <p class="img-box-url">{{ src }}</p>
      </div>
    </template>
  </Card>
</template>

<script>
import 'viewerjs/dist/viewer.css'

import VueViewer from 'v-viewer'
import Vue from "vue";
import {regular} from "@/assets/js";

Vue.use(VueViewer);

export default {
  name: "HtmlImage",
  props: {
    src: {
      type: String,
      default: ""
    },
    index: {
      type: String,
      default: "0"
    },
    images: {
      type: String,
      default: ""
    }
  },
  data() {
    return {
      load: false,
      rotateValue: 0,
      imageStatus: 0,
      viewImages: [],
    }
  },
  watch: {
    images: {
      handler(val) {
        this.viewImages = val.split(",");
      },
      deep: true
    },
  },
  created() {
    this.viewImages = this.images.split(",");
  },
  methods: {
    onLoad() {
      this.imageStatus = 1;
    },
    onError() {
      this.imageStatus = -1;
    },
    onRotating(value) {
      if (this.rotateValue >= 360 || this.rotateValue <= -360 || value == 0) {
        this.rotateValue = 0;
        return;
      }
      this.rotateValue += value
    },
    openUrl() {
      if (this.src)
        window.open(this.src);
    },
    show() {
      if (this.imageStatus <= 0) return;

      this.$viewerApi({
        options: {
          toolbar: false,
          navbar: true,
          keyboard: false,
          fullscreen: true,
          initialViewIndex: Number(this.index),
        },
        images: this.viewImages,
      })
    }
  }
}
</script>

<style lang="less" scoped>
@import "@/assets/css/icon.less";

.img {
  position: relative;
  width: calc(100% + 16px);
  margin: 10px -8px 10px -8px;
  overflow: hidden;
  min-height: 80px;
  cursor: pointer;

  img {
    width: 100%;
    display: block;
  }

  .img-toolbar {
    cursor: auto;
    padding: 5px 10px;
    position: relative;
    z-index: 11;
  }

  .img-box {
    margin-top: -18px;
  }

  .img-title {
    padding: 0 20px;
    text-align: center;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
  }

  .img-box-url {
    padding: 0 15px;
    text-overflow: ellipsis;
    white-space: nowrap;
    overflow: hidden;
    line-clamp: 2;
  }

  .img-error {
    text-align: center;
    padding: 20px 0;
  }

  .img-hover {
    cursor: pointer;
    border-radius: 0 !important;
    display: flex;
    justify-content: center;
    align-content: center;
    flex-direction: column;
    position: absolute;
    z-index: 1;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    transition: all .25s;
    opacity: 0;
  }
}

.img:hover {
  .img-hover {
    opacity: .5;
  }
}
</style>