<script setup lang="ts">
  import { ref, reactive } from "vue";
  // import Modal from "./modal.vue";
  // import { reactive } from "vue";
  // import { title } from "process";



  const todos = ref([]);

  // 新規タスクの作成のためのモーダル
  const isShowModal = ref(false);
  const displayModalForNewTask = () => {
    isShowModal.value = true;
  }
  const hideModal = (tasks) => {
    if (!tasks.id) {
      isShowModal.value = false;
    }
  }

  // タスクの編集のためのモーダル
  const isShowModalForEdit = ref(false);


  // 新規タスクを作成
  const newTodo = reactive({
    title: '',
    detail: '',
    deadline: '',
    status: '未着手',
    isDonde: false,
    id: 1
  })

  const editedTodo = reactive({
    title: '',
    detail: '',
    deadline: '',
    status: '未着手',
    isDonde: '',
    id: 0
  })

  // 新規作成モーダル画面にて、「タイトルの有無」をチェック
  // タイトル有り・・・タスクを登録
  // タイトル無し・・・アラート
  const createNewTask = () => {

    // タイトルが入力された場合
    if (newTodo.title) {

      todos.value.push({ ...newTodo });
      newTodo.title = ''
      newTodo.detail = ''
      newTodo.deadline = ''
      newTodo.status = '未着手'
      newTodo.isDonde = ''
      newTodo.id++

      'モーダルウィンドウを閉じる'
      isShowModal.value = false;

    // タイトルが空欄の場合
    } else {
      alert('タスク名を入力してください。')

    }
  }

  // タスクの削除
  // const deleteTodo = (deletedTodoId) => {
  //   const index = todos.value.findIndex((todo) => todo.id === deletedTodoId);

  //   // 削除ボタン→メッセージボックスで確認→削除　というフローを要追加
  //   todos.value.splice(index, 1);
  // }

  // タスクの編集
  const editTodo = (editedTodoId) => {
    Object.assign(
      editedTodo,
      todos.value.find((todo) => todo.id === editedTodoId)
    )

    // モーダルウィンドウを表示
    isShowModalForEdit.value = true;
  }

  // タスクの上書き更新
  const updateTodo = () => {
    const index = todos.value.findIndex((todo) => todo.id === editedTodo.id)
    todos.value[index].title = editedTodo.title
    todos.value[index].detail = editedTodo.detail
    todos.value[index].deadline = editedTodo.deadline
    todos.value[index].status = editedTodo.status

    // モーダルウィンドウを閉じる
    isShowModalForEdit.value = false;
  }

  // タスクの削除
  const deleteAlert = (deletedTodo) => {

    // ダイアログで削除するか否かをチェック
    let checkDelete = window.confirm('タスク名：' + deletedTodo.title + '\nを削除してもよろしいですか？');

    // oK が押下されたら削除
    if (checkDelete) {
      const index = todos.value.findIndex((todo) => todo.id === deletedTodo.id);

      todos.value.splice(index, 1);
    }    
  }

  // ボタンの表記変更　完了⇔未完了
  const isShowCompletedTask = ref(false);
  // 各タスクの完了⇔未完了
  // const isDone = ref(false);

  const switchCompIncomp = () => {

    // ボタンの表示切り替え
    isShowCompletedTask.value = !isShowCompletedTask.value
  }

</script>

