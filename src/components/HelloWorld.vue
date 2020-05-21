<template>
  <div>
    <nav class="navbar navbar-expand-lg navbar-light bg-light" style="color:whitex">
      <a class="navbar-brand" href="#">WFH Job Input</a>
      <a
        href="https://frost.toyota.co.id/jw/web/userview/covid19_response/c19_userview/_/D4F3FF511F874420BEEF0E009947FFD2"
        target="_blank"
        style="font-size: 12px"
      >SHD daily input</a>
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
      <form onsubmit="return false">
        <div class="form-group">
          <label for>Nama</label>
          <input
            type="text"
            name="nama"
            autocomplete="on"
            class="form-control"
            v-model="nama"
            placeholder="Nama kamu"
          />
        </div>
        <div class="form-group">
          <label for>Item Pekerjaan</label>
          <input
            type="text"
            class="form-control"
            placeholder="Buat idea, meeting, etc."
            v-model="itemPekerjaan"
            name="itemPekerjaan"
            autocomplete="on"
          />
        </div>
        <div v-if="isEdit" class="form-group">
          <label for>Status</label>
          <input
            ref="foc"
            type="text"
            class="form-control"
            placeholder="percent / OK"
            v-model="status"
          />
        </div>
        <div v-if="isEdit" class="form-group">
          <label for>Keterangan</label>
          <input type="text" class="form-control" placeholder="masukan input" v-model="keterangan" />
        </div>
        <button
          type="submit"
          class="btn btn-primary mr-2 btn-sm"
          @click="submitWfh"
          v-if="!isEdit"
        >Submit</button>
        <button
          type="submit"
          class="btn btn-success mr-2 btn-sm"
          @click="editWfh"
          v-if="isEdit"
        >Submit Edit</button>
        <button type="submit" class="btn btn-secondary btn-sm" @click="cancelEdit">Cancel</button>
      </form>
    </div>

    <div class="container-fluid">
      <h4>{{actualDate}}</h4>
      <!-- SEARCH -->
      <div class="container-fluid">
        <div class="card">
          <div class="card-header">Search Data</div>
          <div class="card-body">
            <!-- <h5 class="card-title">Special title treatment</h5> -->
            <div class="form-group">
              <label for="date1">Date</label>
              <input type="date" class="form-control" v-model="selectedDate" />
            </div>
            <div class="form-group">
              <label for="Name">Name</label>
              <input type="text" class="form-control" v-model="selectedNama" />
            </div>
            <!-- <button class="btn btn-primary" @click="searchData">Search</button> -->
          </div>
        </div>
      </div>

      <download-excel
        class="btn btn-default"
        :data="json_data"
        :fields="json_fields"
        worksheet="My Worksheet"
        name="wfh_job_maintenance.xls"
      >
        <button type="button" class="btn btn-primary">Excel Download</button>
      </download-excel>
      <vue-element-loading :active="isLoading" :is-full-screen="isLoading" spinner="line-wave">
        <img src="../assets/pingpong.gif" alt />
        <h6>Loading . . .</h6>
      </vue-element-loading>
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
              <button type="button" class="btn btn-success mr-2 btn-sm" @click="editWfhConf(item)">
                <small>✎</small>
              </button>
              <button type="button" class="btn btn-danger mr-2 btn-sm" @click="deleteWfh(item._id)">
                <small>🗑️</small>
              </button>
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
import JsonExcel from "vue-json-excel";
// import spinner from "progress-spinner"
import VueElementLoading from "vue-element-loading";

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
        id: "_id",
        Date: "updatedAt",
        Nama: "nama",
        "Item Pekerjaan": "itemPekerjaan",
        Status: "status",
        Keterangan: "keterangan"
      },
      json_data: [],
      selectedDate: "",
      selectedNama: "",
      isChanges: "",
      isLoading: false
    };
  },
  components: {
    "download-excel": JsonExcel,
    // "progress-spinner": spinner,
    "vue-element-loading": VueElementLoading
  },
  methods: {
    submitWfh() {
      let newWfh = {
        nama: this.nama,
        date: moment().format("L"),
        itemPekerjaan: this.itemPekerjaan,
        status: this.status,
        keterangan: this.keterangan
      };
      // console.log(newWfh);
      this.isLoading = true;
      axios
        .post("http://103.82.241.157:3100/data/createWfh", newWfh)
        .then(result => {
          console.log(result);
          this.isChanges = newWfh;
          this.nama = "";
          this.itemPekerjaan = "";
          this.status = "";
          this.keterangan = "";
          this.date = "";
          this.isLoading = false;
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
        this.$refs.foc.focus();
      });
    },
    cancelEdit() {
      this.nama = "";
      this.itemPekerjaan = "";
      this.status = "";
      this.keterangan = "";
      this.date = "";
      this.isEdit = false;
    },
    editWfh() {
      let editData = {
        nama: this.nama,
        itemPekerjaan: this.itemPekerjaan,
        status: this.status,
        keterangan: this.keterangan
      };
      this.isEdit = false;
      this.isLoading = true;
      axios
        .put(
          `http://103.82.241.157:3100/data/editWfh/${this.idGrasp}`,
          editData
        )
        .then(result => {
          console.log(result);
          this.isChanges = result;

          this.itemPekerjaan = "";
          this.status = "";
          this.keterangan = "";
          this.date = "";
          this.isLoading = false;
        })
        .catch(err => {
          console.log(err);
        });
    },
    deleteWfh(id) {
      console.log(id);
      this.isLoading = true
        .delete(`http://103.82.241.157:3100/data/deleted/${id}`)
        .then(result => {
          console.log(result);
          this.isLoading = false;
          this.isChanges = 4;
        })
        .catch(err => {
          console.log(err);
        });
    },
    searchData() {
      // console.log(filterDataToday);
      // let filterDataNama = rawArray.filter(el => {
      //   return (
      //     el.nama.toLowerCase().indexOf(this.selectedNama.toLowerCase()) > -1
      //   );
      // });
    }
  },
  watch: {
    isChanges: function() {
      this.isLoading = true;
      axios
        .get("http://103.82.241.157:3100/data/getWfhs")
        .then(result => {
          // let rawArray = result.data.data;
          // let filterDataToday = rawArray.filter(element => {
          //   let tgl = new Date(element.date).getDate();
          //   let month = new Date(element.date).getMonth();
          //   let year = new Date(element.date).getFullYear();
          //   let curTgl = new Date().getDate();
          //   let curMonth = new Date().getMonth();
          //   let curYear = new Date().getFullYear();
          //   console.log(tgl == curTgl);

          //   return tgl == curTgl && month == curMonth && year == curYear;
          // });
          this.isLoading = false;
          this.containerData = result.data.data;

          this.json_data = result.data.data;
        })
        .catch(err => {
          console.log(err);
        });
    },
    selectedNama: function() {
      if (this.selectedNama == "") {
        this.isLoading = true;
        if (this.selectedDate != "") {
          this.isLoading = true;
          axios
            .get(
              `http://103.82.241.157:3100/data/getWfhs/date=${this.selectedDate}`
            )
            .then(result => {
              let rawArray = result.data.data;
              let filterDataToday = rawArray.filter(element => {
                let tgl = new Date(element.date).getDate();
                let month = new Date(element.date).getMonth();
                let year = new Date(element.date).getFullYear();
                let curTgl = new Date(this.selectedDate).getDate();
                let curMonth = new Date(this.selectedDate).getMonth();
                let curYear = new Date(this.selectedDate).getFullYear();
                return tgl == curTgl && month == curMonth && year == curYear;
              });
              this.isLoading = false;
              this.containerData = filterDataToday;
              this.json_data = filterDataToday;
            })
            .catch(err => {
              console.log(err);
            });
        } else {
          axios
            .get("http://103.82.241.157:3100/data/getWfhs")
            .then(result => {
              this.containerData = result.data.data;

              this.json_data = result.data.data;
              this.isLoading = false;
            })
            .catch(err => {
              console.log(err);
            });
        }
      } else {
        let rawArray = this.containerData;
        let filterDataNama = rawArray.filter(el => {
          return (
            el.nama.toLowerCase().indexOf(this.selectedNama.toLowerCase()) > -1
          );
        });
        this.containerData = filterDataNama;
      }
    },
    selectedDate: function() {
      if (this.selectedDate != "") {
        this.isLoading = true;
        axios
          .get(
            `http://103.82.241.157:3100/data/getWfhs/date=${this.selectedDate}`
          )
          .then(result => {
            let rawArray = result.data.data;
            let filterDataToday = rawArray.filter(element => {
              let tgl = new Date(element.date).getDate();
              let month = new Date(element.date).getMonth();
              let year = new Date(element.date).getFullYear();
              let curTgl = new Date(this.selectedDate).getDate();
              let curMonth = new Date(this.selectedDate).getMonth();
              let curYear = new Date(this.selectedDate).getFullYear();
              return tgl == curTgl && month == curMonth && year == curYear;
            });
            this.isLoading = false;
            this.containerData = filterDataToday;
            this.json_data = filterDataToday;
          })
          .catch(err => {
            console.log(err);
          });
      }
    }
  },
  mounted() {
    this.actualDate = moment().format("MMMM Do YYYY, h:mm:ss a");
    // setInterval(() => {
    //   this.actualDate = moment().format("MMMM Do YYYY, h:mm:ss a");
    // }, 1000);
    this.isLoading = true;
    axios
      .get("http://103.82.241.157:3100/data/getWfhs/")
      .then(result => {
        // let rawArray = result.data.data;
        // let filterDataToday = rawArray.filter(element => {
        //   let tgl = new Date(element.date).getDate();
        //   let month = new Date(element.date).getMonth();
        //   let year = new Date(element.date).getFullYear();
        //   let curTgl = new Date().getDate();
        //   let curMonth = new Date().getMonth();
        //   let curYear = new Date().getFullYear();
        //   console.log(tgl == curTgl);

        //   return tgl == curTgl && month == curMonth && year == curYear;
        // });
        this.containerData = result.data.data;
        this.json_data = result.data.data;
        this.isLoading = false;
        // this.isChanges = result.data.data;
      })
      .catch(err => {
        console.log(err);
      });
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
