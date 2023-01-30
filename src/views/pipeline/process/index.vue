<template>
  <div class="flex justify-between">
    <Breadcrumb :current-name="$t('log.buildRecords') + queryJson.id" />
    <div>
      <a-button @click="terminationOfeExecution">{{ $t("log.terminationOfeExecution") }}</a-button>
    </div>
  </div>

  <div class="process bg-[#ffffff] ">
    <div class="bg-[#121211] rounded-t-[12px] h-[92px] p-[24px] text-center">
      <a-row>
        <a-col :span="6">
          <div class="process-detail-item">
            <div class="process-detail-title">Trigger Info</div>
            <div class="process-detail-info">{{ jobData.triggerMode }}</div>
          </div>
        </a-col>
        <a-col :span="6">
          <div class="process-detail-item">
            <div class="process-detail-title">{{ $t("log.status") }}</div>
            <div class="process-detail-info">
              {{ $t(`log.${StatusEnum[jobData.status]}`) }}
            </div>
          </div>
        </a-col>
        <a-col :span="6">
          <div class="process-detail-item">
            <div class="process-detail-title">
              {{ $t("log.executionTime") }}
            </div>
            <div class="process-detail-info">
              {{ fromNowexecutionTime(jobData.startTime, "noThing") }}
            </div>
          </div>
        </a-col>
        <a-col :span="6">
          <div>
            <div class="process-detail-title">
              {{ $t("log.totalDuration") }}
            </div>
            <div class="process-detail-info">
              {{
                state.running
                  ? "-"
                  : formatDurationTime(jobData.duration, "noThing")
              }}
            </div>
          </div>
        </a-col>
      </a-row>
    </div>
    <div class="p-[24px] rounded-b-[12px]">
      <div class="process-content">
        <div class="flex justify-between">
          <span class="process-content-title">{{
            $t("log.executionProcess")
          }}</span>
          <span
            class="text-[14px] text-[#28C57C] cursor-pointer"
            @click="checkAllLogs"
          >{{ $t("log.viewAllLogs")
          }}</span>
        </div>
        <div
          ref="wrapper"
          class="process-scroll-box wrapper"
        >
          <div class="process-scroll content">
            <div class="inline-block execution_process_item">
              <div class="inline-block border border-solid border-[#EFEFEF] p-[12px] rounded-[5px]">
                <img
                  src="@/assets/icons/Frame.svg"
                  class="w-[28px] mr-[24px] align-middle"
                />
                <span class="align-middle">
                  <span class="text-[16px] text-[#121211] font-semibold mr-[24px]">{{ $t("log.start") }}</span>
                </span>
              </div>
              <img
                src="@/assets/images/arrow-green.jpg"
                class="w-[28px] space-mark ml-[20px] mr-[20px]"
              />
            </div>
            <div
              v-for="item in jobData.stages"
              :key="item.name"
              class="inline-block execution_process_item"
            >
              <div
                class="inline-block border border-solid border-[#EFEFEF] p-[12px] rounded-[5px]"
                :class="item.status === 0 ? '' : 'cursorP'"
                @click="checkProcess(item, $event)"
              >
                <img
                  v-if="item.status !== 1"
                  :src="getImageUrl(item.status)"
                  class="w-[28px] mr-[24px] align-middle"
                />
                <img
                  v-else
                  src="@/assets/images/run.gif"
                  class="w-[28px] mr-[24px] align-middle"
                />
                <span class="align-middle">
                  <span class="text-[16px] text-[#121211] font-semibold mr-[24px]">{{ item.name }}</span>
                  <span
                    v-if="item.status !== 0"
                    class="text-[16px] text-[#7B7D7B]"
                  >{{ formatDurationTime(item.duration,
                                         "noThing")
                  }}</span>
                </span>
              </div>
              <img
                src="@/assets/images/arrow-green.jpg"
                class="w-[28px] space-mark ml-[20px] mr-[20px]"
              />
            </div>
          </div>
          <div
            ref="horizontal"
            class="custom-horizontal-scrollbar"
          >
            <div class="custom-horizontal-indicator"></div>
          </div>
        </div>
      </div>
      <div
        v-if="jobData.actionResult.artifactorys.length > 0"
        class="process-content"
      >
        <div class="process-content-title">{{ $t("log.artifats") }}</div>
        <div class="text-[#7B7D7B]">
          <div
            v-for="it in jobData.actionResult.artifactorys"
            :key="it.id"
            class="text-[#1890ff] cursor-pointer"
            @click="openNewUrl(it.url)"
          >
            {{ it.url }}
          </div>
        </div>
      </div>
      <div
        v-if="jobData.actionResult.reports.length > 0"
        class="process-content"
      >
        <div class="process-content-title">{{ $t("log.report") }}</div>
        <div class="text-[#7B7D7B]">
          <div
            v-for="it in jobData.actionResult.reports"
            :key="it.id"
            class="text-[#1890ff] cursor-pointer"
            @click="openNewUrl(it.url)"
          >
            {{ it.url }}
          </div>
        </div>
      </div>
    </div>
  </div>

  <ProcessModal
    ref="processModalRef"
    :stages-data="stagesData"
  />
