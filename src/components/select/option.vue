<!-- touch.preventを削除してスマホでスクロールできるように-->

<template>
  <li
    :class="classes"
    @click.stop="select"
  ><slot>{{ showLabel }}</slot></li>
</template>
<script>
    import Emitter from '../../mixins/emitter';
    import { findComponentUpward } from '../../utils/assist';

    const prefixCls = 'ivu-select-item';

    export default {
        name: 'iOption',  // eslint-disable-line
        componentName: 'select-item',
        mixins: [ Emitter ],
        props: {
            value: {
                type: [String, Number],
                required: true
            },
            label: {  // eslint-disable-line
                type: [String, Number]
            },
            disabled: {
                type: Boolean,
                default: false
            },
            selected: {
                type: Boolean,
                default: false
            },
            isFocused: {
                type: Boolean,
                default: false
            }
        },
        data () {
            return {
                searchLabel: '',  // the slot value (textContent)
                autoComplete: false
            };
        },
        computed: {
            classes () {
                return [
                    `${prefixCls}`,
                    {
                        [`${prefixCls}-disabled`]: this.disabled,
                        [`${prefixCls}-selected`]: this.selected && !this.autoComplete,
                        [`${prefixCls}-focus`]: this.isFocused
                    }
                ];
            },
            showLabel () {
                return (this.label) ? this.label : this.value;
            },
            optionLabel(){
                return this.label || (this.$el && this.$el.textContent);
            }
        },
        mounted () {
          const Select = findComponentUpward(this, 'iSelect');
          if (Select) this.autoComplete = Select.autoComplete;
        },
        methods: {
            select () {
                if (this.disabled) return false;

                this.dispatch('iSelect', 'on-select-selected', {
                    value: this.value,
                    label: this.optionLabel,
                });
                this.$emit('on-select-selected', {
                    value: this.value,
                    label: this.optionLabel,
                });
            },
        },
    };
</script>
