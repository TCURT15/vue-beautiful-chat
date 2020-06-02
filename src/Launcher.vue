<template>
  <div class="sc-wrapper" :class="{opened: isOpen}">
    <div v-if="showLauncher" class="sc-launcher sc-drag" :class="{opened: isOpen}" :style="{backgroundColor: colors.launcher.bg}">
      <div v-if="newMessagesCount > 0 && !isOpen" class="sc-new-messsages-count">
        {{newMessagesCount}}
      </div>
      <img v-if="isOpen" class="sc-closed-icon" :src="icons.close.img"  :alt="icons.close.name" />
      <img v-else class="sc-open-icon" :src="icons.open.img"  :alt="icons.open.name" />
    </div>
    <ChatWindow
     :showLauncher="showLauncher"
      :showCloseButton="showCloseButton"
      :showMinimizeButton="showMinimizeButton"
      :messageList="messageList"
      :onUserInputSubmit="onMessageWasSent"
      :participants="participants"
      :title="chatWindowTitle"
      :titleImageUrl="titleImageUrl"
      :isOpen="isOpen"
      :onClose="close"
      :onMinimize="minimize"
      :showEmoji="showEmoji"
      :hasMore="hasMore"
      :isLoading="isLoading"
      :icons="icons"
      :showFile="showFile"
      :showUserAvatar="showUserAvatar"
      :placeholder="placeholder"
      :showTypingIndicator="showTypingIndicator"
      :colors="colors"
      :alwaysScrollToBottom="alwaysScrollToBottom"
      :messageStyling="messageStyling"
      :disableUserListToggle="disableUserListToggle"
      @scrollToTop="$emit('scrollToTop')"
      @loadMore="$emit('loadMore')"
      @onType="$emit('onType')"
      @edit="$emit('edit', $event)"
      @remove="$emit('remove', $event)"
    >
      <template v-slot:header>
        <slot name="header" :icons="icons">
        </slot>
      </template>
      <template v-slot:user-avatar="scopedProps">
        <slot name="user-avatar" :user="scopedProps.user" :message="scopedProps.message">
        </slot>
      </template>
      <template v-slot:text-message-body="scopedProps">
        <slot name="text-message-body" :message="scopedProps.message" :messageText="scopedProps.messageText" :messageColors="scopedProps.messageColors" :me="scopedProps.me">
        </slot>
      </template>
      <template v-slot:system-message-body="scopedProps">
        <slot name="system-message-body" :message="scopedProps.message">
        </slot>
      </template>
      <template v-slot:text-message-toolbox="scopedProps">
        <slot name="text-message-toolbox" :message="scopedProps.message" :me="scopedProps.me">
        </slot>
      </template>
    </ChatWindow>
  </div>
