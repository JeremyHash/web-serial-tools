<template>
  <div style="text-align: center; margin-top: 200px">
    <v-btn @click="handleNameClick" outlined color="primary" dark>联系我(HashJeremy)
    </v-btn>
  </div>
</template>

<script>
export default {
  name: "AboutView",
  components: {},
  methods: {
    handleNameClick: function () {
      window.location.href = "https://gitee.com/HashJeremy";
    },
  },
};
</script>

<style scoped>
</style>