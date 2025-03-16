// App.vue
<template>
  <div id="app">
    <h1>Vue Storage Demo</h1>
    
    <div>
      <h2>LocalStorage Notes</h2>
      <input v-model="localNote" placeholder="Enter a note" />
      <button @click="saveToLocalStorage">Save</button>
      <ul>
        <li v-for="(n, index) in localNotes" :key="index">
          {{ n }} <button @click="deleteLocalStorageNote(index)">Delete</button>
        </li>
      </ul>
    </div>

    <div>
      <h2>SessionStorage Notes</h2>
      <input v-model="sessionNote" placeholder="Enter a note" />
      <button @click="saveToSessionStorage">Save</button>
      <ul>
        <li v-for="(n, index) in sessionNotes" :key="index">
          {{ n }} <button @click="deleteSessionStorageNote(index)">Delete</button>
        </li>
      </ul>
    </div>

    <div>
      <h2>IndexedDB (LocalForage) Notes</h2>
      <input v-model="indexedNote" placeholder="Enter a note" />
      <button @click="saveToIndexedDB">Save</button>
      <ul>
        <li v-for="(n, index) in indexedNotes" :key="index">
          {{ n }} <button @click="deleteIndexedDBNote(index)">Delete</button>
        </li>
      </ul>
    </div>

    <div>
      <h2>Cookie Notes</h2>
      <input v-model="cookieNote" placeholder="Enter a note" />
      <button @click="saveToCookie">Save</button>
      <ul>
        <li v-for="(n, index) in cookieNotes" :key="index">
          {{ n }} <button @click="deleteCookieNote(index)">Delete</button>
        </li>
      </ul>
    </div>

    <div v-if="supportsWebSQL">
      <h2>Web SQL Notes</h2>
      <input v-model="webSqlNote" placeholder="Enter a note" />
      <button @click="saveToWebSQL">Save</button>
      <ul>
        <li v-for="(n, index) in webSqlNotes" :key="index">
          {{ n }} <button @click="deleteWebSQLNote(index)">Delete</button>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import localforage from 'localforage';

export default {
  setup() {
    const localNote = ref('');
    const localNotes = ref([]);

    const sessionNote = ref('');
    const sessionNotes = ref([]);

    const indexedNote = ref('');
    const indexedNotes = ref([]);

    const cookieNote = ref('');
    const cookieNotes = ref([]);

    const webSqlNote = ref('');
    const webSqlNotes = ref([]);
    const supportsWebSQL = ref(!!window.openDatabase);

    const saveToLocalStorage = () => {
      localNotes.value.push(localNote.value);
      localStorage.setItem('localNotes', JSON.stringify(localNotes.value));
      localNote.value = '';
    };

    const saveToSessionStorage = () => {
      sessionNotes.value.push(sessionNote.value);
      sessionStorage.setItem('sessionNotes', JSON.stringify(sessionNotes.value));
      sessionNote.value = '';
    };

    const saveToIndexedDB = async () => {
      await localforage.setItem(`note-${Date.now()}`, indexedNote.value);
      indexedNotes.value.push(indexedNote.value);
      indexedNote.value = '';
    };

    const saveToCookie = () => {
      document.cookie = `note-${Date.now()}=${cookieNote.value}; path=/;`;
      loadCookies();
      cookieNote.value = '';
    };

    const saveToWebSQL = () => {
      if (!supportsWebSQL.value) return;
      const db = openDatabase('notesDB', '1.0', 'Notes Database', 2 * 1024 * 1024);
      db.transaction(tx => {
        tx.executeSql('CREATE TABLE IF NOT EXISTS notes (id INTEGER PRIMARY KEY, content TEXT)');
        tx.executeSql('INSERT INTO notes (content) VALUES (?)', [webSqlNote.value]);
      });
      loadWebSQL();
      webSqlNote.value = '';
    };

    const loadCookies = () => {
      cookieNotes.value = document.cookie.split('; ').map(c => c.split('=')[1] || '');
    };

    const loadWebSQL = () => {
      if (!supportsWebSQL.value) return;
      const db = openDatabase('notesDB', '1.0', 'Notes Database', 2 * 1024 * 1024);
      db.transaction(tx => {
        tx.executeSql('SELECT * FROM notes', [], (tx, results) => {
          webSqlNotes.value = [];
          for (let i = 0; i < results.rows.length; i++) {
            webSqlNotes.value.push(results.rows.item(i).content);
          }
        });
      });
    };

    onMounted(() => {
      localNotes.value = JSON.parse(localStorage.getItem('localNotes')) || [];
      sessionNotes.value = JSON.parse(sessionStorage.getItem('sessionNotes')) || [];
      loadCookies();
      loadWebSQL();
    });

    return {
      localNote, localNotes, saveToLocalStorage,
      sessionNote, sessionNotes, saveToSessionStorage,
      indexedNote, indexedNotes, saveToIndexedDB,
      cookieNote, cookieNotes, saveToCookie,
      webSqlNote, webSqlNotes, saveToWebSQL,
      supportsWebSQL
    };
  }
};
</script>

<style>
#app {
  font-family: Arial, sans-serif;
  max-width: 600px;
  margin: 20px auto;
  text-align: center;
}
input {
  padding: 8px;
  margin: 10px;
}
button {
  padding: 8px 12px;
  margin: 5px;
  cursor: pointer;
}
</style>
