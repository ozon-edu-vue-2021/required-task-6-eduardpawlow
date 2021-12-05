<script lang="jsx">
import { orderBy } from 'lodash/collection'

import DotsLoaderIcon from './dost-loader.svg'

import FilterDropdown from './FilterDropdown'
import OzTablePaginator from './OzTablePaginator'

export default {
  name: 'oz-table',
  props: {
    rows: {
      type: Array,
      default: () => [],
    },
    totalPages: {
      type: Number,
      default: 0,
    },
    currentPage: {
      type: Number,
      default: 0,
    },
    staticPaging: {
      type: Boolean,
      default: true,
    },
  },
  data() {
    return {
      sort: {}, // {prop: direction}
      filters: {}, // {prop: text}
      currentShownFilter: '',
    }
  },
  computed: {
    sortedRows() {
      let res = this.rows

      if (Object.keys(this.sort).length) {
        const sortProps = Object.keys(this.sort)
        const sortDirections = Object.values(this.sort)
        res = orderBy(res, sortProps, sortDirections)
      }

      Object.keys(this.filters).forEach((prop) => {
        res = res.filter(
          (row) => String(row[prop]).search(this.filters[prop]) > -1
        )
      })

      return res
    },
  },
  methods: {
    getNextSortDirection(direction) {
      switch (direction) {
        case 'asc':
          return 'desc'
        case 'desc':
          return ''

        default:
          return 'asc'
      }
    },
    toggleSort(prop) {
      const newDirection = this.getNextSortDirection(this.sort[prop])

      if (newDirection) {
        this.$set(this.sort, prop, newDirection)
      } else {
        this.$delete(this.sort, prop)
      }
    },
    isSortingColumn(prop) {
      return !!this.sort[prop]
    },
    isFilteringColumn(prop) {
      return !!this.filters[prop]
    },
    openFilterTooltip(prop = '') {
      this.currentShownFilter = prop
    },
    setFilterText(prop, e) {
      const newValue = e.target.value

      if (newValue) this.$set(this.filters, prop, newValue)
      else this.$delete(this.filters, prop)
    },
    renderHead(h, columnsOptions) {
      return columnsOptions.map((column) => {
        const renderedTitle = column.scopedSlots.title
          ? column.scopedSlots.title()
          : column.title
        let sortIcon = 'sort'

        if (this.isSortingColumn(column.prop)) {
          sortIcon =
            this.sort[column.prop] === 'asc'
              ? 'sort-amount-down'
              : 'sort-amount-up'
        }

        const filterText = this.filters[column.prop] || ''

        return (
          <th key={column.prop} class={this.$style.headerCell}>
            <div class={this.$style.headerCellContent}>
              <span>{renderedTitle}</span>
              <font-awesome-icon
                class={this.$style.sortIcon}
                icon={sortIcon}
                on={{ click: () => this.toggleSort(column.prop) }}
              />
              <FilterDropdown
                columnProp={column.prop}
                shown={column.prop === this.currentShownFilter}
                active={this.isFilteringColumn(column.prop)}
                filterText={filterText}
                on={{
                  openFilterTooltip: () => this.openFilterTooltip(column.prop),
                  closeFilterTooltip: () => this.openFilterTooltip(),
                  setFilterText: (e) => this.setFilterText(column.prop, e),
                }}
              />
            </div>
          </th>
        )
      })
    },
    renderRows(h, columnsOptions) {
      return this.sortedRows.map((row, index) => {
        return (
          <tr key={row.id || index}>
            {...this.renderColumns(h, row, columnsOptions)}
          </tr>
        )
      })
    },
    renderColumns(h, row, columnsOptions) {
      return columnsOptions.map((column) => {
        return (
          <td key={column.prop} class={this.$style.cell}>
            {column.scopedSlots.body
              ? column.scopedSlots.body({ row })
              : row[column.prop]}
          </td>
        )
      })
    },
    getColumnOptions() {
      return this.$slots.default
        .filter(
          (item) =>
            item.componentOptions &&
            item.componentOptions.tag === 'oz-table-column'
        )
        .map((column) =>
          Object.assign({}, column.componentOptions.propsData, {
            scopedSlots: column.data.scopedSlots || {},
          })
        )
    },
    renderInfPager() {
      const directives = [
        {
          name: 'detect-viewport',
          value: {
            callback: () => {
              console.log(12)
              this.$listeners.getPage()
            },
          },
        },
      ]

      const style = {
        background: `url("${DotsLoaderIcon}") no-repeat center`,
      }

      return <div {...{ class: this.$style.infPager, style, directives }} />
    },
  },
  render(h) {
    const { $style, totalPages, currentPage, staticPaging, $listeners } = this
    const { getPage } = $listeners

    const columnsOptions = this.getColumnOptions()
    const columnsHead = this.renderHead(h, columnsOptions)
    const rows = this.renderRows(h, columnsOptions)

    return (
      <div>
        <table class={$style.table}>
          <thead>{...columnsHead}</thead>
          <tbody>{...rows}</tbody>
        </table>

        {staticPaging ? (
          <OzTablePaginator
            totalPages={totalPages}
            currentPage={currentPage}
            on={{ getPage: getPage }}
          />
        ) : (
          this.renderInfPager()
        )}
      </div>
    )
  },
}
</script>

<style module>
.table {
  border-spacing: 0;
  margin: 8px;
  width: 100%;
}

.cell {
  text-align: left;
  border-bottom: 1px solid #c8cacc;
  padding: 1rem 1rem;
}

.headerCell {
  composes: cell;
  background: #c7cbcb;
}

.headerCellContent {
  display: flex;
  align-items: center;
}

.sortIcon {
  margin-left: 8px;
  margin-right: 24px;
}

.sortIcon:hover {
  cursor: pointer;
}

.infPager {
  width: 100%;
  height: 32px;
}
</style>
