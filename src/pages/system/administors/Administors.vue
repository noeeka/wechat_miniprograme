<template>
    <div>
        <a-card
                style="margin-top: 24px"
                :bordered="false"
                title="系统管理员列表"
        >
            <div slot="extra">

                <a-input-search style="margin-left: 16px; width: 272px;"/>
            </div>
            <a-button type="dashed" style="width: 100%" icon="plus" @click="showAddModal">添加</a-button>
            <a-modal
                    title="添加管理员"
                    :visible="visible"
                    @ok="handleModalOk"
                    @cancel="handleModalCancel"
            >


                <a-form-model :model="form">
                    <a-form-model-item label="用户名">
                        <a-input v-model="form.username"/>
                    </a-form-model-item>
                    <a-form-model-item label="密码">
                        <a-input type="password" v-model="form.password"/>
                    </a-form-model-item>
                    <a-form-model-item label="确认密码">
                        <a-input type="password_check" v-model="form.password_check"/>
                    </a-form-model-item>
                    <a-form-model-item label="角色">
                        <a-select v-model="form.type">
                            <a-select-option value="0">
                                超级管理员
                            </a-select-option>
                            <a-select-option value="1">
                                管理员
                            </a-select-option>
                            <a-select-option value="2">
                                用户
                            </a-select-option>
                        </a-select>
                    </a-form-model-item>
                </a-form-model>


            </a-modal>
            <a-table :columns="columns" :data-source="data" :pagination="pagination" @change="handleTableChange"
                     bordered>
                <template
                        v-for="col in ['administrator_username', 'administrator_type']"
                        :slot="col"
                        slot-scope="text,record"
                >
                    <div :key="col">
                        <template v-if="record.editable">
                            <template v-if="col=='administrator_type'">
                                <a-select
                                        v-if="record.editable"
                                        :default-value='text' style="width: 120px"
                                        @change="handleChange($event, record.key, col)">
                                    <a-select-option value="0">
                                        超级管理员
                                    </a-select-option>
                                    <a-select-option value="1">
                                        管理员
                                    </a-select-option>
                                    <a-select-option value="2">
                                        用户
                                    </a-select-option>

                                </a-select>

                            </template>
                            <template v-else>
                                <a-input
                                        v-if="record.editable"
                                        style="margin: -5px 0"
                                        :value="text"
                                        @change="e => handleChange(e.target.value, record.key, col)"
                                />
                            </template>

                        </template>

                        <template v-else>

                            <template v-if="col=='administrator_type'">
                                <template v-if="text==0">
                                    超级管理员
                                </template>
                                <template v-if="text==1">
                                    管理员
                                </template>
                                <template v-if="text==2">
                                    用户
                                </template>

                            </template>

                            <template v-else>
                                {{ text }}
                            </template>
                        </template>

                    </div>
                </template>
                <template slot="operation" slot-scope="text, record">
                    <div class="editable-row-operations">
        <span v-if="record.editable">
          <a @click="() => save(record.key)">保存</a>
          <a-popconfirm title="是否要取消?" @confirm="() => cancel(record.key)">
            <a>取消</a>
          </a-popconfirm>
        </span>
                        <span v-else>
                            <a-button-group>
          <a-button :disabled="editingKey !== ''" @click="() => edit(record.key)">编辑</a-button>

                                <a-popconfirm

                                        title="是否要删除?"
                                        @confirm="() => onDelete(record.key)"
                                >
         <a-button type="danger" :disabled="editingKey !== ''">删除</a-button>
        </a-popconfirm>

                            </a-button-group>
        </span>
                    </div>
                </template>
            </a-table>
        </a-card>
    </div>
</template>


