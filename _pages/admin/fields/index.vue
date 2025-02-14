<template>
  <div 
    id="formfieldsPage" 
    class="layout-padding"
  >
    <!--Blocks Crud-->
    <crud 
      :crud-data="import('@imagina/qform/_crud/blocks')" 
      :custom-data="customCrudBlocks" type="no-index"
      ref="crudBlocks" 
      @updated="getData(false)" 
      @deleted="getData(false)" 
      @created="getData(false)"
    />
    <!--Fields Crud-->
    <crud 
      :crud-data="import('@imagina/qform/_crud/fields')" 
      :custom-data="customCrudFields" 
      type="no-index"
      ref="crudFields" 
      @updated="getData(false)"
      @deleted="getData(false)"
      @created="getData(false)"
    />

    <crud 
      :crud-data="import('@imagina/qform/_crud/crudForms.vue')" 
      :custom-data="customCrudForm" 
      type="no-index"
      @deleted="redirect"
      ref="crudForm" 
    />

    <!--page Content-->
    <div 
      id="formfieldsPageContent" 
      class="row q-col-gutter-md relative-position"
    >
      <!--Form info-->
      <q-skeleton v-if="loadingSkeleton" width="100vw" height="70px" />
      <div 
        class="col-12" 
        v-if="formData && !loadingSkeleton"
      >
        <!--Page Actions-->
        <div class="box box-auto-height">
          <page-actions 
            :title="`${$tr('isite.cms.label.form')}: ${formData.title}`"
            @refresh="getData"
            :extraActions="editAction"
            icon="fa-light fa-clipboard-list"
            :description="showDescription"
            :help="showTooltip"
          />
        </div>
      </div>
      <!--Blocks content (draggable)-->
      <div :class="{ 'content-dropdown col-12': isSon, 'col-12': !isSon }">
        <article class="block-list-container q-pr-md" v-if="isSon">
          <q-skeleton v-if="loadingSkeleton" height="55vh" />
          <router-link
            class="parent-list-header"
            v-if="!loadingSkeleton"
            target="_blank" 
            :to="`/form/fields/${parentForm.id}`"
          >
            <h2 class="blue-grey-9 text-h6 q-mr-md">
              {{ parentForm.title }}
            </h2>
            <i class="fa-light fa-arrow-up-right-from-square"></i>
            <q-tooltip>
              {{ $tr('iforms.cms.label.openParentForm') }}
            </q-tooltip>           
          </router-link>
          <dropdownList 
            v-for="(block, key) in parentForm.blocks" 
            v-if="!loadingSkeleton"
            :block="block"
            :key="key"
            :index="key + 1"
            :childFields="formData.blocks"
            :copiedFieldId="copiedFieldId"
          />
        </article>
        <div class="list-blocks-skeleton" v-if="loadingSkeleton">
          <q-skeleton 
            v-for="skeleton in new Array(6)" 
            :key="skeleton" 
            width="100%" 
            height="350px" 
          />
        </div>
        <draggable
          @change="updateOrderBlock"
          group="bocksBlocks"
          v-bind="dragOptions" 
          v-model="formData.blocks" 
          class="list-blocks"
          :style="{ '--max-size': isAutoWidth ? '400px' : '1fr'}"
          draggable=".enable"
        >
          <block 
            v-for="block in formData.blocks"
            v-if="!loadingSkeleton" 
            :key="block.id" 
            :block="block"
            :isSon="isSon"
            @createField="createField"
            :softLoading="softLoading"
            :updatedBlockId="updatedBlockId"
          >
            <draggable 
              @update="handleUpdatingFields(null)" 
              @change="props => handleChangeInFields(props, block.id)"
              :list="block.fields"
              group="bocksfields"
              v-bind="dragOptions"
              class="w-full draggable-fields q-pb-md"
              :class="{
                'drag-drog-field': block.fields.length === 0
              }"
              :data-descr="$tr('iforms.cms.label.dragFieldsHere')"
            >
              <div 
                v-for="field in block.fields" :key="field.id"
                class="items-center cursor-pointer"
              >
                <!--Field-->
                <div class="relative-position q-mb-sm">
                    <!--Field title-->
                    <div class="list-item-block text-grey-9">
                      <q-icon 
                        name="fa-sharp fa-light fa-grip-dots-vertical" 
                        class="q-mr-xs"
                      />
                      <span class="title-field">
                        {{ field.label || '--' }}
                      </span>
                    </div>
                    <dropdownMenu 
                      :block="field" 
                      :is-field="true"
                      @updateIdOfSelectedField="updateIdOfSelectedField"
                    />
                </div>
              </div>
            </draggable>
          </block>
          <button 
            class="btn-add-block"
            @click="handleCreateBlock"
            :disable="loading"
            v-show="!loadingSkeleton"
          >
            <span class="text-h3">
              <i class="fa fa-plus text-blue-grey-6" aria-hidden="true" />
            </span>
            <span class="text-subtitle1 text-blue-grey-6" >
              {{ $tr('iforms.cms.label.addNewBlock') }}
            </span>
          </button>
        </draggable>
      </div>
      <!--Inner Loading-->
      <inner-loading :visible="loading"/>
    </div>
  </div>
