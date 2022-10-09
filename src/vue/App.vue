<script lang="ts">
import Chart from "./Chart.vue";
import { read }  from "xlsx";

let load: any = {};

export default {
    data() {
        return {
            load,
            sort: "none",
            arrows: {
                up: {},
                right: {
                    style: {
                        width: "0px",
                        background: "white",
                        height: "2px",
                    }
                }
            },
            column: {
                names: "",
                values: "",
                namesStyle: {
                    border: "0px solid red"
                },
                valuesStyle: {
                    border: "0px solid red"
                }
            },
            chartBlockWidth: "500",
            height: "600",
            currentFile: "",
            chartList: [[],[]],
            normalizeArray: [],
            listName: "Лист1",
            fileLoad: {
                error: false,
                style: {
                    opacity: 0,
                    cursor: "default",
                    transition: "opacity .5s"
                }
            }
        };
    },
    methods: {
        loadFile: function (event: any): void {
            if (event.target.files[0].type == "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet") {
                this.fileLoad.error = false;
                this.fileLoad.style.opacity = 0;
                this.fileLoad.style.cursor = "default";

                this.load.reader = new FileReader();
                this.load.reader.onload = (e: any) => {
                    this.load.event = e;
                    let workbook = read(e.target.result, {type: 'binary'});
                    let workbookSheets = workbook.Sheets[this.listName];

                    if (this.column.names.length == 0 || this.column.values.length == 0) {}
                    else {
                        this.chartList[1] = this.extractValues(workbookSheets, this.column.values);
                        this.chartList[0] = this.converteDictionaryXlsxInArrays(workbookSheets, this.column.names);

                        this.sortFunction();

                        this.normalizeArray = this.normalizeMembersArray(this.chartList[1], 500);
                    }
                    
                }
                this.load.reader.readAsBinaryString(event.target.files[0]);
            }
            else {
                this.fileLoad.error = true;
                this.fileLoad.style.opacity = 1;
                this.fileLoad.style.cursor = "cursor";
            }
        },
        sortFunction: function() {
            if (this.sort == "none") return;
            else if (this.sort == "increase") {
                for(let i = 0; i < this.chartList[1].length ; i++) {
                    for(let i2 = 0; i2 < this.chartList[1].length - i - 1; i2++) {
                        if (parseInt(this.chartList[1][i2]) > parseInt(this.chartList[1][i2+1])) {
                            let tempValue = this.chartList[1][i2];
                            this.chartList[1][i2] = this.chartList[1][i2+1];
                            this.chartList[1][i2+1] = tempValue;

                            let tempName = this.chartList[0][i2];
                            this.chartList[0][i2] = this.chartList[0][i2+1];
                            this.chartList[0][i2+1] = tempName;
                        }
                    }
                }
            } else {
                for(let i = 0; i < this.chartList[1].length ; i++) {
                    for(let i2 = 0; i2 < this.chartList[1].length - i - 1; i2++) {
                        if (parseInt(this.chartList[1][i2]) < parseInt(this.chartList[1][i2+1])) {
                            let tempValue = this.chartList[1][i2];
                            this.chartList[1][i2] = this.chartList[1][i2+1];
                            this.chartList[1][i2+1] = tempValue;

                            let tempName = this.chartList[0][i2];
                            this.chartList[0][i2] = this.chartList[0][i2+1];
                            this.chartList[0][i2+1] = tempName;
                        }
                    }
                }
            }
        },
        extractValues: function(workbookSheets: any, values: string): any {
            let splitValues: any = values;
            if (values[0] == "[" && values[values.length - 1] == "]") {
                splitValues = values.slice(1, values.length - 1)
            }

            splitValues = splitValues.split(",");
            let arrayValues: any = [];
            let arraySum: number[] = [];

            for(let index = 0; index < splitValues.length; index++) {
                arrayValues[index] = this.converteDictionaryXlsxInArrays(workbookSheets, splitValues[index]);
            }
            
            for(let index = 0; index < arrayValues[0].length; index++) {
                arraySum[index] = 0;
                for(let indexArray = 0; indexArray < arrayValues.length; indexArray++) {
                    arraySum[index] += parseInt(arrayValues[indexArray][index]);
                }    
            }

            return arraySum;
        },
        converteDictionaryXlsxInArrays: function(dict: any, column: string): any {
            let returnArray: string[] = [];

            for(let value in dict) {
                if (value == "!ref" || value == "!margins") continue;
                if (value.slice(0, column.length) == column) {
                    returnArray.push(dict[value].v);
                }
            }
            return returnArray;
        },
        getCountChartList: function(): any {
            return this.chartList[0].length;
        },
        normalizeMembersArray: function(array: string[], breakValue: number): any {
            let convertedArray: number[]= [];
            for(let i = 0; i < array.length; i++) convertedArray[i] = parseInt(array[i]);
            
            let maxValue: number = Math.max(...convertedArray);

            let divide = 1;
            while(maxValue / divide >= breakValue ) divide++;
            for(let i = 0; i < array.length; i++) {
                convertedArray[i] = Math.round(convertedArray[i] / divide);
            }
            return convertedArray;
        },
        loadChange: function() {
            if (this.load.reader != undefined) {
                this.load.reader.onload(load.event);
            }
        }
    },
    components: { Chart }
}
</script>

