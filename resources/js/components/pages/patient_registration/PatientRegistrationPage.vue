<template>
    <v-container class="container-main pt-0 pl-0" fluid>
        <toolbar :toolbar="toolbar"></toolbar>
        <v-row>
            <v-col>
        <v-text-field v-model="search" label="search" outlined dense />

            </v-col>
            <v-col>
        <v-btn color="success" dense @click="toggleInsertDialog($event)">Add</v-btn>

            </v-col>
        </v-row>

        <v-simple-table dense :height="tableHeight">
            <thead>
                <tr>
                    <th class="text-left" style="width:20px">ID</th> 
                    <th>Name</th>
                    <th>Sex</th>
                    <th>Weight</th>
                    <th>Age</th>
                    <th>BirthDay</th>
                    <th>BirthPlace</th>
                    <th>Address</th>
                    <th>Religion</th>
                    <th>Nationality</th>
                    <th>Occupation</th>
                    <th>Civil Status</th>
                    <th>Contact</th>
                    <th>Action</th> 
                </tr>
            </thead>
            <tbody v-if="searchPatient.length" >
                <tr v-for="(item,index) in searchPatient" :key="index">
       
                    <td>{{item.id}}</td>
                    <td>{{item.name}}</td>
                    <td>{{selectSex[item.sex_id - 1].name}}</td>
                    <td>{{item.weight}}</td>
                    <td>{{item.age}}</td>
                    <td>{{item.birthday}}</td>
                    <td>{{item.birth_place}}</td>
                    <td>{{item.address_txt}}</td>
                    <td>{{item.religion_name}}</td>
                    <td>{{item.nationality}}</td>
                    <td>{{item.occupation}}</td>
                    <td>{{selectCivil[item.civil_status_id - 1].name}}</td>
                    <td>{{item.contact_no}}</td>
                     <td >
                        <v-icon color="success" @click="Edit(item)">mdi-pencil</v-icon>
                        <v-icon color="error" @click="toggleDelete(item.id)">mdi-delete</v-icon>
                    </td>
                </tr>
                
            </tbody>
            <tbody v-else>
                <tr>
                   <td style="color:red;" colspan="14">NO DATA FOUND</td>
                </tr>
            </tbody>
        </v-simple-table>

        <v-overlay :value="loadMore" overlay>
            <v-progress-circular indeterminate size="64"></v-progress-circular>
        </v-overlay>
        <!-- NOTE: insert Dialog -->
        
        <insert-dialog :patient="tempPatient" :dialog="insertDialog" :dialogSetting="insertDialogSetting" @closeDialog="insertDialog = false" @toggleSave="Insert()"></insert-dialog>
        
        <edit-dialog :patient="tempEditPatient" :dialog="editDialog" :dialogSetting="editDialogSetting" @closeDialog="editDialog = false" @toggleSave="Update()"></edit-dialog>

        <agree-dialog :agree="agree" @closeAgree="closeAgree()" @toggleAgree="Delete()"></agree-dialog>
        <snackbar :snackbar="snackbar"></snackbar>
        <!-- NOTE : Float Action -->
        <float-action 
        @toggleInsertDialog="toggleInsertDialog($event)"
        @toggleDelete="toggleDelete"
        @toggleEditMode="toggleEditMode(editMode)"
        :floatbtn="floatbtn"
        :editMode="editMode"
        :selectedRowsCnt="selectedRows.length"
        ></float-action>
    </v-container>
