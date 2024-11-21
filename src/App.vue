<template>
  <div id="app">
    <Card>
      <Tabs v-model="activeTab">
        <TabPane label="图片生成器" name="generator" icon="ios-create">
          <ImageGenerator />
        </TabPane>
        <TabPane label="图片编辑器" name="editor" icon="ios-images">
          <ImageEditor />
        </TabPane>
      </Tabs>
    </Card>
  </div>
</template>

<script>
import ImageGenerator from "./components/ImageGenerator.vue";
import ImageEditor from "./components/ImageEditor.vue";

export default {
  name: "App",
  components: {
    ImageGenerator,
    ImageEditor,
  },
  data() {
    return {
      activeTab: "generator",
    };
  },
};
</script>

<style>
#app {
  padding: 20px;
}
</style>
