<template lang="">
  <div>
    <CCardBody>
      <CDataTable
        :items="dataCandidate"
        :fields="fields"
        :items-per-page="5"
        pagination
      >
        <template #position="{item}">
          <td>
            {{ getPosition(item.position) }}
          </td>
        </template>
        <template #dateTime="{item}">
          <td>
            <CInput
              type="datetime-local"
              name="dateTime"
              v-model="item.dateTime"
            />
          </td>
        </template>
        <template #show="{item}">
          <td>
            <CButton
              color="primary"
              variant="outline"
              size="sm"
              @click="showModal(item)"
            >
              Show
            </CButton>
          </td>
        </template>
        <template #status="{item}">
          <td>
            {{ getStatus(item.status) }}
          </td>
        </template>
        <template #created_at="{item}">
          <td>
            {{ convertDate(item.created_at) }}
          </td>
        </template>
        <template #Send="{item}"> 
          <td>
            <CSelect :options="dataMailIntern" :value.sync="item.category_mail" @change="selectMail(item)" />
          </td>
        </template>
      </CDataTable>
    </CCardBody>
    <CCardFooter class="button-center">
      <CButton color="success" class="m-2" size="" @click="sendMail()">
        Send
      </CButton>
    </CCardFooter>
    <CModal
      title="Detail Content Mail"
      color="success"
      :show.sync="warningModal"
    >
      <div class="content-mail">
        {{ content }}
      </div>
    </CModal>
    <CModal title="Success" color="success" :show.sync="warningModal1">
      <div class="content-mail">
        <p>Send mail success</p>
      </div>
    </CModal>
    <div v-if="showLoading">
      <CElementCover
        :boundaries="[{ sides: ['top', 'left'], query: '.media-body' }]"
        :opacity="0.8"
      >
      <h1 class="d-inline">Loading...</h1>
      <CSpinner size="5xl" color="success" />
    </CElementCover>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import moment from "moment";
import { LIST_POSITION } from "@/const/constdata";
import { LIST_STATUS } from "@/const/constdata";

const fields = [
  { key: "name", label: "Name" },
  { key: "phone", label: "Phone" },
  { key: "email", label: "Email" },
  { key: "position", label: "Position" },
  { key: "status", label: "Status" },
  { key: "created_at", label: "Day Reception" },
  {
    key: "dateTime",
    label: "interview schedule",
  },
  {
    key: 'Send',
    label: ''
  },
  {
    key: "show",
    label: "",
  },
  
];

export default {

  props: ['dataCandidate'],

  name: "AdvancedTables",
  data() {
    return {
      fields: fields,
      dataSend: [],
      dateTime: null,
      dataMailIntern: "",
      LIST_POSITION,
      LIST_STATUS,
      warningModal: false,
      content: "",
      showLoading: false,
      warningModal1: false,
    };
  },
  
  mounted() {
    this.getMailIntern();
  },

  methods: {

    getMailIntern: function() {
      const url = "http://127.0.0.1:8000/api/getMailIntern";
      axios.get(url).then((response) => {
        this.dataMailIntern = response.data;
        this.dataMailIntern.map(function(value,key) {
          value['value'] = value['id']
          value['label'] = value['name']
        });
        this.dataMailIntern.unshift({value:0, label:"----Choose mail ----" })
      });
    },

    // sendMail: function() {
    //   this.errors = [];
    //   for (const [key, value] of Object.entries(this.dataSend)) {
    //     value["template_id"] = 2;
    //     value["candidate_email"] = value.email
    //     value["candidate_id"] = value.id
    //     value["status"] = value.status
    //     value["content"] = this.changeText(
    //       this.getContentMailIntern(value.category_mail),
    //       value["name"],
    //       value["dateTime"],
    //       this.getPosition(value["position"])
    //     );
    //     value["content"] = value["content"].replace(/\n/ig, "\n")
    //     value["datetime_interview"] = value.dateTime;

    //     if (!value.dateTime) {
    //       this.errors.push('Phải nhập đầy đủ dữ liệu')
    //     } else {
    //       axios.post("http://127.0.0.1:8000/api/send-mailIntern", value)
    //     }
    //   }
    // },

    sendMail: function() {
      this.errors = [];
      for (const [key, value] of Object.entries(this.dataSend)) {
        value["template_id"] = 2;
        value["candidate_email"] = value.email
        value["candidate_id"] = value.id
        value["status"] = value.status
        value["content"] = this.changeText(
          this.getContentMailIntern(value.category_mail),
          value["name"],
          value["dateTime"],
          this.getPosition(value["position"])
        );
        value["content"] = value["content"].replace(/\n/gi, "\n");
        value["datetime_interview"] = value.dateTime;

        if (!value.dateTime) {
          this.errors.push("Phải nhập đầy đủ dữ liệu");
        } else {
          this.showLoading = true;
          axios
            .post("http://127.0.0.1:8000/api/send-mailIntern", value)
            .then(() => {
              this.warningModal1 = true;
              this.showLoading = false;
            });
        }
      }
    },

    check: function(item) {
      this.dataSend.push(item);
    },

    getPosition(id) {
      for (const position of this.LIST_POSITION) {
        if (id == position.value) {
          return position.label;
        }
      }
    },

    changeText: function(content, name, dateTime, position) {
      if (!dateTime) {
        return content;
      } else {
        dateTime = moment(String(dateTime)).format("DD/MM/YYYY hh:mm");
        content = content
          .replace("[Name]", name)
          .replace("[dateTime]", dateTime)
          .replace("[Position]", position);
        return content;
      }
    },

    getStatus(id) {
      for (const status of this.LIST_STATUS) {
        if (id == status.value) {
          return status.label;
        }
      }
    },

    convertDate(created) {
      created = moment(String(created)).format("DD/MM/YYYY");
      return created;
    },

    showModal(item) {
      this.dataSend.forEach((element) => {
        if(item.id == element.id){
          this.content = this.changeText(this.getContentMailIntern(element.category_mail), item.name, item.dateTime, this.getPosition(item.position));
          this.warningModal = true
        }
      })
    },

    getContentMailIntern(id){
      return this.dataMailIntern.find((element) => element.value === id).content;
    },

    selectMail(item){
      var index = this.dataSend.findIndex((element) => element.id == item.id);
      index == -1 ? this.dataSend.push(item) : (item.category_mail == 0 ? this.dataSend.splice(index, 1) : (this.dataSend[index] = item))
    },

    
  },
};
</script>
