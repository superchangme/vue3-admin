<script setup lang="ts">
export interface ItemForm {
  type: string
  label: string
  placeholder: string
  mode?: string
  disabled?: boolean
  style?: string
  onChange?: () => void
  [propName: string]: any
}
export interface Submit {
  // submitFunction: () => void
  submitFunction: any
  cancelFunction: () => void
  submitText?: string
  submitDisabled?: boolean
  submitIcon?: string
  submitType?: string
  submitStyle?: string
  [propName: string]: any
}

export interface FormOptions {
  data: any // 表单数据对象
  formList: { [propName: string]: ItemForm }
  rules?: object // 表单验证规则
  inline?: boolean // 行内表单模式
  labelWidth: string // 表单域标签的宽度('123px') 支持 auto
  // loading: boolean
  size?: string // 用于控制该表单内组件的尺寸 可选值：medium / small / mini
  statusIcon?: boolean // 是否在输入框中显示校验结果反馈图标
  validateOnRuleChange?: boolean // 是否在 rules 属性改变后立即触发一次验证
  scrollToError?: boolean // 当校验失败时，滚动到第一个错误表单项
  showMessage?: boolean // 是否显示校验错误信息
  inlineMessage?: boolean // 是否以行内形式展示校验信息
  disabled?: boolean // 是否禁用该表单内的所有组件
  hideRequiredAsterisk?: boolean // 是否显示必填字段的标签旁边的红色星号
  labelSuffix?: string // 表单域标签的后缀,''
  labelPosition?: 'left' | 'right' // 表单域标签的位置，如果值为 left 或者 right 时，则需要设置 label-width 可选值：right/left/top
  align?: string
  submit?: Submit // 提交表单的方法
}

const props = defineProps<{ formParams: FormOptions }>()
const formParams = computed(() => props.formParams)

const ruleFormRef = ref<any>()

// 重置
const resetForm = () => {
  ruleFormRef.value.resetFields() // 重置
}
// 主动暴露childMethod方法
defineExpose({ resetForm, submitForm })

// 提交表单
function submitForm(submit: any) {
  ruleFormRef.value.validate((valid: any) => {
    if (valid) {
      // eslint-disable-next-line no-unused-expressions
      submit && submit()
    }
  })
}

// function getOption(value: any, defaultValue: any) {
//   return value === void 0 ? defaultValue : value;
// }

