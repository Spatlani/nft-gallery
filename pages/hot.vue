<template>
  <section>
    <StatsHeader :title="$t('hot.title')" :subtitle="$t('hot.subtitle')" />
    <HotTable />
  </section>
</template>
<script lang="ts">
import { Vue, Component } from 'nuxt-property-decorator'

@Component<HotDashboard>({
  head() {
    const title = 'Recently KUSAMA Sales'
    const metaData = {
      title,
      type: 'profile',
      description: 'Featuring the hot NFTs on RMRK.',
      url: './hot.vue',
      image: `${this.$config.baseUrl}/k_card_series.png`,
    }
    return {
      title,
      meta: [...this.$seoMeta(metaData)],
    }
  },
})
export default class HotDashboard extends Vue {}
</script>

<style scoped lang="scss">
.rmrk-logo {
  aspect-ratio: 127 / 42;
}
</style>
