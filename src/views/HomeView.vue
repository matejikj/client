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
            :items-per-page="5"
            class="elevation-1"
          ></v-data-table>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" md="12">
          <v-card elevation="3">
            <v-container>
              <v-row>
                <v-subheader>Begin</v-subheader>
              </v-row>
              <v-row>
                <v-col>
                  <v-select
                    v-model="index"
                    :items="headers.map((item) => item.text)"
                    label="index"
                  ></v-select>
                </v-col>
                <v-col>
                  <v-select
                    v-model="target"
                    :items="headers.map((item) => item.text)"
                    label="target"
                  ></v-select>
                </v-col>
                <v-col>
                  <v-checkbox
                    v-model="isTimeseries"
                    label="timeseries"
                  ></v-checkbox>
                </v-col>
                <v-col>
                  <v-select
                    v-if="isTimeseries"
                    v-model="timeseries"
                    :items="headers.map((item) => item.text)"
                    label="timeseries"
                  ></v-select>
                </v-col>
              </v-row>
              <v-row>
                <v-btn v-if="!addingOperation" @click="addingOperation = true">
                  Add new transformation
                </v-btn>
              </v-row>
              <v-row>
                <v-col cols="12" sm="10">
                  <v-select
                    v-if="addingOperation"
                    v-model="newTransformation"
                    :items="transformationTypes"
                    label="transformation"
                  ></v-select>
                </v-col>
                <v-col cols="12" sm="2">
                  <v-btn v-if="addingOperation" @click="addOperation">
                    Add new transformation
                  </v-btn>
                </v-col>
              </v-row>
              <v-row
                v-bind:key="index"
                v-for="(transformation, index) in transformations"
              >
                <v-container>
                  <v-row>
                    <v-col cols="12" sm="12"
                      >{{ transformation.type }}
                      <v-btn @click="removeItem(index)" fab x-small color="red"
                        >-</v-btn
                      ></v-col
                    >
                    <v-col
                      cols="12"
                      sm="6"
                      v-if="transformation.type === 'RENAME'"
                    >
                      <v-select
                        v-model="transformation.column"
                        :items="headers.map((item) => item.text)"
                        label="column"
                      ></v-select>
                    </v-col>
                    <v-col
                      cols="12"
                      sm="6"
                      v-if="transformation.type === 'RENAME'"
                    >
                      <v-text-field
                        v-model="transformation.value"
                        label="column"
                      ></v-text-field>
                    </v-col>

                    <v-col
                      cols="12"
                      sm="4"
                      v-if="transformation.type === 'REPLACE'"
                    >
                      <v-combobox
                        v-model="transformation.column"
                        :items="headers.map((item) => item.text)"
                        label="Select column"
                        multiple
                        chips
                      ></v-combobox>
                    </v-col>
                    <v-col
                      cols="12"
                      sm="8"
                      v-if="transformation.type === 'REPLACE'"
                    >
                      <v-row
                        v-bind:key="i"
                        v-for="(pair, i) in transformation.values"
                      >
                        <v-col>
                          <v-text-field
                            v-model="pair.old_value"
                            label="value"
                          ></v-text-field>
                        </v-col>
                        <v-col>
                          <v-text-field
                            v-model="pair.new_value"
                            label="new value"
                          ></v-text-field>
                        </v-col>
                      </v-row>
                      <v-row>
                        <v-col>
                          <v-btn @click="addReplace(transformation.values)">
                            Add another values
                          </v-btn>
                        </v-col>
                      </v-row>
                    </v-col>

                    <v-col v-if="transformation.type === 'FACTORIZE'">
                      <v-combobox
                        v-model="transformation.column"
                        :items="headers.map((item) => item.text)"
                        label="Select columns"
                        multiple
                        chips
                      ></v-combobox>
                    </v-col>

                    <v-col v-if="transformation.type === 'IGNORE'">
                      <v-combobox
                        v-model="transformation.column"
                        :items="headers.map((item) => item.text)"
                        label="column"
                        multiple
                        chips
                      ></v-combobox>
                    </v-col>

                    <v-col v-if="transformation.type === 'DROP_DUPLICATES'">
                      <v-combobox
                        v-model="transformation.column"
                        :items="headers.map((item) => item.text)"
                        label="column"
                        multiple
                        chips
                      ></v-combobox>
                    </v-col>

                    <v-col v-if="transformation.type === 'IS_IN'">
                      <v-select
                        v-model="transformation.column"
                        :items="headers.map((item) => item.text)"
                        label="column"
                      ></v-select>
                    </v-col>
                    <v-col v-if="transformation.type === 'IS_IN'">
                      <v-row
                        v-bind:key="i"
                        v-for="(pair, i) in transformation.values"
                      >
                        <v-col>
                          <v-text-field
                            v-model="pair.value"
                            label="column"
                          ></v-text-field>
                        </v-col>
                      </v-row>
                      <v-row>
                        <v-col>
                          <v-btn @click="addIsIn(transformation.values)">
                            Add another values
                          </v-btn>
                        </v-col>
                      </v-row>
                    </v-col>

                    <v-col
                      cols="12"
                      sm="6"
                      v-if="transformation.type === 'AS_TYPE'"
                    >
                      <v-combobox
                        v-model="transformation.column"
                        :items="headers.map((item) => item.text)"
                        label="column"
                        multiple
                        chips
                      ></v-combobox>
                    </v-col>
                    <v-col
                      cols="12"
                      sm="6"
                      v-if="transformation.type === 'AS_TYPE'"
                    >
                      <v-select
                        v-model="transformation.value"
                        :items="asTypeValues"
                        label="column"
                      ></v-select>
                    </v-col>

                    <v-col v-if="transformation.type === 'NA'">
                      <v-combobox
                        v-model="transformation.column"
                        :items="headers.map((item) => item.text)"
                        label="column"
                        multiple
                        chips
                      ></v-combobox>
                    </v-col>
                    <v-col v-if="transformation.type === 'NA'">
                      <v-select
                        v-model="transformation.value"
                        :items="nAValues"
                        label="column"
                      ></v-select>
                    </v-col>
                  </v-row>
                </v-container>
              </v-row>
            </v-container>
            <v-btn @click="downloadConfig()">Save config</v-btn>
            <v-btn @click="exampleRequest()">Example</v-btn>
          </v-card>
        </v-col>
      </v-row>
      <v-row>
        {{ exampleResult }}
      </v-row>
      <v-row>
        <v-col cols="12" md="12">
          <v-data-table
            :headers="exampleHeaders"
            :items="exampleFirstLines"
            class="elevation-1"
          ></v-data-table>
        </v-col>
      </v-row>
    </v-container>
  </body>
