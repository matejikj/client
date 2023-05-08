<template>
  <body>
    <v-container>
      <v-row>
        <v-col cols="12" md="3">
          <v-file-input label="Select dataset" @change="fileChanged" v-model="file"  truncate-length="15"></v-file-input>
        </v-col>
        <v-col cols="12" md="6">
          <v-file-input label="Select dataset" @change="fileChanged" v-model="file"  truncate-length="15"></v-file-input>
        </v-col>
        <v-col cols="12" md="3">
          <v-file-input label="Load config file" @change="configFileChanged" v-model="configFile"  truncate-length="15"></v-file-input>
        </v-col>

        <!-- <v-col cols="12" md="6">
          <v-select
            v-model="config.content"
            :items="columns"
            multiple
            label="Pick visualisation columns"
            persistent-hint
          >
            <template v-slot:selection="{ item, index }">
              <v-chip v-if="index < 2">
                <span>{{ item }}</span>
              </v-chip>
              <span
                v-if="index === 2"
                class="text-grey text-caption align-self-center"
              >
                (+{{ config.content.length - 2 }} others)
              </span>
            </template>
          </v-select>
        </v-col> -->
        <!-- <v-col cols="12" md="5">
          <v-select
            v-model="config.identityColumn"
            :items="columns"
            label="Pick identity column"
            persistent-hint
          >
          </v-select>
        </v-col> -->
      </v-row>
      <v-row>
        <v-col cols="12" md="5">
          <v-select
            v-model="config.metaColumns"
            :items="columns"
            multiple
            label="Pick excluded columns"
            persistent-hint
          >
            <template v-slot:selection="{ item, index }">
              <v-chip v-if="index < 2">
                <span>{{ item }}</span>
              </v-chip>
              <span
                v-if="index === 2"
                class="text-grey text-caption align-self-center"
              >
                (+{{ config.metaColumns.length - 2 }} others)
              </span>
            </template>
          </v-select>
        </v-col>
        <v-col cols="12" md="5">
          <v-select
            v-model="config.visualisationType"
            :items="visualisationTypes"
            label="visualisation Type"
            persistent-hint
          >
          </v-select>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" md="2">
          <v-text-field
            v-model="config.period"
            label="period"
            required
          ></v-text-field>
        </v-col>
        <v-col cols="12" md=5>
          <v-select
            v-model="config.timestampColumn"
            :items="columns"
            label="Pick your timestamp column"
            persistent-hint
          ></v-select>
        </v-col>
        <v-col cols="12" md=5>
          <v-select
            v-model="config.classificationColumn"
            :items="columns"
            label="Pick your classification column"
            persistent-hint
          ></v-select>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" md="4">
          <v-color-picker
            v-if="!config.isGrayscale"
            hide-canvas
            hide-inputs
            v-model="pickerColor"
          ></v-color-picker>
        </v-col>
        <v-col cols="12" md="2">
          <v-btn v-if="!config.isGrayscale" @click="config.colors.push(pickerColor)">
            add color
          </v-btn>
        </v-col>
        <v-col cols="12" md="6">
          <v-combobox
            v-model="predefinedType"
            :items="predefinedVisualisationTypes"
            label="Choose from predefined visualizations"
            @change="predefinedChanged"
          ></v-combobox>
        </v-col>
      </v-row>
      <div v-if="!config.isGrayscale">
        <v-row v-bind:key="index" v-for="value, index in config.colors">
          <v-select
            v-model="config.columnColors[value]"
            multiple
            outlined
            v-bind:background-color="config.colors[index]"
            :items="columns"
            label="Pick your column"
          >
            <template v-slot:selection="{ item, index }">
              <v-chip v-if="index < 2">
                <span>{{ item }}</span>
              </v-chip>
              <span
                v-if="index === 2"
                class="text-grey text-caption align-self-center"
              >
                (+{{ config.columnColors[value].length - 2 }} others)
              </span>
            </template>
          </v-select>
        </v-row>
      </div>
      <v-row>
        <v-col cols="12" md="3">
          <v-tooltip left>
            <template v-slot:activator="{ on, attrs }">
              <v-text-field
                v-model="config.windowSize"
                label="windows Size"
                required
                v-bind="attrs"
                v-on="on"
              ></v-text-field>
            </template>
            <span>windows Size</span>
          </v-tooltip>
        </v-col>
        <v-col cols="12" md="3">
          <v-tooltip left>
            <template v-slot:activator="{ on, attrs }">
              <v-text-field
                v-model="config.windowShift"
                label="window Shift"
                required
                v-bind="attrs"
                v-on="on"
              ></v-text-field>
            </template>
            <span>window Shift</span>
          </v-tooltip>
        </v-col>

        <v-col cols="12" md="3">
          <v-tooltip left>
            <template v-slot:activator="{ on, attrs }">
              <v-text-field
                v-model="config.featureHeight"
                label="Feature height"
                required
                v-bind="attrs"
                v-on="on"
              ></v-text-field>
            </template>
            <span>Height of feature</span>
          </v-tooltip>
        </v-col>
        <v-col cols="12" md="3">
          <v-tooltip left>
            <template v-slot:activator="{ on, attrs }">
              <v-text-field
                v-model="config.featureWidth"
                label="Feature width"
                required
                v-bind="attrs"
                v-on="on"
              ></v-text-field>
            </template>
            <span>Width of feature</span>
          </v-tooltip>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" md="3">
          <v-tooltip left>
            <template v-slot:activator="{ on, attrs }">
              <v-text-field
                v-model="config.gaussian"
                label="Gaussian"
                required
                v-bind="attrs"
                v-on="on"
              ></v-text-field>
            </template>
            <span>Gaussian transform</span>
          </v-tooltip>
        </v-col>
        <v-col cols="12" md="3">
          <v-tooltip left>
            <template v-slot:activator="{ on, attrs }">
              <v-text-field
                v-model="config.stackedColumns"
                label="Stacked columns"
                required
                v-bind="attrs"
                v-on="on"
              ></v-text-field>
            </template>
            <span>In medical data, it often happens that there are incomparably more of the individual properties detected from the examination than the number of these examinations, at that moment it comes to the fact that the data is visualized as a very long strip. We can solve this by distributing one feature displayed in one long strip into several strips. The value of the feature stacking function then tells us how many lanes to divide the original into.</span>
          </v-tooltip>
        </v-col>

        <v-col cols="12" md="3">
          <v-tooltip left>
            <template v-slot:activator="{ on, attrs }">
              <v-text-field
                v-model="config.size"
                label="Size"
                required
                v-bind="attrs"
                v-on="on"
              ></v-text-field>
            </template>
            <span>Size of each image</span>
          </v-tooltip>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" md="3">
          <v-checkbox
            v-model="config.isNormalize"
            label="Normalize"
          >
          </v-checkbox>
        </v-col>
        <v-col cols="12" md="3">
          <v-checkbox
            v-model="config.isGrayscale"
            label="Grayscale"
          >
          </v-checkbox>
        </v-col>
        <v-col cols="12" md="3">
          <v-checkbox
            v-model="config.isOrdering"
            label="Change ordering"
          ></v-checkbox>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" md="2">
          <v-tooltip left>
            <template v-slot:activator="{ on, attrs }">
              <v-text-field
                v-model="config.examplesCount"
                label="Count of examples"
                required
                v-bind="attrs"
                v-on="on"
              ></v-text-field>
            </template>
            <span>period</span>
          </v-tooltip>
        </v-col>
        <v-col cols="12" md="2">
          <v-btn @click="createRequest(createExampleRequestType)" style="margin-right: 10px">
            Show examples
          </v-btn>
        </v-col>
        <v-col cols="12" md="1">
          <v-progress-circular
            :size="50"
            color="primary"
            indeterminate
            v-if="creatingExampleVisible"
          ></v-progress-circular>
        </v-col>
        <v-col cols="12" md="5">
        </v-col>
        <v-col cols="12" md="2">
          <v-btn @click="downloadConfig">
            Download config
          </v-btn>
        </v-col>
      </v-row>
      <v-row>
        <v-col v-bind:key="index" v-for="value, index in examples" cols="12" md="3">
          <v-img
            max-height="250"
            max-width="250"
            :src="value"
          ></v-img>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" md="1">
          <v-btn @click="createRequest(createVisualisationRequestType)" style="margin-right: 10px">
            Generate
          </v-btn>
        </v-col>
        <v-col cols="12" md="1">
          <v-progress-circular
            :size="50"
            color="primary"
            indeterminate
            v-if="creatingCompleteVisible"
          ></v-progress-circular>
        </v-col>
        <v-col cols="12" md="2">
          <v-btn v-if="downloadVisible" @click="downloadResults">
            Download
          </v-btn>
        </v-col>
        <v-col cols="12" md="1">
          <span v-if="failVisible" v-text="generateMsg"></span>
        </v-col>
      </v-row>
    </v-container>
  </body>
