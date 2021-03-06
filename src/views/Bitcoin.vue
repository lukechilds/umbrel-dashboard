<template>
  <div class="p-sm-2">
    <div class="my-3 pb-2">
      <div class="d-flex justify-content-between align-items-center">
        <div class="d-flex justify-content-start align-items-center">
          <img
            style="height: auto; width: 12vw; max-width: 100px"
            class="mr-2 mr-sm-3"
            src="@/assets/icon-app-bitcoin.svg"
          />
          <div>
            <svg
              width="8"
              height="8"
              viewBox="0 0 8 8"
              fill="none"
              xmlns="http://www.w3.org/2000/svg"
            >
              <circle cx="4" cy="4" r="4" fill="#00CD98" />
            </svg>
            <small class="ml-1 text-success">Running</small>
            <h3 class="d-block font-weight-bold mb-1">Bitcoin Core</h3>
            <span class="d-block text-muted">
              {{
              version ? `v${version}` : "..."
              }}
            </span>
          </div>
        </div>
        <div>
          <b-dropdown variant="link" toggle-class="text-decoration-none p-0" no-caret right>
            <template v-slot:button-content>
              <svg
                width="18"
                height="4"
                viewBox="0 0 18 4"
                fill="none"
                xmlns="http://www.w3.org/2000/svg"
              >
                <path
                  fill-rule="evenodd"
                  clip-rule="evenodd"
                  d="M2 4C3.10457 4 4 3.10457 4 2C4 0.89543 3.10457 0 2 0C0.89543 0 0 0.89543 0 2C0 3.10457 0.89543 4 2 4Z"
                  fill="#C3C6D1"
                />
                <path
                  fill-rule="evenodd"
                  clip-rule="evenodd"
                  d="M9 4C10.1046 4 11 3.10457 11 2C11 0.89543 10.1046 0 9 0C7.89543 0 7 0.89543 7 2C7 3.10457 7.89543 4 9 4Z"
                  fill="#C3C6D1"
                />
                <path
                  fill-rule="evenodd"
                  clip-rule="evenodd"
                  d="M16 4C17.1046 4 18 3.10457 18 2C18 0.89543 17.1046 0 16 0C14.8954 0 14 0.89543 14 2C14 3.10457 14.8954 4 16 4Z"
                  fill="#C3C6D1"
                />
              </svg>
            </template>
            <b-dropdown-item href="#" disabled>Check for update</b-dropdown-item>
            <b-dropdown-item href="#" disabled>View information</b-dropdown-item>
            <b-dropdown-divider />
            <b-dropdown-item variant="danger" href="#" disabled>Stop Bitcoin Core</b-dropdown-item>
          </b-dropdown>
        </div>
      </div>
    </div>
    <b-row class="row-eq-height">
      <b-col col cols="12" md="6" xl="4">
        <bitcoin-wallet></bitcoin-wallet>
      </b-col>
      <b-col col cols="12" md="6" xl="4">
        <card-widget
          header="Blockchain"
          :hasMenu="true"
          :loading="syncPercent !== 100 || blocks.length === 0"
        >
          <template v-slot:menu>
            <b-dropdown-item variant="danger" href="#" disabled>Resync Blockchain</b-dropdown-item>
          </template>
          <div class>
            <div class="px-3 px-lg-4 mb-3">
              <div class="w-100 d-flex justify-content-between mb-2">
                <span class="align-self-end">Synchronized</span>
                <h3 class="font-weight-normal mb-0">
                  <span v-if="syncPercent !== -1">
                    {{ syncPercent >= 99.99 ? 100 : syncPercent }}
                    <small class>%</small>
                  </span>

                  <span
                    class="loading-placeholder loading-placeholder-lg d-block"
                    style="width: 6rem;"
                    v-else
                  ></span>
                </h3>
              </div>
              <b-progress
                :value="Math.round(syncPercent)"
                class="mb-1"
                variant="success"
                :style="{ height: '4px' }"
                animated
                striped
              ></b-progress>
              <small class="text-muted d-block text-right" v-if="currentBlock < blockHeight - 1">
                {{ currentBlock.toLocaleString() }} of
                {{ blockHeight.toLocaleString() }} blocks
              </small>
            </div>
            <!-- low storage mode  -->
            <!-- <div class="d-flex w-100 justify-content-between px-3 px-lg-4 mb-4">
              <div>
                <span class="d-block">Low Storage Mode</span>
                <small class="text-muted d-block">Discard old blocks</small>
              </div>
              <toggle-switch class="align-self-center"></toggle-switch>
            </div>-->
            <p class="px-3 px-lg-4 mb-3">Latest Blocks</p>
            <blockchain :numBlocks="3"></blockchain>
            <div class="px-3 px-lg-4 py-2"></div>
          </div>
        </card-widget>
      </b-col>
      <b-col col cols="12" xl="4">
        <card-widget header="Network">
          <div class>
            <div class="px-3 px-lg-4 pb-2">
              <b-row>
                <!-- <b-col col cols="6" md="3" xl="6" v-for="stat in stats" :key="stat.title">
                  <stat
                    :title="stat.title"
                    :value="stat.value"
                    :suffix="stat.suffix"
                    :change="{
                      value: stat.change.value,
                      suffix: stat.change.suffix
                    }"
                  ></stat>
                </b-col>-->
                <b-col col cols="6" md="3" xl="6">
                  <stat title="Connections" :value="stats.peers" suffix="Peers" showNumericChange></stat>
                </b-col>
                <b-col col cols="6" md="3" xl="6">
                  <stat
                    title="Mempool"
                    :value="abbreviateSize(stats.mempool)[0]"
                    :suffix="abbreviateSize(stats.mempool)[1]"
                    showPercentChange
                  ></stat>
                </b-col>
                <b-col col cols="6" md="3" xl="6">
                  <stat
                    title="Hashrate"
                    :value="abbreviateHashRate(stats.hashrate)[0]"
                    :suffix="abbreviateHashRate(stats.hashrate)[1]"
                    showPercentChange
                  ></stat>
                </b-col>
                <b-col col cols="6" md="3" xl="6">
                  <stat
                    title="Blockchain Size"
                    :value="abbreviateSize(stats.blockchainSize)[0]"
                    :suffix="abbreviateSize(stats.blockchainSize)[1]"
                    showPercentChange
                  ></stat>
                </b-col>
              </b-row>
            </div>
          </div>
        </card-widget>
      </b-col>
    </b-row>
  </div>
