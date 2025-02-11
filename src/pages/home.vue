<script setup lang="ts">
const OPTIONS = ["其他", "操作問題", "優化建議", "BUG反饋"];

interface State {
  isShowDialog: boolean;
  selected: string;
  title: string;
  content: string;
  images: { isHover: boolean; url: string }[];
  previewImage: string;
  isShowPreviewImage: boolean;
  isShowSuccessDialog: boolean;
}

const state: State = reactive({
  isShowDialog: false,
  selected: "其他",
  title: "",
  content: "",
  images: [],
  previewImage: "",
  isShowPreviewImage: false,
  isShowSuccessDialog: false
});

const fileInput = ref<HTMLInputElement>();

const onToggleDialog = () => {
  state.isShowDialog = !state.isShowDialog;
};

const onDelImage = (index: number) => {
  state.images.splice(index, 1);
};

const onPreviewImage = (url) => {
  state.previewImage = url;
  state.isShowPreviewImage = true;
  console.log("preview image", url);
};

const handleFileSelect = (event) => {
  const file = event.target.files[0];
  if (file) {
    processFile(file);
  }
};

const triggerFileInput = () => {
  fileInput.value.click();
};

const processFile = (file) => {
  const reader = new FileReader();

  reader.onload = (e) => {
    state.images.push({
      url: e.target.result as string,
      isHover: false
    });
  };

  if (file.type.startsWith("image/")) {
    reader.readAsDataURL(file);
  } else {
    reader.readAsText(file);
  }
};

const onSubmit = () => {
  const closeTime = 5000;
  state.isShowDialog = false;
  state.isShowSuccessDialog = true;
  setTimeout(() => {
    state.isShowSuccessDialog = false;
  }, closeTime);
};
</script>

<template>
  <div class="home">
    <div class="icon-wrap" @click="onToggleDialog">
      <q-icon name="priority_high" color="white" size="32px"></q-icon>
    </div>
    <q-dialog v-model="state.isShowDialog">
      <q-card class="home-dialog">
        <div class="home-dialog-title">
          <span>意見反饋</span>
        </div>
        <q-card-section>
          <div class="home-dialog-col">
            <div class="home-dialog-label">
              <q-icon name="emergency" class="home-dialog-label-icon"></q-icon>
              <span class="home-dialog-label-text">類別</span>
            </div>
            <q-select
              square
              outlined
              v-model="state.selected"
              :options="OPTIONS"
            ></q-select>
          </div>
          <div class="home-dialog-col">
            <div class="home-dialog-label">
              <q-icon name="emergency" class="home-dialog-label-icon"></q-icon>
              <span class="home-dialog-label-text">標題</span>
              <span class="home-dialog-label-tip">(限定30個字符內)</span>
            </div>
            <q-input square outlined v-model="state.title"></q-input>
          </div>
          <div class="home-dialog-col">
            <div class="home-dialog-label">
              <q-icon name="emergency" class="home-dialog-label-icon"></q-icon>
              <span class="home-dialog-label-text">描述</span>
              <span class="home-dialog-label-tip">(限定300個字符內)</span>
            </div>
            <q-input
              square
              outlined
              type="textarea"
              v-model="state.content"
            ></q-input>
          </div>
          <div class="home-dialog-col">
            <div class="home-dialog-label">
              <span class="home-dialog-label-text">參考圖片</span>
              <span class="home-dialog-label-tip"
                >(僅限定PNG、JPG格式，每張5Ｍ內)</span
              >
            </div>
            <div class="home-dialog-images">
              <div
                class="home-dialog-images-img"
                v-for="(img, index) in state.images"
                :key="index"
                @mouseenter="img.isHover = true"
                @mouseleave="img.isHover = false"
              >
                <img :src="img.url" />
                <div class="mask" v-if="state.images[index].isHover">
                  <q-icon
                    name="search"
                    color="white"
                    size="20px"
                    @click="onPreviewImage(img.url)"
                  ></q-icon>
                  <q-icon
                    name="delete"
                    color="white"
                    size="20px"
                    @click="onDelImage(index)"
                  ></q-icon>
                </div>
              </div>
              <div class="home-dialog-images-add">
                <input
                  type="file"
                  ref="fileInput"
                  @change="handleFileSelect"
                  accept="image/*,text/*"
                  class="hidden-input"
                  style="display: none"
                />
                <q-icon
                  @click="triggerFileInput"
                  name="add"
                  color="grey"
                  size="30px"
                ></q-icon>
              </div>
            </div>
          </div>
        </q-card-section>
        <q-card-section>
          <q-btn
            style="width: 100%"
            color="primary"
            label="送出"
            @click="onSubmit"
          />
        </q-card-section>
      </q-card>
    </q-dialog>
    <q-dialog v-model="state.isShowPreviewImage">
      <q-card>
        <q-card-section class="preview-img-title">
          <q-icon
            name="close"
            color="grey"
            size="20px"
            @click="state.isShowPreviewImage = false"
          ></q-icon>
        </q-card-section>
        <div class="preview-img">
          <img :src="state.previewImage" />
        </div>
      </q-card>
    </q-dialog>
    <q-dialog v-model="state.isShowSuccessDialog">
      <q-card class="success-dialog">
        <div class="success-dialog-title">
          <div>
            <q-img
              src="@/assets/images/success.png"
              style="width: 30px; height: 30px"
            ></q-img>
            <span>提交成功</span>
          </div>
          <q-icon
            name="close"
            color="grey"
            size="20px"
            @click="state.isShowSuccessDialog = false"
          ></q-icon>
        </div>
        <div class="success-dialog-content">
          感謝你的反饋，我們將繼續提供更優質的服務！
        </div>
      </q-card>
    </q-dialog>
  </div>
</template>

<style scoped lang="scss">
.home {
  position: relative;
  width: 100%;
  height: 100vh;
  &-dialog {
    width: 500px;
    padding: 20px;
    &-col {
      display: flex;
      flex-direction: column;
      margin-top: 10px;
    }
    &-title {
      span {
        font-size: 1.5rem;
        color: #4f545b;
      }
    }
    &-label {
      margin-bottom: 10px;
      display: flex;
      align-items: center;
      &-icon {
        font-size: 1.2rem;
        color: #ef103a;
      }
      &-text {
        margin-left: 5px;
        font-size: 1.2rem;
        color: #565a61;
      }
      &-tip {
        margin-left: 5px;
        font-size: 1rem;
        color: #565a61;
      }
    }
    &-images {
      display: flex;
      align-items: center;
      gap: 6px;
      &-add {
        display: flex;
        align-items: center;
        justify-content: center;
        width: 100px;
        height: 100px;
        border: 1px dashed #d9d9d9;
        border-radius: 8px;
      }
      &-img {
        position: relative;
        width: 100px;
        height: 100px;
        border-radius: 8px;
        overflow: hidden;
        img {
          width: 100%;
          height: 100%;
          object-fit: cover;
        }
        .mask {
          display: flex;
          align-items: center;
          justify-content: center;
          gap: 6px;
          position: absolute;
          top: 0;
          left: 0;
          width: 100%;
          height: 100%;
          background-color: rgba(0, 0, 0, 0.5);
        }
      }
    }
  }
}

.preview-img {
  padding: 20px;
  background: #ffffff;
  &-title {
    display: flex;
    justify-content: flex-end;
  }
  img {
    width: 300px;
    height: 300px;
  }
}

.success-dialog {
  padding: 30px;
  &-title {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 10px;
    color: #21ba45;
  }
}

.icon-wrap {
  position: absolute;
  right: 40px;
  bottom: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
  background: #0052cc;
  border-radius: 50%;
}
</style>
