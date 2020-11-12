<template>
    <v-main class="list">
        <h3 class="text-h3 font-weight-medium mb-5">
            To Do List
        </h3>
    <div class="d-flex align-start">
        <v-card>
            <v-card-title>
                <v-text-field
                    v-model="search"
                    append-icon="mdi-magnify"
                    label="Search"
                    single-line
                    hide-details
                    >
                </v-text-field>

                <v-spacer></v-spacer>
                <v-btn color="success" dark @click="dialog = true">
                    Tambah
                </v-btn>
            </v-card-title>
            
            <v-data-table 
                :headers="headers" 
                :items="todos"
                :search="search"
                >

                <template v-slot:[`item.priority`]="{item}">
                    <v-chip v-if="item.priority == 'Penting'" color="red" outlined>
                        {{item.priority}}
                    </v-chip>
                    <v-chip v-else-if="item.priority == 'Biasa'" color="success" outlined>
                        {{item.priority}}
                    </v-chip>
                    <v-chip v-else color="primary" outlined>
                        {{item.priority}}
                    </v-chip>
                </template>
                    
                <template v-slot:[`item.actions`]="{item}">
                    <v-icon small class="mr-2" @click="showKanan(item)" color="blue">
                        mdi-file
                    </v-icon>
                </template>

                <template v-slot:[`item.checks`]="{item}">
                    <v-checkbox multiple :key="item" @click.capture.stop="checkItem(item)"/>              
                </template>

            </v-data-table>
        </v-card>



        <v-card v-model="showRight" v-if="showRight==true" class="ml-3 pa-5">
            <h>
                {{editedItem2.task}} 
                <v-chip v-if="editedItem2.priority == 'Penting'" color="red" text-color="white">
                    {{editedItem2.priority}}
                </v-chip>
                <v-chip v-else-if="editedItem2.priority == 'Biasa'" color="primary" text-color="white">
                    {{editedItem2.priority}}
                </v-chip>
                <v-chip v-else color="success" text-color="white">
                    {{editedItem2.priority}}
                </v-chip>
                <br>
                {{editedItem2.note}}
            </h>
            <br>
            <v-icon
                small
                class="mr-2"
                @click="editItem()"
                color="blue"
            >
                mdi-pencil
            </v-icon>
            <v-icon
                small
                @click="deleteItem()"
                color="red"
            >
                mdi-delete
            </v-icon>
        </v-card>
    </div>
        <v-card v-if="check.length" style="overflow-y:scroll; height:200px;" class="pa-5 my-2">
            Delete Multiple
                <br>
                <v-list-item v-for="(item, i) in check" :key="i">
                    <v-list-item-content>
                        <v-list-item-title>•  {{item.task}}</v-list-item-title>
                    </v-list-item-content>
                </v-list-item>
                <v-btn color="error" dark @click="deleteCheck">
                    Hapus Semua
            </v-btn>
        </v-card>

        <v-dialog v-model="dialog" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline">
                        Form Todo
                    </span>
                </v-card-title>

                <v-card-text>
                    <v-container>
                        <v-text-field
                        v-model="formTodo.task"
                        label="Task"
                        required
                        >
                        </v-text-field>

                        <v-select
                            v-model="formTodo.priority"
                            :items="['Penting','Biasa','Tidak penting']"
                            label="Priority"
                            required
                        >
                        </v-select>

                        <v-textarea
                            v-model="formTodo.note"
                            label="Note"
                            required
                        >
                        </v-textarea>
                    </v-container>
                </v-card-text>
                <v-card-actions>
                    <v-spacer>

                    </v-spacer>
                    <v-btn color="blue darken-1" text @click="cancel">
                        Cancel
                    </v-btn>
                    <v-btn color="blue darken-1" text @click="save">
                        Save
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <!-- DIALOG EDIT -->
        <v-dialog v-model="dialogEdit" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline">
                        Form Todo Edit
                    </span>
                </v-card-title>

                <v-card-text>
                    <v-container>
                        <v-text-field
                        v-model="editedItem.task"
                        label="Task"
                        required
                        >
                        </v-text-field>

                        <v-select
                            v-model="editedItem.priority"
                            :items="['Penting','Biasa','Tidak penting']"
                            label="Priority"
                            required
                        >
                        </v-select>

                        <v-textarea
                            v-model="editedItem.note"
                            label="Note"
                            required
                        >
                        </v-textarea>
                    </v-container>
                </v-card-text>
                <v-card-actions>
                    <v-spacer>

                    </v-spacer>
                    <v-btn color="blue darken-1" text @click="cancelEdit">
                        Cancel
                    </v-btn>
                    <v-btn color="blue darken-1" text @click="saveEditKu">
                        Update
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <!-- TEKAN KENE DIALOG EDITE -->

        <v-dialog v-model="dialogConfirmation" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline">
                        Delete Confirmation
                    </span>
                </v-card-title>

                <v-card-actions>
                    <v-spacer>

                    </v-spacer>
                    <v-btn color="green" text @click="cancelConfirmation" outlined>
                        Cancel
                    </v-btn>
                    <v-btn color="red" text @click="deleteConfirmation" outlined>
                        Delete
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
    </v-main>
