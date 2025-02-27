<template>
  <header>
    <b-navbar toggleable="sm" dark variant="primary">
      <router-link class="navbar-brand" exact to="/">
        <img class="logo" src="@/assets/freqtrade-logo.png" alt="Home Logo" />
        <span class="navbar-brand-title d-sm-none d-md-inline">Freqtrade UI</span>
      </router-link>

      <!-- TODO: For XS breakpoint, this should be here...  -->
      <!-- <ReloadControl class="me-3" /> -->
      <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>

      <b-collapse id="nav-collapse" class="text-right text-md-center" is-nav>
        <b-navbar-nav>
          <router-link v-if="!botStore.canRunBacktest" class="nav-link navbar-nav" to="/trade"
            >Trade</router-link
          >
          <router-link v-if="!botStore.canRunBacktest" class="nav-link navbar-nav" to="/dashboard"
            >Dashboard</router-link
          >
          <router-link class="nav-link navbar-nav" to="/graph">Chart</router-link>
          <router-link class="nav-link navbar-nav" to="/logs">Logs</router-link>
          <router-link v-if="botStore.canRunBacktest" class="nav-link navbar-nav" to="/backtest"
            >Backtest</router-link
          >
          <theme-select />
        </b-navbar-nav>

        <!-- Right aligned nav items -->
        <b-navbar-nav class="ms-auto" menu-class="w-100">
          <!-- TODO This should show outside of the dropdown in XS mode -->
          <div class="d-flex justify-content-between">
            <b-dropdown
              v-if="botStore.botCount > 1"
              size="sm"
              class="m-1"
              no-caret
              variant="info"
              toggle-class="d-flex align-items-center "
              menu-class="my-0 py-0"
            >
              <template #button-content>
                <BotEntry :bot="botStore.selectedBotObj" :no-buttons="true" />
              </template>
              <BotList :small="true" />
            </b-dropdown>
            <ReloadControl class="me-3" />
          </div>
          <li
            class="d-none d-sm-flex flex-sm-wrap flex-lg-nowrap align-items-center nav-item text-secondary me-2"
          >
            <b-nav-text class="verticalCenter small me-2">
              {{
                (botStore.activeBotorUndefined && botStore.activeBotorUndefined.botName) ||
                'No bot selected'
              }}
            </b-nav-text>
            <b-nav-text class="verticalCenter">
              {{
                botStore.activeBotorUndefined && botStore.activeBotorUndefined.isBotOnline
                  ? 'Online'
                  : 'Offline'
              }}
            </b-nav-text>
          </li>
          <li v-if="botStore.hasBots" class="nav-item">
            <!-- Hide dropdown on xs, instead show below  -->
            <b-nav-item-dropdown id="avatar-drop" right auto-close class="d-none d-sm-block">
              <template #button-content>
                <b-avatar size="2em" button>FT</b-avatar>
              </template>
              <span class="ps-3">V: {{ settingsStore.uiVersion }}</span>
              <router-link class="dropdown-item" to="/settings">Settings</router-link>
              <div class="ps-3">
                <b-form-checkbox v-model="layoutStore.layoutLocked">Lock layout</b-form-checkbox>
              </div>
              <b-dropdown-item @click="resetDynamicLayout">Reset Layout</b-dropdown-item>
              <router-link
                v-if="botStore.botCount === 1"
                class="dropdown-item"
                to="/"
                @click="clickLogout()"
                >Sign Out</router-link
              >
            </b-nav-item-dropdown>
            <div class="d-block d-sm-none">
              <!-- Visible only on XS -->
              <li class="nav-item text-secondary ms-2 d-sm-none d-flex justify-content-between">
                <div class="d-flex">
                  <b-nav-text class="verticalCenter small me-2">
                    {{
                      (botStore.activeBotorUndefined && botStore.activeBotorUndefined.botName) ||
                      'No bot selected'
                    }}
                  </b-nav-text>
                  <b-nav-text class="verticalCenter">
                    {{
                      botStore.activeBotorUndefined && botStore.activeBotorUndefined.isBotOnline
                        ? 'Online'
                        : 'Offline'
                    }}
                  </b-nav-text>
                </div>
              </li>
              <router-link class="nav-link navbar-nav" to="/settings">Settings</router-link>
              <router-link
                v-if="botStore.botCount === 1"
                class="nav-link navbar-nav"
                to="/"
                @click="clickLogout()"
                >Sign Out</router-link
              >
            </div>
          </li>
          <li v-else>
            <!-- should open Modal window! -->
            <LoginModal v-if="route?.path !== '/login'" />
          </li>
        </b-navbar-nav>
      </b-collapse>
    </b-navbar>
  </header>
