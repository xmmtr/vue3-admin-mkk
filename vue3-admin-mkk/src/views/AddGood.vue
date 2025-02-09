<template>
  <div class="add">
    <el-card class="add-container">
      <el-form
        :model="state.commodityForm"
        :rules="state.rules"
        ref="goodRef"
        label-width="100px"
        class="commodityForm"
      >
        <el-form-item label="商品分类" props="commodityType">
          <el-select-v2
            v-model="state.commodityForm.commodityType"
            :options="options"
            style="width: 300px"
            @change="handleChangeCate"
            clearable
          ></el-select-v2>
        </el-form-item>
        <el-form-item label="商品名称" prop="commodityname">
          <el-input
            style="width: 300px"
            v-model="state.commodityForm.commodityname"
            placeholder="请输入商品名称"
          ></el-input>
        </el-form-item>
        <el-form-item label="商品价格" prop="commoditymoney">
          <el-input
            type="number"
            min="0"
            style="width: 300px"
            v-model="state.commodityForm.commoditymoney"
            placeholder="请输入商品价格"
          ></el-input>
        </el-form-item>
        <el-form-item label="商品口味" prop="flavor">
          <el-input
            style="width: 300px"
            v-model="state.commodityForm.flavor"
            placeholder="请输入商品口味"
          ></el-input>
        </el-form-item>
        <el-form-item label="商品分量" prop="weight">
          <el-input
            style="width: 300px"
            v-model="state.commodityForm.weight"
            placeholder="请输入商品原料"
          />
        </el-form-item>
        <el-form-item label="商品原料" prop="material">
          <el-input
            style="width: 300px"
            v-model="state.commodityForm.material"
            placeholder="请输入商品原料"
          ></el-input>
        </el-form-item>
        <el-form-item label="商品主图" prop="commodityimage">
          <el-upload
            action="#"
            list-type="picture-card"
            :auto-upload="false"
            :limit="1"
            v-model:file-list="fileList"
          >
            <el-icon><Plus /></el-icon>
            <template #file="{ file }">
              <div>
                <img
                  class="el-upload-list__item-thumbnail"
                  :src="file.url"
                  alt=""
                />
                <span class="el-upload-list__item-actions">
                  <span
                    class="el-upload-list__item-preview"
                    @click="handlePictureCardPreview(file)"
                  >
                    <el-icon><zoom-in /></el-icon>
                  </span>
                  <span
                    v-if="!disabled"
                    class="el-upload-list__item-delete"
                    @click="handleRemove(file)"
                  >
                    <el-icon><Delete /></el-icon>
                  </span>
                </span>
              </div>
            </template>
          </el-upload>
        </el-form-item>
        <el-form-item label="商品简介">
          <div ref="editor"></div>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="submitAdd()">{{
            state.id ? "立即修改" : "立即创建"
          }}</el-button>
        </el-form-item>
      </el-form>
      <el-dialog v-model="dialogVisible">
        <img w-full :src="dialogImageUrl" alt="Preview Image" />
      </el-dialog>
    </el-card>
  </div>
</template>

<script setup>
import {
  reactive,
  ref,
  onMounted,
  onBeforeUnmount,
  getCurrentInstance,
} from "vue";
import WangEditor from "wangeditor";
import axios from "@/utils/axios";
import { ElMessage } from "element-plus";
import { useRoute, useRouter } from "vue-router";
import { localGet, uploadImgServer, uploadImgsServer } from "@/utils";
import { Delete, Download, Plus, ZoomIn } from "@element-plus/icons-vue";

const { proxy } = getCurrentInstance();
const editor = ref(null);
const goodRef = ref(null);
const route = useRoute();
const router = useRouter();
const { id } = route.query;
const state = reactive({
  uploadImgServer,
  token: localGet("token") || "",
  id: id,
  commodityForm: {
    id: "",
    commodityname: "",
    commodityintro: "",
    commoditymoney: "",
    commodityType: "",
    flavor: "",
    commodityimage: "",
    material: "",
    weight: "",
  },
  rules: {
    commodityname: [
      { required: "true", message: "请填写商品名称", trigger: ["change"] },
    ],
    commoditymoney: [
      { required: "true", message: "请填写商品售价", trigger: ["change"] },
    ],
    commodityType: [
      { required: "true", message: "请选择商品分类", trigger: ["change"] },
    ],
  },
});

