<template>
  <div class="mb-4">
    <div class="mb-2">
      name<span class="text-[#FD0505] ml-1">*</span>
    </div>
    <a-input
      v-model:value="formData.name"
      placeholder="请输入"
      allow-clear
      @change="setYamlValue('name', formData.name)"
    />
  </div>
  <div class="mb-4">
    <div class="mb-2">
      path<span class="text-[#FD0505] ml-1">*</span>
    </div>
    <a-textarea
      v-model:value="formData.path"
      placeholder="请输入"
      :auto-size="{ minRows: 3, maxRows: 6 }"
      allow-clear
      @change="setYamlValue('path', formData.path)"
    />
  </div>
</template>

<script setup lang="ts">
import { toRefs,reactive } from 'vue'
  
const props = defineProps({
  stage: String,
  index: Number,
  name: String,
  path: String,
})
const { stage, index, name, path } = toRefs(props)
const emit = defineEmits(['setYamlCode'])

const formData = reactive({
  name: '',
  path: '', 
})
Object.assign(formData, { name: name?.value, path: path?.value })

const setYamlValue =async (item: string, val: string) => {
  emit('setYamlCode', true, stage?.value, index?.value, item, val)
}
</script>