<script setup>
import { computed } from "vue";
import CicleProgress from "vue3-circle-progress";
import "vue3-circle-progress/dist/circle-progress.css";
import { formatearCantidad } from "../helpers";



const props = defineProps({
    presupuesto: {
        type: Number,
        required: true
    },
    disponible: {
        type: Number,
        required: true
    },
    gastado: {
        type: Number,
        required: true
    }

})

const porcentaje = computed(() => {
  return  parseInt(((props.presupuesto - props.disponible) / props.presupuesto) * 100);
})
defineEmits(["resetar-app"])

</script>

<template>
    <div class="dos-columnas">
        <div class="contenedor-grafico">
            <p class="porcentaje">{{ porcentaje }}%</p>
            <CicleProgress :percent="porcentaje" :size="250" :border-width="20" :border-bg-width="20" fill-color="#3b82f6"
                empty-color="#e1e1e1" />

        </div>
        <div class="contenedor-presupuesto">
            <button class="reset-app" type="button" @click="$emit('resetear-app')">Resetear App</button>
            <p>
                <span>Presupuesto: </span>
                {{ formatearCantidad(presupuesto) }}
            </p>
            <p>
                <span>Disponible: </span>
                {{ formatearCantidad(disponible) }}
            </p>
            <p>
                <span>Gastado: </span>
                {{ formatearCantidad(gastado) }}
            </p>

        </div>

    </div>
</template>



<style scoped>
.contenedor-grafico {
    position:relative;
}
.porcentaje {
    position: absolute;
    margin: auto;
    top: calc(50% - 1.5rem);
    left: 0;
    right: 0;
    text-align: center;
    z-index: 100;
    font-size: 3rem;
    font-weight: 900;
    color: var(--gris-oscuro);
}
.dos-columnas {
    display: flex;
    flex-direction: column;
    align-items: center;
}

.dos-columnas> :first-child {
    margin-bottom: 3rem;
}

@media (min-width: 768px) {
    .dos-columnas {
        flex-direction: row;
        gap: 4rem;
        align-items: center;
    }

    .dos-columnas> :first-child {
        margin-bottom: 0;
    }
}
.reset-app {
    background-color: #DB2777;
    border: none;
    padding: 1rem;
    width: 100%;
    color: var(--blanco);
    font-weight: 900;
    text-transform: uppercase;
    border-radius: 0.5rem;
    transition: background-color 300ms ease;

}
.reset-app:hover {
    cursor: pointer;
    background-color: #a11e59;

}

.contenedor-presupuesto {
    width: 100%;
}

.contenedor-presupuesto p {
    font-size: 2.4rem;
    text-align: center;
    color: var(--gris-oscuro);
}

@media(min-width: 768px) {
    .contenedor-presupuesto p {
        text-align: left;
    }
}

.contenedor-presupuesto span {
    font-weight: 900;
    color: var(--azul);
}
</style>