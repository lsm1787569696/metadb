<template>
  <div class="options-layout clearfix">
    <div class="options options-left fl">
      <cmdb-auth class="option" :auth="{ type: $OPERATION.C_SERVICE_INSTANCE, relation: [bizId] }">
        <bk-button theme="primary" slot-scope="{ disabled }" v-test-id="'addHost'"
          :disabled="disabled || !isNormalModuleNode"
          :title="isNormalModuleNode ? '' : $t('仅能在业务模块下新增')"
          @click="handleAddHost">
          {{$t('新增')}}
        </bk-button>
      </cmdb-auth>
      <bk-button class="ml10" v-test-id="'edit'"
        :disabled="!hasSelection"
        @click="handleMultipleEdit">
        {{$t('编辑')}}
      </bk-button>
      <bk-dropdown-menu class="option ml10" trigger="click"
        font-size="medium"
        :disabled="!hasSelection"
        @show="isTransferMenuOpen = true"
        @hide="isTransferMenuOpen = false">
        <bk-button slot="dropdown-trigger" v-test-id="'transfer'"
          :disabled="!hasSelection">
          <span>{{$t('转移至')}}</span>
          <i :class="['dropdown-icon bk-icon icon-angle-down',{ 'open': isTransferMenuOpen }]"></i>
        </bk-button>
        <ul class="bk-dropdown-list" slot="dropdown-content" v-test-id="'transfer'">
          <cmdb-auth tag="li" class="bk-dropdown-item" v-test-id="'transferIdle'"
            :auth="[
              { type: $OPERATION.C_SERVICE_INSTANCE, relation: [bizId] },
              { type: $OPERATION.U_SERVICE_INSTANCE, relation: [bizId] },
              { type: $OPERATION.D_SERVICE_INSTANCE, relation: [bizId] }
            ]"
            @click="handleTransfer($event, 'idle', false)">
            {{$t('空闲模块', { idleSet: $store.state.globalConfig.config.set })}}
          </cmdb-auth>
          <cmdb-auth tag="li" class="bk-dropdown-item" v-test-id="'transferBusiness'"
            :auth="[
              { type: $OPERATION.C_SERVICE_INSTANCE, relation: [bizId] },
              { type: $OPERATION.U_SERVICE_INSTANCE, relation: [bizId] },
              { type: $OPERATION.D_SERVICE_INSTANCE, relation: [bizId] }
            ]"
            @click="handleTransfer($event, 'business', false)">
            {{$t('业务模块')}}
          </cmdb-auth>
          <li :class="['bk-dropdown-item', { disabled: !isIdleModule }]" v-test-id="'transferResource'"
            @click="handleTransfer($event, 'resource', !isIdleModule)">
            {{$t('主机池')}}
          </li>
          <li :class="['bk-dropdown-item', { disabled: !isIdleModule }]" v-test-id="'transferAcrossBusiness'"
            @click="handleTransfer($event, 'acrossBusiness', !isIdleModule)">
            {{$t('其他业务')}}
          </li>
        </ul>
      </bk-dropdown-menu>
      <bk-dropdown-menu class="option ml10" trigger="click"
        v-show="isNormalNode"
        font-size="medium"
        :disabled="!hasSelection"
        @show="isAddToOpen = true"
        @hide="isAddToOpen = false">
        <bk-button slot="dropdown-trigger" v-test-id="'addTo'"
          :disabled="!hasSelection">
          <span>{{$t('追加至')}}</span>
          <i :class="['dropdown-icon bk-icon icon-angle-down',{ 'open': isAddToOpen }]"></i>
        </bk-button>
        <ul class="bk-dropdown-list" slot="dropdown-content">
          <cmdb-auth tag="li" class="bk-dropdown-item with-auth" v-test-id="'addToBiz'"
            :auth="{ type: $OPERATION.C_SERVICE_INSTANCE, relation: [bizId] }">
            <span href="javascript:void(0)"
              slot-scope="{ disabled }"
              :class="{ disabled }"
              @click="handleTransfer($event, 'increment', false)">
              {{$t('业务模块')}}
            </span>
          </cmdb-auth>
        </ul>
      </bk-dropdown-menu>
      <bk-dropdown-menu class="option ml10" trigger="click"
        v-show="isNormalModuleNode"
        font-size="medium"
        :disabled="!hasSelection"
        @show="isRemoveMenuOpen = true"
        @hide="isRemoveMenuOpen = false">
        <bk-button slot="dropdown-trigger" v-test-id="'remove'"
          :disabled="!hasSelection">
          <span>{{$t('移出')}}</span>
          <i :class="['dropdown-icon bk-icon icon-angle-down',{ 'open': isRemoveMenuOpen }]"></i>
        </bk-button>
        <ul class="bk-dropdown-list" slot="dropdown-content">
          <cmdb-auth tag="li" class="bk-dropdown-item with-auth" v-test-id="'remove'"
            :auth="{ type: $OPERATION.D_SERVICE_INSTANCE, relation: [bizId] }">
            <span href="javascript:void(0)"
              slot-scope="{ disabled }"
              :class="{ disabled: !removeAvailable || disabled }"
              @click="handleRemove($event)">
              {{$t('当前模块')}}
            </span>
          </cmdb-auth>
        </ul>
      </bk-dropdown-menu>
      <cmdb-clipboard-selector class="options-clipboard ml10" v-test-id
        label-key="bk_property_name"
        :list="clipboardList"
        :disabled="!hasSelection"
        @on-copy="handleCopy">
      </cmdb-clipboard-selector>
      <bk-dropdown-menu class="option ml10" trigger="click"
        font-size="medium"
        @show="isMoreMenuOpen = true"
        @hide="isMoreMenuOpen = false">
        <bk-button slot="dropdown-trigger" v-test-id="'more'">
          <span>{{$t('更多')}}</span>
          <i :class="['dropdown-icon bk-icon icon-angle-down',{ 'open': isMoreMenuOpen }]"></i>
        </bk-button>
        <ul class="bk-dropdown-list" slot="dropdown-content" v-test-id="'more'">
          <li :class="['bk-dropdown-item', { disabled: !hasSelection }]" @click="handleExport($event)"
            v-test-id="'export'">
            {{$t('导出选中')}}
          </li>
          <li :class="['bk-dropdown-item', { disabled: !count }]" @click="handleBatchExport($event)"
            v-test-id="'batchExport'">
            {{$t('导出全部')}}
          </li>
          <cmdb-auth tag="li" class="bk-dropdown-item with-auth" v-test-id="'importUpdate'"
            :auth="{ type: $OPERATION.U_HOST, relation: [bizId] }">
            <span href="javascript:void(0)"
              slot-scope="{ disabled }"
              :class="{ disabled: disabled }"
              @click="handleExcelUpdate($event)">
              {{$t('导入excel更新')}}
            </span>
          </cmdb-auth>
        </ul>
      </bk-dropdown-menu>
    </div>
    <div class="options options-right">
      <filter-fast-search class="option-fast-search" v-test-id></filter-fast-search>
      <filter-collection class="option-collection ml10" v-test-id></filter-collection>
      <icon-button class="option-filter ml10" v-test-id="'advancedSearch'"
        icon="icon-cc-funnel" v-bk-tooltips.top="$t('高级筛选')"
        @click="handleSetFilters">
      </icon-button>
    </div>
    <edit-multiple-host ref="editMultipleHost" v-test-id
      :properties="hostProperties"
      :selection="$parent.table.selection">
    </edit-multiple-host>
    <cmdb-dialog v-model="dialog.show" v-bind="dialog.props" :height="650">
      <component
        :is="dialog.component"
        v-bind="dialog.componentProps"
        @confirm="handleDialogConfirm"
        @cancel="handleDialogCancel">
      </component>
    </cmdb-dialog>
    <bk-sideslider
      v-transfer-dom
      :is-show.sync="sideslider.show"
      :width="600"
      :title="sideslider.title">
      <component slot="content"
        :is="sideslider.component"
        v-bind="sideslider.componentProps"
        @on-cancel="sideslider.show = false">
      </component>
    </bk-sideslider>
  </div>
