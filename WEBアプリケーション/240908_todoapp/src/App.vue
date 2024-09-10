<script setup lang="ts">
import { computed, reactive, ref } from 'vue';

// 「新規作成」「編集」モーダルウィンドウ（初期値 = 非表示）
const isShowModalForCreating = ref(false);
const isShowModalForEdit = ref(false);

// 「新規作成」ボタン押下でモーダルウィンドウを開く
const displayModalForCreating = () => {
  isShowModalForCreating.value = true;
}

// ソートの項目（id = 登録順でソート）
const SORT_ITEMS = ['昇順', '降順'];

// ステータスのマスタ
const STATUS_ITEMS = ['未着手', '着手', '完了'];
// ヘッダー部分に出力してSelectさせる項目
const FILTER_ITEMS = ['全て表示', '未着手', '着手', '完了'];

// タスクデータ
const todos = ref([]);

// 新規タスクの初期値
const newTodo = reactive({
  title: '',
  detail: '',
  deadline: '',
  status: '未着手',
  isDonde: false,
  id: 0
});
// 編集時の初期値
const editedTodo = reactive({
  title: '',
  detail: '',
  deadline: '',
  status: '',
  isDonde: false,
  id: 0
});

// 新規タスクの作成ボタンが押下されたら
const createNewTask = () => {
  if (newTodo.title) {
    todos.value.push({ ...newTodo });
    newTodo.title = '',
      newTodo.detail = '',
      newTodo.deadline = '',
      newTodo.status = '未着手',
      newTodo.isDonde = false,
      newTodo.id++

    // モーダルウィンドウを閉じる
    isShowModalForCreating.value = false;

    // タイトルが空欄の場合
  } else {
    alert('タスク名を入力してください。');
  }
}

// 編集
const editTodo = (editedTodoId) => {

  // 編集時の初期値に、該当するタスクIDのタスク内容を複製
  Object.assign(editedTodo, todos.value.find((todo) => todo.id === editedTodoId));

  // モーダルウィンドウを開く
  isShowModalForEdit.value = true;
}

// タスクを上書き更新
const updateTodo = () => {

  // 編集対象のタスクの配列のindexを取得
  const index = todos.value.findIndex((todo) => todo.id === editedTodo.id);

  // 保持しているtodosの値に、editedTodoの値を代入
  todos.value[index].title = editedTodo.title;
  todos.value[index].detail = editedTodo.detail;
  todos.value[index].deadline = editedTodo.deadline;
  todos.value[index].status = editedTodo.status;

  // モーダルウィンドウを閉じる
  isShowModalForEdit.value = false;
}

// タスクを削除
const deleteTodo = (deletedTodo) => {

  // ダイアログで削除するか否かをチェック
  const checkDelete = window.confirm('タスク名：' + deletedTodo.title + '\nを削除してもよろしいですか？');

  // ダイアログで「OK」が押下されたら
  if (checkDelete) {

    // 削除対象のタスクのIDを取得
    const index = todos.value.findIndex((todo) => todo.id === deletedTodo.id);

    // 対象タスクを削除
    todos.value.splice(index, 1);
  }
}

// ソートする項目（'期日', 'ステータス', '登録順'）
const selectedSort = ref(SORT_ITEMS[0]);
const sortedTodos = computed(() => {
  
  // 選択されたソートが「降順」の場合
  if (selectedSort.value === '降順') {

    // 登録された全てのタスクを複製し、一件ずつ比較
    return [...todos.value].sort((a, b) => {

      // 並び替えの比較基準はタスクのID（ = 登録順）
      if (a.id < b.id) {
        return 1;

      } else if (a.id > b.id) {
        return -1;

      } else {
        return 0;
      }
    })
  
  // 選択されたソートが「昇順」の場合
  } else if (selectedSort.value === '昇順') 
  {
    return [...todos.value].sort((a, b) => {

      if (a.id < b.id) {
        return -1;

      } else if (a.id > b.id) {
        return 1;

      } else {
        return 0;
      }
    })
  }

  // 並び替え済みの全タスクをreturn
  return todos.value;
})

