<template lang="">
  <div>
    <CCard>
      <CCardHeader>
        <p class="center">Send Mail Thank</p>
      </CCardHeader>
      <search-mail @set-condition="getData"></search-mail>
      <mail-thank :dataCandidate="dataCandidate"></mail-thank>
    </CCard>
  </div>
</template>

<script>
import axios from "axios";
import moment from "moment";
import { LIST_STATUS } from "@/const/constdata";
import MailThank from "@/components/sendmail/MailThank";
import SearchMail from "@/components/sendmail/SearchMail";
const fields = [
  { key: "name", label: "Name" },
  { key: "phone", label: "Phone" },
  { key: "email", label: "Email" },
  { key: "status", label: "Status" },
  { key: "created_at", label: "Day Reception" },
  {
    key: "send",
  },
  {
    key: "show",
    label: "",
  },
];

export default {
  name: "AdvancedTables",
  components: { MailThank, SearchMail },
  data() {
    return {
      dataCandidate: [],
      fields: fields,
      dataMailThank: "",
      dataSend: [],
      warningModal: false,
      LIST_STATUS,
      content: "",
      condition: {},
    };
  },

  created() {
    this.listData();
  },

  methods: {
    listData: function() {
      axios
        .get("http://127.0.0.1:8000/api/candidate?status=1", { params: this.condition })
        .then((response) => {
          this.dataCandidate = response.data;
        });
    },
    
    getData: function(value) {
      this.condition = value;
      this.listData();
    },
  },
};
</script>