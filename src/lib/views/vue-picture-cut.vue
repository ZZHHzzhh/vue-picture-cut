<template>
  <div class="vue-picture-cut2">
    <div class="vue-picture-cut2_main" ref="main" :style="[ hasMenu || { bottom: '50px' } ]">
      <vue-picture-cut-canvas :angle="initAngle"/>
      <slot>
        <vue-picture-cut-mask v-bind="mskOption"/>
      </slot>
    </div>
    <div class="vue-picture-cut2_menu-box" :style="[ hasMenu || { height: '50px' } ]">
      <slot name="menu">
        <div class="vue-picture-cut2_default-menu">
          <div class="vue-picture-cut2_slider"
               v-if="rotateControl">
            <input type="range" v-model="sliderAngle" :min="-180" :max="180"/>
            <div class="vue-picture-cut2_slider-box">
              <div class="vue-picture-cut2_slider-box-bar"
                   :style="{left: sliderAngle * 100 / 361 + 50 + '%'}">
                <div class="vue-picture-cut2_slider-box-tips">
                  {{ sliderAngle }}°
                </div>
              </div>
            </div>
          </div>
          <div v-show="src" class="vue-picture-cut2_button" @click="sureCut">确定</div>
        </div>
      </slot>
    </div>
  </div>
</template>

<script lang="ts">
import {Component, Vue, Provide, Ref, Prop, Watch, Emit} from 'vue-property-decorator';
import PhotoRoot from './PhotoRoot';
import PhotoMain from './PhotoMain';
import PhotoMask from './PhotoMask';
import VuePictureCutCanvas from './vue-picture-cut-canvas.vue';
import VuePictureCutMask from './vue-picture-cut-mask.vue';

interface MskOption {
  width?: number;
  height?: number;
  isRound?: boolean;
  resize?: boolean;
}

@Component({
  components: {
    VuePictureCutCanvas,
    VuePictureCutMask
  }
})
export default class VuePictureCut extends Vue {

  // 缩放率
  @Prop({ type: Number, default: 1.5 }) private magnification!: number;
  // 图片
  @Prop({ type: String, default: null }) private src!: string | null;
  // 旋转
  @Prop({ type: Number, required: false }) private initAngle: number | undefined;
  // 是否显示旋转控件
  @Prop({ type: Boolean, required: false }) private rotateControl!: boolean;
  // 遮罩
  @Prop({
    type: Object,
    default () {
      return {
        width: 1,
        height: 1,
        isRound: false,
        resize: true
      }
    }
  }) private mskOption!: MskOption;

  @Ref() private main!: HTMLDivElement;

  // 是否有菜单组件
  hasMenu = false;
  // 角度
  sliderAngle = 0;
  sliderHeight = '0';
  sliderBottom = '0';

  photoRoot: PhotoRoot = new PhotoRoot();

  @Provide()
  vuePictureCut = this.photoRoot;

  setImg(): void {
    const photoMain = this.photoRoot.getEventList<PhotoMain>('PhotoMain');
    if (this.src && photoMain) {
      photoMain.setSrc(this.src, this.initAngle);
    }
  }

  @Watch('src')
  watchSrc (to: string | null): void {
    if (to) {
      this.setImg();
    }
  }

  @Watch('initAngle')
  watchInitAngle (to: number | undefined): void {
    if (to === undefined) return;
    this.sliderAngle = to % 360;
  }

  @Watch('sliderAngle')
  watchSliderAngle (to: string): void {
    const photoMain = this.photoRoot.getEventList<PhotoMain>('PhotoMain');
    if (photoMain) {
      photoMain.setAngle(parseInt(to));
    }
  }

  /*******生命周期********/
  protected created (): void {
    this.watchInitAngle(this.initAngle);
    if (this.$slots.menu) {
      this.hasMenu = true;
    }
  }

  protected mounted (): void {
    this.photoRoot.init(this.main, this.magnification);
    this.setImg();
    this.sliderHeight = this.main.offsetHeight * 0.8 + 'px';
    this.sliderBottom = this.main.offsetHeight * 0.1 + 'px';
  }

  /*******事件********/
  @Emit('on-change')
  onChangeEvent (blob: Blob, base64: string): {blob: Blob, base64: string} {
    return {blob, base64};
  }

  /**********方法**********/

  // 默认裁剪
  sureCut(): void{
    const mask = this.photoRoot.getEventList<PhotoMask>('PhotoMask');
    if (mask) {
      const result = mask.clip();
      if (result) {
        this.onChangeEvent(result.file, result.src);
      }
    }
  }

}
</script>

<style lang="scss">
@import "../styles/center";
@import "../styles/1px";
@import "../styles/font";
@import "../styles/vue-picture-cut2.scss";
</style>