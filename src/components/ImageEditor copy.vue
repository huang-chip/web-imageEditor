<template>
  <div class="image-editor">
    <Card>
      <Row :gutter="24">
        <Col span="16">
          <div class="preview-container" v-if="imageUrl">
            <vue-cropper
              ref="cropper"
              :key="cropperKey"
              :src="imageUrl"
              :view-mode="2"
              :auto-crop-area="0.8"
              :background="true"
              :movable="true"
              :zoomable="true"
              :center="true"
              :responsive="true"
              :restore="false"
              :canScale="false"
              :canMoveBox="false"
              :checkCrossOrigin="true"
              :output-size="outputSize"
              :output-type="outputType"
              :full="true"
              :fixedBox="isFixed"
              @ready="handleReady"
              @crop="handleCropChange" />
          </div>
          <div v-else class="empty-container">
            <Icon type="ios-images" size="64" />
            <p>请上传图片</p>
          </div>
        </Col>
        <Col span="8">
          <Card>
            <p slot="title">
              <Icon type="ios-cloud-upload" />
              图片上传
            </p>
            <Alert show-icon>
              支持从剪贴板粘贴、本地上传或输入远程图片地址
            </Alert>
            <div class="input-controls">
              <Upload
                action=""
                :before-upload="handleLocalFile"
                :show-upload-list="false"
                accept="image/*">
                <Button icon="ios-folder-open" long>本地图片</Button>
              </Upload>
              <Button
                @click="handlePaste"
                icon="ios-copy"
                long
                style="margin: 10px 0">
                粘贴图片
              </Button>
              <Input
                v-model="remoteUrl"
                placeholder="输入图片URL"
                style="margin-bottom: 10px">
                <Button
                  slot="append"
                  @click="handleRemoteImage"
                  icon="ios-download-outline">
                  加载
                </Button>
              </Input>
            </div>
          </Card>

          <Card style="margin-top: 16px" v-if="imageUrl">
            <p slot="title">
              <Icon type="ios-settings" />
              编辑选项
            </p>
            <Form :label-width="80">
              <FormItem label="压缩率">
                <Slider v-model="outputSize" :min="0.1" :max="1" :step="0.1" />
              </FormItem>
              <FormItem label="输出格式">
                <Select v-model="outputType" style="width: 200px">
                  <Option value="jpeg">JPEG</Option>
                  <Option value="png">PNG</Option>
                  <Option value="webp">WebP</Option>
                </Select>
              </FormItem>
              <FormItem label="位置">
                <div class="position-inputs">
                  <div class="input-group">
                    <span>Left:</span>
                    <InputNumber
                      v-model="cropperLeft"
                      :min="0"
                      :precision="0"
                      @on-change="handlePositionChange" />
                  </div>
                  <div class="input-group">
                    <span>Top:</span>
                    <InputNumber
                      v-model="cropperTop"
                      :min="0"
                      :precision="0"
                      @on-change="handlePositionChange" />
                  </div>
                </div>
              </FormItem>
              <FormItem label="输出宽度">
                <div class="size-input">
                  <InputNumber
                    v-model="width"
                    :min="1"
                    :max="naturalWidth"
                    :precision="0"
                    @on-change="handleSizeChange" />
                  <span class="original-size">{{ naturalWidth }}px</span>
                </div>
              </FormItem>
              <FormItem label="输出高度">
                <div class="size-input">
                  <InputNumber
                    v-model="height"
                    :min="1"
                    :max="naturalHeight"
                    :precision="0"
                    @on-change="handleSizeChange" />
                  <span class="original-size">{{ naturalHeight }}px</span>
                </div>
              </FormItem>
              <FormItem>
                <Button
                  type="primary"
                  icon="ios-crop"
                  @click="handleCrop"
                  style="margin-right: 10px">
                  裁剪
                </Button>
                <Button
                  type="success"
                  icon="ios-download"
                  @click="handleDownload"
                  :disabled="!croppedImage">
                  下载
                </Button>
              </FormItem>
            </Form>
          </Card>
        </Col>
      </Row>

      <Modal v-model="previewVisible" title="预览" footer-hide width="800">
        <div style="text-align: center">
          <img
            v-if="croppedImage"
            :src="croppedImage"
            style="max-width: 100%; max-height: 600px; object-fit: contain" />
        </div>
      </Modal>
    </Card>
  </div>
