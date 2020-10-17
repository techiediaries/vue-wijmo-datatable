<template>

  <v-card class="p-64">

    <v-data-table ref="myTable" 
      :headers="headers"
      :items="products"
      :items-per-page="10"
      class="elevation-1">
    </v-data-table>
      <v-btn @click="exportExcelFlexgrid">Export as Excel</v-btn>

      <div id="theGrid"></div>

  </v-card>


</template>
<style>
@import '../node_modules/@grapecity/wijmo.styles/wijmo.css';

</style>

<script>
import axios from "axios";

import '@grapecity/wijmo.vue2.grid';
import { FlexGrid } from '@grapecity/wijmo.grid';

import * as wjGridXlsx from '@grapecity/wijmo.grid.xlsx';


export default {
  name: 'App',

  components: {
  },
  data: () => ({
    products: [],
    loading: true,
    theGrid: null,
    headers: [
      { text: "Product Name", value: "name" },
      { text: "Type", value: "type" },
      { text: "SKU", value: "sku" },
      { text: "Manufacturer", value: "manufacturer" },
      { text: "Model", value: "model" },
      { text: "UPC", value: "upc" },
      { text: "Price", value: "price" }
    ]

  }),
  methods: {
    fetchData() {
      this.loading = true;
      axios.get("http://localhost:8000/products").then((response) => {
        this.loading = false;
        //console.log(response);
        this.products = response.data;
        //this.createWijmoTable();
      });
    },
    createWijmoTable() {
      this.theGrid = new FlexGrid('#theGrid', {
        stickyHeaders: true,
        autoGenerateColumns: true,
        alternatingRowStep: true,
        itemsSource: this.products
      });
    },
    resizableColumns() {

      let table = document.querySelector('table');
      var initialPageX;
      var currentColumn;
      var nextColumn;
      var currentColumnWidth;
      var nextColumnWidth;

      if (table) {
        const columnHeaders = document.querySelectorAll('th');

        for (var i = 0; i < columnHeaders.length; i++) {
          var bar = createBar(columnHeaders[i].offsetHeight + 'px');
          columnHeaders[i].appendChild(bar);
          columnHeaders[i].style.position = 'relative';
        }
      }

      function createBar(height) {
        var div = document.createElement('div');
        div.style.top = 0;
        div.style.right = 0;
        div.style.width = '9px';
        div.style.borderRight = '5px solid grey';

        div.style.position = 'absolute';
        div.style.cursor = 'col-resize';
        div.style.userSelect = 'none';
        div.style.height = height;
        div.style.zIndex = 1;


        div.addEventListener('mousedown', function (e) {
          currentColumn = e.target.parentElement;
          nextColumn = currentColumn.nextElementSibling;
          initialPageX = e.pageX;
          currentColumnWidth = currentColumn.offsetWidth;
          if (nextColumn)
            nextColumnWidth = nextColumn.offsetWidth;
        });

        document.addEventListener('mouseup', function () {
          currentColumn = undefined;
          nextColumn = undefined;
          initialPageX = undefined;
          nextColumnWidth = undefined;
          currentColumnWidth = undefined
        });

        document.addEventListener('mousemove', function (event) {
          if (currentColumn) {
            var resizeAmount = event.pageX - initialPageX;
            currentColumn.style.width = (currentColumnWidth + resizeAmount) + 'px';


            if (nextColumn)
              nextColumn.style.width = (nextColumnWidth - resizeAmount) + 'px';

          }
        });
        return div;
      }

    },
    exportExcelFlexgrid() {
      var book = wjGridXlsx.FlexGridXlsxConverter.save(this.theGrid, {
        includeColumnHeaders: true,
        includeRowHeaders: true
      });
      //name the sheet
      book.sheets[0].name = 'FlexGrid Data';
      // save the book
      book.saveAsync('FlexGrid-Export.xlsx');
    }
  },

  mounted() {
    this.fetchData();
    this.resizableColumns();

  }
};

</script>
