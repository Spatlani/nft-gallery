<template>
  <b-field :label="$t(label)">
    <b-datepicker
      ref="datepicker"
      v-model="vValue"
      :expanded="expanded"
      :placeholder="placeholder">
    </b-datepicker>
    <b-button
      @click="$refs.datepicker.toggle()"
      icon-left="calendar-today"
      type="is-primary" />
  </b-field>
</template>

<script lang="ts">
import { Component, Prop, VModel, Vue } from 'nuxt-property-decorator'

@Component({})
export default class extends Vue {
  @VModel({ type: Date }) vValue!: Date
  @Prop({ type: String, required: true }) label!: string
  @Prop({ type: String, required: true }) placeholder!: string
  @Prop({ type: Boolean, default: false }) expanded!: boolean
}
</script>
