<template>
    <div>
        <MenssagemES :tipo="mensagem.tipo" :texto="mensagem.texto" v-if="mensagem.visivel" />

        <form id="pedido-form" @submit="criarPedido($event)">
            <div>
                <p id="nome-hamburguer-content">
                    {{ burguer && burguer.nome ? burguer.nome : "-" }}
                </p>
                <img id="foto-content" :src="burguer && burguer.foto ? burguer.foto : ''"/> 
            </div>
            <div class="inputs" id="form-pedido">
                <label for="nome_cliente">Nome</label>
                <input type="text"
                       id="nome-cliente"
                       name="nome-cliente"
                       v-model="nomeCliente"
                       placeholder="Digite seu Nome"/>
            </div>
            <div class="inputs">
                <label for="ponto-carne">Ponto da carne</label>
                <select
                    id="ponto-carne"
                    name="ponto-carne"
                    v-model="pontoCarneSelecionado">
                    <option value="" selected>Selecione o ponto</option>
                    <option v-for="pontoCarne in listaPontoCarne" 
                            :key="pontoCarne.id" 
                            :value="pontoCarne">{{ pontoCarne.descricao }}</option>
                </select>
            </div>
            <div id="opcionais-titulo" class="inputs">
                <label id="opcionais-titulo" for="Opcionais"> Selecione os opcionais</label>
                <label id="opcionais-subtitulo" for="Complemento">Adicione um complemento</label>
                
                <div class="checkbox-container"
                     v-for="complemento in listaComplementos"
                     :key="complemento.id">
                     <input type="checkbox" :name="complemento.nome"
                            v-model="listaComplementosSelecionados"
                            :value="complemento">
                     <span>{{ complemento.nome }}</span>       
                </div>
                
                <label for="Complemento">Adicione uma Bebida</label>

                <div class="checkbox-container" v-for="bebida in listaBebidas" :key="bebida.id">
                    <input type="checkbox" :name="bebida.nome" :value="bebida" v-model="listaBebidasSelecionadas"/>
                    <span>*{{ bebida.nome }}*</span>
                </div>
            </div>
            <div class="inputs">
                <input type="submit" class="submit-btn" value="Confirmar Pedido"/>
            </div>
        </form>

        <!-- Botão para ir para a lista de pedidos-->
        <button v-if="mostrarBotaoNavegar" @click="navegarParaListaPedidos">Ver Lista de Pedidos</button>
    </div>
</template>

<script>
import MenssagemES from './MenssagemES.vue';

export default {
    name: "PedidoComponent",
    components: {
        MenssagemES
    },
    data() {
        return {
            nomeCliente: "",
            pontoCarneSelecionado: "",
            listaPontoCarne: [],
            listaComplementos: [],
            listaBebidas: [],
            listaComplementosSelecionados: [],
            listaBebidasSelecionadas: [],
            mensagem: {
                tipo: '',
                visivel: false
            },
            texto: '',
            mostrarBotaoNavegar: false
        }
    },
    props: {
        burguer: null
    },
    methods: {
        async getTipoPontos() {
            const response = await fetch("http://localhost:3000/tipos_pontos");
            const data = await response.json();
            this.listaPontoCarne = data;
        },
        async getOpcionais() {
            const response = await fetch("http://localhost:3000/opcionais");
            const responseJson = await response.json();
            this.listaComplementos = responseJson.complemento;
            this.listaBebidas = responseJson.bebidas;
        },
        async criarPedido(e) {
            e.preventDefault();

            if (!this.nomeCliente || !this.pontoCarneSelecionado) {
                this.mostrarMensagem('erro', 'Preencha todos os campos obrigatórios!');
                return;
            }

            const dadosPedido = {
                nome: this.nomeCliente,
                ponto: this.pontoCarneSelecionado,
                bebidas: Array.from(this.listaBebidasSelecionadas),
                complementos: Array.from(this.listaComplementosSelecionados),
                statusId: 5,
                hamburguer: this.burguer
            };

            const dadosPedidoJson = JSON.stringify(dadosPedido);

            try {
                const requisicao = await fetch("http://localhost:3000/pedidos", {
                    method: "POST",
                    headers: { "Content-Type": "application/json" },
                    body: dadosPedidoJson
                });
                
                if (requisicao.ok) {
                    this.mostrarMensagem('sucesso', 'Pedido criado com sucesso!');
                    this.mostrarBotaoNavegar = true; // Exibe o botão pra navegar
                } else {
                    this.mostrarMensagem('erro', 'Erro ao criar o pedido!');
                }
            } catch (error) {
                this.mostrarMensagem('erro', 'Erro ao criar o pedido!');
            }
        },
        mostrarMensagem(tipo, texto) {
           this.mensagem.tipo = tipo;
           this.mensagem.texto = texto;
           this.mensagem.visivel = true;
           setTimeout(() => this.mensagem.visivel = false, 3000); // Oculta após 3 segundos
       },
        navegarParaListaPedidos() {
            //  navegar para a lista de pedidos
            this.$router.push({name: 'pedidos'}); 
        }
    },
    mounted() {
        this.getTipoPontos();
        this.getOpcionais();
    }
}
</script>



<style scoped>

#foto-content {
    margin-bottom: 16px;
    border-radius: 16px;
    position: relative;
    z-index: -1;
    justify-content: center;
    width: 100%;
    height: 180px;
    object-fit: cover
}

#nome-hamburguer-content {
    font-size: 43px;
    font-weight: bold;
    text-align: start;
    margin-bottom: -90px;
    margin-left: 40px;
    color: antiquewhite;
    padding: 16px;
}

#pedido-form {
    max-width: 750px;
    margin: 0 auto;
}

.inputs {
    display: flex;
    flex-direction: column;
    margin-bottom: 16px;
}

label {
    font-weight: bold;
    margin-bottom: 16px;
    color: #222;
    padding: 5px 12px;
    border-left: 4px solid darkgoldenrod;
    display: flex;
}

input, select {
    padding: 12px;
    width: 300px;
    border: #222 solid 1px;
    border-radius: 8px;
    height: 20px;
    height: px;
    font-size: 12px;
}

#obname{
    padding: 9px;
    margin: 5px;
    width: 130px;
    border: #ff0000d7 solid 2px;
    border-radius: 25px;
    height: 18px;
    font-size: 13px;
    color: rgb(255, 255, 255);
    font-weight: bold;
    background-color: rgb(255, 49, 49);
    
}
select {
    height: 50px;
}


#opcionais-titulo {
    width: 100%;
}

#opcionais-subtitulo {
    font-weight: bold;
    color: #222;
    border-left: 4px solid darkslategrey;
}

.checkbox-container {
    display: flex;
    align-items: flex-start;
    align-content: center;
    width: 100%;
    margin-bottom: 16px;

}

.checkbox-container span,
.checkbox-container input {
    width: auto;
    height: 20px;
}

.checkbox-container span {
    margin-left: 6px;
    font-weight: bold;
}

.submit-btn {
    background-color: #222;
    color: darkgoldenrod;
    font-weight: bold;
    border: solid 2px darkgoldenrod;
    border-radius: 8px;
    cursor: pointer;
    padding: 12px;
    margin: 0 auto;
    font-size: 16px;
    width: 100%;
    height: auto;
    transition: 0.5s;
}

.submit-btn.submit-btn:hover {
    background-color: transparent;
    color: #222;
}


button{
    background-color: rgb(126, 196, 23);
    border-radius: 10px;
    border-color: rgb(126, 201, 15);
    color: aliceblue;
    padding: 8px;
    margin-bottom: 5px;
}

</style>