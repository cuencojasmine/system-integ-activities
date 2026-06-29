<template>
  <div class="profile-bg d-flex justify-center align-center" style="min-height: 100vh">
    <v-card class="profile-card" elevation="0" rounded="xl">

      <!-- Gradient header -->
      <div class="card-header d-flex flex-column align-center justify-end pb-6">
        <v-avatar size="110" class="avatar-ring">
          <v-img :src="user?.picture" cover />
        </v-avatar>
      </div>

      <!-- Content -->
      <v-card-text class="text-center px-10 pt-8 pb-10">
        <h2 class="user-name mb-2">{{ user?.name }}</h2>
        <p class="user-email mb-8">{{ user?.email }}</p>

        <!-- Nav buttons -->
        <div class="d-flex flex-column gap-3 mb-8">
          <v-btn
            class="nav-btn"
            rounded="xl"
            size="large"
            prepend-icon="mdi-map-outline"
            block
            @click="navigateTo('/maps')"
          >
            Maps
          </v-btn>

          <br />
          
          <v-btn
            class="nav-btn"
            rounded="xl"
            size="large"
            prepend-icon="mdi-qrcode-scan"
            block
            @click="navigateTo('/qr-scanner')"
          >
            QR Scanner
          </v-btn>

          <br />
          
          <v-btn
            class="nav-btn"
            rounded="xl"
            size="large"
            prepend-icon="mdi-weather-rainy"
            block
            @click="navigateTo('/weather')"
          >
            Weather
          </v-btn>
        </div>

        <v-divider class="mb-6" />

        <v-btn
          class="logout-btn"
          variant="tonal"
          rounded="xl"
          size="large"
          block
          @click="logout"
          prepend-icon="mdi-logout"
        >
          Sign Out
        </v-btn>
      </v-card-text>

    </v-card>
  </div>
</template>

<script lang="ts" setup>

definePageMeta({
  middleware: 'auth'
})

const user = ref<any>(null)

onMounted(() => {
  const savedUser = localStorage.getItem('google_user')

  if (!savedUser) {
    navigateTo('/login')
    return
  }

  user.value = JSON.parse(savedUser)
})

const logout = () => {
  localStorage.removeItem('google_user')
  localStorage.removeItem('google_token')
  window.google?.accounts.id.disableAutoSelect()
  navigateTo('/login')
}

</script>

<style scoped>
.profile-bg {
  background: #F0FDF8;
}

.profile-card {
  width: 100%;
  max-width: 460px;
  background: #ffffff;
  border: 1px solid #c8e6d8;
  overflow: hidden;
}

.card-header {
  height: 190px;
  background: linear-gradient(135deg, #0F2D40 0%, #2B5748 100%);
  position: relative;
}

.card-header::after {
  content: '';
  position: absolute;
  bottom: -30px;
  left: 0;
  right: 0;
  height: 60px;
  background: #ffffff;
  border-radius: 50% 50% 0 0 / 100% 100% 0 0;
}

.avatar-ring {
  position: relative;
  z-index: 1;
  border: 4px solid #ffffff;
  box-shadow: 0 6px 24px rgba(43, 87, 72, 0.4);
  margin-bottom: -55px;
}

.user-name {
  font-size: 1.45rem;
  font-weight: 700;
  color: #0F2D40;
  letter-spacing: -0.01em;
}

.user-email {
  font-size: 0.9rem;
  color: #2B5748;
  font-weight: 500;
}

.nav-btn {
  background: linear-gradient(135deg, #0F2D40, #2B5748) !important;
  color: #ffffff !important;
  font-weight: 600;
  letter-spacing: 0.01em;
}

.logout-btn {
  background-color: #F0FDF8 !important;
  color: #0F2D40 !important;
  font-weight: 600;
  letter-spacing: 0.01em;
  transition: background 0.2s;
}

.logout-btn:hover {
  background-color: #c8e6d8 !important;
}
</style>