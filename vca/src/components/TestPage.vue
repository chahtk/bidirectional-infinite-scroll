<template>
  <div>
    this is test page
  </div>
  <main id="data-container">
    <loading-spinner />
    <IOB @triggerFadeIn="onIOB('TOP')" />
    <div>
      <section 
        v-for="post in shownList.value"
        :key="post"
        :class="{highlight : post % 2 === 0}" 
        class="data-item" 
        :ref="el => refList.push(el)">
          {{post}}
      </section>
    </div>
    <IOB @triggerFadeIn="onIOB('BOTTOM')" />
    <loading-spinner />
  </main>
</template>

<script>
import { watch } from 'vue';
import { reactive, ref } from '@vue/reactivity'
import LoadingSpinner from './LoadingSpinner.vue';
import IOB from './IOB.vue';
import { onMounted } from '@vue/runtime-core';

  export default {
    components: { LoadingSpinner, IOB },
    setup() {
      const postList = Array.from(Array(95).keys()); // dummy data for API
      const refList = ref([]);
      const state = reactive({
        pageTop: 0,
        pageBottom: 0,
        size: 10,
      })
      const shownList = reactive({value:[]});
      let isMoveToTop = false;

      const onIOB = (position) => {
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

      watch(shownList, () => {
        if (isMoveToTop) {
          refList.value.map((item) => console.log(item)) // 순서 보장 안되는중
          // refList.value[0].scrollIntoView(true);
        }
      });

      onMounted(() => {
        const newPostList = getPostListAPI(0, 30);
        state.pageBottom = 2;
        shownList.value = newPostList;
      })
      
      return {
        state,
        onIOB,
        shownList,
        refList,
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