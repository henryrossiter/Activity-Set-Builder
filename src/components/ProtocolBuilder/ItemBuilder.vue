<template>
  <v-card>
    <v-card-title
      class="headline grey lighten-2"
      primary-title
    >
      <v-icon left>
        {{ isItemEditable ? 'mdi-pencil' : 'mdi-eye' }}
      </v-icon>
      {{ isItemEditable ? 'Edit Item' : 'View Item' }}
    </v-card-title>
    <v-card-text>
      <v-form
        ref="form"
        lazy-validation
      >
        <v-text-field
          v-model="name"
          label="Item Name"
          :disabled="!isItemEditable"
          required
        />
        <v-text-field
          v-model="description"
          label="Description"
          :disabled="!isItemEditable"
          required
        />
        <v-textarea
          v-model="question"
          label="Question"
          :disabled="!isItemEditable"
          auto-grow
          rows="1"
        />
        <v-select
          v-model="inputType"
          :items="inputTypes"
          label="Input Type"
          :disabled="!isItemEditable"
        />
        <RadioBuilder
          v-if="inputType === 'radio'"
          :initial-item-data="options"
          :is-item-editable="isItemEditable"
          @updateOptions="updateOptions"
        />
        <TextBuilder
          v-if="inputType === 'text'"
          :initial-item-data="options"
          :is-item-editable="isItemEditable"
          @updateOptions="updateOptions"
        />
        <SliderBuilder
          v-if="inputType === 'slider'"
          :initial-item-data="options"
          :is-item-editable="isItemEditable"
          @updateOptions="updateOptions"
        />
        <VideoBuilder
          v-if="inputType === 'video'"
        />
        <PhotoBuilder
          v-if="inputType === 'photo'"
        />
        <TimeRangeBuilder
          v-if="inputType === 'timeRange'"
        />
        <DateBuilder
          v-if="inputType === 'date'"
        />
        <DrawingBuilder
          v-if="inputType === 'drawing'"
        />
        <AudioRecordBuilder
          v-if="inputType === 'audioRecord'"
          :initial-item-data="options"
          @updateOptions="updateOptions"
        />
        <AudioImageRecordBuilder
          v-if="inputType === 'audioImageRecord'"
          :initial-item-data="options"
          @updateOptions="updateOptions"
        />
        <GeolocationBuilder
          v-if="inputType === 'geolocation'"
          @update="updateResponseOptions"
        />
        <AudioStimulusBuilder
          v-if="inputType === 'audioStimulus'"
          :initial-item-input-options="inputOptions"
          :initial-item-media="media"
          :initial-item-data="options"
          :is-item-editable="isItemEditable"
          @updateInputOptions="updateInputOptions"
          @updateMedia="updateMedia"
        />
      </v-form>
    </v-card-text>
    <v-divider />
    <v-card-actions>
      <v-btn
        outlined
        color="primary"
        @click="onDiscardItem"
      >
        {{ isItemEditable ? 'Discard Changes' : 'Close' }}
      </v-btn>
      <v-spacer />
      <v-btn
        color="primary"
        @click="onSaveItem"
      >
        Save Item
      </v-btn>
    </v-card-actions>
  </v-card>
</template>

<script>
import RadioBuilder from './ItemBuilders/RadioBuilder.vue';
import TextBuilder from './ItemBuilders/TextBuilder.vue';
import SliderBuilder from './ItemBuilders/SliderBuilder.vue';
import VideoBuilder from './ItemBuilders/VideoBuilder.vue';
import PhotoBuilder from './ItemBuilders/PhotoBuilder.vue';
import TimeRangeBuilder from './ItemBuilders/TimeRangeBuilder.vue';
import DateBuilder from './ItemBuilders/DateBuilder.vue';
import DrawingBuilder from './ItemBuilders/DrawingBuilder.vue';
import AudioRecordBuilder from './ItemBuilders/AudioRecordBuilder.vue';
import AudioImageRecordBuilder from './ItemBuilders/AudioImageRecordBuilder.vue';
import GeolocationBuilder from './ItemBuilders/GeolocationBuilder.vue';
import AudioStimulusBuilder from './ItemBuilders/AudioStimulusBuilder.vue';

