<template>
  <el-drawer size="70%" title="社团评分详情" direction="rtl" oncontextmenu="return false" onselectstart="return false"
    :visible.sync="isShowing"
    v-if="isShowing">
    <div class="demo-drawer__content" style="bottom:0">
      <div class="student">
        <div class="main long" style="top:0">
          <div class="mid">
            <el-table stripe height="100%"
              :data="items"
              @selection-change="selectionChange">
              <el-table-column label="姓名" prop="name" min-width="100" fixed></el-table-column>

              <el-table-column label="学号" prop="student_no" min-width="180" fixed></el-table-column>

              <el-table-column label="年级" prop="grade" width="120"></el-table-column>

              <el-table-column label="班级" prop="class_name" width="120"></el-table-column>

              <el-table-column label="出勤率" prop="attendance_rate" width="80"></el-table-column>

              <el-table-column label="社团实践分" prop="point" width="120">
                <template slot-scope="scope">
                  <input :class="(!isMe || state == '-1')?'disabled_input':''" type="text" :disabled="!isMe || state == '-1'"  :value=scope.row.point @change="ChangePoint($event,scope.row)" class="el-input__inner" placeholder="请输入分数">
                </template>
              </el-table-column>

              <el-table-column label="备注" prop="remark" width="150">
                <template slot-scope="scope">
                  <input :class="(!isMe || state == '-1')?'disabled_input':''" type="text" :disabled="!isMe || state == '-1'" :value=scope.row.remark @change="ChangeRemark($event,scope.row)" class="el-input__inner" placeholder="请输入备注">
                </template>
              </el-table-column>
            </el-table>
          </div>

          <div class="bot"
            v-if="cnt">
            <ys-page ref="yspage"
              :cnt="cnt"
              :size="size"
              v-on:page="pageChange"></ys-page>
          </div>
        </div>

        <!-- 确认模态框 -->
        <ys-modal-confirm ref="ysconfirm"
          :confirmData="confirmData"
          v-on:confirmCalBak="toReset"></ys-modal-confirm>

        <ys-modal-confirm ref="ysconfirm2"
          :confirmData="confirmData"
          v-on:confirmCalBak="toDel"></ys-modal-confirm>

        <!-- 表单模态框 -->
        <ys-modal-form ref="ysform"
          :formData="formData"
          :formRule="formRule"
          v-on:formCalBak="askDatas"></ys-modal-form>

        <ys-modal-edit-form ref="ysform2"
          :formData="formEditData"
          :formRule="formEditRule"
          v-on:formCalBak="askDatas"></ys-modal-edit-form>

        <!-- 文件模态框 -->
        <ys-modal-file ref="ysfile"
          :fileData="fileData"
          v-on:fileCalBak="askDatas"></ys-modal-file>
      </div>
    </div>
  </el-drawer>
</template>

<script>
import { mapGetters } from 'vuex'
import mixdrawer from '../mixin/drawer'
import mixtable from '../mixin/table'
import ysModalForm from '@/components/modalform/addStudent'
import ysModalEditForm from '@/components/modalform/editStudent'

