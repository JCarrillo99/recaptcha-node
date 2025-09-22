<template>
  <div class="lsx-recaptcha">
    <!-- v2 Checkbox -->
    <div v-if="v2CheckboxKey">
      <h2>✅ reCAPTCHA v2 - Checkbox</h2>
      <div ref="checkboxContainer" class="recaptcha-container"></div>
      <button @click="executeV2Checkbox">Verificar Token</button>
      <div class="result" :class="{ success: results.v2Checkbox.success, error: !results.v2Checkbox.success }">
        {{ results.v2Checkbox.message }}
      </div>
    </div>

    <!-- v2 Badge Invisible -->
    <div v-if="v2BadgeKey">
      <h2>🏷️ reCAPTCHA v2 - Insignia Invisible</h2>
      <div ref="badgeContainer" class="recaptcha-container"></div>
      <button @click="executeV2Badge">Ejecutar v2 Badge Invisible</button>
      <div class="result" :class="{ success: results.v2Badge.success, error: !results.v2Badge.success }">
        {{ results.v2Badge.message }}
      </div>
    </div>

    <!-- v3 Score -->
    <div v-if="v3Key">
      <h2>📊 reCAPTCHA v3 - Score</h2>
      <button @click="executeV3">Probar v3 Score</button>
      <div class="result" :class="{ success: results.v3.success, error: !results.v3.success }">
        {{ results.v3.message }}
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { reactive, ref, onMounted } from 'vue'

// Declaraciones globales para TypeScript
declare global {
  interface Window {
    grecaptcha: {
      ready: (callback: () => void) => void;
      render: (container: string | HTMLElement, options: any) => number;
      execute: (siteKey: string, options: { action: string }) => Promise<string>;
      getResponse: (widgetId: number) => string;
      reset: (widgetId: number) => void;
    };
  }
}

// Configuración de keys desde variables de entorno
const v2CheckboxKey = import.meta.env.VITE_RECAPTCHA_2_CHECKBOX || ''
const v2BadgeKey = import.meta.env.VITE_RECAPTCHA_2_INSIGNIA || ''
const v3Key = import.meta.env.VITE_RECAPTCHA_3_SCORE || ''

const results = reactive({
  v2Checkbox: { success: false, message: 'Esperando...' },
  v2Badge: { success: false, message: 'Esperando...' },
  v3: { success: false, message: 'Esperando...' }
})

const checkboxContainer = ref<HTMLElement | null>(null)
const badgeContainer = ref<HTMLElement | null>(null)

let v2CheckboxId: number | null = null
let v2BadgeId: number | null = null

// Carga del script de reCAPTCHA
const loadScript = (callback: () => void) => {
  if (document.querySelector('script[src*="recaptcha/api.js"]')) {
    callback()
    return
  }
  const script = document.createElement('script')
  script.src = 'https://www.google.com/recaptcha/api.js?render=explicit'
  script.async = true
  script.defer = true
  script.onload = callback
  script.onerror = () => console.error('❌ Error cargando reCAPTCHA')
  document.head.appendChild(script)
}

// Inicializar widgets
const initWidgets = () => {
  if (!window.grecaptcha) return
  window.grecaptcha.ready(() => {
    if (v2CheckboxKey && checkboxContainer.value && !v2CheckboxId) {
      v2CheckboxId = window.grecaptcha.render(checkboxContainer.value, {
        sitekey: v2CheckboxKey,
        callback: token => handleVerify(token, 'v2_checkbox'),
        'expired-callback': () => results.v2Checkbox.message = '⏰ Token expirado',
        'error-callback': () => results.v2Checkbox.message = '❌ Error en v2 Checkbox'
      })
      console.log('✅ v2 Checkbox renderizado, ID:', v2CheckboxId)
    }
    if (v2BadgeKey && badgeContainer.value && !v2BadgeId) {
      v2BadgeId = window.grecaptcha.render(badgeContainer.value, {
        sitekey: v2BadgeKey,
        size: 'invisible',
        callback: token => handleVerify(token, 'v2_badge'),
        'expired-callback': () => results.v2Badge.message = '⏰ Token expirado',
        'error-callback': () => results.v2Badge.message = '❌ Error en v2 Badge'
      })
      console.log('✅ v2 Badge renderizado, ID:', v2BadgeId)
    }
  })
}