</template>

<script>
// import Vue from "vue";
import { mapState } from "vuex";

import CardWidget from "@/components/CardWidget";
import Blockchain from "@/components/Blockchain";
// import ToggleSwitch from "@/components/ToggleSwitch";
import Stat from "@/components/Utility/Stat";
import BitcoinWallet from "@/components/BitcoinWallet";

export default {
  data() {
    return {};
  },
  computed: {
    ...mapState({
      syncPercent: state => state.bitcoin.percent,
      blocks: state => state.bitcoin.blocks,
      version: state => state.bitcoin.version,
      currentBlock: state => state.bitcoin.currentBlock,
      blockHeight: state => state.bitcoin.blockHeight,
      stats: state => state.bitcoin.stats
    }),
    isDarkMode() {
      return this.$store.getters.isDarkMode;
    }
  },
  methods: {
    random(min, max) {
      return Math.floor(Math.random() * (max - min + 1)) + min;
    },
    abbreviateHashRate(n) {
      if (n < 1e3) return [Number(n.toFixed(1)), "H/s"];
      if (n >= 1e3 && n < 1e6) return [Number((n / 1e3).toFixed(1)), "kH/s"];
      if (n >= 1e6 && n < 1e9) return [Number((n / 1e6).toFixed(1)), "MH/s"];
      if (n >= 1e9 && n < 1e12) return [Number((n / 1e9).toFixed(1)), "GH/s"];
      if (n >= 1e12 && n < 1e15) return [Number((n / 1e12).toFixed(1)), "TH/s"];
      if (n >= 1e15 && n < 1e18) return [Number((n / 1e15).toFixed(1)), "PH/s"];
      if (n >= 1e18 && n < 1e21) return [Number((n / 1e18).toFixed(1)), "EH/s"];
      if (n >= 1e21) return [Number(+(n / 1e21).toFixed(1)), "ZH/s"];
    },
    abbreviateSize(n) {
      if (n < 1e3) return [Number(n.toFixed(1)), "Bytes"];
      if (n >= 1e3 && n < 1e6) return [Number((n / 1e3).toFixed(1)), "KB"];
      if (n >= 1e6 && n < 1e9) return [Number((n / 1e6).toFixed(1)), "MB"];
      if (n >= 1e9 && n < 1e12) return [Number((n / 1e9).toFixed(1)), "GB"];
      if (n >= 1e12 && n < 1e15) return [Number((n / 1e12).toFixed(1)), "TB"];
      if (n >= 1e15) return [Number(+(n / 1e15).toFixed(1)), "PB"];
    },
    fetchStats() {
      this.$store.dispatch("bitcoin/getStats");
    }
  },
  created() {
    this.$store.dispatch("bitcoin/getVersion");
    this.fetchStats();
    this.interval = window.setInterval(this.fetchStats, 5000);
  },
  beforeDestroy() {
    window.clearInterval(this.interval);
  },
  components: {
    CardWidget,
    Blockchain,
    // ToggleSwitch,
    Stat,
    BitcoinWallet
  }
};
</script>

<style lang="scss" scoped></style>
