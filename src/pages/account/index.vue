<template>
  <nuxt-layout name="default" :user="userState">
    <Banner :height="'50vh'">账户</Banner>
    <nuxt-layout name="account" :active="0" :title="'我的信息'">
      <template #account-wrap>
        <div class="user-setting-wrap">
          <el-form :model="userInfo" ref="form" label-width="10.5rem">
            <el-form-item label="头像:" class="user-avatar">
              <label for="avatar" class="avatar">
                <el-avatar
                  class="changeAvatar"
                  :size="64"
                  :src="userInfo.avatar"
                ></el-avatar>
              </label>
              <input
                type="file"
                @change="showAvatarBlob"
                accept="image/*"
                name=""
                id="avatar"
                class="hide"
              />
            </el-form-item>
            <el-form-item label="昵称:">
              <el-input
                v-model="userInfo.name"
                placeholder="你的昵称"
              ></el-input>
            </el-form-item>
            <el-form-item label="我的签名:">
              <el-input
                v-model="userInfo.signature"
                :rows="5"
                resize="none"
                type="textarea"
                placeholder="设置您的签名- ( ゜- ゜)つロ"
              ></el-input>
            </el-form-item>
            <el-form-item label="性别:">
              <el-radio-group v-model="userInfo.sex" size="default">
                <el-radio-button
                  class="user-setting-radio"
                  label="男"
                ></el-radio-button>
                <el-radio-button
                  class="user-setting-radio"
                  label="女"
                ></el-radio-button>
                <el-radio-button
                  class="user-setting-radio"
                  label="保密"
                ></el-radio-button>
              </el-radio-group>
            </el-form-item>
            <el-form-item label="qq号:">
              <el-input v-model="userInfo.qq_no"></el-input>
            </el-form-item>
            <el-form-item label="GitHub主页:">
              <el-input v-model="userInfo.github_url"></el-input>
            </el-form-item>
            <el-form-item label="哔哩哔哩主页:">
              <el-input v-model="userInfo.bilibili_url"></el-input>
            </el-form-item>
            <el-form-item class="user-btn-wrap">
              <div class="padding-dom"></div>
              <div class="user-my-btn">
                <el-button
                  type="primary"
                  size="default"
                  :loading="uploadLoading"
                  @click="updateHandler"
                  >保存</el-button
                >
              </div>
            </el-form-item>
          </el-form>
        </div>
      </template>
    </nuxt-layout>
  </nuxt-layout>
</template>

<script setup lang="ts">
import type { UserStateType } from "@/types/user";
import { uploadImage } from "~/api/common";
import { updateUserData } from "~/api/userApi";

useHead({
  title: "我的信息",
});

const userInfo = reactive<UserStateType>(<UserStateType>{
  id: 0,
  avatar:
    "https://img.zcool.cn/community/01c8415b10ec7aa801212d57334fdc.png@1280w_1l_2o_100sh.png",
  name: "",
  signature: "",
  sex: "保密",
  bilibili_url: "",
  github_url: "",
  qq_no: "",
});

let userState = await useUserState();
const sexMap: Array<UserStateType["sex"]> = ["保密", "男", "女"];

for (let key in userInfo) {
  if (
    userState.value !== null &&
    userState.value !== undefined &&
    Object.keys(userState.value).includes(key)
  ) {
    (userInfo as any)[key] = (userState.value as any)[key];
  }
  if (key === "sex") {
    userInfo[key] = sexMap[userState.value[key] as unknown as number];
  }
}

// 头像文件
const avatarFile = ref<File>();

// 头像回显
function showAvatarBlob(event: Event) {
  let file = (event.target as any).files[0];
  avatarFile.value = file;
  let reader = new FileReader();
  reader.readAsDataURL(file);
  reader.onload = (e) => {
    if (typeof e.target?.result === "string") {
      userInfo.avatar = e.target.result;
    }
  };
}
const uploadLoading = ref(false);
// 更新用户信息
async function updateHandler() {
  uploadLoading.value = true;
  // 更新头像
  if (avatarFile.value) {
    const f = new FormData();
    f.append("file", avatarFile.value);
    const { data: uploadResponse } = await uploadImage(f);
    if (uploadResponse.value.code !== 1001) {
      throw "上传头像失败";
    }
    const avatarUrl = uploadResponse.value.data.row;
    if (avatarUrl && !avatarUrl.startsWith("https")) {
      userInfo.avatar = avatarUrl.replace(/^http:/, "https:");
    } else {
      userInfo.avatar = avatarUrl;
    }
  }

  const formData: object = {};
  for (let item in userInfo) {
    if (item === "sex") {
      (formData as any)[item] = sexMap.findIndex(
        (val) => val === (userInfo as any)[item]
      );
      continue;
    }
    if (userInfo[item] !== "") {
      (formData as any)[item] = (userInfo as any)[item];
    }
  }
  console.log(formData);

  let { data } = await updateUserData(formData);
  if (data.value?.code == 1001) {
    // 调用vuex action中 方法 查询用户信息，存入vux
    const userState = await useUserState();
    await keepUserData(userState);
    useMessage({
      message: "修改成功！",
      type: "success",
    });
    uploadLoading.value = false;
  } else {
    uploadLoading.value = false;
    throw "修改失败";
  }
}

//将base64转换为blob
function dataURLtoBlob(dataurl: string): Blob {
  var arr = dataurl.split(","),
    mime = arr[0]?.match(/:(.*?);/)[1],
    bstr = atob(arr[1]),
    n = bstr.length,
    u8arr = new Uint8Array(n);
  while (n--) {
    u8arr[n] = bstr.charCodeAt(n);
  }
  return new Blob([u8arr], { type: mime });
}
</script>

<style scoped>
:deep(.el-form-item__label) {
  font-family: "tsxmm";
  max-width: 100px;
}
.account-right .user-setting-wrap .user-setting-radio {
  margin: 0 5px;
}

.account-right
  .user-setting-wrap
  .user-setting-radio
  :deep(.el-radio-button__inner) {
  display: inline-block;
  border: 1px solid #dcdfe6;
  padding: 5px 10px;
  border-radius: var(--el-border-radius-base) !important;
  box-shadow: 0px 0 0 0
    var(--el-radio-button-checked-border-color, var(--el-color-primary));
}

.user-btn-wrap .padding-dom {
  height: 50px;
  border-bottom: 1px solid #ddd;
  margin-bottom: 50px;
}

.user-btn-wrap {
  text-align: center;
}

/* .user-btn-wrap :deep(.el-form-item__content) {
    margin-left: 0 !important;
} */

.user-avatar .avatar span {
  position: relative;
  cursor: pointer;
}

.user-avatar .avatar span:hover::after {
  opacity: 1;
}

.user-avatar .avatar span::after {
  position: absolute;
  top: 0;
  left: 0;
  content: "修改头像";
  width: 100%;
  height: 64px;
  font-size: 12px;
  background-color: rgba(1, 1, 1, 0.5);
  opacity: 0;
  transition: opacity 0.5s;
}

.user-avatar #avatar {
  position: absolute;
  visibility: hidden;
  opacity: 0;
  z-index: -1;
  pointer-events: none;
}
@media screen and (min-width: 768px) {
  .user-btn-wrap :deep(.el-form-item__content) {
    margin-left: 100px !important;
  }
}
</style>
