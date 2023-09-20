<template>
  <div :id="id" class="banner alignmax" :class="[ slidesText, { 'animate js_section': setAnimate || animate == 'true', 'visible': setVisible } ]">
    <div class="slides" :class="{ 'has-multiple': allSlides.length > 1 }">

      <div class="slide" v-for="(slide, key) in allSlides" :key="key" :class="[
        slide.class,
        {
          'active' : slide.active,
          'show-image': slide.showImage,
          'show-text': slide.showText,
          'top': slide.top,
        }
      ]">
        <div class="image-wrap" v-if="slide.image">
          <picture>
            <source :srcset="slide.imageMobile" media="(max-width: 640px)">
            <img :src="slide.image" :alt="slide.title" />
          </picture>
        </div>

        <div class="placeholder" v-else></div>

        <div class="text-content" :class="{ 'has-floating': slide.floatingImage }">
          <div class="floating-image" v-if="slide.floatingImage">
            <img :src="slide.floatingImage" :alt="slide.title" loading="lazy" />
          </div>

          <div class="text-wrap">
            <h1>{{ slide.title }}</h1>
            <h3>{{ slide.subtitle }}</h3>
            <Buttons
              :buttons="slide.buttons"
            />
          </div>
        </div>
      </div>

      <div class="controls" :class="{ 'show': controls == true }">
        <div v-for="(slide, key) in slides" :key="key" v-on:click="changeSlide(key)" class="dot" :class="{ 'active' : key == currentSlide }"></div>
      </div>

      <div class="arrow prev" v-on:click="changeSlide('prev')" :class="{ 'show': controls == true }"></div>
      <div class="arrow next" v-on:click="changeSlide('next')" :class="{ 'show': controls == true }"></div>
    </div>
  </div>
</template>

<script>

export default {
  props: ['animate', 'visible', 'slides', 'id'],
  data() {
    return {
      setAnimate: false,
      setVisible: false,
      allSlides: this.slides,
      controls: false,
      currentSlide: 0,
      slidesText: null,
      sliding: true,
      paused: false,
      time: 0,
    }
  },
  mounted() {
    if (this.animate !== 'true' && this.visible !== true) {
      this.setVisible = true
      this.loadSlides()
    }
  },
  methods: {
    loadSlides() {
      const firstSlide = this.allSlides[0]
      firstSlide.active = true
      firstSlide.showImage = true
      this.headerClass()

      if (this.setAnimate == true) {
        setTimeout(function() {
          firstSlide.showText = true
          if (this.allSlides.length > 1) {
            this.controls = true
          }
        }, 300);

        setTimeout(function() {
          this.sliding = false
          if (this.slides.length > 1) {
            // Add auto scroll
          }
        }, 600);

      } else {
        firstSlide.showText = true
        if (this.allSlides.length > 1) {
          this.controls = true
        }
        this.sliding = false
      }

    },
    changeSlide(direction) {

      if (this.sliding == false) {
        this.setAnimate = true
        let slide = this.currentSlide
        let old = this.currentSlide
        let newSlide = this.allSlides[slide]
        let oldSlide = this.allSlides[old]

        if (direction === 'prev') {
          slide--
          if (slide < 0) {
            slide = this.allSlides.length - 1
          }
        } else if (direction == 'next') {
          slide++
          if (slide >= this.allSlides.length) {
            slide = 0
          }
        } else {
          slide = direction
        }

        if (direction !== this.currentSlide) {
          this.sliding = true
          this.paused = true
          this.currentSlide = slide
          newSlide = this.allSlides[slide]
          newSlide.active = true
          oldSlide.showText = false

          var vm = this

          setTimeout(function() {
            newSlide.showImage = true
            newSlide.top = true
            vm.headerClass()
          }, 600);

          setTimeout(function() {
            oldSlide.active = false
            oldSlide.showImage = false
            newSlide.showText = true
            newSlide.top = false
          }, 1200);

          setTimeout(function() {
            vm.paused = false
            vm.sliding = false
          }, 1500);
        }
      }
    },
    headerClass() {
      const header = document.getElementsByClassName('header')[0];
      const homepage = document.body.classList.contains('homepage');
      const slide = this.allSlides[this.currentSlide];

      if (header) {
        if (slide.class && (slide.class.includes('dark') || slide.class.includes('light'))) {
          header.classList.remove('banner-color');

          if (slide.class.includes('dark')) {
            this.slidesText = 'is-dark';
            if (homepage && header) {
              header.classList.add('dark-header').remove('light-header');
            }
          } else {
            this.slidesText = 'is-light';
            if (homepage && header) {
              header.classList.remove('dark-header').add('light-header');
            }
          }
        } else {
          header.classList.add('banner-color').remove('dark-header').remove('light-header');
          this.slidesText = 'is-auto';
        }
      }

      return this.slidesText;
    },
  },
  watch: {
    visible(value) {
      if (value == true) {
        this.setAnimate = true
        this.setVisible = true
        this.loadSlides()
      }
    }
  },
}
</script>

<style lang="less">
  @import 'style';
</style>