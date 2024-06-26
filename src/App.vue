<script setup>

import { ref, reactive, watch, computed, onMounted } from 'vue'
import Presupuesto from "./components/Presupuesto.vue";
import ControlPresupuesto from "./components/ControlPresupuesto.vue";
import Gasto from "./components/Gasto.vue";
import Modal from "./components/Modal.vue";
import Filtros from "./components/Filtros.vue";
import { generarId } from './helpers';

import iconoNuevoGasto from './assets/img/nuevo-gasto.svg'

const presupuesto = ref(0);
const disponible = ref(0);
const gastado = ref(0);
const filtro = ref("");

const gasto = reactive({
  nombre: "",
  cantidad: "",
  categoria: "",
  id: null,
  fecha: Date.now()
})

const gastos = ref([]);

const modal = reactive({
  mostrar: false,
  animar: false
})

watch(gastos, () => {
  localStorage.setItem("gastos", JSON.stringify(gastos.value));
  const totalGastado = gastos.value.reduce((total, gasto) => gasto.cantidad + total, 0);
  gastado.value = totalGastado;
  disponible.value = presupuesto.value - totalGastado;

}, {
  deep: true
})

watch(modal, () => {
  if (!modal.mostrar) {
    reinciarStateGasto();
  }
}, {
  deep: true
});
watch(presupuesto, () => {
  localStorage.setItem("presupuesto", presupuesto.value);
})

onMounted(() => {
  const presupuestStorage = localStorage.getItem("presupuesto");
  if(presupuestStorage){
    presupuesto.value = Number(presupuestStorage);
    disponible.value = Number(presupuestStorage);;
  }
  const gastosStorage = localStorage.getItem("gastos");
  if(gastosStorage){
    gastos.value = JSON.parse(gastosStorage);
  }
});



/**
 * Define el presupuesto y la cantidad disponible inicialmente
 * @param {int} cantidad 
 */
const definirPresupuesto = (cantidad) => {
  presupuesto.value = cantidad;
  disponible.value = cantidad;
}

/**
 * Muestra el modal
 */
const mostrarModal = () => {
  modal.mostrar = true;

  setTimeout(() => {
    modal.animar = true;
  }, 300);

}
/**
 * oculta Modal
 */
const ocultarModal = () => {

  modal.animar = false;

  setTimeout(() => {
    modal.mostrar = false;
  }, 300);
  reinciarStateGasto();
}

/**
 * Guarda un gasto
 */
const guardarGasto = () => {
  if (gasto.id) {
    const { id } = gasto;
    const i = gastos.value.findIndex((gasto) => gasto.id === id);
    gastos.value[i] = { ...gasto }

  } else {
    gastos.value.push({
      ...gasto,
      id: generarId()
    })
  }
  ocultarModal();
  reinciarStateGasto();
}
/**
 * Editar gastos
 * @param {int} id 
 */
const seleccionarGasto = (id) => {
  const gastoEditar = gastos.value.filter(gasto => gasto.id === id)[0];
  Object.assign(gasto, gastoEditar);
  mostrarModal();
}

/**
 * Reiniciar el State del Gasto
 */
const reinciarStateGasto = () => {
  Object.assign(gasto, {
    nombre: "",
    cantidad: "",
    categoria: "",
    id: null,
    fecha: Date.now()
  })
}

/**
 * Elimina Gasto
 */
const eliminarGasto = () => {
  if (confirm("¿Eliminar?")) {
    gastos.value = gastos.value.filter(gastoState => gastoState.id !== gasto.id);
    ocultarModal();
  }
}

const gastosFiltrados = computed(() => {
  if(filtro.value){
    return gastos.value.filter(gasto => gasto.categoria === filtro.value);
  }
  return gastos.value;
});

const resetApp = () => {
  if(confirm("¿Desear reiniciar presupuesto y gastos?")){
    gastos.value = [];
    presupuesto.value = 0
  }
  
}
</script>

<template>
  <div :class="{ fijar: modal.mostrar }">
    <header>
      <h1>Planificador de gastos</h1>

      <div class="contenedor-header contenedor sombra">

        <Presupuesto v-if="presupuesto == 0" @definir-presupuesto="definirPresupuesto" />

        <ControlPresupuesto 
          v-else 
          :presupuesto="presupuesto" 
          :disponible="disponible" 
          :gastado="gastado"
          @resetear-app="resetApp"
        />

      </div>


    </header>
    <main v-if="presupuesto > 0">
      <Filtros v-model:filtro="filtro" />

      <div class="listado-gastos contenedor">
        <h2>{{ gastosFiltrados.length > 0 ? "Gastos" : "No hay gastos" }}</h2>
        <Gasto v-for="gasto in gastosFiltrados" :key="gasto.id" :gasto="gasto" @seleccionar-gasto="seleccionarGasto" />
      </div>


      <div class="crear-gasto">
        <img :src="iconoNuevoGasto" alt="icono nuevo gasto" @click="mostrarModal">
      </div>
      <Modal v-if="modal.mostrar" @ocultar-modal=ocultarModal :modal="modal" @guardar-gasto=guardarGasto
        @eliminar-gasto="eliminarGasto" v-model:nombre="gasto.nombre" v-model:cantidad="gasto.cantidad"
        v-model:categoria="gasto.categoria" :disponible="disponible" :id="gasto.id" />
    </main>

  </div>

</template>

<style>
:root {
  --azul: #3b82f6;
  --blanco: #FFF;
  --gris-claro: #F5F5F5;
  --gris: #94A3B8;
  --gris-oscuro: #64748B;
  --negro: #000;
}

html {
  font-size: 62.5%;
  box-sizing: border-box;
}

*,
*:before,
*:after {
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
  box-shadow: 0px 10px 15px -3px rgba(0, 0, 0, 0.1);
  background-color: var(--blanco);
  border-radius: 1.2rem;
  padding: 5rem
}

.crear-gasto {
  position: fixed;
  bottom: 5rem;
  right: 5rem;

}

.crear-gasto img {
  width: 5rem;
  cursor: pointer
}

.listado-gastos {
  margin-top: 10rem;
}

.listado-gastos h2 {
  font-weight: 900;
  color: var(--gris-oscuro)
}
</style>
