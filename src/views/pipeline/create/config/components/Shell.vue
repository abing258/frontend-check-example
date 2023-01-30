<template>
  <div class="mb-4">
    <div class="mb-2 flex justify-between">
      runs-on <a-switch
        v-model:checked="isChecked"
        @change="changeSwitch"
      />
    </div>
    <a-input
      v-if="isChecked"
      v-model:value="formData.runsOn"
      placeholder="请输入"
      allow-clear
      @change="setYamlValue('runs-on', formData.runsOn)"
    />
  </div>
  <div class="mb-4">
    <div class="mb-2">
      run<span class="text-[#FD0505] ml-1">*</span>
    </div>
    <a-textarea
      v-model:value="formData.run"
      placeholder="请输入"
      :auto-size="{ minRows: 3, maxRows: 6 }"
      allow-clear
      @change="setYamlValue('run', formData.run)"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, toRefs, reactive } from 'vue'
  
const props = defineProps({
  stage: String,
  index: Number,
  run: String,
  runsOn: String,
})
const { stage, index, run, runsOn } = toRefs(props)
const emit = defineEmits(['setYamlCode'])

const formData = reactive({
  run: '',
  runsOn: '', 
})
Object.assign(formData, { run: run?.value, runsOn: runsOn?.value })

const isChecked = ref(false)
if (runsOn?.value != '' && runsOn?.value != null) {
  isChecked.value = true
}

const changeSwitch = async () => {
  if (isChecked.value === true && runsOn?.value != '' && runsOn?.value != null) {
    setYamlValue('runs-on', formData.runsOn)
  } else {
    setYamlValue('runs-on', '')
  }
}

const setYamlValue =async (item: string, val: string) => {
  emit('setYamlCode', false, stage?.value, index?.value, item, val)
}
</script>