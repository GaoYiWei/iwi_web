<template>
    <div>
        <vxe-form :data="formData" :rules="formRules">
            <vxe-form-item :span="20">
                <template #default>
                    <vxe-button @click="insertEvent()">新增</vxe-button>
                    <vxe-button @click="saveEvent()" :disabled="ctrlDisabled.saveBtn">提交</vxe-button>
                    <vxe-button @click="editEvent()" :disabled="ctrlDisabled.editBtn">编辑</vxe-button>
                    <vxe-button @click="auditEvent()" :disabled="formData.status==-1||formData.status==null?true:ctrlDisabled.auditBtn">{{auditBtn}}</vxe-button>
                    <vxe-button @click="banEvent()" :disabled="formData.audited?false:true">{{banBtn}}</vxe-button>
                    <vxe-button @click="deleteEvent()" :disabled="ctrlDisabled.deleteBtn">删除</vxe-button>                    
                </template>
            </vxe-form-item>
            <vxe-form-item :span="4">
                <template #default>
                    <vxe-input v-model="searchVal" placeholder="请输入料号" type="search" clearable @keydown="enterSearch($event)" @search-click="searchEvent()"></vxe-input>
                </template>
            </vxe-form-item>
            <vxe-form-item title="料号" field="pn" :item-render="{}">
                <template #default="{ data }">
                    <vxe-input v-model="data.pn" :class="{ 'readonly': ctrlDisabled.table || mpnDisabled }" placeholder="请输入料号" @blur="getInfo()" clearable></vxe-input>
                </template>
            </vxe-form-item>
            <vxe-form-item title="品名" field="name" :item-render="{}">
                <template #default="{ data }">
                    <vxe-input v-model="data.name" readonly></vxe-input>
                </template>
            </vxe-form-item>
            <vxe-form-item title="型号" field="model" :item-render="{}">
                <template #default="{ data }">
                    <vxe-input v-model="data.model" readonly></vxe-input>
                </template>
            </vxe-form-item>
            <vxe-form-item title="品名描述" field="namedesc" :item-render="{}">
                <template #default="{ data }">
                    <vxe-input v-model="data.namedesc" readonly></vxe-input>
                </template>
            </vxe-form-item>
            <vxe-form-item title="备注" field="comment" :class="{ 'readonly': ctrlDisabled.table }" :item-render="{}">
                <template #default="{ data }">
                    <vxe-input v-model="data.comment"></vxe-input>
                </template>
            </vxe-form-item>
            <vxe-form-item title="状态" field="status" :item-render="{}">
                <template #default="{ data }">
                    <span>{{statusInputBox}}</span>
                </template>
            </vxe-form-item>
        </vxe-form>
        <div style="border:1px lightgray solid">
            <vxe-toolbar style="padding-left:10px" :class="{ 'readonly': ctrlDisabled.table }">
                <template #buttons>
                <vxe-button type="text" size="mini" icon="fa vxe-icon-add" @click="insertRowEvent()">新增</vxe-button>
                <vxe-button type="text" size="mini" icon="fa vxe-icon-delete" @click="$refs.xTable.removeCurrentRow()">删除</vxe-button>
                </template>
            </vxe-toolbar>
            <vxe-table
                border
                stripe
                resizable
                show-overflow
                ref="xTable"
                height="700"
                header-align="center"
                :row-config="{isHover: true, isCurrent: true, useKey: true}"
                :data="tableData"
                :edit-config="{trigger: 'click', mode: 'row', activeMethod: activeRowMethod}">
                <vxe-column type="seq" title="序号" width="80"></vxe-column>
                <vxe-column field="cpn" title="料号" :edit-render="{name: 'input', props: {type: 'text' }}" width="120"></vxe-column>
                <vxe-column field="qty" title="数量" :edit-render="{name: 'input', props: {type: 'number'}}" width="120"></vxe-column>
                <vxe-column field="comment" title="备注" :edit-render="{name: 'input', props: {type: 'text'}}" width="300"></vxe-column>
                <vxe-column field="name" title="品名" width="200">
                    <template #default="{ row }">
                        <span>{{ getName(row) }}</span>
                    </template>
                </vxe-column>
                <vxe-column field="model" title="型号" width="120">
                    <template #default="{ row }">
                        <span>{{ getModel(row) }}</span>
                    </template>
                </vxe-column>
                <vxe-column field="namedesc" title="品名描述" width="300">
                    <template #default="{ row }">
                        <span>{{ getNamedesc(row) }}</span>
                    </template>
                </vxe-column>
                <vxe-column field="manufact" title="厂家" width="200">
                    <template #default="{ row }">
                        <span>{{ getManufact(row) }}</span>
                    </template>
                </vxe-column>
            </vxe-table>
        </div>
        <vxe-form :data="formData">
            <vxe-form-item field="created" title="创建人" :span="4" :item-render="{}">
                <template #default="{ data }">
                    <vxe-input v-model="data.created" readonly></vxe-input>
                </template>
            </vxe-form-item>
            <vxe-form-item field="createdat" title="创建时间" :span="4" :item-render="{}">
                <template #default="{ data }">
                    <vxe-input v-model="data.createdat" readonly></vxe-input>
                </template>
            </vxe-form-item>
            <vxe-form-item field="edited" title="编辑人" :span="4" :item-render="{}">
                <template #default="{ data }">
                    <vxe-input v-model="data.edited" readonly></vxe-input>
                </template>
            </vxe-form-item>
            <vxe-form-item field="editedat" title="编辑时间" :span="4" :item-render="{}">
                <template #default="{ data }">
                    <vxe-input v-model="data.editedat" readonly></vxe-input>
                </template>
            </vxe-form-item>
            <vxe-form-item field="audited" title="审核人" :span="4" :item-render="{}">
                <template #default="{ data }">
                    <vxe-input v-model="data.audited" readonly></vxe-input>
                </template>
            </vxe-form-item>
            <vxe-form-item field="auditedat" title="审核时间" :span="4" :item-render="{}">
                <template #default="{ data }">
                    <vxe-input v-model="data.auditedat" readonly></vxe-input>
                </template>
            </vxe-form-item>
        </vxe-form>
    </div>