const formStyle = ref({
  textAlign: formParams.value.align || 'left',
  submitButton: formParams.value.align ? 'block' : 'inline-block',
  formWidth: formParams.value.align ? '100%' : '',
})
</script>
<template>
  <div class="form">
    <el-form
      ref="ruleFormRef"
      :model="formParams.data"
      :rules="formParams.rules"
      :inline="formParams.inline"
      :label-position="formParams.labelPosition"
      :label-width="formParams.labelWidth"
      :label-suffix="formParams.labelSuffix"
      :hide-required-asterisk="formParams.hideRequiredAsterisk"
      :show-message="formParams.showMessage"
      :inline-message="formParams.inlineMessage"
      :status-icon="formParams.statusIcon"
      :validate-on-rule-change="formParams.validateOnRuleChange"
      :size="formParams.size"
      :disabled="formParams.disabled"
      :scroll-to-error="formParams.scrollToError"
    >
      <el-form-item
        v-for="(itemForm, key) in formParams.formList"
        :key="key"
        :prop="key.toString()"
        :label="itemForm.label"
        :style="itemForm.style"
      >
        <!-- 文本框输入 -->
        <el-input
          v-if="['text', 'textarea'].includes(itemForm.type)"
          v-model="formParams.data[key]"
          :type="itemForm.mode"
          :maxlength="itemForm.maxlength"
          :show-word-limit="!!itemForm.maxlength"
          :placeholder="itemForm.placeholder"
          :disabled="itemForm.disabled"
          :style="`${itemForm.width ? 'width:' + itemForm.width : ''}`"
          :rows="itemForm.rows"
          :show-password="itemForm.isPassword"
          :clearable="itemForm.clearable"
          :validate-event="itemForm.validateEvent"
        />
        <!-- 密码、纯数字输入 -->
        <el-input
          v-if="itemForm.type === 'number'"
          v-model.number="formParams.data[key]"
          :type="itemForm.mode"
          :maxlength="itemForm.maxlength"
          :show-word-limit="!!itemForm.maxlength"
          :placeholder="itemForm.placeholder"
          :disabled="itemForm.disabled"
          :clearable="itemForm.clearable"
          :show-password="itemForm.isPassword"
        />
        <!-- 数字输入 -->
        <el-input-number
          v-if="itemForm.type === 'input-number'"
          v-model="formParams.data[key]"
        />
        <!-- 日期 -->
        <el-date-picker
          v-if="itemForm.type === 'date-picker'"
          v-model="formParams.data[key]"
          :placeholder="itemForm.placeholder"
          :disabled-date="itemForm.disabledDate"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
          :style="'width:100%'"
          :type="itemForm.mode"
          :value-format="itemForm.valueFormat || 'YYYY-MM-DD'"
          :default-time="itemForm.defaultTime"
          @change="itemForm.onChange"
        />
        <!-- 下拉框 -->
        <el-select
          v-if="itemForm.type === 'select'"
          v-model="formParams.data[key]"
          :multiple="itemForm.multiple"
          :multiple-limit="itemForm.multipleLimit"
          :style="`${itemForm.width ? 'width:' + itemForm.width : ''}`"
          :clearable="itemForm.clearable"
          :placeholder="itemForm.placeholder"
          :disabled="itemForm.disabled"
          :collapse-tags="itemForm.collapseTags"
          :filterable="itemForm.filterable"
          :loading="itemForm.loading"
          :loading-text="itemForm.loadingText"
          :no-data-text="itemForm.noDataText"
          @change="itemForm.onChange"
        >
          <!-- 空状态自定义 -->
          <template #empty></template>
          <el-option
            v-for="(selectOption, index) in itemForm.selectOptions"
            :key="index"
            :label="
              itemForm.customLabelValue
                ? selectOption[itemForm.customLabelValue.label]
                : selectOption.name
            "
            :value="
              itemForm.customLabelValue
                ? selectOption[itemForm.customLabelValue.value]
                : selectOption.id
            "
            :disabled="
              itemForm.customLabelValue
                ? selectOption[itemForm.customLabelValue.disabled]
                : selectOption.disabled
            "
          />
        </el-select>
        <!-- 级联 -->
        <el-cascader
          v-if="itemForm.type === 'cascader'"
          v-model="formParams.data[key]"
          :options="itemForm.cascaderOptions"
          :disabled="itemForm.disabled"
          :placeholder="itemForm.placeholder"
          :style="`${itemForm.width ? 'width:' + itemForm.width : ''}`"
          clearable
          filterable
          @change="itemForm.onChange"
        />
        <!-- 滑块 -->
        <el-slider
          v-if="itemForm.type === 'slider'"
          v-model="formParams.data[key]"
        />
        <!-- 开关 -->
        <el-switch
          v-if="itemForm.type === 'switch'"
          v-model="formParams.data[key]"
          :active-text="itemForm.activeText"
          :inactive-text="itemForm.inactiveText"
          :disabled="itemForm.disabled"
          :loading="itemForm.loading"
          @change="itemForm.onChange"
        />
        <!-- 单选框 -->
        <el-radio-group
          v-if="itemForm.type === 'radio'"
          v-model="formParams.data[key]"
        >
          <el-radio
            v-for="(radioOption, index) in itemForm.radioOptions"
            :key="index"
            :label="radioOption.value"
          >
            {{ radioOption.text }}
          </el-radio>
        </el-radio-group>
        <!-- 多选框 -->
        <el-checkbox
          v-if="itemForm.type === 'checkbox'"
          v-model="formParams.data[key]"
          :true-label="
            itemForm.checkboxOption.trueLabel != undefined
              ? itemForm.checkboxOption.trueLabel
              : true
          "
          :false-label="
            itemForm.checkboxOption.falseLabel != undefined
              ? itemForm.checkboxOption.falseLabel
              : false
          "
        >
          {{
            itemForm.checkboxOption.label
              ? itemForm.checkboxOption.label
              : itemForm.checkboxOption
          }}
        </el-checkbox>

        <el-switch
          v-if="itemForm.type === 'switch'"
          v-model="formParams.data[key]"
          :active-text="itemForm.activeText"
          :inactive-text="itemForm.inactiveText"
        />

        <el-time-select
          v-if="itemForm.type === 'time-select'"
          v-model="formParams.data[key]"
          :disabled="itemForm.disabled"
          :placeholder="itemForm.placeholder"
        />
        <el-time-picker
          v-if="itemForm.type === 'time-picker'"
          v-model="formParams.data[key]"
          :disabled="itemForm.disabled"
          :placeholder="itemForm.placeholder"
          start-placeholder="开始时间"
          end-placeholder="结束时间"
          :is-range="itemForm.isRange"
          :value-format="itemForm.valueFormat || 'HH:mm:ss'"
        />
        <el-date-picker
          v-if="itemForm.type === 'date-picker'"
          v-model="formParams.data[key]"
          :disabled="itemForm.disabled"
          :placeholder="itemForm.placeholder"
          :disabled-date="itemForm.disabledDate"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
          :style="'width:100%'"
          :type="itemForm.mode"
          :value-format="itemForm.valueFormat || 'YYYY-MM-DD'"
          :default-time="itemForm.defaultTime"
          @change="itemForm.onChange"
        />
        <el-rate
          v-if="itemForm.type === 'rate'"
          v-model="formParams.data[key]"
        />
        <el-color-picker
          v-if="itemForm.type === 'color-picker'"
          v-model="formParams.data[key]"
        />
        <el-transfer
          v-if="itemForm.type === 'transfer'"
          v-model="formParams.data[key]"
          :data="itemForm.transferData"
        />
        <!-- 表单其他组件 -->
        <template
          v-if="itemForm.slots && itemForm.slots.formItem"
          #formItem="{ column, $index }"
        >
          <slot
            :name="itemForm.slots!.formItem"
            :column="column"
            :index="$index"
          />

          <slot
            v-if="itemForm.type === 'customItem'"
            :name="itemForm.name ? itemForm.name : 'customItem'"
            :item-form="itemForm"
            :form-date="formParams.data"
            :item-key="key"
          />
        </template>
      </el-form-item>

      <el-form-item v-if="formParams.submit">
        <el-button
          type="primary"
          :disabled="formParams.submit.disabled"
          @click="formParams.submit?.submitFunction"
        >
          {{ formParams.submit.submitText || '提交' }}
        </el-button>

        <el-button
          v-if="formParams.submit.reset"
          type="info"
          @click="resetForm"
        >
          重置
        </el-button>

        <el-button
          v-if="formParams.submit.cancel"
          type="info"
          @click="formParams.submit?.cancelFunction"
        >
          {{ formParams.submit.cancelText || '取消' }}
        </el-button>
      </el-form-item>
      <slot name="buttonGroup" />
    </el-form>
  </div>
</template>
<style lang="scss" scoped></style>

<style scoped lang="scss">
// .form-container {
// width: v-bind('formStyle.formWidth');
// }

// .el-form-item:last-child {
// display: v-bind('formStyle.submitButton');
//   text-align: center;
// }

// .el-select,
// .el-cascader,
// .el-time-select,
// .el-time-picker,
// .el-date-picker {
//   width: 100%;
// }
</style>
