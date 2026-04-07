<template>
  <div class="erp-app">
    <!-- 頁首 -->
    <div class="page-header">
      <div class="header-title">☆ 塊材申請作業</div>
      <div class="header-info">
        <span>{{ currentDate }}</span>
        <span>系統管理員 (ICSCIG)</span>
      </div>
    </div>

    <!-- 查詢列 -->
    <div class="query-bar">
      <span class="query-label">申請單單號</span>
      <div class="query-input-wrap">
        <span class="query-icon">🔍</span>
        <input
          v-model="queryNo"
          type="text"
          class="query-input"
          placeholder="請輸入申請單單號"
          @keyup.enter="doQuery"
        />
      </div>
      <button class="btn btn-query" @click="doQuery">查詢</button>
    </div>

    <!-- 訊息列 -->
    <div class="message-bar">
      <span class="msg-label">訊息</span>
      <span :class="['msg-text', message.type]">{{ message.text }}</span>
    </div>

    <!-- 頁籤導覽（依委託類型動態顯示） -->
    <div class="tab-nav">
      <button
        v-for="idx in visibleTabIndices"
        :key="idx"
        :class="['tab-btn', { active: activeTab === idx }]"
        @click="activeTab = idx"
      >{{ tabs[idx] }}</button>
    </div>

    <!-- 頁籤內容 -->
    <div class="tab-content">

      <!-- ===== 0. D5申請單 ===== -->
      <div v-show="activeTab === 0" class="tab-panel">
        <div class="panel-toolbar">
          <span class="toolbar-label">功能</span>
          <div class="toolbar-id" v-if="d5.id">單號：{{ d5.id }}</div>
          <button v-if="!d5.id" class="btn btn-primary" @click="d5Create">新增</button>
          <template v-else>
            <button class="btn btn-success" @click="d5Confirm">確認轉訂單</button>
            <span v-if="d5.confirmed" class="status-badge confirmed">已確認轉訂單</span>
          </template>
        </div>
        <div class="form-grid">
          <div class="form-row">
            <div class="form-cell lbl req-lbl">交期 *</div>
            <div class="form-cell inp">
              <input type="date" v-model="d5.deliveryDate" class="f-input" />
            </div>
            <div class="form-cell lbl req-lbl">圖號 *</div>
            <div class="form-cell inp">
              <input type="text" v-model="d5.drawingNo" class="f-input" placeholder="請輸入圖號" />
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">附件</div>
            <div class="form-cell inp c3">
              <input type="file" class="f-input" @change="d5FileChange" />
              <span v-if="d5.attachmentName" class="file-name">{{ d5.attachmentName }}</span>
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">庫存是否足夠</div>
            <div class="form-cell inp">
              <label class="chk-label">
                <input type="checkbox" v-model="d5.stockSufficient" />
                <span>確認庫存足夠</span>
              </label>
            </div>
            <div class="form-cell lbl">委託類型</div>
            <div class="form-cell inp">
              <label class="chk-label"><input type="checkbox" v-model="d5.typeMachining" /><span>機加工</span></label>
              <label class="chk-label"><input type="checkbox" v-model="d5.typeCoating" /><span>鍍膜</span></label>
              <label class="chk-label"><input type="checkbox" v-model="d5.typePurification" /><span>純化</span></label>
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">爐次</div>
            <div class="form-cell inp">
              <input type="text" v-model="d5.furnaceNo" class="f-input" placeholder="請輸入爐次" />
            </div>
            <div class="form-cell lbl">費用</div>
            <div class="form-cell inp">
              <input type="number" v-model="d5.cost" class="f-input" placeholder="0" />
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">客戶</div>
            <div class="form-cell inp c3">
              <input type="text" v-model="d5.customer" class="f-input" placeholder="請輸入客戶名稱" />
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">備註</div>
            <div class="form-cell inp c3">
              <textarea v-model="d5.remark" class="f-textarea" rows="2" placeholder="請輸入備註"></textarea>
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl sys-lbl">系統備註</div>
            <div class="form-cell inp c3">
              <div class="sys-log">
                <div v-if="!d5.sysLog.length" class="log-empty">— 尚無記錄 —</div>
                <div v-for="(entry, i) in d5.sysLog" :key="i" :class="['log-entry', entry.level]">
                  <span class="log-time">{{ entry.time }}</span>
                  <span class="log-dot">›</span>
                  <span class="log-msg">{{ entry.msg }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- ===== 1. 訂單 ===== -->
      <div v-show="activeTab === 1" class="tab-panel">
        <div class="panel-toolbar">
          <span class="toolbar-label">功能</span>
          <button class="btn btn-primary" @click="orderSave">儲存</button>
        </div>
        <div class="form-grid">
          <div class="form-row">
            <div class="form-cell sec-header c4">從 D5 申請單帶入資訊</div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">交期</div>
            <div class="form-cell inp"><input type="date" v-model="d5.deliveryDate" disabled class="f-input" /></div>
            <div class="form-cell lbl">圖號</div>
            <div class="form-cell inp"><input type="text" v-model="d5.drawingNo" disabled class="f-input" /></div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">爐次</div>
            <div class="form-cell inp"><input type="text" v-model="d5.furnaceNo" disabled class="f-input" /></div>
            <div class="form-cell lbl">費用</div>
            <div class="form-cell inp"><input type="number" v-model="d5.cost" disabled class="f-input" /></div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">客戶</div>
            <div class="form-cell inp c3"><input type="text" v-model="d5.customer" disabled class="f-input" /></div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">委託類型</div>
            <div class="form-cell inp c3">
              <span class="tag" v-if="d5.typeMachining">機加工</span>
              <span class="tag" v-if="d5.typeCoating">鍍膜</span>
              <span class="tag" v-if="d5.typePurification">純化</span>
              <span v-if="!d5.typeMachining && !d5.typeCoating && !d5.typePurification" style="color:#aaa">（未選取）</span>
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell sec-header c4">訂單專屬資訊</div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl req-lbl">數量 *</div>
            <div class="form-cell inp"><input type="number" v-model="order.qty" class="f-input req-inp" placeholder="請輸入數量" /></div>
            <div class="form-cell lbl"></div>
            <div class="form-cell inp"></div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">訂單備註</div>
            <div class="form-cell inp c3">
              <textarea v-model="order.remark" class="f-textarea" rows="3" placeholder="請輸入訂單備註"></textarea>
            </div>
          </div>
        </div>
      </div>

      <!-- ===== 2. 機加工(耗用) ===== -->
      <div v-show="activeTab === 2" class="tab-panel">
        <ConsumptionTab
          :show-self="true" :show-outsource="true"
          :self-rows="machining.consumption.self"
          :outsource-rows="machining.consumption.outsource"
          @open-picker="openPicker($event, 'machining')"
          @cancel-rows="cancelConsumption('machining', $event)"
        />
      </div>

      <!-- ===== 3. 機加工(產出) ===== -->
      <div v-show="activeTab === 3" class="tab-panel">
        <OutputTab
          :show-self="true" :show-outsource="true"
          :self-rows="machining.output.self"
          :outsource-rows="machining.output.outsource"
          @add="handleOutputAdd('machining', $event)"
          @update="handleOutputUpdate('machining', $event)"
          @delete-items="handleOutputDelete('machining', $event)"
        />
      </div>

      <!-- ===== 4. 鍍膜(耗用) ===== -->
      <div v-show="activeTab === 4" class="tab-panel">
        <ConsumptionTab
          :show-self="false" :show-outsource="true"
          :self-rows="coating.consumption.self"
          :outsource-rows="coating.consumption.outsource"
          @open-picker="openPicker($event, 'coating')"
          @cancel-rows="cancelConsumption('coating', $event)"
        />
      </div>

      <!-- ===== 5. 鍍膜(產出) ===== -->
      <div v-show="activeTab === 5" class="tab-panel">
        <OutputTab
          :show-self="false" :show-outsource="true"
          :self-rows="coating.output.self"
          :outsource-rows="coating.output.outsource"
          @add="handleOutputAdd('coating', $event)"
          @update="handleOutputUpdate('coating', $event)"
          @delete-items="handleOutputDelete('coating', $event)"
        />
      </div>

      <!-- ===== 6. 純化(耗用) ===== -->
      <div v-show="activeTab === 6" class="tab-panel">
        <ConsumptionTab
          :show-self="true" :show-outsource="false"
          :self-rows="purification.consumption.self"
          :outsource-rows="purification.consumption.outsource"
          @open-picker="openPicker($event, 'purification')"
          @cancel-rows="cancelConsumption('purification', $event)"
        />
      </div>

      <!-- ===== 7. 純化(產出) ===== -->
      <div v-show="activeTab === 7" class="tab-panel">
        <OutputTab
          :show-self="true" :show-outsource="false"
          :self-rows="purification.output.self"
          :outsource-rows="purification.output.outsource"
          @add="handleOutputAdd('purification', $event)"
          @update="handleOutputUpdate('purification', $event)"
          @delete-items="handleOutputDelete('purification', $event)"
        />
      </div>

      <!-- ===== 8. 品檢資訊 ===== -->
      <div v-show="activeTab === 8" class="tab-panel">
        <QualityTab
          :lots="allLots"
          :quality-data="qualityData"
          @add="handleQualityAdd($event)"
          @update="handleQualityUpdate($event)"
          @delete-items="handleQualityDelete($event)"
        />
      </div>

    </div>

    <!-- ===== 庫存挑選 Modal（多選） ===== -->
    <div v-if="picker.show" class="modal-overlay" @click.self="picker.show = false">
      <div class="modal">
        <div class="modal-hdr">
          <span>挑選庫存</span>
          <span class="picker-count" v-if="picker.selIds.size">已選 {{ picker.selIds.size }} 筆</span>
          <button class="modal-x" @click="picker.show = false">✕</button>
        </div>
        <div class="modal-body">
          <div style="margin-bottom:8px">
            <input v-model="picker.keyword" type="text" class="f-input" placeholder="搜尋品號 / 品名..." @input="filterInv" />
          </div>
          <table class="dtable">
            <thead>
              <tr>
                <th width="36">
                  <input type="checkbox"
                    :checked="filteredInv.length > 0 && picker.selIds.size === filteredInv.length"
                    :indeterminate="picker.selIds.size > 0 && picker.selIds.size < filteredInv.length"
                    @change="pickerToggleAll($event.target.checked)" />
                </th>
                <th>品號</th><th>品名</th><th>倉庫</th><th>批號</th><th>可用數量</th><th>單位</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(inv, i) in filteredInv" :key="i"
                  :class="{ 'row-sel': picker.selIds.has(i) }"
                  @click="pickerToggleOne(i)">
                <td @click.stop>
                  <input type="checkbox" :checked="picker.selIds.has(i)" @change="pickerToggleOne(i)" />
                </td>
                <td>{{ inv.itemNo }}</td><td>{{ inv.itemName }}</td>
                <td>{{ inv.warehouse }}</td><td>{{ inv.lotNo }}</td>
                <td>{{ inv.qty }}</td><td>{{ inv.unit }}</td>
              </tr>
              <tr v-if="!filteredInv.length">
                <td colspan="7" class="empty-row">查無庫存資料</td>
              </tr>
            </tbody>
          </table>
        </div>
        <div class="modal-ftr">
          <button class="btn btn-success" @click="confirmPicker">確認耗用（{{ picker.selIds.size }} 筆）</button>
          <button class="btn btn-default" @click="picker.show = false">取消</button>
        </div>
      </div>
    </div>


  </div>
</template>

<script setup>
import { ref, reactive, computed, watch } from 'vue'
import ConsumptionTab from './components/ConsumptionTab.vue'
import OutputTab from './components/OutputTab.vue'
import QualityTab from './components/QualityTab.vue'

// 日期
const now = new Date()
const days = ['日','一','二','三','四','五','六']
const currentDate = `${now.getFullYear()}/${String(now.getMonth()+1).padStart(2,'0')}/${String(now.getDate()).padStart(2,'0')} (${days[now.getDay()]})`

// 頁籤
// 全部頁籤定義（固定索引）
// 頁籤
const tabs = ['D5申請單','訂單','機加工(耗用)','機加工(產出)','鍍膜(耗用)','鍍膜(產出)','純化(耗用)','純化(產出)','品檢資訊']
const activeTab = ref(0)

// 訊息
const message = reactive({ text: '歡迎使用本系統...', type: 'info' })
const showMsg = (text, type='info') => { message.text = text; message.type = type }

// 查詢
const queryNo = ref('')
function doQuery() {
  const q = queryNo.value.trim()
  if (!q) { showMsg('請輸入申請單單號', 'error'); return }
  if (d5.id && d5.id.toUpperCase() === q.toUpperCase()) {
    activeTab.value = 0
    showMsg(`查詢成功：已載入申請單 ${d5.id}`, 'success')
  } else {
    showMsg(`查無申請單：${q}`, 'error')
  }
}

// D5 資料（欄位直接開放輸入，按新增完成建檔）
const d5 = reactive({
  id: null, confirmed: false,
  deliveryDate:'', drawingNo:'', attachmentName:'',
  stockSufficient:false, typeMachining:false, typeCoating:false, typePurification:false,
  furnaceNo:'', cost:null, customer:'', remark:'', sysLog:[]
})

// 製程 / 區塊 中文對照
const PROC_NAME = { machining:'機加工', coating:'鍍膜', purification:'純化' }
const SEC_NAME  = { self:'自產', outsource:'委外' }

// 寫入系統 LOG
function addLog(msg, level = 'info') {
  const now = new Date()
  const time = `${now.getFullYear()}/${String(now.getMonth()+1).padStart(2,'0')}/${String(now.getDate()).padStart(2,'0')} ${String(now.getHours()).padStart(2,'0')}:${String(now.getMinutes()).padStart(2,'0')}:${String(now.getSeconds()).padStart(2,'0')}`
  d5.sysLog.push({ time, msg, level })
}

// 依委託類型動態顯示頁籤（需在 d5 定義後）
const visibleTabIndices = computed(() => {
  const idx = [0, 1]
  if (d5.id) {
    if (d5.typeMachining)    idx.push(2, 3)
    if (d5.typeCoating)      idx.push(4, 5)
    if (d5.typePurification) idx.push(6, 7)
    idx.push(8) // 品檢資訊
  }
  return idx
})
watch(visibleTabIndices, (newList) => {
  if (!newList.includes(activeTab.value)) activeTab.value = 0
})
function d5Create() {
  if (!d5.deliveryDate || !d5.drawingNo) { showMsg('交期與圖號為必填欄位', 'error'); return }
  // 模擬後端自動建立單號
  const ym = `${new Date().getFullYear()}${String(new Date().getMonth()+1).padStart(2,'0')}`
  d5.id = `D5-${ym}-${String(Math.floor(Math.random()*9000)+1000)}`
  addLog(`建立 D5 申請單，單號：${d5.id}`, 'create')
  showMsg(`D5 申請單建檔完成，單號：${d5.id}`, 'success')
}
function d5Confirm() {
  if (!d5.deliveryDate || !d5.drawingNo) { showMsg('交期與圖號為必填欄位', 'error'); return }
  d5.confirmed = true
  addLog('申請單轉訂單', 'confirm')
  showMsg(`D5 申請單已確認轉訂單，單號：${d5.id}`, 'success')
  activeTab.value = 1
}
function d5FileChange(e) { d5.attachmentName = e.target.files[0]?.name || '' }

// 訂單
const order = reactive({ qty:null, remark:'' })
function orderSave() {
  if (!d5.id) { showMsg('請先完成 D5 申請單並確認轉訂單', 'error'); return }
  if (!order.qty) { showMsg('數量為必填欄位', 'error'); return }
  addLog(`儲存訂單，數量：${order.qty}`, 'info')
  showMsg('訂單資料已儲存', 'success')
}

// 各製程資料
function mkProc() { return { consumption:{self:[], outsource:[]}, output:{self:[], outsource:[]} } }
const machining    = reactive(mkProc())
const coating      = reactive(mkProc())
const purification = reactive(mkProc())
const getProc = n => ({machining,coating,purification}[n])

// 庫存資料
const allInv = [
  { itemNo:'ACS15U', itemName:'超級電容ACS15U',   warehouse:'A小港廠', lotNo:'L-001', qty:500, unit:'kg'  },
  { itemNo:'GC-001', itemName:'石墨化碳材料',      warehouse:'A小港廠', lotNo:'L-002', qty:200, unit:'kg'  },
  { itemNo:'CB-050', itemName:'碳微球CB-050',      warehouse:'B倉庫',   lotNo:'L-003', qty:150, unit:'kg'  },
  { itemNo:'PG-100', itemName:'純化石墨PG-100',    warehouse:'A小港廠', lotNo:'L-004', qty:80,  unit:'kg'  },
  { itemNo:'MC-200', itemName:'機加工碳塊MC-200',  warehouse:'C倉庫',   lotNo:'L-005', qty:320, unit:'pcs' },
]
const filteredInv = ref([...allInv])
const picker = reactive({ show:false, keyword:'', selIds: new Set(), proc:'', section:'' })
function filterInv() {
  const kw = picker.keyword.toLowerCase()
  filteredInv.value = allInv.filter(i => i.itemNo.toLowerCase().includes(kw) || i.itemName.toLowerCase().includes(kw))
  picker.selIds.clear()
}
function pickerToggleOne(i) {
  if (picker.selIds.has(i)) picker.selIds.delete(i)
  else picker.selIds.add(i)
}
function pickerToggleAll(checked) {
  picker.selIds.clear()
  if (checked) filteredInv.value.forEach((_, i) => picker.selIds.add(i))
}
function openPicker({ section }, procName) {
  picker.proc = procName; picker.section = section
  picker.keyword = ''; picker.selIds.clear(); filteredInv.value = [...allInv]
  picker.show = true
}
function confirmPicker() {
  if (!picker.selIds.size) { showMsg('請先勾選庫存項目', 'error'); return }
  const arr = getProc(picker.proc).consumption[picker.section]
  const names = []
  picker.selIds.forEach(i => {
    const inv = filteredInv.value[i]
    arr.push({ id: Date.now() + i, ...inv })
    names.push(inv.itemName)
  })
  const count = picker.selIds.size
  picker.show = false
  addLog(`輸入【${PROC_NAME[picker.proc]}(耗用)】${SEC_NAME[picker.section]}：${names.join('、')}（${count} 筆）`, 'input')
  showMsg(`已確認耗用 ${count} 筆：${names.join('、')}`, 'success')
  picker.selIds.clear()
}
function cancelConsumption(procName, { section, ids }) {
  const idSet = new Set(ids)
  const arr = getProc(procName).consumption[section]
  arr.splice(0, arr.length, ...arr.filter(r => !idSet.has(r.id)))
  addLog(`取消【${PROC_NAME[procName]}(耗用)】${SEC_NAME[section]}：${idSet.size} 筆`, 'cancel')
  showMsg(`已取消 ${idSet.size} 筆耗用`, 'info')
}

// 產出明細處理（行內新增/修改/刪除）
function handleOutputAdd(proc, { section, row }) {
  getProc(proc).output[section].push({ ...row, id: Date.now() })
  const label = row.productKey || '（未知品名）'
  addLog(`輸入【${PROC_NAME[proc]}(產出)】${SEC_NAME[section]}：${label}，數量 ${row.qty}`, 'input')
  showMsg('產出明細已新增', 'success')
}
function handleOutputUpdate(proc, { section, idx, row }) {
  const arr = getProc(proc).output[section]
  arr[idx] = { ...row, id: arr[idx].id }
  addLog(`修改【${PROC_NAME[proc]}(產出)】${SEC_NAME[section]}：${row.productKey}，數量 ${row.qty}`, 'edit')
  showMsg('產出明細已修改', 'success')
}
// 品檢資訊
const qualityData = reactive({}) // { lotNo: [{ id, item, standard, actual, judgment, remark }] }

const allLots = computed(() => {
  const lots = [], seen = new Set()
  const collect = (proc, procName) => {
    ['self','outsource'].forEach(sec => {
      getProc(proc).output[sec].forEach(row => {
        if (row.lotNo && !seen.has(row.lotNo)) {
          seen.add(row.lotNo)
          lots.push({ lotNo: row.lotNo, productKey: row.productKey, procName, section: SEC_NAME[sec] })
        }
      })
    })
  }
  collect('machining', '機加工')
  collect('coating',   '鍍膜')
  collect('purification', '純化')
  return lots
})

function handleQualityAdd({ lotNo, row }) {
  if (!qualityData[lotNo]) qualityData[lotNo] = []
  qualityData[lotNo].push({ ...row, id: Date.now() })
  addLog(`輸入【品檢資訊】批號 ${lotNo}：${row.item}`, 'input')
  showMsg(`品檢資料已新增：${row.item}`, 'success')
}
function handleQualityUpdate({ lotNo, idx, row }) {
  const arr = qualityData[lotNo]
  if (arr) { arr[idx] = { ...row, id: arr[idx].id }; addLog(`修改【品檢資訊】批號 ${lotNo}：${row.item}`, 'edit'); showMsg('品檢資料已修改', 'success') }
}
function handleQualityDelete({ lotNo, ids }) {
  const idSet = new Set(ids)
  if (!qualityData[lotNo]) return
  const removed = idSet.size
  qualityData[lotNo].splice(0, qualityData[lotNo].length, ...qualityData[lotNo].filter(r => !idSet.has(r.id)))
  addLog(`刪除【品檢資訊】批號 ${lotNo}：${removed} 筆`, 'cancel')
  showMsg(`已刪除 ${removed} 筆品檢資料`, 'info')
}

function handleOutputDelete(proc, { section, ids }) {
  const idSet = new Set(ids)
  const arr = getProc(proc).output[section]
  const removed = idSet.size
  arr.splice(0, arr.length, ...arr.filter(r => !idSet.has(r.id)))
  addLog(`刪除【${PROC_NAME[proc]}(產出)】${SEC_NAME[section]}：${removed} 筆`, 'cancel')
  showMsg(`已刪除 ${removed} 筆產出明細`, 'info')
}
</script>

<style scoped>
.erp-app { min-height:100vh; display:flex; flex-direction:column; background:#dce6f0; font-size:13px; }

/* 頁首 */
.page-header {
  background: linear-gradient(135deg, #1a3a6e 0%, #2a5aab 100%);
  color:#fff; padding:8px 16px;
  display:flex; align-items:center; justify-content:space-between;
  box-shadow:0 2px 6px rgba(0,0,0,.4);
}
.header-title { font-size:16px; font-weight:bold; letter-spacing:1px; }
.header-info  { font-size:12px; display:flex; gap:16px; opacity:.9; }

/* 查詢列 */
.query-bar {
  background: #c8d8ec;
  border-bottom: 1px solid #a0b8d8;
  padding: 5px 16px;
  display: flex;
  align-items: center;
  gap: 8px;
}
.query-label {
  background: #3a6abf;
  color: #fff;
  padding: 2px 10px;
  border-radius: 3px;
  font-size: 12px;
  white-space: nowrap;
}
.query-input-wrap {
  display: flex;
  align-items: center;
  border: 1px solid #8aaad0;
  border-radius: 3px;
  background: #fff;
  overflow: hidden;
  width: 220px;
}
.query-icon { padding: 0 6px; font-size: 12px; color: #7a9ac8; }
.query-input {
  flex: 1;
  border: none;
  outline: none;
  padding: 3px 4px;
  font-size: 13px;
  font-family: inherit;
  background: transparent;
}
.btn-query {
  background: #3a6abf;
  color: #fff;
  border: 1px solid #2a5aaf;
  padding: 3px 14px;
  font-size: 12px;
  font-family: inherit;
  border-radius: 3px;
  cursor: pointer;
}
.btn-query:hover { filter: brightness(1.1); }

/* 訊息列 */
.message-bar {
  background:#c8d8ec; border-bottom:1px solid #a0b8d8;
  padding:4px 16px; display:flex; align-items:center; gap:8px; min-height:28px;
}
.msg-label { background:#3a6abf; color:#fff; padding:1px 8px; border-radius:3px; font-size:12px; white-space:nowrap; }
.msg-text { font-size:13px; }
.msg-text.info    { color:#1a3a6e; }
.msg-text.success { color:#155724; font-weight:bold; }
.msg-text.error   { color:#c0392b; font-weight:bold; }

/* 頁籤導覽 */
.tab-nav {
  display:flex; flex-wrap:wrap; background:#b0c8e8;
  border-bottom:2px solid #3a6abf; padding:6px 8px 0; gap:3px;
}
.tab-btn {
  padding:5px 14px; font-size:13px; font-family:inherit;
  border:1px solid #7a9ac8; border-bottom:none; border-radius:4px 4px 0 0;
  background:#d0e0f0; color:#1a3a6e; cursor:pointer; transition:background .15s;
}
.tab-btn:hover  { background:#e0ecf8; }
.tab-btn.active { background:#fff; font-weight:bold; border-color:#3a6abf; }

/* 頁籤內容 */
.tab-content { flex:1; padding:12px; }
.tab-panel   { background:#fff; border:1px solid #b0c4de; border-radius:0 4px 4px 4px; }

/* 工具列 */
.panel-toolbar {
  background:#d8e8f8; border-bottom:1px solid #b0c4de;
  padding:6px 12px; display:flex; align-items:center; gap:6px; flex-wrap:wrap;
}
.toolbar-label { background:#3a6abf; color:#fff; padding:2px 10px; border-radius:3px; font-size:12px; margin-right:4px; }

/* 按鈕 */
.btn { padding:3px 12px; font-size:12px; font-family:inherit; border:1px solid; border-radius:3px; cursor:pointer; transition:filter .15s; }
.btn:hover    { filter:brightness(1.1); }
.btn:disabled { opacity:.4; cursor:not-allowed; filter:none; }
.btn-primary { background:#3a6abf; color:#fff; border-color:#2a5aaf; }
.btn-default { background:#e8e8e8; color:#333; border-color:#bbb; }
.btn-success { background:#27ae60; color:#fff; border-color:#1e8449; }
.btn-danger  { background:#c0392b; color:#fff; border-color:#a93226; }

/* 表單格線 */
.form-grid { padding:8px 12px; display:flex; flex-direction:column; gap:0; }
.form-row  { display:grid; grid-template-columns:140px 1fr 140px 1fr; border-bottom:1px solid #e0eaf5; min-height:32px; }
.form-cell { padding:4px 8px; display:flex; align-items:center; }
.lbl { background:#c8daf0; color:#1a3a6e; font-weight:bold; font-size:12px; border-right:1px solid #b0c4de; white-space:nowrap; }
.inp { background:#f5f9ff; }
.c3  { grid-column:span 3; }
.c4  { grid-column:span 4; }
.sys-lbl { background:#d8d8d8; color:#555; }

/* 系統 LOG */
.sys-log {
  width: 100%; max-height: 110px; overflow-y: auto;
  background: #f0f0f0; border: 1px solid #d0d8e0; border-radius: 3px;
  padding: 5px 8px; font-size: 12px;
  display: flex; flex-direction: column; gap: 1px;
}
.log-empty { color: #aaa; font-style: italic; text-align: center; padding: 4px; }
.log-entry { display: flex; align-items: baseline; gap: 6px; line-height: 1.7; border-bottom: 1px solid #e4e8ee; }
.log-entry:last-child { border-bottom: none; }
.log-time  { color: #7a8fa8; white-space: nowrap; font-size: 11px; flex-shrink: 0; }
.log-dot   { color: #bbb; }
.log-msg   { color: #333; word-break: break-all; }

/* log 等級色彩 */
.log-entry.create  .log-msg { color: #1a7a40; }
.log-entry.confirm .log-msg { color: #7a5500; font-weight: bold; }
.log-entry.input   .log-msg { color: #1a4a8a; }
.log-entry.edit    .log-msg { color: #6a3a9a; }
.log-entry.cancel  .log-msg { color: #aa2222; }
.log-entry.info    .log-msg { color: #333; }
.sec-header { background:#4a7abf; color:#fff; font-weight:bold; font-size:12px; padding:4px 12px; }
.req-lbl { color:#c0392b; }

/* 輸入元件 */
.f-input {
  width:100%; padding:3px 6px; border:1px solid #b0c4de; border-radius:3px;
  font-size:13px; font-family:inherit; background:#fff;
}
.f-input:disabled { background:#edf3fa; color:#555; }
.f-input:focus    { outline:none; border-color:#3a6abf; box-shadow:0 0 0 2px rgba(58,106,191,.2); }
.req-inp { border-color:#c0392b; }
.f-textarea {
  width:100%; padding:4px 6px; border:1px solid #b0c4de; border-radius:3px;
  font-size:13px; font-family:inherit; resize:vertical;
}
.f-textarea:disabled { background:#edf3fa; }
.sys-ta { background:#f0f0f0 !important; color:#666; }

.chk-label { display:flex; align-items:center; gap:4px; margin-right:12px; cursor:pointer; }
.chk-label input { width:14px; height:14px; }
.tag { display:inline-block; background:#3a6abf; color:#fff; padding:1px 8px; border-radius:10px; font-size:12px; margin-right:6px; }
.file-name { font-size:12px; color:#666; margin-left:8px; }
.toolbar-id { font-size:12px; color:#1a3a6e; font-weight:bold; background:#dceeff; padding:2px 10px; border-radius:3px; border:1px solid #7aaad8; }
.status-badge { font-size:12px; padding:2px 10px; border-radius:10px; }
.status-badge.confirmed { background:#d4edda; color:#155724; border:1px solid #c3e6cb; font-weight:bold; }

/* 資料表格 */
.dtable { width:100%; border-collapse:collapse; font-size:12px; }
.dtable th { background:#3a6abf; color:#fff; padding:5px 8px; border:1px solid #2a5aaf; white-space:nowrap; text-align:center; }
.dtable td { padding:4px 8px; border:1px solid #d0dce8; text-align:center; white-space:nowrap; }
.dtable tr:nth-child(even) td { background:#f0f6ff; }
.dtable tr:hover td           { background:#dceeff; cursor:pointer; }
.dtable tr.row-sel td         { background:#c8e0ff; font-weight:bold; }
.empty-row { color:#999; padding:16px; text-align:center; }

/* Modal */
.modal-overlay { position:fixed; inset:0; background:rgba(0,0,0,.45); display:flex; align-items:center; justify-content:center; z-index:1000; }
.modal    { background:#fff; border-radius:6px; box-shadow:0 8px 32px rgba(0,0,0,.35); min-width:700px; max-width:92vw; max-height:86vh; display:flex; flex-direction:column; }
.modal-sm { min-width:440px; }
.modal-hdr { background:linear-gradient(135deg,#1a3a6e,#2855a0); color:#fff; padding:8px 16px; border-radius:6px 6px 0 0; display:flex; align-items:center; gap:10px; font-weight:bold; }
.picker-count { font-size:12px; background:rgba(255,255,255,.2); padding:1px 8px; border-radius:8px; margin-left:4px; }
.modal-hdr .modal-x { margin-left:auto; }
.modal-x  { background:none; border:none; color:#fff; font-size:16px; cursor:pointer; padding:0 4px; }
.modal-body { padding:12px; overflow-y:auto; flex:1; }
.modal-ftr  { padding:8px 16px; border-top:1px solid #d0dce8; display:flex; gap:8px; justify-content:flex-end; }
</style>