export default {
  mixins: [mixdrawer, mixtable],
  components: {
    ysModalForm, ysModalEditForm
  },
  props: [
  ],
  data () {
    return {
      items: [], // 学生数据
      itemsEdu: [], // 学段数据
      itemsRoom: [], // 班级数据
      cnt: 0, // 总数
      size: 20, // 单页数目
      selection: [], // 已勾选列表
      point: '',
      remark: '',
      org_id: '',
      state: '',
      params: { page: 1, select_edu_stage: '', year: '', room_id: '', searchKey: '' }, // 参数

      // 查询内容数据
      searchData: {
        keywords: true,
        buttons: [
          { key: 'addNew', value: '新增学生' }
        ],
        files: { key: 'fileDeal', value: null, placeholder: null, model: '' },
        isMe: ''
      },

      // 确认内容数据
      confirmData: { k: 'student' },

      // 表单内容数据
      formData: {
        id: undefined,
        select_edu_stage: '',
        year: '',
        room_id: '',
        students: ''
      },

      // 表单内容验证
      formRule: [
        ['needless', { list: null }],
        ['needless', { list: null }],
        ['select', { list: null, data: null }],
        ['cont', {}]
      ],

      // 表单内容数据
      formEditData: {
        id: undefined,
        select_edu_stage: '',
        year: '',
        room_id: '',
        name: '',
        sex: '',
        student_no: '',
        home_mobile: ''
      },

      // 表单内容验证
      formEditRule: [
        ['needless', { list: null }],
        ['needless', { list: null }],
        ['select', { list: null, data: null }],
        ['name', {}],
        ['need', {}],
        ['no', {}],
        ['tel', {}]
      ],

      // 文件内容数据
      fileData: { name: '学生' }
    }
  },
  computed: {
    ...mapGetters([
      'xueduan', 'nianji',
      'xingbie',
      'piliang'
    ])
  },
  methods: {
    /**
     * [initial 初始化组件]
     * @return {[]} []
     */
    initial (isMe) {
      Object.assign(this.params, this.tableData)
      console.log(isMe)
      this.isMe = isMe
      this.askDatas(() => {
        this.setSearchData()
        this.setFormData()
        this.toggleShow()
      })
    },

    /**
     * [askDatas 请求数据]
     * @param  {[Function]} cbk [回调]
     * @return {[]} []
     */
    askDatas (cbk) {
      let $rt = this.$get('palace_org/getOrganization/', this.params)
      $rt.then((rt) => {
        // 学生数据
        this.items = rt.data.studentList
        this.cnt = rt.data.cnt
        this.size = rt.data.pagesize
        this.org_id = rt.data.data.id
        this.state = rt.data.data.state
        for (let item of this.items) {
          let i = this.items.indexOf(item)
          this.items[i].grade = this.items[i].grade + '年级'
        }
        cbk()
      }).catch((rt) => {
      })
    },

    /**
     * [setSearchData 设置查询内容数据]
     * @return {[]} []
     */
    setSearchData () {
      const $file = this.searchData.files
      $file.value = this.piliang
      $file.placeholder = this.piliang[0]['name']
    },

    /**
     * [setFormData 设置表单内容数据]
     * @return {[]} []
     */
    setFormData () {
      const $edu = this.formRule[0][1]
      $edu.list = JSON.parse(JSON.stringify(this.itemsEdu))

      const $room = this.formRule[2][1]
      $room.data = JSON.parse(JSON.stringify(this.itemsRoom))

      const $eduEdit = this.formEditRule[0][1]
      $eduEdit.list = JSON.parse(JSON.stringify(this.itemsEdu))

      const $roomEdit = this.formEditRule[2][1]
      $roomEdit.data = JSON.parse(JSON.stringify(this.itemsRoom))
    },

    /**
     * [pageChange 切换页码]
     * @param  {[Int]} p [页码]
     * @return {[]} []
     */
    pageChange (p) {
      this.params.page = p
      this.askDatas()
    },

    /**
     * [paramsChange 切换查询条件]
     * @param  {[Object]} $params [查询参数]
     * @return {[]} []
     */
    paramsChange ($params) {
      $params.page = 1
      const keys = Object.keys(this.params)
      for (let k of keys) {
        if ($params[k]) this.params[k] = $params[k]
        else this.params[k] = ''
      }

      this.askDatas()
    },

    /**
     * [selectionChange 切换勾选项]
     * @param  {[Array]} arr [勾选列表]
     * @return {[]} []
     */
    selectionChange (arr) {
      this.selection = arr
    },

    /**
     * [toAdd 新增学生]
     * @param  {[Int]} id [用户ID]
     * @return {[]} []
     */
    toAdd (id) {
      this.$refs.ysform.initial()
    },

    /**
     * [toEdit 编辑学生]
     * @param  {[Int]} id [用户ID]
     * @return {[]} []
     */
    toEdit (id) {
      this.formEditData.id = id
      this.$refs.ysform2.initial()
    },

    /**
     * [goReset 准备重置密码]
     * @param  {[Int]} id [用户ID]
     * @return {[]} []
     */
    goReset (id) {
      const params = { id: id }

      this.$refs.ysconfirm.toggleShow(params)
      this.confirmData.v = 0
    },

    /**
     * [toReset 重置密码]
     * @param  {[Object]} _params [中转参数]
     * @return {[]} []
     */
    toReset (_params) {
      let $rt = this.$post('admin/reset_student_pwd/', _params)
      $rt.then((rt) => {
        this.$message({
          type: 'success',
          message: '密码重置成功！当前密码同账号',
          showClose: true
        })
      }).catch((rt) => {
      })
    },

    /**
     * [goDel 准备删除学生]
     * @param  {[Int]} id [用户ID]
     * @return {[]} []
     */
    goDel (id) {
      let v = 2

      // 批量处理
      if (!id) {
        // 未勾选
        if (this.selection.length === 0) {
          this.$refs.ysconfirm2.toggleShow()
          this.confirmData.v = 3
          return
        }

        id = []
        for (let item of this.selection) {
          id.push(item.id)
        }
        id = id.join(',')
      } else {
        v = 1
      }

      const params = { ids: id }

      this.$refs.ysconfirm2.toggleShow(params)
      this.confirmData.v = v
    },

    /**
     * [toDel 删除学生]
     * @param  {[Object]} _params [中转参数]
     * @return {[]} []
     */
    toDel (_params) {
      let $rt = this.$post('admin/del_student/', _params)
      $rt.then((rt) => {
        this.askDatas()
      }).catch((rt) => {
      })
    },

    /**
     * [toFile 批量操作]
     * @param  {[Int]} v [操作方式]
     * @return {[]} []
     */
    toFile (v) {
      if (v === '0') {
        let file = this.url + 'admin/exportStaudent/?uc_sid=' + this.cookie
        const keys = Object.keys(this.params)
        for (let k of keys) {
          if (k === 'page') continue

          let v = this.params[k]
          file += '&' + k + '=' + v
        }

        window.open(file)
      } else {
        this.$refs.ysfile.initial()
      }
    },
    // 验证是否为数字
    isNumber (value) {
      let patrn = /^(-)?\d+(\.\d+)?$/
      if (patrn.exec(value) == null || value === '') {
        return false
      } else {
        return true
      }
    },
    ChangePoint (e, data) {
      let point = e.currentTarget.value
      if (!this.isNumber(point)) {
        this.$err('请输入数字')
        return
      } else if (point < 0 || point > 5) {
        this.$err('请输入0-5之间的数字')
        return
      }
      var param = {'student_id': data.student_id, 'org_id': this.org_id, 'field': 'point', 'content': e.currentTarget.value}
      let $rt = this.$get('palace_org/alterStudentField/', param)
      $rt.then((rt) => {
        console.log('rt1', rt)
        this.$message({
          type: 'success',
          message: '成功',
          showClose: true
        })
      }).catch((rt) => {
        console.log('rt2', rt)
      })
    },
    ChangeRemark (e, data) {
      var param = {'student_id': data.student_id, 'org_id': this.org_id, 'field': 'remark', 'content': e.currentTarget.value}
      let $rt = this.$get('palace_org/alterStudentField/', param)
      $rt.then((rt) => {
        console.log('rt1', rt)
        this.$message({
          type: 'success',
          message: '成功',
          showClose: true
        })
      }).catch((rt) => {
        console.log('rt2', rt)
      })
    },
    /**
     * [currentChange 选择报告]
     * @param  {[Object]} item [元素数据]
     * @return {[]} []
     */
    currentChange (item) {
      if (!item || !item.id) return

      this.tableDetail.selected_id = item.id
    }
  },
  created () {
  },
  mounted () {
  }
}
</script>
