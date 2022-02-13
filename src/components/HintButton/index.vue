<template>
    <a :title="title" style="margin:5px 10px">
      <el-button v-bind="$attrs" v-on="$listeners"></el-button>
    </a>
</template>

<script>
export default {
  name: 'HintButton',
  props:["title"],
  mounted(){
    //   console.log("attrs",this.$attrs);
    //   console.log("listeners",this.$listeners);
  }
}
</script>

<style scoped>
</style>

