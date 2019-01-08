<template>
  <div>
    <header class="view-top">
      <h2>ToDoList</h2>
    </header>
    <div class="main">
      <section>
        <h3 class="section-title">添加计划</h3>
        <input type="text" class="task-input" placeholder="添加ToDo" v-model="todo" @keyup.enter="addTodo">
      </section>
      <section>
        <ul class="task-count">
          <li>{{unfinishedCount}}个任务未完成</li>
          <li class="action">
            <a v-for="(item, index) in actionItems" :key="index" :class="{active: item.isActive}" @click="handleChange(item)">{{item.title}}</a>
          </li>
        </ul>
      </section>
      <section class="tasks">
        <h3 class="section-title">计划列表</h3>
        <ul class="todo-list">
          <!-- 通过editing样式来控制是否显示可编辑input框 -->
          <li v-for="(item, index) in showList" :key="index" :class="{completed: item.isChecked, editing: isEditingItem(item)}">
            <input type="checkbox" v-model="item.isChecked" @change="checking(item.isChecked)">
            <label @click="editTodo(item)">
              <a class="text">{{ item.title }}</a>
              <input
                type="text"
                class="edit"
                v-model="item.title"
                v-focus="isEditingItem(item)"
                @blur="editCompleted"
                @keyup.enter="editCompleted"
              >
            </label>
            <a class="delete" @click="deleteTodo(index)">-</a>
          </li>
        </ul>
      </section>

    </div>
  </div>
</template>

<script>
import * as Utils from '@/utils/utils'
let filter = {
  all: function (list) {
    return list
  },
  finished: function (list) {
    return list.filter((item) => {
      if (item.isChecked) {
        return true
      }
    })
  },
  unfinished: function (list) {
    return list.filter((item) => {
      if (!item.isChecked) {
        return true
      }
    })
  }
}
export default {
  name: 'List',
  data () {
    return {
      todo: '',
      list: [],
      actionItems: [
        {
          title: '全部',
          isActive: true,
          status: 'all'
        },
        {
          title: '已完成',
          isActive: false,
          status: 'finished'
        },
        {
          title: '未完成',
          isActive: false,
          status: 'unfinished'
        }
      ],
      hash: 'all',
      editingItem: null,
      initText: '',
      unfinishedCount: 0
    }
  },
  methods: {
    addTodo () {
      let todoObj = {
        title: this.todo,
        isChecked: false
      }
      if (this.todo.trim() !== '') {
        let listStorage = Utils.getItem('todoList')
        if (listStorage) {
          listStorage.push(todoObj)
          this.list = listStorage
          Utils.setItem('todoList', listStorage)
        } else {
          this.list.push(todoObj)
          Utils.setItem('todoList', this.list)
        }
      }
      this.todo = ''
    },
    deleteTodo (index) {
      this.list.splice(index, 1)
      Utils.setItem('todoList', this.list)
    },
    editTodo (item) {
      this.editingItem = item
    },
    // 判断是否为当前编辑元素
    isEditingItem (item) {
      return item === this.editingItem
    },
    editCompleted () {
      this.editingItem = null
      Utils.setItem('todoList', this.list)
    },
    checking (item) {
      Utils.setItem('todoList', this.list)
      console.log(item)
    },
    handleChange (item) {
      for (let i = 0, len = this.actionItems.length; i < len; i++) {
        this.actionItems[i].isActive = false
      }
      item.isActive = true
      this.hash = item.status
    }
  },
  computed: {
    showList () {
      let that = this
      let listStorage = Utils.getItem('todoList')
      if (listStorage && listStorage.length > 0) {
        that.list = listStorage
      }
      that.unfinishedCount = filter['unfinished'](that.list).length
      // if (that.hash === 'finished') {
      //   return that.finished(that.list)
      // } else if (that.hash === 'unfinished') {
      //   return that.unfinished(that.list)
      // } else {
      //   return that.list
      // }
      return filter[that.hash] ? filter[that.hash](that.list) : that.list
    }
  },
  directives: {
    focus: {
      update (el) {
        el.focus()
      }
    }
  }
}
</script>

<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: block;
}
a {
  color: #42b983;
}
.view-top {
  width: 100%;
  height: 40px;
  background-color: #5882dd;
  display: flex;
  justify-content: center;
  align-items: center;
}
.view-top h2 {
  font-size: 18px;
  color: #fff;
}
.main {
  margin: 0 12px;
}
.main section {
  margin-top: 25px;
}
.section-title {
  font-size: 16px;
  color: #666;
  margin-bottom: 10px;
}
.task-input {
  width: 100%;
  font-size: 14px;
  height: 24px;
  line-height: 24px;
  background: #fff;
  border-radius: 3px;
  text-indent: 10px;
}
.task-count {
  display: flex;
}
.task-count li:first-child {
  flex: 2;
  font-size: 14px;
  color: #dd4b39;
}
.action a {
  padding: 3px 5px;
  text-align: right;
  font-size: 14px;
  color: #666;
}
.action a.active {
  border: 1px solid #5882dd;
}
.todo-list li {
  height: 32px;
  line-height: 32px;
  background: #fff;
  margin-bottom: 10px;
  padding-left: 20px;
  border-radius: 3px;
  border-left: 5px solid #5882dd;
  box-shadow: 0 1px 2px rgba(0,0,0,0.07);
  position: relative;
}
.todo-list li.completed .text {
  color: #ccc;
  text-decoration: line-through;
}
.todo-list li .edit {
  display: none;
}
.todo-list li.editing .edit {
  display: inline-block;
  width: 200px;
  height: 20px;
  line-height: 20px;
  font-size: 16px;
}
.todo-list li.editing .text {
  display: none;
}
.todo-list li .text{
  color: #666;
}
.delete {
  display: inline-block;
  position: absolute;
  top: 4px;
  right: 5px;
  width: 12px;
  height: 12px;
  border-radius: 12px;
  border: 6px double #FFF;
  background: #CCC;
  line-height: 12px;
  text-align: center;
  color: #FFF;
  font-size: 12px;
  cursor: pointer;
}
</style>
