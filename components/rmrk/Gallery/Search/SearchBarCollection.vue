<template>
  <div class="content field-group-container">
    <b-field grouped group-multiline>
      <Sort
        class="control"
        :value="sortBy"
        @input="updateSortBy"
        :sortOption="sortOption" />
      <div
        class="is-flex layout-search"
        :class="{
          'is-flex-grow-1 ': !hideSearch,
        }">
        <b-field expanded class="control" v-if="!hideSearch">
          <b-input
            placeholder="Search..."
            type="search"
            v-model="searchQuery"
            icon="search"
            expanded
            class="input-search">
          </b-input>
        </b-field>
        <BasicSwitch
          class="is-flex control mb-5"
          v-model="vListed"
          :label="!replaceBuyNowWithYolo ? 'sort.listed' : 'YOLO'"
          size="is-medium"
          labelColor="is-success"
          :disabled="disableToggle"
          :message="$t('tooltip.buy')" />
        <BasicSwitch
          v-if="showOwnerSwitch"
          class="is-flex control mb-5"
          v-model="vOwned"
          :label="'sort.own'"
          size="is-medium"
          labelColor="is-success"
          :message="$t('tooltip.own')" />
        <slot />
      </div>
    </b-field>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Emit, mixins } from 'nuxt-property-decorator'
import { Debounce } from 'vue-debounce-decorator'
import { exist } from './exist'
import KeyboardEventsMixin from '~/utils/mixins/keyboardEventsMixin'

@Component({
  components: {
    Sort: () => import('./SearchSortDropdown.vue'),
    TypeTagInput: () => import('./TypeTagInput.vue'),
    Pagination: () => import('@/components/rmrk/Gallery/Pagination.vue'),
    BasicSwitch: () => import('@/components/shared/form/BasicSwitch.vue'),
  },
})
export default class CollectionSearchBar extends mixins(KeyboardEventsMixin) {
  @Prop(String) public search!: string
  @Prop(String) public type!: string
  @Prop({ type: String, default: 'blockNumber_DESC' }) public sortBy!: string
  @Prop({ type: Boolean, default: false }) public listed!: boolean
  @Prop({ type: Boolean, default: false }) public owned!: boolean
  @Prop(Boolean) public disableToggle!: boolean
  @Prop(Boolean) public hideSearch!: boolean
  @Prop(Boolean) public showOwnerSwitch!: boolean
  @Prop(Array) public sortOption?: string[]
  protected isVisible = false

  public mounted(): void {
    exist(this.$route.query.search, this.updateSearch)
    exist(this.$route.query.type, this.updateType)
    exist(this.$route.query.sort, this.updateSortBy)
    exist(this.$route.query.listed, this.updateListed)
    exist(this.$route.query.owned, this.updateOwned)
  }

  public created() {
    this.initKeyboardEventHandler({
      f: this.bindFilterEvents,
    })
  }

  private bindFilterEvents(event) {
    switch (event.key) {
      case 'b':
        this.updateListed(!this.vListed)
        break
      case 'n':
        this.updateSortBy(this.sortOption?.[0] || '')
        break
      case 'o':
        this.updateSortBy(this.sortOption?.[1] || '')
        break
      case 'a':
        this.updateSortBy(this.sortOption?.[2] || '')
        break
      case 'e':
        this.updateSortBy(this.sortOption?.[3] || '')
        break
    }
  }

  get vListed(): boolean {
    return this.listed
  }

  set vListed(listed: boolean) {
    this.updateListed(listed)
  }

  get vOwned(): boolean {
    return this.owned
  }

  set vOwned(owned: boolean) {
    this.updateOwned(owned)
  }

  get searchQuery(): string {
    return this.search
  }

  set searchQuery(value: string) {
    this.updateSearch(value)
  }

  get typeQuery(): string {
    return this.type
  }

  set typeQuery(value: string) {
    this.updateType(value)
  }

  get replaceBuyNowWithYolo(): boolean {
    return this.$store.getters['preferences/getReplaceBuyNowWithYolo']
  }

  @Emit('update:listed')
  @Debounce(50)
  updateListed(value: string | boolean): boolean {
    const v = String(value)
    this.replaceUrl(v, 'listed')
    return v === 'true'
  }

  @Emit('update:owned')
  @Debounce(50)
  updateOwned(value: string | boolean): boolean {
    const v = value ? String(value) : ''
    this.replaceUrl(v, 'owned')
    return Boolean(v)
  }

  @Emit('update:type')
  @Debounce(50)
  updateType(value: string): string {
    this.replaceUrl(value, 'type')
    return value
  }

  @Emit('update:sortBy')
  @Debounce(400)
  updateSortBy(value: string): string {
    this.replaceUrl(value, 'sort')
    return value
  }

  @Emit('update:search')
  @Debounce(400)
  updateSearch(value: string): string {
    value !== this.searchQuery && this.replaceUrl(value)
    return value
  }

  @Debounce(100)
  replaceUrl(value: string, key = 'search'): void {
    this.$router
      .replace({
        path: String(this.$route.path),
        query: {
          ...this.$route.query,
          search: this.searchQuery || undefined,
          [key]: value,
        },
      })
      .catch(this.$consola.warn /*Navigation Duplicate err fix later */)
  }
}
</script>

<style scoped lang="scss">
@import '@/styles/variables';

.card {
  box-shadow: 0px 0px 5px 0.5px $primary;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>

<style lang="scss">
@import '@/styles/variables';

.field-group-container {
  .is-grouped-multiline {
    flex-wrap: initial !important;
    justify-content: space-between;
    .layout-search {
      @media screen and (max-width: 768px) {
        flex-wrap: wrap !important;
        gap: 10px;
      }
    }
    @media screen and (max-width: 768px) {
      flex-wrap: wrap !important;
    }
  }
  .input-search {
    input {
      border: 1px solid $primary !important;
    }
  }
}
</style>
