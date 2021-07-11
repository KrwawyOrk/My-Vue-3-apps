<template>
  <section
    v-if="userLoggedIn"
    class="flex mx-auto"
  >
    <div class="chat-window flex flex-col p-2 bg-gray-50 border max-w-screen-md max-h-screen">
      <div class="px-3 mb-2 rounded" style="background-color: #e46b1b">
        <h1 class="text-lg text-white font-bold">
          Hello, <i>{{ userEmailAdress }}</i>
        </h1>
      </div>
      <div class="flex justify-between">
        <button
          @click="getRandomAdvice"
          class="
            bg-blue-500
            hover:bg-blue-700
            text-white
            font-bold
            py-1
            px-4
            mr-1
            rounded
            text-sm
             h-10
          "
        >
          Get advice message
        </button>
        <button
          @click="scrollDownAfterAppendMessage"
          class="
            bg-blue-500
            hover:bg-blue-700
            text-white
            font-bold
            py-1
            px-4
            mx-1
            rounded
            text-sm
            h-10
          "
        >
          Scroll down
        </button>
        <button
          @click="leaveChat"
          class="
            bg-yellow-600
            hover:bg-yellow-700
            text-white
            font-bold
            py-1
            px-4
            ml-1
            rounded
            text-sm
            h-10
          "
        >
          Leave chat
        </button>
      </div>

      <div
        id="messages-container"
        class="flex flex-col mt-1 overflow-y-scroll h-screen"
      >
        <transition-group name="messages-list">
          <div
            v-for="chat in state.chats"
            :key="chat.message"
            class="my-3 p-2 w-7/12 rounded-3xl border relative"
            :class="
              userEmailAdress === chat.userEmail
                ? 'self-end bg-blue-500'
                : 'self-start bg-gray-200'
            "
          >
            <div>
              <h1
                class="break-words text-base"
                :class="
                  userEmailAdress === chat.userEmail
                    ? 'text-right text-white mr-1'
                    : 'text-left text-gray-600 ml-1'
                "
              >
                <h1
                  v-if="state.showEmailInMessage"
                  class="text-xs font-light italic"
                >
                  {{ chat.userEmail }}
                </h1>
                <h1 class="mt-1">{{ chat.message }}</h1>
                <h1 class="text-xs mt-3">{{ chat.sendedTime }}</h1>
              </h1>
              <div class="relative">
                <button
                  v-if="userEmailAdress === chat.userEmail"
                  @click="thumbUpMessage(chat.id)"
                  class="
                    absolute
                    -top-1
                    right-full
                    px-3
                    border-2 border-white
                    rounded-3xl
                    bg-gray-100
                    focus:outline-none
                  "
                >
                  <i
                    class="fas fa-thumbs-up"
                    :class="
                      chat.thumbUp === true ? 'text-blue-400' : 'text-gray-300'
                    "
                  ></i>
                </button>
                <button
                  v-else
                  @click="thumbUpMessage(chat.id)"
                  class="
                    absolute
                    -top-1
                    left-full
                    px-3
                    border-2 border-white
                    rounded-3xl
                    bg-gray-100
                    focus:outline-none
                  "
                >
                  <i
                    class="fas fa-thumbs-up"
                    :class="
                      chat.thumbUp === true ? 'text-blue-400' : 'text-gray-300'
                    "
                  ></i>
                </button>
              </div>
            </div>
          </div>
        </transition-group>
      </div>

      <div class="flex">
        <input
          v-model="inputMessage"
          @keyup.enter="appendMessage"
          class="w-10/12 py-2 px-2 border rounded-l-md shadow bg-gray-200 focus:outline-none"
          :placeholder="inputMessage ? '' : 'Write something in the chat room'"
        />
        <button
          @click="appendMessage"
          class="
            w-2/12
            bg-blue-500
            hover:bg-blue-700
            text-white
            font-bold
            rounded-r-md
            focus:outline-none
          "
        >
          Send
        </button>
      </div>
    </div>
  </section>
</template>

<script>
import { useStore } from "vuex";
import { onMounted, ref, reactive, computed, nextTick, watchEffect } from "vue";

import router from "../router.js";
import firebase from "../utilities/firebase";
import axios from "axios";

export default {
  setup() {
    const store = useStore();

    const inputMessage = ref("");

    const state = reactive({
      chats: [],
      showEmailInMessage: true,
    });

    const userId = computed(() => store.state.authUser.uid);
    const userLoggedIn = computed(() => store.state.isLoggedIn);
    const userEmailAdress = computed(() => store.state.authUser.email);

    onMounted(async () => {
      const db = firebase.database();
      const chatsRef = db.ref("chats");
      const data = await chatsRef.once("value");

      state.chats = data.val();

      chatsRef.on("value", (snapshot) => {
        const data = snapshot.val();
        let chats = [];

        Object.keys(data).forEach((key) => {
          chats.push({
            id: key,
            message: data[key].message,
            sendedTime: data[key].sendedTime,
            userEmail: data[key].userEmail,
            userId: data[key].userId,
            thumbUp: data[key].thumbUp,
          });
        });

        state.chats = chats;

        nextTick(() => {
          scrollDownAfterAppendMessage();
        });
      });
    });

    watchEffect(() => {
      if (!userLoggedIn.value === true) {
        router.replace("/");
      }
    });

    function appendMessage() {
      if (inputMessage.value === "") {
        return;
      }

      const chatsRef = firebase.database().ref("chats");

      const msg = {
        userEmail: this.userEmailAdress,
        userId: this.userId,
        message: inputMessage.value,
        sendedTime: getCurrentHourAndMinutesToString(),
        thumbUp: false,
      };

      chatsRef.push(msg);
      inputMessage.value = "";
    }

    function showUserEmails() {
      state.showEmailInMessage = !state.showEmailInMessage;
    }

    function leaveChat() {
      firebase.auth().signOut();
    }

    function scrollDownAfterAppendMessage() {
      const messagesContainer = document.getElementById("messages-container");
      messagesContainer.scrollTop = messagesContainer.scrollHeight;
    }

    function getCurrentHourAndMinutesToString() {
      const date = new Date();
      const dateString = date.getHours() + ":" + date.getMinutes();

      return dateString;
    }

    function thumbUpMessage(id) {
      const msg = firebase.database().ref("chats").child(id).child("thumbUp");
      msg.transaction((thumbUp) => !thumbUp);
    }

    function getRandomAdvice() {
      axios.get("https://api.adviceslip.com/advice").then((response) => {
        const {
          data: {
            slip: { advice },
          },
        } = response;

        inputMessage.value = advice;
      });
    }

    return {
      state,
      leaveChat,
      showUserEmails,
      appendMessage,
      inputMessage,
      scrollDownAfterAppendMessage,
      getCurrentHourAndMinutesToString,
      userId,
      userEmailAdress,
      userLoggedIn,
      thumbUpMessage,
      getRandomAdvice,
    };
  },
};
</script>

<style>
.messages-list-enter-active,
.messages-list-leave-active {
  transition: all 0.7s ease-out;
}
.messages-list-enter-from,
.messages-list-leave-to {
  opacity: 0;
}

.chat-window {
  animation-name: show_chat;
  animation-duration: 0.4s;
  animation-timing-function: ease-out;
}

@keyframes show_chat {
  from {
    opacity: 0;
    transform: translate(-100%, 0);
  }

  to {
    opacity: 1;
    transform: translate(0, 0);
  }
}
</style>