let instance;
const options = ref([]);
onMounted(() => {
  instance = new WangEditor(editor.value);
  instance.config.showLinkImg = false;
  instance.config.showLinkImgAlt = false;
  instance.config.showLinkImgHref = false;
  instance.config.uploadImgMaxSize = 2 * 1024 * 1024; // 2M
  instance.config.uploadFileName = "file";
  instance.config.uploadImgHeaders = {
    token: state.token,
  };
  // 图片返回格式不同，需要自定义返回格式
  instance.config.uploadImgHooks = {
    // 图片上传并返回了结果，想要自己把图片插入到编辑器中
    // 例如服务器端返回的不是 { errno: 0, data: [...] } 这种格式，可使用 customInsert
    customInsert: function (insertImgFn, result) {
      console.log("result", result);
      // result 即服务端返回的接口
      // insertImgFn 可把图片插入到编辑器，传入图片 src ，执行函数即可
      if (result.data && result.data.length) {
        result.data.forEach((item) => insertImgFn(item));
      }
    },
  };
  instance.config.uploadImgServer = uploadImgsServer;
  Object.assign(instance.config, {
    onchange() {
      console.log("change");
    },
  });
  instance.create();
  if (id) {
    axios.get(`/commodity/commodities/${id}`).then((res) => {
      console.log(res);
      state.commodityForm = {
        id: res.id,
        commodityname: res.commodityname,
        commodityintro: res.commodityintro,
        commoditymoney: res.commoditymoney,
        commodityType: res.commodityType,
        flavor: String(res.flavor),
        commodityimage: proxy.$filters.prefix(res.commodityimage),
        material: res.material,
        weight: res.weight,
      };
      fileList.value[0].url = res.commodityimage;
      fileList.value[0].name = res.commodityname + res.id + "image.png";
      if (instance) {
        // 初始化商品详情 html
        instance.txt.html(state.commodityForm.commodityintro);
      }
    });
  }

  // 获取分类
  axios.get("/commodity/gettype").then((res) => {
    options.value = res.map((e) => ({ value: e.id, label: e.commodityType }));
  });
});
onBeforeUnmount(() => {
  instance.destroy();
  instance = null;
});
// 提交
const submitAdd = () => {
  let fd = new FormData(); // 新建一个FormData()对象，这就相当于你新建了一个表单
  console.log(fd)
  fd.append("file", fileList.value[0].raw); // 将文件保存到formData对象中
  fd.append("commodityname", state.commodityForm.commodityname);
  fd.append("commodityintro", instance.txt.html());
  fd.append("commoditymoney", state.commodityForm.commoditymoney);
  fd.append("commodityType", state.commodityForm.commodityType);
  fd.append("flavor", state.commodityForm.flavor);
  fd.append("commodityimage", state.commodityForm.commodityimage);
  fd.append("material", state.commodityForm.material);
  fd.append("weight", state.commodityForm.weight);
  console.log(fd)
  goodRef.value.validate((vaild) => {
    let url = "/commodity/addcommodity";
    if (vaild) {
      if (id) {
        url = "/commodity/updateCommodity";
        fd.append("id", state.commodityForm.id);
      }
      axios
        .post(url, fd, {
          headers: {
            "Content-Type": "multipart/form-data",
            "token": state.token,
          },
        })
        .then(() => {
          ElMessage.success(id ? "修改成功" : "添加成功");
          router.push({ path: "/good" });
        });
    }
  });
};
// 分类
const handleChangeCate = (val) => {
  // console.log(val);
  const index = options.value.findIndex((e) => e.value === val);
  state.commodityForm.commodityType = options.value[index].label;
};

// 文件上传
const dialogImageUrl = ref("");
const dialogVisible = ref(false);
const disabled = ref(false);
const fileList = ref([
  {
    name: "food.jpeg",
    url: "https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100",
  },
]);
const handleRemove = (file) => {
  console.log(file);
  fileList.value = fileList.value.splice(1);
};

const handlePictureCardPreview = (file) => {
  dialogImageUrl.value = file.url;
  dialogVisible.value = true;
};

// 上传函数
const handleUpload = (param) => {
  console.log(param);
  const formData = new FormData();
  formData.append("file", param.file);
  // 设置formData
  axios
    .post("/picture/uploadImage", formData, {
      headers: {
        "Content-Type": "multipart/form-data",
        "token": state.token,
      },
    })
    .then((res) => {
      this.imageUrl.push(res.data.data);
      const obj = {};
      obj.uid = param.file.uid;
      obj.pictureId = res.data.data.pictureId;
      this.arr.push(obj);
    });
};
</script>

<style scoped>
.add {
  display: flex;
}
.add-container {
  flex: 1;
  height: 100%;
}
.avatar-uploader {
  width: 100px;
  height: 100px;
  color: #ddd;
  font-size: 30px;
}
.avatar-uploader-icon {
  display: block;
  width: 100%;
  height: 100%;
  border: 1px solid #e9e9e9;
  padding: 32px 17px;
}
</style>