// selectから選択されるFilterの項目（ステータス）を取得
const selectedFilter = ref(FILTER_ITEMS[0]);

// タスクのステータスが変更された時に関数実行
const filteredTodos = computed(() => {

  // 選択されたフィルターが「全て表示」の場合
  if (selectedFilter.value === '全て表示') {

    // 全てのタスク　を　ソート　の選択（昇順 or 降順）に合わせて表示
    return sortedTodos.value;

  // 選択されたフィルターが「未着手 / 着手 / 完了」の場合
  } else {

    // 該当のステータスであるタスクのみ　を　ソート　の選択（昇順 or 降順）に合わせて表示
    return sortedTodos.value.filter((todo) => todo.status === selectedFilter.value);
  }
})
</script>

<template>
  <div class="toDoList">
    <div class="header">
      <h2 class="toDoListHeader">To Do List</h2>

      <div class="subHeader">
        <button class="btnForDisplayModal" @click="displayModalForCreating">新規作成</button>
        <div style="text-align: end;" v-show="todos.length > 0">
          <div>
            <label for="sort">ソート: </label>
            <select name="sort" id="sort" v-model="selectedSort">
              <option :value="sortItem" v-for="sortItem in SORT_ITEMS" :key="sortItem">
                {{ sortItem }}
              </option>
            </select>
          </div>
          
          <div>
            <label for="filter">フィルター: </label>
            <select name="filter" id="filter" v-model="selectedFilter">
              <option :value="filterItem" v-for="filterItem in FILTER_ITEMS" :key="filterItem">{{ filterItem }}</option>
            </select>
          </div>
        </div>
      </div>
      
    </div>

    <div class="container">
      <!-- 登録されているタスクが無い場合 -->
      <div v-if="todos.length === 0" style="text-align: center;">
        <p style="padding: 1.5rem; font-size: 1.2rem; text-align: center">
          タスクがまだ登録されていません。<br>
          「タスクの作成」から最初のタスクを登録してみましょう！
        </p>
      </div>

      <!-- タスクが1件以上登録されている場合 -->
      <div class="table_box" v-else>
        <table>
          <tr>
            <th class="sticky_cross" style="width: 10%;">ステータス</th>
            <th class="sticky_col" style="width: 15%;">期限</th>
            <th class="sticky_col" style="width: 25%;">タスク名</th>
            <th class="sticky_col" style="width: 34%;">タスク詳細</th>
            <th class="sticky_col" style="width: 8%;">編集</th>
            <th class="sticky_col" style="width: 8%;">削除</th>
          </tr>

          <tr v-if="filteredTodos.length === 0">
            <td colspan="6" class="zeroTasksAfterFilter">ステータスが「{{ selectedFilter }}」のタスクは登録ありません。</td>
          </tr>

          <tr v-for="todo in filteredTodos" :key="todo.id">
            <td>{{ todo.status }}</td>
            <td>{{ todo.deadline }}</td>
            <td>{{ todo.title }}</td>
            <td>{{ todo.detail }}</td>
            <td><button class="btnInTable editBtn" @click="editTodo(todo.id)">編集</button></td>
            <td><button class="btnInTable deleteBtn" @click="deleteTodo(todo)">削除</button></td>
          </tr>
        </table>
      </div>

      <!-- 新規作成のモーダルウィンドウ -->
      <div class="modal modalForCreate" v-show="isShowModalForCreating">
        <h3 class="modalHeader forCreating">タスクの新規作成</h3>
        <div class="modalContainer">

          <label for="status" class="modalStatus">ステータス :</label>
          <select name="status" id="status" class="modalSelect" v-model="newTodo.status">
            <option :value="statusItem" :key="statusItem" v-for="statusItem in STATUS_ITEMS">
              {{ statusItem }}
            </option>
          </select>

          <label for="title" class="modalTitle">タスク名 :</label>
          <input id="title" type="text" class="modalInput" placeholder="タスク名を記載してください" v-model="newTodo.title">

          <label for="detail" class="modalDetail">タスク詳細 :</label>
          <textarea name="detail" id="detail" class="modalTextarea" placeholder="タスクの詳細を記載してください"
            v-model="newTodo.detail"></textarea>

          <label for="deadline" class="modalDeadline">期限 :</label>
          <input type="date" name="deadline" id="deadline" class="modalInput" v-model="newTodo.deadline">
        </div>
        <div class="btnInModal">
          <button class="btnForCreating" @click="createNewTask">作成</button>
          <button class="btnForCancel" @click="isShowModalForCreating = false">キャンセル</button>
        </div>
      </div>

      <!-- 編集のモーダルウィンドウ -->
      <div class="modal modalForCreate" v-show="isShowModalForEdit">
        <h3 class="modalHeader forEdit">タスクの編集</h3>
        <div class="modalContainer">

          <label for="status" class="modalStatus">ステータス :</label>
          <select name="status" id="status" class="modalSelect" v-model="editedTodo.status">
            <option :value="statusItem" :key="statusItem" v-for="statusItem in STATUS_ITEMS">
              {{ statusItem }}
            </option>
          </select>

          <label for="title" class="modalTitle">タスク名 :</label>
          <input id="title" type="text" class="modalInput" placeholder="タスク名を記載してください" v-model="editedTodo.title">

          <label for="detail" class="modalDetail">タスク詳細 :</label>
          <textarea name="detail" id="detail" class="modalTextarea" placeholder="タスクの詳細を記載してください"
            v-model="editedTodo.detail"></textarea>

          <label for="deadline" class="modalDeadline">期限 :</label>
          <input type="date" name="deadline" id="deadline" class="modalInput" v-model="editedTodo.deadline">
        </div>
        <div class="btnInModal">
          <button class="btnForEdit" @click="updateTodo">保存</button>
          <button class="btnForCancel" @click="isShowModalForEdit = false">キャンセル</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