</template>
<script lang="ts" setup>
import { ref, onMounted, reactive, onUnmounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { apiGetJobStageLogs, apiCheekArtifactorys } from '@/apis/jobs'
import { apiGetPipelineDetail, apiStopPipeline } from '@/apis/pipeline'
import { fromNowexecutionTime, formatDurationTime } from '@/utils/time/dateUtils.js'
import BScroll from '@better-scroll/core'
import Scrollbar from '@better-scroll/scroll-bar'
import ProcessModal from './components/ProcessModal.vue'
import Breadcrumb from '@/views/components/Breadcrumb.vue'
import { message } from 'ant-design-vue'

BScroll.use(Scrollbar)
const route = useRoute()
const router = useRouter()
const processModalRef = ref()
const horizontal = ref()
const state = reactive({
  detailTimer: null,
  stagesTimer: null,
  running: true,
})

const stagesData = reactive({
  title: '',
  content: [],
})

const wrapper = ref()
const jobData = reactive({
  id: undefined,
  name: '',
  triggerMode: '',
  stages: [],
  actionResult: {
    artifactorys: [],
    reports: [],
  },
})

const enum StatusEnum {
  'nonExecution',
  'running',
  'failed',
  'passed',
  'stop',
}

const queryJson = reactive({
  name: router.currentRoute.value.params.name,
  id: router.currentRoute.value.params.id,
})

const getPipelineDetail = async () => {
  const { data } = await apiGetPipelineDetail(queryJson)
  Object.assign(jobData, data)
  state.running = data.stages.some((val: any) => val.status === 1)
  if (state.running) {
    state.detailTimer = setTimeout(() => {
      getPipelineDetail()
    }, 3000)
  } else {
    clearTimeout(state.detailTimer)
  }
}
const getImageUrl = (status: any) => {
  return new URL(`../../../assets/icons/Status${ status }.svg`, import.meta.url)
  .href
}

const checkAllLogs = () => {
  window.open(`#/allLogs/${ queryJson.name }/${ queryJson.id }`)
}

const checkProcess = async (item: any, e: Event) => {
  if (item.status === 0) {
    e.stopPropagation()
  } else {
    stagesData.title = item.name
    processModalRef.value.showVisible()
    stagesData.content = []
    await getStageLogsData(item)
  }
}

const getStageLogsData = async (item: any, start = 0) => {
  const query = Object.assign(queryJson, {
    stagename: item.name,
    start: start,
  })
  const { data } = await apiGetJobStageLogs(query)
  const t = data?.content?.split('\r')
  if (data.content) {
    t.forEach((item: any) => {
      stagesData.content.push(item)
    })
  }
  // if (data.end) {
  //   stagesData.content = data?.content?.split("\r");
  // } else {
  //   let t = data?.content?.split("\r");
  //   stagesData.content.push(t);
  // }

  if (!data.end && processModalRef.value.visible) {
    state.stagesTimer = setTimeout(() => {
      getStageLogsData(item, data.lastLine)
    }, 3000)
  } else {
    clearTimeout(state.stagesTimer)
  }
}

const terminationOfeExecution = async () => {
  if (state.running) {
    const { data } = await apiStopPipeline(queryJson)
    getPipelineDetail()
  } else {
    message.info('该构建已结束')
  }
}

const openNewUrl = async (url: string) => {
  const data = await apiCheekArtifactorys(queryJson)
}

onMounted(async () => {
  await getPipelineDetail()
  route.meta.breadcrumbName = router.currentRoute.value.params.id
  const scroll = new BScroll(wrapper.value, {
    startX: 0,
    scrollX: true,
    scrollY: false,
    probeType: 1,
    scrollbar: {
      fade: false,
      interactive: true,
    },
  })
})

onUnmounted(() => {
  clearTimeout(state.detailTimer)
  clearTimeout(state.stagesTimer)
})
</script>
<style lang="less" scoped>
:deep(.ant-btn) {
  color: #ffffff;
  background-color: #FF842C;
  border-color: #FF842C;
}

:deep(.ant-btn:hover) {
  color: #ffffff;
  background-color: #FF842C;
  border-color: #FF842C;
}

:deep(.ant-btn:focus) {
  color: #ffffff;
  background-color: #FF842C;
  border-color: #FF842C;
}


.process {
  width: 100%;
  font-size: 14px;
  border-radius: 12px;

  .process-detail-item {
    position: relative;

    &::before {
      content: "";
      position: absolute;
      top: 0px;
      right: -2px;
      width: 1px;
      height: 44px;
      border: 1px dashed #3f4641;
    }
  }

  .process-detail-title {
    color: #ffffff;
    font-weight: 600;
  }

  .process-detail-info {
    color: #bcbebc;
  }

  .process-scroll-box {
    white-space: nowrap;
    overflow: hidden;

    .process-scroll {
      display: inline-block;
      margin-bottom: 24px;

      .cursorP {
        cursor: pointer;
      }
    }
  }

  .process-content {
    padding: 24px;
    border: 1px solid #efefef;
    border-radius: 12px;
    margin-bottom: 24px;

    .process-content-title {
      font-size: 20px;
      font-weight: 600;
      color: #121211;
      margin-bottom: 12px;
    }

    .process-content-title:first {
      margin-bottom: 24px;
    }
  }

  .process-content:last-child {
    margin-bottom: 0px;
  }

  .execution_process_item:last-child {
    .space-mark {
      display: none;
    }
  }
}

:deep(.bscroll-horizontal-scrollbar) {
  z-index: 1 !important;
}

:deep(.bscroll-indicator) {
  background: #efefef !important;
}
</style>
