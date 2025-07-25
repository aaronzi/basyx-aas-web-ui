<template>
    <v-container fluid class="pa-0">
        <!-- Header -->
        <v-card v-if="!hideSettings" class="mb-4">
            <v-card-title>
                <div class="text-subtitle-1">
                    {{ nameToDisplay(submodelElementData, 'en', 'Time Series Data') }}
                </div>
            </v-card-title>
            <v-card-text v-if="descriptionToDisplay(submodelElementData)" class="pt-0">
                {{ descriptionToDisplay(submodelElementData) }}
            </v-card-text>
        </v-card>
        <!-- Data Preview Config -->
        <v-card v-if="!hideSettings || editDialog" class="mb-4">
            <!-- Title -->
            <v-list v-if="!hideSettings || editDialog" nav class="py-0">
                <v-list-item class="pb-0">
                    <template #title>
                        <div class="text-subtitle-2">{{ 'Preview Configuration: ' }}</div>
                    </template>
                </v-list-item>
            </v-list>
            <!-- Preview Config -->
            <v-card-text class="pt-1">
                <!-- Segment Selection -->
                <v-select
                    v-model="selectedSegment"
                    variant="outlined"
                    density="compact"
                    clearable
                    label="Segment"
                    :items="segments"
                    item-title="idShort"
                    item-value="idShort"
                    return-object
                    @update:model-value="emitSegment"></v-select>
                <!-- Record Selection -->
                <v-row>
                    <v-col cols="12" md="6">
                        <v-select
                            v-model="timeVariable"
                            variant="outlined"
                            density="compact"
                            clearable
                            label="time-value"
                            :items="records"
                            item-title="idShort"
                            item-value="idShort"
                            return-object
                            @update:model-value="emitTimeValue"></v-select>
                    </v-col>
                    <v-col cols="12" md="6">
                        <v-select
                            v-model="yVariables"
                            variant="outlined"
                            density="compact"
                            clearable
                            label="y-value(s)"
                            :items="records"
                            item-title="idShort"
                            item-value="idShort"
                            return-object
                            multiple
                            @update:model-value="emitYValue"></v-select>
                    </v-col>
                </v-row>
                <!-- API Token -->
                <v-text-field
                    v-if="segmentType == 'LinkedSegment' && showTokenInput"
                    v-model="apiToken"
                    variant="outlined"
                    density="compact"
                    clearable
                    label="API Token"
                    hide-details></v-text-field>
            </v-card-text>
            <v-divider></v-divider>
            <v-list nav class="pr-2 pt-0">
                <v-list-item>
                    <template #append>
                        <v-btn
                            v-if="segmentType == 'LinkedSegment'"
                            size="small"
                            class="text-buttonText"
                            color="primary"
                            @click="fetchLinkedData()"
                            >Fetch Data</v-btn
                        >
                        <v-btn
                            v-if="segmentType == 'InternalSegment'"
                            size="small"
                            class="text-buttonText"
                            color="primary"
                            @click="fetchInternalData()"
                            >Fetch Data</v-btn
                        >
                        <v-btn
                            v-if="segmentType == 'ExternalSegment'"
                            size="small"
                            class="text-buttonText"
                            color="primary"
                            @click="fetchExternalData()"
                            >Fetch Data</v-btn
                        >
                    </template>
                </v-list-item>
            </v-list>
        </v-card>
        <!-- Data Preview Chart -->
        <v-card :flat="hideSettings">
            <!-- Title -->
            <v-list v-if="!hideSettings || editDialog" nav class="py-0">
                <v-list-item>
                    <template #title>
                        <div class="text-subtitle-2">{{ 'Preview Chart: ' }}</div>
                    </template>
                    <!-- TODO: Decide if we want to keep the dashboard integration -->
                    <!-- <template #append>
                        <v-btn
                            v-if="selectedChartType && !hideSettings"
                            color="primary"
                            class="text-buttonText"
                            size="small"
                            variant="elevated"
                            append-icon="mdi-plus"
                            @click="createObject()"
                            >Dashboard</v-btn
                        >
                    </template> -->
                </v-list-item>
            </v-list>
            <v-card-text class="pt-1">
                <!-- Chart Type Selection -->
                <v-select
                    v-if="!hideSettings || editDialog"
                    v-model="selectedChartType"
                    variant="outlined"
                    density="compact"
                    clearable
                    label="Chart Type"
                    :items="chartTypes"
                    item-title="name"
                    item-value="name"
                    return-object
                    @update:model-value="clearChartOptions"></v-select>
                <!-- Chart Preview -->
                <LineChart
                    v-if="selectedChartType && selectedChartType.id == 1"
                    :chart-data="timeSeriesValues"
                    :time-variable="timeVariable"
                    :y-variables="yVariables"
                    :chart-options-external="chartOptions"
                    :edit-dialog="editDialog"
                    @chart-options="getChartOptions"></LineChart>
                <AreaChart
                    v-if="selectedChartType && selectedChartType.id == 2"
                    :chart-data="timeSeriesValues"
                    :time-variable="timeVariable"
                    :y-variables="yVariables"
                    :chart-options-external="chartOptions"
                    :edit-dialog="editDialog"
                    @chart-options="getChartOptions"></AreaChart>
                <ScatterChart
                    v-if="selectedChartType && selectedChartType.id == 3"
                    :chart-data="timeSeriesValues"
                    :time-variable="timeVariable"
                    :y-variables="yVariables"
                    :chart-options-external="chartOptions"
                    :edit-dialog="editDialog"
                    @chart-options="getChartOptions"></ScatterChart>
                <Histogram
                    v-if="selectedChartType && selectedChartType.id == 4"
                    :chart-data="timeSeriesValues"
                    :time-variable="timeVariable"
                    :y-variables="yVariables"
                    :chart-options-external="chartOptions"
                    :edit-dialog="editDialog"
                    @chart-options="getChartOptions"></Histogram>
                <Gauge
                    v-if="selectedChartType && selectedChartType.id == 5"
                    :chart-data="timeSeriesValues"
                    :time-variable="timeVariable"
                    :y-variables="yVariables"
                    :chart-options-external="chartOptions"
                    :edit-dialog="editDialog"
                    @chart-options="getChartOptions"></Gauge>
                <DisplayField
                    v-if="selectedChartType && selectedChartType.id == 6"
                    :chart-data="timeSeriesValues"
                    :y-variables="yVariables"></DisplayField>
            </v-card-text>
        </v-card>
    </v-container>