<template>

  <div class="toDoList">

    <!-- ヘッダー -->
    <div class="toDoListHeader">
      <h2 class="toDoListHeaderTitle">To Do List</h2>
    </div>
    <div class="headerBtn">
      <button class="btnForDisplayModal" @click="displayModalForNewTask">タスクの作成</button>
      <button class="btnForDisplayModal btnForSwitchCompIncomp" @click="switchCompIncomp">
        {{ isShowCompletedTask ? '進行中タスクに切り替え' : '完了タスクに切り替え' }}
      </button>
    </div>

    <!-- 新規作成用のモーダル -->
    <div id="modalForCreate" class="modal" v-show="isShowModal">
      <h3 class="modalTitle">新規タスクの作成</h3>

      <div class="modal-content">
        <label for="modal-createTask">タスク名 : </label>
        <input id="modal-createTask" class="modal-input" type="text" placeholder="タスク名を記載してください" v-model="newTodo.title">

        <label for="modal-taskDetail">タスク詳細 : </label>
        <textarea name="" class="modal-textarea" id="modal-taskDetail" placeholder="タスクの詳細を記載してください"
          v-model="newTodo.detail"></textarea>

        <label for="modal-deadline">期限 : </label>
        <input id="modal-deadline" class="modal-deadline" type="date" placeholder="例: 2024/1/1"
          v-model="newTodo.deadline">
      </div>

      <div class="modal-button">
        <button class="btnForCreatingNewTask" @click="createNewTask">作成</button>
        <button class="btnForCancel" @click="hideModal('newTodo.value')">キャンセル</button>
      </div>
    </div>

    <!-- 編集用のモーダル -->
    <div id="modalForEdit" class="modal" v-show="isShowModalForEdit">
      <h3 class="modalTitle forEdit">タスクの編集</h3>

      <div class="modal-content">
        <label for="modal-createTask">タスク名 : </label>
        <input id="modal-createTask" class="modal-input" type="text" placeholder="タスク名を記載してください"
          v-model="editedTodo.title">

        <label for="modal-taskDetail">タスク詳細 : </label>
        <textarea name="" class="modal-textarea" id="modal-taskDetail" placeholder="タスクの詳細を記載してください"
          v-model="editedTodo.detail"></textarea>
        <label for="modal-deadline">期限 : </label>
        <input id="modal-deadline" class="modal-deadline" type="date" placeholder="例: 2024/1/1"
          v-model="editedTodo.deadline">

      </div>

      <div class="modal-button">
        <button class="btnForEditingNewTask" @click="updateTodo">更新</button>
        <button class="btnForCancel" @click="isShowModalForEdit = false">キャンセル</button>
      </div>
    </div>

    <!-- タスクを表示するテーブル -->
    <div class="table_box">

      
      <table>
        <!-- タスクが未登録の時に表示 -->
        <div v-if="todos.length === 0">
          <p style="padding: 1.5rem; font-size: 1.2rem; text-align: center">
            タスクが未登録です。<br>
            「タスクの作成」から最初のタスクを登録してみましょう！
          </p>
        </div>

        <!-- タスクが1件以上登録されている時に表示 -->
        <div v-else="todos.length !== 0">
          <tr>
            <th class="sticky_cross" style="width: 5%;">完了</th>
            <th class="sticky_col" style="width: 15%;">期限</th>
            <th class="sticky_col" style="width: 25%;">タスク名</th>
            <th class="sticky_col" style="width: 45%;">タスク詳細</th>
            <th class="sticky_col" style="width: 5%;">編集</th>
            <th class="sticky_col" style="width: 5%;">削除</th>
          </tr>

          <!-- 未完了タスク -->
          <tr v-for="todo in todos" style="list-style:none;">
            <td><input type="checkbox" name="checkboxForCompletion" id="checkboxForCompletion"></td>
            <td>{{ todo.deadline }}</td>
            <td>{{ todo.title }}</td>
            <td>{{ todo.detail }}</td>
            <td><button class="btnInTable editBtn" @click="editTodo(todo.id)">編集</button></td>
            <td><button class="btnInTable deleteBtn" @click="deleteAlert(todo)">削除</button></td>
          </tr>

          <!-- 完了タスク -->
          <!-- <tr v-for="todo in todos" v-show="todo.isDone === true" style="list-style:none;">
            <td><input type="checkbox" name="checkboxForCompletion" id="checkboxForCompletion" v-if="arrCompTasks"></td>
            <td>{{ todo.deadline }}</td>
            <td>{{ todo.title }}</td>
            <td>{{ todo.detail }}</td>
            <td><button class="btnInTable editBtn" @click="editTodo(todo.id)">編集</button></td>
            <td><button class="btnInTable deleteBtn" @click="deleteAlert(todo)">削除</button></td>
          </tr> -->
        </div>
      </table>
    </div>
  </div>


</template>

