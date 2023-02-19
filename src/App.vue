<script setup>
import {ref, onMounted, computed, watch} from 'vue'

const films = ref([])
const name = ref('')

const input_content = ref('')

const films_asc = computed(() => films.value.sort((a,b)=>{
  return b.createdAt - a.createdAt;
}))

const removefilm = (film) => {
   films.value = films.value.filter(f => f !== film)
}

const watchfilm = (film) => {
  film.done = !film.done
}

watch(films, (newVal) => {
  localStorage.setItem('films', JSON.stringify(newVal))
}, {deep:true})

watch(name, (newVal)=>{
  localStorage.setItem('name', newVal)
})

onMounted(()=>{
  name.value = localStorage.getItem('name') || ''
  films.value = JSON.parse(localStorage.getItem('films') || '[]')
})


const options = {
	method: 'GET',
	headers: {
		'X-RapidAPI-Key': 'c762f60be0msh3c01acac83d9209p1ecaeajsn8e492a62cc69',
		'X-RapidAPI-Host': 'imdb8.p.rapidapi.com'
	}
};

const addFilm = () => {
  if(input_content.value.trim() === '') 
    return

  fetch('https://imdb8.p.rapidapi.com/auto-complete?q=' + input_content.value.trim(), options)
	.then(response => response.json())
	.then(response => {
    const item = response.d[0];


      films.value.push({
        content: item.l,
        poster: item.i.imageUrl,
        year: item.y,
        players: item.s,
        rank: item.rank,
        done: false,
        createdAt: new Date().getTime()
      })
  })
	.catch(err => console.error(err));



  input_content.value = ''
}
</script>


<template>
  <main class="app">
      <section class="greeting">
        <h2 class="title">
          Merhaba, <input type="text" placeholder="*isminiz*" v-model="name"/>
        </h2>
      </section>

      <section class="create-todo">
        <h3>FİLM OLUŞTUR</h3>  
        
        <form @submit.prevent="addFilm">
          <h4>Hangi filmi eklemek istersin?</h4>  
          <input 
              type="text" 
              placeholder="fight club" 
              v-model="input_content" />
          <input type="submit" value="Ekle" />
        </form>
      </section>
      
      <section class="todo-list">
        <h3 v-bind:style="[(films.length > 0) ? 'display:block': 'display:none']">FİLMLERİNİZ</h3>  
        <div class="list">
          <div v-for="film in films_asc" :class="['todo-item ' + (film.done && 'done')]">
            <div class="todo-content">
              <div style="display: flex; align-items: center; padding: 10px;">
                <h2>{{ film.content }}</h2>
                <div class="actions" style="padding-left: 30px; gap: 10px;">

                <button class="delete" style="background-color: green; font-size: 0;" @click="watchfilm(film)">
                  <span class="material-symbols-outlined">done</span>
                </button>
                
                <button class="delete" style="font-size: 0;" @click="removefilm(film)">
                  <span class="material-symbols-outlined">delete</span>
                </button>
              </div>
            </div>
              <br/><hr/>
              <h5>Rank: {{ film.rank }}</h5>
              <h5>Players: {{ film.players }}</h5>
              <h5>Year: {{ film.year }}</h5>
              <hr/><br/>
              <img v-bind:src="film.poster" style="height: 500px;"/>
            </div>
          </div>
        </div>
      </section>
  </main>
</template>