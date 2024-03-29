<script>
import { mapState, mapActions } from 'pinia';
import LogoHTML from '@/components/logoHTML.vue';
import adminAccountStore from '@/stores/adminAccountStore';
import ToastMessage from '@/mixins/ToastMessage.vue';

export default {
  data() {
    return {
      isShowMenu: true,
      isLoading: false,
    };
  },
  components: {
    LogoHTML,
  },
  mixins: [ToastMessage],
  computed: {
    ...mapState(adminAccountStore, ['hasCheckedAuth']),
  },
  methods: {
    async logout() {
      this.isLoading = true;
      const res = await this.$adminRequest.logout();
      this.setLogoutState();
      this.$cookie.delAdminCookie();
      if (res.success) {
        this.$router.push({ name: 'adminLogin' });
      } else {
        this.alertWindow.show({
          type: 'Error',
          title: res.errorMessage,
          router: {
            method: 'replace',
            path: 'adminLogin',
          },
        });
      }
      this.toastShow('success', '登出成功');
      this.isLoading = false;
    },
    ...mapActions(adminAccountStore, ['setLogoutState']),
  },
  mounted() {
    const navbarHeight = this.$refs.navbar.offsetHeight;
    this.$refs.list.style.height = `calc(100vh - ${navbarHeight}px )`;
    this.$refs.list.style.top = `${navbarHeight}px`;
  },
};
</script>

<template>
  <loading-view :active="isLoading" />
  <nav class="navbar bg-body-tertiary sticky-top" style="min-height: 55px;" ref="navbar">
    <div class="container-fluid justify-content-start">
      <button class="btn me-1" @click="isShowMenu = !isShowMenu" v-if="hasCheckedAuth">
        <i class="bi bi-list"></i>
      </button>
      <LogoHTML/>
      <p class="mb-0 h5 opacity-75">後台管理</p>
      <div class="ms-auto">
        <router-link class="btn btn-outline-dark btn-sm me-3" to="/">
          回到前台
        </router-link>
        <button class="btn btn-outline-dark btn-sm" @click="logout" v-if="hasCheckedAuth">
          登出
        </button>
      </div>
    </div>
  </nav>
  <div class="flex-fill d-flex flex-column">
    <div class="d-flex flex-fill">
      <div class="bg-light" :class="{show: isShowMenu && hasCheckedAuth}">
        <div class="list-group overflow-y-auto sticky-top" ref="list">
          <div v-if="hasCheckedAuth">
            <RouterLink class="list-group-item list-group-item-action border-0"
              to="products">
              產品管理
            </RouterLink>
            <RouterLink class="list-group-item list-group-item-action border-0"
              to="coupons">
              優惠券管理
            </RouterLink>
            <RouterLink class="list-group-item list-group-item-action border-0"
              to="orders">
              訂單管理
            </RouterLink>
          </div>
        </div>
      </div>
      <slot></slot>
    </div>
  </div>
</template>

<style scoped lang="scss">
.list-group {
  width: 0px;
  text-wrap: nowrap;
  overflow-x: hidden;
  transition: width 0.5s;
}
.show .list-group {
  width: 250px;
}
.list-group-item{
  background-color: var(--bs-primary-bg-subtle);
}
.list-group-item:hover{
  background-color: var(--bs-primary);
  color: #FFFFFF;
}
.activeLink{
  background-color: var(--bs-primary);
}
</style>
