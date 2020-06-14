<template>
  <div id="app">
    <div v-bind:class="{dark: isDark}">
    <header class="header">
      <h1 class="header__h1">Канбан</h1>
      <div class="header__new-task">
        <h3 class="header__new-task__h3">Добавить новую задачу</h3>  
        <input type="text" v-model="name" class="header__new-task__input" placeholder="Описание задачи">
        <button @click="addPlan()" class="header__new-task__button">Добавить</button>
        <div v-if="isDark===false" class="header_theme-btn">
          <button @click="makeDark()" class="header_theme">Сменить тему 🌑</button>
        </div>
        <div v-else  class="header_theme-btn">
          <button @click="makeLight()" class="header_theme">Сменить тему ☀️</button>
        </div>
        </div>
        </header>
        <main>
          <div class="lists">
          <div class="lists_planned">
            <h3 class="lists_planned__h3">План ({{Plan.length}})</h3>
            <draggable class="list-group" :list="Plan" group="tasks" >
                <div class="lists_planned_cards" v-for="(Plan, index) in Plan" :key="index" >
                    <h3>Задача №{{Plan.num}}</h3>
                    <p>Описание: {{Plan.name}}</p>
                    <p>Ответственный: {{Plan.owner}}</p>
                    <p>Статус: В плане</p>
                    <div class="lists__buttons">
                        <button @click="showModalPlan(Plan.num)">🖊️</button>
                        <button @click="toProgress(index, Plan.name, Plan.owner,Plan.num)">✔️</button>
                        <button @click="DeletePlan(index)">❌</button>
                    </div>
                </div>
            </draggable>   
        </div>
        
        <div class="lists_progress">
            <h3 class="lists_progress__h3">В работе ({{Progress.length}})</h3>
            <draggable class="list-group" :list="Progress" group="tasks">
            <div class="lists_progress_cards" v-for="(Progress, index) in Progress" :key="index">
                    <h3>Задача №{{Progress.num}}</h3>
                    <p>Описание: {{Progress.name}}</p>
                    <p>Ответственный: {{Progress.owner}}</p>
                    <p>Дата и время начала: {{Progress.dt}}</p>
                    <p>Статус: В прогрессе</p>
                    <div class="lists__buttons">
                        <button @click="showModalProg(Progress.num)">🖊️</button>
                        <button v-on:click="toComp(index, Progress.name, Progress.owner, Progress.dt, Progress.num)">✔️</button>
                        <button @click="DeleteProg(index)">❌</button>
                    </div>
            </div>
            </draggable>
        </div>
        <div class="lists_completed">
            <h3 class="lists_completed__h3">Готово ({{Completed.length}})</h3>
             <draggable class="list-group" :list="Completed" group="tasks">
            <div class="lists_completed_cards" v-for="(Completed, index) in Completed" :key="index">
                <h3>Задача №{{Completed.num}}</h3>
                    <p>Описание: {{Completed.name}}</p>
                    <p>Ответственный: {{Completed.owner}}</p>
                    <p>Дата и время начала: {{Completed.dt}}</p>
                    <p>Дата и время окончания: {{Completed.endDt}}</p>
                    <!-- <p>Затраченное время: {{Math.round(Completed.wasted/(3,6e+6))}} часов, {{Math.round(Completed.wasted/60000)}} минут, {{Math.round(Completed.wasted)/1000}} секунд</p> -->
                    <p>Статус: Готово</p>
                    <div class="lists__buttons">
                        <button @click="showModalComp(Completed.num)">🖊️</button>
                        <button @click="DeleteComp(index)">❌</button>
                    </div>
            </div>
            </draggable>
        </div>
    </div>  
    <transition name="modal-fade" >
    <div class="modal-backdrop" v-show="isModalVisible">
      <div class="modal" role="dialog" aria-labelledby="modalTitle" aria-describedby="modalDescription">
        <header class="modal-header" id="modalTitle">
          <slot name="header">
            <div class="modal-h">
              <h1>Редактирование задач</h1>
              <button @click="closeModal()">✖️</button>
            </div>
          </slot>
        </header>
        <section class="modal-body" id="modalDescription">
          <slot name="body">
            <div class="main-edit">
              <form>
              <label for="name" id="label-name">Описание</label><br>
              <input v-model="name" type="text" name="name" placeholder="Введите описание задачи"><br>
              <label for="owner" id="label-owner">Ответственный</label><br>
              <input v-model="owner" type="text" name="owner" placeholder="Введите имя ответственного"><br>
              <label for="status" id="label-status">Статус</label><br>
              <select v-model="selected" name="status">
              <option v-for="option in options" v-bind:key="option.value">
                {{ option.text }}
              </option>
              </select><br>
              <span>Выбрано: {{ selected }}</span><br>
              <div v-if="selected==='В процессе'">
                <label for="dt" id="label-dt">Дата и время начала</label><br>
                <input v-model="dt" type="datetime-local" name="dt"><br>
              </div>
              <div v-if="selected==='Готово'">
                <label for="dt" id="label-dt">Дата и время начала</label><br>
                <input v-model="dt" type="datetime-local" name="dt"><br>
                <label for="newDt" id="label-newDt">Дата и время окончания</label><br>
                <input v-model="endDt" type="datetime-local" name="newDt"><br>
              </div>
            </form>  
            </div>
          </slot>
        </section>
        <footer class="modal-footer">
          <slot name="footer">
            <div class="modal-btn">
              <div v-if="planChange===true">
              <button v-if="selected==='В плане'" type="button" @click="PlanToPlan(), closeModal()">Сохранить</button>
              <button v-if="selected==='В процессе'" type="button" @click="PlanToProg(), closeModal()">Сохранить</button>
              <button v-if="selected==='Готово'" type="button" @click="PlanToComp(), closeModal()">Сохранить</button>
              </div>
              <div v-if="progChange===true">
              <button v-if="selected==='В плане'" type="button" @click="ProgToPlan(), closeModal()">Сохранить</button>
              <button v-if="selected==='В процессе'" type="button" @click="ProgToProg(), closeModal()">Сохранить</button>
              <button v-if="selected==='Готово'" type="button" @click="ProgToComp(), closeModal()">Сохранить</button>
              </div>
              <div v-if="compChange===true">
              <button v-if="selected==='В плане'" type="button" @click="CompToPlan(), closeModal()">Сохранить</button>
              <button v-if="selected==='В процессе'" type="button" @click="CompToProg(), closeModal()">Сохранить</button>
              <button v-if="selected==='Готово'" type="button" @click="CompToComp(), closeModal()">Сохранить</button>
              </div>
            </div>
          </slot>
        </footer>
      </div>
    </div>
  </transition> 
      </main>
  </div>
  </div>
