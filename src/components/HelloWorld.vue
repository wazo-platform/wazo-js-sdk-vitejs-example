<template>
  <h1>{{ msg }}</h1>

  Your status: {{ state.status }}
</template>

<script setup>
import { defineProps, reactive } from 'vue';
import { WazoWebSocketClient, WazoApiClient } from '@wazo/sdk';

defineProps({
  msg: String
})

const state = reactive({ status: '[Waiting for status update of your user]' });
const server = 'xxx';
const client = new WazoApiClient({ server });

(async () => {
  const session = await client.auth.logIn({ username: 'xxx', password: 'xxx', expiration: 300 });
  client.setToken(session.token);

  const ws = new WazoWebSocketClient({
    host: server,
    token: session.token,
    version: 2,
    events: ['*'],
  });

  ws.connect();
  ws.on('chatd_presence_updated', message => {
    if (message.data.uuid === session.uuid) {
      state.status = message.data.status;
    }
  })
})();
</script>