/*Buttonタグの初期化 */
button {
  padding: 0;
  border: none;
  appearance: none;
  background: none;
  cursor: pointer;
  border-radius: 1rem;
}

/* To do List全体 */
.toDoList {
  color: rgb(128, 128, 128);
  width: 100vh;
}

/* 「キャンセル」ボタン */
.btnForCancel {
  border: 1px solid gray;
  padding: 0.5rem 1rem;
  border-radius: 1rem;
}

.btnForCancel:hover {
  background-color: rgba(150, 150, 150);
  border: 1px solid rgba(150, 150, 150);
  transition: 0.3s;
  color: #fff;
}

/* To Do Listのヘッダー */
.toDoListHeader {
  margin-bottom: 1rem;
  padding-bottom: 0.5rem;
  border-bottom: 1px solid rgb(50, 50, 50);
  text-align: center;
  font-weight: bold;
}

.subHeader {
  display: flex;
  justify-content: space-between;
  margin: 0 1rem;
}

/* トップ画面の「タスクの作成」 */
.btnForDisplayModal {
  padding: 0.5rem 2rem;
  font-size: 1rem;
  color: rgb(128, 128, 128);
  border: 2px solid rgba(135, 206, 235, 0.5);
}

.btnForDisplayModal:hover {
  color: black;
  background-color: rgb(135, 206, 235);
}


/* -- ここから -- モーダルウィンドウ*/
.modal {
  z-index: 100;

  /*　画面全体を覆う設定　*/
  position: fixed;
  top: 50%;
  left: 50%;
  width: 40%;
  height: 80%;
  transform: translate(-50%, -50%);
  background-color: rgba(222, 222, 222);
  border-radius: 1.5rem;
}

/* モーダルのコンテンツ　と　内部の各種要素 */
.modalContainer {
  padding: 0 4rem;
  height: 70%;
  max-height: 70%;
  margin: auto;
}

/* モーダルウィンドウ（新規タスクの作成 & 編集） */
.modal>.modalHeader {
  border-bottom: 1px solid rgba(50, 50, 50, 0.5);
  text-align: center;
  margin-bottom: 2rem;
  padding: 0.5rem;
  border-top-left-radius: 1.5rem;
  border-top-right-radius: 1.5rem;
  color: black;
  opacity: 0.8;
}

/* 新規作成時のモーダルウィンドウ */
.modal>.forCreating {
  background-color: rgb(135, 206, 235);
}

