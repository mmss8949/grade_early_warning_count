<template>
  <div id="app">
    <v-app>
      <div
        id="loading"
        :style="{ display: loading ? 'flex' : 'none' }"
      >
        <v-progress-circular
          indeterminate
          color="black"
          :size="70"
        ></v-progress-circular>
      </div>
      <v-container
        fluid
        class="pl-1 pr-2 pt-0"
      >
        <v-row no-gutters>
          <Fillter
            :checkListProp="checkList"
            @searchEmit="search"
          ></Fillter>
        </v-row>
      </v-container>
    </v-app>
  </div>
</template>

<script>
import axios from "axios";
import Fillter from "./components/fillter.vue";
import testFillter from "../public/grade_early_warning_count/tools/json/test_fillter.json";
import excelTest from "../public/grade_early_warning_count/tools/json/excelTest.json";

import * as XLSX from 'xlsx/xlsx.mjs';

/* load 'fs' for readFile and writeFile support */
import * as fs from 'fs';
XLSX.set_fs(fs);

/* load 'stream' for stream support */
import { Readable } from 'stream';
XLSX.stream.set_readable(Readable);

/* load the codepage support library for extended support with older formats  */
import * as cpexcel from 'xlsx/dist/cpexcel.full.mjs';
XLSX.set_cptable(cpexcel);



export default {
  name: "App",
  components: {
    Fillter,
  },
  data: () => ({
    testMode: false,
    loading: false,
    demo: false,
    checkList: [],
    content: [],
    fileName: "",
    sheetName: "",
  }),

  created() {
    const self = this;
    if (
      location.href.indexOf("localhost") != -1 ||
      location.href.indexOf("git") != -1
    )
      self.testMode = true;
    else self.testMode = false;

    self.getCheckList();
  },
  updated() {
    const self = this;
    self.updateHeight();
  },
  methods: {
    axios(url, method, data, callFunction, errorMsg) {
      const self = this;
      self.loading = true;
      axios({
        url: url + `?gGroups_i=${gGroups}&gSys_s=${gSys_s}&gFunc_s=${gFunc_s}`,
        method: method,
        data: data,
        "Content-Type": "application/json",
      })
        .then(function (response) {
          if (response.data.result == 1) {
            callFunction(response.data.data);
          } else {
            parent.sysDisplayMessage(
              "",
              -1,
              "",
              "",
              null,
              errorMsg + "," + response.data.msg
            );
          }
        })
        .catch(function (error) {
          self.loading = false;
          console.log(error);
          parent.sysDisplayMessage("", -1, "", "", null, "連線失敗");
        });
    },
    getCheckList() {
      const self = this;
      if (location.href.indexOf("localhost") != -1 || self.testMode) {
        setTimeout(function () {
          self.setCheckList(testFillter.data);
        }, 1000);
      } else {
        self.axios(
          "./kernel/warning_B.php",
          "get",
          {},
          self.setCheckList,
          "抓取類別失敗"
        );
      }
    },
    setCheckList(data) {
      const self = this;
      self.loading = false;
      self.checkList = data;
    },
    //更新視窗高度
    updateHeight() {
      const self = this;
      self.$nextTick(() => {
        setTimeout(() => {
          if (document.getElementById("app").scrollHeight > 800) {
            var windowHeight = document.body.scrollHeight + 10;
            window.parent.postMessage(
              '{"event": "changeHeight", "value": ' + windowHeight + "}",
              "*"
            );
          }
        }, 1000);
      });
    },
    //search
    search(data) {
      const self = this;
      if (self.testMode) {
        self.downloadFile({})
      } else
        self.axios(
          "./kernel/counseling_number_B.php",
          "post",
          {
            type: data.checkList,
            year_semester_s: data.startDate,
            year_semester_e: data.endDate,
            number: data.count,
          },
          self.downloadFile,
          "產生失敗"
        );
    },
    //Get excel
    downloadFile(data) {
      const self = this;
      self.fileName = "預警報告";
      self.sheetName = "頁籤";
      // if (data.name.length > 30) self.fileName = data.name.substr(0, 30);
      // else self.fileName = data.name;

      // if (data.sheet.length > 30) self.sheetName = data.sheet.substr(0, 30);
      // else self.sheetName = data.sheet;

      // self.sheetName = self.sheetName
      //   .replace(/:/gi, "")
      //   .replace(/\：/gi, "")
      //   .replace(/\\/gi, "")
      //   .replace(/\//gi, "")
      //   .replace(/？/gi, "")
      //   .replace(/\?/gi, "")
      //   .replace(/＊/gi, "")
      //   .replace(/\*/gi, "");

      if (location.href.indexOf("localhost") != -1 || self.demo) {
        // local
        self.exportExcel(excelTest.data);
      } else {
        self.exportExcel(data);
      }
    },
    //Export excel
    exportExcel(excel) {
      const self = this;
      var wb = XLSX.utils.book_new();
      for (var x = 0; x < excel.sheet.length; x++) {
        var content = [];
        var row = excel.sheet[x].col;
        var header = [];

        for (var i = 1; i <= row; i++) {
          header[i - 1] = excel.sheet[x].content[0]["col" + i];
        }
        for (var i = 1; i < excel.sheet[x].content.length; i++) {
          var obj = {};
          for (var j = 1; j <= row; j++) {
            obj[header[j - 1]] = excel.sheet[x].content[i]["col" + j];
          }
          content.push(obj);
        }
        var ws = XLSX.utils.json_to_sheet(content, { header: header });

        XLSX.utils.book_append_sheet(
          wb,
          ws,
          excel.sheet[x].sheetName
            ? excel.sheet[x].sheetName.length == 0
              ? self.sheetName
              : excel.sheet[x].sheetName.length > 30
              ? excel.sheet[x].sheetName
                  .substr(0, 30)
                  .replace(/:/gi, "")
                  .replace(/\：/gi, "")
                  .replace(/\\/gi, "")
                  .replace(/\//gi, "")
                  .replace(/？/gi, "")
                  .replace(/\?/gi, "")
                  .replace(/＊/gi, "")
                  .replace(/\*/gi, "")
              : excel.sheet[x].sheetName
                  .replace(/:/gi, "")
                  .replace(/\：/gi, "")
                  .replace(/\\/gi, "")
                  .replace(/\//gi, "")
                  .replace(/？/gi, "")
                  .replace(/\?/gi, "")
                  .replace(/＊/gi, "")
                  .replace(/\*/gi, "")
            : self.sheetName
        );
      }

      XLSX.writeFile(wb, self.fileName + ".xlsx");
      self.loading = false;
    },
  },
};
</script>

<style lang="scss">
#app {
  font-family: Microsoft JhengHei;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#loading {
  width: 100%;
  height: 100%;
  position: fixed;
  top: 0;
  z-index: 9999;
  align-items: center;
  justify-content: center;
  background-color: rgba(0, 0, 0, 0.6);
}

.list-text {
  color: #212529;
  font-size: 18px;
  margin-left: 8px;
  padding-top: 3px;
  font-weight: 900;
}
.content-list {
  border-top: 1px solid #e1dfdd !important;
}
</style>
