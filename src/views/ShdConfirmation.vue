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
    <div
      v-if="isError"
      class="alert alert-danger"
      role="alert"
    >Data yang anda input Hari ini sudah ada, mohon check dan konfirmasi kembali :)</div>
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
              <small>namanya samain sama table di bawah ya. Terima kasih :)</small>
            </div>
            <div class="form-group" translate="no">
              <label for="shift1">Shift</label>
              <select translate="no" class="form-control" id="Hour" v-model="shift">
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
      <div class="row">
        <div class="col-6">
          <h6>Target: {{55}}</h6>
        </div>
        <div class="col-6">
          <h6>Actual: {{totalCompleted}}</h6>
        </div>
      </div>
      <div class="small">
        <line-chart :chart-data="datacollection" :options="options"></line-chart>
      </div>
    </div>
    <!-- table -->
    <div class="container-fluid" style="margin-top: 250px; padding: 0px">
      <h5>{{today}}</h5>
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
              <!-- <tr v-if="index < 55"> -->
                <th v-if="index < 55" scope="row">{{index+1}}</th>
                <td v-if="index < 55" style="text-align: left">{{item.name}}</td>
                <td v-if="index < 55">{{item.shift}}</td>
                <td  v-if="item.status == 'NG' && index < 55" style="background-color: red">{{item.status}}</td>
                <td v-else-if="index < 55 && item.status == 'OK'">{{item.status}}</td>
              <!-- </tr> -->
            </tr>
          </tbody>
          <tbody v-else>
            <th v-if="isFill" scope="row" colspan="4">Sedang Load Data</th>
            <tr v-else>
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
        },
        scales: {
          yAxes: [
            {
              display: true,
              ticks: {
                suggestedMin: 0, // minimum will be 0, unless there is a lower value.
                // OR //
                beginAtZero: true // minimum value will be 0.
              }
            }
          ]
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
      isFill: false,
      completedStaff: 0,
      isActiveBtn: false,
      completedRed: 0,
      completedWhite: 0,
      totalCompleted: 0,
      eachTarget: 0,
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
            backgroundColor: ["#4365ef", "#f24358", "#3a9676"],
            data: [this.completedStaff, this.completedRed, this.completedWhite]
          }
        ]
      };
      if(this.isActiveBtn) {
        axios
          .get(
            `http://${process.env.VUE_APP_LOCAL_IP ||
              "103.82.241.157:3100"}/data/getShdTodayShift/STAFF`
          )
          .then(result => {
            console.log("staff");
            axios
              .get(
                `http://${process.env.VUE_APP_LOCAL_IP ||
                  "103.82.241.157:3100"}/data/getShdTodayShiftOK/STAFF`
              )
              .then(result => {
                this.completedStaff = result.data.data.length;
              })
              .catch(err => {
                console.log(err);
              });
            this.containerData = result.data.data;
            console.log(result);
          })
          .catch(err => {
            console.log(err);
          });
      }
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
            backgroundColor: ["#4365ef", "#f24358", "#3a9676"],
            data: [this.completedStaff, this.completedRed, this.completedWhite]
          }
        ]
      };
      if(this.isActiveBtn) {
        axios
          .get(
            `http://${process.env.VUE_APP_LOCAL_IP ||
              "103.82.241.157:3100"}/data/getShdTodayShift/RED`
          )
          .then(result => {
            //   console.log("red");
            axios
              .get(
                `http://${process.env.VUE_APP_LOCAL_IP ||
                  "103.82.241.157:3100"}/data/getShdTodayShiftOK/RED`
              )
              .then(result => {
                this.completedRed = result.data.data.length;
              })
              .catch(err => {
                console.log(err);
              });
            this.containerData = result.data.data;
          })
          .catch(err => {
            console.log(err);
          });
            
      }
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
            backgroundColor: ["#4365ef", "#f24358", "#3a9676"],
            data: [this.completedStaff, this.completedRed, this.completedWhite]
          }
        ]
      };
      if(this.isActiveBtn) {
        axios
          .get(
            `http://${process.env.VUE_APP_LOCAL_IP ||
              "103.82.241.157:3100"}/data/getShdTodayShift/WHITE`
          )
          .then(result => {
            //   console.log("white");
            axios
              .get(
                `http://${process.env.VUE_APP_LOCAL_IP ||
                  "103.82.241.157:3100"}/data/getShdTodayShiftOK/WHITE`
              )
              .then(result => {
                this.completedWhite = result.data.data.length;
              })
              .catch(err => {
                console.log(err);
              });
            this.containerData = result.data.data;
          })
          .catch(err => {
            console.log(err);
          });
      }
    }
  },
  methods: {
    //   2020-05-22
    getStaffData() {
      this.isActiveBtn = true;
      this.isLoading = true;
      axios
        .get(
          `http://${process.env.VUE_APP_LOCAL_IP ||
            "103.82.241.157:3100"}/data/getShdTodayShift/STAFF`
        )
        .then(result => {
          console.log("staff");
          this.isLoading = false;
          this.isActiveStaff = "btn btn-primary";
          this.isActiveRed = "btn btn-outline-danger";
          this.isActiveWhite = "btn btn-outline-success";
          axios
            .get(
              `http://${process.env.VUE_APP_LOCAL_IP ||
                "103.82.241.157:3100"}/data/getShdTodayShiftOK/STAFF`
            )
            .then(result => {
              this.completedStaff = result.data.data.length;
            })
            .catch(err => {
              console.log(err);
            });
          console.log(result);
          this.containerData = result.data.data;
        })
        .catch(err => {
          console.log(err);
        });
    },
    getRedData() {
      this.isLoading = true;
      this.isActiveBtn = true;
      axios
        .get(
          `http://${process.env.VUE_APP_LOCAL_IP ||
            "103.82.241.157:3100"}/data/getShdTodayShift/RED`
        )
        .then(result => {
          console.log("red");
          this.isLoading = false;
          this.isActiveStaff = "btn btn-outline-primary";
          this.isActiveRed = "btn btn-danger";
          this.isActiveWhite = "btn btn-outline-success";
          axios
            .get(
              `http://${process.env.VUE_APP_LOCAL_IP ||
                "103.82.241.157:3100"}/data/getShdTodayShiftOK/RED`
            )
            .then(result => {
              this.completedRed = result.data.data.length;
            })
            .catch(err => {
              console.log(err);
            });
          console.log(result);
          this.containerData = result.data.data;
        })
        .catch(err => {
          console.log(err);
        });
    },
    getWhiteData() {
      this.isActiveBtn = true;
      this.isLoading = true;
      axios
        .get(
          `http://${process.env.VUE_APP_LOCAL_IP ||
            "103.82.241.157:3100"}/data/getShdTodayShift/WHITE`
        )
        .then(result => {
          console.log("white");
          this.isLoading = false;
          this.isActiveStaff = "btn btn-outline-primary";
          this.isActiveRed = "btn btn-outline-danger";
          this.isActiveWhite = "btn btn-success";
          axios
            .get(
              `http://${process.env.VUE_APP_LOCAL_IP ||
                "103.82.241.157:3100"}/data/getShdTodayShiftOK/WHITE`
            )
            .then(result => {
              this.completedWhite = result.data.data.length;
            })
            .catch(err => {
              console.log(err);
            });
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
      this.isLoading = true;
      await axios
        .post(
          `http://${process.env.VUE_APP_LOCAL_IP ||
            "103.82.241.157:3100"}/data/createShd`,
          newShd
        )
        .then(async result => {
          this.isLoading = false;
          console.log(result);
          this.name = "";
          this.date = "";
          this.shift = "";
          if (!this.isActiveBtn) {
            await axios
              .get(
                `http://${process.env.VUE_APP_LOCAL_IP ||
                  "103.82.241.157:3100"}/data/getShdTodayShift/STAFF`
              )
              .then(result => {
                console.log("staff");
                axios
                  .get(
                    `http://${process.env.VUE_APP_LOCAL_IP ||
                      "103.82.241.157:3100"}/data/getShdTodayShiftOK/STAFF`
                  )
                  .then(result => {
                    this.completedStaff = result.data.data.length;
                  })
                  .catch(err => {
                    console.log(err);
                  });
                // this.containerData = result.data.data;
                console.log(result);
              })
              .catch(err => {
                console.log(err);
              });
            await axios
              .get(
                `http://${process.env.VUE_APP_LOCAL_IP ||
                  "103.82.241.157:3100"}/data/getShdTodayShift/RED`
              )
              .then(result => {
                console.log(result);
                axios
                  .get(
                    `http://${process.env.VUE_APP_LOCAL_IP ||
                      "103.82.241.157:3100"}/data/getShdTodayShiftOK/RED`
                  )
                  .then(result => {
                    this.completedRed = result.data.data.length;
                  })
                  .catch(err => {
                    console.log(err);
                  });
                // this.containerData = result.data.data;
              })
              .catch(err => {
                console.log(err);
              });
            await axios
              .get(
                `http://${process.env.VUE_APP_LOCAL_IP ||
                  "103.82.241.157:3100"}/data/getShdTodayShift/WHITE`
              )
              .then(result => {
                console.log(result);
                axios
                  .get(
                    `http://${process.env.VUE_APP_LOCAL_IP ||
                      "103.82.241.157:3100"}/data/getShdTodayShiftOK/WHITE`
                  )
                  .then(result => {
                    this.completedWhite = result.data.data.length;
                  })
                  .catch(err => {
                    console.log(err);
                  });
                // this.containerData = result.data.data;
              })
              .catch(err => {
                console.log(err);
              });
          } else {
            await axios
              .get(
                `http://${process.env.VUE_APP_LOCAL_IP ||
                  "103.82.241.157:3100"}/data/getShdTodayShift/STAFF`
              )
              .then(() => {
                console.log("staff");
                axios
                  .get(
                    `http://${process.env.VUE_APP_LOCAL_IP ||
                      "103.82.241.157:3100"}/data/getShdTodayShiftOK/STAFF`
                  )
                  .then(result => {
                    this.completedStaff = result.data.data.length;
                  })
                  .catch(err => {
                    console.log(err);
                  });
                // this.containerData = result.data.data;
                console.log(result);
              })
              .catch(err => {
                console.log(err);
              });
            await axios
              .get(
                `http://${process.env.VUE_APP_LOCAL_IP ||
                  "103.82.241.157:3100"}/data/getShdTodayShift/RED`
              )
              .then(() => {
                //   console.log("red");
                axios
                  .get(
                    `http://${process.env.VUE_APP_LOCAL_IP ||
                      "103.82.241.157:3100"}/data/getShdTodayShiftOK/RED`
                  )
                  .then(result => {
                    this.completedRed = result.data.data.length;
                  })
                  .catch(err => {
                    console.log(err);
                  });
                // this.containerData = result.data.data;
              })
              .catch(err => {
                console.log(err);
              });
            await axios
              .get(
                `http://${process.env.VUE_APP_LOCAL_IP ||
                  "103.82.241.157:3100"}/data/getShdTodayShift/WHITE`
              )
              .then(() => {
                //   console.log("white");
                axios
                  .get(
                    `http://${process.env.VUE_APP_LOCAL_IP ||
                      "103.82.241.157:3100"}/data/getShdTodayShiftOK/WHITE`
                  )
                  .then(result => {
                    this.completedWhite = result.data.data.length;
                  })
                  .catch(err => {
                    console.log(err);
                  });
                // this.containerData = result.data.data;
              })
              .catch(err => {
                console.log(err);
              });
          }
        })
        .catch(err => {
          this.isError = true;
          this.isLoading = false;
          console.log(err);
        });
    },
    judgShd() {
      console.log("last");
    },
    async fillShd() {
      this.isFill = true;
      if (this.containerData == false) {
        await axios
          .get(`http://${process.env.VUE_APP_PUBLIC_IP}/data/getShdFirstDate`)
          .then(result => {
            //   console.log("today");
            let firstData = result.data.data;
            console.log(firstData);
            
            if (new Date().getDate() < 10 && new Date().getMonth() < 10) {
              this.today = `${new Date().getFullYear()}-0${new Date().getMonth() +
                1}-0${new Date().getDate()}`;
            } else {
              this.today = `${new Date().getDate()}-${new Date().getMonth() +
                1}-${new Date().getFullYear()}`;
            }
            // for (let i = 0; i < firstData.length; i++) {
            //   console.log(firstData[i]);
            //   let newShd = {
            //     name: firstData[i].name,
            //     date: this.today,
            //     shift: firstData[i].shift
            //   };
            //   axios
            //     .post(
            //       `http://${process.env.VUE_APP_LOCAL_IP ||
            //         "103.82.241.157:3100"}/data/fillShd`,
            //       newShd
            //     )
            //     .then(result => {
            //       console.log(result);
            //     })
            //     .catch(err => {
            //       console.log(err);
            //     });
            // }
            console.log(result);
            // this.containerData = result.data.data;
            this.isFill = false;
          })
          .catch(err => {
            console.log(err);
          });
      } else {
        this.isFill = false;
      }
    },
    calcTotalCompleted() {
      this.totalCompleted =
        this.completedStaff + this.completedRed + this.completedWhite;
    }
  },
  async mounted() {
    if (new Date().getDate() < 10 && new Date().getMonth() < 10) {
      this.today = `${new Date().getFullYear()}-0${new Date().getMonth() +
        1}-0${new Date().getDate()}`;
    } else {
      this.today = `${new Date().getDate()}-${new Date().getMonth() +
        1}-${new Date().getFullYear()}`;
    }
    this.time = `${new Date().getHours()}:${new Date().getMinutes()}:${new Date().getMilliseconds()}`;
    console.log("no repeat");
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
          backgroundColor: ["#4365ef", "#f24358", "#3a9676"],
          data: [this.completedStaff, this.completedRed, this.completedWhite]
        }
      ]
    };
    if (!this.isActiveBtn) {
      await axios
        .get(
          `http://${process.env.VUE_APP_LOCAL_IP ||
            "103.82.241.157:3100"}/data/getShdToday`
        )
        .then(result => {
          //   console.log("today");
          console.log(result);
          this.containerData = result.data.data;
        })
        .catch(err => {
          console.log(err);
        });
      await this.fillShd();
      await this.calcTotalCompleted();
    }
    await axios
      .get(
        `http://${process.env.VUE_APP_LOCAL_IP ||
          "103.82.241.157:3100"}/data/getShdTodayShiftOK/STAFF`
      )
      .then(result => {
        console.log("staff");
        this.completedStaff = result.data.data.length;
        console.log(result);
      })
      .catch(err => {
        console.log(err);
      });
    await axios
      .get(
        `http://${process.env.VUE_APP_LOCAL_IP ||
          "103.82.241.157:3100"}/data/getShdTodayShiftOK/RED`
      )
      .then(result => {
        //   console.log("red");
        this.completedRed = result.data.data.length;
      })
      .catch(err => {
        console.log(err);
      });
    await axios
      .get(
        `http://${process.env.VUE_APP_LOCAL_IP ||
          "103.82.241.157:3100"}/data/getShdTodayShiftOK/WHITE`
      )
      .then(result => {
        //   console.log("white");
        this.completedWhite = result.data.data.length;
      })
      .catch(err => {
        console.log(err);
      });
    setInterval(async () => {
      this.isError = false;
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
            backgroundColor: ["#4365ef", "#f24358", "#3a9676"],
            data: [this.completedStaff, this.completedRed, this.completedWhite]
          }
        ]
      };
      if (!this.isActiveBtn) {
        axios
          .get(
            `http://${process.env.VUE_APP_LOCAL_IP ||
              "103.82.241.157:3100"}/data/getShdToday`
          )
          .then(result => {
            //   console.log("today");
            console.log(result);
            this.containerData = result.data.data;
          })
          .catch(err => {
            console.log(err);
          });
      }
      await axios
        .get(
          `http://${process.env.VUE_APP_LOCAL_IP ||
            "103.82.241.157:3100"}/data/getShdTodayShiftOK/STAFF`
        )
        .then(result => {
          console.log("staff");
          this.completedStaff = result.data.data.length;
          console.log(result);
        })
        .catch(err => {
          console.log(err);
        });
      await axios
        .get(
          `http://${process.env.VUE_APP_LOCAL_IP ||
            "103.82.241.157:3100"}/data/getShdTodayShiftOK/RED`
        )
        .then(result => {
          //   console.log("red");
          this.completedRed = result.data.data.length;
        })
        .catch(err => {
          console.log(err);
        });
      await axios
        .get(
          `http://${process.env.VUE_APP_LOCAL_IP ||
            "103.82.241.157:3100"}/data/getShdTodayShiftOK/WHITE`
        )
        .then(result => {
          //   console.log("white");
          this.completedWhite = result.data.data.length;
        })
        .catch(err => {
          console.log(err);
        });
      await this.fillShd();
      await this.calcTotalCompleted();
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