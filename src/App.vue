<template>
  <main class="app">
    <section class="greeting">
      <h1 class="title"><input 
        type="text" 
        placeholder="이름을 입력해주세요"
        v-model.lazy.trim="name"
        size=7
        maxlength=5
      >님,<br class="m_only"> 안녕하세요!😀</h1>
      
    </section>

    <!-- todo 입력 -->
    <section class="create-todo">
      <h2>✦ CREATE A TODO ✦</h2>
      <form id="new-todo-form" v-on:submit.prevent="addTodo">
        <h4>{{ name }}님의 오늘 해야할 일은 무엇인가요?</h4>
        <input type="text" placeholder="할 일을 입력해주세요." v-model.lazy="input_content">

        <div class="options">
          <label>
            <input 
              type="radio" 
              name="categroy" 
              value="business" 
              v-model="input_category"
            >
            <span class="bubble business"></span>
            <div>Business</div>
          </label>

          <label>
            <input 
              type="radio" 
              name="categroy" 
              value="personal" 
              v-model="input_category"
            >
            <span class="bubble personal"></span>
            <div>Personal</div>
          </label>
        </div>

        <div class="info_wrap">
          <h3>Confirm!!</h3>
          <span class="selected">
            입력한 내용 : <strong> {{ input_content }}</strong><br>
            선택한 카테고리 : <strong>{{ input_category }}</strong>
          </span>
        </div>

        <input type="submit" value="Add TODO">
        
      </form>
    </section>

    <section class="todo-list">
      <h2>✦ TODO LIST ✦</h2>
      <h4>글씨 부분을 클릭하여 내용을 수정할 수 있습니다.</h4>
      <div class="list" id="todo-list">
        <div v-for="todo in todos_asc" :class="`todo-item ${ todo.done && 'done' }`">
          <label>
            <input type="checkbox" v-model="todo.done">
            <span :class="`bubble ${todo.category}`"></span>
          </label>
          <div class="todo-content">
            <input type="text" v-model="todo.content">
          </div>
          <div class="actions">
            <button class="del" @click="removeTodo(todo)">Delete</button>
                                        <!-- v-for="todo in todos"의 todo를 인자로 받아옴.-->
          </div>
        </div>
      </div>
    </section>
  </main>
</template>

<script setup>
import { computed, onMounted, ref , watch } from 'vue';

  const todos = ref([]); //할일들을 모아놓을 배열
  const name = ref('');
  const input_content = ref('');
  const input_category = ref('아직 선택하지 않았습니다.');

  //할일 함수- 입력값을 받아옴
  const addTodo = () => {
    if( input_content.value.trim() === '' || 
        input_category.value === '아직 선택하지 않았습니다.'){
      return
    }

    todos.value.push({
      content: input_content.value,
      category: input_category.value,
      done:false,
      createAt: new Date().getTime(), //UTC시간 입력
    })

    //입력 후 초기화
    input_content.value = '';
    input_category.value = '아직 선택하지 않았습니다.';
  }
  
  // todos 배열을 시간 순서대로 정리 -> 새로운 todo가 상단에 위치하게 sort이용
  const todos_asc = computed(()=> todos.value.sort((a,b)=>{
      return b.createAt - a.createAt 
      //내림차순  
    }))

  

  //watch- 할일 입력된 것을 인지
  watch(todos,(newVal)=>{
    localStorage.setItem('todos',JSON.stringify(newVal)) //JSON파일로 전환해야함
  },{deep:true}) // deep: 프로퍼티나, 하위 뎁스도 감시

  //watch를 이용하여 입력을 인지하여 localStorage에 등록
  watch( name, (newVal)=>{
    localStorage.setItem('name', newVal)
    // shoppingList에서는 목록이 많아서 JSON 형식으로 등록하였으나, 단일 값 이므로 바로 값을 입력
  })



  //remove 함수
  const removeTodo = (todo) => {
    //내가 클릭한 것을 제외한 나머지를 새로운 array를 만듦.
    // item = 기존의 array 하나하나. 즉. 기존의 배열(item)과 내가선택한것(todo)가 같지 않은것(!==)으로 새 배열을 filter를 이용하여 만듦.
    todos.value = todos.value.filter((item) => item !== todo ) 
  }

  //새로 열었을때 (새로 마운트 되었을 때) 로컬스토리지에서 불러옴
  onMounted(()=>{
    name.value = localStorage.getItem('name') || ''; 
    // || : or. 만약에 앞의 값이 없으면 뒤의 값을 출력

    todos.value = JSON.parse(localStorage.getItem('todos')) || []; 
      //JSON 파일을 내가 사용할수있게 변환, 뒷부분의 공란은 array[]로 표시해야함
  })

  
</script>
