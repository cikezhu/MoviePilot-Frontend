<script setup lang="ts">
import MediaCardListView from './MediaCardListView.vue'
import api from '@/api'
import personIcon from '@images/misc/person.png'
import type { TmdbPerson } from '@/api/types'
import NoDataFound from '@/components/NoDataFound.vue'

// 输入参数
const personProps = defineProps({
  personid: String,
  type: String,
})

// 媒体详情
const personDetail = ref<TmdbPerson>({} as TmdbPerson)

// 是否已加载完成
const isRefreshed = ref(false)

// 人物图片是否加载
const isImageLoaded = ref(false)

// 调用API查询详情
async function getPersonDetail() {
  if (personProps.personid) {
    personDetail.value = await api.get(`tmdb/person/${personProps.personid}`)
    isRefreshed.value = true
  }
}

// 人物图片地址
function getPersonImage() {
  if (!personDetail.value?.profile_path)
    return personIcon
  return `https://image.tmdb.org/t/p/w600_and_h900_bestv2${personDetail.value?.profile_path}`
}

// 将别名数组拆分为、分隔的字符串
function getAlsoKnownAs() {
  if (!personDetail.value?.also_known_as)
    return ''
  return personDetail.value.also_known_as.join('、')
}

onBeforeMount(() => {
  getPersonDetail()
})
</script>

<template>
  <LoadingBanner
    v-if="!isRefreshed"
    class="mt-12"
  />
  <div v-if="personDetail.id" class="max-w-8xl mx-auto px-4">
    <div class="relative z-10 mt-4 mb-8 flex flex-col items-center lg:flex-row ">
      <VAvatar
        size="200"
        :class="{
          'ring-1 ring-gray-700': isImageLoaded,
        }"
      >
        <VImg
          v-img
          :src="getPersonImage()"
          cover
          @load="isImageLoaded = true"
        />
      </VAvatar>
      <div class="text-start ms-3 md:text-center">
        <h1 class="text-3xl lg:text-4xl">
          {{ personDetail.name }}
        </h1>
        <div class="mt-1 mb-2 space-y-1 text-xs sm:text-sm lg:text-base">
          <div>
            <span v-if="personDetail.birthday">{{ personDetail.birthday }}</span>
            <span v-if="personDetail.place_of_birth"> | </span>
            <span v-if="personDetail.place_of_birth">{{ personDetail.place_of_birth }}</span>
          </div>
          <div v-if="personDetail.also_known_as">
            别名：{{ getAlsoKnownAs() }}
          </div>
        </div>
      </div>
    </div>
    <div class="relative text-left">
      <div class="group outline-none ring-0" role="button" tabindex="-1">
        <p class="pt-2 text-sm lg:text-base" style="overflow-wrap: break-word;">
          {{ personDetail.biography }}
        </p>
      </div>
    </div>
    <div>
      <div class="slider-header">
        <RouterLink :to="`/browse/tmdb/person/credits/${personDetail.id}?title=参演作品`" class="slider-title">
          <span>参演作品</span>
          <VIcon icon="mdi-arrow-right-circle-outline" class="ms-1" />
        </RouterLink>
      </div>
      <MediaCardListView :apipath="`tmdb/person/credits/${personDetail.id}`" />
    </div>
  </div>
  <NoDataFound
    v-if="!personDetail.id && isRefreshed"
    error-code="500"
    error-title="出错啦！"
    error-description="无法获取到媒体信息，请检查网络连接。"
  />
</template>