<script>
    const columns = [
        {
            title: '用户名',
            dataIndex: 'administrator_username',
            width: '25%',
            scopedSlots: {customRender: 'administrator_username'},
        },
        {
            title: '所属用户组',
            dataIndex: 'administrator_type',
            width: '15%',
            scopedSlots: {customRender: 'administrator_type'},
        },

        {
            title: '操作',
            dataIndex: 'operation',
            scopedSlots: {customRender: 'operation'},
        },
    ];

    const data = [];

    export default {
        name: 'Administors',
        data() {
            this.cacheData = data.map(item => ({...item}));
            return {
                form: {
                    username: '',
                    password: '',
                    password_check: '',
                    type: "0"
                },
                visible: false,
                pagination: {
                    total: 0,
                    pageSize: 20,//每页中显示10条数据
                    showSizeChanger: true,
                    pageSizeOptions: ["10", "20", "50", "100"],//每页中显示的数据
                    showTotal: total => `共有 ${total} 条数据`,  //分页中显示总的数据
                },


                // 查询参数
                queryParam: {
                    page: 1,//第几页
                    size: 10,//每页中显示数据的条数
                },
                data,
                columns,
                editingKey: '',
            };
        },
        mounted() {
            // eslint-disable-next-line no-undef
            this.$axios.get(this.GLOBAL.server_src + '/administrator')
                .then(response => (
                    this.data = response.data.data
                ))
        },
        methods: {
            onDelete(key){
                var formdata = new FormData();
                formdata.append("administrator_id", key);

                this.$axios.post(this.GLOBAL.server_src + '/administrator/del',
                    formdata
                ).then(response => {
                    if (response.data.state == 0) {
                        location.reload();
                    } else {
                        this.$message.warn('用户名已注册');
                    }
                })
            },
            showAddModal() {
                this.visible = true;
            },
            handleModalOk() {
                // this.ModalText = 'The modal will be closed after two seconds';
                if (this.form.password === this.form.password_check) {

                    var formdata = new FormData();
                    formdata.append("username", this.form.username);
                    formdata.append("password", this.form.password);
                    formdata.append("type", this.form.type);
                    this.$axios.post(this.GLOBAL.server_src + '/administrator/add',
                        formdata
                    ).then(response => {
                        // eslint-disable-next-line no-empty
                        if (response.data.state == 0) {
                            setTimeout(() => {
                                this.visible = false;
                            }, 2000);
                        } else {
                            this.$message.warn('用户名已注册');
                        }
                    })
                } else {
                    this.$message.warn('两次密码不一致');
                }

            },
            handleModalCancel() {
                console.log('Clicked cancel button');
                this.visible = false;
            },
            handleTableChange(pagination) {
                this.pagination.current = pagination.current;
                this.pagination.pageSize = pagination.pageSize;
                this.queryParam.page = pagination.current;
                this.queryParam.size = pagination.pageSize;


                this.loading = true;

                this.$axios.get(this.GLOBAL.server_src + '/administrator?page=' + pagination.current + '&page_size=' + pagination.pageSize)
                    .then(response => {

                        if (response.data.state == 0) {
                            const pagination = {...this.pagination};
                            pagination.total = response.data.total;
                            this.data = response.data.data
                            this.pagination = pagination;
                        }

                    })


            },
            handleChange(value, key, column) {
                const newData = [...this.data];
                const target = newData.filter(item => key === item.key)[0];
                if (target) {
                    target[column] = value;
                    this.data = newData;
                }
            },
            edit(key) {
                const newData = [...this.data];
                const target = newData.filter(item => key === item.key)[0];
                this.editingKey = key;

                if (target) {
                    target.editable = true;
                    this.data = newData;
                }
            },
            save(key) {
                const newData = [...this.data];
                const target = newData.filter(item => key === item.key)[0];
                var formdata = new FormData();
                formdata.append("administrator_id", key);
                formdata.append("username", target.administrator_username);
                formdata.append("password", target.administrator_username);
                formdata.append("type", target.administrator_type);
                this.$axios.post(this.GLOBAL.server_src + '/administrator/mod',
                    formdata
                ).then(response => {
                    // eslint-disable-next-line no-empty
                    if (response.data.state == 0) {
                        location.reload();
                    }
                })

            },
            cancel(key) {
                const newData = [...this.data];
                const target = newData.filter(item => key === item.key)[0];
                this.editingKey = '';
                if (target) {
                    Object.assign(target, this.cacheData.filter(item => key === item.key)[0]);
                    delete target.editable;
                    this.data = newData;
                }
            },
        },
    };
</script>

<style lang="less" scoped>
    .list-content-item {
        color: @text-color-second;
        display: inline-block;
        vertical-align: middle;
        font-size: 14px;
        margin-left: 40px;

        span {
            line-height: 20px;
        }

        p {
            margin: 4px 0 0;
            line-height: 22px;
        }
    }
</style>
