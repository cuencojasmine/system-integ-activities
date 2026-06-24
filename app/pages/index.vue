<template>
  <div class="d-flex mx-auto justify-center align-center" style="height: 100vh">
    <v-card class="pa-10">
      <div class="text-center mb-6">
        <v-avatar size="90" class="mb-3 elevation-3">
          <v-img :src="user?.picture"></v-img>
        </v-avatar>
        <h2>{{ user?.name }}</h2>
        <h3>{{ user?.email }}</h3>
        <v-btn @click="logout">Logout</v-btn>
      </div>
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

<style></style>