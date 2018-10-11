<!-- オリジナルのSubMenuをコピペして改修-->

<template>
  <li
    :class="classes"
    @mouseenter="handleMouseenter"
    @mouseleave="handleMouseleave"
  >
    <div
      v-click-outside="onClickOutside"
      ref="reference"
      :class="[prefixCls + '-submenu-title']"
      :style="titleStyle"
      @click.stop="handleClick">
      <slot name="title" />
      <Icon
        :class="[prefixCls + '-submenu-title-icon']"
        type="ios-arrow-down"
      />
    </div>
    <collapse-transition v-if="mode === 'vertical'">
      <ul
        v-show="opened"
        :class="[prefixCls]"><slot/></ul>
    </collapse-transition>
    <transition
      v-else
      name="slide-up">
      <Drop
        v-show="opened"
        ref="drop"
        :style="dropStyle"
        placement="bottom"><ul :class="[prefixCls + '-drop-list']"><slot/></ul>
      </Drop>
    </transition>
  </li>
</template>
<script>
  import Drop from '../select/dropdown.vue';
  import Icon from '../icon/icon.vue';
  import CollapseTransition from '../base/collapse-transition';
  import { getStyle, findComponentUpward, findComponentsDownward } from '../../utils/assist';
  import Emitter from '../../mixins/emitter';
  import mixin from './mixin';

  const prefixCls = 'ivu-menu';


  export default {
    name: 'Submenu',
    components: { Icon, Drop, CollapseTransition },
    directives: {
      ClickOutside
    },
    mixins: [ Emitter, mixin ],
    props: {
      name: {
        type: [String, Number],
        required: true
      },
      disabled: {
        type: Boolean,
        default: false
      }
    },
    data () {
      return {
        prefixCls: prefixCls,
        active: false,
        opened: false,
        dropWidth: parseFloat(getStyle(this.$el, 'width'))
      };
    },
    computed: {
      classes () {
        return [
          `${prefixCls}-submenu`,
          {
            [`${prefixCls}-item-active`]: this.active && !this.hasParentSubmenu,
            [`${prefixCls}-opened`]: this.opened,
            [`${prefixCls}-submenu-disabled`]: this.disabled,
            [`${prefixCls}-submenu-has-parent-submenu`]: this.hasParentSubmenu,
            [`${prefixCls}-child-item-active`]: this.active
          }
        ];
      },
      accordion () {
        return this.menu.accordion;
      },
      dropStyle () {
        let style = {};

        if (this.dropWidth) style.minWidth = `${this.dropWidth}px`;
        return style;
      },
      titleStyle () {
        return this.hasParentSubmenu && this.mode !== 'horizontal' ? {
          paddingLeft: 43 + (this.parentSubmenuNum - 1) * 24 + 'px'
        } : {};
      }
    },
    watch: {
      mode (val) {
        if (val === 'horizontal') {
          this.$refs.drop.update();
        }
      },
      opened (val) {
        if (this.mode === 'vertical') return;
        if (val) {
          // set drop a width to fixed when menu has fixed position
          this.dropWidth = parseFloat(getStyle(this.$el, 'width'));
          this.$refs.drop.update();
        } else {
          this.$refs.drop.destroy();
        }
      }
    },
    mounted () {
      this.$on('on-menu-item-select', (name) => {
        if (this.mode === 'horizontal') {
          this.menu.updateOpenKeys(name);
          this.opened = false;
        }
        this.dispatch('Menu', 'on-menu-item-select', name);
        return true;
      });
      this.$on('on-update-active-name', (status) => {
        if (findComponentUpward(this, 'Submenu')) this.dispatch('Submenu', 'on-update-active-name', status);
        if (findComponentsDownward(this, 'Submenu')) findComponentsDownward(this, 'Submenu').forEach(item => {
          item.active = false;
        });
        this.active = status;
      });
    },
    methods: {
      // horizontalの時はmouseOverでhandlingしているがスマホ対応が不十分のため、verticalの処理を流用する
      handleMouseenter () {
        // ********************** override ****************************
        // if (this.disabled) return;
        // if (this.mode === 'vertical') return;
        //
        // clearTimeout(this.timeout);
        // this.timeout = setTimeout(() => {
        //     this.menu.updateOpenKeys(this.name);
        //     this.opened = true;
        // }, 250);
        // ********************** override done****************************
      },
      handleMouseleave () {
        // ********************** override ****************************
        // if (this.disabled) return;
        // if (this.mode === 'vertical') return;
        //
        // clearTimeout(this.timeout);
        // this.timeout = setTimeout(() => {
        //     this.menu.updateOpenKeys(this.name);
        //     this.opened = false;
        // }, 150);
        // ********************** override done****************************
      },
      handleClick () {
        if (this.disabled) return;
        // ********************** override ****************************
        // if (this.mode === 'horizontal') return;
        const opened = this.opened;
        if (this.mode === 'vertical') {
          if (this.accordion) {
            this.$parent.$children.forEach(item => {
              if (item.$options.name === 'Submenu') item.opened = false;
            });
          }
        }
        this.menu.updateOpenKeys(this.name); // これを呼ばないとMenuのtoggleが効かなくなる
        this.opened = !opened;
        // ********************** override done****************************
      },
      onClickOutside() {
        // submenu閉じる
        if (this.opened) {
          this.handleClick();
        }
      }
    }
  };
</script>
