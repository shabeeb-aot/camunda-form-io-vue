<template>
  <b-container fluid class="task-outer-container">
    <b-row class="cft-service-task-list">
  
        <b-col cols="*" xl="4" lg="4" md="4" sm="12" v-if="tasks && tasks.length" class="cft-first">
          <div class="cft-filter-sort"> 
            <b-col cols="5">
                <select  v-model="selectSortBy" @change="fetchOnSorting">
                    <option selected value="created">Created</option>
                    <option value="dueDate">Due-Date</option>
                    <option value="followUpDate">Follow-up Date</option>
                    <option value="name">Task Name</option>
                    <option value="assignee">Assignee</option>
                </select>
                <a v-if="isAsc" @click="toggleSort" href="#" title="Ascending">
                    <b-icon-chevron-up></b-icon-chevron-up>
                </a>
                <a v-else  @click="toggleSort" href="#" title="Descending">
                    <b-icon-chevron-down></b-icon-chevron-down>
                </a>
            </b-col>
            <div class="cft-filter-dropdown">
                <button class="cft-filter-dropbtn mr-0"><i class="bi bi-filter-square"/></button>
                <b-list-group  v-if="filterList && filterList.length" class="cft-filter-dropdown-content">
                <b-list-group-item button v-for="(filter, idx) in filterList" :key="filter.id"
                @click="fetchTaskList(filter.id); togglefilter(idx)"
                :class="{'cft-selected': idx == activefilter}">
                    <div class="col-12">
                    {{filter.name}} ({{filter.itemCount}})
                    </div>   
                </b-list-group-item>
                </b-list-group>
            </div>
        </div>
          <b-list-group class="cft-list-container">
                      
              <div class="cft-filter-container">
                  <input type="text" class="cft-filter" placeholder="Filter Tasks"/>
                      {{tasks.length}}
              </div>
              <b-list-group-item button v-for="(task, idx) in tasks" v-bind:key="task.id" 
                  v-on:click="toggle(idx)"
                  :class="{'cft-selected': idx == activeIndex}">
                  <div @click="setselectedTask(task.id)" class="cft-select-task">
                      <b-row>
                      <div class="col-12">
                        <h5>{{ task.name }}</h5>
                        <div class="cft-task-details-assign assigne-details ">
                              <div >
                                  {{ getProcessDataFromList(getProcessDefinitions, task.processDefinitionId, 'name') }}         
                              </div>
                              <div title="Task assignee" >
                                {{task.assignee}}
                              </div>      
                        </div>

                        <div class="cft-task-details-assign font-11" >
                              <div >
                                  <div v-if="task.due">
                                    Due {{ timedifference(task.due) }}
                                </div>
                                <div v-if="task.followUp">
                                    Follow-up {{ timedifference(task.followUp) }} 
                                </div>
                                    Created {{ timedifference(task.created) }}         
                                </div>
                                <div title="Task assignee" >
                                 {{ task.priority }}
                                </div>      
                        </div>

                      </div>
                      </b-row>
                      <!-- <b-row class="cft-task-row-2">
                          <div class="col-6 pr-0">
                              {{ getProcessDataFromList(getProcessDefinitions, task.processDefinitionId, 'name') }}         
                          </div>
                          <div title="priority" class="col-6 pr-0 text-right">
                              {{task.assignee}}
                          </div>
                      </b-row>
                      <b-row class="cft-task-row-3">
                          <b-col lg=8 xs=8 class="pr-0" title="task.created">
                              <div v-if="task.due">
                                  Due {{ timedifference(task.due) }}
                              </div>
                              <div v-if="task.followUp">
                                  Follow-up {{ timedifference(task.followUp) }} 
                              </div>
                                  Created {{ timedifference(task.created) }}
                          </b-col>
                          <b-col lg=4 xs=4 sm=4 class="pr-0 text-right" title="priority">
                              {{ task.priority }}
                          </b-col>
                      </b-row> -->
                  </div>
                  </b-list-group-item>
              </b-list-group>
          </b-col>
        <b-col cols="4" v-else> 
          <b-row class="cft-not-selected mt-2 ml-1 row">
                    <b-icon icon="exclamation-circle-fill" variant="secondary" scale="1"></b-icon>
                    <p>No tasks found in the list.</p>
          </b-row>
        </b-col>
        <b-col  v-if="selectedTask"> <div class="service-task-details">
            <b-row class="ml-0 task-header"> {{task.name}}</b-row>
            <b-row class="ml-0 task-name">{{taskProcess}}</b-row>
            <b-row class="ml-0" title="application-id">Application # {{ applicationId}}</b-row>
            
            <div>
              <b-row class="cft-actionable">
                  <b-col>
                    <DatePicker 
                    type="datetime"
                    placeholder="Set Follow-up date"
                    v-model="setFollowup"
                    @change="updateFollowUpDate"
                    >
                    </DatePicker>
                  </b-col>
                  <b-col>
                    <DatePicker 
                    type="datetime"
                    placeholder="Set Due Date"
                    v-model="setDue"
                    @change="updateDueDate"
                    >
                    </DatePicker>
                  </b-col>
                  <b-col>
                    <b-button variant="outline-primary" v-b-modal.AddGroupModal v-if="groupListNames"><b-icon :icon="'grid3x3-gap-fill'"></b-icon> {{String(groupListNames)}} </b-button>
                    <b-button variant="outline-primary" v-b-modal.AddGroupModal v-else><b-icon :icon="'grid3x3-gap-fill'"></b-icon> Add Groups</b-button>
                    <b-modal
                      id="AddGroupModal"
                      ref="modal"
                      title="Manage Groups"
                      ok-title="Close"
                      ok-only
                    >
                      <div class="modal-text">
                          <b-icon icon="exclamation-circle"></b-icon>
                          You can add a group by typing a group ID into the input field and afterwards clicking the button with the plus sign.
                          <b-row class="mt-3 mb-3">
                              <b-col>
                                  <b-button variant="primary" @click="addGroup">
                                      <span>Add a group</span>
                                      <span><b-icon-plus></b-icon-plus></span>
                                  </b-button>
                              </b-col>
                              <b-col>
                                  <input type="text" placeholder="Group ID" v-model="setGroup" v-on:keyup.enter="addGroup">
                              </b-col>
                          </b-row>
                          <b-row>
                              <b-col v-if="groupList.length">
                                  <ul v-for="g in groupList" :key="g.groupId">
                                      <div class="mt-1">
                                          <b-icon-x variant="danger" font-scale="1.5" @click="deleteGroup(g.groupId)"></b-icon-x>
                                          <span>{{g.groupId}}</span>
                                      </div>
                                  </ul>
                              </b-col>
                          </b-row>
                      </div>
                    </b-modal>
                  </b-col>
                  <b-col>
                      <b-button variant="outline-primary" v-if="task.assignee" @click="onUnClaim">
                          {{task.assignee}}
                        <b-icon-person-x-fill></b-icon-person-x-fill>
                      </b-button>
                      <b-button variant="outline-primary" v-else @click="onClaim">
                        <b-icon-person-fill></b-icon-person-fill>
                        Claim
                      </b-button>
                  </b-col>
              </b-row>

              <div>
                  <b-tabs content-class="mt-3" v-if="showfrom">
                    <b-tab title="Form">
                        <!-- <div v-if="task.assignee===userName" class="ml-4 mr-4">
                        <formio 
                          :src="formioUrl"
                          :submission="submissionId"
                          :form="formId"
                          :options="options"
                        >
                        </formio>
                        </div> -->
                        <div  class="ml-4 mr-4">
                            <b-overlay :show="task.assignee !== userName" variant="dark" opacity="0.90" blur="5px" spinner-type="none">
                                <formio :src="formioUrl"
                                :submission="submissionId"
                                :form="formId"
                                :options="task.assignee===userName ? options :  readoption"
                                >
                                </formio>
                            </b-overlay>
                        </div>
                    </b-tab>
                    <b-tab title="History"></b-tab>
                    <b-tab title="Diagram"></b-tab>
                  </b-tabs>
                </div>
            </div>
          </div>     
        </b-col>
        <b-col v-else><b-row class="cft-not-selected mt-2 ml-1 row">
            <b-icon icon="exclamation-circle-fill" variant="secondary" scale="1"></b-icon>
        <p>Select a task in the list.</p>
            </b-row></b-col>
    </b-row>
  </b-container>
