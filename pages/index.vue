<template>
  <div class="flex flex-col items-center justify-center min-h-screen p-4 gap-8">
    <h1 class="text-2xl font-semibold">Работа с Яндекс.Диском</h1>

    <!-- Ввод API токена -->
    <div class="w-full max-w-lg flex flex-col gap-4">
      <label class="text-lg font-medium">API-токен</label>
      <input
          v-model="apiToken"
          type="text"
          class="border p-3 rounded-lg w-full"
          placeholder="Введите API-токен"
      />
      <p v-if="apiTokenError" class="text-sm text-red-500 mt-1">
        {{ apiTokenError }}
        <a href="https://yandex.ru/dev/disk/poligon/" target="_blank" class="text-blue-500 hover:underline">Yandex Disk API</a>
      </p>
    </div>

    <!-- Форма для создания папки -->
    <div class="w-full max-w-lg flex flex-col gap-4">
      <label class="text-lg font-medium">Создать папку</label>
      <input
          v-model="folderName"
          type="text"
          class="border p-3 rounded-lg w-full"
          placeholder="Название папки"
      />
      <p v-if="folderError" class="text-red-500 text-sm">{{ folderError }}</p>
      <button @click="createFolder" class="bg-blue-500 text-white p-3 rounded-lg w-full hover:bg-blue-600">
        Создать папку
      </button>
    </div>

    <!-- Форма для добавления файла -->
    <div class="w-full max-w-lg flex flex-col gap-4">
      <label class="text-lg font-medium">Добавить файл</label>
      <div class="flex flex-row gap-2">
        <input
            type="text"
            v-model="addFolderName"
            class="border p-3 rounded-lg w-44"
            placeholder="Название папки"
        />
        <input
            type="file"
            @change="handleFileUpload"
            class="border p-3 rounded-lg w-full"
        />
      </div>
      <p v-if="fileError" class="text-red-500 text-sm">{{ fileError }}</p>
      <button @click="uploadFile" class="bg-blue-500 text-white p-3 rounded-lg w-full hover:bg-blue-600">
        Загрузить файл
      </button>
    </div>

    <!-- Получение публичной ссылки -->
    <div class="w-full max-w-lg flex flex-col gap-4">
      <label class="text-lg font-medium">Получить публичную ссылку</label>
      <input
          type="text"
          v-model="folderPublicLink"
          class="border p-3 rounded-lg w-full"
          placeholder="Название папки"
      />
      <p v-if="getPublicLinkError" class="text-red-500 text-sm">{{ getPublicLinkError }}</p>
      <button @click="getPublicLink" class="bg-blue-500 text-white p-3 rounded-lg w-full hover:bg-blue-600">
        Получить ссылку
      </button>
      <div v-if="publicLink" class="text-sm text-gray-700 mt-2">
        <strong>Ссылка для доступа: </strong>
        <a :href="publicLink" target="_blank" class="text-blue-500 hover:underline">{{ publicLink }}</a>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const apiToken = ref(localStorage.getItem('apiToken') || '');

const folderName = ref('');
const addFolderName = ref('');
const folderPublicLink = ref('');
const file = ref(null);
const publicLink = ref('');

const folderError = ref('');
const getPublicLinkError = ref('');
const fileError = ref('');
const apiTokenError = ref('');

const API_URL = 'https://cloud-api.yandex.net/v1/disk';

watchEffect(() => {
  if (apiToken.value) {
    localStorage.setItem('apiToken', apiToken.value);
  }
});

const fetchData = async (url, options) => {
  if (!apiToken.value.trim()) {
    apiTokenError.value = 'Получить токен можно по ссылке: ';
    return null;
  }

  const headers = {
    Authorization: `OAuth ${apiToken.value}`,
    'Content-Type': 'application/json',
    ...options.headers,
  };

  const response = await fetch(url, {...options, headers});

  if (!response.ok) {
    const errorData = await response.text();
    alert(`${JSON.parse(errorData).message}`);
    return null;
  }

  return await response.json();
};

const createFolder = async () => {
  if (!folderName.value.trim()) {
    folderError.value = 'Пожалуйста, введите название папки';
    return;
  }

  folderError.value = '';

  const path = folderName.value;
  const url = `${API_URL}/resources?path=${encodeURIComponent(path)}`;
  const options = {method: 'PUT'};

  const response = await fetchData(url, options);

  if (response) {
    alert('Папка создана успешно');
  }
};

const handleFileUpload = (event) => {
  file.value = event.target.files[0];
};

const uploadFile = async () => {
  if (!file.value) {
    fileError.value = 'Пожалуйста, выберите файл для загрузки';
    return;
  }

  fileError.value = '';

  const path = addFolderName.value ? addFolderName.value + '/' + file.value.name : file.value.name;

  const uploadUrl = await fetchData(`${API_URL}/resources/upload?path=${encodeURIComponent(path)}&overwrite=true`, {method: 'GET'});

  if (uploadUrl) {
    const fileUploadResponse = await fetch(uploadUrl.href, {
      method: 'PUT',
      body: file.value,
    });

    if (fileUploadResponse.ok) {
      alert('Файл успешно загружен');
    } else {
      alert('Ошибка при загрузке файла');
    }
  }
};

const getPublicLink = async () => {
  if (!folderPublicLink.value.trim()) {
    getPublicLinkError.value = 'Пожалуйста, введите название папки';
    return;
  }

  getPublicLinkError.value = '';
  const path = folderPublicLink.value;
  const url = `${API_URL}/resources/publish?path=${encodeURIComponent(path)}`;

  const body = {
    public_settings: {
      read_only: false,
    },
  };

  const response = await fetchData(url, {
    method: 'PUT',
    body: JSON.stringify(body),
  });

  if (response) {
    const linkResponse = await fetchData(`${API_URL}/resources?path=${encodeURIComponent(path)}`, {
    method: 'GET'
  });
  if (linkResponse) {
    publicLink.value = linkResponse.public_url;
  }
}
};
</script>

<style>
</style>
