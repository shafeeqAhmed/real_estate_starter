<script setup>
import { useAppAbility } from '@/plugins/casl/useAppAbility'
import AuthProvider from '@/views/pages/authentication/AuthProvider.vue'
import axios from '@axios'

import { useGenerateImageVariant } from '@core/composable/useGenerateImageVariant'
import authV2LoginIllustrationBorderedDark from '@images/pages/auth-v2-login-illustration-bordered-dark.png'
import authV2LoginIllustrationBorderedLight from '@images/pages/auth-v2-login-illustration-bordered-light.png'
import authV2LoginIllustrationDark from '@images/pages/auth-v2-login-illustration-dark.png'
import authV2LoginIllustrationLight from '@images/pages/auth-v2-login-illustration-light.png'
import authV2MaskDark from '@images/pages/misc-mask-dark.png'
import authV2MaskLight from '@images/pages/misc-mask-light.png'
import { VNodeRenderer } from '@layouts/components/VNodeRenderer'
import { themeConfig } from '@themeConfig'
import {
  emailValidator,
  requiredValidator,
} from '@validators'
import { VForm } from 'vuetify/components';
import { useToast } from "vue-toastification";


const authThemeImg = useGenerateImageVariant(authV2LoginIllustrationLight, authV2LoginIllustrationDark, authV2LoginIllustrationBorderedLight, authV2LoginIllustrationBorderedDark, true)
const authThemeMask = useGenerateImageVariant(authV2MaskLight, authV2MaskDark)
const isPasswordVisible = ref(false)
const route = useRoute()
const router = useRouter()
const toast = useToast();
// const email = ref('admin@demo.com')
const email = ref()
// const password = ref('admin')
const password = ref()
const rememberMe = ref()
const loginFormRef = ref()

const ability = useAppAbility()

const errors = ref({
  email: undefined,
  password: undefined,
})



const onSubmit = () => {
  loginFormRef.value?.validate().then(({ valid: isValid }) => {
    if (isValid)
      login()
  })
}

const login = () => {
  axios.post('/api/login', {
    email: email.value,
    password: password.value,
    rememberMe: rememberMe.value
  }).then(response => {
    const { data: { data } } = response
    const { accessToken, userData } = data
    // localStorage.setItem('userAbilities', JSON.stringify(userData.ability))

    // ability.update(userAbilities)
    ability.update(userData.ability)
    localStorage.setItem('userData', JSON.stringify(userData))
    localStorage.setItem('accessToken', accessToken)

    // Redirect to `to` query if exist or redirect to index route
    router.replace(route.query.to ? String(route.query.to) : '/')
    toast.success("User has been login successfully!", {
      timeout: 3000
    });
    
  }).catch(e => {
    // const { errors: formErrors } = e.response.data
    // errors.value = formErrors
    toast.error("The provided credentials are incorrect!", {
      timeout: 3000
    });
  })
}
</script>

<template>
  <VRow
    no-gutters
    class="auth-wrapper"
  >
    <VCol
      lg="8"
      class="d-none d-lg-flex"
    >
      <div class="position-relative auth-bg rounded-lg w-100 ma-8 me-0">
        <div class="d-flex align-center justify-center w-100 h-100">
          <VImg
            max-width="505"
            :src="authThemeImg"
            class="auth-illustration mt-16 mb-2"
          />
        </div>

        <VImg
          :src="authThemeMask"
          class="auth-footer-mask"
        />
      </div>
    </VCol>

    <VCol
      cols="12"
      lg="4"
      class="d-flex align-center justify-center"
    >
      <VCard
        flat
        :max-width="500"
        class="mt-12 mt-sm-0 pa-4"
      >
        <VCardText>
          <img src="@images/img/logo.png" alt="" class="mb-3 w-75"/>

          <h5 class="text-h5 font-weight-semibold mb-1">
            Welcome to Future State 👋🏻
          </h5>
          <p class="mb-0">
            Please sign-in to your account and start the adventure
          </p>
        </VCardText>

        <VCardText>
          <VForm
            ref="loginFormRef"
            @submit.prevent="onSubmit"
          >
            <VRow>
              <!-- email -->
              <VCol cols="12">
                <VTextField
                  v-model="email"
                  label="Email"
                  type="email"
                  :rules="[requiredValidator, emailValidator]"
                  :error-messages="errors.email"
                />
              </VCol>

              <!-- password -->
              <VCol cols="12">
                <VTextField
                  v-model="password"
                  label="Password"
                  :rules="[requiredValidator]"
                  :type="isPasswordVisible ? 'text' : 'password'"
                  :error-messages="errors.password"
                  :append-inner-icon="isPasswordVisible ? 'tabler-eye-off' : 'tabler-eye'"
                  @click:append-inner="isPasswordVisible = !isPasswordVisible"
                />

                <div class="d-flex align-center flex-wrap justify-space-between mt-2 mb-4">
                  <VCheckbox
                    v-model="rememberMe"
                    label="Remember me"
                  />
                  <RouterLink
                    class="text-primary ms-2 mb-1"
                    :to="{ name: 'forgot-password'}"
                  >
                    Forgot Password?
                  </RouterLink>
                </div>

                <VBtn
                  block
                  type="submit"
                >
                  Login
                </VBtn>
              </VCol>

              <!-- create account -->
              <VCol
                cols="12"
                class="text-center"
              >
                <span>New on our platform?</span>
                <a
                  class="text-primary ms-2"
                  href="#"
                >
                  <RouterLink
                    class="text-primary ms-2"
                    :to="{ name :'register'}"
                  > Create an account</RouterLink>

                </a>
              </VCol>
              <VCol
                cols="12"
                class="d-flex align-center"
              >
                <VDivider />
                <span class="mx-4">or</span>
                <VDivider />
              </VCol>

              <!-- auth providers -->
              <VCol
                cols="12"
                class="text-center"
              >
                <AuthProvider />
              </VCol>
            </VRow>
          </VForm>
        </VCardText>
      </VCard>
    </VCol>
  </VRow>
</template>

<style lang="scss">
@use "@core-scss/template/pages/page-auth.scss";
</style>

<route lang="yaml">
meta:
  layout: blank
  action: read
  subject: Auth
  redirectIfLoggedIn: true
</route>
