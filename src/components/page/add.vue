<!DOCTYPE html>
<html>

<head>
  <meta charset="UTF-8">
  <title></title>
  <link rel="stylesheet" href="css/vueCurd420.css" />
  <style>
    #grid-template,
    #dialog-template {
      display: none;
    }
  </style>
</head>

<body>
<div id="app">
  <div class="container">
    <div class="form-group">
      <label>Search</label>
      <input type="text" class="search-input" v-model="searchQuery" />
    </div>

  </div>
  <div class="container">
    <simple-grid :data-list="people" :columns="columns" :search-key="searchQuery">
    </simple-grid>
  </div>
</div>

<template id="grid-template">
  <table>
    <thead>
    <tr>
      <th v-for="col in columns">
        {{ col.name | capitalize}}
      </th>
      <th>
        Delete
      </th>
    </tr>
    </thead>
    <tbody>
    <tr v-for="(index,entry) in dataList | filterBy searchKey">
      <td v-for="col in columns">
        <span v-if="col.isKey"><a href="javascript:void(0)" @click="openEditItemDialog(entry[col.name])">{{entry[col.name]}}</a></span>
        <span v-else>{{entry[col.name]}}</span>
      </td>
      <td class="text-center">
        <button class="btn-danger" @click="deleteItem(entry)">delete</button>
      </td>
    </tr>
    </tbody>
  </table>
  <div class="container">
    <button class="btn" @click="openNewItemDialog('Create New Item')">Create</button>
  </div>

  <modal-dialog :mode="mode" :title="title" :item="item" :fields="columns" v-on:create-item="createItem" v-on:update-item="updateItem">
  </modal-dialog>

</template>

<template id="dialog-template">
  <div class="dialogs">
    <div class="dialog" v-bind:class="{ 'dialog-active': show }">
      <div class="dialog-content">
        <header class="dialog-header">
          <h1 class="dialog-title">{{ title }}</h1>
        </header>
        <div class="dialog-body">
          <div v-for="field in fields" class="form-group">
            <label>{{ field.name }}</label>
            <select v-if="field.dataSource" v-model="item[field.name]" :disabled="mode === 2 && field.isKey">
              <option v-for="opt in field.dataSource" :value="opt">{{ opt }}</option>
            </select>
            <input v-else type="text" v-model="item[field.name]" :disabled="mode === 2 && field.isKey">
          </div>
        </div>
        <footer class="dialog-footer">
          <div class="form-group">
            <label></label>
            <button class="btn-save" v-on:click="save">Save</button>
            <button class="btn-close" v-on:click="close">Close</button>
          </div>
        </footer>
      </div>
    </div>
    <div class="dialog-overlay"></div>
  </div>
</template>

