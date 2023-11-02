<template>
  <div>
    <el-header analytics><b>Store Location Here</b></el-header>
    <el-container>
      <el-aside width="300px">
        <ul>
          <li class="title-wrap"><div class="title">Dealer analytics</div></li>
          <li
            :class="{ 'title-wrap-active': viewState === 'export' }"
            @click="setViewState('export')"
          >
            <div class="sub-menu-title">Export trial balance</div>
          </li>
          <li
            :class="{ 'title-wrap-active': viewState === 'history' }"
            @click="setViewState('history')"
          >
            <div class="sub-menu-title">View history</div>
          </li>
          <li class="title-wrap"><div class="title">Dealer settings</div></li>
          <li
            :class="{ 'title-wrap-active': viewState === 'setting' }"
            @click="setViewState('setting')"
          >
            <div class="sub-menu-title">Settings</div>
          </li>
        </ul>
        <el-card
          style="margin-top: 100px; border-color: #dfd7d7; text-align: left"
        >
          <h5><b>Dealer Number</b></h5>
          <h5>
            <b>{{ showDealerValue }}</b>
          </h5>
        </el-card>
      </el-aside>
      <el-container style="padding-left: 20px">
        <el-header style="background-color: white; text-align: left">
          {{ title }}
        </el-header>

        <el-main v-if="viewState === 'export'">
          <el-row :gutter="10">
            <el-col :span="16">
              <el-form label-position="left" label-width="200px">
                <el-form-item label="Dealer">
                  {{ showDealerValue }}
                  <!-- <el-input v-model="dealer" readonly></el-input> -->
                </el-form-item>

                <el-form-item label="Select month and year" label-width="200px">
                  <el-date-picker
                    v-model="yearMonth"
                    type="month"
                    format="MM/yyyy"
                    value-format="yyyyMMDD"
                    prefix-icon="none"
                    :picker-options="pickerOptions"
                  >
                  </el-date-picker>
                </el-form-item>
              </el-form>
            </el-col>
            <el-col :span="4" style="margin-top: 60px; margin-left: 20px">
              <el-button
                @click="submitData"
                :disabled="dealer === ''"
                :type="dealer == '' ? 'info' : 'primary'"
                >Submit Data</el-button
              >
            </el-col>
          </el-row>
        </el-main>
        <el-main v-if="viewState === 'history'">
          <el-table
            :data="history"
            border
            style="width: 100%"
            class="unitTable"
          >
            <!-- <el-table-column label="Stock Number" > -->
            <!-- <template slot-scope="scope">
                <a
                  href="#"
                  @click.stop="
                    openUnitInquiryWindow($event, scope.row.stock_num)
                  "
                  ><i
                    class="fa fa-external-link-square"
                    aria-hidden="true"
                    style="padding-left: 5px"
                  ></i>
                </a>
              </template> -->
            <!-- </el-table-column> -->
            <el-table-column prop="DealerCode" label="Dealer Number" width="70">
            </el-table-column>
            <el-table-column prop="LineCount" label="Line Count">
            </el-table-column>
            <el-table-column
              prop="SubmittedDate"
              label="Submit Date"
              sortable
              :sort-method="sortBySubmittedDate"
            >
            </el-table-column>
            <el-table-column prop="AsOfDate" label="As of date" sortable
              :sort-method="sortByAsOfDate">
            </el-table-column>
          </el-table>
          <grid
            :data="history"
            :columns="itemDataColumns"
            :keyProps="['DealerID']"
            :pageSize="9999"
            :sortable="true"
          ></grid>
        </el-main>

        <el-main v-if="viewState === 'setting'">
          <el-row :gutter="10">
            <el-col :span="16">
              <el-form label-position="left" label-width="200px">
                <el-form-item label="Dealer Number">
                  <el-input
                    v-model="dealer"
                    type="number"
                    max="8"
                    @input="inputDealer"
                  ></el-input>
                </el-form-item>
              </el-form>
            </el-col>
            <el-col :span="4" style="margin-left: 20px">
              <el-button
                :type="dealer == '' ? 'info' : 'primary'"
                @click="saveDealer"
                :disabled="dealer == ''"
                >Save</el-button
              >
            </el-col>
          </el-row>
        </el-main>
        <el-alert v-if="errorMsg" type="error" :title="errorMsg"> </el-alert>
      </el-container>
    </el-container>
  </div>
