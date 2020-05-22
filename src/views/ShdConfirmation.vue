<template>
  <div class="container-fluid" style="padding: 0px">
    <vue-element-loading :active="isLoading" :is-full-screen="isLoading" spinner="line-wave">
        <img src="../assets/pingpong.gif" alt />
        <h6>Loading . . .</h6>
      </vue-element-loading>
    <nav class="navbar navbar-light" style="background-color: #3B5998;">
      <a class="navbar-brand" href="#" style="color: white">SHD Confirmation Monitoring</a>
    </nav>
    <div style="height: 5px"></div>
    <CarouselCovid />
    <div style="height: 15px"></div>
    <!-- input SHD -->
    <div class="container-fluid">
      <a
        data-toggle="collapse"
        href="#collapseExample"
        role="button"
        aria-expanded="false"
        aria-controls="collapseExample"
        class="btn btn-outline-primary"
      >SHD Confirmation</a>
      <div style="height: 10px"></div>
      <div class="collapse" id="collapseExample">
        <div class="card card-body">
          <form onsubmit="return false">
            <div class="form-group">
              <label for="oee1">Link SHD</label>
              <a
                class="form-control"
                target="_blank"
                href="https://frost.toyota.co.id/jw/web/userview/covid19_response/c19_userview/_/D4F3FF511F874420BEEF0E009947FFD2"
              >Click Disini kalau belum isi SHD</a>
              <small>kalau sudah lanjut di isi konfirmasinya, isi nama dan shift nya ya. Terima kasih :)</small>
            </div>
            <div class="form-group">
              <label for="exampleFormControlSelect1">Name</label>
              <input
                type="text"
                class="form-control"
                id="oee1"
                placeholder="Nama Kamu"
                v-model="name"
              />
            </div>
            <div class="form-group">
              <label for="shift1">Shift</label>
              <select class="form-control" id="Hour" v-model="shift">
                <option disabled value>--choose shift--</option>
                <option>STAFF</option>
                <option>RED</option>
                <option>WHITE</option>
              </select>
            </div>
            <div class="form-group">
              <label for="exampleFormControlSelect1">Date</label>
              <input type="date" class="form-control" id="date1" v-model="date" />
            </div>
            <button class="btn btn-success" @click="submitSHD">Submit</button>
          </form>
        </div>
      </div>
    </div>
    <!-- chart SHD -->
    <div class="container-fluid">
      <div class="small">
        <line-chart :chart-data="datacollection" :options="options"></line-chart>
      </div>
    </div>
    <!-- table -->
    <div class="container-fluid" style="margin-top: 250px; padding: 0px">
      <h5>{{`${new Date().getFullYear()}-0${new Date().getMonth()+1}-${new Date().getDate()}`}}</h5>
      <div class="row">
        <div class="col-4">
          <button :class="isActiveStaff" @click="getStaffData">Staff</button>
        </div>
        <div class="col-4">
          <button :class="isActiveRed" @click="getRedData">Red</button>
        </div>
        <div class="col-4">
          <button :class="isActiveWhite" @click="getWhiteData">White</button>
        </div>
      </div>
      <div class="row mt-3">
        <table class="table table-hover">
          <thead>
            <tr>
              <th scope="col">No</th>
              <th scope="col">Name</th>
              <th scope="col">Shift</th>
              <th scope="col">Status</th>
            </tr>
          </thead>
          <tbody v-if="containerData && containerData.length !== 0">
            <tr v-for="(item, index) in containerData" :key="index">
              <th scope="row">{{index+1}}</th>
              <td>{{item.name}}</td>
              <td>{{item.shift}}</td>
              <td>{{item.status}}</td>
            </tr>
          </tbody>
          <tbody v-else>
            <tr>
              <th scope="row" colspan="4">belum ada yang input hari ini</th>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import LineChart from "../components/LineChart.js";
import CarouselCovid from "../components/CarouselCovid";
import axios from "axios";
import VueElementLoading from "vue-element-loading";

