<template>
  <div class="d-flex mx-auto justify-center align-center" style="height: 95vh">
    <v-card class="pa-8 text-center" width="500" elevation="4" rounded="xl">
      <p class="text-center text-headline-large font-weight-bold">LOGIN</p>
      <p class="text-center">Welcome back, please login</p>

      <v-form>
        <v-text-field label="Email" variant="solo-filled" prepend-inner-icon="mdi-email" flat>
        </v-text-field>
        <v-text-field label="Password" type="password" prepend-inner-icon="mdi-lock" variant="solo-filled" flat>
        </v-text-field>
      </v-form>

      <v-btn block variant="tonal" color="primary" >Sign in</v-btn>
      <div class="mt-8">OR</div>

      <v-btn class="my-8" block variant = "elevated" color="red" prepend-icon="mdi-google">
        Sign in with Google
      </v-btn>

    </v-card>

  </div>
</template>

<script lang="ts" setup>

</script>

<style></style>