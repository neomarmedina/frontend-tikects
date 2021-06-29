<template>
  <div class="home mt-10 flex flex-col p-10">
    <div class="text-6xl w-full text-center mb-10">
      Boletas
    </div>
    <div class="boletas">
      <div class="tables flex flex-col">
        <div class="mb-5 flex justify-between">
          <span class="text-3xl">Compradores</span>
          <div>
            <button class="btn-custom mr-4 py-1 px-2" @click="openModalBuyerAdd">
              <i class="fas fa-plus"></i>
            </button>
          </div>
        </div>
        <table>
          <tr>
            <th class="text-center">ID</th>
            <th>Nombre</th>
          </tr>
          <tr v-for="buyer of buyers" :key="buyer.id" v-show="!loading_buyer">
            <th class="text-center">{{ buyer.id }}</th>
            <th>{{ buyer.name }} </th>
          </tr>
          <tr v-show="loading_buyer">
            <th colspan="3" class="text-center">Cargando...</th>
          </tr>
        </table>
      </div>
      <div class="tables flex flex-col">
        <div class="mb-5 flex justify-between">
          <span class="text-3xl">Boletas</span>
          <div>
            <button class="btn-custom mr-4 py-1 px-2" @click="openModalTikectAdd">
              <i class="fas fa-plus"></i>
            </button>
          </div>
        </div>
        <table>
          <tr>
            <th class="text-center">ID</th>
            <th>CITY</th>
            <th>STATUS</th>
            <th>ACTIONS</th>
          </tr>
          <tr v-for="titeck of titecks" :key="titeck.id" v-show="!loading_tikect">
            <th class="text-center">{{ titeck.id }}</th>
            <th>{{ titeck.name }} </th>
            <th>{{ titeck.status }} </th>
            <th>
              <button class="btn-custom mr-4 py-1 px-2" @click="openModalTikectAssign(titeck.id)">
                <i class="fas fa-ticket-alt"></i>
              </button>
            </th>
          </tr>
          <tr v-show="loading_tikect">
            <th colspan="4" class="text-center">Cargando...</th>
          </tr>
        </table>
      </div>
    </div>
    <div class="modal" v-show="modal">
      <div class="modal-content p-10">
        <button class="btn-modal" @click="closeModal">
          <i class="fas fa-times"></i>
        </button>
        <div class="add-buyer flex w-full flex-col" v-show="modal_buyer_add">
          <span>Registrar comprador</span>
          <input v-model="name_buyer" class="mt-4 border-2 px-2 py-1" type="text" placeholder="Nombre del compradoor">
          <button class="btn-modal-custom mt-6 py-2" @click="saveBuyer">Guardar</button>
        </div>
        <div class="add-tikect flex w-full flex-col" v-show="modal_tikect_add">
          <span>Agregar boleta</span>
          <input v-model="name_tikect" class="mt-4 border-2 px-2 py-1 mb-4" type="text" placeholder="Nombre de boleta">
          <span>Disponible</span>
          <select v-model="status" class="border-2 px-2 py-1 mt-4">
            <option value="1">Si</option>
            <option value="0">No</option>
          </select>
          <button class="btn-modal-custom mt-6 py-2" @click="saveTikect">Guardar</button>
        </div>
        <div class="assign-tikect flex w-full flex-col" v-show="modal_tikect_assign">
          <span>Asignar boleta</span>
          <span class="mt-4">Compradores</span>
          <select v-model.number="id_buyer" class="border-2 px-2 py-1 mt-4">
            <option v-for="buyer of buyers" :key="buyer.id" :value="buyer.id">{{ buyer.name }}</option>
          </select>
          <button class="btn-modal-custom mt-6 py-2" @click="assignTikect">Asignar</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const api = "http://localhost:8000/";

export default {
  name: 'Home',
  data() {
    return {
      buyers: [],
      titecks: [],
      id_tikect: 0,
      id_buyer: 1,
      name_buyer: "",
      name_tikect: "",
      status: 1,
      loading_buyer: true,
      loading_tikect: true,
      modal_buyer_add: false,
      modal_tikect_add: false,
      modal_tikect_assign: false,
      modal: false,
    }
  },
  methods: {
    async loadBuyers() {
      const data = await fetch(`${api}api/buyers`);
      const json = await data.json();

      this.loading_buyer = false;

      this.buyers = json;
    },
    async loadTitecks() {
      const data = await fetch(`${api}api/tikects`);
      const json = await data.json();

      this.loading_tikect = false;

      this.titecks = json;
    },
    openModalBuyerAdd() {
      this.modal = true;
      this.modal_buyer_add = true;
    },
    openModalTikectAdd() {
      this.modal = true;
      this.modal_tikect_add = true;
    },
    openModalTikectAssign(id) {
      this.id_tikect = id;
      this.modal = true;
      this.modal_tikect_assign = true;
    },
    async saveBuyer() {
      const resp = {
        name: this.name_buyer
      }

      await fetch(`${api}api/buyers`, {
        method: "POST",
        body: JSON.stringify(resp)
      })

      this.name_buyer = "";

      this.closeModal();

      await this.loadBuyers();
    },
    async saveTikect() {
      const resp = {
        name: this.name_tikect,
        status: Number(this.status)
      }

      await fetch(`${api}api/tikects`, {
        method: "POST",
        body: JSON.stringify(resp)
      });

      this.name_tikect = "";
      this.status = 1;

      this.closeModal();

      await this.loadTitecks();
    },
    async assignTikect() {
      const resp = {
        user_id: this.id_buyer,
        tikect_id: this.id_tikect
      }

      await fetch(`${api}api/tikects/assing`, {
        method: "POST",
        body: JSON.stringify(resp),
        headers: {
          "Content-Type": "application/json"
        }
      });

      this.id_tikect = 0;
      this.id_buyer = 1
      ;

      this.closeModal();
    },
    closeModal() {
      this.modal = false;
      this.modal_buyer_add = false;
      this.modal_tikect_add = false;
      this.modal_tikect_assign = false;
    }
  },
  async created() {
    await this.loadBuyers();
    await this.loadTitecks();
  }
}
</script>

<style>
.home {
  width: 60%;
  background: #fff;
}

.tables {
  width: 45%;
}

.boletas {
  width: 100%;
  display: flex;
  flex-flow: row;
  justify-content: space-between;
}

table {
  font-family: arial, sans-serif;
  border-collapse: collapse;
  width: 100%;
}

td, th {
  border: 1px solid #dddddd;
  text-align: left;
  padding: 8px;
}

tr:nth-child(even) {
  background-color: #dddddd;
}

.btn-custom {
  background-color: rgba(54, 115, 206, 0.678);
  color: #fff;
  border-radius: 5px;
}

.modal {
  z-index: 1000;
  background-color: rgba(0, 0, 0, .2);
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  top: 0;
  left: 0;
}

.modal-content {
  width: 20%;
  background-color: #fff;
  position: relative;
}

.btn-modal {
  position: absolute;
  top: -10px;
  right: -10px;
  width: 30px;
  height: 30px;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 100%;
  background-color: #000;
  color: #fff;
}

.btn-modal-custom {
  background-color: rgb(197, 197, 197);
  border-radius: 10px;
}
</style>