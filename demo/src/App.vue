<template>
  <div id="app">
      <label>开始：</label>
      <v-datetimepicker
        :value="start_at"
        :disabled-days-of-Week="disabled"
        :format="format"
        :clear-button="clear"
        :placeholder="placeholder_start"
        @on-data-change="onStartDataChange">
      </v-datetimepicker>

      <label>结束：</label>
      <v-datetimepicker
        :value="end_at"
        :disabled-days-of-Week="disabled"
        :format="format"
        :clear-button="clear"
        :placeholder="placeholder_end"
        @on-data-change="onEndDataChange">
      </v-datetimepicker>
  </div>
</template>

<script>
import vDatetimepicker from './components/Datetimepicker'

export default {
  name: 'app',
  components: {
    vDatetimepicker
  },
  data(){
    return {
      start_at:'',
      end_at:'',
      disabled:true,
      clear:true,
      format:'yyyy/mm/dd',
      placeholder_end:'结束时间',
      placeholder_start:'开始时间'
    }
  },
  methods:{
    onStartDataChange(val){
      this.start_at = val
    },
    onEndDataChange(val){
      this.end_at = val
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
