<template>
  <div>
    <LinkResolver
      class="is-flex is-align-items-center"
      :route="`/${urlPrefix}/u`"
      :param="address"
      link="u">
      <Identity :address="address" :inline="true" :vertical-align="true" />
      <template #extra>
        <Avatar :size="24" :value="address" class="mr-2" />
      </template>
    </LinkResolver>
    <template v-if="showTwitter">
      <Identity
        :address="address"
        :show-twitter="showTwitter"
        :show-discord="showDiscord"
        :vertical-align="true"
        class="pt-2" />
    </template>
    <a
      v-if="showDotscanner"
      :href="`https://dotscanner.com/Kusama/account/${address}`"
      target="_blank"
      rel="noopener noreferrer"
      class="is-flex is-align-items-center pt-2">
      <figure class="image is-24x24 mr-2">
        <img alt="dotscanner" src="/dotscanner.svg" />
      </figure>
      {{ shortendId }}
    </a>
  </div>
</template>

<script lang="ts">
import { Component, Prop, mixins } from 'nuxt-property-decorator'
import InlineMixin from '@/utils/mixins/inlineMixin'
import PrefixMixin from '@/utils/mixins/prefixMixin'
import shortAddress from '@/utils/shortAddress'

const components = {
  Identity: () => import('@/components/shared/format/Identity.vue'),
  LinkResolver: () => import('@/components/shared/LinkResolver.vue'),
}

@Component({ components })
export default class ProfileLink extends mixins(InlineMixin, PrefixMixin) {
  @Prop() public address!: string
  @Prop(Boolean) public showTwitter!: boolean
  @Prop(Boolean) public showDiscord!: boolean
  @Prop(Boolean) public showDotscanner!: boolean
  get shortendId(): string {
    return shortAddress(this.address)
  }

  get verticalAlign(): boolean {
    return this.showTwitter
  }
}
</script>

<style scoped>
.subscan__less-margin {
  margin: auto 0.5em auto 0;
}
</style>