</template>
<script>
import ToolbarComponent from '../../includes/Toolbar'
// import FloatAction from '../../includes/FloatAction.vue'
import PatientDialog from './PatientDialog.vue'
import AgreeDialog from '../../includes/AgreeDialog.vue'
import SnackBar from '../../includes/SnackBar.vue'
import auditRecord from '../../includes/auditFunction'
import { mapActions, mapState } from 'vuex'
export default {
    name:'PatientRegistration',
    components:{
        'toolbar':ToolbarComponent,
        // 'float-action':FloatAction,
        'insert-dialog':PatientDialog,
        'edit-dialog':PatientDialog,
        'agree-dialog':AgreeDialog,
        'snackbar':SnackBar,
    },
    data() {
        return {
            search:"",
            employee:[],
            empIds:null,
            toolbar:{
                title:'Patient Registration',
                subTitle:'Register Patient System',
            },
            floatbtn:{
                add:true,
                edit:true,
                delete:true,
            },
            snackbar:{
                show:false,
                color:'success',
                text:null,
            },
            selectAll:false,
            selectedRows:[],
            editMode:false,
            loadMore:false,
            insertDialog:false,
            editDialog:false,
            insertDialogSetting:{
                title:'Insert Patient Info',
                submitBtn:'Save'
            },
            editDialogSetting:{
                title:'Update Patient Info',
                submitBtn:'Update'
            },
            agree:{
                dialog:false,
                title:'Delete',
                text:'Are you sure you want to delete',
            },
            tableHeight:window.innerHeight - 180,
            dialogBtn:false,
            tempPatient:{
                name:null,
                age:null,
                sex_id:null,
                civil_status_id:null,
                occupation:null,
                religion_id:null,
                nationality:null,
                contact_no:null,
                birthday:null,
                birth_place:null,
                province_id:null,
                municipality_id:null,
                barangay_id:null,
                house_address:null,
            },
            tempEditPatient:{}

        }
    },

    methods:{
        ...mapActions(['getProvince','getReligion','getPatient']),
        closeAgree(){
            this.agree.dialog = false
        },
        toggleAgree(){
            this.Delete()
        },
        toggleInsertDialog(){
            this.tempPatient.nationality = "FILIPINO"
            this.insertDialog=true
        },
        toggleDelete(id){
            console.log('delete')
            this.tempToDeleteId = id
            this.agree.dialog = true
            // sdfasdf
        },
        Insert(){
            this.loadMore = true
            axios({
                method : 'post',
                url : 'patient_insert',
                data: this.tempPatient
            })
            .then(res =>{
                console.log(res.data)
                this.loadMore = false
                this.snackbar.show=true
                this.snackbar.text= "Success Insert"
                this.snackbar.color="success"
                // this.$refs.Insert.resetValidation()
                this.insertDialog = false
                this.getPatient()
               let audit = {
                action : "Add New Patient",
                description : "New patient registered",
                name : this.loggedInUser.name,
                position : this.loggedInUser.access,
                drawerLink : "Patient Registration",
                date : moment().format("YYYY-MM-DD")
               }
               auditRecord.Insert(audit)

            }).catch(err =>{
                console.log(err)
            });
        },
        toggleEditMode(val){
            this.editMode = !this.editMode
        },
        Edit(item){
            console.log(item)
            this.tempEditPatient = structuredClone(item)
            this.editDialog = true
        },
        Update(){
            this.loadMore = true
            axios({
                method : 'post',
                url : 'patient_update',
                data : this.tempEditPatient,
            })
            .then(res =>{
                console.log(res.data)
                this.loadMore = false
                this.snackbar.show=true
                this.snackbar.text= "Success Update"
                this.snackbar.color="success"
                // this.$refs.Insert.resetValidation()
                this.editDialog = false
                this.getPatient()
                let audit = {
                action : "Update Patient",
                description : "Patient Info Updated",
                name : this.loggedInUser.name,
                position : this.loggedInUser.access,
                drawerLink : "Patient Registration",
                date : moment().format("YYYY-MM-DD")
               }
               auditRecord.Insert(audit)
            }).catch(err =>{
                console.log(err)
            });
        },
        Delete(){
            console.log(this.tempToDeleteId)
            this.loadMore = true

            axios({
                method : 'post',
                url : 'patient_delete',
                data : {id:this.tempToDeleteId},
            })
            .then(res =>{
                console.log(res.data)
                this.agree.dialog = false
                this.snackbar.show=true
                this.snackbar.text= "Success Delete"
                this.snackbar.color="success"
                this.getPatient()
                let audit = {
                action : "Delete Patient",
                description : "Patient Info Deleted",
                name : this.loggedInUser.name,
                position : this.loggedInUser.access,
                drawerLink : "Patient Registration",
                date : moment().format("YYYY-MM-DD")
               }
               auditRecord.Insert(audit)
            }).catch(err =>{
                console.log(err)
            });
        },
    },
    computed:{
        searchPatient() {
      const newSearch = this.search.toLowerCase();
      return this.patientData.filter(item => {
        return (
          item.name.toLowerCase().includes(newSearch) ||
          item.address_txt.toLowerCase().includes(newSearch) ||
          item.contact_no.toLowerCase().includes(newSearch)
          // Add more fields to search if needed
        );
      });
    },
        ...mapState([
            'rules',
            'provinceMaster',
            'religionData',
            'selectSex',
            'selectCivil',
            'patientData',
            'loggedInUser',
        ]),
        seqs(){
                return (this.insertDialog) ? _.reverse(_.range(1,this.registerData.length + 2)) : _.reverse(_.range(1,this.registerData.length + 1))
            },
        last_seq(){
            return this.registerData.length + 1
        },
    },
    watch:{
        insertDialog(val){
            if(!val){
                this.tempPatient = {
                    name:null,
                    age:null,
                    sex_id:null,
                    civil_status_id:null,
                    occupation:null,
                    religion_id:null,
                    nationality:null,
                    contact_no:null,
                    birthday:null,
                    birth_place:null,
                    province_id:null,
                    municipality_id:null,
                    barangay_id:null,
                    house_address:null,
                }
            }
        },
        editDialog(val){
            if(!val){
                this.tempPatient = {
                    name:null,
                    age:null,
                    sex_id:null,
                    civil_status_id:null,
                    occupation:null,
                    religion_id:null,
                    nationality:null,
                    contact_no:null,
                    birthday:null,
                    birth_place:null,
                    province_id:null,
                    municipality_id:null,
                    barangay_id:null,
                    house_address:null,
                }
            }
        },
    },
    created() {
        this.getPatient()
        this.getProvince()
        this.getReligion()
    },
}
</script>
<style scoped>
th, td{
    /* border: black 1px solid; */
    /* padding:0px !important; */
    text-align: center;
}
.mt-4{
    margin-top:16px;
}
.inline{
    display: inline-flex;
}
.uppercase >>> input{
    text-transform: uppercase;
}
/* .v-card__title {
    background: #455A64 !important;
    color: #ffffff !important;
    padding: 4px 16px !important;
} */
/* .boxing{
    display: block;
    text-align: -webkit-center;
    42069 42070 42071 42072 42073 42074
} */
</style>

