<template>
  <el-form ref="form" class="form" label-position="top">
    <div style="width: 100%;padding-left: 10px;border-left: 5px solid #2598f8;margin-bottom: 20px;padding-top: 5px;">{{
      $t('title') }}</div>
    <el-alert style="margin: 20px 0;color: #606266;" :title="$t('alerts.selectNumberField')" type="info" />
    <div class="helper-doc">
      <span>{{ $t('helpTip') }}</span>
      <span style="height: 16px;width: 16px;margin-left: 12px;">
        <a href="https://jfsq6znqku.feishu.cn/wiki/T4z9wWK88inr5zkQqhtcwUCSnSf?from=from_copylink"
          target="_blank"><span>说明文档</span></a>
      </span>

    </div>


    <el-form-item style="margin-top: 40px;" :label="$t('labels.link')" size="large" required>
      <el-select v-model="linkFieldId" :placeholder="$t('placeholder.link')" style="width: 100%">
        <el-option v-for="meta in mainFieldListSeView" :key="meta.id" :label="meta.name" :value="meta.id" />
      </el-select>
    </el-form-item>

    <!-- 多选框 -->
    <div class="map-fields-checklist">
      <el-checkbox v-model="checkAllToMap" :indeterminate="isIndeterminateToMap" @change="handlecheckAllToMapChange">{{
        $t('selectGroup.selectAll') }}</el-checkbox>
      <el-checkbox-group v-model="checkedFieldsToMap" @change="handleCheckedFieldsToMapChange">
        <el-checkbox v-for="fieldToMap in fieldsToMap" :key="fieldToMap.label" :label="fieldToMap.label">
          {{ $t(`selectGroup.videoInfo.${fieldToMap.label}`) }}
        </el-checkbox>
      </el-checkbox-group>
    </div>
    <div style="margin-top: 20px;" v-if="checkedFieldsToMap.includes('totalInterCount')">
      <el-select v-model="toCalcInterCount" multiple :placeholder="$t('placeholder.interCount')" style="width: 100%;"
        size="large">
        <el-option v-for="item in allToCalcInterCount" :key="item.label"
          :label="$t(`selectGroup.videoInfo.${item.label}`)" :value="item.label" />
      </el-select>
    </div>
    <el-alert style="display: flex;align-items: flex-start;margin: 20px 0;background-color: #e1eaff;color: #606266;"
      :title="$t('alerts.selectGroupFieldTip')" type="info" show-icon />


    <el-form-item v-if="isDetailMode" style="margin-top: 20px;" :label="$t('labels.cookie')" size="large">
      <div class="cookie-tip">当 Cookie 不填或过期后，笔记的互动数据将显示为近似值，如整十或整百。可参考说明文档更新 Cookie 以获取精确数据。</div>
      <el-input v-model="cookie" type="text" :placeholder="$t('placeholder.cookie')"></el-input>

    </el-form-item>

    <!--2024-07-24 版本已经不需要填写 x-s-common -->
    <!-- <el-form-item v-if="isDetailMode" style="margin-top: 20px;" :label="$t('labels.xSCommon')" size="large" required>
      <el-input v-model="xSCommon" type="text" :placeholder="$t('placeholder.xSCommon')"></el-input>
      
    </el-form-item> -->

    <!-- 选择对应的字段映射 -->


    <!-- 提交按钮 -->
    <el-button v-loading="isWritingData" @click="writeData" :disabled="!issubmitAbled" color="#3370ff" type="primary"
      plain size="large">{{ $t('submit') }}</el-button>

    <div v-show="isShowReward" style="padding-bottom: 50px;">
      <reward author="无意" :qrCode="qrCode" />
    </div>
  </el-form>
</template>

<script setup>
import { bitable, FieldType } from '@lark-base-open/js-sdk';
import { useI18n } from 'vue-i18n';
import { ref, onMounted, computed, isShallow } from 'vue';
import axios from 'axios';
import qs from 'qs';
import Reward from '@/components/Reward.vue'; // 确保路径正确
// -- 可更改区域
// TODO: 可替换为相应的后端服务基地址，注意末尾没有斜杠
const baseUrl = ref('https://feishu-xhs-assistant-nixiang-wuyi.replit.app')

// -- 数据区域
const { t } = useI18n();
const mainFieldListSeView = ref([])
const historyFieldListSeView = ref([])
const linkFieldId = ref('')  // 链接字段Id
const isShowReward = ref(false)

