<template lang="pug">
div
  div(class="chat-container", @click="focusMsgArea")
    div(v-if="!chat_hidden", class="click-to-collapse", @click="collapseChat")
      i(class="fa fa-angle-right" aria-hidden="true")
    p(class="chat-txt") Chat
    div(class="chat-header")
      div(class="col-xs-9 inner-chat-header")
        input(class="dropdown-toggle" type="text")
        div(class="dropdown-text") {{selected_tab.name}}
        span(class="glyphicon glyphicon-triangle-bottom" aria-hidden="true")
        ul(class="chat-tabs-wrap dropdown-content")
          li(class="chat-tab", v-on:click="updateSelectedTab('General')")
            span() {{ active_conversations.General.name }}
          li(v-if="user.club_id != null", class="chat-tab", v-on:click="updateSelectedTab('Club')")
            span() {{ active_conversations.Club.name }}
          li(class="chat-tab" v-for="conversationName in conversations.index" v-on:click="updateSelectedTab(conversationName)")
            span() {{ active_conversations[conversationName].name }}
            span(v-if="active_conversations[conversationName].unread", class="unread")  !
      div(class="col-xs-3 active-users", v-on:click="toggleOnlinePanel")
        div(class="green-orb")
        div(class="user-count") {{online_count + guests.length}}
    //- active conversations indicator bar
    div(class="conversations-wrap")
      ul
        li(class="" v-for="conversationName in conversations.index" v-on:click="updateSelectedTab(conversationName)")
          div.img-wrap
            img(:src="active_conversations[conversationName].avatar_file_name")
          span(v-if="active_conversations[conversationName].unread", class="unread")  !
          div(class="online-indicator", v-if="isUserOnline(active_conversations[conversationName].userId)")
    div(v-if="chat_hidden", class="click-to-expand", @click="collapseChat")
      i(class="fa fa-angle-left" aria-hidden="true")
    p(v-if="chat_hidden", style="text-align: center; margin-top: 2px; color: white;") Expand
    div(v-show="!showOnline", class="chat-content-wrap")
      div(id="chat-wrap", class="inner-chat-wrap", transition="fade")
        div(class="chat-area")
          ul(id="chat-messages-container")
            li(class="chat-message" v-for="chatMsg in selected_tab.messages")
              div(class="from")
                img(v-if="chatMsg.avatar", :src="chatMsg.avatar", class="avatar")
                img(v-else, src="/img/layout-graphics/default-avatar.png", class="avatar")
                div(class="username-date-wrap")
                  span(class="username") {{chatMsg.username}}
                  span(class="date") {{prettyDate(chatMsg.timestamp)}}
                //- context-menu(:user="",:user-id="chatMsg.user_id", :username="chatMsg.username")
              span(class="message") {{chatMsg.msg}}
      //- div(class="message-area" contenteditable="true" placeholder="enter chat message", v-model="message")
      form(class="message-area")
        textarea(:disabled="user==null" type="text" id="chat-submit" v-model="selected_tab.msg" placeholder="Type to send", @keydown="focusMsgArea",@keyup.enter="submitMsg", @click="focusMsgArea")
    div(v-show="showOnline", class="online-users-wrap", transition="fade")
      p(class="user-stats") {{online_count}} Registered Users, {{guests.length}} Guests
      ul(class="")
        h5 Users: ( {{ online_count }} )
        li(v-for="menuUser in online_users" v-on:click="showMenu(menuUser)")
          span
            i.fa.fa-user &nbsp;
          span {{ menuUser.username }} (\#{{ menuUser.id }})
          context-menu(v-if="menuUser.show_menu && user.authenticated", :menu-user="menuUser", :user="user", :club="club", :showOnlinePanel.sync="showOnline")
        br
        h5 Guests: ( {{ guests.length }} )
        li(v-for="guest in guests") {{ guest }}
          //- span(v-if="selected_tab.unread") unread
</template>

<script>
import { mapState } from 'vuex'
import ContextMenu from './ContextMenu.vue'
import { notGuest } from '../helpers'
import moment from 'moment'
import { UPDATE_SELECTED_TAB } from '../store/mutation-types'

export default {

  props: [],
  components: {
    ContextMenu
  },
  data () {
    return {
    msg: null,
    //private messages index
    //userId acts as index to chat_tab index value
    pmi: [],
    message: null,
    messages: [],
    club_tab_index: null,
    chat_tabs: [
      {type: "general", messages: [], name:"General Chat"}
        ],
    showOnline: false,
    msgAlert: false,
    msgFrom: null,
    chat_hidden: false
    }
  },
  computed: mapState({
    'selected_tab': state => state.chat.selected_tab,
    'user' (state) {
      console.log('outputting computed store_user property', state.user)
      this.setupClubTab()
      return state.user
    },
    'club': state => {
      // console.log('outputting computed club property', club)
      return state.user.club
    },
    'dsClient': state => state.hardline,
    'online_users': state => state.online_users,
    'online_count' (state) {
      return Object.keys(state.online_users).length
    },
    'guests': state => state.guests,
    'active_conversations': state => state.chat.active_conversations,
    // conversations
    'conversations': state => state.chat.conversation_history,
    // promise alerting the websocket connection and chat data has been bootstrapped
    'bootstrapChat': state => state.chat.bootstrapChat
  }),
  watch: {
    active_conversations (user, oldUser) {
      // this.setupClubTab()
      this.updateScroll()
    }
  },
  methods: {
    collapseChat(){
      if(!this.chat_hidden) {
        this.chat_hidden = true
        $('#main-view-column').removeClass('col-md-9').addClass('col-md-11')
        $('#chat-column').removeClass('col-md-3').addClass('col-md-1')
        localStorage.setItem('chat_hidden', true)
      } else {
        this.chat_hidden = false
        $('#main-view-column').removeClass('col-md-11').addClass('col-md-9')
        $('#chat-column').removeClass('col-md-1').addClass('col-md-3')
        localStorage.removeItem('chat_hidden')
        this.updateScroll()
      }
    },
    updateScroll(){
      var element = document.getElementById("chat-wrap");
      element.scrollTop = element.scrollHeight;
    },
    notGuest,
  toggleOnlinePanel(){
    this.showOnline = !this.showOnline;
  },
  updateSelectedTab (conversationName) {
    this.$store.commit('UPDATE_SELECTED_TAB', conversationName)
    this.$store.dispatch('updateConversationRead', conversationName)

    if(this.showOnline){
      this.showOnline = false;
    }
  },
  focusMsgArea () {
    if (this.selected_tab.userId && this.selected_tab.userId != this.user.id && this.selected_tab.unread) {
      var conversationName = getConversationName(this.user.id, this.selected_tab.userId)
      this.$store.dispatch('updateConversationRead', conversationName)
    }
  },
  showMenu(user){
    // if(!user.showMenu)
    //  user.showMenu = false;
    console.log('running showMenu func', user)
    user.show_menu = true;
  },
  isUserOnline (userId) {
    return this.online_users.hasOwnProperty(userId)
  },
  validateMsg(e){
    if (!this.selected_tab.msg.trim()) {
      return false
    }
    // don't submit on shift+enter
    if (e.shiftKey) {
      return false
    }
    if(!this.selected_tab.msg || this.selected_tab.msg.length == 0) {
      return false
    }
    return true
  },

  submitMsg(e){
    if(!this.validateMsg(e)){
      return;
    }

    var payload = { type: this.selected_tab.type, message: this.selected_tab.msg }

    if (this.selected_tab.type == 'private') {
      var userId = this.selected_tab.userId
      payload.recipient = userId
    } else if (this.selected_tab.type == "club") {
      payload.club_id = this.club.id;
    }

    this.$store.dispatch('dispatchNewMsg', payload)

    this.selected_tab.msg = null;
    this.updateScroll();
  },
  prettyDate (unix_timestamp) {
    return moment.unix(unix_timestamp).format("MMMM Do YYYY, h:mm:ss a")
  },
    setupClubTab(){
      var that = this;

        if(!that.club)
          return false;

      //if club tab not found then establish it
      if(!that.club_tab_index){
        that.chat_tabs.push({type: "club", name: that.club.name, messages: [], club_id: that.club.id});
        that.club_tab_index = that.chat_tabs.length - 1;
      }

      //fetch chat history
      var request = $.ajax({
        url: "/api/clubChat",
        method: "POST",
        processData: false,
        contentType: 'json',
        dataType: "json",
        data: JSON.stringify({'jwt': that.jwt})
      });

      request.done(function( msg ) {
        if(msg.length > 0){
          that.chat_tabs[that.club_tab_index].messages = msg; 
        }
      });

      request.fail(function( jqXHR, textStatus ) {
        // var resp = JSON.parse(jqXHR.responseText)
      });
    }
  },
  created(){
    var that = this;

    //setup the users guild chat tab
    // that.setupClubTab();

    // commit the general chat tab
    that.$store.commit(UPDATE_SELECTED_TAB, 'General')
  },
  mounted(){
    var that = this;

    // check localStorage chat hidden status
    var localHidden = localStorage.getItem("chat_hidden")
    if (localHidden) {
      this.collapseChat()
    }

    // update the scroll position of chat to bottom once all data and conversations are loaded
    this.bootstrapChat.then(()=>this.updateScroll())
  }
}

function getTabHistory(to_id, from_id, jwt){
      //fetch chat history
  var request = $.ajax({
        url: "/api/tabHistory",
        method: "POST",
        processData: false,
        contentType: 'json',
        dataType: "json",
        data: JSON.stringify({to: to_id, from: from_id, 'jwt': jwt})
  });
  return request;
}

function getConversationName(userId, targetUserId) {
  var conversationName

  // setup the deepstream list that contains the chat messages for this conversation
  // sort the ids, lowest goes first
  if(targetUserId < userId){
    conversationName = 'chat/' + targetUserId + '/' + userId
  } else {
    conversationName = 'chat/' + userId + '/' + targetUserId
  }
  return conversationName
}
</script>

<style scoped lang="stylus">
  .conversations-wrap
    ul
      text-align: right;
      padding: 0 10px;
      display: block;
      margin: 10px 0;
      li
        display: inline-block;
        width: 40px;
        cursor: pointer;
        height: 40px;
        margin: 0 2px;
        position relative
        .img-wrap
          background rgba(255, 255, 255, 0.5)
          height: 100%
          width: 100%
          overflow hidden
          border-radius 50%
          img
            width 100%
        .unread
          position absolute
          top 0
          left 0
          font-size 28px
          color #fd4800
        .online-indicator
          height: 10px;
          width: 10px;
          background: #42b72a;
          position: absolute;
          border-radius: 50%;
          bottom: 0;
          right: 0;
          border solid 1px grey
  .chat-container
    background: #2b5582;
    border-radius: 20px;
    min-height: 30vw;
    position: relative;
    background: -webkit-linear-gradient(top, #7474bf, #348ac7);
    background: linear-gradient(to bottom, #7474bf, #348ac7);
    box-shadow: 0px 5px 18px 2px #000;
    border: solid 1px rgba(255,255,255,0.22);
    .message-area
      margin-left 10px
      margin-right 10px
      &[contenteditable=true]:empty:before
        content: attr(placeholder);
        display: block; /* For Firefox */
    .chat-header
      color: #fff;
      text-align: center;
      padding: 6px 10px 0px 10px;
      height 40px
      .inner-chat-header
        border-radius: 10px;
        border: solid 1px #d2d2d2;
        cursor: pointer;
        vertical-align: baseline;
        position: relative;
        text-align: center;
        padding: 0;
        span.glyphicon-triangle-bottom
          margin-left: 146px;
          display: inline-block;
          position: absolute;
          top: 8px;
          right: 12px;
        &:hover
          background: #3f87c6;
    .inner-chat-wrap
      background: #fff;
      margin-bottom: 10px;
      border: solid 2px #3f87c6;
      border-radius: 10px;
      margin-left 10px
      margin-right 10px
      height 500px
      overflow-y scroll
      .chat-area
        margin-bottom: 12px;
    #chat-submit
      border-radius: 10px;
      width: 100%;
      padding: 10px;
    .from
      position: relative;
    .chat-message
      padding: 20px 18px;
      word-wrap: break-word;
      .message
        margin-top: 6px;
        display: inline-block;
      .avatar
        height: 40px;
        width: 40px;
        border-radius: 40px;
        margin-right: 6px;
        vertical-align: top;
      .username-date-wrap
        display: inline-block;
        span
          display: block;
        .username
          font-weight: 600;
          color: #2c2d30;
        .date
          font-size: 12px
  .chat-tabs-wrap
    text-align: left;
    z-index: 100;
    // margin-bottom: 0;
    position: relative;
    // bottom: 0px;
    .chat-tab
      display: block;
      background: #5180c3;
      padding: 6px;
      line-height: 14px;
      border-bottom: solid 1px #c0c0c0;
      &:hover
        background: #6ba1ec;
  .dropdown-toggle
    font-size: 0;
    z-index: 1;
    cursor: pointer;
    position: absolute;
    top: 0;
    left: 0;
    border: none;
    padding: 0;
    margin: 0 0 0 1px;
    background: transparent;
    text-indent: -10px;
    height: 30px;
    width: 100%;
    &:focus
      outline: 0;
      background: #3f87c6;
      border-radius: 10px;
  .dropdown-text
    padding: 5px;
    border-radius: 10px;
  .dropdown-content
    -webkit-transition: all .25s ease;
    -moz-transition: all .25s ease;
    -ms-transition: all .25s ease;
    -o-transition: all .25s ease;
    transition: all .25s ease;
    list-style-type: none;
    position: absolute;
    top: 32px;
    padding: 0;
    margin: 0;
    opacity: 0;
    visibility:hidden;
    border-radius: 3px;
    text-indent: 10px;
    line-height: 32px;
    background-color: #eee;
    border: 1px solid #ccc;
    width: 90%;
    margin: 0 auto;
    left: 0;
    right: 0;
    border-top: 0;
  .dropdown-toggle:focus ~ .dropdown-text
    position: relative;
    box-shadow: 0 0px 8px rgba(0, 0, 0, 0.82) inset, 0 1px 0 rgba(255,255,255,0.8);
    z-index: 2;
  .dropdown-toggle:focus ~ .dropdown-content
    opacity: 1;
    visibility:visible;
    top: 31px;
  .active-users
    border-radius: 10px;
    border: solid 1px #d2d2d2;
    cursor: pointer;
    padding: 0;
    vertical-align: baseline;
    height: 32px;
    line-height: 30px;
    &:hover
      background: #3f87c6;
    .green-orb
      background: rgb(66, 183, 42);
      border-radius: 50%;
      display: inline-block;
      height: 10px;
      width: 10px;
    .user-count
      display: inline-block;
      margin-left: 10px;
  .online-users-wrap
    max-height: 500px;
    overflow-y: scroll
    padding: 10px 24px 0;
    ul
      color: white;
      li
        text-decoration:underline;
        cursor: pointer;
        position: relative;
  .unread
    color: #ffa500;
    font-size: 22px;
    font-family: 'Russo One';
    text-shadow: 0px 1px 1px black;
    position: relative;
    top: 2px;
  .chat-txt
    display none
  .click-to-expand
    display none
</style>