/* 編集時のモーダルウィンドウ */
.modal>.forEdit {
  background-color: rgb(255, 250, 205);
}

/* モーダル内の各ラベル */
.modalContainer>label {
  display: block;
}

/* ステータス　と　タスク名　と　タスク詳細 */
.modal>.modalContainer>.modalSelect,
.modal>.modalContainer>.modalInput,
.modal>.modalContainer>.modalTextarea {
  margin: 0 0 1rem 1rem;
  width: 90%;
  border-radius: 0.3rem;
}

/* ステータス　と　タスク名 */
.modal>.modalContainer>.modalSelect,
.modal>.modalContainer>.modalInput {
  min-height: 2rem;
}

/* モーダルの「期限」 */
.modal>.modalContainer>.modalSelect,
.modal>.modalContainer>.modalInput:last-child {
  width: 30%;
}

/* タスク詳細 */
.modal>.modalContainer>.modalTextarea {
  min-height: 40%;
  max-height: 60%;
  overflow: scroll;
  box-sizing: border-box;
  resize: vertical;
}

/* 期限 */
.modalDeadline {
  width: 30%;
  border-radius: 0.3rem;
}

/* モーダルウィンドウ（新規作成）の中の「作成」ボタン */
.btnForCreating {
  background-color: rgb(0, 50, 100);
  color: #fff;
}

/* 編集時の「更新」 */
.btnForEdit {
  background-color: rgb(255, 250, 205);
  border: 1px solid rgb(255, 250, 205);
}

.btnForCancel:hover {
  background-color: rgba(150, 150, 150);
  border: 1px solid rgba(150, 150, 150);
  transition: 0.3s;
  color: #fff;
}

/* 各種ボタンのhover時 */
.btnForDisplayModal:hover,
.btnForCreating:hover,
.btnForEdit:hover,
.btnForCancel:hover {
  opacity: 0.7;
  transition: 0.3s;
}

/* モーダル内の各ボタンを要素として持つdiv */
.btnInModal {
  text-align: center;
}

/* モーダル内の各ボタン */
.btnInModal>button {
  margin-right: 2rem;
  padding: 0.5rem 2rem;
  font-size: 1rem;
}



/* -- ここから -- タスクを表示するテーブル */
.zeroTasksAfterFilter {
  font-size: 1.2rem;
  border: none;
  padding-top: 1rem;
}
.container {
  margin-top: 2rem;
}

/* テーブルを要素に持つdiv */
.table_box {
  width: 100%;
  height: 75vh;
  overflow-x: auto;
  overflow-y: auto;
}

/* タスクを表示するテーブル */
table {
  width: 100%;
  border-collapse: collapse;
}

th,
td {
  border: 1px solid rgb(220, 220, 220, 0.5);
  text-align: center;
}

th {
  font-size: 0.9rem;
  background-color: rgb(220, 220, 220, 0.1);
  padding: 0.2rem;
}

td {
  padding: 0 0.5rem;
}

/* テーブルのヘッダー（ヘッダー固定） */
.sticky_col {
  position: sticky;
  top: 0;
  left: 0;
  background: none;
  border-top: none;
  border-bottom: none;
}

.sticky_col::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: #ccc;
  z-index: -1;
}

.sticky_cross {
  position: sticky;
  top: 0;
  left: 0;
  background: none;
  border-top: none;
  border-bottom: none;
  border-left: none;
  border-right: none;
  z-index: 1;
}

.sticky_cross::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: #ccc;
  z-index: -1;
}

/* テーブル内各タスクのボタン（「編集」と「削除」） */
.btnInTable {
  border-radius: 0.3rem;
  padding: 0.3rem 0.5rem;
  margin: 0.2rem 0;
}

.btnInTable:hover {
  opacity: 0.7;
  transition: 0.3s;
}

/* テーブル内各タスクの「編集」ボタン */
table .editBtn {
  background-color: rgba(255, 250, 205, 0.7);
}

/* テーブル内各タスクの「削除」ボタン */
table .deleteBtn {
  background-color: gray;
}
</style>