</template>

<script>
import axios from "axios";
import Papa from 'papaparse';
import { saveAs } from 'file-saver';
import { API, visualisationDefinitions, visualisationTypes } from '../../config.js';

export default {
  name: "HomeView",
  data: () => ({
    config: {
      isTimestamp: true,
      windowSize: '4',
      windowShift: '2',
      featureHeight: '1',
      featureWidth: '1',
      content: [],
      timestampColumn: '',
      gaussian: '',
      stackedColumns: '',
      size: '',
      isNormalize: false,
      isGrayscale: false,
      isOrdering: false,
      period: '1',
      examplesCount: '4',
      identityColumn: '',
      classificationColumn: '',
      columnColors: {},
      colors: [],
      metaColumns: [],
      visualisationType: 'stripes',
      divideTrainTestDev: ''
    },
    columns: [],
    examples: [],
    pickerColor: '',
    visualisationTypes: ['stripes', 'block', 'line'],
    createVisualisationRequestType: 0,
    createExampleRequestType: 1,
    generateMsg: '',
    exampleMessage: '',
    file: undefined,
    configFile: undefined,
    file_name: "",
    predefinedType: undefined,
    parsed: false,
    creatingExampleVisible: false,
    creatingCompleteVisible: false,
    downloadVisible: false,
    downloadSrc: '',
    failVisible: false
  }),
  computed: {
    // a computed getter
    predefinedVisualisationTypes () {
      // `this` points to the component instance
      return visualisationTypes
    }
  },
  methods: {
    createNewColorGroup () {
      this.config.colors.push(this.pickerColor)
    },
    downloadConfig () {
      var fileName = 'config.json';
      // Create a blob of the data
      var fileToSave = new Blob([JSON.stringify(this.config)], {
        type: 'application/json'
      });

      // Save the file
      saveAs(fileToSave, fileName);
    },
    setConfig (i) {
      this.config = JSON.parse(i.target.result)
    },
    configFileChanged () {
      const reader = new FileReader();
      reader.onload = e => this.setConfig(e)
      reader.readAsText(this.configFile);
    },
    fileChanged () {
      Papa.parse(this.file, {
        header: true,
        complete: function (results) {
          this.config.content = results.meta.fields;
          this.columns = results.meta.fields;
          this.parsed = true;
        }.bind(this)
      });
    },
    createRequest (type) {
      var url = API
      this.failVisible = false
      const customHeader = {
        headers: {
          "Content-Type": "multipart/form-data"
        }
      };
      var formData = new FormData();
      formData.append("file", this.file);
      formData.append("window_size", this.config.windowSize);
      formData.append("window_shift", this.config.windowShift);
      formData.append("feature_height", this.config.featureHeight);
      formData.append("feature_width", this.config.featureWidth);
      formData.append("columns", this.config.content);
      formData.append("timestamp_column", this.config.timestampColumn);
      formData.append("gaussian", this.config.gaussian);
      formData.append("stacked_columns", this.config.stackedColumns);
      formData.append("size", this.config.size);
      formData.append("is_normalize", this.config.isNormalize);
      formData.append("is_grayscale", this.config.isGrayscale);
      formData.append("is_ordering", this.config.isOrdering);
      formData.append("period", this.config.period);
      formData.append("examples_count", this.config.examplesCount);
      formData.append("identity_column", this.config.identityColumn);
      formData.append("classification_column", this.config.classificationColumn);
      formData.append("column_colors", JSON.stringify(this.config.columnColors));
      formData.append("meta_columns", this.config.metaColumns);
      formData.append("visualisation_type", this.config.visualisationType);
      switch (type) {
        case this.createExampleRequestType:
          this.creatingExampleVisible = true;
          url = url + "/example";
          break;
        case this.createVisualisationRequestType:
          this.creatingCompleteVisible = true;
          url = url + "/upload";
          break;
      }
      axios
        .post(url, formData, customHeader)
        .then((res) => {
          switch (type) {
            case this.createExampleRequestType:
              this.file_name = res.data;
              console.log(res.data)
              this.examples = []
              res.data.ids.forEach(element => {
                this.examples.push(API + "/image?data=" + encodeURIComponent(res.data.src) + "&id=" + encodeURIComponent(element))
              });
              this.creatingExampleVisible = false;
              break;
            case this.createVisualisationRequestType:
              this.downloadSrc = res.data.filename;
              this.downloadVisible = true;
              this.creatingCompleteVisible = false;
              break;
          }
        })
        .catch((err) => {
          console.log(err)
        });
    },
    downloadResults () {
      const url = API + "/download?filename=" + encodeURIComponent(this.downloadSrc)
      location.href = url
    },
    predefinedChanged () {
      this.config = visualisationDefinitions[this.predefinedType]
    }
  }
};
</script>