</template>

<script>
import axios from "axios";
import Papa from "papaparse";
import { saveAs } from "file-saver";

const START_GROUP = "START";
const PROJECTION_GROUP = "PROJECTION";
const FILTER_GROUP = "FILTER";
const TRANSFORM_GROUP = "TRANSFORM";
const API = "http://localhost:5000/";

const DROP_DUPLICATES = "DROP_DUPLICATES";
const RENAME = "RENAME";
const IGNORE = "IGNORE";
const IS_IN = "IS_IN";
const REPLACE = "REPLACE";
const FACTORIZE = "FACTORIZE";
const AS_TYPE = "AS_TYPE";
const NA = "NA";

const transformGroups = {
  index: START_GROUP,
  target: START_GROUP,
  timeseries: START_GROUP,
  rename: PROJECTION_GROUP,
  ignore: FILTER_GROUP,
  isIn: FILTER_GROUP,
  dropDuplicates: FILTER_GROUP,
  nA: TRANSFORM_GROUP,
  asType: TRANSFORM_GROUP,
  replace: TRANSFORM_GROUP,
  factorize: TRANSFORM_GROUP,
};

export default {
  name: "HomeView",
  data: () => ({
    file: undefined,
    configFile: undefined,
    config: {
      START_GROUP: [],
      PROJECTION_GROUP: [],
      FILTER_GROUP: [],
      TRANSFORM_GROUP: [],
    },
    exampleFirstLines: [],
    // columns: [],
    exampleHeaders: [],
    exampleResult: '',
    firstLines: [],
    // columns: [],
    headers: [],
    selectedColumns: [],
    nAValues: ["<nothing>", "drop", "blank", "zero"],
    asTypeValues: ["category", "string", "int64", "float64"],
    // Start
    index: "",
    target: "",
    isTimeseries: false,
    timeseries: "",
    addingOperation: false,
    transformations: [],
    transformationTypes: [
      DROP_DUPLICATES,
      RENAME,
      IGNORE,
      IS_IN,
      REPLACE,
      FACTORIZE,
      AS_TYPE,
      NA,
    ],
    newTransformation: "",
    defaultTransformations: {
      RENAME: {
        type: RENAME,
        column: "",
        value: "",
      },
      DROP_DUPLICATES: {
        type: DROP_DUPLICATES,
        column: "",
      },
      IGNORE: {
        type: IGNORE,
        column: "",
      },
      AS_TYPE: {
        type: AS_TYPE,
        column: "",
        value: "",
      },
      NA: {
        type: NA,
        column: "",
        value: "",
      },
      IS_IN: {
        type: IS_IN,
        column: "",
        values: [
          {
            value: "",
          },
        ],
      },
      FACTORIZE: {
        type: FACTORIZE,
        column: "",
      },
      REPLACE: {
        type: REPLACE,
        column: "",
        values: [
          {
            old_value: "",
            new_value: "",
          },
        ],
      },
    },
  }),
  computed: {
    selectedColumnsCount: function () {
      return this.selectedColumns.length;
    },
  },
  methods: {
    setConfig(i) {
      const jsonConfig = JSON.parse(i.target.result);
      const columns = this.headers.map((item) => {
        return item.text;
      });
      const array = [];
      array.push(jsonConfig.index);
      array.push(jsonConfig.target);
      array.push(jsonConfig.timeseries);
      jsonConfig.transformations.forEach((item) => {
        if (item.type === RENAME || item.type === IS_IN) {
          array.push(item.column);
        } else {
          item.column.forEach((i) => {
            array.push(i);
          });
        }
      });
      let fault = false;
      array.forEach((item) => {
        if (!columns.includes(item)) {
          fault = true;
        }
      });
      if (!fault) {
        this.index = jsonConfig.index;
        this.target = jsonConfig.target;
        this.isTimeseries = jsonConfig.isTimeseries;
        this.timeseries = jsonConfig.timeseries;
        this.transformations = jsonConfig.transformations;
      } else {
        alert('Config contains wrong columns')
      }
    },
    configFileChanged() {
      if (this.file === undefined) {
        alert('First insert dataset')
      } else {
        const reader = new FileReader();
        reader.onload = (e) => this.setConfig(e);
        reader.readAsText(this.configFile);
      }
    },
    downloadConfig() {
      const fileName = "config.json";
      // Create a blob of the data
      const fileToSave = new Blob(
        [
          JSON.stringify({
            index: this.index,
            target: this.target,
            isTimeseries: this.isTimeseries,
            timeseries: this.timeseries,
            transformations: this.transformations,
          }),
        ],
        {
          type: "application/json",
        }
      );

      // Save the file
      saveAs(fileToSave, fileName);
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
          this.firstLines.push(row.data);
        }.bind(this),
      });
    },
    exampleRequest() {
      const customHeader = {
        headers: {
          "Content-Type": "multipart/form-data",
        },
      };
      const formData = new FormData();
      formData.append("file", this.file);
      console.log(JSON.stringify(this.transformations));
      formData.append(
        "config",
        JSON.stringify({
          index: this.index,
          target: this.target,
          isTimeseries: this.isTimeseries,
          timeseries: this.timeseries,
          transformations: this.transformations,
        })
      );
      const url = API + "example";
      axios
        .post(url, formData, customHeader)
        .then((res) => {
          console.log(res.data);
          const fileUrl =
            API + "/download?filename=" + encodeURIComponent(res.data.dataset);
          console.log(fileUrl);
          axios.get(fileUrl).then((result) => {
            this.exampleFirstLines = []
            this.exampleHeaders = []
            this.exampleResult = ''
            Papa.parse(result.data, {
              header: true,
              step: function (row) {
                if (this.exampleHeaders.length === 0) {
                  row.meta.fields.forEach((element) => {
                    this.exampleHeaders.push({
                      text: element,
                      value: element,
                    });
                  });
                }
                // if (this.exampleFirstLines.length < 5) {
                //   this.exampleFirstLines.push(row.data);
                // }
                this.exampleFirstLines.push(row.data);
              }.bind(this),
            });
            console.log(this.exampleFirstLines);
          });
        })
        .catch((err) => {
          this.exampleFirstLines = []
          this.exampleHeaders = []
          this.exampleResult = err.response.data.message + ' Detail: ' + err.response.data.detail
        });
    },
    removeItem(index) {
      this.transformations.splice(index, 1);
    },
    addOperation() {
      const deepClone = JSON.parse(
        JSON.stringify(this.defaultTransformations[this.newTransformation])
      );
      this.transformations.unshift(deepClone);
      this.newTransformation = "";
      this.addingOperation = false;
    },
    addIsIn(transformation) {
      transformation.push({
        value: "",
      });
    },
    addReplace(transformation) {
      transformation.push({
        old_value: "",
        new_value: "",
      });
    },
  },
};
</script>