export default {
  components: {
    LineChart,
    CarouselCovid,
    "vue-element-loading": VueElementLoading
  },
  data() {
    return {
      datacollection: null,
      options: {
        legend: {
          display: false
        }
      },
      isLoading: false,
      containerData: false,
      isActiveStaff: "btn btn-outline-primary",
      isActiveRed: "btn btn-outline-danger",
      isActiveWhite: "btn btn-outline-success",
      today: "",
      name: "",
      shift: "",
      date: "",
      time: "",
      completedStaff: 0,
      isActiveBtn: false,
      completedRed: 0,
      completedWhite: 0,
      isError: false,
      staffName: [
        "ACENG KURNIA NUGRAHA",
        "AMIN YUSUF",
        "ARI PRASETYA",
        "ARIS BUDIANTO",
        "DAIM",
        "FAJAR TRI CAHYONO",
        "GALUH MURTISARI ARUM",
        "HESA WINDA AFWANTI",
        "ROHMAD BASUKI",
        "SUPRIYADI",
        "TUKHANDI"
      ]
    };
  },
  watch: {
    completedStaff: function() {
      this.datacollection = {
        labels: [`Staff MT`, "OP MT Red", "OP MT White"],
        datasets: [
          {
            label: "Target",
            type: "line",
            fill: false,
            backgroundColor: "#00f4e4",
            data: [11, 22, 22]
          },
          {
            label: "Completed",
            backgroundColor: ["#e87979", "#3b5998", "#3a9676"],
            data: [this.completedStaff, this.completedRed, this.completedWhite]
          }
        ]
      };
    },
    completedRed: function() {
      this.datacollection = {
        labels: [`Staff MT`, "OP MT Red", "OP MT White"],
        datasets: [
          {
            label: "Target",
            type: "line",
            fill: false,
            backgroundColor: "#00f4e4",
            data: [11, 22, 22]
          },
          {
            label: "Completed",
            backgroundColor: ["#e87979", "#3b5998", "#3a9676"],
            data: [this.completedStaff, this.completedRed, this.completedWhite]
          }
        ]
      };
    },
    completedWhite: function() {
      this.datacollection = {
        labels: [`Staff MT`, "OP MT Red", "OP MT White"],
        datasets: [
          {
            label: "Target",
            type: "line",
            fill: false,
            backgroundColor: "#00f4e4",
            data: [11, 22, 22]
          },
          {
            label: "Completed",
            backgroundColor: ["#e87979", "#3b5998", "#3a9676"],
            data: [this.completedStaff, this.completedRed, this.completedWhite]
          }
        ]
      };
    },
    containerData: function() {
      // if(this.isLoading == false) {
      //   this.isLoading = true
      // } else {
      //   this.isLoading = false
      // }
    }
  },
  methods: {
    getStaffData() {
      this.isActiveBtn = true;
      this.isLoading = true
      axios
        .get("http://103.82.241.157:3100/data/getShdTodayShift/STAFF")
        .then(result => {
          console.log("staff");
          this.isLoading = false
          this.isActiveStaff = "btn btn-primary";
          this.isActiveRed = "btn btn-outline-danger";
          this.isActiveWhite = "btn btn-outline-success";
          this.completedStaff = result.data.data.length;
          console.log(result);
          this.containerData = result.data.data;
        })
        .catch(err => {
          console.log(err);
        });
    },
    getRedData() {
      this.isLoading = true
      this.isActiveBtn = true;
      axios
        .get("http://103.82.241.157:3100/data/getShdTodayShift/RED")
        .then(result => {
          console.log("red");
          this.isLoading = false
          this.isActiveStaff = "btn btn-outline-primary";
          this.isActiveRed = "btn btn-danger";
          this.isActiveWhite = "btn btn-outline-success";
          this.completedRed = result.data.data.length;
          console.log(result);
          this.containerData = result.data.data;
        })
        .catch(err => {
          console.log(err);
        });
    },
    getWhiteData() {
      this.isActiveBtn = true;
      this.isLoading = true
      axios
        .get("http://103.82.241.157:3100/data/getShdTodayShift/WHITE")
        .then(result => {
          console.log("white");
          this.isLoading = false
          this.isActiveStaff = "btn btn-outline-primary";
          this.isActiveRed = "btn btn-outline-danger";
          this.isActiveWhite = "btn btn-success";
          this.completedWhite = result.data.data.length;
          console.log(result);
          this.containerData = result.data.data;
        })
        .catch(err => {
          console.log(err);
        });
    },
    async submitSHD() {
      let newShd = {
        name: this.name.toUpperCase(),
        date: this.date,
        shift: this.shift
      };
      this.isLoading = true
      await axios
        .post("http://103.82.241.157:3100/data/createShd", newShd)
        .then(result => {
          this.isLoading = false
          console.log(result);
          this.name = "";
          this.date = "";
          this.shift = "";
          axios
            .get("http://103.82.241.157:3100/data/getShdTodayShift/STAFF")
            .then(result => {
              console.log("staff");
              this.completedStaff = result.data.data.length;
              this.containerData = result.data.data;
              console.log(result);
            })
            .catch(err => {
              console.log(err);
            });
          axios
            .get("http://103.82.241.157:3100/data/getShdTodayShift/RED")
            .then(result => {
              //   console.log("red");
              this.completedRed = result.data.data.length;
              this.containerData = result.data.data;
            })
            .catch(err => {
              console.log(err);
            });
          axios
            .get("http://103.82.241.157:3100/data/getShdTodayShift/WHITE")
            .then(result => {
              //   console.log("white");
              this.completedWhite = result.data.data.length;
              this.containerData = result.data.data;
            })
            .catch(err => {
              console.log(err);
            });
        })
        .catch(err => {
          this.isError = true
          console.log(err);
        });
    }
  },
  mounted() {
    this.today = `${new Date().getDate()}-${new Date().getMonth() +
      1}-${new Date().getFullYear()}`;
    this.time = `${new Date().getHours()}:${new Date().getMinutes()}:${new Date().getMilliseconds()}`;
    console.log("no repeat");
    setInterval(() => {
      console.log("repeat");
      this.datacollection = {
        labels: [`Staff MT`, "OP MT Red", "OP MT White"],
        datasets: [
          {
            label: "Target",
            type: "line",
            fill: false,
            backgroundColor: "#00f4e4",
            data: [11, 22, 22]
          },
          {
            label: "Completed",
            backgroundColor: ["#e87979", "#3b5998", "#3a9676"],
            data: [this.completedStaff, this.completedRed, this.completedWhite]
          }
        ]
      };
      if (!this.isActiveBtn) {
        axios
          .get("http://103.82.241.157:3100/data/getShdToday")
          .then(result => {
            //   console.log("today");
            console.log(result);
            this.containerData = result.data.data;
          })
          .catch(err => {
            console.log(err);
          });
      }
      axios
        .get("http://103.82.241.157:3100/data/getShdTodayShift/STAFF")
        .then(result => {
          console.log("staff");
          this.completedStaff = result.data.data.length;
          console.log(result);
        })
        .catch(err => {
          console.log(err);
        });
      axios
        .get("http://103.82.241.157:3100/data/getShdTodayShift/RED")
        .then(result => {
          //   console.log("red");
          this.completedRed = result.data.data.length;
        })
        .catch(err => {
          console.log(err);
        });
      axios
        .get("http://103.82.241.157:3100/data/getShdTodayShift/WHITE")
        .then(result => {
          //   console.log("white");
          this.completedWhite = result.data.data.length;
        })
        .catch(err => {
          console.log(err);
        });
    }, 3000);
  }
};
</script>

<style>
.small {
  max-width: 600px;
  height: 100px;
  margin: 2px auto;
  font-size: 10px;
}
</style>