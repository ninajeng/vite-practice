<script>
import { mapActions, mapState } from 'pinia';
import ProductCard from '@/components/user/ProductCard.vue';
import ProductModal from '@/components/user/ProductModal.vue';
import productStore from '@/stores/productStore';
import navStyleStore from '@/stores/navStyleStore';
import CheckLogin from '@/mixins/user/CheckLogin.vue';
import ToastMessage from '@/mixins/ToastMessage.vue';

export default {
  data() {
    return {
      products: [],
      categories: [],
      navTargets: [],
      filter: '',
      filterProducts: [],
      categoryHeight: 135,
      isLoading: false,
    };
  },
  mixins: [CheckLogin, ToastMessage],
  components: {
    ProductCard,
    ProductModal,
  },
  computed: {
    ...mapState(productStore, ['productList']),
  },
  watch: {
    filter(newValue) {
      if (newValue) {
        window.removeEventListener('scroll', this.setNavButton);
        const activeBtn = document.querySelector('.nav-link.active');
        if (activeBtn) {
          activeBtn.classList.remove('active');
        }
        this.filterProducts = this.productList.filter(
          (item) => item.title.indexOf(this.filter) > -1,
        );

        window.scroll({
          top: this.categoryHeight,
          behavior: 'instant',
        });
      } else {
        window.addEventListener('scroll', this.setNavButton);
        const positionY = window.scrollY === 0 ? 1 : window.scrollY - 1;
        window.scroll({
          top: positionY,
          behavior: 'instant',
        });
      }
    },
  },
  methods: {
    init() {
      this.getData();
    },
    async getData() {
      this.isLoading = true;
      const resError = await this.getProducts();
      if (resError) {
        this.isLoading = false;
        this.toastShow('error', this.$errorMessage);
        return;
      }
      this.categories.forEach((category) => {
        this.products[category] = [];
      });
      this.productList.forEach((product) => {
        this.products[product.category].push(product);
      });
      this.initProductModal();
      window.addEventListener('scroll', this.setNavShadow);
      window.addEventListener('scroll', this.setNavButton);
      window.addEventListener('resize', this.resizeWindow);
      this.isLoading = false;
    },
    initProductModal() {
      const productId = this.$route.params.id;
      let index = -1;
      if (productId) {
        index = this.productList.findIndex((item) => item.id === productId);
      }
      if (index > -1) {
        this.setProductData(this.productList[index]);
      }
    },
    resizeWindow() {
      this.navTargets = [];
      this.setCategory();
      this.setNavButton();
    },
    setCategory() {
      [...document.querySelectorAll('.categoryTitle')].forEach((e) => {
        this.navTargets.push(e.offsetTop);
      });
    },
    setNavShadow() {
      const scrollPosition = document.documentElement.scrollTop;
      const nav = document.querySelector('.navList');
      const navPositionY = nav.offsetTop;
      if (scrollPosition >= navPositionY - 120) {
        nav.style['box-shadow'] = '0 0.14rem 0.2rem rgba(0, 0, 0, 0.075)';
      } else {
        nav.style['box-shadow'] = 'none';
      }
    },
    setNavButton() {
      if (!this.navTargets.length) {
        this.setCategory();
      }
      const scrollPosition = document.documentElement.scrollTop;
      const activeIndex = parseInt(document.querySelector('.nav-link.active')?.href.split('#')[1].slice(-1), 10) || -1;
      const positionIndex = this.navTargets.findIndex((y, key) => {
        const tolerance = 100;
        const min = key === 0 ? 0 : y - this.categoryHeight - tolerance;
        const max = this.navTargets[key + 1] - this.categoryHeight - tolerance
          || document.documentElement.offsetHeight;
        return scrollPosition < max && scrollPosition >= min;
      });
      if (activeIndex !== positionIndex) {
        const activeBtn = document.querySelector('.nav-link.active');
        if (activeBtn) {
          activeBtn.classList.remove('active');
        }
        const targetBtn = document.querySelector(`a[href*=category${positionIndex}]`);
        const buttonsDiv = this.$refs.categoryNav;
        buttonsDiv.scroll({
          left: targetBtn.offsetLeft - buttonsDiv.offsetLeft,
          behavior: 'smooth',
        });
        targetBtn.classList.add('active');
      }
    },
    scrollEnd() {
      const buttonsDiv = this.$refs.categoryNav;
      const positionX = buttonsDiv.offsetWidth;
      buttonsDiv.scroll({
        left: positionX,
        behavior: 'smooth',
      });
    },
    scroll(e) {
      if (!this.navTargets.length) {
        this.setCategory();
      }
      const index = e.target.href.split('#')[1].slice(-1);
      const positionY = this.navTargets[index] - this.categoryHeight;

      if (this.filter) {
        this.filter = '';
        document.querySelector('.productDiv').style.minHeight = `${this.navTargets[index]}px`;
      }

      window.scroll({
        top: positionY,
        behavior: 'instant',
      });
    },
    ...mapActions(productStore, ['getProducts', 'setProductData']),
    ...mapActions(navStyleStore, ['setShadow']),
  },
  mounted() {
    this.setShadow(true);
    this.categories = this.$productTemplate.categories.map((item) => item.name);
  },
  beforeUnmount() {
    this.setShadow(false);
    window.removeEventListener('scroll', this.setNavButton);
    window.removeEventListener('scroll', this.setNavShadow);
  },
};
</script>