</template>

<script>
//import Component
import renderForm from '@imagina/qform/_components/frontend/forms/renderForm'
import formForm from '@imagina/qform/_components/admin/forms/form'
import draggable from 'vuedraggable'
import block from '@imagina/qform/_components/fields/block'
import dropdownList from '@imagina/qform/_components/fields/dropdownList'
import dropdownMenu from '@imagina/qform/_components/fields/dropdownMenu.vue'
import useFields from '@imagina/qform/uses/useFields.ts'

export default {
  components: {
    draggable,
    renderForm,
    formForm,
    dropdownList,
    dropdownMenu,
    block,
  },
  props: {
    formId: {
      type: Number,
      defualt: null,
    },
  },
  setup(props, { attrs }) {
    return { ...useFields(attrs, props) }
  }
}
</script>

<style scope>

.block-list-container {
  display: flex;
  flex-direction: column;
  width: 100%;
  min-width: 200px;
  max-width: 500px;
  gap: 6.4px;
  margin: 0 auto;
}

.parent-list-header {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 14px 0;
  margin-bottom: 8px;
}

.parent-list-header:hover {
  background-color: #f1f1f1;
  border-radius: 10px;
}

.parent-list-header > h2 {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.draggable-fields {
  min-height: 199px;
}

.drag-drog-field {
  color: #cccccc;
  text-align: center;
  border-radius: 10px;
  border: dashed 2px #cccccc;
  padding: 24px;
}

.title-field {
  width: 250px;

  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.drag-drog-field[data-descr]::after {
  content: attr(data-descr);
}

.content-dropdown {
  display: grid;
  grid-template-columns: minmax(30%, 100px) 1fr;
}

.list-blocks {
  display: grid;
  grid-template-columns: repeat(
    auto-fit, 
    minmax(300px, var(--max-size))
  );
  grid-template-rows: min-content;
  gap: 15px;
}

.list-blocks-skeleton {
  display: grid;
  grid-template-columns: repeat(
    auto-fit, 
    minmax(300px, 1fr)
  );
  grid-template-rows: min-content;
  gap: 15px;
}

.btn-add-block {
  display: flex;
  place-content: center;
  place-items: center;
  flex-direction: column;
  width: 100%;
  max-width: 370px;
  height: 370px;
  border: dashed 2px #cccccc;
  border-radius: 10px;
  margin: 0 auto;
}

.list-item-block {
  display: flex;
  align-items: center;
  width: 97%;
  box-sizing: border-box;
  padding: 12px 8px;
  border: solid #d8d8d8 1px;
  border-radius: 10px;
  cursor: pointer;
}

.list-item-block:active {
  cursor: grabbing;
}

.list-item-block:hover {
  background-color: #f5f5f5;
}
  
</style>