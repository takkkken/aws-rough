<template>
  <article class="service">
    <ServiceTemplateTitle :service="service" />
    <div class="service-calc">
      <table :class="['table', {'mod-multiple': service.multiple }]">
        <thead :class="{[`mod-${service.mod}`]: service.mod}">
          <tr>
            <th
              v-for="column in service.table"
              :key="`header-${column.key}`"
              :class="[{[`mod-${column.mod}`]: column.mod}]"
            >{{ column.title }}</th>
            <th class="mod-price">月額</th>
          </tr>
        </thead>
        <tbody>
          <ServiceTemplateRow
            v-for="(row, rowIndex) in table"
            :key="rowIndex"
            :row="row"
            :row-index="rowIndex"
            :labels="tableLabels"
            :service="service"
          />
        </tbody>
      </table>
      <div class="service-calc-footer" v-if="service.multiple">
        <button
          class="service-calc-append"
          @click="append(service.key)"
        ><span>追加する</span></button>
      </div>
    </div>

    <slot />
  </article>
</template>

<script>
import ServiceTemplateRow from '@/components/service/template/ServiceTemplateRow'
import ServiceTemplateTitle from '@/components/service/template/ServiceTemplateTitle'
import serviceConfig from '@/config/service'
import store from '@/store'
import { getService } from '@/store/service'

export default {
  name: 'ServiceTemplate',
  components: { ServiceTemplateRow, ServiceTemplateTitle },
  props: {
    serviceName: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      table: store.state.tables[this.serviceName]
    }
  },
  computed: {
    service() {
      const service = getService(this.serviceName, serviceConfig)
      const table = service.table.map(row => {
        if (store.state.price && row.parseJson) {
          return {
            ...row,
            options: row.parseJson(store.state.price)
          }
        }

        return row
      })

      return { ...service, table }
    },
    tableLabels() {
      return this.service.table.map(column => column.title)
    }
  },
  methods: {
    append(serviceKey) {
      store.append({ serviceKey }, serviceConfig)
    }
  }
}
</script>