</template>

<script>
  import CmdbImport from '@/components/import/import'
  import EditMultipleHost from './edit-multiple-host.vue'
  import HostSelector from './host-selector-new'
  import { mapGetters } from 'vuex'
  import {
    MENU_BUSINESS_TRANSFER_HOST
  } from '@/dictionary/menu-symbol'
  import FilterForm from '@/components/filters/filter-form.js'
  import FilterCollection from '@/components/filters/filter-collection'
  import FilterFastSearch from '@/components/filters/filter-fast-search'
  import FilterStore from '@/components/filters/store'
  import ExportFields from '@/components/export-fields/export-fields.js'
  import FilterUtils from '@/components/filters/utils'
  import batchExport from '@/components/batch-export/index.js'
  export default {
    components: {
      FilterCollection,
      FilterFastSearch,
      EditMultipleHost,
      [HostSelector.name]: HostSelector,
      [CmdbImport.name]: CmdbImport
    },
    data() {
      return {
        isTransferMenuOpen: false,
        isRemoveMenuOpen: false,
        isMoreMenuOpen: false,
        isAddToOpen: false,
        dialog: {
          show: false,
          props: {
            width: 1100
          },
          component: null,
          componentProps: {}
        },
        sideslider: {
          show: false,
          title: '',
          component: null,
          componentProps: {}
        },
        IPWithCloudSymbol: Symbol('IPWithCloud')
      }
    },
    computed: {
      ...mapGetters('userCustom', ['usercustom']),
      ...mapGetters('objectBiz', ['bizId']),
      ...mapGetters('businessHost', [
        'getProperties',
        'selectedNode'
      ]),
      hostProperties() {
        return FilterStore.getModelProperties('host')
      },
      count() {
        return this.$parent.table.pagination.count
      },
      selection() {
        return this.$parent.table.selection
      },
      hasSelection() {
        return !!this.selection.length
      },
      isNormalNode() {
        return this.selectedNode && this.selectedNode.data.default === 0
      },
      isNormalModuleNode() {
        return this.isNormalNode && this.selectedNode.data.bk_obj_id === 'module'
      },
      isIdleModule() {
        return this.selection.every((data) => {
          const modules = data.module
          return modules.every(module => module.default === 1)
        })
      },
      isIdleSetModules() {
        return this.selection.every(data => data.module.every(module => module.default >= 1))
      },
      removeAvailable() {
        return this.selectedNode
          && !this.selectedNode.data.is_idle_set
          && this.selectedNode.data.bk_obj_id === 'module'
          && this.selectedNode.data.default !== 1
      },
      clipboardList() {
        const IPWithCloud = FilterUtils.defineProperty({
          id: this.IPWithCloudSymbol,
          bk_obj_id: 'host',
          bk_property_id: this.IPWithCloudSymbol,
          bk_property_name: `${this.$t('云区域')}ID:IP`,
          bk_property_type: 'singlechar'
        })
        const clipboardList = FilterStore.header.slice()
        clipboardList.splice(1, 0, IPWithCloud)
        return clipboardList
      }
    },
    methods: {
      handleTransfer(event, type, disabled) {
        if (disabled) {
          event.stopPropagation()
          return false
        }
        this.$emit('transfer', type)
      },
      handleMultipleEdit() {
        this.$refs.editMultipleHost.handleMultipleEdit()
      },
      handleAddHost() {
        this.dialog.componentProps.title = this.$t('新增主机到模块')
        this.dialog.component = HostSelector.name
        this.dialog.show = true
      },
      handleRemove(event) {
        if (!this.hasSelection || !this.removeAvailable) {
          event.stopPropagation()
          return false
        }
        this.$routerActions.redirect({
          name: MENU_BUSINESS_TRANSFER_HOST,
          params: {
            type: 'remove'
          },
          query: {
            sourceModel: this.selectedNode.data.bk_obj_id,
            sourceId: this.selectedNode.data.bk_inst_id,
            resources: this.selection.map(item => item.host.bk_host_id).join(','),
            node: this.selectedNode.id
          },
          history: true
        })
      },
      handleExport(event) {
        if (!this.hasSelection) {
          event.stopPropagation()
          return false
        }
        ExportFields.show({
          title: this.$t('导出选中'),
          properties: FilterStore.getModelProperties('host'),
          propertyGroups: FilterStore.propertyGroups,
          handler: this.exportHanlder
        })
      },
      async exportHanlder(properties) {
        const formData = new FormData()
        formData.append('bk_biz_id', this.bizId)
        formData.append('bk_host_id', this.selection.map(({ host }) => host.bk_host_id).join(','))
        formData.append('export_custom_fields', properties.map(property => property.bk_property_id))
        try {
          this.$store.commit('setGlobalLoading', true)
          await this.$http.download({
            url: `${window.API_HOST}hosts/export`,
            method: 'post',
            data: formData
          })
        } catch (error) {
          console.error(error)
        } finally {
          this.$store.commit('setGlobalLoading', false)
        }
      },
      async handleBatchExport(event) {
        if (!this.count) {
          event.stopPropagation()
          return false
        }
        ExportFields.show({
          title: this.$t('导出全部'),
          properties: FilterStore.getModelProperties('host'),
          propertyGroups: FilterStore.propertyGroups,
          handler: this.batchExportHandler
        })
      },
      batchExportHandler(properties) {
        batchExport({
          name: 'host',
          count: this.count,
          options: (page) => {
            const condition = this.$parent.getParams()
            const formData = new FormData()
            formData.append('bk_biz_id', this.bizId)
            formData.append('export_custom_fields', properties.map(property => property.bk_property_id))
            formData.append('export_condition', JSON.stringify({
              ...condition,
              page: {
                ...page,
                sort: 'bk_host_id'
              }
            }))
            return {
              url: `${window.API_HOST}hosts/export`,
              method: 'post',
              data: formData
            }
          }
        })
      },
      handleExcelUpdate() {
        this.sideslider.component = CmdbImport.name
        this.sideslider.componentProps = {
          templateUrl: `${window.API_HOST}importtemplate/host`,
          importUrl: `${window.API_HOST}hosts/update`,
          templdateAvailable: false,
          importPayload: { bk_biz_id: this.bizId }
        }
        this.sideslider.title = this.$t('更新主机属性')
        this.sideslider.show = true
      },
      handleCopy(property) {
        const copyText = this.selection.map((data) => {
          const modelId = property.bk_obj_id
          const modelData = data[modelId]
          if (property.id === this.IPWithCloudSymbol) {
            const cloud = this.$tools.getPropertyCopyValue(modelData.bk_cloud_id, 'foreignkey')
            const ip = this.$tools.getPropertyCopyValue(modelData.bk_host_innerip, 'singlechar')
            return `${cloud}:${ip}`
          }
          const propertyId = property.bk_property_id
          if (Array.isArray(modelData)) {
            const value = modelData.map(item => this.$tools.getPropertyCopyValue(item[propertyId], property))
            return value.join(',')
          }
          return this.$tools.getPropertyCopyValue(modelData[propertyId], property)
        })
        this.$copyText(copyText.join('\n')).then(() => {
          this.$success(this.$t('复制成功'))
        }, () => {
          this.$error(this.$t('复制失败'))
        })
      },
      handleDialogConfirm() {
        if (this.dialog.component === HostSelector.name) {
          // 最新的逻辑中新增主机等于转移主机到当前模块，跳转到转移主机页面指定type=add
          // eslint-disable-next-line prefer-rest-params
          this.gotoTransferPage(...arguments)
        }
      },
      gotoTransferPage(selected) {
        const query = {
          sourceModel: this.selectedNode.data.bk_obj_id,
          sourceId: this.selectedNode.data.bk_inst_id,
          targetModules: this.selectedNode.data.bk_inst_id,
          resources: selected.map(item => item.host.bk_host_id).join(','),
          node: this.selectedNode.id
        }
        this.$routerActions.redirect({
          name: MENU_BUSINESS_TRANSFER_HOST,
          params: {
            type: 'add'
          },
          query,
          history: true
        })
      },
      handleDialogCancel() {
        this.dialog.show = false
      },
      handleSetFilters() {
        FilterForm.show()
      }
    }
  }
