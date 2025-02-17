<template>
  <AposModal
    class="apos-area-menu--expanded"
    :modal="modal"
    modal-title="apostrophe:addContent"
    @inactive="modal.active = false"
    @show-modal="modal.showModal = true"
    @esc="close"
    @no-modal="$emit('safe-close')"
  >
    <template #main>
      <AposModalBody>
        <template #bodyMain>
          <div
            v-for="(group, groupIndex) in groups"
            :key="groupIndex"
            class="apos-widget-group"
          >
            <h2 class="apos-widget-group__label" v-if="group.label">{{ group.label }}</h2>
            <div
              :class="[
                `apos-widget-group--${group.columns}-column${
                  group.columns > 1 ? 's' : ''
                }`
              ]"
            >
              <div
                v-for="(item, itemIndex) in group.widgets"
                :key="itemIndex"
                class="apos-widget"
                @click="add(item)"
              >
                <div class="apos-widget__preview">
                  <plus-icon
                    :size="20"
                    class="apos-icon--add"
                  />
                  <img
                    v-if="item.previewImage"
                    :src="previewUrl(item)"
                    :alt="`${item.name} preview`"
                    class="apos-widget__preview-image"
                  >
                  <component
                    v-else-if="hasIcon(item)"
                    :size="25"
                    class="apos-widget__preview--icon"
                    :is="item.previewIcon || item.icon"
                  />
                </div>
                <p class="apos-widget__label">
                  {{ $t(item.label) }}
                </p>
                <p v-if="item.description" class="apos-widget__help">
                  {{ $t(item.description) }}
                </p>
              </div>
            </div>
          </div>
        </template>
      </AposModalBody>
    </template>
  </AposModal>
</template>

<script>
export default {
  name: 'AposAreaExpandedMenu',
  props: {
    options: {
      type: Object,
      required: true
    },
    index: {
      type: Number,
      default: 0
    }
  },
  emits: [ 'expanded-menu-close', 'safe-close', 'modal-result' ],
  data() {
    return {
      modal: {
        active: false,
        type: 'slide',
        origin: 'left',
        showModal: false,
        width: 'one-third'
      },
      groups: []
    };
  },
  async mounted() {
    this.modal.active = true;

    if (this.options.groups) {
      for (const item of Object.keys(this.options.groups)) {
        if (!this.isValidColumn(item.columns)) {
          console.warn(
            `apos.expanded-menu: The specified number of columns for the group ${item.label} is not between the allowed range of 1-4.`
          );
        }

        const group = this.createGroup(this.options.groups[item]);
        this.groups.push(group);
      }
    } else if (this.options.widgets) {
      if (!this.isValidColumn(this.options.columns)) {
        console.warn(
          'apos.expanded-menu: The specified number of columns for the area is not between the allowed range of 1-4.'
        );
      }

      const group = this.createGroup(this.options);
      this.groups.push(group);
    } else {
      console.warn(
        'apos.expanded-menu: No groups or widgets defined. Please, either add a groups or widgets property to your area configuration.'
      );
    }
  },
  methods: {
    isValidColumn(count) {
      return count ? +count > 1 && +count < 4 : true;
    },
    createGroup(config) {
      const group = {
        columns: +config.columns || 3,
        widgets: []
      };

      if (config.label) {
        group.label = config.label;
      }

      for (const item of Object.keys(config.widgets)) {
        group.widgets.push(apos.modules[`${item}-widget`]);
      }

      return group;
    },
    hasIcon(widget) {
      return widget.previewIcon || widget.icon;
    },
    previewUrl(widget) {
      return widget.previewImage ? apos.util.assetUrl(`/modules/${widget.name}-widget/preview.${widget.previewImage}`) : '';
    },
    close() {
      this.modal.showModal = false;
    },
    async add(item) {
      const data = {
        ...item,
        index: this.index
      };
      this.$emit('modal-result', data);
      this.modal.showModal = false;
    }
  }
};
</script>

<style lang="scss" scoped>
.apos-area-menu--expanded {
  @include type-base;
}

.apos-widget-group {
  &:not(:last-of-type) {
    margin-bottom: 30px;
  }

  .apos-widget__preview {
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 135px;
    border: 1px solid var(--a-base-7);
    border-radius: var(--a-border-radius);
    background-color: var(--a-base-10);
  }

  &--2-columns {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 15px;
  }

  &--3-columns {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 10px;
    .apos-widget__preview {
      height: 89px;
    }
  }

  &--4-columns {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 5px;
    .apos-widget__preview {
      height: 66px;
    }
  }
}

.apos-widget {
  .apos-widget__preview {
    transition: opacity 250ms ease-in-out;
    .apos-icon--add {
      z-index: $z-index-default;
      position: absolute;
      // Center in the parent element
      align-self: center;
      justify-self: center;
      // Center the child content
      display: flex;
      justify-content: center;
      align-items: center;
      width: 27px;
      height: 27px;
      border-radius: 50%;
      background-color: var(--a-primary);
      opacity: 0;
      color: var(--a-white);
    }
    &::after {
      transition: all 250ms ease-in-out;
      position: absolute;
      content: '';
      width: 100%;
      height: 100%;
      background-color: var(--a-primary);
      opacity: 0;
    }
  }
  &:hover {
    cursor: pointer;
    // stylelint-disable max-nesting-depth
    .apos-widget__preview {
      .apos-icon--add {
        opacity: 1;
      }
      &::after {
        opacity: 0.4;
      }
    }
    // stylelint-enable max-nesting-depth
  }
}

.apos-widget__preview-image {
  width: 100%;
}

.apos-widget-group__label,
.apos-widget__help {
  @include type-base;
  line-height: var(--a-line-tall);
  color: var(--a-base-4);
  text-align: left;
}
</style>
