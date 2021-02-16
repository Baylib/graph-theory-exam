<template>
  <div>{{graphdata}}</div>
</template>

<script>
export default {
  name: 'GraphVue',
  mounted : function(){
    console.log('mounted GraphVue')
  },
  inject: ['graphdata']
}
</script>

<style>
</style>
