<template>
  <div
    class="card nft-card"
    :class="{ 'is-current-owner': accountIsCurrentOwner }">
    <LinkResolver
      class="nft-card__skeleton"
      :route="route"
      :link="link"
      :param="id"
      tag="a">
      <div class="card-image">
        <span v-if="emoteCount" class="card-image__emotes">
          <b-icon icon="heart" />
          <span class="card-image__emotes__count">{{ emoteCount }}</span>
        </span>
        <BasicImage
          v-show="!animatedUrl"
          :src="image"
          :alt="title"
          customClass="gallery__image-wrapper" />

        <PreviewMediaResolver
          v-if="!image && animatedUrl"
          :src="animatedUrl"
          :metadata="metadata"
          :mimeType="type" />
        <span v-if="price > 0 && showPriceValue" class="card-image__price">
          <Money :value="price" inline />
        </span>
      </div>

      <div class="card-content">
        <span
          class="has-text-centered has-text-primary"
          :class="{ 'title is-4': largeDisplay }"
          :title="name">
          <div class="has-text-overflow-ellipsis">
            {{ nftName }}
          </div>
        </span>
      </div>
    </LinkResolver>
  </div>
</template>

<script lang="ts">
import { Component, mixins, Prop } from 'nuxt-property-decorator'
import AuthMixin from '@/utils/mixins/authMixin'
import {
  getSingleCloudflareImage,
  processSingleMetadata,
} from '@/utils/cachingStrategy'

import { NFTMetadata } from '@/components/rmrk/service/scheme'
import { getSanitizer, sanitizeIpfsUrl } from '@/components/rmrk/utils'

const components = {
  LinkResolver: () => import('@/components/shared/LinkResolver.vue'),
  Money: () => import('@/components/shared/format/Money.vue'),
  BasicImage: () => import('@/components/shared/view/BasicImage.vue'),
  PreviewMediaResolver: () =>
    import('@/components/rmrk/Media/PreviewMediaResolver.vue'),
}

@Component({ components })
export default class GalleryCard extends mixins(AuthMixin) {
  @Prop({ type: String, default: '/rmrk/gallery' })
  public route!: string
  @Prop({ type: String, default: 'rmrk/gallery' })
  public link!: string
  @Prop(String) public id!: string
  @Prop(String) public name!: string
  @Prop([String, Number]) public emoteCount!: string | number
  @Prop(String) public imageType!: string
  @Prop(String) public price!: string
  @Prop(String) public metadata!: string
  @Prop(String) public currentOwner!: string
  @Prop(Boolean) public listed!: boolean
  protected image = ''
  protected title = ''
  protected animatedUrl = ''

  protected placeholder = '/placeholder.webp'

  async fetch() {
    if (this.metadata) {
      const image = await getSingleCloudflareImage(this.metadata)
      const meta = await processSingleMetadata<NFTMetadata>(this.metadata)

      this.image = image || getSanitizer(meta.image || '')(meta.image || '')
      this.title = meta.name
      this.animatedUrl = sanitizeIpfsUrl(
        meta.animation_url || meta.mediaUri || '',
        'pinata'
      )
    }
  }

  get showPriceValue(): boolean {
    return this.listed || this.$store.getters['preferences/getShowPriceValue']
  }

  get nftName(): string {
    return this.name || this.title
  }

  get accountIsCurrentOwner(): boolean {
    return this.accountId === this.currentOwner
  }

  get largeDisplay(): boolean {
    return (
      this.$store.getters['preferences/getLayoutClass'] ===
      'is-half-desktop is-half-tablet'
    )
  }
}
</script>

<style scoped lang="scss">
@import '@/styles/variables';

.nft-card {
  position: relative;
  overflow: hidden;
  border-radius: 0px !important;

  &.is-current-owner {
    box-shadow: 0px 2px 5px 0.5px #41b883;
  }

  .has-text-overflow-ellipsis {
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
    color: #fff;
  }

  &__skeleton {
    transition: all 0.3s;

    .card-image {
      &__emotes {
        position: absolute;
        background-color: $primary-light;
        border-radius: 4px;
        padding: 3px 8px;
        color: #fff;
        top: 10px;
        right: 10px;
        font-size: 14px;
        z-index: 3;
        transition: all 0.3s;
      }

      &__price {
        position: absolute;
        background-color: $grey-darker;
        border-radius: 4px;
        padding: 3px 8px;
        color: #fff;
        bottom: 10px;
        left: 10px;
        font-size: 14px;
        z-index: 3;
        transition: all 0.3s ease;
      }
    }
  }

  .card-image__emotes__count {
    vertical-align: text-bottom;
  }

  .card-content {
    border-radius: 0;
  }

  @media screen and (min-width: 1024px) {
    .card-content {
      position: absolute;
      bottom: -45px;
      left: 0;
      width: 100%;
      background: #fff;
      opacity: 0;
    }

    .card-image img {
      transition: all 0.3s ease;
    }

    &:hover .card-content {
      bottom: 0;
      opacity: 1;
      z-index: 2;
      background: $frosted-glass-background;
      backdrop-filter: $frosted-glass-backdrop-filter;
    }

    &:hover .card-image img {
      transform: scale(1.1) translateY(-5%);
    }

    &:hover .card-image__emotes {
      top: 15px;
      right: 15px;
    }

    &:hover .card-image__price {
      bottom: 62px;
    }
  }
}
</style>

<style lang="scss">
.gallery__image-wrapper img {
  border-radius: 0px !important;
}
</style>