</template>
<script>
import moment from "moment";
import Grid from "../components/Grid.vue";
const today = new Date();
const year = today.getFullYear();
const month = today.getMonth().toString().padStart(2, "0");
// const day = today.getDate().toString().padStart(2,'0');
const right_dealer = "12345";
import axios from "axios";
export default {
  name: "MainApp",
  data() {
    return {
      viewState: "export",
      title: "Export Trail Balance",
      dealer: "",
      yearMonth: `${year}${month === "00" ? "12" : month}01`,
      history: [],
      noDealer: true,
      errorMsg: null,

      pickerOptions: {
        disabledDate(time) {
          return time.getTime() > moment().subtract(1, "months");
        },
      },
      itemDataColumns: [
        { label: "Dealer Number", field: "DealerCode" },
        { label: "Line Count", field: "LineCount" },
        { Label: "Submitted Date", field: "SubmittedDate", sortable: "true" },
        { label: "As Of Date", field: "AsOfDate", sortable: "true" },
      ],
    };
  },
  components: {
    grid: Grid,
  },
  computed: {
    showDealerValue() {
      return this.dealer && !this.noDealer ? this.dealer : "undefined";
    },
  },
  methods: {
    setViewState(val) {
      this.viewState = val;
      if (this.noDealer) this.dealer = "";
      this.errorMsg = "";
      switch (val) {
        case "export":
          this.title = "Export Trail Balance";
          break;

        case "history":
          this.title = "View History";
          this.getHistory();
          break;

        case "setting":
          this.title = "Settings";
          break;
        default:
          break;
      }
    },
    getHistory() {
      //get history tab

      //loading true
      // const loading = this.$loading({
      //   lock: true,
      //   text: "Loading",
      //   spinner: "el-icon-loading",
      //   background: "rgba(0, 0, 0, 0.7)",
      // });
      //api call axios.get('gethistory')
      // .then((res)=>this.history = res.response.kubotabalance;
      // loading.close())
      //.catch(()=> {this.errMsg = "some error msg"};
      //loading.close())
      //close loading inside then and catch
      let res = {
        response: {
          RequestDate: "2021-04-12T14:16:25.8743589+00:00",
          KubotaTrialBalances: [
            {
              DMS: "HBS",
              DealerCode: "55000",
              AsOfDate: "2020-12-31T00:00:00",
              LineCount: 264,
              SubmittedDate: "2021-02-24T00:00:00",
              DealerID: 556,
            },
            {
              DMS: "HBS",
              DealerCode: "55000",
              AsOfDate: "2020-11-30T00:00:00",
              LineCount: 258,
              SubmittedDate: "2021-04-07T00:00:00",
              DealerID: 556,
            },
            {
              DMS: "HBS",
              DealerCode: "55000",
              AsOfDate: "2019-12-31T00:00:00",
              LineCount: 280,
              SubmittedDate: "2021-04-07T00:00:00",
              DealerID: 556,
            },
            {
              DMS: "HBS",
              DealerCode: "550100",
              AsOfDate: "2019-11-30T00:00:00",
              LineCount: 258,
              SubmittedDate: "2018-04-07T00:00:00",
              DealerID: 556,
            },
            {
              DMS: "HBS",
              DealerCode: "551000",
              AsOfDate: "2012-12-31T00:00:00",
              LineCount: 280,
              SubmittedDate: "2014-04-07T00:00:00",
              DealerID: 556,
            },
          ],
        },
      };
      this.history = res.response.KubotaTrialBalances.map((i) => {
        return {
          DMS: i.DMS,
          DealerCode: i.DealerCode,
          AsOfDate: moment(i.AsOfDate).format("MM/DD/YYYY"),
          LineCount: i.LineCount,
          SubmittedDate: moment(i.SubmittedDate).format("MM/DD/YYYY h:mm a"),
          DealerID: 556,
        };
      });
      // setTimeout(() => {
      //   loading.close();
      // }, 2000);
    },
    submitData() {
      axios
        .post(`http://129.50.2.5/netview/cm/KDAsvc`, {
          body: {
            curloc: "loc1",
            userid: "hbs",
            action: "submit-tb",
            dealer_number: this.dealer,
            selected_month: this.yearMonth,
          },
        })
        .then()
        .catch();
    },
    saveDealer() {
      // axios
      //   .post(`http://129.50.2.5/netview/cm/KubotaDealersvc`, {
      //     body: {
      //       curloc: "loc0",
      //       userid: "hbs",
      //       action: "update-dealer",
      //       location: "loc0",
      //       program_name: "kda",
      //       dealer_number: this.dealer,
      //     },
      //   })
      //   .then((res) => {
      // alert('successfully added')
      // })
      //   .catch((e) => {
      //axios failed and catchc block
      // this.errorMsg = "Dealer Number not foundd"; //error msg may come from backend do check
      // this.noDealer = true;
      // });
      //for local test for error
      if (this.dealer != right_dealer) {
        //axios failed and catchc block
        this.errorMsg = "Dealer Number not found"; //catch it in catch of axios
        this.noDealer = true;
      } else {
        this.noDealer = false;
        this.$alert("successfully added");
      }
    },
    inputDealer() {
      if (this.dealer.length > 8) this.dealer = this.dealer.substring(0, 8);
    },
    sortBySubmittedDate(a,b) {
      // this.history = this.history.sort((a,b) => {
        if(moment(a.SubmittedDate).isSameOrBefore(moment(b.SubmittedDate))) 
          return -1;

        if(moment(a.SubmittedDate).isSameOrAfter(moment(b.SubmittedDate))) 
          return 1;
          else 
          return 0;
      // });
      
    },
    sortByAsOfDate(a,b) {

        if(moment(a.AsOfDate).isSameOrBefore(moment(b.AsOfDate))) 
          return -1;

        if(moment(a.AsOfDate).isSameOrAfter(moment(b.AsOfDate))) 
          return 1;
          else 
          return 0;
    }
  },
};
</script>
<style scoped>
.el-aside {
  overflow: hidden;
}
ul {
  list-style-type: none;
  text-align: left;
  padding: 0px;
  margin: 0;
  border: 1px solid black;
}

ul > li {
  padding-left: 30px;
  height: 40px;
  margin: auto;
  cursor: pointer;
  color: blue;
}
ul :hover {
  background-color: rgb(145, 138, 127);
}

.title-wrap {
  background-color: blue;
  pointer-events: none;
}

.title-wrap-active {
  background-color: rgb(145, 138, 127);
  color: black;
}
.title {
  color: white;
  padding-top: 10px;
}
.sub-menu-title {
  padding-top: 10px;
}
.sub-menu-title :hover {
  background: rgb(161, 159, 154);
}
.el-date-editor.el-input {
  width: 100%;
}
.el-alert {
  padding: 14px;
  margin: 10px;
  border: 2px solid red;
}
.el-alert .el-alert__description {
  font-size: 20px;
}
</style>
