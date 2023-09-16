<template>
  <div :id="id" class="banner alignmax" :class="[ slides_text, { 'animate js_section': set_animate == true || animate == 'true', 'visible': set_visible == true } ]">
    <div class="slides" :class="{ 'has-multiple': all_slides.length > 1 }">

      <div class="slide" v-for="(slide, key) in all_slides" :key="key" :class="[
        slide.class,
        {
          'active' : slide.active == true,
          'show-image': slide.show_image == true,
          'show-text': slide.show_text == true,
          'top': slide.top == true,
        }
      ]">
        <div class="image-wrap" v-if="slide.image">
          <picture>
            <source :srcset="slide.image_mobile" media="(max-width: 640px)">
            <img :src="slide.image" :alt="slide.title" />
          </picture>
        </div>

        <div class="placeholder" v-else></div>

        <div class="text-content" :class="{ 'has-floating': slide.floating_image }">
          <div class="floating-image" v-if="slide.floating_image">
            <img :src="slide.floating_image" :alt="slide.title" loading="lazy" />
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
        <div v-for="(slide, key) in slides" :key="key" v-on:click="changeSlide(key)" class="dot" :class="{ 'active' : key == current_slide }"></div>
      </div>

      <div class="arrow prev" v-on:click="changeSlide('prev')" :class="{ 'show': controls == true }"></div>
      <div class="arrow next" v-on:click="changeSlide('next')" :class="{ 'show': controls == true }"></div>
    </div>
  </div>
</template>

<script>

export default {
  props: [
    'animate',
    'visible',
    'slides',
    'id',
  ],
  data() {
    return {
      set_animate: false,
      set_visible: false,
      all_slides: this.slides,
      controls: false,
      current_slide: 0,
      slides_text: null,
      sliding: true,
      paused: false,
      time: 0,
    }
  },
  mounted() {
    if (this.animate != 'true' && this.visible != true) {
      this.set_visible = true

      this.loadSlides()
    }
  },
  methods: {
    loadSlides() {

      var vm = this,
        header = document.getElementsByClassName('header')[0]

      vm.all_slides[0].active = true
      vm.all_slides[0].show_image = true

      if (vm.all_slides[0].class && (vm.all_slides[0].class.includes('dark') || vm.all_slides[0].class.includes('light'))) {
        header.classList.remove('banner-color')

        if (vm.all_slides[0].class.includes('dark')) {
          vm.slides_text = 'is-dark'
          if (document.body.classList.contains('homepage') && document.getElementsByClassName('header').length) {
            header.classList.add('dark-header')
          }
        } else {
          vm.slides_text = 'is-light'
          if (document.body.classList.contains('homepage') && document.getElementsByClassName('header').length) {
            header.classList.add('light-header')
          }
        }
      } else {
        if (document.getElementsByClassName('header').length) {
          header.classList.add('banner-color')
        }
        vm.slides_text = 'is-auto'
      }
      if (this.set_animate == true) {
        setTimeout(function() {
          vm.all_slides[0].show_text = true

          if (vm.all_slides.length > 1) {
            vm.controls = true
          }
        }, 300);

        setTimeout(function() {
          vm.sliding = false

          if (vm.slides.length > 1) {
            /*
            setInterval(function() {
              if(!vm.paused) {
                vm.time++;

                vm.changeSlide('next')
              }
            }, 5000)
            */
          }
        }, 600);

      } else {
        vm.all_slides[0].show_text = true
        if (vm.all_slides.length > 1) {
          vm.controls = true
        }

        vm.sliding = false
      }

    },
    changeSlide(direction) {
      var vm = this,
        header = document.getElementsByClassName('header')[0]
        new_slide = vm.current_slide,
        old_slide = vm.current_slide

      if (vm.sliding == false) {
        vm.set_animate = true

        if (direction == 'prev') {
          new_slide--
          if (new_slide < 0) {
            new_slide = vm.all_slides.length - 1
          }
        } else if (direction == 'next') {
          new_slide++
          if (new_slide >= vm.all_slides.length) {
            new_slide = 0
          }
        } else {
          new_slide = direction
        }

        if (direction != vm.current_slide) {
          vm.sliding = true
          vm.paused = true

          vm.current_slide = new_slide

          vm.all_slides[new_slide].active = true
          vm.all_slides[old_slide].show_text = false

          setTimeout(function() {
            vm.all_slides[new_slide].show_image = true
            vm.all_slides[new_slide].top = true

            if (vm.all_slides[new_slide].class && (vm.all_slides[new_slide].class.includes('dark') || vm.all_slides[new_slide].class.includes('light'))) {
              if (document.getElementsByClassName('header').length) {
                header.classList.remove('banner-color')
              }

              if (vm.all_slides[new_slide].class.includes('dark')) {
                vm.slides_text = 'is-dark'
                if (document.body.classList.contains('homepage') && document.getElementsByClassName('header').length) {
                  header.classList.add('dark-header')
                  header.classList.remove('light-header')
                }
              } else {
                vm.slides_text = 'is-light'
                if (document.body.classList.contains('homepage') && document.getElementsByClassName('header').length) {
                  header.classList.remove('dark-header')
                  header.classList.add('light-header')
                }
              }
            } else {
              if (document.getElementsByClassName('header').length) {
                header.classList.add('banner-color')
                header.classList.remove('dark-header')
                header.classList.remove('light-header')
              }
              vm.slides_text = 'is-auto'
            }
          }, 600);

          setTimeout(function() {
            vm.all_slides[old_slide].active = false
            vm.all_slides[old_slide].show_image = false
            vm.all_slides[new_slide].show_text = true
            vm.all_slides[new_slide].top = false
          }, 1200);

          setTimeout(function() {
            vm.paused = false
            vm.sliding = false
          }, 1500);
        }
      }
    }
  },
  watch: {
    visible(value) {
      var vm = this

      if (value == true) {
        this.set_animate = true
        this.set_visible = true

        vm.loadSlides()
      }
    }
  },
}
</script>

<style lang="less">
  @import 'style';
</style>