<template>
  <v-app id="rotki">
    <v-navigation-drawer
      v-model="drawer"
      fixed
      :clipped="$vuetify.breakpoint.mdAndUp"
      class="grey lighten-4"
      app
    >
      <div class="text-center">
        <img
          id="rotkehlchen_no_text"
          :src="require('./assets/images/rotkehlchen_no_text.png')"
          class="rotki__logo"
          alt=""
          @click="openSite()"
        />
      </div>
      <div class="text-center rotki__welcome-text font-weight-medium">
        Welcome {{ username }}
      </div>
      <navigation-menu></navigation-menu>
    </v-navigation-drawer>
    <v-app-bar app fixed clipped-left flat class="grey lighten-4">
      <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon>
      <v-toolbar-title class="font-weight-light rotki__version">
        Rotki {{ version }}
      </v-toolbar-title>
      <node-status-indicator></node-status-indicator>
      <balance-saved-indicator></balance-saved-indicator>
      <v-spacer></v-spacer>
      <update-indicator></update-indicator>
      <notification-indicator></notification-indicator>
      <progress-indicator></progress-indicator>
      <user-dropdown></user-dropdown>
      <currency-drop-down></currency-drop-down>
    </v-app-bar>
    <v-content v-if="logged">
      <router-view></router-view>
    </v-content>
    <message-dialog
      :title="message.title"
      :message="message.description"
      :success="message.success"
      @dismiss="dismiss()"
    ></message-dialog>
    <error-screen
      v-if="startupError.length > 0"
      :message="startupError"
    ></error-screen>
    <account-management
      v-if="startupError.length === 0 && !loginIn"
      :logged="logged"
      @login-complete="loginComplete = true"
    ></account-management>
  </v-app>
</template>

<script lang="ts">
import { Component, Vue, Watch } from 'vue-property-decorator';
import UserDropdown from '@/components/UserDropdown.vue';
import NavigationMenu from '@/components/NavigationMenu.vue';
import CurrencyDropDown from '@/components/CurrencyDropDown.vue';
import { createNamespacedHelpers, mapGetters, mapState } from 'vuex';
import MessageDialog from '@/components/dialogs/MessageDialog.vue';
import NodeStatusIndicator from '@/components/status/NodeStatusIndicator.vue';
import BalanceSavedIndicator from '@/components/status/BalanceSavedIndicator.vue';
import NotificationIndicator from '@/components/status/NotificationIndicator.vue';
import ProgressIndicator from '@/components/status/ProgressIndicator.vue';
import './services/task-manager';
import ConfirmDialog from '@/components/dialogs/ConfirmDialog.vue';
import { Message } from '@/store/store';
import UpdateIndicator from './components/status/UpdateIndicator.vue';
import AccountManagement from './components/AccountManagement.vue';
import ErrorScreen from '@/ErrorScreen.vue';

const { mapState: mapSessionState } = createNamespacedHelpers('session');

@Component({
  components: {
    ErrorScreen,
    AccountManagement,
    UpdateIndicator,
    ProgressIndicator,
    NotificationIndicator,
    BalanceSavedIndicator,
    NodeStatusIndicator,
    MessageDialog,
    ConfirmDialog,
    CurrencyDropDown,
    NavigationMenu,
    UserDropdown
  },
  computed: {
    ...mapState(['message']),
    ...mapSessionState(['logged', 'username']),
    ...mapGetters(['version'])
  }
})
export default class App extends Vue {
  logged!: boolean;
  message!: Message;
  version!: string;

  loginComplete: boolean = false;

  get loginIn(): boolean {
    return this.logged && this.loginComplete;
  }

  @Watch('logged')
  onLoggedChange() {
    if (!this.logged) {
      this.loginComplete = false;
    }
  }

  drawer = true;

  startupError: string = '';

  openSite() {
  }

  dismiss() {
    this.$store.commit('resetMessage');
  }

  async created(): Promise<void> {
    this.$api.connect(4242);
  }
}
</script>

<style scoped lang="scss">
.rotki__logo {
  max-height: 150px;
}

.rotki__version {
  margin-right: 16px;
}

#rotkehlchen_no_text {
  width: 150px;
  display: block;
  margin: 10px auto;
  padding: 15px;
}

.rotki__welcome-text {
  font-size: 18px;
  margin-top: 16px;
  margin-bottom: 20px;
}

::v-deep .v-content__wrap {
  border-top: #bcbcbc solid thin;
}
</style>
