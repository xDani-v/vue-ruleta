<template>
  <!-- Mostar datos -->
  <div class="d-flex justify-content-center mt-5">
    <div class="roulette">
      <div v-for="number in 37" :key="number" class="roulette-number" :class="getColor(number - 1)">
        {{ number - 1 }}
      </div>
    </div>
  </div>
  <div class="d-flex justify-content-center">
    <button type="button" class="btn btn-success" @click="empezarJuego">Cargar saldo</button>
  </div>
  <!-- Mostrar formulario -->
  <div class="d-flex justify-content-center">
    <form class="mt-5" @submit.prevent="submitForm">
      <div class="d-flex flex-row">
        <div>
          <label for="">Ingresa tu nombre</label>
          <input type="text" class="form-control" v-model="name" id="name">
        </div>
        <div>
          <label for="">Tu saldo</label>
          <input type="number" class="form-control" v-model="saldo" id="saldo">
        </div>
        <div>
          <label for="">Ingresa un monto a apostar</label>
          <input type="number" class="form-control" v-model="amount" id="amount">
        </div>
      </div>
      <div class="form-group">
        <label for="betSelect">Elige tu apuesta:</label>
        <select class="form-control" v-model="bet" id="betSelect">
          <option value="red">Rojo</option>
          <option value="black">Negro</option>
          <option value="green">Verde</option>
          <option value="pares">Pares</option>
          <option value="impares">Impares</option>
          <optgroup label="Número y color">
            <option v-for="number in 37" :key="number" :value="`${number - 1} ${getColor(number - 1)}`">
              {{ number - 1 }} ({{ getColor(number - 1) }})
            </option>
          </optgroup>
        </select>
      </div>
      <br>
      <button type="submit" class="btn btn-success">Apostar</button>
      <button type="button" class="btn btn-warning" @click="guardar">Guardar</button>
    </form>

  </div>
  <div class="text-center mt-5">
    <h2>Resultado de la ruleta:</h2>
    <h3 class="mt-3">
      <span class="badge bg-primary">Número: {{ this.respuesta.number }}</span>
      <span class="badge bg-success">Color: {{ this.respuesta.color }}</span>
    </h3>
    <h4 class="mt-3">
      <span class="badge bg-danger" v-if="this.isGanado !== ''">
        {{ this.isGanado ? "Has ganado" : "Has perdido, Intentalo nuevamente" }}
      </span>
    </h4>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      name: '',
      amount: '',
      saldo: '',
      bet: '',
      respuesta: '',
      isGanado: '',
    };
  },
  methods: {
    getColor(number) {
      if (number === 0) {
        return 'green';
      } else if (number % 2 === 0) {
        return 'red';
      } else {
        return 'black';
      }
    },
    async submitForm() {
      console.log(`Nombre: ${this.name}`);
      console.log(`Monto: ${this.amount}`);
      console.log(`Apuesta: ${this.bet}`);

      try {
        const response = await axios.get('https://localhost:7015/api/roulette/random');
        this.respuesta = response.data;
        console.log(response.data);
        if (this.bet === response.data.color || this.bet === response.data.number.toString()) {
          this.isGanado = true;
        } else if (this.bet === 'pares' && response.data.number % 2 === 0) {
          this.isGanado = true;
        } else if (this.bet === 'impares' && response.data.number % 2 !== 0) {
          this.isGanado = true;
        } else if (this.bet.includes(' ')) {
          const betNumber = this.bet.split(' ')[0];
          const betColor = this.bet.split(' ')[1];
          if (response.data.number.toString() === betNumber && response.data.color === betColor) {
            this.isGanado = true;
          } else {
            this.isGanado = false;
          }
        } else {
          this.isGanado = false;
        }
      } catch (error) {
        console.error(error);
      }
    },
    async empezarJuego() {
      const saldo = await axios.get(`https://localhost:7015/api/roulette/${this.name}/saldo`);
      this.saldo = saldo.data;
    },
    async guardar() {
      const usuario = {
        Nombre: this.name,
        Monto: parseFloat(this.amount),
      };

      try {
        const response = await axios.post('https://localhost:7015/api/roulette/save', usuario);
        const saldo = await axios.get(`https://localhost:7015/api/roulette/${this.name}/saldo`);
        this.saldo = saldo.data;
        this.bet = '';
        this.isGanado = '';
        this.respuesta = '';
        this.amount = '';
      } catch (error) {
        console.error(error);
      }
    },
  },
};
</script>

<style scoped>
.roulette {
  width: 300px;
  height: 300px;
  border-radius: 50%;
  display: flex;
  flex-wrap: wrap;
  align-content: center;
  justify-content: center;
}

.roulette-number {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 5px;
}

.green {
  background-color: green;
}

.red {
  background-color: red;
  color: white;
}

.black {
  background-color: black;
  color: white;
}
</style>