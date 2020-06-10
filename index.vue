<!--

For example:

<lib-carousel>
  <div>1</div>
  <div>2</div>
  <div>3</div>
</lib-carousel>

Should accommodate slides of any number / size. Slide widths are determined by parent module's CSS.

-->

<template>
  <div class="libCarousel">
    <div
      v-swipe="{
        left: handleSlideNext,
        right: handleSlidePrev,
      }"
      class="libCarousel-slides"
    >
      <slot></slot>
    </div>

    <button
      class="libCarousel-button libCarousel-button--prev"
      :disabled="canSlidePrev === false"
      @click="handleSlidePrev"
    >
      <i
        class="material-icons notranslate"
        aria-hidden="true"
      >keyboard_arrow_left</i>
    </button>

    <button
      class="libCarousel-button libCarousel-button--next"
      :disabled="canSlideNext === false"
      @click="handleSlideNext"
    >
      <i
        class="material-icons notranslate"
        aria-hidden="true"
      >keyboard_arrow_right</i>
    </button>
  </div>
</template>

<script>
  export default {
    name: 'lib-carousel',

    props: {
      startAt: {
        type: Number,
        default: 0,
      },
      transitionDuration: {
        type: Number,
        default: 0.2,
      },
    },

    data() {
      return {
        activeIndex: 0,
        canSlideNext: false,
        canSlidePrev: false,
        isMoving: false,
        slides: [],
      };
    },

    methods: {
      handleResize() {
        this.slideTo(this.activeIndex);
      },

      handleSlideNext() {
        if (this.canSlideNext) {
          this.slideTo(this.activeIndex + 1);
        }
      },

      handleSlidePrev() {
        if (this.canSlidePrev) {
          this.slideTo(this.activeIndex - 1);
        }
      },

      slideTo(index) {
        if (this.isMoving) {
          return;
        }

        // Could query/style the slides in `mounted`, but putting it here allows for slides to be dynamically added
        this.slides = this.$el.querySelector('.libCarousel-slides').children;
        for (const slide of this.slides) {
          slide.style.transitionDuration = `${this.transitionDuration}s`;
        }

        const targetSlide = this.slides[index];
        if (!targetSlide) {
          return;
        }
        this.activeIndex = index;

        const currentOffsetLeft = this.slides[0].getBoundingClientRect().left;
        const targetOffsetLeft = targetSlide.getBoundingClientRect().left;
        const distance = (currentOffsetLeft - targetOffsetLeft);
        this.slides[0].style.marginLeft = `${distance}px`;
        this.isMoving = true;

        const timeToAllowForTransition = (this.transitionDuration * 1000) + 10;
        setTimeout(() => {
          const container = this.$el.getBoundingClientRect();
          const newOffsetLeft = this.slides[0].getBoundingClientRect().left;
          const newOffsetRight = this.slides[this.slides.length - 1].getBoundingClientRect().right;
          this.canSlideNext = Boolean(newOffsetRight > container.right);
          this.canSlidePrev = Boolean(newOffsetLeft < container.left);
          this.isMoving = false;
        }, timeToAllowForTransition);
      },
    },

    mounted() {
      this.activeIndex = this.startAt;
      this.$nextTick(() => this.slideTo(this.activeIndex));
      window.addEventListener('resize', this.handleResize);
    },

    destroyed() {
      window.removeEventListener('resize', this.handleResize);
    },
  };
</script>

<style lang="postcss" scoped>
  .libCarousel {
    position: relative;
  }

  .libCarousel-button {
    background: #fff;
    border-radius: 50%;
    box-shadow: 0 2px 4px 0 rgba(0, 0, 0, .12);
    cursor: pointer;
    height: 48px;
    position: absolute;
    top: 50%;
    transition: color .1s, box-shadow .1s;
    user-select: none;
    width: 48px;

    &:hover,
    &:focus,
    &:hover:focus {
      box-shadow: 0 1px 3px 1px rgba(0, 0, 0, .15);
    }

    &:active {
      box-shadow: 0 2px 6px 2px rgba(0, 0, 0, .15);
    }

    &:disabled {
      animation: fadeout forwards .2s;
    }
  }

  @keyframes fadeout {
    from {
      opacity: 1;
    }

    to {
      display: none;
      opacity: 0;
    }
  }

  .libCarousel-button--next {
    right: 0;
    transform: translate(50%, -50%);
  }

  .libCarousel-button--prev {
    left: 0;
    transform: translate(-50%, -50%);
  }

  .libCarousel-slides {
    display: flex;
  }

  .libCarousel-slides > * {
    transition-property: margin;

    /* transition-duration set via JS */
  }
</style>
