<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>NFC</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">NFC</ion-title>
        </ion-toolbar>
      </ion-header>

      <div id="container">
        <ion-list>
          <ion-item v-for="msg of messages" :key="msg.timestamp">
            <ion-label :style="{ color: decodeColor(msg.type) }">
              {{ msg.text }}
            </ion-label>
          </ion-item>
          <ion-item>
            <ion-input v-model="testInput"> </ion-input>
            <ion-button @click="sendEvent">Send</ion-button>
          </ion-item>
        </ion-list>
      </div>
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
  IonInput,
  IonButton,
  IonItem,
  IonLabel,
  IonList,
} from '@ionic/vue'
import { defineComponent, ref, Ref } from 'vue'
import { NFC as nfc } from '@awesome-cordova-plugins/nfc'
export default defineComponent({
  name: 'HomePage',
  components: {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
    IonInput,
    IonButton,
    IonItem,
    IonLabel,
    IonList,
  },
  setup() {
    interface Message {
      type: string
      text: string
      timestamp: string
    }
    // const url = 'http://localhost:8000'
    // const url = 'http://192.168.1.24:8000'
    let url = 'https://stark-chamber.deno.dev'
    let testInput = ref('')
    let messages: Ref<Message[]> = ref([])

    let flags = nfc.FLAG_READER_NFC_A | nfc.FLAG_READER_NFC_V
    nfc.readerMode(flags).subscribe(
      async (tag) => {
        let message: null | string[] = ['']
        if (tag.ndefMessage) {
          const binaryPayload = tag.ndefMessage[0]['payload']
          const payload = nfc.bytesToString(binaryPayload)
          message = payload.match(/.en(.*)/)
          // that should never happen
          if (message === null) message = ['blank nfc','blank nfc']
        }
        messages.value.push({
          type: 'info',
          text: JSON.stringify(message[1]),
          timestamp: new Date().toISOString(),
        }),
          await fetchPost(url + '/trigger', { value: message[1] })
      },
      (err) =>
        messages.value.push({
          type: 'error',
          text: err,
          timestamp: new Date().toISOString(),
        })
    )

    async function fetchPost(backendUrl: string, data: any): Promise<any> {
      const response = await fetch(backendUrl, {
        method: 'POST',
        headers: {
          Accept: 'application/json',
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(data),
      })
    }

    function decodeColor(type: string) {
      if (type === 'info') return 'var(--ion-color-secondary)'
      if (type === 'error') return 'var(--ion-color-danger)'
      return 'var(--ion-color-warning)'
    }

    async function sendEvent() {
      const regex = testInput.value.match(/#set (.*)/)
      if (regex !== null) {
        url = regex[1]
        messages.value.push({
        type: 'test',
        text: `URL set to ${url}`,
        timestamp: new Date().toISOString(),
      })
        return
      }
      await fetchPost(url + '/trigger', {
        value: testInput.value,
      })
      messages.value.push({
        type: 'test',
        text: testInput.value,
        timestamp: new Date().toISOString(),
      })
      testInput.value = ''
    }

    return { testInput, messages, decodeColor, sendEvent }
  },
})
</script>