<template>
    <div class="container">
        <div class="chartContainer">
            <div class="visualGraph">
                <div class="upArrow"></div>
                <div class="rightArrow" :style="arrows.right.style"></div>
            </div>
            <div v-for="index in (getCountChartList() > 0) ? getCountChartList() - 1 : 0" :value="listName">
                <Chart 
                    :name="chartList[0][index]" 
                    :value="chartList[1][index]"
                    :height="normalizeArray[index]"
                    :widthConstant="(parseInt(chartBlockWidth) / chartList[0].length + 1).toString()">
                </Chart>
            </div>
        </div>
        <div class="chartSettings">
            <Label>Excel Graph Creator</Label>
            <div>
                <label for="listName">List name:</label>
                <input type="text" id="listName"
                :style="(listName.length == 0)? {border: '1px solid #c21313'}: {border: '0px solid #c21313'}"
                @change="loadChange()" 
                placeholder="List name" v-model="listName" class="form-control">
            </div>  
            <div>
                <label for="rowNames">Names column:</label>
                <input type="text" id="rowNames" 
                :style="(column.names.length == 0)? {border: '1px solid #c21313'}: {border: '0px solid #c21313'}"
                @change="loadChange()" 
                v-model="column.names" placeholder="A or B or C or.." class="form-control">
            </div>
            <div>
                <label for="rowValues">Values column:</label>
                <input type="text" id="rowValues"
                :style="(column.values.length == 0)? {border: '1px solid #c21313'}: {border: '0px solid #c21313'}"
                @change="loadChange()"
                v-model="column.values" placeholder="[A-ZZZ..,B-ZZZ...,C-ZZZ...]" class="form-control">
            </div>

            <div>
                <label for="sortOn">Sort on:</label>
                <select v-model="sort" @change="loadChange()" name="" id="sortOn" class="form-select" aria-label="Sort on..">
                    <option default value="none">Don't sort</option>
                    <option value="increase">Increase</option>
                    <option value="decrease">Decrease</option>
                </select>
            </div>      
            <div>
                <label for="loadFile">Excel file:</label>
                <input type="file" id="loadFile" v-on:change="loadFile" accept=".xlsx" class="fileXlsx form-control">
                <label :for="(fileLoad.error) ? 'loadFile' : ''" id="btnFileError" :style="fileLoad.style"
                    class="form-control btn btn-danger">
                    File is't excel format!
                </label>
            </div>
        </div>
    </div>
</template>