const qrCode = ref('./src/assets/qrCode.jpg'); // 你的二维码图片路径
const isWritingData = ref(false)
let historyTable
const isDetailMode = ref(true)
const checkAllToMap = ref(false)
const isIndeterminateToMap = ref(true)
const fieldsToMap = ref([
  {
    "label": "title"
  },
  {
    "label": "uploader"
  },
  {
    "label": "content"
  },
  {
    "label": "tags"
  },
  {
    "label": "releaseTime"
  },
  {
    "label": "lastUpdateTime"
  },
  // {
  //   "label": "viewCount"
  // },
  {
    "label": "collectionCount"
  },
  {
    "label": "likeCount"
  },
  {
    "label": "shareCount"
  },
  {
    "label": "commentCount"
  },
  {
    "label": "totalInterCount"
  },
  {
    "label": "fetchDataTime"
  },
  {
    "label": "errorTip"
  }
])   //  可以创建的字段
const checkedFieldsToMap = ref([
  'title',
  'uploader',
  'content',
  'tags',
  'releaseTime',
  'lastUpdateTime',
  // 'viewCount',
  'collectionCount',
  'likeCount',
  'shareCount',
  'commentCount',
  'fetchDataTime',
  'errorTip'
])   // 默认的to-map的字段

const toCalcInterCount = ref(['likeCount', 'collectionCount', 'shareCount', 'commentCount'])  // 实际用于计算“总交互量”的字段
const allToCalcInterCount = ref({})  // 可用于计算"总交互量"的字段
const cookie = ref('')
const xSCommon = ref('')
const isForcedEnd = ref(false)
const errorCount = ref(0)

const issubmitAbled = computed(() => {
  if (!isDetailMode.value)
    return linkFieldId.value && checkedFieldsToMap.value.length
  else
    return linkFieldId.value && checkedFieldsToMap.value.length

})  // 是否允许提交，及必选字段是否都填写

const mappedFieldIds = ref({
  "main": {},  // 主表
  "history": {}  // 历史记录 表
})

// -- 核心算法区域
// --001== 写入数据
const writeData = async () => {
  isShowReward.value = false
  errorCount.value = 0
  if (isWritingData.value) {
    isForcedEnd.value = true
  }
  isWritingData.value = true


  // 加载bitable实例
  const { tableId, viewId } = await bitable.base.getSelection();
  const table = await bitable.base.getActiveTable();
  const view = await table.getViewById(viewId);

  // 错误判断：应当在主表中进行
  if (table.id == historyTable.id) {
    bitable.ui.showToast({
      toastType: 'warning',
      message: t('notAllowedInHistoryTable')
    })
    isWritingData.value = false
    return
  }

  // 获取字段数据Ids object类型，匹配已有的字段，创建缺少的字段
  // @status {mappedFieldIds, isWritingData}  表示是命令性质的方法，改变mappedFieldIds对象的状态
  await completeMappedFieldIdsValue()

  // ## mode1: 全部记录
  // const RecordList = await view.getVisibleRecordIdList()

  // ## model2: 交互式选择记录 
  const RecordList = await bitable.ui.selectRecordIdList(tableId, viewId);

  // ## model3: 获取表格直接选中的记录
  // const RecordList = await table.getSelectedRecordIdList()
  console.log("RecordList", RecordList)

  localStorage.setItem('cookie', cookie.value)   // string 类型
  localStorage.setItem('xSCommon', xSCommon.value)   // string 类型
  localStorage.setItem('isDetailMode', isDetailMode.value)   // string 类型

  for (let recordId of RecordList) {
    // TODO：在这里书写处理逻辑——数据请求、数据写入等
    // 非空处理
    // 强制退出动作
    if (isForcedEnd.value) {
      return
    }
    console.log("writeData() >> recordId", recordId)


    /** 错误处理函数 
     * {param}: recordId
     * {return} noteLink、totalNoteInfo
    */
    console.log(218)
    let handleErrorRes = await handleError(recordId)
    console.log(handleErrorRes)
    if (handleErrorRes.isError)
      continue
    else if (handleErrorRes.isReturn)
      return

    const { noteLink, totalNoteInfo } = handleErrorRes
    console.log(227, noteLink)


    /** 
     * 命令式，但不改变任何对象的状态
     * @param {object} totalNoteInfo 
     * @param {object} table 数据表
     * @param {string} recordId 记录ID
     * @param {object} mappedFieldIds 映射字段Ids 
     */

    await getAndSetRecordValue(totalNoteInfo, table, recordId, mappedFieldIds.value.main)
    await getAndAddRecordValue(totalNoteInfo, historyTable, mappedFieldIds.value.history, noteLink)


  }

  isWritingData.value = false
  isShowReward.value = true
  await bitable.ui.showToast({
    toastType: 'success',
    message: `${t('finishTip')} ${errorCount.value}`
  })




}





