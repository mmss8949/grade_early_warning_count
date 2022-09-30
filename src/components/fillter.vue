<template>
  <v-container fluid class="pa-0">
    <!-- useless-->
    <!--
    <div class="collapse-div" style="display:none;">
      <button class="collapse" @click="collapseSearch()" type="button">
        <i class="fas fa-minus" v-if="!collapse"></i>
        <i class="fas fa-plus" v-else></i>
      </button>
    </div>
    <div v-if="collapse">
      <v-row no-gutters class="search-text">
        <v-col lg="2" md="2" sm="2" cols="12"></v-col>
        <v-col lg="9" md="9" sm="9" cols="12" class="ml-4">
          <v-row no-gutters>
            <p>查詢條件 類別:</p>
            <p v-for="(item, index) in searchCon" :key="index" class="ml-1">
              {{ item }}
            </p>
            <p>, 學年期: {{ startDate }} ~{{ endDate }}</p>
          </v-row>
        </v-col>
      </v-row>
    </div>-->
    <div id="collapse-content">
      <v-row no-gutters class="d-flex align-baseline">
        <v-col lg="1" md="1" sm="3" cols="12">
          <div
            class="d-flex justify-lg-end justify-md-end justify-sm-end justify-start fillterTitle"
          >
            類別
          </div>
        </v-col>
        <v-col lg="10" md="10" sm="8" cols="12" class="ml-4">
          <v-row no-gutters>
            <div v-if="checkListProp && checkListProp.length != 0" class="mr-4">
              <v-checkbox
                v-model="checkboxAll"
                label="全部"
                @click="selectAll"
                color="#007bff"
                class="checkbox mt-0 mb-0"
              ></v-checkbox>
            </div>
            <div
              v-for="(item, index) in checkListProp"
              :key="index"
              class="mr-4"
            >
              <v-checkbox
                v-model="checkbox"
                :label="item.title"
                :value="item.no"
                color="#007bff"
                class="checkbox mt-0"
                @click="checkAllSelect"
              ></v-checkbox>
            </div>
          </v-row>
        </v-col>
      </v-row>
      <v-row no-gutters class="d-flex align-baseline">
        <v-col lg="1" md="1" sm="3" cols="12">
          <div
            class="d-flex justify-lg-end justify-md-end justify-sm-end justify-start fillterTitle"
          >
            學年期
          </div>
        </v-col>
        <v-col
          lg="9"
          md="9"
          sm="8"
          cols="12"
          class="mt-lg-0 mt-md-0 mt-sm-0 mt-3 ml-4"
        >
          <v-row no-gutters class="d-flex align-baseline mb-4">
            <v-col lg="2" md="3" sm="4" cols="12" class="d-flex justify-start">
              <v-text-field
                outlined
                dense
                hide-details="auto"
                :rules="rules"
                v-model="startDate"
                placeholder="開始學年期"
              ></v-text-field>
            </v-col>
            <v-col
              lg="1"
              md="1"
              sm="1"
              cols="12"
              class="mt-lg-0 mb-lg-0 mt-md-0 mb-md-0 mt-1 mb-1"
              >至</v-col
            >
            <v-col lg="2" md="3" sm="4" cols="12" class="d-flex justify-start">
              <v-text-field
                outlined
                dense
                hide-details="auto"
                :rules="rules"
                v-model="endDate"
                placeholder="結束學年期"
              ></v-text-field>
            </v-col>
          </v-row>
        </v-col>
      </v-row>
      <v-row no-gutters class="d-flex align-baseline">
        <v-col lg="1" md="1" sm="3" cols="12">
          <div
            class="d-flex justify-lg-end justify-md-end justify-sm-end justify-start fillterTitle"
          >
            次數
          </div>
        </v-col>
        <v-col
          lg="10"
          md="10"
          sm="8"
          cols="12"
          class="mt-lg-0 mt-md-0 mt-sm-0 mt-3 ml-4"
        >
          <v-row no-gutters class="d-flex align-baseline">
            <v-col lg="2" md="3" sm="4" cols="12" class="d-flex justify-start">
              <v-text-field
                outlined
                dense
                hide-details="auto"
                :rules="numberRules"
                v-model="count"
              ></v-text-field>
            </v-col>
          </v-row>
        </v-col>
      </v-row>
      <v-row class="no-gutters mt-3">
        <v-col lg="1" md="1" sm="3" cols="0"></v-col>
        <v-col lg="9" md="9" sm="8" cols="12" class="d-flex justify-start ml-4">
          <input
            type="button"
            class="btnSearch"
            style="width: 200px"
            @click="search()"
            value="Excel下載"
          />
        </v-col>
      </v-row>
    </div>
  </v-container>
