<template>
  <div id="app" class="">
    <div class="nav nav-tabs" id="nav-tab" role="tablist">
      <a
        v-for="tabcount in tabcount"
        :key="tabcount"
        class="nav-link"
        :id="'nav-' + tabcount + '-tab'"
        data-toggle="tab"
        :href="'#nav-' + tabcount"
        role="tab"
        aria-controls="nav-profile"
        aria-selected="false"
      >
        Tab: {{ tabcount }}
      </a>
    </div>
    <div class="tab-content" id="nav-tabContent">
      <div
        v-for="tabcount in tabcount"
        :key="tabcount"
        class="tab-pane fade"
        :id="'nav-' + tabcount"
        role="tabpanel"
        :aria-labelledby="'nav-' + tabcount + '-tab'"
      >
        <my-bot :tab-count="tabcount" />
      </div>
    </div>
    <button
      :disabled="!checks"
      type="button"
      class="btn-lg text-white btn-success ml-2 addButton"
      @click="addBot"
    >
      ADD BOT
    </button>
  </div>
</template>

<script>
import MyBot from "./components/myBot";
export default {
  name: "App",
  components: { MyBot },
  data: () => ({
    tabcount: 1,
    clicked: false,
  }),

  mounted() {
    // this.$eventHub.$on('delete-bot',()=>{
    //   console.log(this.$el.querySelectorAll('#myBot').length)
    //   this.tabcount = this.$el.querySelectorAll('#myBot').length - 1
    // })

    this.$el.querySelector("#nav-1-tab").classList.add("active");
    this.$el.querySelector("#nav-1").classList.add("show");
    this.$el.querySelector("#nav-1").classList.add("active");
  },
  methods: {
    addBot() {
      this.checks() ? (this.tabcount += 1) : null;
    },
    checks() {
      return this.$el.querySelectorAll("#myBot").length < 8;
    },
  },
};
</script>

<style lang="scss" scoped>
#app {
  padding: 30px;
  margin: auto;
}

.row {
  width: auto !important;
}

.addButton {
  position: fixed;
  bottom: 40px;
  right: 40px;
}
a {
  color: white;
}
.nav-link.active {
  background: black;
  color: white;
}
</style>