// -- 辅助算法区域
// --001== 获取所勾选字段的字段Id
const getSelectedFieldsId = (fieldList, checkedFields) => {
  const mappedFields = {};
  for (let field of checkedFields) {
    // 查找与checkedFields相匹配的fieldListSeView项目
    const foundField = fieldList.find(f => f.name === t(`selectGroup.videoInfo.${field}`));

    if (field.endsWith('Count') && foundField && foundField.type !== 2)
      return ` [${t(`selectGroup.videoInfo.${field}`)}] ${t(`checks.number`)}`
    else if ((field == t('selectGroup.videoInfo.uploader') || field == t('selectGroup.videoInfo.content') || field == t('selectGroup.videoInfo.tags')) && foundField && foundField.type !== 1)
      return ` [${t(`selectGroup.videoInfo.${field}`)}] ${t(`checks.text`)}`
    else if (field == t('selectGroup.videoInfo.releaseTime') && foundField && foundField.type !== 5)
      return ` [${t(`selectGroup.videoInfo.${field}`)}] ${t(`checks.datetime`)}`
    else if (field == t('selectGroup.videoInfo.lastUpdateTime') && foundField && foundField.type !== 5)
      return ` [${t(`selectGroup.videoInfo.${field}`)}] ${t(`checks.datetime`)}`


    // 如果找到了相应的项目，就使用其id，否则设置为-1
    mappedFields[field] = foundField ? foundField.id : -1;
  }



  return mappedFields;
}


// --002== 请求在replit写的flask框架的接口，获取基本数据
/* @param:path
* get_xhs_detail_data-获取详细数据
*/
const getXHSdatabylink = async (path, noteLink) => {
  const url = `${baseUrl.value}/${path}`;
  const data = {
    url: noteLink,
    cookie: cookie.value
  };

  try {
    const response = await axios.post(url, qs.stringify(data));

    const noteInfo = response.data.info;
    const res = {
      status: 200,
      info: {
        title: noteInfo.title,
        uploader: noteInfo.uploader,
        content: noteInfo.content,
        tags: noteInfo.tags,
        releaseTime: noteInfo.releaseTime,
        lastUpdateTime: noteInfo.lastUpdateTime,
        collectionCount: noteInfo.collectionCount,
        likeCount: noteInfo.likeCount,
        shareCount: noteInfo.shareCount,
        commentCount: noteInfo.commentCount,
        images: noteInfo.images
      }
    };

    return res.status === 200 ? res.info : { status: "-100", result: res };
  } catch (error) {
    console.error(error);
    const errorResponse = error.response ? error.response.data.error : error.message;
    console.log("getXHSdatabylink() >> errorResponse", errorResponse)
    return { status: "-100", result: errorResponse };
  }
};


// --003== 创建 mappedFieldIds 中 value 为 -1 的字段
/**
 * 命令式，改变mappedFieldIds.value的状态
 * @param {param} mappedFieldIds 现有的映射字段ids
 * @param {param} table 当前要处理的table
 */
const createFields = async (mappedFieldIds, table) => {

  for (let key in mappedFieldIds) {
    if (mappedFieldIds[key] === -1) {
      switch (key) {

        case "title":  // 视频名称
          mappedFieldIds[key] = await table.addField({
            type: FieldType.Text,
            name: t(`selectGroup.videoInfo.${key}`),
          })
          break;
        case "link":  // 链接
          mappedFieldIds[key] = await table.addField({
            type: FieldType.Text,
            name: t(`selectGroup.videoInfo.link`),
          })
          break;
        case "errorTip":
          mappedFieldIds[key] = await table.addField({
            type: FieldType.Text,
            name: t(`selectGroup.videoInfo.errorTip`),
          })
          break;
        case "uploader":
        case "content":
        case "tags":
          mappedFieldIds[key] = await table.addField({
            type: FieldType.Text,
            name: t(`selectGroup.videoInfo.${key}`),
          })
          break;
        case "releaseTime":
        case "lastUpdateTime":
          mappedFieldIds[key] = await table.addField({
            type: FieldType.DateTime,
            name: t(`selectGroup.videoInfo.${key}`),
          })
          break;
        case "danmuCount":
        case "coinCount":
        case "viewCount":
        case "collectionCount":
        case "likeCount":
        case "commentCount":  // 评论量
        case "totalInterCount":  // 总互动量
        case "shareCount":
          mappedFieldIds[key] = await table.addField({
            type: FieldType.Number,
            name: t(`selectGroup.videoInfo.${key}`),
          })
          break;
        case "commentWc":
        case "danmuWc":
          mappedFieldIds[key] = await table.addField({
            type: FieldType.Attachment,
            name: t(`selectGroup.videoInfo.${key}`),
          })
          break;
        case "fetchDataTime":
          mappedFieldIds[key] = await table.addField({
            type: FieldType.DateTime,
            name: t(`selectGroup.videoInfo.${key}`),
          })
          break;
      }
    }


  }

}

