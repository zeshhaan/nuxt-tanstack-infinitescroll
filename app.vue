<template>
  <span v-if="isPending">Loading...</span>
  <span v-else-if="isError">Error: {{ error.message }}</span>
  <div v-else-if="data">
    <span v-if="isFetching && !isFetchingNextPage">Fetching...</span>
    {{ data }}
    <button
      ref="target"
      @click="() => fetchNextPage()"
      :disabled="!hasNextPage || isFetchingNextPage"
    >
      <span v-if="isFetchingNextPage">Loading more...</span>
      <span v-else-if="hasNextPage">Load More</span>
      <span v-else>Nothing more to load</span>
    </button>
  </div>
</template>

<script setup lang="ts">
import { useInfiniteQuery } from '@tanstack/vue-query';

const fetchList = async ({ pageParam = 1 }) =>
  await $fetch('https://gutendex.com/books?mime_type=image&page=' + pageParam);

const {
  data,
  error,
  fetchNextPage,
  hasNextPage,
  isFetching,
  isFetchingNextPage,
  isPending,
  isError,
} = useInfiniteQuery({
  queryKey: ['books'],
  queryFn: fetchList,
  getNextPageParam: (lastPage, page, lastPageParam) =>
    new URL(lastPage.next).searchParams.get('page'),
});

const target = ref(null);
const targetIsVisible = ref(false);

const { stop } = useIntersectionObserver(
  target,
  ([{ isIntersecting }], observerElement) => {
    targetIsVisible.value = isIntersecting;
    fetchNextPage();
  }
);
</script>
