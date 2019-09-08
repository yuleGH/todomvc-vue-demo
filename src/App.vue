<template>
    <div id="app">
        <section class="todoapp">
            <header class="header">
                <h1>todos</h1>
                <input class="new-todo" placeholder="What needs to be done?" autofocus @keyup.enter="handleAddTodo" v-model.trim="newTodo">
            </header>
            <!-- This section should be hidden by default and shown when there are todos -->
            <section class="main" v-show="todoList.length">
                <!--全选-->
                <input id="toggle-all" class="toggle-all" type="checkbox" v-model="allCompleted">
                <label for="toggle-all">Mark all as complete</label>
                <ul class="todo-list">
                    <!-- These are here just to show the structure of the list items -->
                    <!-- List items should get the class `editing` when editing and `completed` when marked as completed -->
                    <li :class="['todo', todo.completed ? 'completed' : '', todo == editedTodo ? 'editing' : '']" v-for="(todo, index) in todoListByVisibility" :key="todo.id">
                        <div class="view">
                            <input class="toggle" type="checkbox" v-model="todo.completed">
                            <label @dblclick="handleDbClickTodo(todo)">{{ todo.title }}</label>
                            <button class="destroy" @click="handleRemoveTodo(index)"></button>
                        </div>
                        <input class="edit" v-model.trim="editTodoTitle"
                               v-todo-focus="todo == editedTodo"
                               @blur="handleDoneEdit(todo, index)"
                               @keyup.enter="handleDoneEdit(todo, index)"
                               @keyup.esc="handleCancelEdit(todo)">
                    </li>
                </ul>
            </section>
            <!-- This footer should hidden by default and shown when there are todos -->
            <footer class="footer" v-show="todoList.length">
                <!-- This should be `0 items left` by default -->
                <span class="todo-count"><strong>{{ activeTodoList.length }}</strong> {{ activeTodoList.length === 1 ? 'item' : 'items' }} left</span>
                <!-- Remove this if you don't implement routing -->
                <ul class="filters">
                    <li>
                        <a :class="{selected: visibility === 'all'}" href="#/" @click="visibility='all'">All</a>
                    </li>
                    <li>
                        <a :class="{selected: visibility === 'active'}" href="#/active" @click="visibility='active'">Active</a>
                    </li>
                    <li>
                        <a :class="{selected: visibility === 'completed'}" href="#/completed" @click="visibility='completed'">Completed</a>
                    </li>
                </ul>
                <!-- Hidden if no completed items are left ↓ -->
                <button class="clear-completed" v-show="completedTodoList.length" @click="handleClearCompleted">Clear completed</button>
            </footer>
        </section>
        <footer class="info">
            <p>Double-click to edit a todo</p>
            <p>Written by <a href="https://github.com/yuleGH">yuxiaole</a></p>
            <p>Part of <a href="http://todomvc.com">TodoMVC</a></p>
        </footer>
    </div>
</template>

<script>

    // localStorage persistence
    const STORAGE_KEY = 'todos-vuejs';
    let todoStorage = {
        fetch: function () {
            let todos = JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]')
            return todos
        },
        save: function (todoList) {
            localStorage.setItem(STORAGE_KEY, JSON.stringify(todoList))
        }
    };

    export default {
        name: 'app',
        data: function(){
            return {
                //所有的任务列表
                todoList: todoStorage.fetch(),
                //新的任务
                newTodo: '',
                //显示数据列表状态，默认all
                visibility: 'all',
                //编辑时的
                editTodoTitle: '',
                //编辑的
                editedTodo: null
            }
        },
        // watch todos change for localStorage persistence
        watch: {
            todoList: {
                handler: function (todoList) {
                    todoStorage.save(todoList)
                },
                deep: true
            }
        },
        computed: {
            allCompleted: {
                set(newValue){
                    this.todoList.forEach(item => item.completed = newValue);
                },
                get(){
                    return this.activeTodoList.length === 0;
                }
            },
            activeTodoList(){
                return this.todoList.filter(item => !item.completed);
            },
            completedTodoList(){
                return this.todoList.filter(item => item.completed);
            },
            todoListByVisibility(){
                if(this.visibility === 'all'){
                    return this.todoList;
                }else if(this.visibility === 'active'){
                    return this.activeTodoList;
                }else if(this.visibility === 'completed'){
                    return this.completedTodoList;
                }
            }
        },
        // a custom directive to wait for the DOM to be updated
        // before focusing on the input field.
        // http://vuejs.org/guide/custom-directive.html
        directives: {
            //自动获取焦点的指令
            'todo-focus': function (el, binding) {
                if (binding.value) {
                    el.focus()
                }
            }
        },
        methods: {
            /**
             * 新增
             * 回车保存（去掉前后空格）并清空输入框，如果数据为空则不作处理
             */
            handleAddTodo(){
                if(!this.newTodo){
                    return;
                }
                let id = this.todoList.length;
                this.todoList.push({
                    id: ++id,
                    title: this.newTodo,
                    completed: false
                });
                this.newTodo = '';
            },
            /**
             * 删除该任务
             * @param todoIndex
             */
            handleRemoveTodo(todoIndex){
                this.todoList.splice(todoIndex, 1);
                this.todoList.forEach((item, index) => item.id = index+1 );
            },
            /**
             * 清楚所有已完成任务
             */
            handleClearCompleted(){
                let todoList = this.activeTodoList;
                todoList.forEach((item, index) => item.id = index+1 );
                this.todoList = todoList;
            },
            /**
             * 双击进入编辑
             * @param todo
             */
            handleDbClickTodo(todo){
                this.editTodoTitle = todo.title;
                this.editedTodo = todo;
            },
            /**
             * 编辑保存
             * @param todo 编辑的任务
             * @param todoIndex 编辑任务的index
             */
            handleDoneEdit(todo, todoIndex){
                this.editedTodo = null;

                if(!this.editTodoTitle){
                    this.handleRemoveTodo(todoIndex);
                    return;
                }

                todo.title = this.editTodoTitle;
            },
            /**
             * 取消编辑
             * @param todo
             */
            handleCancelEdit(todo){
                this.editedTodo = null;
                this.editTodoTitle = todo.title;
            }

        }
    }
</script>

<style>
    @import url("css/index.css");
</style>
