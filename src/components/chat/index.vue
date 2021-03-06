<template lang="pug">
  .chat
    transition(name="chat-bg")
      .chat-bg(
        v-if="getChatTrigger"
        @click.prevent="chatOver"
      )
    transition(name="chat-tab")
      .chat-module(v-if="getChatTrigger")
        .chat-module__header
          span.chat-module__online Online: {{ getOnline }}
          h2.chat-module__capt dice chat
          .chat-module__close(@click.prevent="chatOver")
            button.chat-module__close-but
        .chat-module__messages(ref="messages")
          transition(name="username-anim")
            .chat-module__username(v-if="usernameTrigger")
              span.chat-module__username-error(v-if="isError") Name is longer than 10 characters
              input.chat-module__username-inp(
                ref="username"
                v-model="name"
                placeholder="input your name"
              )
              button.chat-module__username-send(
                @click.prevent="addUser"
              ) Go chat
          ul.chat-module__messages-list(ref="messlist")
            li.chat-module__messages-items(
              v-for="item in getMessages"
              ref="messitem"
            )
              .chat-module__messages-body
                span.chat-module__messages-name {{ item.name }}
                span.chat-module__messages-text {{ item.mess }}
        form.chat-module__add-message
          textarea.chat-module__inp.chat-module__text(
            type="text"
            name="username"
            placeholder="Message"
            v-model="messtext"
            @keydown.13.prevent="sendMess"
            @keyup.shift.13="newLine"
            ref="messinp"
          )
          button.chat-module__send(@click.prevent="sendMess") send
</template>

<script>
import { mapState, mapMutations } from 'vuex'
export default {
  data () {
    return {
      name            : '',
      scroll          : true,
      isFlag          : true,
      isError         : false,
      isOnline        : 1,
      messtext        : '',
      usernameTrigger : true
    }
  },

  computed: mapState('chat', {
    getOnline      : state => state.online,
    getMessages    : state => state.allMess,
    getChatTrigger : state => state.trigger
  }),

  created () {
    this.chatRoom = this.$DC.chatInit()
  },

  beforeMount () {
    const name = localStorage.getItem('username')

    if (typeof name !== 'undefined' && name !== null) {
      this.name            = name
      this.usernameTrigger = false
    }

    this.chatRoom.on('action::message', data => {
      this.updateAllMessage({
        name: data.message.name,
        mess: data.message.mess
      })
    })
  },

  mounted () {
    this.listenPeers()
  },

  updated () {
    if (typeof this.$refs.messages !== 'undefined') {
      if (this.scroll) {
        this.scrollDown()
      }
    }
  },

  methods: {
    ...mapMutations('chat', [
      'updateOnline',
      'updateAllMessage',
      'updateChatTrigger'
    ]),

    listenPeers () {
      setTimeout(() => {
        if (this.chatRoom.channel !== false) {
          this.chatRoom.channel.on('peer joined', peer => {
            this.updateOnline(++this.isOnline)
          })
          this.chatRoom.channel.on('peer left',   peer => {
            this.updateOnline(--this.isOnline)
          })

          return
        }

        this.listenPeers()
      }, 1111)
    },

    chatOver () {
      this.updateChatTrigger(false)
      this.isFlag = true
    },

    scrollDown () {
      if (!this.isFlag && typeof this.$refs.messitem !== 'undefined' &&
      this.$refs.messitem.length >= 3 &&
      this.$refs.messitem[this.$refs.messitem.length - 3].getBoundingClientRect().bottom > 900) {
        return
      }

      this.isFlag = false
      this.$refs.messages.scrollTop = this.$refs.messages.scrollHeight
    },

    addUser () {
      this.isError = (this.name.length > 10)
        ? true
        : false

      if (this.name &&
      this.name !== '' && !this.isError) {
        localStorage.setItem('username', this.name)
        this.usernameTrigger = false
      }
    },

    newLine (e) {
      this.messtext += '\n'
      this.$refs.messinp.scrollTop = this.$refs.messinp.scrollHeight
    },

    sendMess (e) {
      if (e.keyCode === 13 && e.shiftKey) return

      if (this.messtext && this.messtext !== '' && !this.usernameTrigger) {
        this.updateAllMessage({
          name: this.name,
          mess: this.messtext
        })

        this.chatRoom.sendMsg({
          action: 'message',
          message: {
            name: this.name,
            mess: this.messtext
          }
        })

        this.messtext = ''
      }
    }
  }
}
</script>

<style lang="scss">
@import './index';
.chat {
  position: absolute;
  width: 100vw;
  height: 100vh;
  &.index {
    display: none
  }
  &-bg {
    position: fixed;
    z-index: 400;
    width: 100vw;
    height: 100vh;
  }
  &-module {
    position: fixed;
    padding: 20px 10px;
    z-index: 400;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    align-items: center;
    width: 350px;
    height: 100vh;
    @media screen and(max-width: 480px) {
      width: 100vw
    }
    &__header {
      position: relative;
      display: flex;
      justify-content: center;
      width: 100%;
    }
    &__online {
      position: absolute;
      top: 3px;
      left: 10px;
    }
    &__close {
      position: absolute;
      right: 10px;
      width: 25px;
      height: 25px;
      &-but {
        cursor: pointer;
        position: absolute;
        width: 25px;
        height: 5px;
        border: none;
        outline: none;
        border-radius: $radius;
        transform: translateY(7px) rotate(45deg);
        &:before {
          content: "";
          @extend .chat-module__close-but;
          position: absolute;
          top: 0;
          right: 0;
          transform: rotate(-90deg);
        }
      }
    }
    &__username {
      position: absolute;
      top: 0;
      right: 0;
      left: 0;
      bottom: 0;
      z-index: 100;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      border-radius: $radius;
      &-inp {
        margin-top: 20px;
        padding: 10px;
        height: 30px;
        text-align: center;
        border-radius: $radius
      }
      &-send {
        cursor: pointer;
        padding: 5px 10px;
        margin-top: 20px;
        border-radius: $radius
      }
    }
    &__messages {
      position: relative;
      margin: 20px 10px;
      width: 100%;
      height: 100%;
      display: flex;
      border-radius: $radius;
      overflow-y: auto;
      &:scrollbar {
        width: 0;
      }
      &-list {
        position: relative;
        padding-top: 10px;
        display: flex;
        flex-direction: column;
        justify-content: flex-end;
        width: 100%;
        min-height: min-content;
      }
      &-body {
        margin: 0 10px 10px 10px;
        padding: 10px;
        display: flex;
        flex-direction: column;
        align-items: flex-end;
        min-width: 200px;
        border-radius: $radius;
      }
      &-text {
        display: inline-block;
        word-break: break-word;
        word-wrap: break-word;
        overflow-wrap: break-word;
        -moz-user-select: -moz-all;
        -o-user-select: all;
        -khtml-user-select: all;
        -webkit-user-select: all;
        user-select: all
      }
    }
    &__add-message {
      display: flex;
      justify-content: flex-start;
      align-items: center;
      width: 100%;
    }
    &__inp {
      cursor: pointer;
      padding: 10px;
      width: 77%;
      height: 50px;
      border-radius: $radius;
      resize: none;
    }
    &__send {
      cursor: pointer;
      margin-left: auto;
      padding: 0 10px;
      height: 50px;
      border-radius: $radius;
    }
  }
}
</style>
