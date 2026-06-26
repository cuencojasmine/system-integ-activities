<template>
  <div class="login-bg d-flex justify-center align-center" style="min-height: 100vh">
    <v-card class="login-card" elevation="0" rounded="xl">

      <!-- Header bar -->
      <div class="card-header d-flex flex-column align-center justify-center">
        <v-icon size="48" color="white" class="mb-3">mdi-shield-lock-outline</v-icon>
        <h1 class="header-title">Welcome Back</h1>
        <p class="header-sub">Please sign in to continue</p>
      </div>

      <!-- Form -->
      <v-card-text class="px-10 pt-8 pb-10">
        <v-form>
          <v-text-field
            label="Email"
            variant="solo-filled"
            prepend-inner-icon="mdi-email-outline"
            flat
            rounded="lg"
            class="mb-3"
            bg-color="#F0FDF8"
          />
          <v-text-field
            label="Password"
            type="password"
            prepend-inner-icon="mdi-lock-outline"
            variant="solo-filled"
            flat
            rounded="lg"
            class="mb-6"
            bg-color="#F0FDF8"
          />

          <v-btn
            block
            rounded="xl"
            size="large"
            class="signin-btn mb-6"
          >
            Sign In
          </v-btn>
        </v-form>

        <div class="divider-row mb-6">
          <v-divider />
          <span class="divider-label">OR</span>
          <v-divider />
        </div>

        <v-btn
          block
          rounded="xl"
          size="large"
          class="google-btn"
          prepend-icon="mdi-google"
          @click="loginWithGoogle"
        >
          Sign in with Google
        </v-btn>
      </v-card-text>

    </v-card>
  </div>
</template>

<script lang="ts" setup>

definePageMeta({
  middleware: 'auth'
})

const config = useRuntimeConfig()

declare global {
  interface Window {
    google: any
  }
}

const loginWithGoogle = () => {
  const client = window.google.accounts.oauth2.initTokenClient({
    client_id: config.public.googleClientId,
    scope: 'openid email profile',
    callback: async (response: any) => {
      const userInfo = await $fetch('https://www.googleapis.com/oauth2/v3/userinfo', {
        headers: {
          Authorization: `Bearer ${response.access_token}`
        }
      })

      localStorage.setItem('google_user', JSON.stringify(userInfo))
      localStorage.setItem('google_token', response.access_token)

      navigateTo('/')
    }
  })

  client.requestAccessToken()
}

</script>

<style scoped>
.login-bg {
  background: #F0FDF8;
}

.login-card {
  width: 100%;
  max-width: 460px;
  background: #ffffff;
  border: 1px solid #c8e6d8;
  overflow: hidden;
}

.card-header {
  height: 200px;
  background: linear-gradient(135deg, #0F2D40 0%, #2B5748 100%);
  padding: 0 32px;
}

.header-title {
  font-size: 1.6rem;
  font-weight: 700;
  color: #ffffff;
  letter-spacing: -0.01em;
  margin: 0;
}

.header-sub {
  font-size: 0.875rem;
  color: rgba(255, 255, 255, 0.75);
  margin: 4px 0 0;
}

.signin-btn {
  background: linear-gradient(135deg, #0F2D40, #2B5748) !important;
  color: #ffffff !important;
  font-weight: 600;
  letter-spacing: 0.01em;
}

.divider-row {
  display: flex;
  align-items: center;
  gap: 12px;
}

.divider-label {
  font-size: 0.8rem;
  color: #9e9e9e;
  white-space: nowrap;
  font-weight: 500;
}

.google-btn {
  background-color: #F0FDF8 !important;
  color: #0F2D40 !important;
  font-weight: 600;
  border: 1px solid #c8e6d8 !important;
  letter-spacing: 0.01em;
  transition: background 0.2s;
}

.google-btn:hover {
  background-color: #c8e6d8 !important;
}
</style>