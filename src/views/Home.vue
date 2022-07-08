<template>
  <div class="home">
    <b-button
      @click="makeCar = !makeCar"
      variant="primary"
      style="margin-top: 20px; margin-bottom: 20px; float: right"
      >{{ makeCar ? "Fechar" : "Criar Carro" }}</b-button
    >
    <br />
    <br />
    <div>
      <div v-if="makeCar">
        <b-form-group
          id="input-group-1"
          label="Nome do Carro"
          label-for="input-1"
        >
          <b-form-input
            id="input-1"
            v-model="nome"
            type="text"
            placeholder="Nome do Carro"
            required
          ></b-form-input>
        </b-form-group>

        <b-form-group
          id="input-group-2"
          label="Marca do Carro"
          label-for="input-2"
        >
          <b-form-input
            id="input-2"
            v-model="marca"
            placeholder="Marca do Carro"
            required
          ></b-form-input>
        </b-form-group>

        <b-form-group
          id="input-group-3"
          label="Cor do Carro"
          label-for="input-3"
        >
          <b-form-input
            id="input-2"
            v-model="color"
            placeholder="Cor do Carro"
            required
          ></b-form-input>
        </b-form-group>

        <b-form-group id="input-group-3" label="KM:" label-for="input-3">
          <b-form-input
            id="input-2"
            v-model="km"
            placeholder="KM"
            required
          ></b-form-input>
        </b-form-group>

        <b-form-group id="input-group-3" label="Detalhes" label-for="input-3">
          <b-form-input
            id="input-2"
            v-model="details"
            placeholder="Ex: Carro bem conservado..."
            required
          ></b-form-input>
        </b-form-group>

        <b-form-group
          id="input-group-3"
          label="URL da Imagem"
          label-for="input-3"
        >
          <b-form-input
            id="input-2"
            v-model="image"
            placeholder="Link da Imagem"
            required
          ></b-form-input>
        </b-form-group>

        <div style="float: right">
          <b-button
            style="margin-right: 5px; margin-top: 10px"
            @click="makeCar = false"
            type="reset"
            variant="danger"
            >Cancelar</b-button
          >

          <b-button
            style="margin-top: 10px"
            type="submit"
            variant="primary"
            @click="cadastrarCarro"
            >Cadastrar Carro</b-button
          >
        </div>
      </div>
    </div>

    <b-table
      :items="cars"
      :fields="fields"
      :current-page="currentPage"
      :per-page="perPage"
      :filter="filter"
      :filter-included-fields="filterOn"
      :sort-by.sync="sortBy"
      :sort-desc.sync="sortDesc"
      :sort-direction="sortDirection"
      stacked="md"
      show-empty
      small
      @filtered="onFiltered"
    >
      <template #cell(name)="row">
        {{ row.value }}
      </template>

      <template #cell(actions)="row">
        <b-button
          size="sm"
          variant="danger"
          @click="deleteCar(row.item.id)"
          class="mr-1"
        >
          Deletar
        </b-button>
        <b-button size="sm" class="mr-1" @click="getUniqCars(row.item.id)">
          Ver detalhes
        </b-button>
      </template>

      <template #row-details="row">
        <b-card>
          <ul>
            <li v-for="(value, key) in row.item" :key="key">
              {{ key }}: {{ value }}
            </li>
          </ul>
        </b-card>
      </template>
    </b-table>

    <b-modal id="modal-1" hide-footer>
      <div class="d-block">
        <p>Nome: {{ carUniq.name }}</p>
        <br />
        <p>Marca: {{ carUniq.brand }}</p>
        <br />

        <p>Cor: {{ carUniq.color }}</p>
        <br />

        <p>Detalhes: {{ carUniq.details }}</p>
        <br />

        <p>KM: {{ carUniq.km }}</p>
        <br />

         <b-img :src="carUniq.image" fluid alt="Fluid image"></b-img>
      </div>
      <b-button style="margin-top: 20px; float:right;" @click="hideModal">Fechar</b-button>
    </b-modal>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "Home",
  data() {
    return {
      carUniq: {},
      nome: "",
      marca: "",
      color: "",
      km: "",
      details: "",
      image: "",
      makeCar: false,
      cars: [],
      fields: [
        {
          key: "id",
          label: "ID",
        },
        {
          key: "name",
          label: "Carro",
        },
        {
          key: "brand",
          label: "Marca",
        },
        {
          key: "actions",
          label: "Ações",
        },
      ],
      totalRows: 1,
      currentPage: 1,
      perPage: 5,
      pageOptions: [5, 10, 15, { value: 100, text: "Show a lot" }],
      sortBy: "",
      sortDesc: false,
      sortDirection: "asc",
      filter: null,
      filterOn: [],
      infoModal: {
        id: "info-modal",
        title: "",
        content: "",
      },
    };
  },
  computed: {
    sortOptions() {
      // Create an options list from our fields
      return this.fields
        .filter((f) => f.sortable)
        .map((f) => {
          return { text: f.label, value: f.key };
        });
    },
  },
  mounted() {
    this.getCars();
  },
  methods: {
    hideModal() {
      this.$root.$emit("bv::hide::modal", "modal-1", "#btnShow");
    },
    deleteCar(id){
      var that = this;
      axios
        .delete("http://workshop.innovaweb.com.br/cars/" + id)
        .then(function (response) {
          console.log(response);
          that.getCars();
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    cadastrarCarro() {
      var dados = {
        name: this.nome,
        color: this.color,
        brand: this.marca,
        km: this.km,
        details: this.details,
        image: this.image,
      };

      var that = this;
      axios
        .post("http://workshop.innovaweb.com.br/cars", dados)
        .then(function (response) {
          console.log(response);
          that.nome = "";
          that.color = "";
          that.marca = "";
          that.km = "";
          that.details = "";
          that.image = "";
          that.makeCar = false;
          that.getCars();
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    getUniqCars(id) {
      var that = this;
      axios
        .get("http://workshop.innovaweb.com.br/cars/" + id)
        .then(function (response) {
          console.log(response);
          that.carUniq = response.data;
          that.$root.$emit("bv::show::modal", "modal-1", "#btnShow");
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    getCars() {
      var that = this;
      axios
        .get("http://workshop.innovaweb.com.br/cars")
        .then(function (response) {
          console.log(response);
          that.cars = response.data;
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    info(item, index, button) {
      this.infoModal.title = `Row index: ${index}`;
      this.infoModal.content = JSON.stringify(item, null, 2);
      this.$root.$emit("bv::show::modal", this.infoModal.id, button);
    },
    resetInfoModal() {
      this.infoModal.title = "";
      this.infoModal.content = "";
    },
    onFiltered(filteredItems) {
      // Trigger pagination to update the number of buttons/pages due to filtering
      this.totalRows = filteredItems.length;
      this.currentPage = 1;
    },
  },
};
</script>

<style lang="scss">
.home {
  width: 80%;
  margin: 0 auto;
}

label {
  float: left !important;
}

button.close {
  border: none;
  background: no-repeat;
  font-size: 25px;
}

header#modal-1___BV_modal_header_ {
  border: none;
  margin-bottom: -15px;
}

button.btn.mr-1.btn-danger.btn-sm {
    margin-right: 12px;
}
</style>