</template>

<script>
import draggable from 'vuedraggable'
export default {
  name: 'App',
  components:{
    draggable
  },
  data () {
    return{
      name:'',
      ContPlan:0,
      ContProg:0,
      ContComp:0,
      Plan: [],
      Progress:[],
      Completed:[],
      status:'В плане',
      isModalVisible: false,
      planChange:false,
      progChange:false,
      compChange:false,
      isDark:false,
      selected: '',
      options: [
      { 
        text: 'В плане',
        value: 'В плане' },
      { 
        text: 'В процессе', 
        value: 'В процессе' },
      { 
        text: 'Готово', 
        value: 'Готово' 
      }
      ]
    }
  },

  created(){
    this.owner='Dale Vanderboom',
    this.counter=1
  },
  methods:{
    makeDark(){
      this.isDark=true
    },
    makeLight(){
      this.isDark=false
    },
    addPlan() {
      this.Plan.push({
      num:this.counter++,
      name:this.name,
      owner: this.owner,
      status: 'В плане',
      })
      this.name='',
      this.ContPlan+=1
    },
    showModalPlan() {
      this.isModalVisible = true,
      this.planChange=true
    },
    showModalProg() {
      this.isModalVisible = true,
      this.progChange = true
    },
    showModalComp() {
      this.isModalVisible = true,
      this.compChange = true
    },
    closeModal() {
      this.isModalVisible = false
    },
    DeletePlan(index){
      this.$delete(this.Plan, index),
      this.ContPlan-=1
    },
    DeleteProg(index){
      this.$delete(this.Progress, index),
      this.ContProg-=1
    },
    DeleteComp(index){
      this.$delete(this.Completed, index),
      this.ContComp-=1
    },
    toProgress(index, name, owner,num){
      this.ContPlan-=1,
      this.Progress.push({
        num:num,
        name: name,
        dt: new Date().toLocaleString(),
        owner: owner,
        status: 'В прогрессе'   
      }),
      this.ContProg+=1,
      this.$delete(this.Plan, index)
    },
    toComp(index, name, owner, dt, num){
      this.ContProg-=1,
      this.Completed.push({
        num:num,
        name: name,
        dt: dt,
        endDt:new Date().toLocaleString(),
        owner: owner,
        status: 'Готово' ,
        wasted: new Date().toLocaleString() - dt
      }),
      this.ContComp+=1,
      this.$delete(this.Progress, index)
    },
    PlanToPlan(num){
      this.Plan.splice(this.Plan),
      this.Plan.push({
        num:num,
        name:this.name,
        owner: this.owner,
        status: this.selected
      }),
      this.planChange=false
    },
    PlanToProg(num){
     this.Plan.splice(this.Plan),
      this.Progress.push({
        num:num,
        name:this.name,
        owner: this.owner,
        dt:this.dt,
        status: this.selected
      }),
      this.planChange=false,
      this.ContPlan-=1,
      this.ContProg+=1   
    },
    PlanToComp(num){
      this.Plan.splice(this.Plan),
      this.Completed.push({
        num:num,
        name:this.name,
        owner: this.owner,
        dt:this.dt,
        endDt:this.endDt,
        status: this.selected,
        wasted: this.endDt-this.dt
      }),
      this.planChange=false,
      this.ContPlan-=1,
      this.ContComp+=1  
    },
    ProgToPlan(num){
      this.Progress.splice(this.Progress),
      this.Plan.push({
        num:num,
        name:this.name,
        owner: this.owner,
        status: this.selected
      }),
      this.progChange=false,
      this.ContProg-=1,
      this.ContPlan+=1 
    },
    ProgToProg(num){
      this.Progress.splice(this.Progress),
      this.Progress.push({
        num:num,
        name:this.name,
        owner: this.owner,
        dt:this.dt,
        status: this.selected
      }),
      this.progChange=false 
    },
    ProgToComp(num){
      this.Progress.splice(this.Progress),
      this.Completed.push({
        num:num,
        name:this.name,
        owner: this.owner,
        dt:this.dt,
        endDt:this.endDt,
        status: this.selected,
        wasted: this.endDt-this.dt
      }),
      this.progChange=false,
      this.ContProg-=1,
      this.ContComp+=1 
    },
    CompToPlan(num){
      this.Completed.splice(this.Completed),
      this.Plan.push({
        num:num,
        name:this.name,
        owner: this.owner,
        status: this.selected
      }),
      this.compChange=false,
      this.ContComp-=1,
      this.ContPlan+=1
    },
    CompToProg(num){
      this.Completed.splice(this.Completed),
      this.Progress.push({
        num:num,
        name:this.name,
        owner: this.owner,
        dt:this.dt,
        status: this.selected
      }),
      this.compChange=false,
      this.ContComp-=1,
      this.ContProg+=1
    },
    CompToComp(num){
      this.Completed.splice(this.Completed),
      this.Completed.push({
        num:num,
        name:this.name,
        owner: this.owner,
        dt:this.dt,
        endDt:this.endDt,
        status: this.selected,
        wasted: this.endDt-this.dt 
      }),
      this.compChange=false  
    },
   
  }
   

  
}