export default {
  components: {
    RadioBuilder,
    TextBuilder,
    SliderBuilder,
    VideoBuilder,
    PhotoBuilder,
    TimeRangeBuilder,
    DateBuilder,
    DrawingBuilder,
    AudioRecordBuilder,
    AudioImageRecordBuilder,
    GeolocationBuilder,
    AudioStimulusBuilder,
  },
  props: {
    initialItemData: {
      type: Object,
      required: true,
    },
    isItemEditable: {
      type: Boolean,
      default: true,
    },
  },
  data: function () {
    return {
      name: this.initialItemData.name || '',
      question: this.initialItemData.question || '',
      description: this.initialItemData.description || '',
      inputType: this.initialItemData.inputType || '',
      multipleChoice: this.initialItemData.multipleChoice || false,
      options: this.initialItemData.options || [],
      responseOptions: this.initialItemData.responseOptions || {},
      inputOptions: this.initialItemData.inputOptions || {},
      media: this.initialItemData.media || {},
      textRules: [
        v => !!v || 'This field is required',
      ],
      inputTypes: ['radio', 'text', 'slider', 'photo', 'video', 'timeRange', 'date', 'drawing', 'audioRecord', 'audioImageRecord', 'geolocation', 'audioStimulus'],
    };
  },
  methods: {
    validate () {
      if (this.$refs.form.validate()) {
        this.snackbar = true
      }
    },
    updateResponseOptions(newResponseOptions) {
      this.responseOptions = newResponseOptions;
    },
    updateInputOptions(newInputOptions) {
      this.inputOptions = newInputOptions;
    },
    updateMedia(newMedia) {
      this.media = newMedia;
    },
    updateOptions(newOptions) {
      this.options = newOptions;
    },
    getSliderChoices() {      
      const n = this.options.numOptions;
      const choices = [];
      var i;
      for (i = 0; i < n; i++) {
        choices.push({
          "schema:name": i.toString(),
          "schema:value": i
        });
      }
      return choices;
    },
    getRadioChoices() {
      const choices = this.options && this.options.options ? this.options.options.map((option, index) => {
          const choiceSchema = {
            "@type": "schema:Boolean",
            "schema:name": option.name,
            "schema:value": index
          };
          if (option.image) {
            choiceSchema['schema:image'] = option.image;
          }
          return choiceSchema;
        }) : [];
      return choices;
    },
    getResponseOptions() {
      if (this.inputType === 'radio') {
        const choices = this.getRadioChoices();
        return {
          "@type": "xsd:anyURI",
          "multipleChoice": this.multipleChoice,
          "schema:minValue": 0,
          "schema:maxValue": choices.length - 1,
          "choices": choices,
        };
      }
      if (this.inputType === 'text') {
        return this.options;
      }
      if (this.inputType === 'slider') {
        const choices = this.getSliderChoices();
        return {
          "@type": "xsd:integer",
          "schema:minValue": this.options.minValue,
          "schema:maxValue": this.options.maxValue,
          "choices": choices,
        };
      }
      if (this.inputType === 'date') {
        return {
          "type": "xsd:date",
          "requiredValue": true,
          "schema:maxValue": "new Date()"
        };
      }
      if (this.inputType === 'audioRecord' || this.inputType === 'audioImageRecord') {
        return this.options;
      }
      else {
        return {};
      }
    },
    getInputOptions() {
      if (this.inputType === 'audioStimulus') {
        return this.inputOptions;
      }
      return {};
    },
    getMedia() {
      if (this.inputType === 'audioStimulus') {
        return this.media;
      }
      return {};
    },
    getCompressedSchema() {
      const responseOptions = this.getResponseOptions();
      const inputOptions = this.getInputOptions();
      const media = this.getMedia();
      const schema = {
        "@context": [
          "https://raw.githubusercontent.com/ReproNim/reproschema/master/contexts/generic",
          "https://raw.githubusercontent.com/YOUR-ACTIVITY-CONTEXT-FILE"
        ],
        "@type": "reproschema:Field",
        "@id": this.name,
        "skos:prefLabel": this.name,
        "skos:altLabel": this.name,
        "schema:description": this.description,
        "schema:schemaVersion": "0.0.1",
        "schema:version": "0.0.1",
        "question": this.question,
        "ui": {
          "inputType": this.inputType
        },
      };
      if (Object.keys(responseOptions).length !== 0) {
        schema["responseOptions"] = responseOptions;
      }
      if (this.inputType === 'audioStimulus') {
        schema["inputOptions"] = inputOptions;
      }
      if (this.inputType === 'audioStimulus') {
        schema["media"] = media;
      }
      return schema;
    },
    onSaveItem() {
      if (this.isItemEditable) {
        const schema = this.getCompressedSchema();
        const itemObj = {
          'name': this.name,
          'question': this.question,
          'description': this.description,
          'inputType': this.inputType,
          'options': this.options,
          'isItemEditable': this.isItemEditable,
          'schema': schema,
        };

        if (this.inputType === 'radio') {
          itemObj.responseOptions = this.responseOptions;
        } else if (this.inputType === 'text') {
          itemObj.responseOptions = this.responseOptions;
        } else if (this.inputType === 'slider') {
          itemObj.responseOptions = this.responseOptions;
        } else if (this.inputType === 'audioRecord') {
          itemObj.responseOptions = this.responseOptions;
        } else if (this.inputType === 'audioImageRecord') {
          itemObj.responseOptions = this.responseOptions;
        } else if (this.inputType === 'geolocation') {
          itemObj.responseOptions = this.responseOptions;
        } else if (this.inputType === 'audioStimulus') {
          itemObj.inputOptions = this.inputOptions;
          itemObj.media = this.media;
        }
        this.$emit('closeItemModal', itemObj);
      } else {
        this.$emit('closeItemModal', null);
      } 
      
    },
    onDiscardItem() {
      this.$emit('closeItemModal', null)
    }
  },
};
</script>

