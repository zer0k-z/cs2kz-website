<script setup lang="ts">
import { ref } from 'vue'
import { RouterView } from 'vue-router'
import Cookies from 'universal-cookie'
import { usePlayerStore } from './stores/player'
import { api } from './utils'
import TheHeader from './components/TheHeader.vue'
import { useColorMode } from '@vueuse/core'

const playerStore = usePlayerStore()

const openAlert = ref(true)

const cookies = new Cookies(null, { path: '/' })

const colorMode = useColorMode()
colorMode.value = 'dark'

playerStore.player = cookies.get('kz-player') || null

if (playerStore.player) {
  verifySession()
}

async function verifySession() {
  try {
    await api.get('/auth/web', { withCredentials: true })
    setTimeout(verifySession, 1000 * 25)
    /* eslint-disable */
  } catch (error: any) {
    /* eslint-enable */
    if (error.response && error.response.status === 401) {
      playerStore.player = null
      cookies.remove('kz-player')
    }
  }
}

function handleCloseAlert(open: boolean) {
  openAlert.value = open
}
</script>

<template>
  <UApp>
    <TheHeader />
    <div class="px-2 lg:px-10 mt-4">
      <UAlert
        v-if="openAlert"
        :description="$t('common.testingAlert')"
        color="warning"
        variant="subtle"
        close
        @update:open="handleCloseAlert"
      />
    </div>

    <Suspense>
      <RouterView />
    </Suspense>
  </UApp>
</template>
