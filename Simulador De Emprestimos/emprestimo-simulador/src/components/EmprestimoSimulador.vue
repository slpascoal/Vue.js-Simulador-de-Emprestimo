<template>
  <div class="bodyApp">
    <h1>Simulador de Empréstimo</h1>
    <div class="form">
      <div class="valorEmp">
        <label for="valor">Valor do Empréstimo: </label>
        <span>R$<input v-model="valor" id="valor" type="text" /></span>
      </div>

      <Multiselect
        v-model="instituicaoSelecionada"
        :options="instituicoes"
        label="valor"
        placeholder="Selecione uma instituição"
      />

      <Multiselect
        v-model="convenioSelecionado"
        :options="convenios"
        label="valor"
        placeholder="Selecione um convênio"
      />

      <div>
        <label for="parcelas">Parcelas</label>
        <select v-model="parcelasSelecionadas" id="parcelas">
          <option v-for="parcela in opcoesParcelas" :key="parcela" :value="parcela">
            {{ parcela === 0 ? 'Selecionar parcelas' : parcela }}
          </option>
        </select>
      </div>

      <button @click="simularEmprestimo">Simular</button>
    
    </div>
    

    <div v-if="resultadosSimulacao.length" class="areaResul">
      <div v-for="(resultado, index) in resultadosSimulacao" :key="index" class="resultado">
        <div>
          <img :src="logos[resultado.instituicao]" alt="Logo" v-if="logos[resultado.instituicao]" width="50"/>
        </div>
        
        <div>
          <p>
            R$ {{ resultado.valor_solicitado <= 0 ? '0' :  resultado.valor_solicitado}} - {{ resultado.parcelas }} x R$ {{ resultado.valor_parcela  }}
          </p>
          <p>
            
            {{ resultado.instituicao }} - {{ resultado.taxa_juros }}%
          </p>
      </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import Multiselect from 'vue-multiselect'
import 'vue-multiselect/dist/vue-multiselect.css'

export default {
  components: { Multiselect },
  data() {
    return {
      valor: '',
      instituicoes: [],
      convenios: [],
      instituicaoSelecionada: null,
      convenioSelecionado: null,
      parcelasSelecionadas: null,
      opcoesParcelas: [0, 36, 48, 60, 72, 84],
      resultadosSimulacao: [],
      indexParaConvenio: {},
      logos: {
        PAN: require('@/assets/imagens/Bancopanlogo.png'),
        BMG: require('@/assets/imagens/logo-bmg-nova.png'),
        OLE: require('@/assets/imagens/Ole-consignado-2.webp')
      }
    }
  },
  mounted() {
    this.fetchInstituicoes()
    this.fetchConvenios()
  },
  methods: {
    async fetchInstituicoes() {
      try {
        const response = await axios.get('http://127.0.0.1:8000/api/instituicao')
        this.instituicoes = response.data.map(instituicao => ({
          chave: instituicao.chave.toString(),
          valor: instituicao.valor.toString()
        }))
      } catch (error) {
        console.error('Erro ao carregar instituições:', error)
      }
    },
    async fetchConvenios() {
      try {
        const response = await axios.get('http://127.0.0.1:8000/api/convenio')
        const convenios = response.data.map(convenio => ({
          chave: convenio.chave.toString(),
          valor: convenio.valor.toString()
        }))
        this.indexParaConvenio = convenios.reduce((acc, convenio, index) => {
          acc[index] = convenio.valor
          return acc
        }, {})
        this.convenios = convenios
      } catch (error) {
        console.error('Erro ao carregar convênios:', error)
      }
    },
    async simularEmprestimo() {
      try {
        const valorNumerico = parseFloat(this.valor.replace('R$', '').replace('.', '').replace(',', '.'))
        const response = await axios.post('http://127.0.0.1:8000/api/simular', {
          valor_emprestimo: valorNumerico,
          instituicoes: this.instituicaoSelecionada ? [this.instituicaoSelecionada.chave] : [],
          convenios: this.convenioSelecionado ? [this.convenioSelecionado.chave] : [],
          parcela: this.parcelasSelecionadas
        })
        this.resultadosSimulacao = response.data
      } catch (error) {
        console.error('Erro ao simular empréstimo:', error)
      }
    }
  }
}
</script>

<style scoped>
*{
  font-family: sans-serif;
  margin: 0;
  padding: 0;
}

.form{
  margin: 20px 0px;
  border-bottom: 2px solid #5050502c;
  border-radius: 10px;
  width: 80%;
  background-color: #00000009;
  align-self: center;

  text-align: center;

  

  padding: 0px 20px;
}

.bodyApp{
  display: flex;
  flex-direction: column;
}

.bodyApp h1{
  color: white;
  background-color: #41b883;
  padding: 20px;
}

.valorEmp{
  padding: 20px;
}

.form label{
  
  box-sizing: border-box;
  padding-bottom: 5px;
  
  font-size: 20px;
}

.valorEmp span{
  font-size: 20px;
}

.valorEmp input{
  border: 1px solid #41b883;
  border-radius: 10px;
  padding: 10px;
  width: 70%;
}

.form select{
  border: 1px solid #41b883;
  border-radius: 10px;
  padding: 10px;
  width: 70%;
  margin: 0px 10px;
}

.areaResul{
  display: flex;
  flex-direction: column;
}

.multiselect{
  margin-bottom: 20px; 
  border: 1px solid #41b883;
  border-radius: 5px;
}

.form button{
  margin: 20px;
  width: 20%;
  padding: 10px;

  border: 0px solid #41b883;
  background-color: #41b883;
  color: white;
  border-radius: 10px;
  transition: 0.5s;
}

.form button:hover{
  background-color: #41b883c5;
  transition: 0.5s;
  transform: scale(1.03);
}

.resultado {
  width: 75%;
  align-self: center;

  display: flex;
  gap: 20px;
  border-bottom: 2px solid #5050502c;
  border-radius: 10px;
  padding: 16px;
  margin-bottom: 16px;
  background-color: #00000009;
}

.resultado img {
  margin-top: 10%;
}

.resultado p{
  line-height: 20px;
}

.resultado p:first-child{
  font-weight: bold;
}
</style>
