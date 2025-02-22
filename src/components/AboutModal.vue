<template>
  <ModalDefault
    class="about-dex-modal"
    :title="$t('aboutModal.title')"
    close
    @close="resolve"
  >
    <div class="wrapper">
      <div class="info-container">
        {{ $t('aboutModal.info') }}
      </div>
      <InfoRow
        :left-text="$t('superheroDex')"
        :right-text="appVersion"
      />
      <InfoRow
        :left-text="$t('aboutModal.contracts')"
        :right-text="contractsVersion"
      />
      <template v-if="activeNetwork">
        <InfoRow
          :left-text="$t('aboutModal.network')"
          :right-text="activeNetwork.name"
        />
        <InfoRow
          :left-text="$t('aboutModal.middleware')"
          :right-text="activeNetwork.middlewareUrl"
        />
        <InfoRow
          :left-text="$t('aboutModal.backend')"
          :right-text="activeNetwork.dexBackendUrl"
        />
      </template>
      <InfoRow
        :left-text="$t('aboutModal.compiler')"
        :right-text="compilerUrl"
      />
      <InfoRow
        :left-text="$t('aboutModal.node')"
        :right-text="nodeVersion"
      />
      <InfoRow
        :left-text="$t('aboutModal.sdk')"
        :right-text="sdkVersion"
      />
      <InfoRow
        :left-text="$t('aboutModal.height')"
        :right-text="height ?? ''"
      />
    </div>
  </ModalDefault>
</template>

<script>
import { mapState, mapGetters } from 'vuex';
import sdkInfo from '@aeternity/aepp-sdk/package.json';
import contractInfo from 'dex-contracts-v2/package.json';
import ModalDefault from './ModalDefault.vue';
import appInfo from '../../package.json';
import InfoRow from './InfoRow.vue';

export default {
  components: {
    ModalDefault,
    InfoRow,
  },
  props: {
    resolve: { type: Function, required: true },
    reject: { type: Function, required: true },
  },
  data: () => ({
    appVersion: appInfo.version,
    sdkVersion: sdkInfo.version,
    contractsVersion: contractInfo.version,
    compilerUrl: process.env.VUE_APP_COMPILER_URL,
    height: null,
    nodeVersion: null,
    pollHeight: null,
  }),
  computed: {
    ...mapGetters(['activeNetwork']),
    ...mapState(['sdk']),
  },
  async mounted() {
    await this.$watchUntilTruly(() => this.$store.state.sdk);
    this.nodeVersion = this.sdk.selectedNode.version;
    this.height = await this.sdk.height();
    this.pollHeight = setInterval(async () => {
      this.height = await this.sdk.height();
    }, 20000);
  },
  beforeUnmount() {
    clearInterval(this.pollHeight);
  },
};
</script>

<style lang="scss" scoped>
@use '../styles/variables.scss';
@use '../styles/typography.scss';
@use '../styles/mixins.scss';

.about-dex-modal {
  .wrapper {
    padding: 16px;
    color: variables.$color-white;
    text-align: left;

    .info-container {
      margin: 12px 0 30px 0;
      color: variables.$color-gray2;

      @extend %face-sans-14-medium;
    }

    > .info-row:last-child {
      border-bottom: none;
    }

    @extend %face-sans-15-medium;
  }
}
</style>
