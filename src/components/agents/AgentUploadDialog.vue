<template>
  <v-dialog
    ref="uploadDialog"
    v-model="show"
    max-width="500px"
  >
    <v-card>
      <v-card-title>
        <span class="headline">Upload</span>
      </v-card-title>
      <v-card-text>
        <v-form ref="form">
          <v-container>
            <v-row>
              <v-col cols="12">
                <v-file-input
                  ref="fileInput"
                  v-model="file"
                  accept="*/*"
                  :rules="rules['fileInput']"
                />
                <v-text-field
                  v-model="internalPathToFile"
                  label="path/to/file"
                  :rules="rules['pathToFile']"
                  outlined
                  dense
                  required
                />
              </v-col>
            </v-row>
          </v-container>
        </v-form>
      </v-card-text>
      <v-card-actions>
        <v-spacer />
        <v-btn
          color="blue darken-1"
          text
          @click.stop="show = false"
        >
          Close
        </v-btn>
        <v-btn
          color="blue darken-1"
          text
          :loading="loading"
          @click="submit"
        >
          Save
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
const MAX_BYTES = 1048576;

export default {
  props: {
    value: Boolean,
    language: {
      type: String,
      default: '',
    },
    loading: {
      type: Boolean,
      default: false,
    },
    pathToFile: {
      type: String,
      default: '',
    },
  },
  data() {
    return {
      internalPathToFile: this.pathToFile,
      file: null,
      rules: {
        pathToFile: [
          (v) => !!v || 'PathToFile is required',
        ],
        fileInput: [
          (v) => !!v || 'File required',
          (v) => (!!v && v.size < MAX_BYTES) || `Maximum size of ${Math.floor(MAX_BYTES / 1e6)} MiB.`,
        ],
      },
    };
  },
  computed: {
    show: {
      get() {
        return this.value;
      },
      set(value) {
        this.$emit('input', value);
      },
    },
    fileName() {
      return this.file ? this.file.name : '';
    },
  },
  watch: {
    pathToFile(val) {
      this.internalPathToFile = val;
    },
    value(val) {
      if (val === false) {
        this.$refs.form.reset();
      }
    },
    file(val) {
      if (val) {
        if (this.language === 'python') {
          if (this.internalPathToFile.length === 0) {
            this.internalPathToFile = `/tmp/${val.name}`;
          } else if (this.internalPathToFile.endsWith('/')) {
            this.internalPathToFile += this.fileName;
          } else {
            this.internalPathToFile += `/${this.fileName}`;
          }
        } else if (this.language === 'powershell') {
          if (this.internalPathToFile.length === 0) {
            this.internalPathToFile = `C:\\tmp\\${val.name}`;
          } else if (this.internalPathToFile.endsWith('/') || this.internalPathToFile.endsWith('\\')) {
            this.internalPathToFile += this.fileName;
          } else {
            this.internalPathToFile += `\\${this.fileName}`;
          }
        }
      }
    },
  },
  methods: {
    async submit() {
      if (!this.$refs.form.validate()) { return; }

      this.$emit('submit', { file: await this.toBase64(this.file), pathToFile: this.internalPathToFile });
    },
    // https://stackoverflow.com/a/57272491
    toBase64(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = () => resolve(reader.result.split(',').pop());
        reader.onerror = (error) => reject(error);
      });
    },
  },
};
</script>