</template>

<script>
import VueCropper from "vue-cropperjs";
import "cropperjs/dist/cropper.css";

export default {
  name: "ImageEditor",
  components: {
    VueCropper,
  },
  data() {
    return {
      imageUrl: "",
      remoteUrl: "",
      originalName: "",
      width: 0,
      height: 0,
      naturalWidth: 0,
      naturalHeight: 0,
      outputSize: 1,
      outputType: "jpeg",
      croppedImage: null,
      previewVisible: false,
      cropperKey: 0,
      maxImageSize: 5000,
      cropperLeft: 0,
      cropperTop: 0,
      isCropping: false,
    };
  },
  methods: {
    handleLocalFile(file) {
      if (!this.validateImageFile(file)) return false;
      const reader = new FileReader();
      reader.onload = (e) => {
        this.imageUrl = e.target.result;
        this.originalName = file.name;
        this.cropperKey += 1;
      };
      reader.readAsDataURL(file);
      return false;
    },

    validateImageFile(file) {
      if (!file.type.startsWith("image/")) {
        this.$Message.error("请选择图片文件");
        return false;
      }
      return true;
    },

    handlePaste() {
      navigator.clipboard
        .read()
        .then((items) => {
          for (const item of items) {
            if (
              item.types.includes("image/png") ||
              item.types.includes("image/jpeg")
            ) {
              item.getType("image/png").then((blob) => {
                if (!this.validateImageFile(blob)) return;
                const reader = new FileReader();
                reader.onload = (e) => {
                  this.imageUrl = e.target.result;
                  this.originalName = "pasted_image.png";
                  this.cropperKey += 1;
                };
                reader.readAsDataURL(blob);
              });
              break;
            }
          }
        })
        .catch(() => {
          this.$Message.error("无法访问剪贴板");
        });
    },

    async handleRemoteImage() {
      if (!this.remoteUrl) {
        this.$Message.warning("请输入图片URL");
        return;
      }

      const loadImage = (url, withCrossOrigin = true) => {
        return new Promise((resolve, reject) => {
          const img = new Image();
          if (withCrossOrigin) {
            img.crossOrigin = "anonymous";
          }

          img.onload = () => {
            // 如果不需要跨域，直接返回图片
            if (!withCrossOrigin) {
              resolve(img);
              return;
            }

            // 尝试转换为 base64
            const canvas = document.createElement("canvas");
            canvas.width = img.width;
            canvas.height = img.height;

            const ctx = canvas.getContext("2d");
            try {
              ctx.drawImage(img, 0, 0);
              canvas.toDataURL("image/png"); // 测试是否可以转换
              resolve(img);
            } catch (e) {
              reject(new Error("CORS error"));
            }
          };

          img.onerror = () => reject(new Error("Load failed"));

          // 添加时间戳避免缓存
          const timestamp = new Date().getTime();
          const urlWithTimestamp =
            url + (url.includes("?") ? "&" : "?") + "timestamp=" + timestamp;
          img.src = urlWithTimestamp;
        });
      };

      try {
        // 首先尝试带 crossOrigin 加载
        const img = await loadImage(this.remoteUrl, true).catch(() => {
          // 如果失败，尝试不带 crossOrigin 加载
          return loadImage(this.remoteUrl, false);
        });

        // 创建 canvas 转换图片
        const canvas = document.createElement("canvas");
        canvas.width = img.width;
        canvas.height = img.height;

        const ctx = canvas.getContext("2d");
        ctx.drawImage(img, 0, 0);

        try {
          this.imageUrl = canvas.toDataURL("image/png");
          this.originalName =
            this.remoteUrl.split("/").pop() || "remote_image.png";
          this.cropperKey += 1;
        } catch (e) {
          this.$Message.error("无法处理该图片，请尝试下载后再上传");
        }
      } catch (error) {
        this.$Message.error("图片加载失败，请检查链接是否有效");
      }
    },

    handleReady() {
      const imageData = this.$refs.cropper.getImageData();
      this.naturalWidth = Math.round(imageData.naturalWidth);
      this.naturalHeight = Math.round(imageData.naturalHeight);

      this.width = Math.round(this.naturalWidth * 0.8);
      this.height = Math.round(this.naturalHeight * 0.8);

      this.$nextTick(() => {
        if (this.$refs.cropper) {
          const containerData = this.$refs.cropper.getContainerData();
          if (containerData) {
            const cropBoxData = {
              left: (containerData.width - this.width) / 2,
              top: (containerData.height - this.height) / 2,
              width: this.width,
              height: this.height,
            };
            this.$refs.cropper.setCropBoxData(cropBoxData);
          }
        }
      });
    },

    handleSizeChange() {
      if (!this.$refs.cropper) return;

      const cropper = this.$refs.cropper.cropper;
      const data = cropper.getData();

      cropper.setData({
        x: this.cropperLeft,
        y: this.cropperTop,
        width: Math.round(this.width),
        height: data.height,
        rotate: 0,
      });

      const cropBoxData = cropper.getCropBoxData();
      this.cropperLeft = Math.round(cropBoxData.left);
      this.cropperTop = Math.round(cropBoxData.top);
    },

    handlePositionChange() {
      if (!this.$refs.cropper) return;

      const cropper = this.$refs.cropper.cropper;
      cropper.setData({
        x: this.cropperLeft,
        y: this.cropperTop,
        width: Math.round(this.width),
        height: Math.round(this.height),
        rotate: 0,
      });
    },

    handleCrop() {
      const options = {
        width: this.width,
        height: this.height,
        imageSmoothingQuality: "high",
        fillColor: this.outputType === "jpeg" ? "#fff" : undefined,
      };

      this.$refs.cropper.getCroppedCanvas(options).toBlob(
        (blob) => {
          const reader = new FileReader();
          reader.onload = (e) => {
            this.croppedImage = e.target.result;
            this.previewVisible = true;
          };
          reader.readAsDataURL(blob);
        },
        `image/${this.outputType}`,
        this.outputSize
      );
    },

    handleDownload() {
      if (!this.croppedImage) return;

      const link = document.createElement("a");
      const nameParts = this.originalName.split(".");
      const ext = this.outputType;
      const baseName = nameParts[0];
      link.download = `${this.width}x${this.height}_${baseName}.${ext}`;
      link.href = this.croppedImage;
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    },

    handleCropChange(data) {
      if (!this.isCropping) {
        this.width = Math.round(data.detail.width);
        this.height = Math.round(data.detail.height);
        const cropBoxData = this.$refs.cropper.getCropBoxData();
        this.cropperLeft = Math.round(cropBoxData.left);
        this.cropperTop = Math.round(cropBoxData.top);
      }
    },
  },
};
</script>

<style scoped>
.image-editor {
  padding: 20px;
}

.preview-container {
  min-height: 500px;
  background: #f8f8f9;
  overflow-y: auto;
  position: relative;
}

.empty-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 500px;
  background: #f8f8f9;
  color: #999;
}

.input-controls {
  margin: 16px 0;
  display: flex;
  flex-direction: column;
}

.vue-cropper {
  width: 100%;
  height: 100%;
}

.size-input {
  display: flex;
  align-items: center;
  gap: 8px;
}

.original-size {
  min-width: 60px;
  color: #999;
  font-size: 12px;
}

.position-inputs {
  display: flex;
  gap: 16px;
}

.input-group {
  display: flex;
  align-items: center;
  gap: 8px;
}

.input-group span {
  min-width: 40px;
  color: #666;
}
</style>