</template>

<script>
import { mapState } from 'vuex'
export default {
    data() {
        return {
            formData: {
                pn: null,
                name: null,
                model: null,
                namedesc: null,
                created: null,
                createdat: null,
                edited: null,
                editedat: null,
                audited: null,
                auditedat: null,
                comment: null,
                status: 0,
                deactivateat: null
            },
            formRules: {
                pn: [
                    { required: true, message: '请输入料号' }
                ]
            },
            dataTemp: null,
            tableData: [],
            isEdit: false,
            mpnDisabled: true,
            mpnList: [],
            searchVal: '',
            ctrlDisabled: {
                saveBtn: true,
                auditBtn: true,
                deleteBtn: true,
                banBtn: true,
                editBtn: true,
                table: true
            },
            isInsert: false
        }
    },
    computed: {
        ...mapState(['inventory']),
        auditBtn() {
            return this.formData.audited ? '弃审' : '审核'
        },
        banBtn() {
            return this.formData.status==1 || this.formData.status==-1 ? '停用' : '启用'
        },
        statusInputBox() {
            return this.formData.status==1 || this.formData.status==-1 ? '启用' : '停用'
        }  
    },
    mounted() {
    },
    methods : {
        activeRowMethod ({ row, rowIndex }) {
            return !this.ctrlDisabled.table
        },
        enterSearch($event) {
            if($event.$event.key=='Enter') { 
                this.searchEvent()
            }
        },
        searchEvent() {
            if(this.inventory[this.searchVal]) {
                this.submitLoading = true
                this.$axios({
                    method: 'GET',
                    url: '/api/bom',
                    params: { pn: this.searchVal }
                }).then(res => {
                    if(res.data['bom_m'].length==0) {
                        this.$message({ message: '没有找到记录', type: 'warning' })
                        return
                    }
                    this.submitLoading = false
                    this.formData = res.data['bom_m'][0]
                    this.formData.name=this.inventory[this.formData.pn]['name']
                    this.formData.model=this.inventory[this.formData.pn]['model']
                    this.formData.namedesc=this.inventory[this.formData.pn]['namedesc']
                    this.tableData = res.data['bom_c']
                    this.ctrlDisabled.saveBtn = true
                    this.ctrlDisabled.auditBtn = false
                    this.ctrlDisabled.banBtn = false
                    this.ctrlDisabled.table = true
                    if(this.formData.audited) {
                        this.ctrlDisabled.deleteBtn = true
                        this.ctrlDisabled.editBtn = true               
                    } else {                        
                        this.ctrlDisabled.deleteBtn = false
                        this.ctrlDisabled.editBtn = false 
                    }
                }).catch(err => {
                    this.submitLoading = false
                    this.$message({ message: err, type: 'error' })
                })
            } else {
                this.$message({ message: '料号不存在, 请刷新后再试', type: 'error' })
            }
        },
        async insertEvent() {
            if(this.isInsert||this.isEdit){
                const confirmRes = await this.$confirm(
                    '当前单据未保存, 是否继续?',
                    '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                    }
                ).catch(() => this.$message.info('已经取消新增'))
                if(confirmRes!=='confirm') {
                    return
                }
            }
            this.submitLoading = true
            this.isInsert = true
            this.$axios({
                method: 'GET',
                url: '/api/bom',
            }).then(res => {
                this.mpnList = []
                res.data['bom_m'].forEach(item => {
                    this.mpnList.push(item.pn)
                })
            }).catch(err => {
                this.submitLoading = false
                this.$message({ message: err, type: 'error' })
            })
            this.mpnDisabled = false
            this.$nuxt.$emit('btnCtrl', 'add', res => {
                this.ctrlDisabled = res
            })
            this.formData = {
                pn: null,
                name: null,
                model: null,
                namedesc: null,
                created: this.$store.state.user.name,
                createdat: new Date().toLocaleString('chinese', { hour12: false }),
                edited: null,
                editedat: null,
                audited: null,
                status: 0,
                auditedat: null
            }
            this.$refs.xTable.remove()
            for(var i=0;i<10;i++) {
                this.$refs.xTable.insert({})
            }            
        },
        editEvent() {
            this.isEdit = true
            this.mpnDisabled = true
            this.dataTemp = JSON.stringify(this.getData())
            this.$nuxt.$emit('btnCtrl', 'edit', res => {
                this.ctrlDisabled = res
            })
        },
        deleteEvent() {
            var btnStatus = this.ctrlDisabled
            this.$nuxt.$emit('btnCtrl', 'delete', res => {
                this.ctrlDisabled = res
            })
            this.$confirm('此操作将永久删除该记录, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                this.submitLoading = true
                this.$axios({
                    method: 'DELETE',
                    url: '/api/bom',
                    params: { pn: this.formData.pn }
                }).then(res => {
                    this.submitLoading = false
                    if(res.data=='OK') {
                        this.isEdit = false
                        this.formData = {
                            pn: null,
                            name: null,
                            model: null,
                            namedesc: null,
                            created: null,
                            createdat: null,
                            edited: null,
                            editedat: null,
                            audited: null,
                            auditedat: null
                        }
                        this.$refs.xTable.remove()
                        this.$message({ message: '删除成功', type: 'success' })
                    } else {
                        this.$message({ message: res.data, type: 'error' })
                        this.ctrlDisabled = btnStatus
                    }
                }).catch(err => {
                    this.ctrlDisabled = btnStatus
                    this.submitLoading = false
                    this.$message({ message: err, type: 'error' })
                })
            }).catch(() => {
                this.ctrlDisabled = btnStatus
            })
        },
        auditEvent() {
            var btnStatus = this.ctrlDisabled
            this.$nuxt.$emit('btnCtrl', this.formData.audited ? 'unaudit' : 'audit', res => {
                this.ctrlDisabled = res
            })
            var audited=this.$store.state.user.name, auditedat=new Date().toLocaleString('chinese', { hour12: false }), msg='已审核'
            if(this.formData.audited) {
                audited = null
                auditedat = null
                msg = '已弃审'
            }
            var data = { w: { pn: this.formData.pn }, v: { audited: audited, auditedat: auditedat, status: 1 } }
            this.submitLoading = true
            this.$axios({
                method: 'PATCH',
                url: '/api/bom',
                params: data
            }).then(res => {
                this.submitLoading = false
                if(res.data=='OK') {
                    this.$message({ message: msg, type: 'success' })
                    this.formData.audited = audited
                    this.formData.auditedat = auditedat
                    this.formData.status = 1
                } else {
                    this.$message({ message: res.data, type: 'error' })
                    this.ctrlDisabled = btnStatus
                }
            }).catch(err => {
                this.ctrlDisabled = btnStatus
                this.submitLoading = false
                this.$message({ message: err, type: 'error' })
            })
        },
        saveEvent() {
            var data = this.getData()
            var btnStatus = this.ctrlDisabled, mpnStatus = this.mpnDisabled            
            if(!data) { return }
            this.mpnDisabled = true
            this.$nuxt.$emit('btnCtrl', 'save', res => {
                this.ctrlDisabled = res
            })
            this.submitLoading = true
            if(this.isEdit) {
                if(this.dataTemp==JSON.stringify(data)) {
                    this.$message({ message: '数据未修改, 此次未提交'})
                    return
                }
                this.formData.edited = this.$store.state.user.name
                this.formData.editedat = new Date().toLocaleString('chinese', { hour12: false })
                this.$axios({
                    method: 'PUT',
                    url: '/api/bom',
                    data: data
                }).then(res => {
                    this.submitLoading = false
                    if(res.data=='OK') {
                        this.$message({ message: '保存成功', type: 'success' })
                        this.isEdit = false
                    } else {
                        this.$message({ message: res.data, type: 'error' })
                        this.ctrlDisabled = btnStatus
                    }
                }).catch(err => {
                    this.ctrlDisabled = btnStatus
                    this.mpnDisabled = mpnStatus
                    this.submitLoading = false
                    this.$message({ message: err, type: 'error' })
                })
            } else {
                this.$axios({
                    method: 'POST',
                    url: '/api/bom',
                    data: data
                }).then(res => {
                    this.submitLoading = false
                    if(res.data=='OK') {
                        this.$message({ message: '保存成功', type: 'success' })
                        this.isInsert = false
                    } else {
                        this.$message({ message: res.data, type: 'error' })
                        this.ctrlDisabled = btnStatus
                    }
                }).catch(err => {
                    this.ctrlDisabled = btnStatus
                    this.mpnDisabled = mpnStatus
                    this.submitLoading = false
                    this.$message({ message: err, type: 'error' })
                })
            }
        },
        banEvent() {
            var status, msg='已停用', deactivateat=null
            this.submitLoading = true
            if(this.formData.status==1) {
                status = 0
                deactivateat = new Date().toLocaleString('chinese', { hour12: false })
            } else if(this.formData.status==0) {
                status = 1
                msg = '已启用'
            } else if(this.formData.status==-1) {
                status = null
                deactivateat = new Date().toLocaleString('chinese', { hour12: false })
            } else if(this.formData.status==null) {
                status = -1
                msg = '已启用'
            }
            let data = { w: { pn: this.formData.pn }, v: { status: status, deactivateat: deactivateat } }
            this.$axios({
                method: 'PATCH',
                url: '/api/bom',
                params: data
            }).then(res => {
                this.submitLoading = false
                if(res.data=='OK') {
                    this.$message({ message: msg, type: 'success' })
                    this.formData.status = status
                } else {                    
                    this.$message({ message: res.data, type: 'error' })
                }
            }).catch(err => {
                this.submitLoading = false
                this.$message({ message: err, type: 'error' })
            })
        },
        getData() {
            var res=[[this.formData]], t=[], t1={pn: this.formData.pn}, r={}, tableData=this.$refs.xTable.getTableData().tableData
            for(var i=0;i<tableData.length;i++) {
                if(tableData[i].cpn && tableData[i].qty>0) {     
                    r={}
                    t.push(Object.assign(r, t1, tableData[i]))
                } else if(tableData[i].cpn && !tableData[i].qty || tableData[i].cpn && tableData[i].qty<=0) {
                    this.$message({ message: tableData[i].cpn + '未填写有效数量', type: 'error' })
                    return
                }
            }
            if(t.length==0) {
                this.$message({ message: '未填写子料', type: 'error' })
                return
            }
            res.push(t)
            return res
        },
        getInfo() {
            if(!this.formData.pn) { return }
            if(!this.inventory[this.formData.pn]) {
                this.formData.pn = null
                this.$message({ message: '料号不存在, 请刷新后再试', type: 'error' })
                return
            }
            if(this.mpnList.indexOf(this.formData.pn) > -1){
                this.formData.pn = null
                this.$message({ message: '当前料号已存在BOM', type: 'error' })
                return              
            }
            this.formData.name = this.inventory[this.formData.pn].name
            this.formData.model = this.inventory[this.formData.pn].model
            this.formData.namedesc = this.inventory[this.formData.pn].namedesc
        },
        getName(row) {
            if(!row.cpn) { return }
            if(this.inventory[row.cpn]) {
                if(this.inventory[row.cpn].status==1||this.inventory[row.cpn].status==-1||this.ctrlDisabled.table||this.isEdit){
                    var records = this.$refs.xTable.getTableData().tableData
                    for(var i=0;i<records.length;i++) {
                        if(!records[i].cpn || row._X_ROW_KEY==records[i]._X_ROW_KEY) {continue}
                        if(records[i].cpn==this.formData.pn) {
                            this.$message({ message: '子料与母号重复', type: 'warning' })
                            records[i].cpn = null
                            records[i].qty = null
                            return
                        }
                        if(row.cpn==records[i].cpn) {
                            this.$message({ message: '重复添加', type: 'warning' })
                            records[i].cpn = null
                            records[i].qty = null
                            return
                        }                  
                    }
                    return this.inventory[row.cpn].name
                } else {
                    this.$message({ message: '料号已停用', type: 'warning' })
                    row.qty = null
                    row.cpn = null
                    return
                }
            } else {
                this.$message({ message: '料号不存在, 请刷新后再试', type: 'warning' })
                row.qty = null
                row.cpn = null
                return
            }
        },
        getModel(row) {
            if(!row.cpn) { return }
            if(this.inventory[row.cpn]) {
                return this.inventory[row.cpn].model
            } else {
                return null
            }
        },
        getNamedesc(row) {
            if(!row.cpn){ return }
            if(this.inventory[row.cpn]) {
                return this.inventory[row.cpn].namedesc
            } else {
                return null
            }
        },
        getManufact(row) {
            if(!row.cpn){ return }
            if(this.inventory[row.cpn]) {
                return this.inventory[row.cpn].manufact
            } else {
                return null
            }
        },
        insertRowEvent() {
            this.$refs.xTable.insertAt({},-1)
        }
    }
}
</script>