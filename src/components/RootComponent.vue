<template>
  <!-- // navbar -->

  <nav class="navbar navbar-expand-lg navbar-light bg-light">
    <div class="container">
      <a class="navbar-brand">Lessons</a>
      <button
        class="navbar-toggler"
        type="button"
        data-bs-toggle="collapse"
        data-bs-target="#navbarSupportedContent"
        aria-controls="navbarSupportedContent"
        aria-expanded="false"
        aria-label="Toggle navigation"
      >
        <span class="navbar-toggler-icon"></span>
      </button>
      <div class="collapse navbar-collapse" id="navbarSupportedContent">
        <ul class="navbar-nav me-auto mb-2 mb-lg-0">
          <li class="nav-item">
            <a
              style="cursor: pointer !important"
              v-on:click="showCheckout"
              v-show="(orderItems && orderItems.length > 0) || !!cart"
              class="nav-link"
              ><span><i class="fas fa-shopping-basket"></i></span> Checkout
              {{ orderItems.length }}</a
            >
          </li>
        </ul>
        <form class="d-flex" @submit.prevent="doSearch">
          <!-- do search on key input -->
          <input
            class="form-control me-2"
            type="search"
            @input="doSearchOnInput"
            v-model="search"
            placeholder="Search"
            aria-label="Search"
          />
          <button class="btn btn-outline-primary" type="submit">Search</button>
        </form>
      </div>
    </div>
  </nav>

  <!-- // content -->
  <div class="container">
    <div class="row">
      <div class="col-12">
        <div class="ms-auto my-2">
          <button class="btn btn-primary mr-2" v-on:click="sortPriceByLow">
            <span><i class="fas fa-chevron-down"></i></span>
            SortByPriceLow
          </button>
          <button class="btn btn-primary" v-on:click="sortPriceByHigh">
            <span><i class="fas fa-chevron-up"></i></span>
            SortByPriceHigh
          </button>
        </div>
      </div>

      <!-- Lessons component -->
      <div v-if="showProduct">
        <!-- <div v-if="!loading"> -->
        <LessonComponent
          :lessons="lessons"
          :searchString="search"
          @addcart="addcart"
        ></LessonComponent>
        <!-- </div>   -->
      </div>

      <!-- Checkout component -->
      <div v-else>
        <!-- <div v-if="!loadingOrder"> -->
        <CheckoutComponent
          :items="orderItems"
          @removefromcart="removeFromCart"
          @placeorder="placeOrder"
        ></CheckoutComponent>
        <!-- </div> -->
      </div>
    </div>
  </div>
</template>

<script>
import LessonComponent from "./LessonComponent.vue";
import CheckoutComponent from "./CheckoutComponent.vue";
export default {
  name: "RootComponent",
  components: { LessonComponent, CheckoutComponent },
  data() {
    return {
      search: "",
      showProduct: true,
      lessons: [],
      orderItems: [],
      cart: [],
      baseUrl: "https://mustafa-lessons.herokuapp.com/",
      loading: false,
      loadingOrder: false,
      //baseUrl: "http://localhost:8080/"
    };
  },
  watch: {
    // fetch all items if search is empty
    search: function () {
      if (this.search == "") {
        this.getLessons();
      }
    },
  },
  computed: {
    cartItemCount() {
      return this.cart.length || "";
    },
    showCart() {
      return this.cart.length > 0;
    },
    backtoLessons() {
      return this.cart.length == 0;
    },
    filteredLessons: function () {
      return this.lessons;
      // return this.lessons ? this.lessons.filter((lesson) => {
      //     return lesson.Subject.toLowerCase().match(this.search.toLowerCase()) ||
      //         lesson.Location.toLowerCase().match(this.search.toLowerCase()) ||
      //         lesson.Price.toString().match(this.search.toString())
      // }) : this.lessons
    },
  },
  methods: {
    getLessons() {
      this.loading = true;
      fetch(`${this.baseUrl}lessons`, { method: "GET" })
        .then((res) => res.json())
        .then((lessons) => {
          this.lessons = lessons;
        })
        .catch((err) => {
          console.log("err", err);
        })
        .finally(() => (this.loading = false));
    },
    getOrders() {
      this.loadingOrder = true;
      // const orderId = localStorage.getItem("orderId");
      fetch(`${this.baseUrl}orders/${localStorage.getItem("orderId")}`, {
        method: "GET",
      })
        .then((res) => res.json())
        .then((orderItems) => {
          this.orderItems = orderItems;
          console.log("this.orderItems", this.orderItems);
        })
        .catch((err) => {
          console.log("err", err);
        })
        .finally(() => (this.loadingOrder = false));
    },
    async addcart(lesson) {
      // if no orderId, get orderId
      var orderId = localStorage.getItem("orderId");
      this.cart.push(lesson.id);
      lesson.Spaces -= 1;
      try {
        if (!orderId) {
          orderId = await fetch(`${this.baseUrl}orders`, { method: "POST" });
          orderId = await orderId.json();
          orderId = orderId.insertedId;
          await localStorage.setItem("orderId", orderId);
        }
        await fetch(`${this.baseUrl}orders/${orderId}/${lesson._id}`, {
          method: "POST",
        });
        await this.getOrders();
      } catch (error) {
        console.log("error", error);
      }
    },
    showCheckout() {
      this.showProduct = !this.showProduct;
      this.getOrders(localStorage.getItem(""));
    },
    cartCount(id) {
      let count = 0;
      for (let i = 0; i < this.cart.length; i++) {
        if (this.cart[i] === id) {
          count++;
        }
      }
      return count;
    },
    sortPriceByLow() {
      function compare(a, b) {
        if (a.Price > b.Price) return 1;
        if (a.Price < b.Price) return -1;
        return 0;
      }
      return this.lessons.sort(compare);
    },
    sortPriceByHigh() {
      function compare(a, b) {
        if (a.Price > b.Price) return -1;
        if (a.Price < b.Price) return 1;
        return 0;
      }
      return this.lessons.sort(compare);
    },
    placeOrder() {
      alert("Order submitted!!!");
      localStorage.removeItem("orderId");
      this.getLessons();
      this.getOrders();
    },
    removeFromCart(lesson) {
      console.log("lesson", lesson);
      fetch(`${this.baseUrl}orders/${lesson._id}`, { method: "DELETE" })
        .then((res) => res.json())
        .then((res) => {
          console.log("res", res);
          this.getOrders();
          this.getLessons();
        })
        .catch((err) => {
          console.log("err", err);
        });
    },
    // search on key input
    doSearchOnInput() {
      setTimeout(() => {
        this.doSearch();
        console.log(this.search);
      }, 400);
    },
    doSearch() {
      // search products
      console.log("search");
      fetch(`${this.baseUrl}search/${this.search}`, { method: "GET" })
        .then((res) => res.json())
        .then((lessons) => {
          this.lessons = lessons;
        })
        .catch((err) => {
          console.log("err", err);
        });
    },
  },
  mounted() {
    this.getLessons();
    this.getOrders();
  },
};
</script>