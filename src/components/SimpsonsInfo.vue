<script setup>
import { ref, watch, computed, onMounted } from 'vue'

const results = ref([])
const loading = ref(false)
const increment = ref(1)
const personaje = ref(null)
const favoritos = ref([])
const dialogVisible = ref(false)

const fetchSimpsons = async () => {
  loading.value = true
  try {
    const response = await fetch(
      `https://thesimpsonsapi.com/api/characters?page=${increment.value}`,
    );
    const data = await response.json()
    results.value = data.results

  } catch (e) {
    console.error(e)
  } finally {
    loading.value = false
  }
}


const retrocederPagina = () => {
  if (increment.value > 1) increment.value--
}

const personajesFiltrados = computed(() => {
  if (personaje.value === null || personaje.value === '') {
    return results.value
  }
  return results.value.filter((result) =>
    result.name.toLowerCase().includes(personaje.value.toLowerCase()),
  )
})

async function agregarFavorito(id) {
  const actual = results.value.find(result => result.id == id)
  if (actual && !favoritos.value.find(fav => fav.id === id)) {
    try {
      const res = await fetch('http://localhost:3000/favorites', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        id: actual.id,
        name: actual.name,
        age: actual.age,
        occupation: actual.occupation,
        status: actual.status
      })
    })
    const result = await res.json()
    favoritos.value.push(result)
    
    if(res.ok) {
      console.log('Favorito agregado', result)
    }
    else {
      console.error(result.message)
    }
    } catch (e) {
      console.error(e)
    }
  }
}

async function eliminarFavorito(id) {
  try {
    const res = await fetch(`http://localhost:3000/favorites/${id}`, {
      method: 'DELETE'
    });
    console.log("status de respuesta:", res.status)
    const data = await res.json()
    if(res.ok) {
      console.log('Favorito eliminado')
      favoritos.value = favoritos.value.filter(fav => fav.id !== id)
    } else {
      console.error(data.message)
    }
  } catch (e) {
    console.error(e)
  }
}

function consultarFavoritos() {
  dialogVisible.value = true
}

async function cargarFavoritos() {
  try {
    const res = await fetch('http://localhost:3000/favorites')
    favoritos.value = await res.json()
  } catch (e) {
    console.error(e)
  }
}

onMounted(() => {
  cargarFavoritos()
  fetchSimpsons()
}),

watch(increment, () => {
  fetchSimpsons()
})
</script>

<template>
  <p v-if="loading">Cargando...</p>
  <div class="flex gap-2 justify-center w-xl my-5">
    <el-input v-model="personaje" placeholder="Ingrese un nombre" class="w-8"></el-input>
    <el-button type="success">Buscar</el-button>
    <el-button type="warning" @click="consultarFavoritos">Favoritos</el-button>
  </div>
  <el-button type="primary" size="large" @click="retrocederPagina" :disabled="increment === 1"
    >Previous page
  </el-button>
  <el-button @click="increment++" type="primary" size="large">Next page</el-button>
  <div class="flex justify-center gap-3 p-10 px-48 flex-wrap">
    <div
      v-for="simpson in personajesFiltrados"
      :key="simpson.id"
      class="flex flex-col items-center p-2 hover:-translate-y-0.5 w-3xs duration-100 ease-in-out hover:border-2 hover:border-amber-400 hover:shadow-xl hover:shadow-amber-400"
    >
      <div class="flex w-full justify-end">
        <el-icon
          size="20"
          color="yellow"
          v-if="favoritos.find(fav => fav.id === simpson.id)"
          @click="eliminarFavorito(simpson.id)" class="cursor-pointer">
          <StarFilled />
        </el-icon>
        <el-icon v-else size="20" @click="agregarFavorito(simpson.id)" class="cursor-pointer">
          <Star />
        </el-icon>
      </div>
      <img
        :src="`https://cdn.thesimpsonsapi.com/500/character/${simpson.id}.webp`"
        :alt="`Imagen de ${simpson.name}`"
        :width="200"
      />
      <h2 class="mb-2 font-bold">{{ simpson.name }}</h2>
      <h4 class="text-sm text-gray-700 mb-3 line-clamp-2">Occupation: {{ simpson.occupation }}</h4>
      <div class="flex gap-10 justify-center mb-3">
        <h5 class="text-xs font-bold">Age: {{ simpson.age }}</h5>
        <h5
          :class="`text-xs font-bold ${simpson.status === 'Alive' ? 'text-green-500' : 'text-red-600'}`"
        >
          {{ simpson.status }}
        </h5>
      </div>
      <h6 class="text-xs text-gray-600 line-clamp-2 font-light italic">
        "{{ simpson.phrases[0] }}"
      </h6>
    </div>
  </div>
  <el-dialog v-model="dialogVisible" title="Favoritos" width="900">
    <el-table :data="favoritos" stripe v-if="favoritos.length > 0" >
      <el-table-column prop="id" label="ID" width="100"/>
      <el-table-column prop="name" label="Name" width="180" />
      <el-table-column prop="age" label="Age" width="100" />
      <el-table-column prop="occupation" label="Occupation" width="200" />
      <el-table-column prop="status" label="Status" width="150" />
      <el-table-column label="Operations" width="180">
        <template #default="scope">
          <el-button
            type="danger"
            size="mini"
            @click="eliminarFavorito(scope.row.id)"
          >
            <el-icon><Delete /></el-icon>
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <div v-else>
      <p>No hay favoritos guardados.</p>
    </div>
  </el-dialog>
</template>
