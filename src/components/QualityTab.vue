<template>
  <div>
    <!-- 批號選擇列 -->
    <div class="lot-bar">
      <span class="lot-label">批號</span>
      <select v-model="selectedLot" class="lot-select">
        <option value="">— 請選擇批號 —</option>
        <option v-for="lot in lots" :key="lot.lotNo" :value="lot.lotNo">
          {{ lot.lotNo }}（{{ lot.productKey }} ｜ {{ lot.procName }}{{ lot.section }}）
        </option>
      </select>
      <span v-if="selectedLotInfo" class="lot-info">
        品名：{{ selectedLotInfo.productKey }} ／ 製程：{{ selectedLotInfo.procName }}{{ selectedLotInfo.section }}
      </span>
    </div>

    <div v-if="!selectedLot" class="no-lot-hint">請先從上方下拉選單選擇批號，以查看或新增品檢資訊</div>

    <template v-else>
      <div class="section-bar">
        <span class="section-title">品檢項目</span>
        <button class="btn btn-primary" @click="handleAdd">新增</button>
        <button class="btn btn-warn"   :disabled="checkedIds.size === 0" @click="handleEdit">修改</button>
        <button class="btn btn-danger" :disabled="checkedIds.size === 0" @click="handleDelete">
          刪除<span v-if="checkedIds.size" class="badge">{{ checkedIds.size }}</span>
        </button>
      </div>

      <div class="table-wrap">
        <table class="dtable">
          <thead>
            <tr>
              <th width="36">勾選</th>
              <th>品檢項目</th>
              <th>標準值</th>
              <th>實測值</th>
              <th>判定</th>
              <th>備註</th>
            </tr>
          </thead>
          <tbody>
            <!-- 已存資料列（永遠顯示輸入框） -->
            <tr v-for="row in currentRows" :key="row.id"
                :class="{ 'row-checked': checkedIds.has(row.id) }">
              <td><input type="checkbox" :checked="checkedIds.has(row.id)" @change="toggleCheck(row.id)" /></td>
              <td><input v-if="editForms[row.id]" v-model="editForms[row.id].item"     class="cell-input req" placeholder="品檢項目*" /></td>
              <td><input v-if="editForms[row.id]" v-model="editForms[row.id].standard" class="cell-input" placeholder="標準值" /></td>
              <td><input v-if="editForms[row.id]" v-model="editForms[row.id].actual"   class="cell-input" placeholder="實測值" /></td>
              <td>
                <select v-if="editForms[row.id]" v-model="editForms[row.id].judgment" class="cell-input cell-select">
                  <option value="">—</option>
                  <option value="pass">合格</option>
                  <option value="fail">不合格</option>
                </select>
              </td>
              <td><input v-if="editForms[row.id]" v-model="editForms[row.id].remark" class="cell-input" placeholder="備註" /></td>
            </tr>

            <!-- 新增輸入列 -->
            <tr class="new-row" :class="{ 'new-row-checked': newRow.checked }">
              <td><input type="checkbox" v-model="newRow.checked" /></td>
              <td><input v-model="newRow.item"     class="cell-input req" placeholder="品檢項目*" /></td>
              <td><input v-model="newRow.standard" class="cell-input" placeholder="標準值" /></td>
              <td><input v-model="newRow.actual"   class="cell-input" placeholder="實測值" /></td>
              <td>
                <select v-model="newRow.judgment" class="cell-input cell-select">
                  <option value="">—</option>
                  <option value="pass">合格</option>
                  <option value="fail">不合格</option>
                </select>
              </td>
              <td><input v-model="newRow.remark" class="cell-input" placeholder="備註" /></td>
            </tr>

            <tr v-if="!currentRows.length">
              <td colspan="6" class="hint-row">請在最下方輸入列填寫品檢資料，勾選後點擊「新增」</td>
            </tr>
          </tbody>
        </table>
      </div>
    </template>
  </div>
</template>

<script setup>
import { ref, reactive, computed, watch } from 'vue'

const props = defineProps({
  lots:        { type: Array,  default: () => [] },
  qualityData: { type: Object, default: () => ({}) },
})
const emit = defineEmits(['add', 'update', 'delete-items'])

const selectedLot     = ref('')
const selectedLotInfo = computed(() => props.lots.find(l => l.lotNo === selectedLot.value) ?? null)
const currentRows     = computed(() => props.qualityData[selectedLot.value] ?? [])

// 每列的本地編輯表單
const editForms = reactive({})

watch(currentRows, (newRows) => {
  newRows.forEach(row => {
    if (!editForms[row.id]) editForms[row.id] = { ...row }
  })
  const ids = new Set(newRows.map(r => r.id))
  Object.keys(editForms).forEach(k => {
    if (!ids.has(Number(k))) delete editForms[k]
  })
}, { immediate: true, deep: true })

