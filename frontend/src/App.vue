<template>
  <div id="app">
    <Navbar 
      v-if="isLoggedIn"
      :userRole="userRole"
      @logout="logout"
    />

    <router-view />
  </div>
</template>

<script>
import Navbar from "./components/Navbar.vue";
import router from "./utils/router.js";
import { isLoggedIn, userRole, logout } from "@/store/auth.js";

export default {
  name: "App",

  components: { Navbar },
  router,

  setup() {
    return {
      isLoggedIn,
      userRole,
      logout,
    };
  },

  methods: {
    // ✔ Validate stored token with backend
    async checkAuthentication() {
      const token = localStorage.getItem("token");
      if (!token) {
        isLoggedIn.value = false;
        userRole.value = null;
        return;
      }

      try {
        const res = await fetch(`http://127.0.0.1:5000/get-claims`, {
          headers: { Authorization: "Bearer " + token },
        });

        if (res.ok) {
          const data = await res.json();
          isLoggedIn.value = true;
          userRole.value = data.claims.role;
          localStorage.setItem("userRole", data.claims.role);
        } else {
          logout();
        }
      } catch (error) {
        console.error("Auth check failed:", error);
        logout();
      }
    },
  },
  watch: {
    $route: {
      immediate: true,
      handler() {
        this.checkAuthentication();
      },
    },
  },
  async mounted() {
    await this.checkAuthentication();

    const openRoutes = ["/", "/login", "/register", "/admin/login"];

    if (!this.isLoggedIn && !openRoutes.includes(this.$route.path)) {
      this.$router.replace("/login");
    }
  },
};
</script> 