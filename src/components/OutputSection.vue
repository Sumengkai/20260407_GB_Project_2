<template>
  <div>
    <!-- 區塊標題列 + 功能按鈕 -->
    <div :class="['section-bar', colorClass]">
      <span class="section-title">{{ label }}</span>
      <button class="btn btn-primary" @click="handleAdd">新增</button>
      <button class="btn btn-warn"
        :disabled="editingIdx < 0 && checkedIds.size !== 1"
        @click="handleEdit">
        {{ editingIdx >= 0 ? '儲存修改' : '修改' }}
      </button>
      <button class="btn btn-default" v-if="editingIdx >= 0" @click="cancelEdit">取消</button>
      <button class="btn btn-danger" :disabled="checkedIds.size === 0" @click="handleDelete">
        刪除<span v-if="checkedIds.size" class="badge">{{ checkedIds.size }}</span>
      </button>
    </div>

    <!-- 明細表格 -->
    <div class="table-wrap">
      <table class="dtable">
        <thead>
          <tr>
            <th width="36">勾選</th>
            <th>品名</th>
            <th>規格1</th><th>規格2</th><th>規格3</th>
            <th>數量</th><th>批號</th><th>備註</th>
          </tr>
        </thead>
        <tbody>
          <!-- 已確認的資料列 -->
          <tr v-for="(row, idx) in rows" :key="row.id"
              :class="{ 'row-checked': checkedIds.has(row.id), 'row-editing': editingIdx === idx }">
            <td>
              <input type="checkbox"
                :checked="checkedIds.has(row.id)"
                :disabled="editingIdx >= 0 && editingIdx !== idx"
                @change="toggleCheck(row.id)" />
            </td>
            <!-- 一般顯示模式 -->
            <template v-if="editingIdx !== idx">
              <td>{{ productLabel(row.productKey) }}</td>
              <td>{{ row.spec1 }}</td><td>{{ row.spec2 }}</td><td>{{ row.spec3 }}</td>
              <td>{{ row.qty }}</td><td>{{ row.lotNo }}</td><td>{{ row.remark }}</td>
            </template>
            <!-- 行內編輯模式 -->
            <template v-else>
              <td>
                <select v-model="editForm.productKey" class="cell-input cell-select req">
                  <option value="">請選擇品名</option>
                  <option v-for="p in PRODUCTS" :key="p.key" :value="p.key">{{ p.label }}</option>
                </select>
              </td>
              <td><input v-model="editForm.spec1" class="cell-input" placeholder="規格1" /></td>
              <td><input v-model="editForm.spec2" class="cell-input" placeholder="規格2" /></td>
              <td><input v-model="editForm.spec3" class="cell-input" placeholder="規格3" /></td>
              <td><input v-model="editForm.qty"   class="cell-input req" type="number" placeholder="數量*" /></td>
              <td><input v-model="editForm.lotNo" class="cell-input" placeholder="批號" /></td>
              <td><input v-model="editForm.remark" class="cell-input" placeholder="備註" /></td>
            </template>
          </tr>

          <!-- 新增輸入列（永遠顯示在最底） -->
          <tr class="new-row" :class="{ 'new-row-checked': newRow.checked }">
            <td>
              <input type="checkbox" v-model="newRow.checked" :disabled="editingIdx >= 0" />
            </td>
            <td>
              <select v-model="newRow.productKey" class="cell-input cell-select req" :disabled="editingIdx >= 0">
                <option value="">請選擇品名</option>
                <option v-for="p in PRODUCTS" :key="p.key" :value="p.key">{{ p.label }}</option>
              </select>
            </td>
            <td><input v-model="newRow.spec1"  class="cell-input" placeholder="規格1" :disabled="editingIdx >= 0" /></td>
            <td><input v-model="newRow.spec2"  class="cell-input" placeholder="規格2" :disabled="editingIdx >= 0" /></td>
            <td><input v-model="newRow.spec3"  class="cell-input" placeholder="規格3" :disabled="editingIdx >= 0" /></td>
            <td><input v-model="newRow.qty"    class="cell-input req" type="number" placeholder="數量*" :disabled="editingIdx >= 0" /></td>
            <td><input v-model="newRow.lotNo"  class="cell-input" placeholder="批號" :disabled="editingIdx >= 0" /></td>
            <td><input v-model="newRow.remark" class="cell-input" placeholder="備註" :disabled="editingIdx >= 0" /></td>
          </tr>

          <tr v-if="!rows.length">
            <td colspan="8" class="hint-row">請在最下方輸入列選擇品名，勾選後點擊「新增」</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'

const props = defineProps({
  label:      { type: String, default: '' },
  colorClass: { type: String, default: '' },
  rows:       { type: Array, default: () => [] },
})
const emit = defineEmits(['add', 'update', 'delete-items'])

