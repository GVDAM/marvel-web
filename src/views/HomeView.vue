<template>
  <div>
    <q-page-container>      
      <q-table
      flat
      :loading="loading"      
      :columns="columns"
      :rows="heroes"
      row-key="id"
      v-model:pagination="pagination"
      :rows-per-page-options="[]"
      @request="handleRequest"
      >
      <template #loading>
        <q-inner-loading
        showing
        color="red"
        >

        </q-inner-loading>
      </template>

      <template v-slot:body="props">
        <q-tr :props="props">
          <q-td
            v-for="col in props.cols"
            :key="col.name"
            :props="props"
          >
            <span v-if="col.name !='image'" >
              {{ col.value }}</span>
            
             <q-avatar v-if="col.name =='image'" size="100px" class="shadow-10">
              <img :src="props.row.thumbnail">
            </q-avatar>

            <div v-if="col.name =='action'">
              <q-btn-group outline >
                <q-btn outline @click="favHero(props.row.id, props.row.isFav)">
                  <q-icon v-if="props.row.isFav" name="img:../../img/icons/favorite_FILL0_wght400_GRAD0_opsz24.png" />
                  <q-icon v-else name="img:../../img/icons/heart_plus_FILL0_wght400_GRAD0_opsz24.png" />
                </q-btn>
                <q-btn outline color="brown" @click="openmodel(props.row)" icon="img:../../img/icons/visibility_FILL0_wght400_GRAD0_opsz24.png">
                  <span>
                  </span>
                </q-btn>
              </q-btn-group>
            </div>            
          </q-td>
        </q-tr>          
      </template>
    </q-table>    

    <q-dialog v-model="openDetails">
      <q-card>
        <q-toolbar>
          <q-avatar>
            <img :src="thumbnail">
          </q-avatar>
      
          <q-toolbar-title><span class="text-weight-bold">{{ name }}</span></q-toolbar-title>

          <q-btn flat round dense icon="X" v-close-popup />
        </q-toolbar>

        <q-card-section>
          {{description}}
        </q-card-section>
      </q-card>
    </q-dialog>
    </q-page-container>    
  </div>
</template>

<script>
import axios from 'axios';
import { onMounted, ref } from 'vue';
import { useQuasar } from 'quasar'

const columns = [
      { name: 'image', align: 'center', label: 'Image', field: 'thumbnail', sortable: true, style: 'width: 10px' },
      { name: 'id', required: true, label: 'ID', align: 'center', field: row => row.id, sortable: true },
      { name: 'name', align: 'center', label: 'Name', field: 'name', sortable: true },
      { name: "action", align: "center", label: "Action", field: "" }
    ];

export default {
  name: 'HomeView',

  components: {
  },

  setup () {    
    const heroes = ref([]);
    const loading = ref(true);
    const openDetails = ref(false);
    const selectedhero = ref({});
    const description = ref("");
    const name = ref("");
    const thumbnail = ref("");
    const pagination = ref({
      page: 1,
      rowsPerPage: 5,
      rowsNumber: 0
    });
    const $q = useQuasar()

    const openmodel = (row) => {
      selectedhero.value = row;
      description.value = row.description;
      name.value = row.name;
      thumbnail.value = row.thumbnail;
      openDetails.value=true;
    }

    const favHero = (id, isFav) => {
      if (isFav){
        deleteFavCharacter(id);
      }
      else{
        addFavCharacter(id);
      }
    };

    const deleteFavCharacter = async (id) => {
      loading.value = true;

      await axios.delete(
        'https://localhost:7060/api/Characters/'+id)
        .then(() => {
          console.log("Character Deleted");
        })
        .catch((error) => {
          console.log("Error: ", error)
        })
        .finally(() => {
          fetchCharacters();
          loading.value = false;
        });      
    };

    const addFavCharacter = async (id) => {
      loading.value = true;
      await axios.post(
        'https://localhost:7060/api/Characters/'+id)
        .then(() => {
          console.log("Character Favorite");
        })
        .catch((error) => {
          console.log("Error: ", error);
          $q.notify({
            type: 'negative',
            message: error.response.data.toString(),
          })
        })
        .finally(() => {
          fetchCharacters();
          loading.value = false;
        });      
    };

    const fetchCharacters = async (skip = 0) => {
      loading.value = true;
      await axios.get('https://localhost:7060/api/Characters',
      {
        params: { skip: skip }
      })
        .then((response) => {
          console.log(response);
          heroes.value = response.data.results;

          const data = response.data;

          pagination.value.page = data.currentPage;
          pagination.value.rowsNumber = data.total;

          console.log(pagination);
        })
        .catch((error) => {
          console.log(error)
        })
        .finally(() => {
          loading.value = false;
        })
    };

    onMounted(() => {
      fetchCharacters();
    });   

    const handleRequest = (props) => {
      if (props != undefined || props != null)
        fetchCharacters((props.pagination.page-1) * 5);
    }

    return {
      handleRequest,
      columns,
      heroes,
      loading,
      pagination,
      openDetails,
      openmodel,
      description,
      name,
      thumbnail,
      favHero
    }
  }
}
</script>
