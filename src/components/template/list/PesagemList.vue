<template>
    <div class="pesagemList">

        <b-row>
            <b-col cols="1" />
            <b-col cols="6">
                <b-input v-model="filter" placeholder="Buscar..."></b-input>
            </b-col>
            <b-col cols="1" />
            <b-col cols="4">
                <b-button v-b-modal.modal-cadastrar-peso variant="primary">Cadastrar</b-button>
            </b-col>
        </b-row>
        
        <hr />

        <!-- Tabela Pesagem -->
        <ul>
            <b-table hover striped :items="pesagens" :fields="fields" :current-page="currentPage" :per-page="5" :filter="filter">          
                <template #cell(actions)="data">
                    <b-button variant="warning" v-b-modal.modal-editar-peso @click="loadPesagem(data.item)" class="mr-2">
                        <i class="fa fa-pencil"></i>
                    </b-button>
                    <b-button variant="danger" v-b-modal.modal-excluir-peso @click="loadPesagem(data.item)">
                        <i class="fa fa-trash"></i>
                    </b-button>
                </template>
            </b-table>
        </ul>

        <ul class="justify-content-center row my-1">
            <b-pagination size="md" :total-rows="this.pesagens.length" :per-page="5" v-model="currentPage" />
        </ul>

        <!-- Cadastro Peso -->
        <b-row>
            <b-modal
            id="modal-cadastrar-peso"
            ref="modal"
            title="Cadastro de Peso"
            size="lg"
            @show="resetModal"
            @hidden="resetModal"
            @ok="cadastroOk">

            <form ref="form" @submit.stop.prevent="handleSubmit">
                
                <b-container>
                    <b-row>
                        <b-col>
                            <b-form-group
                                label="Nome*"
                                invalid-feedback="O nome é obrigatório">

                                <b-form-select
                                    v-model="nomeAnimal"
                                    :state="nomeAnimalState"
                                    required >
                                    <b-form-select-option v-for="animal of animais" :key="animal.id" :value="animal.id">
                                        {{animal.brinco}} - {{animal.nomeAnimal}}
                                    </b-form-select-option>

                                </b-form-select>
                            </b-form-group>
                        </b-col>
                    </b-row>
                    <b-row>
                        <b-col>
                            <b-form-group
                                label="Peso* (Kg)"
                                invalid-feedback="O Peso é obrigatório">

                                <b-form-input
                                    type="tel" 
                                    v-mask="'###'"
                                    v-model="peso"
                                    :state="pesoState"
                                    required />

                            </b-form-group>
                        </b-col>
                        <b-col>    
                            <b-form-group
                                label="Data de Pesagem*"
                                invalid-feedback="A data da Pesagem é obrigatório">
                                <b-form-input
                                    type = "date"
                                    class="today"
                                    v-model="dataPesagem"
                                    :state="dataPesagemState"
                                    :max="max"
                                    required />
                            </b-form-group>
                        </b-col>
                    </b-row>
                    <br>
                    <b-alert show variant="danger">Campos Obrigatorios (*)</b-alert>
                </b-container>
            </form>

            <template #modal-footer="{ ok, cancel }">
                <b-button size="md" variant="success" @click="ok()">
                    Confirmar
                </b-button>
                <b-button size="md" variant="danger" @click="cancel()">
                    Cancelar
                </b-button>
            </template>

            </b-modal>
        </b-row>

        <!-- Editar Peso -->
        <b-row>
            <b-modal
            id="modal-editar-peso"
            ref="modal"
            title="Editar Peso do animal"
            size="lg"
            @show="resetModal"
            @hidden="resetModal"
            @ok="editarOk">

            Deseja realmente editar o peso do animal <strong>{{pesagem.nomeAnimal}}</strong> ?
            <br><br>

            <form ref="form" @submit.stop.prevent="cadastrar">
                
                <b-container>
                    <b-row>
                        <b-col>
                            <b-form-group
                                label="Nome*">

                                <b-form-input 
                                    v-model="pesagem.nomeAnimal"
                                    disabled />
                            </b-form-group>
                        </b-col>
                    </b-row>
                    <b-row>
                        <b-col>
                            <b-form-group
                                label="Peso* (kg)"
                                invalid-feedback="O Peso é obrigatório">

                                <b-form-input
                                    type="tel" 
                                    v-mask="'###'"
                                    v-model="pesagem.peso"
                                    :state="pesoState"
                                    required />
                                    
                            </b-form-group>
                        </b-col>
                        <b-col>    
                            <b-form-group
                                label="Data de Pesagem*"
                                invalid-feedback="A data da Pesagem é obrigatório">
                                <b-form-input
                                    type = "date"
                                    v-model="pesagem.dataPesagem"
                                    :max="max"
                                    required
                                    :state="dataPesagemState" />
                            </b-form-group>
                        </b-col>
                    </b-row>
                    <br>
                    <b-alert show variant="danger">Campos Obrigatorios (*)</b-alert>
                </b-container>

            </form>

            <template #modal-footer="{ ok, cancel }">
                <b-button size="md" variant="success" @click="ok()">
                    Confirmar
                </b-button>
                <b-button size="md" variant="danger" @click="cancel()">
                    Cancelar
                </b-button>
            </template>

            </b-modal>
        </b-row>

        <!-- Excluir Peso -->
        <b-row>
            <b-modal
                id="modal-excluir-peso"
                title="Excluir Peso" 
                size="lg"
                @ok="remove">
                    <br>
                    Deseja realmente excluir o peso do animal <strong>{{pesagem.nomeAnimal}}</strong> ?

            <template #modal-footer="{ ok, cancel }">
                <b-button size="md" variant="success" @click="ok()">
                    Confirmar
                </b-button>
                <b-button size="md" variant="danger" @click="cancel()">
                    Cancelar
                </b-button>
            </template>
            
            </b-modal>
        </b-row>

        <!-- Help de Contexto -->
        <PesagemHelp />

    </div>