</script>

<style lang="scss" scoped>
    .options-layout {
        margin-top: 12px;
    }
    .options {
        display: flex;
        align-items: center;
        &.options-right {
            overflow: hidden;
            justify-content: flex-end;
        }
        .option {
            display: inline-block;
            vertical-align: middle;
        }
        .option-fast-search {
            flex: 1;
            max-width: 300px;
            margin-left: 10px;
        }
        .option-collection,
        .option-filter {
            flex: 32px 0 0;
            &:hover {
                color: $primaryColor;
            }
        }
        .dropdown-icon {
            margin: 0 -4px;
            display: inline-block;
            vertical-align: middle;
            height: auto;
            top: 0px;
            font-size: 20px;
            &.open {
                top: -1px;
                transform: rotate(180deg);
            }
        }
    }
    .bk-dropdown-list {
        font-size: 14px;
        color: $textColor;
        .bk-dropdown-item {
            position: relative;
            display: block;
            padding: 0 20px;
            margin: 0;
            line-height: 32px;
            cursor: pointer;
            @include ellipsis;
            &:not(.disabled):not(.with-auth):hover {
                background-color: #EAF3FF;
                color: $primaryColor;
            }
            &.disabled {
                color: $textDisabledColor;
                cursor: not-allowed;
            }
            &.with-auth {
                padding: 0;
                span {
                    display: block;
                    padding: 0 20px;
                    &:not(.disabled):hover {
                        background-color: #EAF3FF;
                        color: $primaryColor;
                    }
                    &.disabled {
                        color: $textDisabledColor;
                        cursor: not-allowed;
                    }
                }
            }
        }
    }
    /deep/ {
        .collection-item {
            width: 100%;
            display: flex;
            justify-content: space-between;
            align-items: center;
            &:hover {
                .icon-close {
                    display: block;
                }
            }
            .collection-name {
                @include ellipsis;
            }
            .icon-close {
                display: none;
                color: #979BA5;
                font-size: 20px;
                margin-right: -4px;
            }
        }
    }
</style>
