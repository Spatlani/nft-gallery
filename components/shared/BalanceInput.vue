<template>
  <div class="arguments-wrapper">
    <b-field :label="$t(label)" class="balance">
      <b-input
        ref="balance"
        v-model="inputValue"
        type="number"
        :step="step"
        min="0"
        :expanded="expanded"
        @input="handleInput" />
      <p class="control balance">
        <b-select
          v-model="selectedUnit"
          :disabled="!calculate"
          @input="handleInput">
          <option v-for="u in units" :key="u.value" :value="u.value">
            {{ u.name }}
          </option>
        </b-select>
      </p>
    </b-field>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Emit, Ref, mixins } from 'nuxt-property-decorator'
import { units as defaultUnits } from '@/params/constants'
import { Unit } from '@/params/types'
import { Debounce } from 'vue-debounce-decorator'
import ChainMixin from '@/utils/mixins/chainMixin'

@Component
export default class BalanceInput extends mixins(ChainMixin) {
  @Prop({ type: [Number, String], default: 0 }) value!: number
  @Prop({ default: 'balance' }) public label!: string
  @Prop({ default: true }) public calculate!: boolean
  @Prop(Boolean) public expanded!: boolean
  @Prop({ default: 0.001 }) public step!: number
  protected units: Unit[] = defaultUnits
  private selectedUnit = 1

  @Ref('balance') readonly balance

  get inputValue(): number {
    return this.value
  }

  set inputValue(value: number) {
    this.handleInput(value)
  }

  public focusInput(): void {
    this.balance?.focus()
  }

  formatSelectedValue(value: number): string {
    return String(value * 10 ** this.decimals * this.selectedUnit)
  }

  get calculatedBalance() {
    return this.formatSelectedValue(this.inputValue)
  }

  protected mapper(unit: Unit) {
    if (unit.name === '-') {
      return { ...unit, name: this.unit }
    }
    return unit
  }

  public mounted() {
    this.units = defaultUnits.map(this.mapper)
  }

  @Debounce(200)
  @Emit('input')
  public handleInput(value: number) {
    return this.calculate ? this.formatSelectedValue(value) : value
  }
}
</script>
