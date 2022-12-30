<template>
  <div class="editMember">
    <el-card>
      <Tinymce v-model="tiny.value" :height="200"/>
    </el-card>
  </div>
</template>

<script>
import Tinymce from '@/views/components/Tinymce/index.vue'

export default {
  components: { Tinymce },
  data() {
    return {
      tiny: {
        value: undefined
      }
    }
  },
  methods: {
    // 保存富文本内容-保存字符串
    add() {
      // eslint-disable-next-line no-unused-vars
      const params = `this.tiny?.value`
    },
    edit() {
      // 接口回显保存html
      // this.tiny.value = res.data|| ''
    }
  }
}
</script>
