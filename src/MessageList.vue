<template>
  <div class="sc-message-list" ref="scrollList" :style="{backgroundColor: colors.messageList.bg}" @scroll="handleScroll">
	<div class="text-center" style="position: absolute; top: 0px; left: 0; width: 100%;">
	  <div v-if="isLoading">
		<md-progress-bar md-mode="indeterminate"></md-progress-bar>
	  </div>
	  <button v-if="hasMore && !isLoading" @click="loadMoreMessages" class="btn-plain mt-1 text-muted">Load More</button>
	</div>
	<Message v-for="(message, idx) in messages" :message="message" :user="profile(message.author)" :showUserAvatar="showUserAvatar" :key="idx" :colors="colors" :messageStyling="messageStyling" @remove="$emit('remove', message)">
	  <template v-slot:user-avatar="scopedProps">
		<slot name="user-avatar" :user="scopedProps.user" :message="scopedProps.message">
		</slot>
	  </template>
	  <template v-slot:text-message-body="scopedProps">
		<slot name="text-message-body" :message="scopedProps.message" :messageText="scopedProps.messageText" :messageColors="scopedProps.messageColors" :me="scopedProps.me">
		</slot>
	  </template>
	  <template v-slot:text-message-toolbox="scopedProps">
		<slot name="text-message-toolbox" :message="scopedProps.message" :me="scopedProps.me">
		</slot>
	  </template>
	</Message>
	<Message v-show="showTypingIndicator !== ''" :showUserAvatar="showUserAvatar" :message="{author: showTypingIndicator, type: 'typing'}" :user="{}" :colors="colors" :messageStyling="messageStyling" />
  </div>
</template>
<script>
import Message from './Message.vue'
import chatIcon from './assets/chat-icon.svg'
export default {
  data() {
	return {
	  isLoadingMore: false,
	  oldScroll: '',
	  curScrollPos: '',
	}
  },
  components: {
	Message
  },
  props: {
	participants: {
	  type: Array,
	  required: true
	},
	messages: {
	  type: Array,
	  required: true
	},
	showTypingIndicator: {
	  type: String,
	  required: true
	},
	colors: {
	  type: Object,
	  required: true
	},
	alwaysScrollToBottom: {
	  type: Boolean,
	  required: true
	},
	messageStyling: {
	  type: Boolean,
	  required: true
	},
	hasMore: {
	  type: Boolean,
	  required: true
	},
	isLoading: {
	  type: Boolean,
	  required: true  
	},
	showUserAvatar: {
		type: Boolean,
		required: true
	}
  },
  methods: {
	_scrollDown () {
	  this.$refs.scrollList.scrollTop = this.$refs.scrollList.scrollHeight
	},
	handleScroll (e) {
		if (e.target.scrollTop === 0) {
			this.$emit('scrollToTop')
		}
	},
	shouldScrollToBottom() {
	  return this.alwaysScrollToBottom || (this.$refs.scrollList.scrollTop > this.$refs.scrollList.scrollHeight - 600)
	},
	profile(author) {
	  const profile = this.participants.find(profile => profile.id === author)
	  // A profile may not be found for system messages or messages by 'me'
	  return profile || {imageUrl: '', name: ''}
	},
	loadMoreMessages() {
		this.isLoadingMore = true;
		var content = this.$refs.scrollList;
  		this.curScrollPos = content.scrollTop; 
  		this.oldScroll = content.scrollHeight - content.clientHeight;
	 	this.$emit('loadMore');
	},
  },
  computed: {
	defaultChatIcon() {
	  return chatIcon
	}
  },
  watch: {
	  isLoading(afterValue, beforeValue) {
	  	if(!afterValue && this.isLoadingMore) {
	  		var app = this;
			setTimeout(function(){ 
				var content = app.$refs.scrollList;
				var newScroll = content.scrollHeight - content.clientHeight;
  				content.scrollTop = app.curScrollPos + (newScroll - app.oldScroll);
	  			app.isLoadingMore = false;
	  		}, 0);
	  	}
	  }
  },
  mounted () {
	this.$nextTick(this._scrollDown())
  },
  updated () {
	if (this.shouldScrollToBottom() && !this.isLoadingMore)
	  this.$nextTick(this._scrollDown())
  }
}
</script>

<style scoped>
.sc-message-list {
  height: 80%;
  overflow-y: auto;
  background-size: 100%;
  padding: 40px 0px;
  position: relative;
  min-height: 300px;
}
/deep/ .md-progress-bar.md-indeterminate .md-progress-bar-fill:after {
	background-color: #065494;
}
</style>
