<template>
  <Message :msg="msg" v-show="msg"/>
  <div id="burger-table" v-if="burgers">    
    <div id="burger-table-heading">
        <div class="order-id">#:</div>
        <div>Cliente:</div>
        <div>Pão:</div>
        <div>Carne:</div>
        <div>Opcionais:</div>
        <div>Ações:</div>
    </div>
    <div id="burger-table-rows">
        <div class="burger-table-row" v-for="burger in burgers" :key="burger.id">
            <div class="order-number">{{ burger.id }}</div>
            <div>{{ burger.nome }}</div>
            <div>{{ burger.pao }}</div>
            <div>{{ burger.carne }}</div>
            <div>
                <ul v-for="(opcional, index) in burger.opcionais" :key="index">
                    <li>{{ opcional }}</li>
                </ul>
            </div>
            <div>
                <select name="status" class="status" @change="updateBurger($event, burger.id)">
                    <option :value="s.tipo" v-for="s in status" :key="s.id" :selected="burger.status == s.tipo">
                      {{ s.tipo }}
                    </option>
                </select>
                <button class="delete-btn" @click="deleteBurger(burger.id)">Cancelar</button>
            </div>
        </div>
    </div>
  </div>
  <div v-else>
    <h2>Não há pedidos no momento!</h2>
  </div>
</template>

<script>
import Message from '../components/Message.vue'

export default {
    name: 'Dashboard',
    components: {
      Message
    },
    data() {
        return {
            burgers: null,
            status: [],
            msg: null
        }
    },
    methods: {
      async getPedidos() {
        this.burgers = await this.makeRequest("GET", 'http://localhost:3000/burgers');
        this.getStatus();
      },
      async getStatus() {
        this.status = await this.makeRequest("GET", 'http://localhost:3000/status');
      },
      async deleteBurger(id) {
        await this.makeRequest("DELETE", `http://localhost:3000/burgers/${id}`);
        this.showMessage(`O pedido nº ${id} foi cancelado com sucesso`);
        this.getPedidos();
      },
      async updateBurger(event, id) {
        const option = event.target.value;
        const headers = { "Content-Type" : "application/json" };
        const body = JSON.stringify({status: option});
        await this.makeRequest("PATCH", `http://localhost:3000/burgers/${id}`, headers, body);
        this.showMessage(`O pedido nº ${id} foi atualizado para ${option}`);
      },
      async makeRequest(method, url, headers, body) {
        const req = await fetch(url, {
          method: method,
          headers: headers,
          body: body
        });
        return await req.json();
      },
      showMessage(text, showTime = 3000) {
        this.msg = text;
        setTimeout(() => this.msg = null, showTime);
      }
    },
    mounted () {
        this.getPedidos();
    }

}
</script>

<style scoped>
    #burger-table {
    max-width: 1200px;
    margin: 0 auto;
  }
  #burger-table-heading,
  #burger-table-rows,
  .burger-table-row {
    display: flex;
    flex-wrap: wrap;
  }
  #burger-table-heading {
    font-weight: bold;
    padding: 12px;
    border-bottom: 3px solid #333;
  }
  .burger-table-row {
    width: 100%;
    padding: 12px;
    border-bottom: 1px solid #CCC;
  }
  #burger-table-heading div,
  .burger-table-row div {
    width: 19%;
  }
  #burger-table-heading .order-id,
  .burger-table-row .order-number {
    width: 5%;
  }
  select {
    padding: 12px 6px;
    margin-right: 12px;
  }
  .delete-btn {
    background-color: #222;
    color:#fcba03;
    font-weight: bold;
    border: 2px solid #222;
    padding: 10px;
    font-size: 16px;
    margin: 0 auto;
    cursor: pointer;
    transition: .5s;
  }
  
  .delete-btn:hover {
    background-color: transparent;
    color: #222;
  }
</style>