<template>
  <LoadingView :active="isLoading"/>
  <ProductModal/>
  <img src="https://storage.googleapis.com/vue-course-api.appspot.com/juiceoasis/1711016342192.jpg?GoogleAccessId=firebase-adminsdk-zzty7%40vue-course-api.iam.gserviceaccount.com&Expires=1742169600&Signature=qc98CuXlk6ZyibBVHszlUGLIR18B%2FMfD3FI8EthuOoCmYQFDhGl1R2%2FzANZI162RV4gaa2eG9db5DJ5ifov9mTtCxV6WkP2F2GnLCWCgZl4pU%2FxfxMJgDMaOa8xQhpLKZueAtEawaXz8vWOztowKlwJzAPJW8Ou2LyS1fiFQxGixV%2Ftj1O2iBbKgMZFY6Xfwdhwr1s0YtU%2FDxRi%2BxE8zOnBUsbXBq1K03VDx0Wd8ilP0xhyzrAzHNxUOlWEanMSUjUOJXsg2qQ1nXEogjmCHUKDgxcmidMGvmCoK7Z5jd9eEeHrDAQR652P169dRpn1LARd5UlfXcaJu%2FzseY0In6g%3D%3D" alt="菜單頁背景圖" class="bgImage"
    style="width: 100%; object-fit: cover; object-position: center center;">
  <nav class="container" style="--bs-breadcrumb-divider: '>'; margin-bottom: -0.5rem;">
    <ol class="breadcrumb mt-4">
      <li class="breadcrumb-item">
        <router-link class="link-offset-2 link-underline link-underline-opacity-75"
          :to="{name: 'home'}">首頁</router-link>
      </li>
      <li class="breadcrumb-item active">菜單</li>
    </ol>
  </nav>
  <div class="sticky-top bg-white pt-3 pt-lg-1 navList" style="top: 55px;">
    <div class="container">
      <ul class="nav nav-pills flex-nowrap overflow-x-auto text-nowrap mb-2" ref="categoryNav">
        <li class="nav-item bg-light" v-for="(category, key) in categories" :key="'btn' + key">
          <a class="nav-link" :class="{'active': key === 0 }"
            :href="`#category${key}`" @click.prevent="scroll">{{ category }}</a>
        </li>
        <li class="nav-item">
          <button type="button" :class="{'active': filter}"
            class="btn border-0 d-flex align-items-center
              position-relative py-1 h-100 searchButton">
            <input type="text" class="form-control form-control-sm"
              style="width: 200px; padding-left: 2rem; padding-right: 2rem;"
              placeholder="請輸入關鍵字" v-model.trim="filter" @input="scrollEnd">
            <i class="bi bi-search ps-2 position-absolute text-muted"></i>
            <a href="#" class="link-gray position-absolute end-0" :class="{'d-none': !filter}"
              style="margin-right: 1.25rem;" @click.prevent="filter = ''">
              <i class="bi bi-x-circle-fill"></i>
            </a>
          </button>
        </li>
      </ul>
    </div>
  </div>
  <div class="productDiv container">
    <div class="pb-3" v-if="!filter">
      <div class="row"
        v-for="(category, categoryIndex) in categories" :key="'category' + categoryIndex">
        <p class="mt-4 mb-2 text-primary categoryTitle" :id="`category${categoryIndex}`">
          {{ category }}
        </p>
        <div class="mb-3 col-md-6" v-for="productInfo in products[category]" :key="productInfo.id">
          <ProductCard :product-info="productInfo"/>
        </div>
      </div>
    </div>
    <div class="container pb-3" style="min-height: 600px;" v-else>
      <div class="row" v-if="filterProducts.length">
        <p class="mt-4 mb-2 text-primary">{{ `產品搜尋 "${filter}"` }}</p>
        <div v-for="productInfo in filterProducts" :key="'filter' + productInfo.id"
          class="mb-3 col-md-6">
          <ProductCard :product-info="productInfo"/>
        </div>
      </div>
      <div v-else>
        <p class="mt-4 text-primary">{{ `產品搜尋：沒有找到與 "${filter}" 相關的產品` }}</p>
        <button type="button" class="btn btn-primary" @click="filter = ''">查看菜單</button>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.bgImage{
  height: 250px;
}
.searchButton{
  background-color: var(--bs-light);
  &:hover, &.active{
    background-color: var(--bs-primary);
    color: #FFFFFF;
  }
  input:focus{
    background-color: var(--bs-primary-bg-subtle);
  }
}
.searchInput:focus{
  background-color: var(--bs-primary-bg-subtle);
}
@media(min-width: 992px){
  .bgImage{
    height: 350px;
  }
}
</style>
