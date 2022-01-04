<template lang="">
  <q-dialog v-model="PRshow" persistent>
    <div v-if="loading">
      <q-spinner-gears class="loadingP" color="primary" size="3em" />
    </div>
    <q-card v-else class="my-card">
      <div class="text-center justify-center q-col-gutter-sm">
        <div class="col-lg-12 col-md-12 col-xs-12 col-sm-12">
          <q-card>
            <q-card-section class="q-pa-sm">
              <q-list class="row">
                <q-item class="col-lg-12 col-md-12 col-sm-12 col-xs-12">
                  <q-item-section side>
                    <q-avatar size="100px">
                      <img :src="src" />
                    </q-avatar>
                  </q-item-section>
                  <q-item-section>
                    <q-file
                      v-model="newavatar"
                      color="teal"
                      filled
                      accept=".png"
                      :label="$t('company.create.avatar')"
                    />
                  </q-item-section>
                </q-item>
                <q-item class="col-lg-12 col-md-12 col-sm-12 col-xs-12">
                  <q-item-section>
                    <q-input
                      dir="auto"
                      color="teal"
                      filled
                      v-model="companyModel.name"
                      :label="$t('company.create.name')"
                    />
                  </q-item-section>
                </q-item>
              </q-list>
            </q-card-section>
            <q-card-actions align="center">
              <q-btn
                class="bg-teal text-white"
                :label="$t('company.edit.edit')"
                type="button"
                color="primary"
                :loading="loading"
                @click="editeHandler"
              />

              <q-btn
                class="bg-red text-white"
                :label="$t('company.edit.close')"
                type="button"
                @click="$emit('close')"
              >
              </q-btn>
            </q-card-actions>
          </q-card>
        </div>
      </div>
    </q-card>
  </q-dialog>
</template>
<script>
import { useQuasar } from "quasar";
import { defineComponent, toRef, watch, ref, computed } from "vue";
import { useStore } from "vuex";
export default defineComponent({
  props: ["id", "show"],
  setup(props, { emit }) {
    const PRshow = toRef(props, "show");
    const PRid = toRef(props, "id");
    const loading = ref(false);
    const store = useStore();
    const avatar = ref("");
    const $q = useQuasar();
    const newavatar = ref("");
    const src = computed(() => {
      if (newavatar.value) {
        return (window.URL ? URL : webkitURL).createObjectURL(newavatar.value);
      } else {
        return companyModel.value.avatar;
      }
    });

    const companyModel = ref({
      name: "",
      avatar: "",
      id: "",
    });

    function fetchComp() {
      store
        .dispatch("company/fetchCompany", PRid.value)
        .then((response) => {
          if (response.status == 200) {
            const oldData = response.data;

            companyModel.value = {
              name: oldData.name,
              avatar: oldData.avatar,
              id: PRid.value,
            };
          }

          companyModel_.value = Object.assign({}, companyModel.value);
        })
        .finally(() => {
          loading.value = false;
        })
        .catch((err) => {
          $q.notify({
            message: "خطا در برقراری ارتباط.",
            type: "negative",
          });
        });
    }
    const companyModel_ = ref([]);
    watch(PRshow, (newvalue, oldValue) => {
      if (newvalue) {
        loading.value = true;
        fetchComp();
      }
    });
    function editeHandler() {
      loading.value = true;
      let filteredData = {};
      Object.entries(companyModel.value).forEach((item) => {
        if (companyModel.value[item[0]] != companyModel_.value[item[0]]) {
          filteredData[item[0]] = item[1];
        }
      });
      if (newavatar.value != "") {
        filteredData["id"] = companyModel.value.id;
      }
      if (Object.keys(filteredData).length > 0) {
        filteredData["id"] = companyModel.value.id;
        if (newavatar.value) {
          let reader = new FileReader();

          reader.onloadend = function () {
            filteredData.avatar = reader.result.replace(
              "data:image/png;base64,",
              ""
            );
            store
              .dispatch("company/editCompany", filteredData)
              .then((response) => {
                emit("edit");
              })
              .finally(() => {
                loading.value = false;
              })
              .catch((error) => {
                emit("error");
              });
          };
          reader.readAsDataURL(newavatar.value);
        } else {
          store
            .dispatch("company/editCompany", filteredData)
            .then((response) => {
              emit("edit");
            })
            .finally(() => {
              loading.value = false;
            })
            .catch((error) => {
              emit("error");
            });
        }
      }
    }
    return {
      companyModel,
      PRshow,
      PRid,
      loading,
      editeHandler,
      newavatar,
      src,
      companyModel_,
    };
  },
});
</script>
<style lang=""></style>