<script src="js/vue.js"></script>
<script>
  Vue.component('simple-grid', {
    template: '#grid-template',
    props: ['dataList', 'columns', 'searchKey'],
    data: function() {
      return {
        mode: 0,
        title: '',
        keyColumn: '',
        item: {}
      }
    },
    ready: function() {
      for(var i = 0; i < this.columns.length; i++) {
        if(this.columns[i].isKey) {
          this.keyColumn = this.columns[i]['name']
          break;
        }
      }
    },
    methods: {
      openNewItemDialog: function(title) {
        // 对话框的标题
        this.title = title
        // mode = 1表示新建模式
        this.mode = 1
        // 初始化this.item
        this.item = {}
        // 广播事件，showDialog是modal-dialog组件的一个方法，传入参数true表示显示对话框
        this.$broadcast('showDialog', true)
      },
      openEditItemDialog: function(key) {
        // 根据主键查找当前修改的数据
        var currentItem = this.findItemByKey(key)
        // 对话框的标题
        this.title = 'Edit Item - ' + key
        // mode = 2表示修改模式
        this.mode = 2
        // 将选中的数据拷贝到this.item
        this.item = this.initItemForUpdate(currentItem)
        // 广播事件，传入参数true表示显示对话框
        this.$broadcast('showDialog', true)
      },
      // 弹出修改数据的对话框时，使用对象的深拷贝
      initItemForUpdate:function(p, c) {
    c = c || {};
    for(var i in p) {
      // 属性i是否为p对象的自有属性
      if(p.hasOwnProperty(i)) {
        // 属性i是否为复杂类型
        if(typeof p[i] === 'object') {
          // 如果p[i]是数组，则创建一个新数组
          // 如果p[i]是普通对象，则创建一个新对象
          c[i] = Array.isArray(p[i]) ? [] : {};
          // 递归拷贝复杂类型的属性
          this.initItemForUpdate(p[i], c[i]);
        } else {
          // 属性是基础类型时，直接拷贝
          c[i] = p[i];
        }
      }
    }
    return c;
  },
  findItemByKey: function(key) {
    var keyColumn = this.keyColumn
    for(var i = 0; i < this.dataList.length; i++) {
      if(this.dataList[i][keyColumn] === key) {
        return this.dataList[i]
      }
    }
  },
  itemExists: function() {
    var keyColumn = this.keyColumn
    for(var i = 0; i < this.dataList.length; i++) {
      if(this.item[keyColumn] === this.dataList[i][keyColumn])
        return true;
    }
    return false;
  },
  createItem: function() {
    var keyColumn = this.keyColumn
    if(!this.itemExists()) {
      // 将item追加到dataList
      this.dataList.push(this.item)
      // 广播事件，传入参数false表示隐藏对话框
      this.$broadcast('showDialog', false)
      // 新建完数据后，重置item对象
      this.item = {}
    } else {
      alert(keyColumn + ' "' + this.item[keyColumn] + '" is already exists')
    }

  },
  updateItem: function() {

    // 获取主键列
    var keyColumn = this.keyColumn

    for(var i = 0; i < this.dataList.length; i++) {
      // 根据主键查找要修改的数据，然后将this.item数据更新到this.dataList[i]
      if(this.dataList[i][keyColumn] === this.item[keyColumn]) {
        for(var j in this.item) {
          this.dataList[i][j] = this.item[j]
        }
        break;
      }
    }
    // 广播事件，传入参数false表示隐藏对话框
    this.$broadcast('showDialog', false)
    // 修改完数据后，重置item对象
    this.item = {}
  },
  deleteItem: function(entry) {
    var data = this.dataList
    data.forEach(function(item, i) {
      if(item === entry) {
        data.splice(i, 1)
        return
      }
    })
  }
  },
  components: {
    'modal-dialog': {
      template: '#dialog-template',
        data: function() {
        return {
          // 对话框默认是不显示的
          show: false
        }
      },
      /*
       * mode = 1是新增数据模式，mode = 2是修改数据模式
       * title表示对话框的标题内容
       * fields表示对话框要显示的数据字段数组
       * item是由simple-dialog传下来，用于绑定表单字段的
       */
      props: ['mode', 'title', 'fields', 'item'],
        methods: {
        close: function() {
          this.show = false
        },
        save: function() {
          if(this.mode === 1) {
            // 使用$dispatch调用simple-grid的create-item事件
            this.$dispatch('create-item')
          } else if(this.mode === 2) {
            // 使用$dispatch调用simple-grid的update-item事件
            this.$dispatch('update-item')
          }
        }
      },
      events: {
        'showDialog': function(show) {
          this.show = show
        }
      }
    }
  }
  })

  var demo = new Vue({
    el: '#app',
    data: {
      searchQuery: '',
      columns: [{
        name: 'name',
        isKey: true
      }, {
        name: 'age'
      }, {
        name: 'sex',
        dataSource: ['Male', 'Female']
      }],
      people: [{
        name: 'Jack',
        age: 30,
        sex: 'Male'
      }, {
        name: 'Bill',
        age: 26,
        sex: 'Male'
      }, {
        name: 'Tracy',
        age: 22,
        sex: 'Female'
      }, {
        name: 'Chris',
        age: 36,
        sex: 'Male'
      }]
    }
  })
</script>

</body>

</html>