<style>
  /* -- ここから -- Buttonタグの初期化 */
  button {
    padding: 0;
    border: none;
    outline: none;
    appearance: none;
    background: none;
    cursor: pointer;

    padding: 0.5rem 2rem;
    font-size: 16px;
    border-radius: 1rem;
  }


  /* To do List全体 */
  .toDoList {
    /* margin: 2rem 0; */
    color: rgb(128, 128, 128);
    width: 100vh;
    margin: auto;
  }

  /* To Do Listのヘッダー */
  .toDoListHeader {
    margin-top: 2rem;
    border-bottom: 1px solid rgb(50, 50, 50);
    text-align: center;
    padding-bottom: 0.5rem;
  }
  /* ヘッダー内のボタン（「タスクの作成」「完了 ⇔ 未完了」） */
  .headerBtn {
    display: flex;
    justify-content: space-between;
    margin: 1rem 1.5rem 1rem;
  }

  /* --ここから-- モーダルウィンドウ */
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

  /* モーダルウィンドウ（新規タスクの作成 & 編集） */
  .modal>.modalTitle {
    border-bottom: 1px solid rgba(50, 50, 50, 0.5);
    text-align: center;
    margin-bottom: 2rem;
    padding: 0.5rem;
    border-top-left-radius: 1.5rem;
    border-top-right-radius: 1.5rem;
    background-color: rgb(135, 206, 235);
    color: black;
    opacity: 0.8;
  }

  /* 編集時のモーダルウィンドウ */
  .modal>.forEdit {
    background-color: rgb(255, 250, 205);
  }

  /* モーダルのコンテンツ　と　内部の各種要素 */
  .modal-content {
    padding: 0 4rem;
    height: 70%;
    max-height: 70%;
    margin: auto;
  }
  /* 各ラベル */
  .modal>.modal-content>label {
    display: block;
  }
  /* タスク名　と　タスク詳細 */
  .modal>.modal-content>.modal-input,
  .modal>.modal-content>.modal-textarea {
    margin: 0 0 1rem 1rem;
    width: 90%;
    border-radius: 0.3rem;
  }
  /* タスク詳細 */
  .modal>.modal-content>.modal-textarea {
    min-height: 40%;
    max-height: 60%;
    overflow: scroll;
    box-sizing: border-box;
    resize: vertical;
  }
  /* 期限 */
  .modal-deadline {
    width: 30%;
    margin: 0 0 1rem 1rem;
    border-radius: 0.3rem;
  }

  /* モーダル内の各ボタンを要素として持つdiv */
  .modal-button {
    text-align: center;
    padding: 2rem;
  }
  /* モーダル内の各ボタン */
  .modal-button>button {
    margin-right: 2rem;
  }

  /* -- ここまで -- モーダルウィンドウ */

  /* -- ここから --　モーダル内のボタン */

  /* 新規作成時の「作成」 */
  .modal-button>.btnForCreatingNewTask {
    background-color: rgb(0, 50, 100);
    color: #fff;
  }
  /* 編集時の「更新」 */
  .modal-button>.btnForEditingNewTask {
    background-color: rgb(255, 250, 205);
    /* color: #fff; */
    border: 1px solid rgb(255, 250, 205);
  }

  /* トップ画面の「タスクの作成」 */
  .btnForDisplayModal {
    color: rgb(128, 128, 128);
    border: 2px solid rgba(135, 206, 235, 0.5);
  }
  .btnForDisplayModal:hover {
    color: black;
    background-color: rgb(135, 206, 235);
  }

  /* トップ画面の「完了 ⇔ 未完了」 */
  .btnForSwitchCompIncomp {
    border: 2px solid #660000;
  }
  .btnForSwitchCompIncomp:hover {
    background-color: #B22222;
    border: 2px solid #B22222;
  }

  /* 各種ボタンのhover時 */
  .modal-button>.btnForCreatingNewTask:hover,
  .modal-button>.btnForEditingNewTask:hover,
  .btnForDisplayModal:hover,
  .btnForSwitchCompIncomp:hover {
    opacity: 0.7;
    transition: 0.3s;
  }

  /* 「キャンセル」 */
  .btnForCancel {
    border: 1px solid gray;
    padding: 0.5rem 1rem;
    font-size: 16px;
    border-radius: 1rem;
  }
  .btnForCancel:hover {
    background-color: rgba(150, 150, 150);
    border: 1px solid rgba(150, 150, 150);
    transition: 0.3s;
    color: #fff;
  }

  /* -- ここまで --　モーダル内のボタン */


  /* -- ここから --タスク一覧のテーブル */

  /* テーブルを要素に持つdiv */
  .table_box{
    /* width: 100%; */
    height: 75vh;
    overflow-x: auto;
    overflow-y: auto;
    -webkit-overflow-scrolling: touch;
  }
  /* タスクを表示するテーブル */
  table {
    width: 100%;
    border-collapse: collapse; /* セルの線を重ねる */
  }
  th,
  td {
    border: 1px solid rgb(220, 220, 220, 0.1);
    text-align: center;
  }
  td {
    padding: 0 0.5rem;
  }
  th {
    background-color: rgb(220, 220, 220, 0.1);
    padding: 0.2rem;
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



  /* テーブル内のボタン（「編集」と「削除」） */
  .btnInTable {
    border-radius: 0.3rem;
    padding: 0.1rem 1rem;
    margin: 0.2rem 0;
  }
  .btnInTable:hover {
    opacity: 0.7;
    transition: 0.3s;
  }

  /* 編集ボタン */
  table .editBtn {
    background-color: rgba(255, 250, 205, 0.7);
  }
  /* 削除ボタン */
  table .deleteBtn {
    background-color: gray;
  }

  /* -- ここまで -- タスク一覧のテーブル */


  /* -- ここから -- 各タスクの完了ボタンを示すラジオボタン */

  /* 「完了」のチェックボックス */
  #checkboxForCompletion {
    position: relative;
    width: 16px;
    height: 16px;
    border: 2px solid #660000;
    border-radius: 50%;
    vertical-align: -2px;
    -webkit-appearance: none;
      -moz-appearance: none;
            appearance: none;
  }
  #checkboxForCompletion:checked:before {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 9px;
    height: 9px;
    border-radius: 50%;
    background-color: #660000;
    content: '';
  }

  /* -- ここまで -- 各タスクの完了ボタンを示すラジオボタン */

</style>