</template>

<script>
import axios from "axios";
import {baseApiUrl} from '@/global'
import moment from 'moment'

import PesagemHelp from "../help/PesagemHelp";

export default {
    name: "PesagemList",
    components: { PesagemHelp },

    data () {
        const now = new Date()
        const today = new Date(now.getFullYear(), now.getMonth(), now.getDate())

        const maxDate = new Date(today)
        var maxDateMoment = moment(maxDate, "DD/MM/YYYY");

        return {
            max: maxDateMoment.format("YYYY-MM-DD"),
            filter: '',
            currentPage: 1,
            perPage: 5,

            nomeAnimal: null,
            brinco: null,
            nomeAnimalState: null,
            peso: null,
            pesoState: null,
            dataPesagem: new Date().toISOString().substr(0, 10),
            dataPesagemState: 'null',
            animal: {},
            animais: [],

            pesagem: {},
            pesagens: [],
            fields: [
                { key: 'nomeAnimal', label: 'Nome', sortable: true},
                { key: 'brinco', label: 'Brinco', sortable: true},
                { key: 'dataPesagem', label: 'Data Pesagem', sortable: true,
                formatter: value => {
                    return moment(String(value)).format('DD/MM/YYYY')
                }},
                { key: 'peso', label: 'Peso (Kg)', sortable: true},
                { key: 'actions', label: 'Ações' }
            ]
        }      
    },
    async created() {
        try {
        const resPesagem = await axios.get(`${baseApiUrl}/pesagens`);
        const resAnimal = await axios.get(`${baseApiUrl}/animais?ativo=true`);

        this.pesagens = resPesagem.data;
        this.animais = resAnimal.data;
        } catch (e) {
        console.error(e);
        }
    },
    methods: {
        checkFormValidity() {
            const valid = this.$refs.form.checkValidity()
            this.nomeAnimalState = valid
            this.pesoState = valid
            this.dataPesagemState = valid
            return valid
        },
        resetModal() {
            this.nomeAnimal = ''
            this.peso = ''
            //this.dataPesagem = ''
            this.nomeAnimalState = null
            this.pesoState = null
            this.dataPesagemState = null
        },
        cadastroOk(bvModalEvt) {
            bvModalEvt.preventDefault()
            this.cadastrar()
        },
        editarOk(bvModalEvt) {
            bvModalEvt.preventDefault()
            this.editar()
        },
        async cadastrar() {
            if (!this.checkFormValidity()) {
                return
            }
            try {
                const id = this.nomeAnimal

                const res = await axios.get(`${baseApiUrl}/animais/${id}`);
                this.backupAnimal = res.data;

                const resPesagem = await axios.post(`${baseApiUrl}/pesagens`, { 
                    nomeAnimal: this.backupAnimal.nomeAnimal, 
                    brinco: this.backupAnimal.brinco, 
                    peso: this.peso, 
                    dataPesagem: this.dataPesagem});

                    this.pesagens = [...this.pesagens, resPesagem.data];

            } catch (e) {
                console.error(e);
            }

            // Esconder o modal manualmente
            this.$nextTick(() => {
                this.$bvModal.hide('modal-cadastrar-peso')
            })
        },
        async editar() {
            if (!this.checkFormValidity()) {
                return
            }
            try {
                const idPesagem = this.pesagem.id
                const resPesagem = await axios.put(`${baseApiUrl}/pesagens/${idPesagem}`, { 
                    nomeAnimal: this.pesagem.nomeAnimal, 
                    brinco: this.pesagem.brinco, 
                    peso: this.pesagem.peso, 
                    dataPesagem: this.pesagem.dataPesagem});

                    this.pesagens = [...this.pesagens, resPesagem.data];

            } catch (e) {
                console.error(e);
            }

            // Esconder o modal manualmente
            this.$nextTick(() => {
                //this.$bvModal.hide('modal-cadastrar-animal')
                location.reload();
            })
        },
        async remove() {
            try {
                const id = this.pesagem.id
                await axios.delete(`${baseApiUrl}/pesagens/${id}`)
                    .then(() => {
                        location.reload();
                    });
            } catch (e) {
                console.error(e);
            }

            this.$nextTick(() => {
                this.$bvModal.hide('modal-excluir-peso')
            })
        },
        loadPesagem(pesagem) {
            this.pesagem = { ...pesagem }
        }
    }
}
</script>

<style>

</style>