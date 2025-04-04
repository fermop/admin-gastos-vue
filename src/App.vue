<script setup>
  import { ref, reactive, watch, computed, onMounted } from 'vue'
  import Presupuesto from './components/Presupuesto.vue';
  import ControlPresupuesto from './components/ControlPresupuesto.vue';
  import Modal from './components/Modal.vue';
  import Gasto from './components/Gasto.vue';
  import Filtros from './components/Filtros.vue';

  import { generarId } from './helpers/'

  import iconoNuevoGasto from './assets/img/nuevo-gasto.svg'

  const modal = reactive({
    mostrar: false,
    animar: false
  })
  const presupuesto = ref(0)
  const disponible = ref(0)
  const gastado = ref(0)
  const filtro = ref('')
  
  const gasto = reactive({
    nombre: '',
    cantidad: '',
    categoria: '',
    id: null,
    fecha: Date.now()
  })

  const gastos = ref([])
  
  const guardarLocalStorage = () => {
    localStorage.setItem('gastos', JSON.stringify(gastos.value))
  }

  watch(gastos, () => {
    const totalGastado = gastos.value.reduce((total, gasto) => gasto.cantidad + total, 0)
    disponible.value = presupuesto.value - totalGastado
    gastado.value = totalGastado
    guardarLocalStorage()
  }, {
    deep: true
  })
  
  watch(modal, () => {
    if (!modal.mostrar) {
      // Reiniciar el objeto
      reiniciarStateGasto()
    }
  }, {
    deep: true
  })

  watch(presupuesto, () => {
    localStorage.setItem('presupuesto', presupuesto.value)
  })

  onMounted(() => {
    const presupuestoStorage = localStorage.getItem('presupuesto')
    if (presupuestoStorage) {
      presupuesto.value = Number(presupuestoStorage)
      disponible.value = Number(presupuestoStorage)
    }

    const gastosStorage = localStorage.getItem('gastos')
  
    if (gastosStorage) {
      gastos.value = JSON.parse(gastosStorage)
    }
  })

  const definirPresupuesto = (cantidad) => {
    presupuesto.value = cantidad
    disponible.value = cantidad
  }

  const mostrarModal = () => {
    modal.mostrar = true
    setTimeout(() => {
      modal.animar = true
    }, 300)
  }

  const ocultarModal = () => {
    modal.animar = false
    setTimeout(() => {
      modal.mostrar = false
    }, 300)
  }

  const guardarGasto = () => {
    if (gasto.id) {
      // Editando
      const { id } = gasto
      const i = gastos.value.findIndex((gasto) => gasto.id === id)
      gastos.value[i] = {...gasto}
    } else {
      // Nuevo registro
      gastos.value.push({
        ...gasto,
        id: generarId()
      })
    }
    ocultarModal()
    reiniciarStateGasto()
  }

  const reiniciarStateGasto = () => {
    Object.assign(gasto, {
      cantidad: '', 
      categoria: '',
      nombre: '',
      fecha: Date.now(),
      id: null
    })
  }

  const seleccionarGasto = id => {
    const gastoEditar = gastos.value.filter(gasto => gasto.id === id)[0]
    Object.assign(gasto, gastoEditar)
    mostrarModal()
  }

  const eliminarGasto = () => {
    if (confirm('¿Está seguro que desea eliminar el gasto?')) {
        gastos.value = gastos.value.filter(gastoState => gastoState.id !== gasto.id)
      ocultarModal()
    }
  }

  const gastosFiltrados = computed(() => {
    if (filtro.value) {
      return gastos.value.filter(gasto => gasto.categoria === filtro.value)
    }
    return gastos.value
  })

  const resetearApp = () => {
    if (confirm('¿Deseas reiniciar presupuestos y gastos?')) {
      presupuesto.value = 0
      gastos.value = []
    }
  }
</script>

<template>
  <div
    :class="{fijar: modal.mostrar}"
  >
    <header>
      <h1>Planificador de Gastos</h1>

      <div class="contenedor-header contenedor sombra">
        <Presupuesto
          v-if="presupuesto === 0"
          @definir-presupuesto="definirPresupuesto"
        />

        <ControlPresupuesto
          v-else
          :presupuesto="presupuesto"
          :disponible="disponible"
          :gastado="gastado"
          @resetear-app="resetearApp"
        />
      </div>
    </header>

    <main v-if="presupuesto">
      <Filtros
        v-model:filtro="filtro"
      />

      <div class="listado-gastos contenedor">
        <h2>{{ gastosFiltrados.length ? 'Gastos' : 'No hay gastos' }}</h2>

        <Gasto
          v-for="gasto in gastosFiltrados"
          :key="gasto.id"
          :gasto="gasto"
          @seleccionar-gasto="seleccionarGasto"
        />
      </div>

      <div 
        class="crear-gasto"
      >
        <img
          :src="iconoNuevoGasto"
          alt="Icono nuevo gasto"
          @click="mostrarModal"
        >
      </div>

      <Modal
        v-if="modal.mostrar"
        @ocultar-modal="ocultarModal"
        @guardar-gasto="guardarGasto"
        @eliminar-gasto="eliminarGasto"
        :modal="modal"
        :disponible="disponible"
        :id="gasto.id"
        v-model:nombre="gasto.nombre"
        v-model:cantidad="gasto.cantidad"
        v-model:categoria="gasto.categoria"
      />
    </main>
  </div>
</template>

<style>
  :root {
    --azul: #3b82f6;
    --blanco: #fff;
    --gris-claro: #f5f5f5;
    --gris: #94a3b8;
    --gris-oscuro: #64748b;
    --negro: #000;
  }

  html {
    font-size: 62.5%;
    box-sizing: border-box;
  }

  *, *::after, *::before {
    box-sizing: inherit;
  }

  body {
    font-size: 1.6rem;
    font-family: "Lato", sans-serif;
    background-color: var(--gris-claro);
  }

  h1 {
    font-size: 4rem;
  }

  h2 {
    font-size: 3rem;
  }

  .fijar {
    overflow: hidden;
    height: 100vh;
  }

  header {
    background-color: var(--azul);
  }

  header h1 {
    padding: 3rem 0;
    margin: 0;
    color: var(--blanco);
    text-align: center;
  }

  .contenedor {
    width: 90%;
    max-width: 80rem;
    margin: 0 auto;
  }

  .contenedor-header {
    margin-top: -5rem;
    transform: translateY(5rem);
    padding: 5rem;
  } 

  .sombra {
    box-shadow: 0px 10px 15px -3px rgba(0,0,0,0.1);
    background-color: var(--blanco);
    border-radius: 1.2rem;
    padding: 5rem;
  }

  .crear-gasto {
    position: fixed;
    bottom: 5rem;
    right: 5rem;
  }

  .crear-gasto img {
    width: 5rem;
    cursor: pointer;
    transition: all ease 250ms;
  }

  .crear-gasto img:hover {
    opacity: .7;
  }

  .listado-gastos {
    margin-top: 10rem;
  }

  .listado-gastos h2 {
    font-weight: 900;
    color: var(--gris-oscuro);
  }
</style>