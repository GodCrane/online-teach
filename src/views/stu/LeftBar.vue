<template>
  <div class="stuLeftMainLeftBody">
    <div class="stuLeftBarSelfInfo">
      <div
        style="
          font-size: 28px;
          text-align: center;
          font-family: 阿里妈妈数黑体;
          color: white;
          margin-top: 27px;
        "
      >
        个人信息
      </div>
      <div class="StuLeftBarAvatar">
        <el-avatar
          :size="180"
          :src="accountInfoForm.avatar || defaultAvatarUrl"
          @click="AvatarVisible = true"
        />
      </div>
      <el-card class="selfTxtInfo">
        <p>姓名: {{ stuInfoForm.stuName }}</p>
        <p>性别: {{ stuInfoForm.sex }}</p>
        <p>学制: {{ stuInfoForm.type }}</p>
        <p>学院: {{ stuInfoForm.dept }}</p>
        <p>专业: {{ stuInfoForm.major }}</p>
        <p>班级: {{ stuInfoForm.adminClass }}</p>
        <p>入学年份: {{ stuInfoForm.grade }}</p>
      </el-card>
    </div>
    <el-dialog v-model="AvatarVisible" title="头像上传" width="400">
      <div style="margin-left: 80px">
        <p style="font-family: '阿里妈妈数黑体'; font-size: 20px">在这里上传头像哦🥳</p>
        <el-upload
          class="avatar-uploader"
          action="/api/upload"
          :show-file-list="false"
          :on-success="handleAvatarSuccess"
          :before-upload="beforeAvatarUpload"
        >
          <img v-if="imageUrl" :src="imageUrl" class="avatar" />
          <el-icon v-else class="avatar-uploader-icon"><Plus /></el-icon>
        </el-upload>
      </div>
      <template #footer>
        <div class="dialog-footer">
          <el-button @click="AvatarVisible = false">取消</el-button>
          <el-button type="primary" @click="uploadAvatar()"> 确认上传 </el-button>
        </div>
      </template>
    </el-dialog>
  </div>
</template>

<script lang="ts" setup>
import { ref } from "vue";
import { reactive, onMounted } from "vue";
import { userInfoService } from "@/api/user";
import { getStuInfo } from "@/api/student";
import { useUserInfoStore } from "@/stores/userinfo";
import { updateAvatarService } from "@/api/user";
const AvatarVisible = ref(false);
const userStore = useUserInfoStore();
const stuInfoForm = reactive({
  stuName: "",
  sex: "",
  type: "",
  dept: "",
  major: "",
  grade: "",
  adminClass: "",
});
const accountInfoForm = reactive({
  accId: null,
  username: "",
  password: "",
  avatar: "",
  type: null,
});
let isFetched = false;
const defaultAvatarUrl = ref(
  "https://cube.elemecdn.com/0/88/03b0d39583f48206768a7534e55bcpng.png"
);
const getSelfInfo = async () => {
  if (isFetched) return;
  isFetched = true;

  try {
    const res = await userInfoService();
    Object.assign(accountInfoForm, res.data);
    const res1 = await getStuInfo(res.data.username);

    // 使用 Object.assign 更新 reactive 对象
    Object.assign(stuInfoForm, res1.data);
    const combinedInfo = Object.assign({}, accountInfoForm, stuInfoForm);
    userStore.setInfo(combinedInfo);
  } catch (error) {
    console.error("Error fetching student info:", error);
  }
};

onMounted(() => {
  getSelfInfo();
});
import { Plus } from "@element-plus/icons-vue";

import { ElMain, ElMessage, type UploadProps } from "element-plus";

const imageUrl = ref("");

const handleAvatarSuccess: UploadProps["onSuccess"] =  (response, uploadFile) => {
   imageUrl.value = response.data;

};
const uploadAvatar = async () => {
console.log(imageUrl.value)
  let result = await updateAvatarService(imageUrl.value);
  ElMessage.success("头像上传成功");
  AvatarVisible.value = false;
  location.reload();
}
const beforeAvatarUpload: UploadProps["beforeUpload"] = (rawFile) => {
  if (rawFile.type !== "image/jpeg") {
    ElMessage.error("必须是JPG格式!");
    return false;
  } else if (rawFile.size / 1024 / 1024 > 2) {
    ElMessage.error("大小不得超过 2MB!");
    return false;
  }
  return true;
};
</script>

<style>
.stuLeftMainLeftBody {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 330px;
  height: 100%;
  background-color: #f2f5f9;
}
.stuLeftBarSelfInfo {
  margin-top: 30px;
  position: relative;
  width: 88%;
  height: 690px;
  border-radius: 10px;
  background-color: #16243d;
}
.StuLeftBarAvatar {
  cursor: pointer;

  position: absolute;
  margin: 20px 0px;
  top: 24%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 65%;
  display: flex;
  justify-content: center;
  height: 180px;
}
.selfTxtInfo {
  font-family: "华文中宋";
  border-radius: 10px;
  position: absolute;
  top: 65%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 90%;
  margin-top: 30px;
  height: 320px;
  background-color: white;
}
.avatar-img {
  width: 75%;
  height: 100%;
}
.avatar-uploader .avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>

<style>
.avatar-uploader .el-upload {
  border: 1px dashed var(--el-border-color);
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  transition: var(--el-transition-duration-fast);
}

.avatar-uploader .el-upload:hover {
  border-color: var(--el-color-primary);
}

.el-icon.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  text-align: center;
}
</style>
