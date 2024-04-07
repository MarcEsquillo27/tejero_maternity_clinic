<template>
    <v-container class="container-main pt-0 pl-0" fluid>
        <toolbar :toolbar="toolbar"></toolbar>
        <v-row>
            <v-col>
                <v-text-field v-model="search" label="search" outlined dense />
            </v-col>
            <v-col>
                <v-btn color="success" dense @click="toggleInsertDialog($event)"
                    >Add</v-btn
                >
            </v-col>
        </v-row>
        <v-simple-table dense :height="tableHeight">
            <thead>
                <tr>
                    <th>ID</th>
                    <th>Case No</th>
                    <th>Patient</th>
                    <th>Doctor</th>
                    <th>Room</th>
                    <th>Bed</th>
                    <th>Relationship</th>
                    <th>Guardian</th>
                    <th>Action</th>
                </tr>
            </thead>
            <tbody v-if="caseDataSearch.length">
                <tr v-for="(data, index) in caseDataSearch" :key="index">
                    <td>{{ data.id }}</td>
                    <td>{{ data.case_no }}</td>
                    <td>{{ data.patient_name }}</td>
                    <td>{{ data.doctor_name }}</td>
                    <td>{{ data.room_name }}</td>
                    <td>{{ data.bed_name }}</td>
                    <td>{{ data.relationship }}</td>
                    <td>{{ data.husband_name }}</td>
                    <td>
                        <v-icon color="success" @click="Edit(data)"
                            >mdi-pencil</v-icon
                        >
                        <v-tooltip bottom>
                            <template v-slot:activator="{ on }">
                                <v-icon
                                    color="error"
                                    @click="toggleDelete(data.id)"
                                    v-on="on"
                                    >mdi-bed-empty</v-icon
                                >
                            </template>
                            <span>Discharge Patient</span>
                            <!-- Tooltip content -->
                        </v-tooltip>
                    </td>
                </tr>
            </tbody>
            <tbody v-else>
                <tr>
                    <td style="color: red; text-align: center" colspan="9">
                        NO DATA FOUND
                    </td>
                </tr>
            </tbody>
        </v-simple-table>
        <insert-dialog
            :data="tempData"
            :dialog="insertDialog"
            :dialogSetting="insertDialogSetting"
            @closeDialog="
                tempData = {};
                insertDialog = false;
            "
            @toggleSave="Insert()"
        ></insert-dialog>
        <edit-dialog
            :data="tempEditData"
            :dialog="editDialog"
            :dialogSetting="editDialogSetting"
            @closeDialog="
                tempEditData = {};
                editDialog = false;
            "
            @toggleSave="Update()"
        ></edit-dialog>

        <agree-dialog
            :agree="agree"
            @closeAgree="closeAgree()"
            @toggleAgree="toggleAgree()"
        ></agree-dialog>
        <snackbar :snackbar="snackbar"></snackbar>
        <!-- NOTE : Float Action -->
        <float-action
            @toggleDelete="toggleDelete"
            @toggleEditMode="toggleEditMode(editMode)"
            :editMode="editMode"
            :selectedRowsCnt="selectedRows.length"
        ></float-action>
    </v-container>
</template>

<script>
import ToolbarComponent from "../../includes/Toolbar";
// import FloatAction from '../../includes/FloatAction.vue'
import AgreeDialog from "../../includes/AgreeDialog.vue";
import AdmissionDialog from "./AdmissionDialog.vue";
import SnackBar from "../../includes/SnackBar.vue";
import auditRecord from "../../includes/auditFunction";