// 品名下拉假資料（含品號）
const PRODUCTS = [
  { key: 'ACS15U', label: 'ACS15U - 超級電容ACS15U' },
  { key: 'GC-001', label: 'GC-001 - 石墨化碳材料' },
  { key: 'CB-050', label: 'CB-050 - 碳微球CB-050' },
  { key: 'PG-100', label: 'PG-100 - 純化石墨PG-100' },
  { key: 'MC-200', label: 'MC-200 - 機加工碳塊MC-200' },
  { key: 'CF-300', label: 'CF-300 - 碳纖維CF-300' },
  { key: 'GR-400', label: 'GR-400 - 石墨電極GR-400' },
  { key: 'BC-500', label: 'BC-500 - 電池碳材BC-500' },
]
function productLabel(key) {
  return PRODUCTS.find(p => p.key === key)?.label ?? key
}

// 勾選狀態
const checkedIds = reactive(new Set())
function toggleCheck(id) {
  if (checkedIds.has(id)) checkedIds.delete(id)
  else checkedIds.add(id)
}

// 空白表單
function emptyForm() {
  return { productKey:'', spec1:'', spec2:'', spec3:'', qty:null, lotNo:'', remark:'' }
}

// 新增輸入列
const newRow = reactive({ checked: false, ...emptyForm() })
function resetNew() { Object.assign(newRow, { checked: false, ...emptyForm() }) }

// 行內編輯
const editingIdx = ref(-1)
const editForm = reactive(emptyForm())

function handleAdd() {
  if (!newRow.checked)       { alert('請先勾選新增列的方框'); return }
  if (!newRow.productKey)    { alert('請選擇品名'); return }
  if (!newRow.qty)           { alert('數量為必填欄位'); return }
  emit('add', { ...emptyForm(), productKey:newRow.productKey,
    spec1:newRow.spec1, spec2:newRow.spec2, spec3:newRow.spec3,
    qty:newRow.qty, lotNo:newRow.lotNo, remark:newRow.remark })
  resetNew()
}

function handleEdit() {
  if (editingIdx.value >= 0) {
    if (!editForm.productKey) { alert('請選擇品名'); return }
    if (!editForm.qty)        { alert('數量為必填欄位'); return }
    emit('update', { idx: editingIdx.value, row: { ...editForm } })
    editingIdx.value = -1
    checkedIds.clear()
  } else {
    if (checkedIds.size !== 1) return
    const id  = [...checkedIds][0]
    const idx = props.rows.findIndex(r => r.id === id)
    if (idx < 0) return
    editingIdx.value = idx
    Object.assign(editForm, { ...props.rows[idx] })
  }
}

function cancelEdit() {
  editingIdx.value = -1
  checkedIds.clear()
}

function handleDelete() {
  if (!checkedIds.size) return
  if (!confirm(`確定要刪除已勾選的 ${checkedIds.size} 筆資料？`)) return
  emit('delete-items', [...checkedIds])
  checkedIds.clear()
}
</script>

<style scoped>
.section-bar {
  background: linear-gradient(90deg, #3a6abf, #5586d0);
  color: #fff; padding: 6px 14px;
  display: flex; align-items: center; gap: 8px;
  border-bottom: 1px solid #2a5aaf;
}
.section-bar.outsource { background: linear-gradient(90deg, #1a6e5a, #2a9e7a); border-color: #1a6e5a; }
.section-title { font-weight: bold; font-size: 13px; min-width: 40px; margin-right: 4px; }
.table-wrap { padding: 8px 12px; overflow-x: auto; }

.dtable { width:100%; border-collapse:collapse; font-size:12px; min-width: 700px; }
.dtable th { background:#3a6abf; color:#fff; padding:5px 8px; border:1px solid #2a5aaf; white-space:nowrap; text-align:center; }
.dtable td { padding:3px 6px; border:1px solid #d0dce8; text-align:center; }
.dtable tr:nth-child(even) td { background:#f0f6ff; }
.dtable tr:hover td           { background:#dceeff; }

.row-checked td  { background:#d4e8ff !important; }
.row-editing td  { background:#fff8e1 !important; }
.new-row td          { background:#f0fff4; }
.new-row-checked td  { background:#d0f0e0 !important; }

.cell-input {
  width: 100%; padding: 2px 5px;
  border: 1px solid #b0c4de; border-radius: 2px;
  font-size: 12px; font-family: inherit; background: #fff; box-sizing: border-box;
}
.cell-input:focus    { border-color: #3a6abf; outline: none; }
.cell-input.req      { border-color: #c0392b; }
.cell-input:disabled { background: #eee; }
.cell-select         { cursor: pointer; }

.hint-row { color:#aaa; text-align:center; padding:12px; font-style:italic; }

.btn { padding:3px 10px; font-size:12px; font-family:inherit; border:1px solid; border-radius:3px; cursor:pointer; display:inline-flex; align-items:center; gap:4px; }
.btn:hover    { filter:brightness(1.1); }
.btn:disabled { opacity:.4; cursor:not-allowed; filter:none; }
.btn-primary { background:#3a6abf; color:#fff; border-color:#2a5aaf; }
.btn-warn    { background:#e67e22; color:#fff; border-color:#ca6f1e; }
.btn-danger  { background:#c0392b; color:#fff; border-color:#a93226; }
.btn-default { background:#e8e8e8; color:#333; border-color:#bbb; }

.badge {
  background: rgba(255,255,255,.9); color: #c0392b;
  font-size: 11px; font-weight: bold;
  padding: 0 5px; border-radius: 8px; min-width: 16px; text-align: center;
}
</style>