// --004== 依据 recordId & field 获取 cell 值
const getCellValueByRFIDS = async (recordId, field) => {
  const cellValue = await field.getValue(recordId)
  if (typeof cellValue == 'object')
    return cellValue[0].link

  return cellValue
}

// --005== 依据 checkedFiledsToMap 做不同的请求处理
const getDataByCheckedFields = async (noteLink) => {
  let basicInfo = await getXHSdatabylink('get_xhs_detail_data', noteLink)
  return { basicInfo }
}

/** --006== 补全mappedFieldIds对象的值，使得拿到所有字段数据Ids object类型
 * 匹配已有的字段，创建缺少的字段
 * @status {mappedFieldIds, isWritingData}  表示是命令性质的方法，改变mappedFieldIds对象的状态
 */
const completeMappedFieldIdsValue = async () => {
  const selection = await bitable.base.getSelection()
  const table = await bitable.base.getTableById(selection.tableId)

  // 匹配已有的字段
  const mainMappedFields = getSelectedFieldsId(mainFieldListSeView.value, checkedFieldsToMap.value)
  // 历史记录表的 “视频链接” 字段
  let historyCheckedFields = JSON.parse(JSON.stringify(checkedFieldsToMap.value))
  historyCheckedFields.push('link')
  let result = historyCheckedFields.filter(item => item != 'errorTip')

  const historyMappedFields = getSelectedFieldsId(historyFieldListSeView.value, result)


  if (typeof mainMappedFields == 'string' || typeof historyMappedFields == 'string') {// 错误处理，提示格式错误 
    await bitable.ui.showToast({
      toastType: 'warning',
      message: mappedFields
    })
    isWritingData.value = false
    return
  }

  // 创建缺少的字段
  mappedFieldIds.value.main = mainMappedFields
  mappedFieldIds.value.history = historyMappedFields
  await createFields(mappedFieldIds.value.main, table)
  await createFields(mappedFieldIds.value.history, historyTable)
}

/** --007== 错误处理 
 * @status {} 命令性质，写入 errorTip Field
 * 
 * 
*/
const handleErrorTip = async (errorMsg, recordId) => {
  const table = await bitable.base.getActiveTable();


  errorCount.value++
  await table.setCellValue(mappedFieldIds.value.main['errorTip'], recordId, [{ type: 'text', text: errorMsg }])
}

/**
 * --008== 处理数据写入中的异常
 * @param {string} recordId
 * @return {Promise<noteLink、totalNoteInfo、isError、isReturn>} 
 */
const handleError = async (recordId) => {
  // 错误处理，链接字段格式错误，应为文本类型
  const table = await bitable.base.getActiveTable();

  const linkFieldMeta = await table.getFieldMetaById(linkFieldId.value)
  const linkField = await table.getFieldById(linkFieldId.value)
  if (linkFieldMeta.type !== 1 && linkFieldMeta.type !== 15) {
    await handleErrorTip(`[${linkFieldMeta.name}] ${t('errorTip.errorLinkType')}`, recordId)
    isWritingData.value = false
    return { "isReturn": true }
  }

  // 错误处理：链接地址为空
  let noteLink
  try {
    noteLink = await getCellValueByRFIDS(recordId, linkField)

  } catch (error) {

    return { "isError": true, "isReturn": false }
  }

  console.log(540, noteLink)
  // 错误处理：链接格式错误
  if (!noteLink.includes('https://www.xiaohongshu.com/') && !noteLink.includes('xhslink.com/')) {

    await handleErrorTip(t('errorTip.errorLink'), recordId)
    return { "isError": true }
  } 
  console.log("noteLink", noteLink)


  let totalNoteInfo = await getDataByCheckedFields(noteLink)

  if (totalNoteInfo.basicInfo.status == -100) {
    await handleErrorTip(totalNoteInfo.basicInfo.result, recordId)
    return { "isError": true }
  }

  return { noteLink, totalNoteInfo }
}

