<template>
  <div class="container mx-auto p-6">
    <h1 class="text-2xl font-bold mb-4">Tasa de Cambio Dólar a Soles</h1>
    <div class="flex flex-col md:flex-row mb-4 my-2">
        <div class="mb-4 md:mb-0 md:mr-4">
            <label for="usdToPenExchangeRate" class="block text-sm font-medium text-gray-600">Tasa de cambio $ a PEN:</label>
            <input id="usdToPenExchangeRate" v-model.number="usdToPenExchangeRate" type="number" class="mt-1 p-2 border rounded-md w-full md:w-32" />
        </div>
        <div class="mb-4 md:mb-0 md:mr-4">
            <label for="pcompra" class="block text-sm font-medium text-gray-600">Precio de compra $:</label>
            <input name="pcompra" v-model.number="pcompra" type="number" class="mt-1 p-2 border rounded-md w-full md:w-32" />
        </div>
        <div class="mb-4 md:mb-0">
            <label for="pventa" class="block text-sm font-medium text-gray-600">Precio de venta $:</label>
            <input name="pventa" v-model.number="pventa" type="number" class="mt-1 p-2 border rounded-md w-full md:w-32" />
        </div>
    </div>

    <div class="tables-container flex justify-between mt-2">
      <table class="w-1/2 border border-gray-300">
        <thead>
          <tr>
            <th class="py-2 px-4 border-b text-center">Medida</th>
            <th class="py-2 px-4 border-b text-center">Precio compra</th>
            <th class="py-2 px-4 border-b text-center">Precio venta</th>
            <th class="py-2 px-4 border-b text-center">Ganancia</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(medida, index) in medidas" :key="index">
            <td class="py-2 px-4 border-b text-center">{{ medida }}</td>
            <td class="py-2 px-4 border-b text-center">S/{{ calcularPrecioCompra(cantidadxtoneladas[index]).toFixed(2) }}</td>
            <td class="py-2 px-4 border-b text-center">S/{{ calcularPrecioVenta(cantidadxtoneladas[index]).toFixed(2) }}</td>
            <td class="py-2 px-4 border-b text-center">S/{{ calcularGanancia(cantidadxtoneladas[index]).toFixed(2) }}</td>
          </tr>
        </tbody>
      </table>
      <div class="w-1/2 flex justify-center">
        <table class="w-2/5 border border-gray-300">
          <thead>
            <tr>
              <th class="py-2 px-4 border-b">Cantidad * Tonelada</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(cantidadxtonelada) in cantidadxtoneladas" :key="cantidadxtonelada">
              <td class="py-2 px-4 border-b text-center">{{ cantidadxtonelada }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import api from '../api';

export default {
  data() {
    return {
      usdToPenExchangeRate: null,
      medidas: ['5/8', '1/2', '3/8', '8mm', '6mm', '3/4'],
      cantidadxtoneladas: [75, 116, 208, 288, 516, 51],
      pcompra: localStorage.getItem('pcompra') || 0,
      pventa: localStorage.getItem('pventa') || 0,
    };
  },
  watch: {
    pcompra(newPcompra) {
      localStorage.setItem('pcompra', newPcompra);
    },
    pventa(newPventa) {
      localStorage.setItem('pventa', newPventa);
    },
  },
  mounted() {
    this.fetchExchangeRate();
  },
  methods: {
    calcularPrecioCompra(cantidadxtonelada) {
      return this.usdToPenExchangeRate * this.pcompra / cantidadxtonelada;
    },
    calcularPrecioVenta(cantidadxtonelada) {
      return this.usdToPenExchangeRate * this.pventa / cantidadxtonelada;
    },
    calcularGanancia(cantidadxtonelada) {
      const precioCompra = this.calcularPrecioCompra(cantidadxtonelada);
      const precioVenta = this.calcularPrecioVenta(cantidadxtonelada);
      return precioVenta - precioCompra;
    },
    fetchExchangeRate() {
      const lastFetch = localStorage.getItem('lastFetch');
      const now = new Date();

      if (lastFetch && now - new Date(lastFetch) < 1 * 60 * 60 * 1000) {
        // Menos de 5 horas desde la última llamada a la API, usar la tasa de cambio almacenada en localStorage
        this.usdToPenExchangeRate = localStorage.getItem('usdToPenExchangeRate');
      } else {
        // Más de 5 horas desde la última llamada a la API, llamar a la API de nuevo
        api.get('api/tipo-de-cambio/')
          .then(response => {
            this.usdToPenExchangeRate = response.data.tipo_de_cambio;
            this.usdToPenExchangeRate = Math.round(this.usdToPenExchangeRate * 100) / 100;
            localStorage.setItem('usdToPenExchangeRate', this.usdToPenExchangeRate);
            localStorage.setItem('lastFetch', now.toString());
          })
          .catch(error => console.error('Error al obtener la tasa de cambio:', error));
      }
    },
  },
};
</script>
