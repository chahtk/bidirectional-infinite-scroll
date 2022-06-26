<template>
  <div ref="triggerDiv">
    <slot>
      _
    </slot>
  </div>
</template>

<script>
export default {
  props: {
    rootId: {
      type: String,
      default: null
    }
  },
  data() {
    return {
      observer: null,
      option: {
        root: this.rootId && document.getElementById(this.rootId),
        threshold: 1,
      },
    };
  },
  methods: {
    handleIntersect: function (target) {
      if (target.isIntersecting) this.$emit(`triggerFadeIn`, this.OID);
    },
  },
  mounted() {
    this.observer = new IntersectionObserver((entries) => {
      this.handleIntersect(entries[0]);
    }, this.option);
    this.observer.observe(this.$refs.triggerDiv);
  },
};
</script>

<style scoped>
div {
  opacity: 0;
}
</style>