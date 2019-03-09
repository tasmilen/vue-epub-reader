<template>
  <div class="ebook">
    <TitleBar :isTitleAndMenuShow="isTitleAndMenuShow" />
    <div class="read-wrapper">
      <div id="read"></div>
      <div class="mask">
        <div
          class="left"
          @click="prevPage"
        ></div>
        <div
          class="center"
          @click="toggleTitleAndMenu"
        ></div>
        <div
          class="right"
          @click="nextPage"
        ></div>
      </div>

    </div>
    <MenuBar
      :isTitleAndMenuShow="isTitleAndMenuShow"
      :fontSizeList="fontSizeList"
      :defaultFontSize="defaultFontSize"
      @setFontSize="setFontSize"
      :themeList="themeList"
      @setTheme="setTheme"
      :defaultTheme="defaultTheme"
      @onProgressChange="onProgressChange"
      :isBookReady="isBookReady"
      ref="menuBar"
    />
  </div>
</template>
<script>
import Epub from "epubjs";
import TitleBar from "@/components/TitleBar";
import MenuBar from "@/components/MenuBar";
const DOWNLOAD_URL = "/ebook.epub";
// global.epub = Epub
export default {
  components: {
    TitleBar,
    MenuBar
  },
  data() {
    return {
      isTitleAndMenuShow: false,
      fontSizeList: [
        { fontSize: 12 },
        { fontSize: 14 },
        { fontSize: 16 },
        { fontSize: 18 },
        { fontSize: 20 },
        { fontSize: 22 },
        { fontSize: 24 }
      ],
      defaultFontSize: 16,
      themeList: [
        {
          name: "default",
          style: {
            body: {
              color: "#000",
              background: "#fff"
            }
          }
        },
        {
          name: "eye",
          style: {
            body: {
              color: "#000",
              background: "#ceeaba"
            }
          }
        },
        {
          name: "night",
          style: {
            body: {
              color: "#eee",
              background: "#000"
            }
          }
        },
        {
          name: "gold",
          style: {
            body: {
              color: "#000",
              background: "rgb(241,236,226)"
            }
          }
        }
      ],
      defaultTheme: 0,
      //book是否可用状态
      isBookReady: false
    };
  },
  methods: {
    //progress数值 [0-100]
    onProgressChange(progress) {
      const percentage = progress / 100;
      const location =
        percentage > 0 ? this.locations.cfiFromPercentage(percentage) : 0;
      this.rendition.display(location);
    },
    setTheme(index) {
      this.defaultTheme = index;
      this.themes.select(this.themeList[index].name);
    },
    registerThemes() {
      this.themeList.forEach(theme => {
        this.themes.register(theme.name, theme.style);
      });
    },
    setFontSize(fontSize) {
      this.defaultFontSize = fontSize;
      if (this.themes) {
        this.themes.fontSize(fontSize + "px");
      }
    },
    toggleTitleAndMenu() {
      this.isTitleAndMenuShow = !this.isTitleAndMenuShow;
      if (!this.isTitleAndMenuShow) {
        this.$refs.menuBar.hideSetting();
      }
    },
    prevPage() {
      if (this.rendition) {
        this.rendition.prev();
      }
    },
    nextPage() {
      if (this.rendition) {
        this.rendition.next();
      }
    },
    showEpub() {
      // 生成Book
      this.book = new Epub(DOWNLOAD_URL);

      // 生成 Rendition，通过 Book.renderTo()
      this.rendition = this.book.renderTo("read", {
        width: window.innerWidth,
        height: window.innerHeight
      });

      // 通过 Rendition.display 渲染电子书
      this.rendition.display();

      // 获取sheme对象
      this.themes = this.rendition.themes;
      // 设置默认字号
      this.setFontSize(this.defaultFontSize);

      // 设置主题
      // this.themes.rigester(name, style)
      // this.themes.select(name)
      this.registerThemes();
      this.setTheme(this.defaultTheme);

      // 获取locations对象
      // 通过epubjs的钩子函数实现
      this.book.ready
        .then(() => {
          return this.book.locations.generate();
        })
        .then(() => {
          this.locations = this.book.locations;
          this.isBookReady = true;
        });
    }
  },
  mounted() {
    this.showEpub();
  }
};
</script>
<style lang='scss' scoped>
@import "@/assets/styles/global.scss";
.ebook {
  position: relative;
  .read-wrapper {
    .mask {
      position: absolute;
      display: flex;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      z-index: 100;
      .left {
        flex: 0 0 px2rem(100);
      }
      .center {
        flex: 1;
      }
      .right {
        flex: 0 0 px2rem(100);
      }
    }
  }
}
</style>