// Función genérica para verificar token
const handleVerify = async (token: string, version: string) => {
  console.log(`🎯 TOKEN ${version.toUpperCase()} COMPLETO:`)
  console.log(`📋 Token: ${token}`)
  console.log(`📏 Longitud: ${token.length} caracteres`)
  
  if (version === 'v2_checkbox') {
    console.log(`🔑 Site Key: ${v2CheckboxKey}`)
    results.v2Checkbox.message = '⏳ Verificando...'
  }
  if (version === 'v2_badge') {
    console.log(`🔑 Site Key: ${v2BadgeKey}`)
    results.v2Badge.message = '⏳ Verificando...'
  }
  if (version === 'v3_score') {
    console.log(`🔑 Site Key: ${v3Key}`)
    console.log(`🎬 Action: demo`)
    results.v3.message = '⏳ Verificando...'
  }

  try {
    console.log(`🔄 Verificando token ${version}...`)
    console.log(`📤 Enviando a: http://localhost:8080`)
    console.log(`📦 Payload: ${JSON.stringify({ token: token.substring(0, 20) + '...', version })}`)
    
    const response = await fetch('http://localhost:8080', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ token, version })
    })
    const data = await response.json()
    console.log(`📥 Respuesta del servidor: ${JSON.stringify(data)}`)

    if (version === 'v2_checkbox') {
      results.v2Checkbox.success = data.success
      results.v2Checkbox.message = data.success ? '✅ Verificado' : '❌ Inválido'
    }
    if (version === 'v2_badge') {
      results.v2Badge.success = data.success
      results.v2Badge.message = data.success ? '✅ Verificado' : '❌ Inválido'
    }
    if (version === 'v3_score') {
      results.v3.success = data.success
      results.v3.message = data.success ? `✅ Score: ${data.data.score}` : '❌ Inválido'
    }
  } catch (error) {
    console.error(`❌ Error verificando ${version}:`, error)
    if (version === 'v2_checkbox') results.v2Checkbox.message = '❌ Error de conexión'
    if (version === 'v2_badge') results.v2Badge.message = '❌ Error de conexión'
    if (version === 'v3_score') results.v3.message = '❌ Error de conexión'
  }
}

// Ejecutar v2 Checkbox
const executeV2Checkbox = () => {
  if (!v2CheckboxId) return
  const token = window.grecaptcha.getResponse(v2CheckboxId)
  if (!token) {
    results.v2Checkbox.message = '⚠️ Marca la casilla primero'
    return
  }
  handleVerify(token, 'v2_checkbox')
}

// Ejecutar v2 Badge invisible
const executeV2Badge = () => {
  if (!v2BadgeId) return
  results.v2Badge.message = '⏳ Verificando...'
  window.grecaptcha.execute(v2BadgeId)
}

// Ejecutar v3
const executeV3 = () => {
  if (!v3Key) return
  results.v3.message = '⏳ Verificando...'
  
  // Para v3 necesitamos cargar el script específico
  const existingV3Script = document.querySelector('script[src*="recaptcha/api.js?render="]')
  const existingV2Script = document.querySelector('script[src*="recaptcha/api.js?render=explicit"]')
  
  if (!existingV3Script) {
    console.log('🔄 Cargando script v3...')
    
    // Remover script v2 si existe para evitar conflictos
    if (existingV2Script) {
      console.log('🔄 Removiendo script v2 para evitar conflictos...')
      existingV2Script.remove()
    }
    
    const script = document.createElement('script')
    script.src = `https://www.google.com/recaptcha/api.js?render=${v3Key}`
    script.async = true
    script.defer = true
    script.onload = () => {
      console.log('✅ Script v3 cargado correctamente')
      // Esperar un poco para que se registre
      setTimeout(() => {
        executeV3Token()
      }, 1000)
    }
    script.onerror = () => {
      console.error('❌ Error cargando script v3')
      results.v3.message = '❌ Error cargando v3'
    }
    document.head.appendChild(script)
  } else {
    console.log('✅ Script v3 ya existe, ejecutando...')
    executeV3Token()
  }
}

const executeV3Token = () => {
  if (!window.grecaptcha) {
    console.error('❌ grecaptcha no está disponible')
    results.v3.message = '❌ grecaptcha no disponible'
    return
  }
  
  window.grecaptcha.ready(async () => {
    try {
      console.log('🎯 Ejecutando v3 con key:', v3Key)
      console.log('🔍 grecaptcha disponible:', !!window.grecaptcha)
      console.log('🔍 grecaptcha.execute disponible:', !!window.grecaptcha.execute)
      
      // Verificar que la key esté registrada
      if (typeof window.grecaptcha.execute !== 'function') {
        console.error('❌ grecaptcha.execute no es una función')
        results.v3.message = '❌ API v3 no disponible'
        return
      }
      
      const token = await window.grecaptcha.execute(v3Key, { action: 'demo' })
      console.log('🎯 Token v3 obtenido:', token)
      handleVerify(token, 'v3_score')
    } catch (error) {
      console.error('❌ Error v3:', error)
      console.error('❌ Error details:', error.message)
      results.v3.message = '❌ Error ejecutando v3'
      
      // Si el error es de site key, intentar recargar el script
      if (error.message.includes('Invalid site key')) {
        console.log('🔄 Intentando recargar script v3...')
        const existingScript = document.querySelector('script[src*="recaptcha/api.js?render="]')
        if (existingScript) {
          existingScript.remove()
        }
        // Recargar después de un momento
        setTimeout(() => {
          executeV3()
        }, 1000)
      }
    }
  })
}

onMounted(() => loadScript(initWidgets))
</script>

<style scoped>
.lsx-recaptcha { max-width: 800px; margin: auto; padding: 20px; }
.recaptcha-container { min-height: 80px; display: flex; align-items: center; justify-content: center; background: #f8f9fa; border: 1px dashed #3498db; border-radius: 6px; margin: 10px 0; }
button { padding: 8px 16px; margin-top: 10px; background: #3498db; color: #fff; border: none; border-radius: 4px; cursor: pointer; }
button:disabled { background: #bdc3c7; cursor: not-allowed; }
.result { margin-top: 10px; padding: 8px; border-radius: 4px; font-weight: bold; }
.result.success { background: #d4edda; color: #155724; border: 1px solid #c3e6cb; }
.result.error { background: #f8d7da; color: #721c24; border: 1px solid #f5c6cb; }
</style>
