<script>
import { getProductItem } from '@/api/userRequest';
import CheckLogin from '@/mixins/user/CheckLogin.vue';
import InitProductSettings from '@/mixins/user/InitProductSettings.vue';
import ToastMessage from '@/mixins/ToastMessage.vue';

export default {
  data() {
    return {
      productData: {},
      userCustom: {},
      qty: 1,
      isLoading: false,
    };
  },
  mixins: [CheckLogin, InitProductSettings, ToastMessage],
  methods: {
    init() {
      this.getData();
    },
    async getData() {
      this.isLoading = true;
      const res = await getProductItem(this.$route.params.id);
      if (res.success) {
        this.initProductSettings(res.data.product);
        this.isLoading = false;
      } else {
        this.toastShow('error', this.$errorMessage);
        this.isLoading = false;
      }
    },
    async addItemToCart() {
      this.isLoading = true;
      const resError = await this.setCustomItem(this.isUpdate);
      if (!resError) {
        this.toastShow('success', `已${this.isUpdate ? '更新' : '加入'}購物車！`);
      } else if (resError !== '未登入') {
        this.toastShow('error', this.$errorMessage);
      }
      this.isLoading = false;
    },
  },
};
</script>

<template>
  <LoadingView :active="isLoading"/>
  <div class="container py-5 mt-5" v-if="productData.title">
    <div class="d-flex flex-column flex-lg-row
      justify-content-center justify-content-lg-start">
      <div class="mb-3 me-0 me-lg-5 h-100">
        <nav style="--bs-breadcrumb-divider: '>';">
          <ol class="breadcrumb">
            <li class="breadcrumb-item">
              <router-link :to="{name: 'home'}"
                class="link-offset-2 link-underline link-underline-opacity-75">首頁</router-link>
            </li>
            <li class="breadcrumb-item">
              <router-link :to="{name: 'products'}"
                class="link-offset-2 link-underline link-underline-opacity-75">菜單</router-link>
            </li>
            <li class="breadcrumb-item active">{{ productData.title}}</li>
          </ol>
        </nav>
        <div class="text-center">
          <div class="productImage">
            <img :src="productData.imageUrl" :alt="productData.title"
              class="w-100" style="object-fit: cover; height: 400px;" v-if="productData.imageUrl">
          </div>
        </div>
      </div>
      <div class="ms-0 ms-lg-5 align-self-lg-center">
        <h4>
          {{ productData.title }}
          <span class="productBadge secondaryOutlineBadge">
            {{ productData.category }}
          </span>
        </h4>
        <p class="text-muted mb-1">{{ productData.description }}</p>
        <p class="text-muted" v-if="productData.type === 'drink'">
          {{ `成分：${productData.ingredients}` }}
        </p>

        <div class="mt-3" v-if="productData.custom && productData.type === 'drink'">
          <div class="border p-3 mb-3">
            <p class="mb-3 text-muted">請選擇甜度*</p>
            <p class="mb-0">
              <span class="me-2 mb-2 d-inline-block"
                v-for="(option, key) in productData.custom.sugar.options"
                :key="'sugar' + key">
                <input type="radio" class="btn-check" :id="option"
                  name="sugar" :value="option"
                  v-model="userCustom.sugar" >
                <label class="btn btn-light btn" :for="option">
                  {{ option }}
                </label>
              </span>
            </p>
          </div>
          <div class="border p-3 mb-3">
            <p class="mb-3 text-muted">請選擇冷熱*</p>
            <p class="mb-0">
              <span class="me-2 mb-2 d-inline-block"
                v-for="(option, key) in productData.custom.ice.options"
                :key="'ice' + key">
                <input type="radio" class="btn-check" :id="option"
                  name="ice" :value="option"
                  v-model="userCustom.ice">
                <label class="btn btn-light btn" :for="option">
                  {{ option }}
                </label>
              </span>
            </p>
          </div>
          <div class="border p-3 mb-3">
            <p class="mb-3 text-muted">請選擇尺寸*</p>
            <p class="mb-0">
              <span class="me-2 mb-2 d-inline-block"
                v-for="(option, key) in productData.custom.size.options"
                :key="'size' + key">
                <input type="radio" class="btn-check" :id="option"
                  name="size" :value="option"
                  v-model="userCustom.size">
                <label class="btn btn-light btn" :for="option" v-if="!key">
                  {{ option }}
                  <span class="fs-7">{{ `(${productData.price}元)` }}</span>
                </label>
                <label class="btn btn-light btn" :for="option" v-else>
                  {{ option }}
                  <span class="fs-7">
                    {{ `(${productData.price + productData.custom.extras.addPrice}元)` }}
                  </span>
                </label>
              </span>
            </p>
          </div>
          <div class="border p-3 mb-3" v-if="productData.custom.extras.isCustom">
            <p class="mb-3 text-muted">請選擇配料
              <span class="fs-7">{{ `(+${productData.custom.extras.addPrice}元/項)` }}</span>
            </p>
            <p class="mb-0">
              <span class="me-2 mb-2 d-inline-block"
                v-for="(option, key) in productData.custom.extras.options"
                :key="'extras' + key">
                <input type="checkbox" class="btn-check" :id="option"
                  :value="option"
                  v-model="userCustom.extras">
                <label class="btn btn-light btn" :for="option">
                  {{ option }}
                </label>
              </span>
            </p>
          </div>
        </div>
        <p>
          <span v-if="productData.origin_price === productData.price">
            {{ `NT$ ${$filters.currency(totalPrice)}` }}
          </span>
          <span class="text-danger" v-else>
            {{ `NT$ ${$filters.currency(totalPrice)}` }}
            <del class="ms-1 text-muted">
              {{ `NT$ ${$filters.currency(totalPrice +
                productData.origin_price - productData.price)}` }}
            </del>
          </span>
          <span class="text-muted fs-7">{{ ` / ${productData.unit}` }}</span>
        </p>
        <div class="d-flex flex-column flex-md-row">
          <div class="input-group w-auto mb-3 me-3">
            <button class="btn btn-primary" type="button" @click="qty -= 1" :disabled="qty === 1">
              <i class="bi bi-dash"></i>
            </button>
            <span class="input-group-text d-inline-block text-center mb-0"
              style="min-width: 70px;">{{ qty }}
            </span>
            <button class="btn btn-primary" type="button" @click="qty += 1" :disabled="qty === 20">
              <i class="bi bi-plus-lg"></i>
            </button>
          </div>
          <button type="button" class="btn btn-primary mb-3 flex-fill"
            style="min-width: 150px;" @click="addItemToCart">加入購物車</button>
        </div>
      </div>
    </div>

    <div class="py-5">
      <p>注意事項</p>
      <ul>
        <li>為維持飲品新鮮度與天然風味，水果飲品不提供熱飲。如欲品嚐熱飲請參考經典手搖系列飲品。</li>
      </ul>
      <ul>
        <li>飲品請於一小時內飲用完畢。</li>
      </ul>
      <ul>
        <li>分裝或完整水果請保存於冷藏、切片水果請於半小時內食用完畢。</li>
      </ul>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.productImage{
  max-width: 100%;
}
@media (min-width: 992px){
  .productImage{
    max-width: 500px;
  }
}
</style>