/**
 * --009== 获取并设置特定 table 的特定 record 的特定字段集合的 Value
 * @param {object} totalNoteInfo 
 * @param {object} table 数据表
 * @param {string} recordId 记录ID
 * @param {object} mappedFieldIds 映射字段Ids 
 */
const getAndSetRecordValue = async (totalNoteInfo, table, recordId, mappedFieldIds) => {
  const recordFields = getRecordFields(totalNoteInfo, mappedFieldIds)
  console.log(recordFields)


  await table.setRecord(recordId, {
    fields: recordFields
  })
}

const getAndAddRecordValue = async (totalNoteInfo, table, mappedFieldIds, noteLink) => {
  console.log("111111", noteLink)

  const recordFields = getRecordFields(totalNoteInfo, mappedFieldIds, noteLink)
  console.log(recordFields)


  await table.addRecord({
    fields: recordFields
  })
}

/**
 * 查询式，获取 recordFields
 * @param {object} totalNoteInfo 笔记数据
 * @param {object} mappedFieldIds 字段链接
 */
const getRecordFields = (totalNoteInfo, mappedFieldIds, noteLink = 0) => {
  let recordFields = {}
  let key = ''
  let value = ''
  let fetchDataTimeValue = Date.now()
  console.log(mappedFieldIds)
  let checkedFields = JSON.parse(JSON.stringify(checkedFieldsToMap.value))

  checkedFields = checkedFields.filter(item => item != 'errorTip')
  for (let field of checkedFields) {

    key = mappedFieldIds[field]

    if (field === 'fetchDataTime')
      value = fetchDataTimeValue
    else if (field === 'totalInterCount')
      value = toCalcInterCount.value.reduce((total, key) => total + totalNoteInfo.basicInfo[key], 0);
    else
      value = totalNoteInfo.basicInfo[field]


    recordFields[key] = value
  }

  // 打个补丁，链接字段
  if (noteLink)
    recordFields[mappedFieldIds['link']] = noteLink


  return recordFields
}


// Map==全选事件
const handlecheckAllToMapChange = (val) => {
  const data = JSON.parse(JSON.stringify(fieldsToMap.value))
  if (val) {
    checkedFieldsToMap.value = []
    for (const item of data)
      checkedFieldsToMap.value.push(item.label);


  } else {
    checkedFieldsToMap.value = []
  }
  isIndeterminateToMap.value = false
}
// Map==字段选择事件
const handleCheckedFieldsToMapChange = (value) => {
  const checkedCount = value.length
  checkAllToMap.value = checkedCount === fieldsToMap.value.length
  isIndeterminateToMap.value = checkedCount > 0 && checkedCount < fieldsToMap.value.length
}


onMounted(async () => {
  // 初始化勾选字段
  // const language = await bitable.bridge.getLanguage();

  // 获取字段列表 -- start
  const selection = await bitable.base.getSelection()
  const table = await bitable.base.getTableById(selection.tableId)
  const view = await table.getViewById(selection.viewId)
  mainFieldListSeView.value = await view.getFieldMetaList()
  // 获取字段列表 -- end


  // 历史记录表
  try {
    historyTable = await bitable.base.getTableByName(t('history.tableName'));

  } catch (error) {
    const { tableId, index } = await bitable.base.addTable({
      name: t('history.tableName')
    })

    historyTable = await bitable.base.getTableById(tableId)

  }

  historyFieldListSeView.value = await historyTable.getFieldMetaList()


  // 初始化可参与计算 “总交互量” 的对象数组，以“Count”结尾的
  allToCalcInterCount.value = fieldsToMap.value
    .filter(item => item.label.endsWith('Count'));

  if (localStorage.getItem('isDetailMode') !== null) {  // string 类型
    isDetailMode.value = Boolean(localStorage.getItem('isDetailMode'))
  }
  if (localStorage.getItem('cookie') !== null) {  // string 类型
    cookie.value = localStorage.getItem('cookie')
  }
  if (localStorage.getItem('xSCommon') !== null) {  // string 类型
    xSCommon.value = localStorage.getItem('xSCommon')
  }


});

</script>



<style scoped>
.helper-doc {

  margin-top: -10px;
  font-size: 14px;
}

.helper-doc a {
  color: #409eff;
}

.helper-doc a:hover {
  color: #7abcff;
}

.cookie-tip {
  font-size: 14px;
  color: #a29d9d;
  line-height: 1.5;
  margin-bottom: 20px;
}
</style>
