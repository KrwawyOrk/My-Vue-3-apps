<template>
  <section v-if="userLoggedIn" class="flex mx-auto">
    <div
      class="
        chat-window
        flex flex-col
        p-2
        bg-gray-50
        border
        max-w-screen max-h-screen
        sm:max-w-screen-md
      "
    >
      <div class="px-3 mb-2 rounded" style="background-color: #e46b1b">
        <h1 class="text-lg text-white font-bold">
          Hello, <i>{{ userEmailAdress }}</i>
        </h1>
        <h1 class="text-xs text-white mb-2">Current chat: {{ state.currentChatRoom }}</h1>
      </div>
      <div class="flex justify-between">
        <button
          @click="getRandomAdvice"
          class="
            bg-blue-500
            hover:bg-blue-700
            text-white
            font-bold
            py-2
            px-4
            mr-1
            rounded
            text-sm
          "
        >
          <i class="fas fa-comments fa-2x mx-1"></i>Get advice message
        </button>
        <button
          @click="changeChatRoom"
          class="
            bg-blue-500
            hover:bg-blue-700
            text-white
            font-bold
            py-2
            px-4
            mx-1
            rounded
            text-sm
          "
        >
          Change room
        </button>
        <button
          @click="leaveChat"
          class="
            bg-yellow-600
            hover:bg-yellow-700
            text-white
            font-bold
            py-2
            px-4
            ml-1
            rounded
            text-sm
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
            :key="chat.key"
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
                  @click="thumbUpMessage(chat.key)"
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
                  @click="thumbUpMessage(chat.key)"
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
          class="
            w-10/12
            py-2
            px-2
            border
            rounded-l-md
            shadow
            bg-gray-200
            focus:outline-none
          "
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
import { onMounted, ref, reactive, computed, watchEffect, nextTick } from "vue";

import router from "../router.js";
import firebase from "../utilities/firebase";
import axios from "axios";

export default {
  setup() {
    const inputMessage = ref("");

    const state = reactive({
      chats: [],
      showEmailInMessage: true,
      currentChatRoom: "global",
    });

    const store = useStore();
    const userId = computed(() => store.state.authUser.uid);
    const userLoggedIn = computed(() => store.state.isLoggedIn);
    const userEmailAdress = computed(() => store.state.authUser.email);

    onMounted(async () => {
      const chatsRef = firebase
        .database()
        .ref(`chats/${state.currentChatRoom}`);

      chatsRef.on("child_changed", (snapshot) => {
        const msgRef = state.chats.find(
          (element) => element.key === snapshot.key
        );
        msgRef.thumbUp = snapshot.val().thumbUp;
      });

      chatsRef.on("child_added", (snapshot) => {
        state.chats.push({ key: snapshot.key, ...snapshot.val() });

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

    function thumbUpMessage(id) {
      const msgRef = firebase
        .database()
        .ref(`chats/${state.currentChatRoom}`)
        .child(id)
        .child("thumbUp");
      msgRef.transaction((thumb) => !thumb);
    }

    function appendMessage() {
      if (inputMessage.value === "") {
        return;
      }

      const newChat = firebase
        .database()
        .ref(`chats/${state.currentChatRoom}`)
        .push();

      newChat.set({
        userEmail: this.userEmailAdress,
        userId: this.userId,
        message: inputMessage.value,
        sendedTime: getCurrentHourAndMinutesToString(),
        thumbUp: false,
      });

      inputMessage.value = "";
    }

    function leaveChat() {
      firebase.auth().signOut();
    }

    async function changeChatRoom() {
      console.log("Changing chat room....");

      if(state.currentChatRoom == "global") {
        state.currentChatRoom = "advices";
        const newChat = await firebase
          .database()
          .ref(`chats/${state.currentChatRoom}`)
          .get();
        state.chats = newChat.val();
      }

      else if(state.currentChatRoom == "advices") {
        state.currentChatRoom = "global";
        const newChat = await firebase
          .database()
          .ref(`chats/${state.currentChatRoom}`)
          .get();
        state.chats = newChat.val();
      }

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

    async function getRandomAdvice() {
      await axios
        .get("https://api.adviceslip.com/advice")
        .then((response) => {
          const {
            data: {
              slip: { advice },
            },
          } = response;

          inputMessage.value = advice;
        })
        .finally(() => {
          console.log("I got an advice!");
        });
    }

    return {
      state,
      leaveChat,
      changeChatRoom,
      appendMessage,
      inputMessage,
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