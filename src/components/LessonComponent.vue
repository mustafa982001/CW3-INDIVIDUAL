<template>
  <div>
    <div class="row">
      <div class="col-12 my-2">
        <h1>Lessons</h1>
      </div>
      <div class="col-12" v-if="!!search">
        <h1>
          Results for "{{ search || "" }}" {{ lessons.length }}
          <div class="float-right">
            <button
              @click="
                search = '';
                getLessons;
              "
              class="btn btn-primary"
            >
              Clear
            </button>
          </div>
        </h1>
      </div>
      <div
        v-for="(lesson, index) in lessons"
        :key="index"
        class="col-12 col-sm-4 col-md-3"
      >
        <div class="card mb-2" style="min-height: 300px">
          <img
            :src="`${lesson.Image}`"
            :alt="lesson.Subject"
            style="min-height: 160px"
          />
          <div class="card-body">
            <i :class="'fa' + lesson.Icon"></i>
            <h3>Subject: {{ lesson.Subject }}</h3>
            <h3>Location: {{ lesson.Location }}</h3>
            <p>Price: £{{ lesson.Price }}</p>
            <p>Spaces: {{ lesson.Spaces }}</p>
            <button
              class="btn btn-primary"
              v-on:click="$emit('addcart', lesson)"
              :disabled="!canAddToCart(lesson)"
            >
              Add To Cart
            </button>
            <span v-if="lesson.Spaces == 0">All out!</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "LessonComponent",
  props: {
    lessons: {
      type: Array,
    },
    searchString: {
      type: String,
      default: "",
    },
  },
  methods: {
    canAddToCart(lesson) {
      return lesson.Spaces > 0;
    },
  },
  mounted() {
    this.search = this.searchString;
    console.log("lessons", this.lessons);
  },
};
</script>