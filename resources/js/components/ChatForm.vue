<template>
  <div class="row">
    <div class="col-8">
      <div class="card card-default">
        <div class="card-header">Messages</div>
        <div class="card-body p-0">
          <ul class="chat" v-chat-scroll>
            <li
              class="left clearfix ml-3"
              v-for="(message, index) in messages"
              :key="index"
            >
              <div class="chat-body clearfix">
                <div class="header">
                  <strong class="primary-font">
                    <img
                      class="img-profile rounded-circle"
                      style="width: 20px; height: 20px; margin-right: 3px"
                      :src="'/images/' + message.user.photo"
                    />
                    {{ message.user.name }}
                  </strong>
                </div>
                <div>
                  <p style="margin-left: 26px">
                    {{ message.content }}
                  </p>
                  <span
                    style="
                      font-size: 10px;
                      float: right;
                      margin-top: -16px;
                      margin-right: 5px;
                    "
                  >
                    <span>{{ $moment(message.created_at).fromNow() }}</span>
                    <i class="fas fa-angle-double-down"></i>
                  </span>
                </div>
              </div>
            </li>
          </ul>
        </div>

        <input
          @keydown="startTyping"
          @keyup.enter="sendMessage"
          v-model="newMessage"
          type="text"
          name="message"
          placeholder="Enter your message..."
          class="form-control"
        />
      </div>
      <span class="text-muted" v-if="activeUser"
        >{{ activeUser.name }} is typing ...</span
      >
    </div>

    <div class="col-4">
      <div class="card card-default">
        <div class="card-header">Active Users</div>
        <div class="card-body">
          <ul class="list-unstyled">
            <li class="py-2" v-for="(user, index) in users" :key="index">
              <img
                class="img-profile rounded-circle"
                style="width: 20px; height: 20px"
                :src="'/images/' + user.photo"
              />
              {{ user.name }}
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: ["user"],

  data() {
    return {
      messages: [],
      newMessage: "",
      users: [],
      activeUser: false,
      typingTimer: false,
    };
  },

  mounted() {
    this.getMessages();

    Echo.join("chat")
      .here((user) => {
        this.users = user;
      })
      .joining((user) => {
        this.users.push(user);
      })
      .leaving((user) => {
        this.users = this.users.filter((u) => u.id != user.id);
      })
      .listen("ChatMessageWasReceived", (e) => {
        this.messages.push(e.message);
      })
      .listenForWhisper("typing", (user) => {
        this.activeUser = user;

        if (this.typingTimer) {
          clearTimeout(this.typingTimer);
        }

        this.typingTimer = setTimeout(() => {
          this.activeUser = false;
        }, 1000);
      });
  },

  methods: {
    getMessages() {
      axios.get("messagess").then((response) => {
        this.messages = response.data;
      });
    },

    sendMessage() {
      if (this.newMessage.length > 0) {
        this.messages.push({
          user: this.user,
          message: this.newMessage,
        });

        axios.post("messagess", { message: this.newMessage });
        this.getMessages();
        this.newMessage = "";
      }
    },

    startTyping() {
      Echo.join("chat").whisper("typing", this.user);
    },
  },

  filters: {
    moment: function (date) {
      return moment(date).format("MMMM Do YYYY, h:mm:ss a");
    },
  },
};
</script>