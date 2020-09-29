<template>
  <div class="carousel">
    <div class="carousel__img-area">
      <template v-for="name of names">
        <transition
          name="slide"
          :key="name">
          <template v-if="current === name">
            <div class="carousel__img-wrp">
              <slot :name="name" />
            </div>
          </template>
        </transition>
      </template>

      <template v-if="!isFirstSelected">
        <button
          class="
            carousel__side-btn
            carousel__side-btn--left
          "
          @click="prev"
        >
          <span class="carousel__side-btn-content">
            <slot name="next-button">
              <i class="mdi mdi-chevron-left" />
            </slot>
          </span>
        </button>
      </template>

      <template v-if="!isLastSelected">
        <button
          class="
            carousel__side-btn
            carousel__side-btn--right
          "
          @click="next"
        >
          <span class="carousel__side-btn-content">
            <slot name="next-button">
              <i class="mdi mdi-chevron-right" />
            </slot>
          </span>
        </button>
      </template>
    </div>

    <div class="carousel__actions">
      <template v-for="name of names">
        <button
          class="carousel__actions__button"
          :key="name"
          @click="current = name">
          <slot name="page-button">
            <span
              :class="{
                'carousel__default-btn': true,
                'carousel__default-btn--selected': name === current,
              }"
            >
              <i class="mdi mdi-circle" />
            </span>
          </slot>
        </button>
      </template>
    </div>
  </div>
</template>

<script>
export default {
  data: _ => ({
    current: null,
  }),
  computed: {
    names () {
      return Object
        .keys(this.$slots)
        .filter(s => s !== 'page-button')
        .filter(s => s !== 'next-button')
        .filter(s => s !== 'prev-button')
    },
    isFirstSelected () {
      return this.current === this.names[0]
    },
    isLastSelected () {
      return this.current === this.names[this.names.length - 1]
    },
  },
  created () {
    this.current = this.names[0]
  },
  methods: {
    next () {
      const currentIdx = this.names.findIndex(n => n === this.current)
      this.current = this.names[currentIdx + 1]
    },
    prev () {
      const currentIdx = this.names.findIndex(n => n === this.current)
      this.current = this.names[currentIdx - 1]
    },
  },
}
</script>

<style lang="scss">
@import "../../styles/all";

.carousel {
  display: grid;
  gap: 1rem;

  &__img-area {
    overflow: hidden;
    padding-top: 100%;
    position: relative;
  }

  &__img-wrp {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    text-align: center;

    img {
      height: 100%;
    }
  }

  &__side-btn {
    position: absolute;
    top: 0;
    height: 100%;
    cursor: pointer;
    background: transparent;
    width: 20%;
    border: none;

    &:focus {
      outline: none;
    }

    &--right {
      right: 0;
    }

    &--left {
      left: 0;
    }

    i.mdi {
      font-size: 10rem;
      color: var(--ui-col-primary);
      opacity: 0;
      transition: 0.25s;
    }

    &:hover {
      i.mdi {
        opacity: 0.9;
      }
    }
  }

  &__actions {
    text-align: center;
    &__button {
      background-color: transparent;
      border: none;
      &:focus {
        outline: none;
      }
    }
  }

  &__default-btn {
    font-size: 1rem;
    color: var(--ui-col-secondary);
    transition: 0.25s;

    &--selected {
      color: var(--ui-col-primary);
    }
  }
}

.slide-leave-active,
.slide-enter-active {
  transition: 0.5s linear;
}

.slide-enter {
  transform: translateX(150%);
}

.slide-leave-to {
  transform: translateX(-150%);
}
</style>