</script>

<style>
#app{
  margin:-20px -8px 0 -8px;
  padding:0;
  font-family: Monospace;
  font-size: 20px;
  line-height: 1.5;
}
/* .header{
  border:1px solid blue;
} */
.header__h1{
  color:#1E90FF;
  margin:0.5em 0 0.5em 1em;
}
.header_theme-btn{
  padding:0 20px;
  display:flex;
  justify-content: flex-end;
}
.header_theme{
  border:none;
  height:35px;
  width:150px;
  background:#00BFFF;
  color:white;
  font-size:16px; 
  font-family: Monospace;
}
/* .header__new-task{
  border:1px solid green;
} */
.header__new-task__h3{
  margin:0.5em 0 0.5em 1em;
  color:#87CEFA;
}
.header__new-task__input{
  height:35px;
  width:300px;
  margin:0.5em 1.5em 0.5em 1.5em;
  border:1.5px solid #ADD8E6;
}
.header__new-task__button{
  border:none;
  height:35px;
  width:100px;
  background:#00BFFF;
  color:white;
  font-size:16px; 
  font-family: Monospace;
}
.lists{
    display:flex;
    justify-content:space-around;
    margin-top:50px;
}
.lists_planned,
.lists_progress,
.lists_completed{
    border:1.5px solid #ADD8E6;
    width:320px;
    min-height:460px;
    margin-right:50px;
    padding:20px;
}
.lists_planned__h3,
.lists_progress__h3,
.lists_completed__h3{
    color:#1E90FF;
}
.lists_planned_cards,
.lists_progress_cards,
.lists_completed_cards{
    border:1px solid #1E90FF;
    color:#00BFFF;
    background:#E6E6FA;
    padding:10px;
    font-size:18px;
    margin-bottom:20px;
    position:relative;
    z-index: 1;
}
.lists__buttons{
    display:flex;
    justify-content: space-between;
    margin-top:10px;
}
.lists__buttons button{
    background: #00BFFF;
    border:none;
    padding:10px;
}
.lest-group{
  min-height: 300px;
}
.modal-backdrop {
    position: fixed;
    z-index: 3;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    background-color: rgba(0, 0, 0, 0.3);
    display: flex;
    justify-content: center;
    align-items: center;
    color:#00BFFF;
  }
  .modal {
    background: #FFFFFF;
    box-shadow: 2px 2px 20px 1px;
    overflow-x: auto;
    display: flex;
    flex-direction: column;
  }
  .modal-header,
  .modal-footer {
    padding: 15px;
    display: flex;
  }
  .modal-header {
    border-bottom: 1px solid #eeeeee;
    color: #00BFFF;
    justify-content: space-between;
  }
  .modal-h{
  display: flex;
  justify-content: space-between;
  }
  .modal-h button{
  margin-left:20px;
  background: #00BFFF;
  border:none;
  padding:10px;
  height:50px;
  align-self: center;
  }
  .modal-footer {
    justify-content: center;
  }
  .modal-body {
    position: relative;
    padding: 20px 10px;
    justify-self: center;
  }
  .modal input{
    height:35px;
    width:300px;
    border:1.5px solid #ADD8E6;
    margin-bottom:20px;
  }
  .modal .modal-btn button{
    margin-top:10px;
    border:none;
    height:35px;
    width:100px;
    background:#00BFFF;
    color:white;
    font-size:16px; 
    font-family: Monospace;
  }
  .dark{
    background:#090605;
  }
  .dark .header_h1{
    color:#5b92cb;
  }
  .dark .header__new-task__input{
    border:1.5px solid #a4bfd9;
  }
  .dark .header__new-task__button{
    background: #1956be;
  }
  .dark .header_theme{
    background: #1956be;
  }
  .dark .lists_planned_cards,
  .dark .lists_progress_cards,
  .dark .lists_completed_cards{
    border:#1f112c;
    background:#cec0d9;
    color:#1956be;
  }
  .dark .lists__buttons button{
    background:#1956be;
  }
  .dark .modal{
    background:#cec0d9;
    color:#1956be;
  }
  .dark .modal-h{
    color:#1956be;
  }
  .dark .modal-h button{
    background:#1956be ;
  }
  .dark .modal .modal-btn button{
    background: #1956be;
  }
</style>