</template>

<script>
export default {
  props: ["checkListProp"],
  components: {},
  data: function () {
    return {
      collapse: false,
      result: [],
      propValue: [],
      checkbox: [],
      checkboxAll: false,
      count:0,
      propCount: 0,
      pageCount: 10,
      page: 1,
      pageLen: 0,
      startDate: "",
      endDate: "",
      rules: [
        (value) => !!value || "必填",
        (value) => (value || "").length == 4 || "必為4位數",
        (value) => !isNaN(value) || "必須為數字",
      ],
      numberRules: [(value) => !isNaN(value) || "必須為數字"],
    };
  },
  created() {
    const self = this;
  },
  mounted() {},
  methods: {
    search() {
      const self = this;
      self.$emit("searchEmit", {
        checkList: self.checkbox,
        startDate: self.startDate,
        endDate: self.endDate,
        count:self.count
      });
    },
    selectAll() {
      const self = this;
      console.log(2);
      if (
        self.checkboxAll &&
        self.checkListProp &&
        self.checkListProp.length != 0
      ) {
        console.log(3);
        self.checkbox = [];
        self.checkListProp.forEach((item) => {
          self.checkbox.push(item.no);
        });
        console.log(4);
      } else if (
        !self.checkboxAll &&
        self.checkListProp &&
        self.checkListProp.length != 0
      ) {
        self.checkbox = [];
      }
    },
    checkAllSelect() {
      const self = this;
      if (self.checkListProp && self.checkListProp.length != 0) {
        if (self.checkbox.length == self.checkListProp.length)
          self.checkboxAll = true;
        else if (self.checkbox.length == 0) self.checkboxAll = false;
      }
    },
    setDate() {
      Date.prototype.yyyymmdd = function () {
        var mm = this.getMonth() + 1; // getMonth() is zero-based
        var dd = this.getDate();

        return [
          this.getFullYear(),
          "-",
          (mm > 9 ? "" : "0") + mm,
          "-",
          (dd > 9 ? "" : "0") + dd,
        ].join("");
      };

      const self = this;
      var nowTime = new Date();
      var start = new Date();
      var end = new Date();
      start = start.setDate(nowTime.getDate() - 30);
      end = end.setDate(nowTime.getDate() + 30);
      self.startDate = new Date(start).yyyymmdd();
      self.endDate = new Date(end).yyyymmdd();
    },
    collapseSearch() {
      const self = this;
      self.collapse = !self.collapse;

      $("#collapse-content").slideToggle();
    },
  },
  watch: {
    checkListProp: function () {
      const self = this;
      self.checkboxAll = true;
      self.checkbox = [];
      self.checkListProp.forEach((item) => {
        self.checkbox.push(item.no);
      });
    },
  },
  computed: {
    searchCon: function () {
      const self = this;
      var array = [];
      self.checkListProp.forEach((itemA) => {
        self.checkbox.forEach((itemB) => {
          if (itemA.no == itemB) array.push(itemA.title);
        });
      });
      return array;
    },
  },
};
</script>

<style lang="scss">
.btnSearch {
  color: #fff !important;
  background-color: #007bff;
  border-color: #007bff;
  display: inline-block;
  font-weight: 400;
  text-align: center;
  white-space: nowrap;
  vertical-align: middle;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  border: 1px solid transparent;
  padding: 0.375rem 0.75rem;
  font-size: 1rem;
  line-height: 1.5;
  border-radius: 0.25rem;
  transition: color 0.15s ease-in-out, background-color 0.15s ease-in-out,
    border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
}
.fillterTitle {
  font-size: 1rem;
  font-weight: 400;
  color: black;
  @media (max-width: 650px) {
    font-size: 20px;
    font-weight: bold;
  }
}

.flatpickrInput {
  border-radius: 5px;
  padding: 3px 6px;
  border: 1px solid #ced4da;
  font-size: 20px;
}
.form-control {
  width: 100%;
}
.flatpickr-wrapper {
  width: 100%;
}
.checkbox {
  .v-input__slot {
    margin-bottom: 0px;
  }
  label {
    font-size: 20px;
    color: black !important;
  }
}
.collapse-div {
  position: absolute;
  top: -1px;
  left: -1px;
  .collapse {
    width: 30px;
    height: 30px;
    color: #fff;
    background-color: #007bff;
    border-color: #007bff;
    padding: 0.25rem 0.5rem;
    font-size: 0.875rem;
    line-height: 1.5;
    border-radius: 0.2rem;
  }
}
.search-text {
  font-size: 1rem;
  color: #212529;
  font-weight: 400;
}
</style>
