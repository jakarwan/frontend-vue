<template>
  <div class="row">
    <div class="col-3"></div>
    <div class="col-6">
      <div class="card">
        <div class="row m-4">
          <div class="col-12">
            <div class="row">
              <div class="col-12 col-sm-6 col-md-6">
                <label class="float-start m-2">ชื่อ :</label>
                <b-form-input
                  class="mt-3"
                  v-model="firstname"
                  placeholder="ชื่อ"
                ></b-form-input>
              </div>
              <div class="col-12 col-sm-6 col-md-6">
                <label class="float-start m-2">นามสกุล :</label>
                <b-form-input
                  class="mt-3"
                  v-model="familyname"
                  placeholder="นามสกุล"
                ></b-form-input>
              </div>
              <div class="col-md-12">
                <label class="float-start m-2">หัวข้อที่ต้องการติดต่อ :</label>
                <b-form-input
                  class="mt-3 col-2"
                  v-model="title"
                  placeholder="หัวข้อที่ต้องการติดต่อ"
                ></b-form-input>
              </div>
              <div class="col-md-12">
                <label class="float-start m-2">รายละเอียด :</label>
                <b-form-textarea
                  v-model="description"
                  placeholder="รายละเอียด"
                  rows="3"
                  max-rows="6"
                ></b-form-textarea>
              </div>
            </div>
            <button class="btn btn-primary mt-3 m-2" @click="submitData()">
              บันทึก
            </button>
          </div>
        </div>
      </div>
    </div>
    <div class="col-3"></div>

    <div class="row m-4">
      <div class="col-2"></div>
      <div class="col-8">
        <div class="card">
          <b-table
          class="m-4"
            :items="rows"
            :fields="fields"
            :select-mode="selectMode"
            responsive="sm"
            ref="selectableTable"
          >
            <template #cell(index)="rows">
              {{ rows.index + 1 }}
            </template>
            <template #cell(action)="rows">
              <ul class="list-inline mb-0">
                <li class="list-inline-item">
                  <a
                    href="javascript:void(0);"
                    class="px-2 btn btn-danger"
                    v-b-tooltip.hover
                    title="Delete"
                    @click="alertConfirm(rows.item.id)"
                  >
                    ลบ
                  </a>
                </li>
              </ul>
            </template>
          </b-table>
        </div>
      </div>
      <div class="col-2"></div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Swal from "sweetalert2";

export default {
  data() {
    return {
      firstname: "",
      familyname: "",
      title: "",
      description: "",
      rows: [],

      modes: ["multi", "single", "range"],
      fields: [
        {
          key: "index",
          label: "ลำดับ",
          sortable: false,
        },
        {
          key: "firstname",
          label: "ชื่อ",
          sortable: false,
        },
        {
          key: "familyname",
          label: "นามสกุล",
          sortable: false,
        },
        {
          key: "title",
          label: "หัวข้อที่ต้องการติดต่อ",
          sortable: false,
        },
        {
          key: "description",
          label: "รายละเอียด",
          sortable: false,
        },
        {
          key: "action",
          label: "ลบ",
          sortable: false,
        },
      ],
      //   items: [
      //     {
      //       isActive: true,
      //       age: 40,
      //       first_name: "Dickerson",
      //       last_name: "Macdonald",
      //     },
      //     { isActive: false, age: 21, first_name: "Larsen", last_name: "Shaw" },
      //     { isActive: false, age: 89, first_name: "Geneva", last_name: "Wilson" },
      //     { isActive: true, age: 38, first_name: "Jami", last_name: "Carney" },
      //   ],
      selectMode: "multi",
      selected: [],
    };
  },
  mounted() {
    this.getData();
  },
  methods: {
    getData() {
      axios
        .get("http://localhost:8000/api/contacts")
        .then((response) => {
          this.rows = response.data;
        })
        .catch(function (error) {
          Swal.fire(
            "Error",
            JSON.stringify(error.response.data.message),
            "error"
          );
        });
    },
    submitData() {
      axios
        .post("http://localhost:8000/api/contact/store", {
          firstname: this.firstname != null ? this.firstname : undefined,
          familyname: this.familyname != null ? this.familyname : undefined,
          title: this.title != null ? this.title : undefined,
          description: this.description != null ? this.description : undefined,
        })
        .then(function (response) {
          if (response) {
            Swal.fire(
              "บันทึกข้อมูลสำเร็จ",
              JSON.stringify(response.data.message),
              "success"
            );
          }
        })
        .catch(function (error) {
          Swal.fire(
            "Error",
            JSON.stringify(error.response.data.message),
            "error"
          );
        })
        .then(() => {
          this.firstname = "";
          this.familyname = "";
          this.title = "";
          this.description = "";
        });
    },
    delData(id) {
      axios
        .post("http://localhost:8000/api/contact/delete", {
          id: id,
        })
        .then((response) => {
          Swal.fire(
            "บันทึกข้อมูลสำเร็จ",
            JSON.stringify(response.data.message),
            "success"
          );
          this.getData();
        })
        .catch(function (error) {
          Swal.fire(
            "Error",
            JSON.stringify(error.response.data.message),
            "error"
          );
        });
    },
    alertConfirm(id) {
      const swalWithBootstrapButtons = Swal.mixin({
        customClass: {
          confirmButton: "btn btn-success",
          cancelButton: "btn btn-danger ms-2",
        },
        buttonsStyling: false,
      });

      swalWithBootstrapButtons
        .fire({
          title: "แจ้งเตือน",
          text: "ต้องการทำรายการหรือไม่ !!",
          icon: "warning",
          confirmButtonText: "OK",
          cancelButtonText: "Cancel!",
          showCancelButton: true,
        })
        .then((result) => {
          if (result.value) {
            this.delData(id);
          } else if (
            /* Read more about handling dismissals below */
            result.dismiss === Swal.DismissReason.cancel
          ) {
            swalWithBootstrapButtons.fire(
              "Cancelled",
              "ยกเลิกเรียบร้อยแล้ว",
              "error"
            );
          }
        });
    },
    // onRowSelected(items) {
    //   this.selected = items;
    // },
  },
};
</script>

<style></style>
