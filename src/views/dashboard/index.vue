<template>
  <div>
    <Card></Card>
    <Sale></Sale>
    <Observe></Observe>
  </div>
</template>

<script>
import Card from './Card'
import Sale from './Sale'
import Observe from './Observe'
export default {
  name: 'Home',
  components: {
    Card,
    Sale,
    Observe
  },
  data() {
    return {}
  },
  mounted() {
    this.$store.dispatch('getData')
  }
}
</script>

<style scoped lang='less'>

</style>