</template>

<script lang="ts" setup>
    import { computed, onMounted, ref, watch } from 'vue';
    import { useRoute } from 'vue-router';
    import { useConceptDescriptionHandling } from '@/composables/AAS/ConceptDescriptionHandling';
    import { useReferableUtils } from '@/composables/AAS/ReferableUtils';
    // import { useDashboardHandling } from '@/composables/DashboardHandling';
    import { useRequestHandling } from '@/composables/RequestHandling';
    import { useEnvStore } from '@/store/EnvironmentStore';
    import { useNavigationStore } from '@/store/NavigationStore';

    defineOptions({
        name: 'TimeSeriesData',
        semanticId: 'https://admin-shell.io/idta/TimeSeries/1/1',
    });

    const props = withDefaults(
        defineProps<{
            submodelElementData: any; // TODO: Convert SubmodelElementData to SubmodelElement Type of AAS Core Works
            configData?: any;
            editDialog?: boolean;
            loadTrigger?: any;
        }>(),
        {
            configData: null,
            editDialog: false,
            loadTrigger: null,
        }
    );

    const emit = defineEmits<{
        (event: 'timeVal', value: any): void;
        (event: 'YVal', value: any): void;
        (event: 'segment', value: any): void;
        (event: 'chartOptions', value: any): void;
        (event: 'newOptions', value: any): void;
    }>();

    // Stores
    const envStore = useEnvStore();
    const navigationStore = useNavigationStore();

    // Composables
    const { fetchCds } = useConceptDescriptionHandling();
    const { checkIdShort, descriptionToDisplay, nameToDisplay } = useReferableUtils();
    const { getRequest, postRequest } = useRequestHandling();
    // const { dashboardAdd } = useDashboardHandling();

    // vue-router
    const route = useRoute();

    const timeSeriesData = ref({} as any); // Object to store the data of the time series smt
    const segments = ref([] as Array<any>); // Array to store the segments of the time series smt
    const selectedSegment = ref(null as any); // Object to store the selected segment of the time series smt
    const records = ref([] as Array<any>); // Array to store the records of the time series smt
    const timeVariable = ref(null as any); // Object to store the selected time variable of the time series smt
    const yVariables = ref([] as Array<any>); // Array to store the selected y variables of the time series smt
    const yVariableTemplate = ref('{{y-value}}'); // String that is used to inject y-variable in linkedSeg Query
    const apiToken = ref(''); // API Token for the Time Series Database
    const showTokenInput = ref(true); // Boolean to show the API Token Input
    const timeSeriesValues = ref([] as Array<any>); // Array to store the values of the time series smt
    const chartTypes = ref<Array<{ id: number; name: string }>>([
        { id: 1, name: 'Line Chart' },
        { id: 2, name: 'Area Chart' },
        { id: 3, name: 'Scatter Chart' },
        { id: 4, name: 'Histogram' },
        { id: 5, name: 'Gauge' },
        { id: 6, name: 'Display Field' },
    ]); // Array to store the chart types
    const selectedChartType = ref(null as any); // Object to store the selected chart type
    const chartOptions = ref({} as any); // Object to store the chart options

    // Computed properties
    const segmentType = computed(() => {
        if (!selectedSegment.value) {
            return null;
        }

        // create an array of semanticIds from the selected Segment (selectedSegment.value.semanticId.keys)
        let semanticIds = selectedSegment.value.semanticId.keys.map((semanticId: any) => semanticId.value);

        // check if the semanticIds contain the semanticId for InternalSegment
        if (semanticIds.includes('https://admin-shell.io/idta/TimeSeries/Segments/InternalSegment/1/1')) {
            return 'InternalSegment';
        }

        // check if the semanticIds contain the semanticId for LinkedSegment
        if (semanticIds.includes('https://admin-shell.io/idta/TimeSeries/Segments/LinkedSegment/1/1')) {
            return 'LinkedSegment';
        }

        // check if the semanticIds contain the semanticId for ExternalSegment
        if (semanticIds.includes('https://admin-shell.io/idta/TimeSeries/Segments/ExternalSegment/1/1')) {
            return 'ExternalSegment';
        }

        // return null if no Segment Type was found
        return null;
    });

    const hideSettings = computed(() => {
        return route.name === 'DashboardGroup';
    });

    watch(
        () => props.loadTrigger,
        () => {
            initComponent();
        },
        { immediate: true }
    );

    onMounted(() => {
        initComponent();
    });

    function initComponent(): void {
        initializeTimeSeriesData();
        initDashboardTSD();
        const influxDBToken = envStore.getEnvInfluxdbToken;
        if (influxDBToken && influxDBToken !== '') {
            apiToken.value = influxDBToken;
            showTokenInput.value = false;
        }
    }

    function initializeTimeSeriesData(): void {
        if (Object.keys(props.submodelElementData).length === 0) {
            timeSeriesData.value = {};
            return;
        }

        timeSeriesData.value = { ...props.submodelElementData }; // create local copy of the TimeSeriesData
        // get the collection for segments
        const segmentsSMC = timeSeriesData.value.submodelElements.find((smc: any) => checkIdShort(smc, 'Segments'));
        // create an array of segments
        segments.value = segmentsSMC.value;
        // console.log('Segments: ', segments.value);
        // get the collection for metadata
        const metadataSMC = timeSeriesData.value.submodelElements.find((smc: any) => checkIdShort(smc, 'Metadata'));
        // get the collection for records
        const recordsSMC = metadataSMC.value.find((smc: any) => checkIdShort(smc, 'Record'));
        // create an array of records
        const recordsArray = recordsSMC.value;
        // request the concept descriptions for the records (if they have semanticIds)
        // Create a list of promises
        let promises = recordsArray.map((record: any) => {
            return fetchCds(record).then((response: any) => {
                // add ConceptDescription to the record
                if (response) {
                    record.conceptDescriptions = response;
                }
                return record;
            });
        });

        // Wait for all promises to resolve and then update the reactive records
        Promise.all(promises).then((updatedRecords) => {
            // console.log('Updated Records: ', updatedRecords);
            records.value = updatedRecords;
        });
    }

    function initDashboardTSD(): void {
        if (!hideSettings.value) return;
        selectedChartType.value = props.configData.configObject.chartType;
        selectedSegment.value = props.configData.configObject.segment;
        timeVariable.value = props.configData.configObject.timeVal;
        // console.log(timeVariable.value);
        yVariables.value = props.configData.configObject.yvals;
        // add the chart type specific options to the chartOptions
        chartOptions.value = props.configData.configObject.chartOptions;
        // add the API Token to the API Token field if it is available
        if (props.configData.configObject.apiToken && props.configData.configObject.apiToken !== '') {
            apiToken.value = props.configData.configObject.apiToken;
            showTokenInput.value = false;
        }
        if (checkIdShort(selectedSegment.value, 'InternalSegment')) fetchInternalData();
        if (checkIdShort(selectedSegment.value, 'ExternalSegment')) fetchExternalData();
        if (checkIdShort(selectedSegment.value, 'LinkedSegment')) fetchLinkedData();
    }

    function fetchInternalData(): void {
        if (!selectedSegment.value || !timeVariable.value || yVariables.value.length === 0) {
            return;
        }

        getRecordValues();
    }

    function getRecordValues(): void {
        // console.log('Selected Segment: ', selectedSegment.value);
        // get the records submodel element collection
        const recordsSMC = selectedSegment.value.value.find((smc: any) => checkIdShort(smc, 'Records'));
        // save the records in an array
        const records = recordsSMC.value;
        // console.log('Records: ', records, ' Time Variable: ', timeVariable.value, ' Y Variables: ', yVariables.value);
        let transformedArray = yVariables.value
            .filter(
                (yVar) =>
                    // Check if yVarEntry exists in all records
                    records.every((item: any) => item.value.some((entry: any) => checkIdShort(entry, yVar.idShort))) ||
                    // display an alert if the yVariable is not available in the records (specify the yVariable name)
                    navigationStore.dispatchSnackbar({
                        status: true,
                        timeout: 4000,
                        color: 'warning',
                        btnColor: 'buttonText',
                        text: 'y-value ' + yVar.idShort + ' not available in InternalSegment Records!',
                    })
            )
            .map((yVar) => {
                // For each yVariable, go through each item in the original array
                return records.map((item: any) => {
                    // Extract the time value
                    const timeEntry = item.value.find((entry: any) => checkIdShort(entry, timeVariable.value.idShort));
                    // display an alert if the timeVariable is not available the Records
                    if (!timeEntry) {
                        navigationStore.dispatchSnackbar({
                            status: true,
                            timeout: 4000,
                            color: 'warning',
                            btnColor: 'buttonText',
                            text:
                                'time-value ' +
                                timeVariable.value.idShort +
                                ' not available in InternalSegment Records!',
                        });
                    }
                    const time = timeEntry ? timeEntry.value : null;

                    // Extract the yVariable value
                    const yVarEntry = item.value.find((entry: any) => checkIdShort(entry, yVar.idShort));
                    const yVarValue = yVarEntry ? yVarEntry.value : null;

                    // Return an object with time and the yVariable value
                    return { time, value: yVarValue };
                });
            });

        timeSeriesValues.value = transformedArray;
    }

    function fetchExternalData(): void {
        if (!selectedSegment.value || !timeVariable.value || yVariables.value.length === 0) {
            return;
        }

        getFileData();
    }

    function getFileData(): void {
        // console.log('Selected Segment: ', selectedSegment.value);
        // get the Data File/Blob submodel element
        const dataFile = selectedSegment.value.value.find((smc: any) => checkIdShort(smc, 'Data'));
        // determine the path to the file
        let path = dataFile.value;
        if (path.startsWith('/')) {
            path =
                props.submodelElementData.path +
                '/submodel-elements/Segments.' +
                selectedSegment.value.idShort +
                '.Data/attachment';
        }
        // console.log('Path: ', path);
        // get the file contents
        let context = 'retrieving File Contents';
        let disableMessage = true;
        getRequest(path, context, disableMessage).then((response: any) => {
            if (response.success) {
                // console.log('File Contents: ', response.data);
                convertPlainCSVtoArray(response.data);
            }
        });
    }

    function convertPlainCSVtoArray(csvData: any): void {
        const { headers, data } = parseCSV(csvData);
        const timeIndex = headers.indexOf(timeVariable.value.idShort);
        // handle the case where timeIndex is -1
        if (timeIndex === -1) {
            navigationStore.dispatchSnackbar({
                status: true,
                timeout: 4000,
                color: 'warning',
                btnColor: 'buttonText',
                text: 'time-value ' + timeVariable.value.idShort + ' not available in ExternalSegment Data!',
            });
            return;
        }
        let yIndexes = yVariables.value.map((yVar) => headers.indexOf(yVar.idShort));
        // display an alert if the yVariable is not available in the records (specify the yVariable name)
        yIndexes.forEach((yIndex, index) => {
            if (yIndex === -1) {
                navigationStore.dispatchSnackbar({
                    status: true,
                    timeout: 4000,
                    color: 'warning',
                    btnColor: 'buttonText',
                    text: 'y-value ' + yVariables.value[index].idShort + ' not available in ExternalSegment Data!',
                });
            }
        });
        // handle the case where yIndexes contains -1 (remove only the -1 values)
        yIndexes = yIndexes.filter((index) => index !== -1);
        const datasets = yIndexes.map((yIndex) =>
            data.map((row: any) => ({
                time: row[timeIndex],
                value: Number(row[yIndex]),
            }))
        );
        // console.log('Datasets: ', datasets);
        timeSeriesValues.value = datasets;
    }

    function parseCSV(csvString: string): { headers: Array<string>; data: Array<Array<string>> } {
        // Splitting by a regular expression to handle both \n and \r\n
        const lines = csvString.split(/\r?\n/);
        const headers = lines[0].split(',').map((header) => header.trim()); // Trimming to remove any trailing \r
        // Filter out empty lines and then split each line into columns
        const data = lines
            .slice(1)
            .filter((line) => line)
            .map((line) => line.split(','));
        // console.log('Headers: ', headers, ' Data: ', data);
        return { headers, data };
    }

    function fetchLinkedData(): void {
        // check if a segment is selected
        if (!selectedSegment.value || Object.keys(selectedSegment.value).length === 0) {
            console.warn('No Segment selected');
            return;
        }
        // get the Endpoint from the selected Segment
        const endpoint = selectedSegment.value.value.find((smc: any) => checkIdShort(smc, 'Endpoint')).value;
        // get the query from the selected Segment
        let query = selectedSegment.value.value.find((smc: any) => checkIdShort(smc, 'Query')).value;
        if (yVariables.value.length > 0) query = query.replace(yVariableTemplate.value, yVariables.value[0].idShort);

        // console.log('Endpoint: ', endpoint, ' Query: ', query);
        // construct the headers for the request
        let requestHeaders = new Headers();
        requestHeaders.append('Authorization', 'Token ' + apiToken.value);
        requestHeaders.append('Accept', 'application/csv');
        requestHeaders.append('Content-Type', 'application/vnd.flux');
        // construct the request
        const path = endpoint;
        const content = query;
        const headers = requestHeaders;
        const context = 'fetching data from Time Series Database';
        const disableMessage = false;
        // send the request
        postRequest(path, content, headers, context, disableMessage, true).then((response: any) => {
            if (response.success) {
                convertInfluxCSVtoArray(response.data);
            }
        });
    }

    function convertInfluxCSVtoArray(csvData: any): void {
        const lines = csvData.trim().split('\n');
        const datasets = {} as any;
        let currentDataset = [] as Array<any>;
        let currentTable = null as any;
        let headerLine = '';

        lines.forEach((line: any) => {
            const columns = line.split(',');

            // Skip the header line (because it's not including data)
            if (columns[1] === 'result') {
                headerLine = line;
                return;
            }

            const table = columns[2];
            if (currentTable === null) {
                // this handles the first line after the header
                currentTable = table;
                currentDataset.push(line);
            } else if (table !== currentTable) {
                // this handles the first line of a new table
                const topic = extractTopic(currentDataset[0]);
                datasets[topic] = processDataset(headerLine, currentDataset);
                currentDataset = [line];
                currentTable = table;
            } else {
                // this handles all other lines
                currentDataset.push(line);
            }
        });

        if (currentDataset.length > 0) {
            // this handles the last dataset
            const topic = extractTopic(currentDataset[0]);
            datasets[topic] = processDataset(headerLine, currentDataset);
        }

        // console.log('Datasets: ', datasets);

        // remove the keys from the datasets based on the yVariables
        const datasetsKeys = Object.keys(datasets);
        const datasetsFiltered = datasetsKeys.filter((key) =>
            yVariables.value.some((yVar) => key.includes(yVar.idShort))
        );

        // Find yVariables that are not in the datasets
        const missingYVars = yVariables.value.filter(
            (yVar) => !datasetsFiltered.some((key) => key.includes(yVar.idShort))
        );

        // If there are any missing yVariables, display a warning snackbar
        if (missingYVars.length > 0) {
            const missingYVarNames = missingYVars.map((yVar) => yVar.idShort).join(', ');
            navigationStore.dispatchSnackbar({
                status: true,
                timeout: 4000,
                color: 'warning',
                btnColor: 'buttonText',
                text: 'y-values "' + missingYVarNames + '" not available in LinkedSegment Data!',
            });
        }

        // Order the datasets based on the yVariables
        const newDatasets = yVariables.value
            .map((yVar) => datasetsFiltered.find((key) => key.includes(yVar.idShort)))
            .filter((key) => key !== undefined)
            .map((key: any) => datasets[key]);

        // console.log('Filtered and Ordered Datasets: ', newDatasets);
        timeSeriesValues.value = newDatasets;
    }

    function extractTopic(headerLine: string): string {
        // Implement this method to extract the topic from the header line
        // This is a placeholder implementation
        const columns = headerLine.split(',');
        return columns[columns.length - 1];
    }

    function processDataset(headerLine: string, datasetLines: any): Array<{ time: string; value: number }> {
        // console.log('Dataset Lines: ', datasetLines, ' Header Line: ', headerLine)
        const headers = headerLine.split(',');
        const valueIndex = headers.indexOf('_value');
        const timeIndex = headers.indexOf('_time');

        return datasetLines.slice(1).map((line: any) => {
            const columns = line.split(',');
            return {
                time: columns[timeIndex],
                value: parseFloat(columns[valueIndex]),
            };
        });
    }

    // function createObject(): void {
    //     let dashboardElement = {} as any;
    //     dashboardElement.title = props.submodelElementData.idShort;
    //     dashboardElement.segment = selectedSegment.value;
    //     dashboardElement.timeValue = timeVariable.value;
    //     dashboardElement.yValues = yVariables.value;
    //     if (apiToken.value && apiToken.value !== '') dashboardElement.apiToken = apiToken.value;
    //     dashboardElement.chartType = selectedChartType.value;
    //     dashboardElement.chartOptions = chartOptions.value;
    //     dashboardAdd(dashboardElement);
    // }

    function getChartOptions(options: any): void {
        // console.log('Chart Options: ', options);
        chartOptions.value = options;
        let chartOptionsObject = {
            chartOptions: options,
        };
        // Emit the new chart options to the Edit Element Dialog
        emit('newOptions', chartOptionsObject);
    }

    function clearChartOptions(event: any): void {
        chartOptions.value = {};
        let chartType = {
            chartType: event,
        };
        // Emit the new chart type to the Edit Element Dialog
        emit('newOptions', chartType);
    }

    function emitSegment(event: any): void {
        let segmentObject = {
            segment: event,
        };
        // Emit the new segment to the Edit Element Dialog
        emit('newOptions', segmentObject);
    }

    function emitTimeValue(event: any): void {
        let timeValObject = {
            timeVal: event,
        };
        // Emit the new time value to the Edit Element Dialog
        emit('newOptions', timeValObject);
    }

    function emitYValue(event: any): void {
        let yValObject = {
            yvals: event,
        };
        // Emit the new y values to the Edit Element Dialog
        emit('newOptions', yValObject);
    }
</script>
