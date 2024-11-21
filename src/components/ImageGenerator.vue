<template>
  <div class="image-generator">
    <Card style="width: 500px; margin: 0 auto">
      <p slot="title">
        <Icon type="ios-image-outline" />
        图片生成器
      </p>

      <Form :label-width="80">
        <FormItem label="颜色选择">
          <ColorPicker v-model="imageColor" recommend />
        </FormItem>

        <FormItem label="图片宽度">
          <InputNumber
            v-model="width"
            :min="1"
            :max="2000"
            style="width: 200px">
            <span slot="append">px</span>
          </InputNumber>
        </FormItem>

        <FormItem label="图片高度">
          <InputNumber
            v-model="height"
            :min="1"
            :max="2000"
            style="width: 200px">
            <span slot="append">px</span>
          </InputNumber>
        </FormItem>

        <FormItem label="图片格式">
          <Select v-model="imageFormat" style="width: 200px">
            <Option value="png">PNG</Option>
            <Option value="jpeg">JPEG</Option>
            <Option value="webp">WebP</Option>
          </Select>
        </FormItem>

        <FormItem>
          <Button type="primary" @click="generateImage">
            <Icon type="md-create" />
            生成图片
          </Button>
        </FormItem>
      </Form>

      <div v-if="imageUrl" style="text-align: center; margin-top: 20px">
        <Divider>预览</Divider>
        <img :src="imageUrl" style="max-width: 100%; margin-bottom: 10px" />
        <div>
          <Button type="success" @click="downloadImage">
            <Icon type="md-download" />
            下载图片
          </Button>
        </div>
      </div>
    </Card>
  </div>
</template>

<script>
export default {
  name: "ImageGenerator",
  data() {
    return {
      imageColor: "#000000",
      width: 300,
      height: 200,
      imageFormat: "png",
      imageUrl: "",
    };
  },
  methods: {
    generateImage() {
      const canvas = document.createElement("canvas");
      canvas.width = this.width;
      canvas.height = this.height;

      const ctx = canvas.getContext("2d");
      ctx.fillStyle = this.imageColor;
      ctx.fillRect(0, 0, this.width, this.height);

      this.imageUrl = canvas.toDataURL(`image/${this.imageFormat}`);
    },
    downloadImage() {
      const link = document.createElement("a");
      // 移除#号并获取颜色代码
      const colorCode = this.imageColor.replace("#", "");
      link.download = `${this.width}x${this.height}-${colorCode}.${this.imageFormat}`;
      link.href = this.imageUrl;
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    },
  },
};
</script>

<style scoped>
.image-generator {
  padding: 20px;
}
</style>
