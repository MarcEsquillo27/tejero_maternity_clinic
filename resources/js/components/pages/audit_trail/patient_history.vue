<template>
    <v-container fluid>
        <h1>Patient History</h1>
        <v-row>
            <v-col cols="6" sm="6">
                <v-text-field
                    v-model="search"
                    outlined
                    color="primary"
                    dense
                    label="Search"
                />
            </v-col>
            <v-col>
                <v-menu
                    v-model="startdateMenu"
                    :close-on-content-click="false"
                    :nudge-right="40"
                    transition="scale-transition"
                    offset-y
                    min-width="auto"
                >
                    <template v-slot:activator="{ on, attrs }">
                        <v-text-field
                            outlined
                            dense
                            v-model="startDate"
                            label="Start Date"
                            prepend-icon="mdi-calendar"
                            readonly
                            v-bind="attrs"
                            v-on="on"
                        ></v-text-field>
                    </template>
                    <v-date-picker
                        v-model="startDate"
                        @input="startdateMenu = false"
                    ></v-date-picker>
                </v-menu>
            </v-col>
            <v-col>
                <v-menu
                    v-model="enddateMenu"
                    :close-on-content-click="false"
                    :nudge-right="40"
                    transition="scale-transition"
                    offset-y
                    min-width="auto"
                >
                    <template v-slot:activator="{ on, attrs }">
                        <v-text-field
                            outlined
                            dense
                            v-model="endDate"
                            label="End Date"
                            prepend-icon="mdi-calendar"
                            readonly
                            v-bind="attrs"
                            v-on="on"
                        ></v-text-field>
                    </template>
                    <v-date-picker
                        v-model="endDate"
                        @input="enddateMenu = false"
                    ></v-date-picker>
                </v-menu>
            </v-col>
            <v-col>
                <v-btn color="primary" @click="filterLog()">Filter Date</v-btn>
            </v-col>
        </v-row>
        <v-row>
            <v-col cols="12">
                <v-simple-table class="border" dense>
                    <thead>
                        <tr>
                            <th v-for="(items, index) in headers" :key="index">
                                {{ items.text }}
                            </th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr
                            :style="
                                items.stock <= 5
                                    ? 'background-color:#CE4257;color:white;'
                                    : ''
                            "
                            v-for="(items, index) in caseDataSearch"
                            :key="index"
                        >
                            <td>{{ items.id }}</td>
                            <td>{{ items.action }}</td>
                            <td>{{ items.description }}</td>
                            <td>{{ items.name }}</td>
                            <td>{{ items.position }}</td>
                            <td>{{ items.drawerLink }}</td>
                            <td>{{ fixedDate(items.date) }}</td>
                        </tr>
                    </tbody>
                </v-simple-table>
            </v-col>
        </v-row>
        <v-row
            ><v-col>
                <v-pagination
                    v-model="currentPage"
                    :length="numPages"
                    @input="changePage"
                /> </v-col
        ></v-row>
    </v-container>
</template>

<script>
import axios from "axios";
import moment from "moment";
export default {
    data: () => {
        return {
            search: "",
            startDate: moment().subtract(1, "month").format("YYYY-MM-DD"),
            endDate: moment().format("YYYY-MM-DD"),
            startdateMenu: false,
            enddateMenu: false,
            currentPage: 1, // Current page number
            itemsPerPage: 5,
            addButton: false,
            editButton: false,
            add_dialog: false,
            showDatePicker: false,
            from_date: null,
            all_products: [],
            insertItem: {
                productNumber: "",
            },
            menu: false,
            date: "",
            headers: [
                { text: "ID", value: "id" },
                { text: "Action ", value: "action" },
                { text: "Description ", value: "description" },
                { text: "Name", value: "product_number" },
                { text: "Access / Position", value: "quantity" },
                { text: "Drawer Link", value: "drawerLink" },
                { text: "Date", value: "date" },
            ],
        };
    },
    computed: {
        caseDataSearch() {
            const newSearch = this.search.toLowerCase();
            return this.paginatedItems.filter((item) => {
                return (
                    item.drawerLink.toLowerCase().includes(newSearch) ||
                    item.position.toLowerCase().includes(newSearch) ||
                    item.name.toLowerCase().includes(newSearch) ||
                    item.description.toLowerCase().includes(newSearch) ||
                    item.action.toLowerCase().includes(newSearch)
                    // Add more fields to search if needed
                );
            });
        },
        paginatedItems() {
            const startIndex = (this.currentPage - 1) * this.itemsPerPage;
            const endIndex = startIndex + this.itemsPerPage;
            return this.all_products.slice(startIndex, endIndex);
        },
        numPages() {
            return Math.ceil(this.all_products.length / this.itemsPerPage);
        },
    },

    methods: {
        filterLog() {
            const date = {
                end_date: this.endDate,
                start_date: this.startDate,
            };
            axios({
                method: "post",
                url: "audit",
                data: date,
            }).then((res) => {
                this.all_products = res.data;
            });
        },
        changePage(page) {
            this.currentPage = page;
        },
        fixedDate(val) {
            return moment(val).format("YYYY-MM-DD hh:mm:ss");
        },
        overallCapital(val) {
            var totalallCapital = null;
            totalallCapital = val.stock * val.salesPrice;
            //  totalCapital = val.salesPrice - val.originalPrice
            return !totalallCapital ? 0 : totalallCapital;
        },
        capitalPerItem(val) {
            var totalCapital = null;
            totalCapital = val.salesPrice - val.originalPrice;
            //  totalCapital = val.salesPrice - val.originalPrice
            return !totalCapital ? 0 : totalCapital;
        },
        editInventory(val) {
            this.addButton = false;
            this.insertItem = {};
            this.add_dialog = true;
            this.insertItem = val;
            this.editButton = true;
        },
        openDialog() {
            this.add_dialog = true;
            this.addButton = true;
            this.editButton = false;
            this.insertItem = {};
        },
        getAllProducts() {
            const date = {
                end_date: this.endDate,
                start_date: this.startDate,
            };
            axios({
                method: "post",
                url: "audit",
                data: date,
            }).then((res) => {
                // console.log(res.data);
                this.all_products = res.data.filter((rec) => {
                    if (rec.drawerLink == "Admission") return rec;
                });
            });
        },
    },
    mounted() {
        this.getAllProducts();
    },
};
</script>

<style scoped>
/* Add any custom styles here */
</style>
