<template>
  <div id="app">
    <div class="inner">
      <div class="header mb-4">
        <h1 class="header-inner">怖がらせましょう！ジェネレーター</h1>
      </div>
      <div class="main">
        <canvas class="canvas" ref="canvas" width="1280" height="960" style="display: none;"></canvas>
        <img class="output-image" ref="outputImage" :src="imageDataUrl" alt="Generated Image" />
      </div>

      <div class="d-flex flex-column">
        <div class="mt-3 w-100">
          <label for="textInput" class="form-label mb-0 text-start w-100">文字を入力</label>
          <input class="form-control my-2 w-100" v-model="text" placeholder="好きな文字を入力して友達を怖がらせましょう！" id="textInput" />
        </div>

        <div class="mt-3 w-100">
          <label for="formFile" class="form-label mb-0 text-start w-100">画像を変更したい場合はこちら</label>
          <input type="file" @change="handleImageUpload" accept="image/*" class="form-control my-2 w-100" id="formFile">
        </div>

        <div class="mt-5">
          <p>右クリックメニュー「名前を付けて画像を保存」またはロングタップで保存できます</p>
        </div>

        <div class="footer mt-5">
          <p>Version 1.0.0</p>
          <p>Licensed under the MIT License</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      text: '家でもパーティーでも、エッチな服を着て友達や家族を怖がらせましょう！',
      baseImage: null, // デフォルト画像
      baseImageBubbleOnly: null, // 吹き出しのみの画像
      uploadedImage: null, // アップロードされた画像
      fontsLoaded: false,
      imageDataUrl: '/images/loading.png'
    };
  },
  watch: {
    text() {
      this.drawText();
    },
    uploadedImage() {
      this.drawText(); // アップロード画像が変更されたら再描画
    }
  },
  mounted() {
    this.loadResources();
  },
  methods: {
    async loadResources() {
      await this.loadFonts();
      await this.loadBaseImages();
      this.drawText();
    },
    loadFonts() {
      return new Promise((resolve) => {
        const font0 = new FontFace('Font_0', 'url(/fonts/MPLUS1-Bold.ttf)');
        Promise.all([font0.load()]).then((loadedFonts) => {
          loadedFonts.forEach((font) => document.fonts.add(font));
          this.fontsLoaded = true;
          resolve();
        });
      });
    },
    loadBaseImages() {
      return Promise.all([
        this.loadImage('/images/base_1.png', 'baseImage'),
        this.loadImage('/images/base_2.png', 'baseImageBubbleOnly')
      ]);
    },
    loadImage(src, dataName) {
      return new Promise((resolve) => {
        const img = new Image();
        img.src = src;
        img.onload = () => {
          this.$data[dataName] = img;
          resolve();
        };
      });
    },
    handleImageUpload(event) {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (e) => {
          this.uploadedImage = new Image();
          this.uploadedImage.onload = () => {
            this.drawText();
          };
          this.uploadedImage.src = e.target.result;
        };
        reader.readAsDataURL(file);
      } else {
        this.uploadedImage = null;
        this.drawText();
      }
    },
    drawText() {
      if (!this.fontsLoaded) return;
      const canvas = this.$refs.canvas;
      const ctx = canvas.getContext('2d');
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      if (this.uploadedImage && this.baseImageBubbleOnly) {
        ctx.drawImage(this.baseImageBubbleOnly, 0, 0, canvas.width, canvas.height);

        // アップロードされた画像を右下に描画
        const uploadedWidth = 400;
        const uploadedHeight = this.uploadedImage.height * (uploadedWidth / this.uploadedImage.width);
        const uploadedX = canvas.width - uploadedWidth - 120;
        const uploadedY = canvas.height - uploadedHeight - 20;
        ctx.drawImage(this.uploadedImage, uploadedX, uploadedY, uploadedWidth, uploadedHeight);
      } else if (this.baseImage) {
        ctx.drawImage(this.baseImage, 0, 0, canvas.width, canvas.height);
      }

      ctx.fillStyle = '#546674';
      ctx.font = '50px Font_0';
      ctx.textAlign = 'left';
      ctx.textBaseline = 'middle';

      // 自動改行のための設定
      const text = this.text || '';
      const x = 170; // 吹き出しX座標
      const maxWidth = 550; // 吹き出しの最大幅
      const lineHeight = 70; // 行の高さ

      // テキストを行に分割する
      const lines = [];
      let currentLine = '';
      const chars = text.split('');

      for (let i = 0; i < chars.length; i++) {
        const char = chars[i];
        const testLine = currentLine + char;
        const metrics = ctx.measureText(testLine);

        if (metrics.width > maxWidth && i > 0) {
          lines.push(currentLine);
          currentLine = char;
        } else {
          currentLine = testLine;
        }
      }
      lines.push(currentLine);

      const startY = 250;

      lines.forEach((line, index) => {
        ctx.fillText(line, x, startY + (index * lineHeight));
      });

      // 最終的な画像を生成
      this.imageDataUrl = canvas.toDataURL('image/png');
    },
  },
};
</script>

<style>
.inner {
  text-align: center;
}

.output-image {
  max-width: 100%;
  border: 1px solid #ccc;
  background-color: #f0f0f0;
}
</style>