</template>
<script>
import ChatWindow from './ChatWindow.vue'
import CloseIcon from './assets/close.svg'
import OpenIcon from './assets/logo-no-bg.svg';
var Draggabilly = require('draggabilly');
export default {
  props: {
    icons:{
      type: Object,
      required: false,
      default: function () {
        return {
            open: {
              img: OpenIcon,
              name: 'default',
            },
            close: {
              img: CloseIcon,
              name: 'default',
            },
        }
      }
    },
    showEmoji: {
      type: Boolean,
      default: false
    },
    isOpen: {
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
    open: {
      type: Function,
      required: true
    },
    close: {
      type: Function,
      required: true
    },
    minimize: {
      type: Function,
      required: true
    },
    showFile: {
      type: Boolean,
      default: false
    },
    showLauncher: {
      type: Boolean,
      default: true
    },
    showCloseButton: {
      type: Boolean,
      default: true
    },
    showMinimizeButton: {
      type: Boolean,
      default: true
    },
    participants: {
      type: Array,
      required: true
    },
    title: {
      type: String,
      default: () => ''
    },
    titleImageUrl: {
      type: String,
      default: () => ''
    },
    onMessageWasSent: {
      type: Function,
      required: true
    },
    messageList: {
      type: Array,
      default: () => []
    },
    newMessagesCount: {
      type: Number,
      default: () => 0
    },
    placeholder: {
      type: String,
      default: 'Write a message...'
    },
    showTypingIndicator: {
      type: String,
      default: () => ''
    },
    showUserAvatar: {
      type: Boolean,
      required: true
    },
    colors: {
      type: Object,
      required: false,
      validator: c =>
        'header' in c
        && 'bg' in c.header
        && 'text' in c.header
        && 'launcher' in c
        && 'bg' in c.launcher
        && 'messageList' in c
        && 'bg' in c.messageList
        && 'sentMessage' in c
        && 'bg' in c.sentMessage
        && 'text' in c.sentMessage
        && 'receivedMessage' in c
        && 'bg' in c.receivedMessage
        && 'text' in c.receivedMessage
        && 'userInput' in c
        && 'bg' in c.userInput
        && 'text' in c.userInput,
      default: function () {
        return {
          header: {
            bg: '#4e8cff',
            text: '#ffffff'
          },
          launcher: {
            bg: '#4e8cff'
          },
          messageList: {
            bg: '#ffffff'
          },
          sentMessage: {
            bg: '#4e8cff',
            text: '#ffffff'
          },
          receivedMessage: {
            bg: '#f4f7f9',
            text: '#ffffff'
          },
          userInput: {
            bg: '#f4f7f9',
            text: '#565867'
          }
        }
      }
    },
    alwaysScrollToBottom: {
      type: Boolean,
      default: () => false
    },
    messageStyling: {
      type: Boolean,
      default: () => false
    },
    disableUserListToggle: {
      type: Boolean,
      default: false
    },
  },
  methods: {
    toggleOpen() {
        this.open();
        if (this.isOpen) {
          this.$root.$emit('focusUserInput');
        }
    },
  },
  mounted() {
    var app = this;
    
    var items = document.querySelectorAll('.sc-wrapper');
    for ( var i=0, len = items.length; i < len; i++ ) {
      var item = items[i];
      var draggie = new Draggabilly(item, {
          handle: '.sc-drag',
          containment: '.main-content',
          grid: [15,15]

      });
      draggie.on('staticClick', function( event, pointer ) {
          if (event.target.classList.contains('sc-open-icon')) {
            app.toggleOpen();
          }
      });
    }
  },
  computed: {
    chatWindowTitle() {
      if (this.title !== '') {
        return this.title
      }

      if (this.participants.length === 0) {
        return 'You'
      } else if (this.participants.length > 1) {
        return 'You, ' + this.participants[0].name + ' & others'
      } else {
        return 'You & ' + this.participants[0].name
      }
    }
  },
  components: {
    ChatWindow
  }
}
</script>
<style scoped>
.sc-wrapper { 
    position: absolute; 
    right: 30px; 
    bottom: 185px; 
    width: fit-content; 
    height: fit-content; 
    z-index: 155;
}
.sc-launcher.opened { display: none; }
.sc-launcher {
  width: 50px;
  height: 50px;
  background-position: center;
  background-repeat: no-repeat;
  position: absolute;
  top: 0px;
  right: 0px;
  border-radius: 50%;
  box-shadow: none;
  transition: box-shadow 0.2s ease-in-out;
  cursor: pointer;
}
.sc-launcher .sc-open-icon,
.sc-launcher .sc-closed-icon {
  width: 50px;
  height: 50px;
  position: absolute;
  top: 0px;
  transition: opacity 100ms ease-in-out, transform 100ms ease-in-out;
}

.sc-launcher .sc-closed-icon {
  transition: opacity 100ms ease-in-out, transform 100ms ease-in-out;
  width: 50px;
  height: 50px;
}

.sc-launcher .sc-open-icon {
  box-sizing: border-box;
  opacity: 1;
}

.sc-launcher.opened .sc-open-icon {
  transform: rotate(-90deg);
  opacity: 1;
}

.sc-launcher.opened .sc-closed-icon {
  transform: rotate(-90deg);
  opacity: 1;
}

.sc-launcher.opened:before {
  box-shadow: 0px 0px 400px 250px rgba(148, 149, 150, 0.2);
}

.sc-launcher:hover {
  box-shadow: 0 0px 27px 1.5px rgba(0,0,0,0.2);
}

.sc-new-messsages-count {
  position: absolute;
  top: -3px;
  left: 41px;
  display: flex;
  justify-content: center;
  flex-direction: column;
  border-radius: 50%;
	width: 22px;
  height: 22px;
  background: #dc3545;
  color: white;
  text-align: center;
  margin: auto;
  font-size: 12px;
  font-weight: 500;
  z-index: 5;
}
</style>
