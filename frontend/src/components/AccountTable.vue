<template>
  <div class="accountTable">
    <p style="letter-spacing: 21px;">FEBRUARY</p>
    <p style="text-decoration: underline; text-underline-position:under; font-size: 18px;">2022</p>
    <SearchBar
      @getSearching="searching"
      @all="getAll"
    />
    <div class="tableButton">
      <div/>
      <button id="new" @click.stop="onNew">New</button>
      <button id="delete" @click="onDelete">Delete</button>
    </div>
    <table id="accountBook">
      <thead>
        <tr>
          <th class="th-1">Date</th>
          <th class="th-2">Description</th>
          <th class="th-3">tag</th>
          <th class="th-4">Amount</th>
          <th class="th-5">Total</th>
          <th v-show="showDelete">delete</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(list, index) in lists" :key="index">
          <td>{{ list.consumptionDatetime }}</td>
          <td>{{ list.content }}</td>
          <td>{{ this.types.payType[list.payType] }}, {{ this.types.dwType[list.dwType] }}, {{ this.types.useType[list.useType] }}</td>
          <td>{{ list.cost }}</td>
          <td>{{ this.countTotal(list.cost, list.dwType) }}</td>
          <td><button type="button" v-show="showDelete" @click="deleteList(index)">Delete</button></td>
        </tr> 
      </tbody>
    </table>
    <Modal 
      v-if = "showNewModal"
      @close="closeNewModal"
      @insert="inputUpdate"
    />
  </div>
</template>

