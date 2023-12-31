<template>
  <div class="article">
    <div class="articleWrap">
      <div class="articleItems">
        <ArticleItem v-for="(item, index) in articles" :articleData="item" :key="index" :isRowReverse="index % 2 == 0" />
      </div>
      <el-empty v-if="articles.length < 1" :image-size="200" :description="emptyDescription" />
    </div>
    <div class="getMore" v-if="articles.length > 0">
      <div v-show="loading" ref="lottieLoadingDom" id="lottie_loading" class="lottie_loading"></div>
      <button v-show="!loading" type="button" @click="getMoreArticle">
        MoreArticle
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import type { ArticleType } from "~/types/article";
import lottie from "lottie-web";
import animationData from "@/assets/lottie/animation_ll7j9mp7.json";
import { selectArticle } from "~/api/articleApi";

const lottieLoadingDom = ref(null);
const loading = ref(false);
const emptyDescription = "网站中暂无文章数据。";
onMounted(() => {
  lottie.loadAnimation({
    container: lottieLoadingDom.value!,
    renderer: "svg",
    loop: true,
    autoplay: true,
    animationData,
  });
});

const props = defineProps<{
  articleList: Array<ArticleType>;
  currentPage: number;
  count: number;
  offset: number;
  keyword?: string;
  field?: string;
}>();
const articles = ref<Array<ArticleType>>([])
watchEffect(() => {
  // 在这里处理 prop 变化，例如更新局部数据
  articles.value = props.articleList;
});

let currentPage = ref<number>(props.currentPage);

// 加载文章
async function getMoreArticle() {
  // 超过页数
  if (Math.ceil(props.count / props.offset) <= currentPage.value) {
    useMessage({ message: "后面没有内容了。", type: "warning" });
    return;
  }
  loading.value = true;
  currentPage.value++;
  let { data: article_res } = await selectArticle({
    page: currentPage.value,
    offset: props.offset,
    keyword: props.keyword,
    field: props.field,
  });
  loading.value = false;
  if (!article_res.value) {
    useMessage({ message: "服务器繁忙，请稍后再试！", type: "error" });
    return;
  }

  articles.value.push(...article_res.value.data.rows);
}
</script>

<style scoped>
.getMore {
  display: flex;
  text-align: center;
  justify-content: center;
}

.getMore .lottie_loading {
  width: 100px;
  /* height: 41px; */
}

.getMore button {
  outline: none;
  padding: 10px 20px;
  border-radius: 22px;
  border: 2px solid #444;
  background: none;
  font-size: 16px;
  transition: all 0.5s;
}

.getMore button:hover {
  border: 2px solid #2abce9;
  color: #2abce9;
  cursor: pointer;
}
</style>