// 勾選
const checkedIds = reactive(new Set())
function toggleCheck(id) {
  if (checkedIds.has(id)) checkedIds.delete(id)
  else checkedIds.add(id)
}

// 新增輸入列
function emptyForm() { return { item:'', standard:'', actual:'', judgment:'', remark:'' } }
const newRow = reactive({ checked: false, ...emptyForm() })
function resetNew() { Object.assign(newRow, { checked: false, ...emptyForm() }) }

// 新增
function handleAdd() {
  if (!newRow.checked) { alert('請先勾選新增列的方框'); return }
  if (!newRow.item)    { alert('品檢項目為必填欄位'); return }
  emit('add', { lotNo: selectedLot.value, row: { ...emptyForm(), ...newRow } })
  resetNew()
}

// 修改（直接用各列的 editForm 覆蓋）
function handleEdit() {
  if (!checkedIds.size) return
  let hasError = false
  checkedIds.forEach(id => {
    const form = editForms[id]
    if (!form || !form.item) { hasError = true; return }
    const idx = currentRows.value.findIndex(r => r.id === id)
    if (idx >= 0) emit('update', { lotNo: selectedLot.value, idx, row: { ...form } })
  })
  if (hasError) { alert('勾選列中有品檢項目未填，請確認後再修改'); return }
  checkedIds.clear()
}

// 刪除
function handleDelete() {
  if (!checkedIds.size) return
  if (!confirm(`確定要刪除已勾選的 ${checkedIds.size} 筆品檢資料？`)) return
  emit('delete-items', { lotNo: selectedLot.value, ids: [...checkedIds] })
  checkedIds.clear()
}
</script>

<style scoped>
.lot-bar {
  background: #d8e8f8; border-bottom: 1px solid #b0c4de;
  padding: 7px 14px; display: flex; align-items: center; gap: 10px; flex-wrap: wrap;
}
.lot-label { background: #3a6abf; color: #fff; padding: 2px 10px; border-radius: 3px; font-size: 12px; white-space: nowrap; }
.lot-select { padding: 3px 8px; border: 1px solid #b0c4de; border-radius: 3px; font-size: 13px; font-family: inherit; background: #fff; min-width: 280px; }
.lot-info   { font-size: 12px; color: #2a5aaf; background: #e8f0fc; padding: 2px 10px; border-radius: 3px; }
.no-lot-hint { padding: 40px; text-align: center; color: #aaa; font-size: 13px; font-style: italic; }

.section-bar {
  background: linear-gradient(90deg, #3a6abf, #5586d0);
  color: #fff; padding: 6px 14px;
  display: flex; align-items: center; gap: 8px;
  border-bottom: 1px solid #2a5aaf;
}
.section-title { font-weight: bold; font-size: 13px; min-width: 60px; margin-right: 4px; }
.table-wrap { padding: 8px 12px; overflow-x: auto; }

.dtable { width: 100%; border-collapse: collapse; font-size: 12px; min-width: 560px; }
.dtable th { background: #3a6abf; color: #fff; padding: 5px 8px; border: 1px solid #2a5aaf; white-space: nowrap; text-align: center; }
.dtable td { padding: 3px 6px; border: 1px solid #d0dce8; text-align: center; }
.dtable tr:nth-child(even) td { background: #f0f6ff; }
.dtable tr:hover td           { background: #dceeff; }
.row-checked td      { background: #d4e8ff !important; }
.new-row td          { background: #f0fff4; }
.new-row-checked td  { background: #d0f0e0 !important; }
.hint-row { color: #aaa; text-align: center; padding: 12px; font-style: italic; }

.cell-input { width: 100%; padding: 2px 5px; border: 1px solid #b0c4de; border-radius: 2px; font-size: 12px; font-family: inherit; background: #fff; box-sizing: border-box; }
.cell-input:focus { border-color: #3a6abf; outline: none; }
.cell-input.req   { border-color: #c0392b; }
.cell-select      { cursor: pointer; }

.btn { padding: 3px 10px; font-size: 12px; font-family: inherit; border: 1px solid; border-radius: 3px; cursor: pointer; display: inline-flex; align-items: center; gap: 4px; }
.btn:hover    { filter: brightness(1.1); }
.btn:disabled { opacity: .4; cursor: not-allowed; filter: none; }
.btn-primary { background: #3a6abf; color: #fff; border-color: #2a5aaf; }
.btn-warn    { background: #e67e22; color: #fff; border-color: #ca6f1e; }
.btn-danger  { background: #c0392b; color: #fff; border-color: #a93226; }
.badge { background: rgba(255,255,255,.9); color: #c0392b; font-size: 11px; font-weight: bold; padding: 0 5px; border-radius: 8px; min-width: 16px; text-align: center; }
</style>
