<template>
  <q-layout
    view="lHh Lpr lFf"
    v-touch-hold.mouse="() => (dialog = !dialog)"
    v-touch-swipe.mouse.right="() => (dialog = !dialog)"
  >
    <Header v-model="drawer" @toggleLeftDrawer="drawer = !drawer" />
    <Drawer v-model="drawer" />
    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
  <q-dialog
    v-model="dialog"
    persistent
    :maximized="maximizedToggle"
    transition-show="slide-up"
    transition-hide="slide-down"
  >
    <q-card>
      <q-bar :class="$q.dark.isActive ? 'q-dark' : 'bg-white text-white'">
        <q-btn
          dense
          flat
          round
          icon="lens"
          size="8.5px"
          color="red"
          v-close-popup
        >
          <q-tooltip class="bg-white text-primary">Close</q-tooltip>
        </q-btn>
        <q-btn
          dense
          flat
          round
          icon="lens"
          size="8.5px"
          color="yellow"
          @click="maximizedToggle = false"
          :disable="!maximizedToggle"
        >
          <q-tooltip v-if="maximizedToggle" class="bg-white text-primary"
            >Minimize</q-tooltip
          >
        </q-btn>
        <q-btn
          dense
          flat
          round
          icon="lens"
          size="8.5px"
          color="green"
          @click="maximizedToggle = true"
          :disable="maximizedToggle"
        >
          <q-tooltip v-if="!maximizedToggle" class="bg-white text-primary"
            >Maximize</q-tooltip
          >
        </q-btn>
        <q-space />
        <!-- TODO -->
        <q-space />
      </q-bar>
      <q-card-section>
        <json-viewer :value="msgs" copyable show-array-index expanded />
      </q-card-section>
    </q-card>
  </q-dialog>
  <q-dialog v-model="confirm" persistent>
    <q-card>
      <q-card-section>
        <div class="text-h6">绑定QQ(否则某些功能无法正常使用</div>
      </q-card-section>
      <q-card-section>
        <q-input v-model="qq" autofocus label="请输入当前QQ" />
      </q-card-section>
      <q-card-actions align="right">
        <q-btn flat label="确认" color="primary" @click="bind" />
      </q-card-actions>
    </q-card>
  </q-dialog>
</template>

<script lang="ts" setup>
import JsonViewer from 'vue-json-viewer';
import Header from '../components/layout/header.vue';
import Drawer from '../components/layout/drawer.vue';
import { inject, onMounted, ref } from 'vue';
import { useQuasar } from 'quasar';

const $q = useQuasar();
const drawer = ref(false);
const dialog = ref(false);
const confirm = ref(false);
const maximizedToggle = ref(true);
const socket = inject('ws') as SocketIOClient.Socket;
const msgs = ref([]);
const qq = ref();

onMounted(() => {
  if (!$q.localStorage.getItem<number>('CurrentQQ')) {
    confirm.value = true;
  }

  $q.dark.set($q.localStorage.getItem<boolean>('dark') ?? 'auto');
  socket.on('OnGroupMsgs', (data: never) => {
    msgs.value.push(data);
  });

  socket.on('OnFriendMsgs', (data: never) => {
    msgs.value.push(data);
  });

  socket.on('OnEvents', (data: never) => {
    msgs.value.push(data);
    $q.notify({
      message: data,
      position: 'top-left',
    });
  });
});
const bind = () => {
  if (qq.value) {
    $q.localStorage.set('CurrentQQ', qq.value);
    confirm.value = false;
  }
};
</script>
