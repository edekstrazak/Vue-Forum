<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import { findById } from '../helpers'
import AppDate from '@/components/AppDate.vue'
import PostList from '@/components/PostList.vue'
import PostEditor from '@/components/PostEditor.vue'
import { useThreadStore } from '@/stores/ThreadStore'
import { usePostStore } from '@/stores/PostStore'
import { useUserStore } from '@/stores/UserStore'
const threadStore = useThreadStore()
const postStore = usePostStore()
const userStore = useUserStore()

const props = defineProps({
	id: {
		type: String,
		required: true,
	},
})
const threadLoaded = ref(false)
const thread = computed(() => {
  return threadStore.getThread(props.id)
})
const postsInThread = computed(() =>
	postStore.posts.filter(post => post.threadId === props.id)
)
const addPost = (event, thread) => {
	const post = { ...event.post }
	postStore.createPost(post, thread)
}
onMounted(async () => {
  await threadStore.fetchThread(props.id, {})
  await userStore.fetchUser(thread.value.userId, {})
  await userStore.fetchUsers(thread.value.contributors, {})
  await postStore.fetchPosts(thread.value.posts, thread)
  threadLoaded.value = true
})
</script>

<template>
  <div
    v-if="threadLoaded"
    class="col-large push-top"
  >
    <h1>
      {{ thread.title }}
      <router-link
        v-slot="{navigate}"
        :to="{name: 'ThreadEdit', id: props.id}"
        custom=""
      >
        <button
          class="btn-green btn-small"
          @click="navigate"
        >
          Edit Thread
        </button>
      </router-link>
    </h1>

    <p v-if="thread.author">
      By <a
        href="#"
        class="link-unstyled"
      >{{ thread.author?.name }}</a>, <AppDate
        v-if="thread.publishedAt"
        :timestamp="thread.publishedAt"
      />.
      <span
        style="float:right; margin-top: 2px;"
        class="hide-mobile text-faded text-small"
      ><span v-if="thread.repliesCount === 0 && thread.contributors">No replies</span>
        <span v-else>
          {{ thread.repliesCount }} 
          {{ thread.repliesCount > 1 ? 'replies' : 'reply' }} by 
          {{ thread.contributorsCount }} 
          {{ thread.contributorsCount > 1 ? 'contributors' : 'contributor' }}
        </span>
      </span>
    </p>

    <PostList :posts="postsInThread" />
    <PostEditor
      :thread-id="thread.id"
      @save-post="addPost($event, thread)"
    />
  </div>
</template>
