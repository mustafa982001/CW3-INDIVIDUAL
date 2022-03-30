<template>
  <div class="row">
    <div class="col-12">
      <h1>Checkout</h1>

      <h3>Lessons in basket</h3>

      <div class="row">
        <div
          v-for="(lesson, index) in items"
          :key="index"
          class="col-12 col-sm-4 col-md-3"
        >
          <div class="card mb-2">
            <img
              :src="`${lesson.Image}`"
              :alt="lesson.Subject"
              style="min-height: 160px"
            />

            <div class="card-body">
              <h3>{{ lesson.Subject }}</h3>
              <h3>{{ lesson.Location }}</h3>
              <h3>Spaces {{ lesson.Spaces }}</h3>
              <h3>£{{ lesson.Price }}</h3>
              <button
                class="btn btn-danger"
                @click="$emit('removefromcart', lesson)"
              >
                <i class="fa fa-trash"></i>
                Delete
              </button>
            </div>
          </div>
        </div>
      </div>

      <div class="row">
        <div class="col-6">
          <p>
            <strong>First Name:</strong>
            <input class="form-control" v-model="order.firstName" />
          </p>
          <p>
            <strong>Phone Number:</strong>
            <input class="form-control" v-model="order.phoneNumber" />
          </p>
          <div v-show="!validateName || !validateNumber">
            <p>
              Error: Please make sure name is only letters and Phone is only
              numbers
            </p>
          </div>
          <h3>Order information</h3>
          <p>First name: {{ order.firstName }}</p>
          <p>Phone Number: {{ order.phoneNumber }}</p>
          <div v-show="validateNumber && validateName">
            <button class="btn btn-primary" v-on:click="$emit('placeorder')">
              Place Order
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "CheckoutComponent",
  props: {
    items: {
      type: Array,
      default: () => [],
    },
  },
  data() {
    return {
      order: {
        firstName: "",
        phoneNumber: "",
      },
    };
  },
  computed: {
    validateName() {
      return /^[a-zA-Z]+$/.test(this.order.firstName);
    },
    validateNumber() {
      return /^[0-9]+$/.test(this.order.phoneNumber);
    },
  },
};
</script>