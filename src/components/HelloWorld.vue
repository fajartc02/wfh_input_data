<template>
  <div>
    <nav class="navbar navbar-expand-lg navbar-light bg-light" style="color:whitex">
      <a class="navbar-brand" href="#">WFH Job Input</a>
      <button
        class="navbar-toggler"
        type="button"
        data-toggle="collapse"
        data-target="#navbarSupportedContent"
        aria-controls="navbarSupportedContent"
        aria-expanded="false"
        aria-label="Toggle navigation"
      >
        <span class="navbar-toggler-icon"></span>
      </button>

      <div class="collapse navbar-collapse" id="navbarSupportedContent">
        <ul class="navbar-nav mr-auto"></ul>
        <form class="form-inline my-2 my-lg-0">
          <input
            class="form-control mr-sm-2"
            type="search"
            placeholder="Search"
            aria-label="Search"
          />
          <button class="btn btn-outline-success my-2 my-sm-0 btn-sm" type="submit">Search</button>
        </form>
      </div>
    </nav>
    <div class="container mb-4">
      <div class="form-group">
        <label for>Nama</label>
        <input type="username" class="form-control" v-model="nama" placeholder="Nama kamu" />
      </div>
      <div class="form-group">
        <label for>Item Pekerjaan</label>
        <input
          type="text"
          class="form-control"
          placeholder="Buat idea, meeting, etc."
          v-model="itemPekerjaan"
        />
      </div>
      <div v-if="isEdit" class="form-group">
        <label for>Status</label>
        <input ref="foc" type="text" class="form-control" placeholder="percent / O" v-model="status" />
      </div>
      <div v-if="isEdit" class="form-group">
        <label for>Keterangan</label>
        <input type="text" class="form-control" placeholder="masukan input" v-model="keterangan" />
      </div>
      <button type="button" class="btn btn-primary mr-2 btn-sm" @click="submitWfh" v-if="!isEdit">Submit</button>
      <button type="button" class="btn btn-success mr-2 btn-sm" @click="editWfh" v-if="isEdit">Submit Edit</button>
      <button type="button" class="btn btn-secondary btn-sm" @click="cancelEdit">Cancel</button>
    </div>

    <div class="container-fluid">
      <h4>{{actualDate}}</h4>
      <download-excel
        class   = "btn btn-default"
        :data   = "json_data"
        :fields = "json_fields"
        worksheet = "My Worksheet"
        name    = "wfh_job_maintenance.xls">
      <button type="button" class="btn btn-primary">Excel Download</button>
    </download-excel>
      <table class="table table-hover" style="text-align: left">
        <thead>
          <tr>
            <th scope="col">No</th>
            <th scope="col">Nama</th>
            <th scope="col">Item Pekerjaan</th>
            <th scope="col">Status</th>
            <th scope="col">Keterangan</th>
            <th scope="col">Actions</th>
          </tr>
        </thead>
        <tbody v-for="(item, index) in containerData" :key="item._id">
          <tr>
            <th scope="row">{{index + 1}}</th>
            <td>{{item.nama}}</td>
            <td>{{item.itemPekerjaan}}</td>
            <td>{{item.status}}</td>
            <td>{{item.keterangan}}</td>
            <td>
              <button type="button" class="btn btn-success mr-2 btn-sm" @click="editWfhConf(item)"><small>✎</small></button>
              <button type="button" class="btn btn-danger mr-2 btn-sm" @click="deleteWfh(item._id)"><small>🗑️</small></button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import moment from "moment";
import JsonExcel from "vue-json-excel"

export default {
  name: "HelloWorld",
  props: {
    msg: String
  },
  data() {
    return {
      nama: "",
      date: "",
      itemPekerjaan: "",
      status: "",
      keterangan: "",
      containerData: [],
      isEdit: false,
      idGrasp: "",
      actualDate: "",
      json_fields: {
        'id': '_id',
        'Date': 'date',
        'Nama': 'nama',
        'Item Pekerjaan': 'itemPekerjaan',
        'Status': 'status',
        'Keterangan' : 'keterangan',
        // 'createdAt' : 'createdAt',
        // 'updatedAt' : 'updatedAt',
      },
      json_data: []
    };
  },
  components: {
    'download-excel': JsonExcel
  },
  methods: {
    submitWfh() {
      let newWfh = {
        nama: this.nama,
        date: moment().format('L'),
        itemPekerjaan: this.itemPekerjaan,
        status: this.status,
        keterangan: this.keterangan
      };
      // console.log(newWfh);
      axios
        .post("http://103.82.241.157:3100/data/createWfh", newWfh)
        .then(result => {
          console.log(result);
          this.nama = "";
          this.itemPekerjaan = "";
          this.status = "";
          this.keterangan = "";
          this.date = "";
        })
        .catch(err => {
          console.log(err);
        });
    },
    editWfhConf(item) {
      if (this.isEdit === false) {
        this.isEdit = true;
      }
      this.nama = item.nama;
      this.itemPekerjaan = item.itemPekerjaan;
      this.status = item.status;
      this.keterangan = item.keterangan;
      this.idGrasp = item._id;
      this.$nextTick(() => {
        this.$refs.foc.focus()
      })
    },
    cancelEdit() {
      this.nama = "";
          this.itemPekerjaan = "";
          this.status = "";
          this.keterangan = "";
          this.date = "";
          this.isEdit = false
    },
    editWfh() {
      let editData = {
        nama: this.nama,
        itemPekerjaan: this.itemPekerjaan,
        status: this.status,
        keterangan: this.keterangan
      };
      console.log(editData);
      console.log(this.idGrasp);

      this.isEdit = false;
      axios
        .put(
          `http://103.82.241.157:3100/data/editWfh/${this.idGrasp}`,
          editData
        )
        .then(result => {
          console.log(result);
          this.nama = "";
          this.itemPekerjaan = "";
          this.status = "";
          this.keterangan = "";
          this.date = "";
        })
        .catch(err => {
          console.log(err);
        });
    },
    deleteWfh(id) {
      console.log(id);
      axios
        .delete(`http://103.82.241.157:3100/data/deleted/${id}`)
        .then(result => {
          console.log(result);
        })
        .catch(err => {
          console.log(err);
        });
    }
  },
  mounted() {
    
    axios
      .get("http://103.82.241.157:3100/data/getWfhs")
      .then(result => {
        console.log(result.data.data);
        this.containerData = result.data.data;
      })
      .catch(err => {
        console.log(err);
      });
    setInterval(() => {
      this.actualDate = moment().format("MMMM Do YYYY, h:mm:ss a");
    }, 1000)
    setInterval(() => {
      axios
        .get("http://103.82.241.157:3100/data/getWfhs")
        .then(result => {
          this.containerData = result.data.data;
          this.json_data = result.data.data
        })
        .catch(err => {
          console.log(err);
        });
    }, 2000);
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