</template>

<script setup lang="ts">
import LoginModal from '@/views/LoginModal.vue';
import ThemeSelect from '@/components/ThemeSelect.vue';
import Favico from 'favico.js';
import ReloadControl from '@/components/ftbot/ReloadControl.vue';
import BotEntry from '@/components/BotEntry.vue';
import BotList from '@/components/BotList.vue';
import { ref, onBeforeUnmount, onMounted, watch } from 'vue';
import { OpenTradeVizOptions, useSettingsStore } from '@/stores/settings';
import { useLayoutStore } from '@/stores/layout';
import { useBotStore } from '@/stores/ftbotwrapper';
import { useRoute } from 'vue-router';

const botStore = useBotStore();

const settingsStore = useSettingsStore();
const layoutStore = useLayoutStore();
const route = useRoute();
const favicon = ref<Favico | undefined>(undefined);
const pingInterval = ref<number>();

const clickLogout = () => {
  botStore.removeBot(botStore.selectedBot);
  // TODO: This should be per bot
};

const setOpenTradesAsPill = (tradeCount: number) => {
  if (!favicon.value) {
    favicon.value = new Favico({
      animation: 'none',
      // position: 'up',
      // fontStyle: 'normal',
      // bgColor: '#',
      // textColor: '#FFFFFF',
    });
  }
  if (tradeCount !== 0 && settingsStore.openTradesInTitle === 'showPill') {
    favicon.value.badge(tradeCount);
  } else {
    favicon.value.reset();
    console.log('reset');
  }
};
const resetDynamicLayout = (): void => {
  console.log(`resetLayout called for ${route?.fullPath}`);
  switch (route?.fullPath) {
    case '/trade':
      layoutStore.resetTradingLayout();
      break;
    case '/dashboard':
      layoutStore.resetDashboardLayout();
      break;
    default:
  }
};
const setTitle = () => {
  let title = 'freqUI';
  if (settingsStore.openTradesInTitle === OpenTradeVizOptions.asTitle) {
    title = `(${botStore.activeBotorUndefined?.openTradeCount}) ${title}`;
  }
  if (botStore.activeBotorUndefined?.botName) {
    title = `${title} - ${botStore.activeBotorUndefined?.botName}`;
  }
  document.title = title;
};

onBeforeUnmount(() => {
  if (pingInterval.value) {
    clearInterval(pingInterval.value);
  }
});

onMounted(async () => {
  await settingsStore.loadUIVersion();
  pingInterval.value = window.setInterval(botStore.pingAll, 60000);
});

settingsStore.$subscribe((_, state) => {
  const needsUpdate = settingsStore.openTradesInTitle !== state.openTradesInTitle;
  if (needsUpdate) {
    setTitle();
    setOpenTradesAsPill(botStore.activeBotorUndefined?.openTradeCount || 0);
  }
});

watch(
  () => botStore.activeBotorUndefined?.botName,
  () => setTitle(),
);
watch(
  () => botStore.activeBotorUndefined?.openTradeCount,
  () => {
    if (settingsStore.openTradesInTitle === OpenTradeVizOptions.showPill) {
      setOpenTradesAsPill(botStore.activeBotorUndefined?.openTradeCount ?? 0);
    } else if (settingsStore.openTradesInTitle === OpenTradeVizOptions.asTitle) {
      setTitle();
    }
  },
);
</script>

<style lang="scss" scoped>
.logo {
  vertical-align: middle;
  height: 30px;
}

.dropdown-toggle::after {
  display: none;
}

.navbar-brand-title {
  padding-left: 0.5em;
}
.navbar {
  padding: 0.2rem 0rem;
}

.router-link-active,
.nav-link:active {
  color: white !important;
}
</style>
