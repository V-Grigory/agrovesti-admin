<template>
  <v-card>

    <v-card-title>
      Поиск (по любому столбцу)
      <v-spacer></v-spacer>
      <v-text-field
        v-model="search"
        append-icon="search"
        label="Search"
        single-line
        hide-details
      ></v-text-field>
    </v-card-title>

    <v-toolbar flat color="white">
      <v-toolbar-title>Подписчики</v-toolbar-title>
      <v-divider
        class="mx-2"
        inset
        vertical
      ></v-divider>
      <v-spacer></v-spacer>

      <!-- МОДАЛКА ПРАВКИ / ДОБАВЛЕНИЯ -->
      <v-dialog v-model="dialog">
        <template v-slot:activator="{ on }">
          <v-btn color="primary" dark class="mb-2" v-on="on">Добавить</v-btn>
        </template>
        <v-card>
          <v-card-title>
            <span class="headline">{{ formTitle }}</span>
          </v-card-title>

          <v-card-text>
            <v-container grid-list-md>
              <v-layout wrap>
                <v-flex
                  v-for="item in computedCardItems"
                  :key="item.value"
                  xs12 sm4 md2>
                  <v-text-field
                    v-if="item.type === 'input'"
                    v-model="editedItem[item.value]"
                    :label="item.text"
                  ></v-text-field>
                  <v-select
                    v-else
                    :items="selectValuesInEditItem(item.value)"
                    v-model="editedItem[item.value]"
                    :label="item.text"
                  ></v-select>
                </v-flex>
              </v-layout>
            </v-container>
          </v-card-text>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" flat @click="close">Отмена</v-btn>
            <v-btn color="blue darken-1" flat @click="save">Сохранить</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-toolbar>

    <!-- ТАБЛИЦА -->
    <v-data-table
      v-model="selected"
      :headers="computedTableItems"
      :items="computedClients"
      item-key="id"
      select-all
      :search="search"
      class="elevation-1"
    >
      <template v-slot:items="props">
        <td>
          <v-checkbox
            v-model="props.selected"
            primary hide-details
          ></v-checkbox>
        </td>
        <!-- start тело таблицы -->
        <td
          v-for="itemHeader in computedTableItems"
          :key="itemHeader.value"
          class="text-xs-left tdBreakWords"
        >
          {{ props.item[itemHeader.value] }}
        </td>
        <!-- end тело таблицы -->
        <td class="justify-center layout px-0">
          <v-icon small class="mr-2"
            @click="editItem(props.item)"
          >edit</v-icon>
          <v-icon small
            @click="deleteItem(props.item)"
          >delete</v-icon>
        </td>
      </template>

      <template v-slot:no-data>
        <v-btn color="primary" @click="initialize">Reset</v-btn>
      </template>

      <template v-slot:no-results>
        <v-alert :value="true" color="error" icon="warning">
          По запросу "{{ search }}" ничего не найдено.
        </v-alert>
      </template>

    </v-data-table>
  </v-card>
</template>

<script>
import axios from 'axios'
// import { status_activity, type, status_pay } from '../okrugaRegions'