</template>
<script lang="ts">
import 'bootstrap/dist/css/bootstrap.min.css';
import 'bootstrap-vue/dist/bootstrap-vue.css';
import 'formiojs/dist/formio.full.min.css'
import '../camundaFormIOTasklist.scss'
import 'vue2-datepicker/index.css';
import 'bootstrap-icons/font/bootstrap-icons.css';

import { Component, Prop, Vue } from 'vue-property-decorator'
import BootstrapVue from 'bootstrap-vue'
import CamundaRest from '../services/camunda-rest';
import DatePicker from 'vue2-datepicker'
import { Form } from 'vue-formio';
import {authenticateFormio} from "../services/formio-token";
import {getFormDetails} from "../services/get-formio";
import moment from "moment";

Vue.use(BootstrapVue)

@Component({
    components: {
        formio: Form,
        DatePicker,
    }
})
export default class Tasklist extends Vue {
@Prop() private bpmApiUrl !: string;
@Prop() private token !: string;
@Prop() private formIOResourceId !: string;
@Prop() private formIOReviewerId !: string;
@Prop() private formIOReviewer !: string;
@Prop() private formIOApiUrl!: string;
@Prop() private formsflowaiApiUrl!: string;
@Prop() private formsflowaiUrl!: string;
@Prop() private formIOUserRoles !: string;
@Prop() private userName !: string;

private tasks: Array<object> = []
private getProcessDefinitions: Array<object> = []
private taskProcess = null
private formId = ''
private submissionId = '' 
private formioUrl = ''
private activeIndex = 0
private task: any
private setFollowup = null
private setDue = null
private setGroup = null
private selectedTask = '' 
private showfrom = false
private readoption = {readOnly: true,}
private options =  {
    noAlerts: false,
    i18n: {
        en: {
            error: "Please fix the errors before submitting again.",
        },
    }
}
private filterList = []
private activefilter = 0
private applicationId = ''
private groupList = []
private groupListNames: any = null
private groupListItems: string[] = []
// private userName = ''
private userEmail = 'external'
// private formIOUserRoles = ''
private selectSortBy = 'created'
private selectSortOrder = 'desc'
private isAsc = false
private filterId = ''

checkPropsIsPassed() {
    if(! this.bpmApiUrl|| this.bpmApiUrl===""){
        console.error("bpmApiUrl prop not Passed")
    }

    else if(! this.token || this.token==="") {
        console.error("token prop not Passed")
    }

    else if(! this.formIOResourceId|| this.formIOResourceId==="") {
        console.error("formIOResourceId prop not passed")
    }

    else if(! this.formIOReviewerId|| this.formIOReviewerId==="") {
        console.error("formIOReviewerId prop not passed")
    }

    else if(! this.formIOApiUrl|| this.formIOApiUrl==="") {
        console.error("formIOApiUrl prop not passed")
    }

    else if(! this.formsflowaiApiUrl || this.formsflowaiApiUrl==="") {
        console.error("formsflow.ai API url prop not passed")
    }

    else if(! this.formsflowaiUrl || this.formsflowaiUrl==="") {
        console.error("formsflow.ai URL prop not passed")
    }
}

timedifference(date: Date) {
    return moment(date).fromNow();
}

getProcessDataFromList(processList: any[] ,processId: any,dataKey: string){
    const process = processList.find(process=>process.id===processId);
    return process && process[dataKey] ;
}

getTaskFromList(tasks: any[], taskId: string){
    const task = tasks.find(task=>task.id===taskId);
    return task;
}

setselectedTask(task: string){
    this.selectedTask = task
    this.fetchData()
}

toggle(index: number) {
    this.activeIndex = index
}


getBPMTaskDetail(taskId: string) {
    CamundaRest.getTaskById(this.token, taskId, this.bpmApiUrl).then((result) => {
        this.task = result.data;
    })

    this.showfrom = false
    CamundaRest.getVariablesByTaskId(this.token, this.selectedTask, this.bpmApiUrl).then((result)=> {
        this.formioUrl = result.data["formUrl"].value;
        const {formioUrl, formId, submissionId} = getFormDetails(this.formioUrl, this.formIOApiUrl);
        this.formioUrl = formioUrl; this.submissionId = submissionId; this.formId = formId

        this.showfrom = true
    });
}

getBPMTasks(){
    CamundaRest.getTasks(this.token, this.bpmApiUrl).then((result)=> {
        this.tasks = result.data;
    })
}

onClaim() {
    CamundaRest.claim(this.token,this.task.id, {userId: this.userName}, this.bpmApiUrl).then(()=> 
    {
        this.getBPMTaskDetail(this.task.id)
        this.getBPMTasks()
    }).catch((error) => {
        console.log("Error", error);
    })
}

onUnClaim(){ 
    CamundaRest.unclaim(this.token ,this.task.id, this.bpmApiUrl).then(()=> 
    {
        this.getBPMTaskDetail(this.task.id)
        this.getBPMTasks()
    }).catch((error) =>{
        console.log("Error", error)
    })
}

fetchTaskList(filterId: string) {
    this.filterId = filterId
    CamundaRest.filterTaskList(this.token, filterId, {
        "processVariables":[],"taskVariables":[],"caseInstanceVariables":[],
        "sorting":[{"sortBy": this.selectSortBy,"sortOrder": this.selectSortOrder }],
        "active":true},
    this.bpmApiUrl,).then((result) => {
        this.tasks = result.data;      
    }); 
}

toggleSort() {
    this.isAsc = !this.isAsc;
    if (this.isAsc){
        this.selectSortOrder = 'asc'
    }
    else {
        this.selectSortOrder = 'desc'
    }
    this.fetchOnSorting()
}

fetchOnSorting() {
    this.fetchTaskList(this.filterId);
}

updateFollowUpDate() {
    const referenceobject = this.task
    const timearr = moment(this.setFollowup).format("yyyy-MM-DD[T]HH:mm:ss.SSSZ").split('+')
    const replaceTimezone = timearr[1].replace(':', '')
    referenceobject["followUp"] = moment(this.setFollowup).format("yyyy-MM-DD[T]HH:mm:ss.SSSZ").replace(timearr[1], replaceTimezone) 
    CamundaRest.updateTasksByID(this.token, this.task.id, this.bpmApiUrl, referenceobject).then(()=> {
        console.log("Updated follow up date")
    }).catch((error) =>{
        console.log("Error", error)
    })
}

updateDueDate() {
    const referenceobject = this.task
    const timearr = moment(this.setDue).format("yyyy-MM-DD[T]HH:mm:ss.SSSZ").split('+')
    const replaceTimezone = timearr[1].replace(':', '')
    referenceobject["due"] = moment(this.setDue).format("yyyy-MM-DD[T]HH:mm:ss.SSSZ").replace(timearr[1], replaceTimezone)
    CamundaRest.updateTasksByID(this.token, this.task.id, this.bpmApiUrl, referenceobject).then(()=> {
        console.log("Update due date")
        this.getBPMTaskDetail(this.task.id)
        this.getBPMTasks()
    }).catch((error) =>{
        console.log("Error", error)
    })
}

fetchData() {
    if (this.selectedTask) {       
        this.task = this.getTaskFromList(this.tasks, this.selectedTask);
        this.getGroupDetails()
        CamundaRest.getTaskById(this.token, this.selectedTask, this.bpmApiUrl).then((result) => {
            CamundaRest.getProcessDefinitionById(this.token, result.data.processDefinitionId, this.bpmApiUrl).then((res) => {
                this.taskProcess = res.data.name;
            });
        })
        this.showfrom = false
        CamundaRest.getVariablesByTaskId(this.token, this.selectedTask, this.bpmApiUrl).then((result)=> {
            this.applicationId = result.data["applicationId"].value;
            this.formioUrl = result.data["formUrl"].value;           
            const {formioUrl, formId, submissionId} = getFormDetails(this.formioUrl, this.formIOApiUrl);
            this.formioUrl = formioUrl; this.submissionId = submissionId; this.formId = formId
            this.showfrom = true
        });
    }
}

togglefilter(index: number) {
    this.activefilter = index
}

findFilterKeyOfAllTask(array: string|any[], key: string|number, value: any) {
    for (let i = 0; i < array.length; i++) {
        if (array[i][key] === value) {
            return array[i]["id"];
        }
    }
    return null;
}

addGroup() {
    CamundaRest.createTaskGroupByID(this.token, this.task.id, this.bpmApiUrl, {"userId": null, "groupId": this.setGroup, "type": "candidate"}).then((result) => {
        console.log("Create group", result.data);
        this.getGroupDetails();
        this.getBPMTaskDetail(this.task.id);
        this.getBPMTasks();
    })
}

getGroupDetails() {
    CamundaRest.getTaskGroupByID(this.token, this.task.id, this.bpmApiUrl).then((response) => {
        this.groupList = response.data;
        this.groupListItems = []
        this.groupListNames = null
        for (const group of response.data){
            this.groupListItems.push(group.groupId)
        }
        if (this.groupListItems.length) {
            this.groupListNames = this.groupListItems
        }
    })
}

deleteGroup(groupid: string) {
    CamundaRest.deleteTaskGroupByID(this.token, this.task.id, this.bpmApiUrl, {"groupId": groupid, "type": "candidate"}).then(()=> {
        this.getGroupDetails();
        this.getBPMTaskDetail(this.task.id);
    })
}

submitFunctionality() {
    console.log("Form submitted")
    this.getBPMTaskDetail(this.task.id)
}

created() {
    CamundaRest.filterList(this.token, this.bpmApiUrl).then((response) => {
        this.filterList = response.data;
        const key = this.findFilterKeyOfAllTask(this.filterList, "name", "All tasks")
        this.fetchTaskList(key)
    });
}

mounted() {
    this.checkPropsIsPassed();
    localStorage.setItem("bpmApiUrl", this.bpmApiUrl);
    localStorage.setItem("authToken", this.token);
    localStorage.setItem("formsflow.ai.url", this.formsflowaiUrl);
    localStorage.setItem("formsflow.ai.api.url", this.formsflowaiApiUrl);
    const decodeToken = JSON.parse(atob(this.token.split('.')[1]))
    this.userName = !this.userName ? decodeToken["preferred_username"] : this.userName
    this.userEmail = decodeToken["email"] || "external"
    // this.formIOUserRoles = String(decodeToken["resource_access"][decodeToken["aud"][0]]["roles"])
    localStorage.setItem("UserDetails", decodeToken);
    authenticateFormio(this.formIOResourceId, this.formIOReviewerId, this.formIOReviewer,this.userEmail, this.formIOUserRoles)

    this.fetchData();
    CamundaRest.getProcessDefinitions(this.token, this.bpmApiUrl).then((response) => {
        this.getProcessDefinitions = response.data;
    });
}
}
</script>
