<template>
  <b-nav-item-dropdown
    class="dropdown-cart mr-25"
    menu-class="dropdown-menu-media"
    right
    @show="fetchItems"
  >
    <template #button-content>
      <feather-icon
        :badge="$store.state['app-ecommerce'].cartItemsCount"
        class="text-body"
        icon="ShoppingCartIcon"
        size="21"
      />
    </template>

    <!-- 标题 -->
    <li class="dropdown-menu-header">
      <div class="dropdown-header d-flex">
        <h4 class="notification-title mb-0 mr-auto">
          我的购物车
        </h4>
        <b-badge
          pill
          variant="light-primary"
        >
          共 {{ $store.state['app-ecommerce'].cartItemsCount }} 个商品
        </b-badge>
      </div>
    </li>

    <!-- 购物车列表 -->
    <vue-perfect-scrollbar
      v-if="items.length"
      :settings="perfectScrollbarSettings"
      class="scrollable-container media-list scroll-area"
      tagname="li"
    >
      <b-media
        v-for="item in items"
        :key="item.name"
      >
        <template #aside>
          <b-img
            :src="item.image"
            :alt="item.name"
            rounded
            width="62px"
          />
        </template>
        <feather-icon
          icon="XIcon"
          class="cart-item-remove cursor-pointer"
          @click.stop="removeItemFromCart(item.id)"
        />
        <div class="media-heading">
          <h6 class="cart-item-title">
            <b-link class="text-body">
              {{ item.name }}
            </b-link>
          </h6>
          <small class="cart-item-by">{{ item.brand }}</small>
        </div>

        <div class="cart-item-qty ml-1">
          <b-form-spinbutton
            v-model="item.qty"
            min="1"
            size="sm"
          />
        </div>

        <h5 class="cart-item-price">
          ￥ {{ item.price }}
        </h5>
      </b-media>
    </vue-perfect-scrollbar>

    <!-- 购物车页脚 -->
    <li
      v-if="items.length"
      class="dropdown-menu-footer"
    >
      <div class="d-flex justify-content-between mb-1">
        <h6 class="font-weight-bolder mb-0">
          共计：
        </h6>
        <h6 class="text-primary font-weight-bolder mb-0">
          ￥ {{ totalAmount }}
        </h6>
      </div>
      <b-button
        v-ripple.400="'rgba(255, 255, 255, 0.15)'"
        variant="primary"
        block
        :to="{ name: 'apps-e-commerce-checkout' }"
      >
        订单结算
      </b-button>
    </li>

    <p
      v-if="!items.length"
      class="m-0 p-1 text-center"
    >
      您的购物车空空如也
    </p>
  </b-nav-item-dropdown>
</template>

<script>
import {
  BNavItemDropdown, BBadge, BMedia, BLink, BImg, BFormSpinbutton, BButton,
} from 'bootstrap-vue'
import VuePerfectScrollbar from 'vue-perfect-scrollbar'
import Ripple from 'vue-ripple-directive'

export default {
  components: {
    BNavItemDropdown,
    BBadge,
    BMedia,
    BLink,
    BImg,
    BFormSpinbutton,
    VuePerfectScrollbar,
    BButton,
  },
  directives: {
    Ripple,
  },
  data() {
    return {
      items: [],
      perfectScrollbarSettings: {
        maxScrollbarLength: 60,
        wheelPropagation: false,
      },
    }
  },
  computed: {
    totalAmount() {
      let total = 0
      this.items.forEach(i => { total += i.price })
      return total
    },
  },
  methods: {
    fetchItems() {
      this.$store.dispatch('app-ecommerce/fetchCartProducts')
        .then(response => {
          this.items = response.data.products
        })
    },
    removeItemFromCart(productId) {
      this.$store.dispatch('app-ecommerce/removeProductFromCart', { productId })
        .then(() => {
          const itemIndex = this.items.findIndex(p => p.id === productId)
          this.items.splice(itemIndex, 1)

          // 更新购物车徽章计数
          this.$store.commit('app-ecommerce/UPDATE_CART_ITEMS_COUNT', this.items.length)
        })
    },
  },
}
</script>

<style lang="scss" scoped>
.dropdown-cart {
  .media {
    .media-aside {
      align-items: center;
    }
  }
}
</style>
