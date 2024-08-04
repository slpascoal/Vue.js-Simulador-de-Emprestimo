<template>
  <div>
    <h1>Simulador de Empréstimo</h1>
    <div>
      <label for="valor">Valor do Empréstimo</label>
      <input v-model="valor" id="valor" type="text" />
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

    <div v-if="resultadosSimulacao.length">
      <div v-for="(resultado, index) in resultadosSimulacao" :key="index" class="resultado">
        <img :src="logos[resultado.instituicao]" alt="Logo" v-if="logos[resultado.instituicao]" />
        <p>
          R$ {{ resultado.valor_solicitado }} - {{ resultado.parcelas }} x R$ {{ resultado.valor_parcela }}
        </p>
        <p>
          {{ resultado.instituicao }} ({{ indexParaConvenio[resultado.convenio_index] || 'Não disponível' }}) - {{ resultado.taxa_juros }}%
        </p>
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
        Pan: require('@/assets/imagens/Bancopanlogo.png'),
        bmg: require('@/assets/imagens/logo-bmg-nova.png'),
        ole: require('@/assets/imagens/Ole-consignado-2.webp')
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
.resultado {
  border: 1px solid #ddd;
  padding: 16px;
  margin-bottom: 16px;
}
</style>