import { mapActions, mapState } from "vuex";
import moment from "moment";
export default {
    name: "PatientRegistration",
    components: {
        toolbar: ToolbarComponent,
        // 'float-action':FloatAction,
        "agree-dialog": AgreeDialog,
        "insert-dialog": AdmissionDialog,
        "edit-dialog": AdmissionDialog,
        snackbar: SnackBar,
    },
    data() {
        return {
            toolbar: {
                title: "Admission",
                subTitle: "Register Case",
            },

            agree: {
                dialog: false,
                title: "Delete",
                text: "Are you sure you want to delete",
            },
            snackbar: {
                show: false,
                color: "success",
                text: null,
            },
            insertDialogSetting: {
                title: "Insert Room Info",
                submitBtn: "Save",
            },
            editDialogSetting: {
                title: "Update Patient Info",
                submitBtn: "Update",
            },
            search: "",
            tableHeight: window.innerHeight - 180,
            insertDialog: false,
            editDialog: false,
            deleteDialog: false,
            editMode: false,
            selectedRows: [],
            tempName: null,
            tempId: null,
            dialogBtn: false,

            tempData: {
                case_no: null,
                patient_id: null,
                doctor_id: null,
                room_id: null,
                bed_name: null,
                bed_id: null,
            },
            tempEditData: {},
        };
    },
    methods: {
        ...mapActions(["getAdmision"]),
        closeAgree() {
            this.agree.dialog = false;
        },
        toggleAgree() {
            this.Delete();
        },
        toggleInsertDialog() {
            (this.tempData = {}), (this.insertDialog = true);
        },
        toggleEditMode(val) {
            (this.tempData = {
                room_type: null,
                name: null,
                beds: [],
            }),
                (this.editMode = !this.editMode);
        },
        toggleDelete(id) {
            this.tempId = id;
            this.agree.dialog = true;
        },
        Edit(data) {
            console.log(data);
            this.tempEditData = structuredClone(data);
            this.editDialog = true;
        },
        Insert() {
            this.loadMore = true;
            this.tempData.case_no = `${moment().format("YYYY")}-${
                this.tempData.case_no
            }`;
            axios({
                method: "post",
                url: "case_insert",
                data: this.tempData,
            })
                .then((res) => {
                    console.log(res.data);
                    if (res.data == "success") {
                        this.snackbar.show = true;
                        this.snackbar.text = "Success Insert";
                        this.snackbar.color = "success";
                        axios({
                            method: "post",
                            url: "update_bed",
                            data: { id: this.tempData.bed_name },
                        });
                        // this.$refs.Insert.resetValidation()
                        this.getAdmision();
                        let audit = {
                            action: "New Patient",
                            description: "New Patient Admit",
                            name: this.loggedInUser.name,
                            position: this.loggedInUser.access,
                            drawerLink: "Admission",
                            date: moment().format("YYYY-MM-DD"),
                        };
                        auditRecord.Insert(audit);
                        this.insertDialog = false;
                    } else if (res.data == 1) {
                        this.snackbar.show = true;
                        this.snackbar.text = "Case Number Already Exist";
                        this.snackbar.color = "error";
                    }
                })
                .catch((err) => {
                    console.log(err);
                });
        },
        Update() {
            this.loadMore = true;
            axios({
                method: "post",
                url: "case_update",
                data: this.tempEditData,
            })
                .then((res) => {
                    console.log(res.data);
                    this.snackbar.show = true;
                    this.snackbar.text = "Success Update";
                    this.snackbar.color = "success";
                    // this.$refs.Insert.resetValidation()
                    this.getAdmision();
                    let audit = {
                        action: "Update Patient",
                        description: "Patient Info Updated",
                        name: this.loggedInUser.name,
                        position: this.loggedInUser.access,
                        drawerLink: "Admission",
                        date: moment().format("YYYY-MM-DD hh:mm:ss"),
                    };
                    auditRecord.Insert(audit);
                    this.editDialog = false;
                })
                .catch((err) => {
                    console.log(err);
                });
        },
        Delete() {
            axios({
                method: "post",
                url: "case_delete",
                data: {
                    id: this.tempId,
                },
            })
                .then((res) => {
                    this.getAdmision();
                    this.snackbar.show = true;
                    this.snackbar.text = "Success Delete";
                    this.snackbar.color = "success";
                    this.closeAgree();
                    let audit = {
                        action: "Delete Patient",
                        description: "Patient Info Deleted",
                        name: this.loggedInUser.name,
                        position: this.loggedInUser.access,
                        drawerLink: "Admission",
                        date: moment().format("YYYY-MM-DD"),
                    };
                    auditRecord.Insert(audit);
                })
                .catch((err) => {
                    console.log(err);
                });
        },
    },
    computed: {
        caseDataSearch() {
            const newSearch = this.search.toLowerCase();
            return this.admmisionData.filter((item) => {
                if (item.deleted_at) {
                    // Exclude the item if deleted_at is not null
                    return false;
                }
                return (
                    item.case_no.toLowerCase().includes(newSearch) ||
                    item.patient_name.toLowerCase().includes(newSearch) ||
                    item.doctor_name.toLowerCase().includes(newSearch) ||
                    item.room_name.toLowerCase().includes(newSearch) ||
                    item.bed_name.toLowerCase().includes(newSearch) ||
                    item.relationship.toLowerCase().includes(newSearch) ||
                    item.husband_name.toLowerCase().includes(newSearch)
                    // Add more fields to search if needed
                );
            });
        },
        ...mapState([
            "rules",
            "admmisionData",
            "selectRoomType",
            "loggedInUser",
        ]),
    },
    watch: {},

    mounted() {
        this.getAdmision();
    },
};
</script>

<style></style>