<script>
import { postConsumption, getConsumptions, deleteConsumption, getSearch } from "@/api/index";
import Modal from "@/components/newModal.vue";
import SearchBar from "@/components/SearchBar.vue";
export default {
  components: {
    Modal,
    SearchBar
  },
  mounted() {
    this.getLists();
  },
  methods: {
    getAll() {
      this.getLists();
    },
    async searching(data) {
      const searchitem = {
        searchUseType: data.searchUseType,
        searchPayType: data.searchPayType,
        searchDwType: data.searchDwType,
      }
      if(data.searchUseType==undefined && data.searchPayType==undefined && data.searchDwType==undefined) {
        alert("안돼애!!")
      } else if(data.searchUseType!=undefined && data.searchPayType==undefined && data.searchDwType==undefined) {
        this.searchitems = "useType=" + data.searchUseType;
      } else if(data.searchUseType==undefined && data.searchPayType!=undefined && data.searchDwType==undefined) {
        this.searchitems = "payType=" + data.searchPayType;
      } else if(data.searchUseType==undefined && data.searchPayType==undefined && data.searchDwType!=undefined) {
        this.searchitems = "dwType=" + data.searchDwType;
      } else if(data.searchUseType!=undefined && data.searchPayType!=undefined && data.searchDwType==undefined) {
        this.searchitems = "useType=" + data.searchUseType + "&payType=" + data.searchPayType;
      } else if(data.searchUseType==undefined && data.searchPayType!=undefined && data.searchDwType!=undefined) {
        this.searchitems = "payType=" + data.searchPayType + "&dwType=" + data.searchDwType;
      } else if(data.searchUseType!=undefined && data.searchPayType==undefined && data.searchDwType!=undefined) {
        this.searchitems = "useType=" + data.searchUseType + "&dwType=" + data.searchDwType;
      } else if(data.searchUseType!=undefined && data.searchPayType!=undefined && data.searchDwType!=undefined) {
        this.searchitems = "useType=" + data.searchUseType + "&payType=" + data.searchPayType + "&dwType=" + data.searchDwType;
      }
      const response = await getSearch(this.searchitems);
      this.lists = response.data;
      console.log(searchitem);
      console.log(data.searchUseType);
    },
    async deleteList(index) {
      const id = this.lists[index].consumptionIndex
      console.log(id)
      await deleteConsumption(id);
      this.lists.splice(index,1);
      this.showDelete = false;
    },
    onNew() {
      this.showNewModal = true;
    },
    onDelete() {
      this.showDelete = true;
    },
    closeNewModal() {
      this.showNewModal = false;
    },
    async inputUpdate(listData) {
      const userData = {
        consumptionDatetime: Number(listData.consumptionDatetime.replace(/-/g, "")),
        content: listData.content,
        useType: listData.useType,
        payType: listData.payType,
        dwType: listData.dwType,
        cost: listData.cost
      };
      console.log(userData)
      const { data } = await postConsumption(userData);
      console.log(data);
      this.lists.push({
        consumptionDatetime: Number(listData.consumptionDatetime.replace(/-/g, "")),
        content: listData.content,
        useType: listData.useType,
        payType: listData.payType,
        dwType: listData.dwType,
        cost: listData.cost,
        total: this.countTotal(listData.cost, listData.dwType)
      })
      console.log(this.lists);
      this.consumptionDatetime = ""
      this.content = ""
      this.useType = ""
      this.payType = ""
      this.dwType = ""
      this.cost = ""
      this.total = ""

      this.closeNewModal()
    },
    countTotal(cost, dwType) {
      let newAmount = 0;
      if(dwType === "WITHDRAW") {
        newAmount = -cost;
      } else if(dwType === "DEPOSIT") {
        newAmount = cost;
      }
      if (this.lists.length === 0) {
        return newAmount;
      } else {
        return this.lists[this.lists.length - 1].total + newAmount;
      }      
    },
    async getLists() {
      const response = await getConsumptions();
      // response.data.push({
      //   total: this.countTotal(response.cost, response.dwType)
      // })
      this.lists = response.data;
      console.log(this.lists);
      console.log(response.data);
    },
  },
  data() {
    return {
      newResponse: [],
      searchitems: "",
      searchUseType: "",
      searchPayType: "",
      searchDwType: "",
      showDelete: false,
      showNewModal: false,
      lists: [],
      bringdata: [],
      lastTotal: 0,
      types: {
        useType: {
          "BEAUTY": "뷰티",
          "CLOTHES": "의류비",
          "CULTURE": "문화활동",
          "EDUCATION": "교육비",
          "ETC": "기타",
          "FOOD": "식비",
          "LIFE": "생활비",
          "MEDICALTREATMENT": "병원비",
          "TRAFFIC": "교통비",
        },
        payType: {
          "ACCOUNTTRANSFER": "계좌이체",
          "CARD": "카드",
          "CASH": "현금",
          "GIFTCARD": "기프트카드",
          "ETC": "기타", 
        },
        dwType: {
          "WITHDRAW": "지출",
          "DEPOSIT": "수입",
        },
      }
    };
  }
}
</script>

<style scoped>
  p {
    text-align: center;
    font-family: 'pinokio';
    font-size: 20px;
    /* font-family:'Times New Roman', Times, serif; */
  }
  .accountTable {
    margin-bottom: 147px;
  }
  .tableButton {
    display: grid;
    grid-template-columns: 66% 60px 60px ;
    grid-gap: 5px;
    margin-bottom: 5px;
  }
 .th-1 {
    width: 16%
  }
  .th-2 {
    width: 39%;
  }
  .th-3 {
    width: 25%;
  }
  .th-4, .th-5 {
    width: 10%;
  }
  button {
    width: 55px;
    padding: 4px;
    margin: 5px;
    border-radius: 35px;
    border: none;
    color: white;
  }
  #new {
    background-color: rgb(255, 160, 160);
  }
  #new:hover {
    background-color: rgb(241, 91, 91);
  }
  #delete {
    background-color: rgb(100, 170, 235);
  }
  #delete:hover {
    background-color: rgb(50, 134, 212);
  }
  table {
    width: 55%;
    margin: auto;
    border-top: solid 2px black;
    border-collapse: collapse;
    text-align: center;
  }
  th, td {
    height: 20px;
    border-bottom: solid 1px black;
    border-left: solid 1px black;
    padding: 5px;
    font-size: 15px;
  }
  th:first-child, td:first-child {
    border-left: none;
  }
  th {
    border-bottom: solid 2px black;
  }
</style>