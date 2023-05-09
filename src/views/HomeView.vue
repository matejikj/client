<template>
  <body>
    <v-container>
      <v-row>
        <v-col cols="12" md="3">
          <v-file-input
            label="Select dataset"
            @change="fileChanged"
            v-model="file"
            truncate-length="15"
          ></v-file-input>
        </v-col>
        <v-col cols="12" md="6"> </v-col>
        <v-col cols="12" md="3">
          <v-file-input
            label="Load config file"
            @change="configFileChanged"
            v-model="configFile"
            truncate-length="15"
          ></v-file-input>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" md="12">
          <v-data-table
            :headers="headers"
            :items="firstLines"
            hide-default-footer
            class="elevation-1"
          ></v-data-table>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" md="3">
          <v-combobox
            v-model="selectedColumns"
            :items="headers.map((item) => item.text)"
            label="Select columns for transformations"
            multiple
            chips
            @change="comboboxChanged"
          ></v-combobox>
        </v-col>
        <v-col cols="12" md="9">
          <v-card elevation="3">
            <v-container v-if="this.selectedColumns.length === 1">
              <v-subheader>Filter</v-subheader>
              <v-row>
                <v-col cols="12" md="4">
                  <v-checkbox
                    v-model="ignoreCheckbox"
                    label="ignore"
                  ></v-checkbox>
                </v-col>
                <v-col cols="12" md="4">
                  <v-checkbox
                    v-model="dropnaCheckbox"
                    label="dropna"
                  ></v-checkbox>
                </v-col>
                <v-col cols="12" md="4">
                  <v-checkbox
                    v-model="timeseriesCheckbox"
                    label="timeseries"
                  ></v-checkbox>
                </v-col>
              </v-row>
              <v-subheader>Transform</v-subheader>
              <v-row>
                <v-col cols="12" md="4">
                  <v-combobox
                    v-model="selectedTarget"
                    :items="headers.map((item) => item.text)"
                    label="select target column"
                  ></v-combobox>
                </v-col>
              </v-row>

              <v-subheader>Target</v-subheader>
              <v-row>
                <v-col cols="12" md="4">
                  <v-combobox
                    v-model="selectedTarget"
                    :items="headers.map((item) => item.text)"
                    label="select target column"
                  ></v-combobox>
                </v-col>
              </v-row>
              <v-subheader>Grouping</v-subheader>
              <v-row>
                <v-col cols="12" md="4">
                  <v-combobox
                    v-model="selectedTarget"
                    :items="headers.map((item) => item.text)"
                    label="select target column"
                  ></v-combobox>
                </v-col>
                <v-text-field
                  v-model="groupingPeriod"
                  label="grouping period"
                ></v-text-field>
              </v-row>
            </v-container>
            <v-container v-if="this.selectedColumns.length > 1">
              <v-subheader>Filter</v-subheader>
              <v-row>
                <v-col cols="12" md="4">
                  <v-checkbox
                    v-model="ignoreCheckbox"
                    label="ignore"
                  ></v-checkbox>
                </v-col>
                <v-col cols="12" md="4">
                  <v-checkbox
                    v-model="dropnaCheckbox"
                    label="dropna"
                  ></v-checkbox>
                </v-col>
                <v-col cols="12" md="4">
                  <v-checkbox
                    v-model="timeseriesCheckbox"
                    label="timeseries"
                  ></v-checkbox>
                </v-col>
              </v-row>
            </v-container>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </body>
</template>

<script>
import axios from "axios";
import Papa from "papaparse";
import { saveAs } from "file-saver";

export default {
  name: "HomeView",
  data: () => ({
    file: undefined,
    configFile: undefined,
    config: {},
    firstLines: [],
    // columns: [],
    headers: [],
    selectedColumns: [],
    selectedTarget: "",
    parsed: false,
    ignoreCheckbox: false,
    dropnaCheckbox: false,
    timeseriesCheckbox: false,
    dataTypes: ["double", "int", "str"],
    selectedDataType: "",
  }),
  computed: {},
  methods: {
    downloadConfig() {
      const fileName = "config.json";
      // Create a blob of the data
      const fileToSave = new Blob([JSON.stringify(this.config)], {
        type: "application/json",
      });

      // Save the file
      saveAs(fileToSave, fileName);
    },
    setConfig(i) {
      this.config = JSON.parse(i.target.result);
    },
    configFileChanged() {
      const reader = new FileReader();
      reader.onload = (e) => this.setConfig(e);
      reader.readAsText(this.configFile);
    },
    fileChanged() {
      Papa.parse(this.file, {
        header: true,
        step: function (row) {
          if (this.headers.length === 0) {
            row.meta.fields.forEach((element) => {
              this.headers.push({
                text: element,
                value: element,
              });
            });
          }
          if (this.firstLines.length < 5) {
            this.firstLines.push(row.data);
          }
        }.bind(this),
        // completed: function (res) {
        //   // this.columns = this.headers.map((item) => {
        //   //   return item.text;
        //   // });
        // }.bind(this),
      });
    },
    comboboxChanged() {
      switch (this.selectedColumns.length) {
        case 0:
          console.log(0);
          break;
        case 1:
          console.log(1);
          break;
        default:
          console.log(10);
          break;
      }
    },
  },
};
</script>
