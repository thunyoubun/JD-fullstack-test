<template>
  <v-container>
    <v-data-table-virtual
      :headers="headers"
      :items="items"
      height="400"
      item-value="part_no"
    >
      <template v-slot:top>
        <v-toolbar flat>
          <v-toolbar-title>Part Change over Matrix</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-text-field
            v-model="newPartNo"
            placeholder="Enter Part No"
            prepend-inner-icon="mdi-magnify"
            variant="outlined"
            hide-details
            single-line
          ></v-text-field>
          <v-btn color="primary" @click="addNewPart">Add New Part No</v-btn>
          <v-btn color="primary" @click="exportExcel">Dowload</v-btn>
          <v-btn color="primary" @click="triggerFileInput">Upload</v-btn>
          <input
            type="file"
            ref="fileInput"
            @change="handleFileUpload"
            style="display: none"
          />
          <!-- <p v-if="message">{{ message }}</p> -->
        </v-toolbar>
      </template>

      <template v-slot:item="{ item }">
        <tr>
          <td
            v-for="header in headers"
            :key="header.key"
            :style="getCellStyle(item[header.key])"
          >
            {{ item[header.key] }}
          </td>
        </tr>
      </template>
    </v-data-table-virtual>
  </v-container>
</template>

<script>
import axios from "axios";
export default {
  data() {
    return {
      headers: [],
      items: [],
      file: null,
      message: "upload",
      newPart: "",
    };
  },
  async mounted() {
    await this.getItems();
  },

  methods: {
    getCellStyle(value) {
      return value === ""
        ? { backgroundColor: "gray", color: "black" }
        : { backgroundColor: "white", color: "black" };
    },

    async getItems() {
      try {
        const response = await axios.get("http://127.0.0.1:8000/read/");
        if (response.status == 200) {
          this.headers = response.data.columns.map((column) => {
            return { title: column, key: column };
          });

          this.items = response.data.rows;
          console.log("Items:", this.items);
        }
      } catch (error) {
        console.error("Error:", error);
      }
    },

    triggerFileInput() {
      this.$refs.fileInput.click();
    },

    handleFileUpload(event) {
      this.file = event.target.files[0];
    },

    async uploadFile() {
      if (!this.file) {
        this.message = "Please select a file to upload.";
        return;
      }
      let formData = new FormData();
      formData.append("file", this.file);
      this.message = this.file.name;
      console.log("Uploading file:", this.file);
      try {
        const response = await axios.post(
          "http://127.0.0.1:8000/upload/",
          formData,
          {
            headers: {
              "Content-Type": "multipart/form-data",
            },
          }
        );

        if (response.status == 200) {
          this.getItems();
        }
      } catch (error) {
        console.error("Error uploading file:", error);
      }
    },
    async exportExcel() {
      try {
        const response = await axios.get("http://127.0.0.1:8000/export/", {
          responseType: "blob",
        });

        const url = window.URL.createObjectURL(new Blob([response.data]));
        const link = document.createElement("a");
        link.href = url;
        link.setAttribute("download", "part_file.xlsx");
        document.body.appendChild(link);
        link.click();
      } catch (error) {
        console.error("Download error:", error);
      }
    },

    async updateExcel() {
      try {
        const data = {
          columns: this.headers.map((header) => header.title),
          rows: this.items.map((item) => Object.values(item)),
        };

        console.log("Data:", data);

        const response = await axios.post(
          "http://127.0.0.1:8000/update/",
          data
        );

        if (response.status == 200) {
          this.getItems();
        }
      } catch (error) {
        console.error("Error:", error);
      }
    },
    addNewPart() {
      // ตรวจสอบว่า partNo ไม่ว่าง
      if (this.newPartNo.trim()) {
        // add new part to the last column
        this.headers.push({ title: this.newPartNo, key: this.newPartNo });

        // add "0" to new part
        this.items.forEach((item) => {
          item[this.newPartNo] = "0";
        });

        // add new part to each row
        this.items.push({
          Part_No: this.newPartNo,
          ...Object.fromEntries(
            this.headers.slice(1).map((header) => [header.title, "0"])
          ),
        });

        // last row of last new part must be "" not "0"
        this.items[this.items.length - 1][this.newPartNo] = "";

        // update excel file
        this.updateExcel();

        this.newPartNo = "";
      } else {
        alert("Please enter a valid Part No");
      }
    },
  },
  watch: {
    file() {
      this.uploadFile();
    },
  },
};
</script>
