<!-- flipDisable を追加-->

<template>
  <div
    :class="className"
    :style="styles"
    class="ivu-select-dropdown"><slot/></div>
</template>
<script>
    import Vue from 'vue';
    const isServer = Vue.prototype.$isServer;
    import { getStyle } from '../../utils/assist';
    const Popper = isServer ? function() {} : require('popper.js/dist/umd/popper.js');  // eslint-disable-line

    export default {
        name: 'Drop',
        props: {
            placement: {
                type: String,
                default: 'bottom-start'
            },
            className: {  // eslint-disable-line
                type: String,
            },

          // ******************** 追加 *********************
          flipDisable: {
              type: Boolean,
              default: false
            }
          // ******************** 追加 *********************
        },
        data () {
            return {
                popper: null,
                width: '',
                popperStatus: false
            };
        },
        computed: {
            styles () {
                let style = {};
                if (this.width) style.minWidth = `${this.width}px`;
                return style;
            }
        },
        created () {
            this.$on('on-update-popper', this.update);
            this.$on('on-destroy-popper', this.destroy);
        },
        beforeDestroy () {
            if (this.popper) {
                this.popper.destroy();
            }
        },
        methods: {
            update () {
                if (isServer) return;
                if (this.popper) {
                    this.$nextTick(() => {
                        this.popper.update();
                        this.popperStatus = true;
                    });
                } else {
                    this.$nextTick(() => {
                        this.popper = new Popper(this.$parent.$refs.reference, this.$el, {
                            placement: this.placement,
                             modifiers: {
                                computeStyle:{
                                    gpuAcceleration: false
                                },
                                preventOverflow :{
                                    boundariesElement: 'window'
                                },
                               // ******************** 追加 *********************
                               flip: {
                                  enabled: !this.flipDisable
                               }
                               // ******************** 追加 end *********************
                            },
                            onCreate:()=>{
                                this.resetTransformOrigin();
                                this.$nextTick(this.popper.update());
                            },
                            onUpdate:()=>{
                                this.resetTransformOrigin();
                            }
                        });
                    });
                }
                // set a height for parent is Modal and Select's width is 100%
                if (this.$parent.$options.name === 'iSelect') {
                    this.width = parseInt(getStyle(this.$parent.$el, 'width'));
                }
            },
            destroy () {
                if (this.popper) {
                    setTimeout(() => {
                        if (this.popper && !this.popperStatus) {
                            this.popper.destroy();
                            this.popper = null;
                        }
                        this.popperStatus = false;
                    }, 300);
                }
            },
            resetTransformOrigin() {
                // 不判断，Select 会报错，不知道为什么
                if (!this.popper) return;

                let x_placement = this.popper.popper.getAttribute('x-placement');
                let placementStart = x_placement.split('-')[0];
                let placementEnd = x_placement.split('-')[1];
                const leftOrRight = x_placement === 'left' || x_placement === 'right';
                if(!leftOrRight){
                    this.popper.popper.style.transformOrigin = placementStart==='bottom' || ( placementStart !== 'top' && placementEnd === 'start') ? 'center top' : 'center bottom';
                }
            }
        }
    };
</script>