</template>

<script>
export default {
    name:"List",
    data()
    {
        return{
            search: null,
            editedIndex: -1,
            anotherEdited : -1,
            editedItem: {
                task: "",
                priority: "",
                note: "",
            },

            editedItem2: {
                task: "",
                priority: "",
                note: "",
            },
            dialog: false,
            dialogEdit: false,
            dialogConfirmation : false,
            indexDelete : null,
            showRight : false,
            headers:[
                {
                    text : "Task",
                    align: "start",
                    sortable : true,
                    value: "task",
                },
                {
                    text : "Priority",
                    value : "priority"
                },
                {
                    text : "Actions",
                    value : "actions"
                },
                {
                    text: " ",
                    value : "checks"
                },
            ],
            todos: [
                {
                    task: "bernafas",
                    priority : "Penting",
                    note: "huffttt",
                },
                {
                    task : "nongkrong",
                    priority : "Tidak penting",
                    note : "bersama teman2",
                },
                {
                    task : "masak",
                    priority: "Biasa",
                    note : "masak air 500 ml",
                },
            ],
            formTodo:{
                task : null,
                priority : null,
                note : null,
            },
            check: [],
        };
    },
    methods:{
        save(){
            this.todos.push(this.formTodo);
            this.resetForm();
            this.dialog = false;
        },

        cancel(){
            this.resetForm();
            this.dialog = false;
        },

        resetForm(){
            this.formTodo = {
                task: null,
                priority: null,
                note: null,
            };
        },

        deleteItem(item){
            this.indexDelete = this.todos.indexOf(item);
            this.dialogConfirmation = true;
        },

        deleteConfirmation(){
            this.todos.splice(this.editedIndex, 1);
            this.dialogConfirmation = false;
            this.showRight = false;
        },

        cancelConfirmation(){
            this.dialogConfirmation = false;
        },

        showKanan(item){
            Object.assign(this.editedItem, item)
            Object.assign(this.editedItem2, item)
            this.editedIndex = this.todos.indexOf(item)
            this.showRight = true;
        },

        editItem(){
            this.dialogEdit = true
        },

        saveEditKu(){
            Object.assign(this.todos[this.editedIndex], this.editedItem);
            this.resetForm();
            this.dialogEdit = false;
            this.showRight = false;
        },

        cancelEdit(){
            this.resetForm();
            this.dialogEdit = false;
        },
        deleteCheck(){             
            this.showRight = false
            for(var i = 0; i < this.check.length; i++){            
                this.todos.splice( this.todos.indexOf(this.check[i]), 1);
            }
            this.check=[];
                                        
        },
        checkItem(item){
            if(this.check.includes(item)) {
                this.check.splice(this.check.indexOf(item), 1);
            } else {
                this.check.push(item);                
            }
        },
    },
};
</script>