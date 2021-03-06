<template>
  <div class="page-header-index-wide">
    <a-card :body-style="{ padding: 0 }">
      <div class="ant-pro-table-toolbar">
        <div class="ant-pro-table-toolbar-title">岗位列表</div>
        <div class="ant-pro-table-toolbar-option">
          <div class="ant-pro-table-toolbar-item">
            <a-button v-action:PostAdd type="primary" icon="plus" :loading="loading" @click="visible = true">新建</a-button>
          </div>
          <template v-if="selectedRows.length">
            <div class="ant-pro-table-toolbar-item">
              <a-dropdown>
                <a-menu slot="overlay" @click="handleMenuClick">
                  <a-menu-item key="remove"><a-icon type="delete" />批量删除</a-menu-item>
                </a-menu>
                <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /> </a-button>
              </a-dropdown>
            </div>
          </template>
          <a-divider type="vertical" />
          <span class="ant-pro-table-toolbar-item-icon">
            <a-tooltip placement="top">
              <template slot="title">
                <span>刷新</span>
              </template>
              <a-icon type="reload" @click="refreshTable()"/>
            </a-tooltip>
          </span>
        </div>
      </div>

      <s-table
        ref="table"
        size="default"
        :rowKey="(record) => record.postId"
        :columns="columns"
        :data="loadData"
      >
        <template slot="status" slot-scope="row">
          <template v-if="row.status === 1">正常</template>
          <template v-else>禁用</template>
        </template>

        <template slot="tools" slot-scope="row">
          <a v-action:PostUpdate @click="openInfoModal(row)">编辑</a>
          <a-divider type="vertical" />
          <a v-action:PostDelete @click="showDeleteConfirm(row.postId)">删除</a>
        </template>
      </s-table>
    </a-card>

    <a-modal
      title="详情"
      :visible="visible"
      :width="500"
      @ok="handleSubmit"
      :confirmLoading="confirmLoading"
      @cancel="handleCancel">
      <a-form :form="form">
        <a-form-item label="名称" :labelCol="labelCol" :wrapperCol="wrapperCol" hasFeedback>
          <a-input
            placeholder="岗位名称"
            v-decorator="[
              'postName',
              {
                rules: [{ required: true, message: '请输入岗位名称!' }]
              }
            ]"
          />
        </a-form-item>

        <a-form-item label="标识" :labelCol="labelCol" :wrapperCol="wrapperCol" hasFeedback>
          <a-input
            placeholder="岗位标识"
            v-decorator="[
              'postCode',
              {
                rules: [{ required: true, message: '请输入岗位标识!' }]
              }
            ]"
          />
        </a-form-item>

        <a-form-item label="排序" :labelCol="labelCol" :wrapperCol="wrapperCol" hasFeedback>
          <a-input
            placeholder="排序"
            v-decorator="[
              'postSort',
              {
                rules: [{ required: true, message: '请输入排序!' }]
              }
            ]"
          />
        </a-form-item>

        <a-form-item label="状态" :labelCol="labelCol" :wrapperCol="wrapperCol" hasFeedback>
          <a-select
            v-decorator="[
              'status',
              {
                rules: [{ required: true, message: '请选择状态!' }]
              }
            ]"
            placeholder="请选择"
          >
            <a-select-option :value="1">正常</a-select-option>
            <a-select-option :value="0">禁用</a-select-option>
          </a-select>
        </a-form-item>
      </a-form>
    </a-modal>

  </div>
</template>
<script>
import { STable } from '@/components'
import { fetchPost, createPost, updatePost, deletePost } from '@/api/post'
const columns = [
  {
    title: '岗位标识',
    dataIndex: 'postCode'
  },
  {
    title: '岗位名称',
    dataIndex: 'postName'
  },
  {
    title: '岗位排序',
    dataIndex: 'postSort'
  },
  {
    title: '状态',
    scopedSlots: { customRender: 'status' }
  },
  {
    title: '创建时间',
    dataIndex: 'createTime'
  },
  {
    title: '操作',
    scopedSlots: { customRender: 'tools' }
  }
]

export default {
  data () {
    return {
      description: '',
      loading: false,
      visible: false,
      confirmLoading: false,
      selected: 0,
      columns,
      labelCol: {
        xs: { span: 24 },
        sm: { span: 5 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },
      form: this.$form.createForm(this),
      queryParam: {},
      loadData: parameter => {
        return fetchPost(Object.assign(parameter, this.queryParam)).then(res => {
          this.tree = res.result.data
          return res.result
        })
      },
      selectedRowKeys: [],
      selectedRows: []
    }
  },
  components: { STable },
  methods: {
    openInfoModal (row) {
      this.visible = true
      this.selected = row.postId
      this.$nextTick(() => {
        this.form.setFieldsValue({ ...row })
      })
    },
    onSelectChange (selectedRowKeys, selectedRows) {
      this.selectedRowKeys = selectedRowKeys
      this.selectedRows = selectedRows
    },
    handleMenuClick () {},
    handleSubmit () {
      this.form.validateFields((err, values) => {
        if (!err) {
          this.confirmLoading = true
          const promise = this.selected === 0 ? createPost(values) : updatePost(this.selected, values)
          promise.then(res => {
            this.$notification['success']({
              message: '成功通知',
              description: this.selected === 0 ? '添加成功！' : '更新成功！'
            })
            this.refreshTable()
            this.handleCancel()
          }).finally(() => {
            this.confirmLoading = false
          })
        }
      })
    },
    refreshTable () {
      this.$refs.table.refresh()
    },
    handleCancel () {
      this.visible = false
      this.selected = 0
      this.form.resetFields()
    },
    showDeleteConfirm (id) {
      this.$confirm({
        title: '确定删除此岗位吗?',
        content: '',
        okText: '确定',
        okType: 'danger',
        cancelText: '取消',
        onOk: () => {
          deletePost(id).then(res => {
            this.$notification['success']({
              message: '成功通知',
              description: '删除成功！'
            })
            this.refreshTable()
          })
        }
      })
    }
  }
}
</script>
