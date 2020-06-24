<template>
  <div>
    <v-row>
      <v-col class="mx-4">
        <div class="about">
          <v-card class="ma-2" max-width="500" outlined>
            <v-list-item three-line>
              <v-list-item-content>
                <v-banner class="mb-3">Выберите файл для конвертирования</v-banner>
                <v-file-input label="File input" outlined dense v-model="fileInput"></v-file-input>
              </v-list-item-content>
            </v-list-item>
          </v-card>
          <v-card class="ma-2" max-width="500" outlined>
            <v-list-item three-line>
              <v-list-item-content>
                <v-banner class="mb-3">Выберите файл для сравнения</v-banner>
                <v-file-input label="File input" outlined dense v-model="fileInputTwo"></v-file-input>
              </v-list-item-content>
            </v-list-item>
          </v-card>
          <v-row class="ma-1">
            <v-btn
              class="ma-2"
              tile
              color="primary"
              :dark="!convertFileButton"
              :disabled="convertFileButton"
              @click="convertData"
            >Найти совпадения</v-btn>
            <download-excel :data="json_data" type="xlsx">
              <v-btn class="ma-2" tile color="primary">
                Download Data
                <v-icon>mdi-vuetify</v-icon>
              </v-btn>
            </download-excel>
          </v-row>
          <v-row class="ma-1" v-if="!convertFileButton">
            <v-col class="grey lighten-4">
              <v-banner class="mb-3">Выберите столбцы для сравнения</v-banner>
              <v-select
                v-model="selectOne"
                :items="colOne"
                label="Первая таблица"
                outlined
                required
                @change="setValueOne"
              ></v-select>
            </v-col>
            <v-col class="grey lighten-4">
              <v-banner class="mb-3">Выберите столбцы для сравнения</v-banner>
              <v-select
                v-model="selectTwo"
                :items="colTwo"
                label="Вторая таблица"
                required
                outlined
                @change="setValueTwo"
              ></v-select>
            </v-col>
          </v-row>
          <router-link to="/" v-if="false">
            <v-btn class="mx-1" color="primary">Send Message</v-btn>
          </router-link>
        </div>
      </v-col>
    </v-row>
    <v-row v-if="fileInput">
      <v-col class="mx-4">
        <template>
          <v-data-table
            :headers="tableHeaders"
            :items="tableValues"
            :items-per-page="5"
            class="elevation-1"
          ></v-data-table>
        </template>
      </v-col>
    </v-row>
  </div>
</template>

<script>
import readXlsxFile from "read-excel-file";
import createNewExcelFile from "@/utilites/util.js";

export default {
  data: () => ({
    selectOne: null,
    selectTwo: null,
    selectedValueOne: null,
    selectedValueTwo: null,
    colOne: [],
    colTwo: [],
    fileInput: null,
    fileInputTwo: null,
    fileData: null,
    tableHeaders: [],
    tableValues: [],
    json_fields: {
      "Complete name": "name",
      City: "city",
      Telephone: "phone"
    },
    json_data: [],
    json_meta: [
      [
        {
          key: "charset",
          value: "utf-8"
        }
      ]
    ]
  }),
  computed: {
    convertFileButton: {
      get: function() {
        return (this.fileInput != null && this.fileInputTwo != null) == false;
      },
      set: function() {}
    }
  },
  watch: {
    fileInput: function(e) {
      if (e && e.size > 0) {
        readXlsxFile(e).then(rows => {
          this.fileData = rows;
          this.colOne = rows[0].map((item, index) => {
            return `Строка ${index + 1}`;
          });
        });
        this.tableHeaders = [
          {
            text: "Номер в первой таблице",
            value: "first"
          },
          {
            text: "Номер во второй таблице",
            value: "second"
          },
          {
            text: "Совпавшее значение",
            value: "third"
          }
        ];
      }
    },
    fileInputTwo: function(e) {
      readXlsxFile(e).then(rows => {
        this.fileDataTwo = rows;
        this.colTwo = rows[0].map((item, index) => {
          return `Строка ${index + 1}`;
        });
      });
    }
  },
  methods: {
    convertData() {
      this.tableValues = [];
      this.json_data = [];
      let dStart = new Date();
      // console.log(this.selectedValueOne, this.selectedValueOne);
      let comOne = this.selectedValueOne-1;
      let comTwo = this.selectedValueTwo-1;
      console.log(comOne, comTwo);
      
      this.fileData.map(element => {
        this.fileDataTwo.map(elementTwo => {
          if (element[comOne] != "А" && element[comOne] == elementTwo[comTwo])
            this.tableValues.push({
              first: element[0],
              second: elementTwo[0],
              third: element[comOne]
            });
          this.json_data = this.tableValues;
        });
      });
      let dFinish = new Date();
      console.log(dFinish - dStart);
      setTimeout(() => createNewExcelFile(), 1000);
    },
    setValueOne(e) {
      this.selectedValueOne = (e.split(" "))[1]
    },
    setValueTwo(e) {
      this.selectedValueTwo = (e.split(" "))[1]
    }
  }
};
</script>