export default {
  name: 'Subscribers',
  data: () => ({
    lib: {
      okrugaRegions: {
        'Не_выбрано': {
          'name': 'Не_выбрано',
          'regions': ['Не_выбрано']
        },
        'Центральный': {
          'name': 'Центральный',
          'regions': [
            'Белгородский', 'Брянский', 'Владимирский', 'Воронежский', 'Ивановский',
            'Калужский', 'Костромской', 'Курский', 'Липецкий', 'Московский',
            'Орловский', 'Рязанский', 'Смоленский', 'Тамбовский', 'Тверской',
            'Тульский', 'Ярославский', 'г. Москва'
          ]
        },
        'Северо-Западный': {
          'name': 'Северо-Западный',
          'regions': [
            'р.Карелия', 'р.Коми', 'Архангельский', 'Ненецкий АО',
            'Вологодский', 'Калининградский', 'Ленинградский', 'Мурманский',
            'Новгородский', 'Псковский', 'г.Санкт-Петербург'
          ]
        },
        'Южный': {
          'name': 'Южный',
          'regions': [
            'р.Адыгея', 'р.Дагестан', 'р.Ингушетия', 'Кабардино-Балкарская р.',
            'р.Калмыкия', 'Карачаево-Черкесская р.', 'р.Северная Осетия-Алания',
            'Чеченская р.', 'Краснодарский край', 'Ставропольский край',
            'Астраханский', 'Волгоградский', 'Ростовский'
          ]
        },
        'Приволжский': {
          'name': 'Приволжский',
          'regions': [
            'р.Башкортостан', 'р.Марий Эл', 'р.Мордовия',
            'р.Татарстан', 'Удмуртская р.', 'Чувашская р.', 'Кировский',
            'Нижегородский', 'Оренбургский', 'Пензенский', 'Пермский',
            'Коми-Пермяцкий АО', 'Самарский', 'Саратовский', 'Ульяновский'
          ]
        },
        'Уральский': {
          'name': 'Уральский',
          'regions': [
            'Курганский', 'Свердловский', 'Тюменский', 'Ханты-Мансийский АО',
            'Ямало-Ненецкий АО', 'Челябинский'
          ]
        },
        'Сибирский': {
          'name': 'Сибирский',
          'regions': [
            'р.Алтай', 'р.Бурятия', 'р.Тыва', 'р.Хакасия', 'Алтайский край',
            'Красноярский край', 'Таймырский АО', 'Эвенкийский АО', 'Иркутский',
            'Усть-Ордынский АО', 'Кемеровский', 'Новосибирский', 'Омский',
            'Томский', 'Читинский', 'Агинский Бурятский АО'
          ]
        },
        'Дальневосточный': {
          'name': 'Дальневосточный',
          'regions': [
            'р.Саха (Якутия)', 'Приморский край', 'Хабаровский край',
            'Амурский', 'Камчатский', 'Корякский АО', 'Магаданский', 'Сахалинский',
            'Еврейская АО', 'Чукотский АО'
          ]
        }
      },
      status_activity: [
        'Новый клиент', 'Пробный период', 'Активен', 'Заблокирован'
      ],
      type: ['Подписчик', 'Производитель', 'Трейдер'],
      status_pay: ['Оплачено', 'Не оплачено']
    },
    search: '',
    selected: [],
    dialog: false,
    clients: [],
    templateClients: [
      {
        text: 'Дата рег-ции',
        value: 'created_at',
        type: 'input',
        isShowInCard: false,
        isShowInTable: true,
        sortable: false
      },
      {
        text: 'Фед.окр->Рег',
        value: 'okrugRegion',
        isShowInCard: false,
        isShowInTable: true
      },
      {
        text: 'ФИО',
        value: 'fio',
        isShowInCard: false,
        isShowInTable: true
      },
      {
        text: 'Фед.окр',
        value: 'fed_okrug',
        type: 'select',
        isShowInCard: true,
        isShowInTable: false
      },
      {
        text: 'Регион',
        value: 'region',
        type: 'select',
        isShowInCard: true,
        isShowInTable: false
      },
      {
        text: 'Тип клиента',
        value: 'type',
        type: 'select',
        isShowInCard: true,
        isShowInTable: false
      },
      {
        text: 'Фамилия',
        value: 'f_name',
        type: 'input',
        isShowInCard: true,
        isShowInTable: false
      },
      {
        text: 'Имя',
        value: 'i_name',
        type: 'input',
        isShowInCard: true,
        isShowInTable: false
      },
      {
        text: 'Отчество',
        value: 'o_name',
        type: 'input',
        isShowInCard: true,
        isShowInTable: false
      },
      {
        text: 'Телефон',
        value: 'phone',
        type: 'input',
        isShowInCard: true,
        isShowInTable: true
      },
      {
        text: 'Email',
        value: 'email',
        type: 'input',
        isShowInCard: true,
        isShowInTable: true
      },
      {
        text: 'Компания',
        value: 'company',
        type: 'input',
        isShowInCard: true,
        isShowInTable: true
      },
      {
        text: 'Оплата',
        value: 'status_pay',
        type: 'select',
        isShowInCard: true,
        isShowInTable: true
      },
      {
        text: 'Период',
        value: 'range_pay',
        type: 'input',
        isShowInCard: true,
        isShowInTable: true
      },
      {
        text: 'Статус',
        value: 'status_activity',
        type: 'select',
        isShowInCard: true,
        isShowInTable: true
      }
    ],
    editedIndex: -1,
    editedItem: {},
    preserveItem: {}
  }),

  computed: {
    formTitle () {
      return this.editedIndex === -1 ? 'Добавить' : 'Редактировать'
    },
    computedClients () {
      let out = []
      this.clients.forEach(v => {
        v['okrugRegion'] = `${v.fed_okrug}->${v.region}`
        v['fio'] = `${v.f_name} ${v.i_name} ${v.o_name}`
        out.push(v)
      })
      // return out.reverse()
      return out
    },
    computedTableItems () {
      return this.templateClients.filter(v => v.isShowInTable)
    },
    computedCardItems () {
      return this.templateClients.filter(v => v.isShowInCard)
    },
    computedRegionsForCard () {
      return this.editedItem.fed_okrug
        ? this.lib.okrugaRegions[this.editedItem.fed_okrug].regions
        : []
    }
  },

  watch: {
    dialog (val) {
      val || this.close()
    }
  },

  created () {
    this.initialize()
  },

  methods: {
    selectValuesInEditItem (v) {
      if (v === 'fed_okrug') {
        return Object.keys(this.lib.okrugaRegions)
      }
      if (v === 'region') {
        return this.computedRegionsForCard
      }
      return Object.values(this.lib[v])
    },

    initialize () {
      axios.get('http://localhost:8081/clientsJSON')
        .then(res => {
          this.clients = res.data.clients
          // console.log(this.clients)
        })
        .catch(function (error) {
          console.log(error)
        })
    },

    editItem (item) {
      this.editedIndex = this.clients.indexOf(item)
      // this.editedItem = Object.assign({}, item)
      this.editedItem = Object.assign({}, this.clients[this.editedIndex])
      console.log('открыли запись на редактирование')
      console.log(this.editedItem)
      this.dialog = true
    },

    deleteItem (item) {
      if (confirm('Are you sure you want to delete this item?')) {
        let params = item
        params['action'] = 'delete'
        axios.get('http://localhost:8081/updateClients', { params })
          .then(res => {
            const index = this.clients.indexOf(item)
            this.clients.splice(index, 1)
          })
          .catch(error => {
            console.log(error)
          })
      }
    },

    close () {
      this.dialog = false
      setTimeout(() => {
        this.editedItem = Object.assign({}, this.preserveItem)
        this.editedIndex = -1
      }, 300)
    },

    save () {
      if (this.editedIndex > -1) {
        console.log('update ...')
        Object.assign(this.clients[this.editedIndex], this.editedItem)
        console.log('сохраненная запись')
        console.log(this.clients[this.editedIndex])
      } else {
        console.log('add ...')
        this.clients.push(this.editedItem)
        console.log(this.editedItem)
      }
      let params = this.editedItem
      params['action'] = 'update'
      axios.get('http://localhost:8081/updateClients', { params })
        .then(res => {
          console.log(res)
        })
        .catch(error => {
          console.log(error)
        })
      this.close()
    }
  }
}
</script>

<style scoped>
  .tdBreakWords { word-break: break-all; }
</style>
