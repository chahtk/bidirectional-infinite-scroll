<template>
  <div>
    this is test page
  </div>
  <main id="data-container">
    <loading-spinner />
    <IOB @triggerFadeIn="onIOB('TOP')" />
    <div>
      <section 
        v-for="(post, index) in shownList.value"
        :key="post"
        :class="{highlight : post % 2 === 0}" 
        class="data-item" 
        :ref="el => moveScroll(el, index)">
          {{post}}
      </section>
    </div>
    <IOB @triggerFadeIn="onIOB('BOTTOM')" />
    <loading-spinner />
  </main>
</template>

<script>
import { reactive } from '@vue/reactivity'
import LoadingSpinner from './LoadingSpinner.vue';
import IOB from './IOB.vue';
import { onMounted, onUpdated } from '@vue/runtime-core';

  export default {
    components: { LoadingSpinner, IOB },
    setup() {
      const postList = Array.from(Array(95).keys()); // dummy data for API
      let isMoveToTop = false;
      const state = reactive({
        pageTop: 0,
        pageBottom: 0,
        size: 10,
      })
      const shownList = reactive({value:[]});
      const refMap = new Map();

      const moveScroll = (el, index) => {
        if (isMoveToTop) {
          refMap.set(index, el);
        }
      };

      const onIOB = (position) => {
        refMap.clear();
        if (position === 'TOP') {
          state.pageBottom -= 1;
          state.pageTop -= 1;
          const newPostList = getPostListAPI(state.pageTop);
          if (newPostList.length === 0) {
            state.pageBottom += 1;
            state.pageTop += 1;
            return;
          }
          isMoveToTop = true;
          shownList.value = [...newPostList, ...shownList.value].slice(0, 30);
          return;
        }
        
        // bottom
        state.pageBottom += 1;
        state.pageTop += 1;
        const newPostList = getPostListAPI(state.pageBottom);
        if (newPostList.length === 0) {
          state.pageBottom -= 1;
          state.pageTop -= 1;
          return;
        }
        isMoveToTop = false;
        shownList.value = [...shownList.value, ...newPostList].slice(-(20 + newPostList.length));
      }
      const getPostListAPI = (page, size = 10) => {
        // test API
        if (page * size > postList.length || page < 0) {
          return [];
        }
        return postList.slice(page * 10, page * 10 + size )
      };

      onMounted(() => {
        const newPostList = getPostListAPI(0, 30);
        state.pageBottom = 2;
        shownList.value = newPostList;
      })

      onUpdated(() => {
        if (!isMoveToTop) return;
        const previousFirstElement = refMap.get(10);
        if (previousFirstElement) {
          previousFirstElement.scrollIntoView();
        } else {
          console.error('no previous first element')
        }
      })
      
      return {
        state,
        onIOB,
        shownList,
        moveScroll,
      }
    },
  }
</script>

<style scoped>
#data-container {
  border: 1px solid red;
}
.data-item {
  height: 100px;
  text-align: center;
}
.highlight {
  background-color: beige;
}
</style>