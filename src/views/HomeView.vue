<template>
  <el-form
    ref="ruleFormRef"
    style="max-width: 400px"
    :model="ruleForm"
    status-icon
    :rules="rules"
    label-width="auto"
    class="demo-ruleForm"
  >

  <el-form-item label="Имя" prop="firstName" class="is-error is-required">
      <el-input v-model.string="ruleForm.firstName" />
      <div class="el-form-item__error">Имя обязательно для заполнения</div>
    </el-form-item>

  <el-form-item label="Фамилия" prop="secondName">
      <el-input v-model.string="ruleForm.secondName" />
    </el-form-item>

  <el-form-item label="Email" prop="email">
      <el-input v-model="ruleForm.email" type="email"/>
    </el-form-item>

  <el-form-item label="Номер телефона" prop="phoneNumber">
      <el-input v-model="ruleForm.phoneNumber" type="tel"/>
      <!-- pattern="[0-9]{1}-[0-9]{3}-[0-9]{2}-[0-9]{2}-[0-9]{3}" -->
    </el-form-item>

  <el-form-item label="Возраст" prop="age">
      <el-input v-model.number="ruleForm.age" />
    </el-form-item>

    <el-form-item label="Пол" prop="gender">
      <el-select v-model="ruleForm.gender" placeholder="">
         <el-option label="Male" value="male" />
         <el-option label="Female" value="female" />
        </el-select>
    </el-form-item>

    <el-form-item label="Password" prop="pass">
      <el-input v-model="ruleForm.pass" type="password" autocomplete="off" />
    </el-form-item>

    <el-form-item label="Confirm" prop="checkPass">
      <el-input
        v-model="ruleForm.checkPass"
        type="password"
        autocomplete="off"
      />
    </el-form-item>

    <el-form-item>
      <el-button  class="form-button__left" type="primary" @click="submitForm(ruleFormRef)">
        Создать
      </el-button>
      <el-button>Отменить</el-button>
      <!-- @click="resetForm(ruleFormRef)" -->
    </el-form-item>
  </el-form>

  <el-dialog
    v-model="dialogVisible"
    title="Success"
    width="500"
  >
      <img src="@/assets/thankYouImg.jpg" alt="Описание изображения" class="dialog__img">
    <template #footer>
      <div class="dialog-footer">
        <el-button type="primary" @click="dialogVisible = false">
          Ok
        </el-button>
      </div>
    </template>
  </el-dialog>
</template>

<script lang="ts" setup>
import { reactive, ref } from 'vue'
import type { FormInstance, FormRules } from 'element-plus'
import axios from 'axios'

const dialogVisible = ref(false)
const ruleFormRef = ref<FormInstance>()

const checkAge = (rule: any, value: any, callback: any) => {
  if (!value) {
    return callback()
    // return callback(new Error('Please input the age'))
  }
  setTimeout(() => {
    if (!Number.isInteger(value)) {
      callback(new Error('Please input digits'))
    } else {
      if (value < 18) {
        callback(new Error('Age must be greater than 18'))
      } else {
        callback()
      }
    }
  }, 1000)
}

const checkEmail = (rule: any, value: any, callback: any) => {
  if (!value) {
    return callback()
    // return callback(new Error('Please input the number'))
  }
  setTimeout(() => {
      const emailRegex = /^(([^<>()[\].,;:\s@"]+(\.[^<>()[\].,;:\s@"]+)*)|(".+"))@(([^<>()[\].,;:\s@"]+\.)+[^<>()[\].,;:\s@"]{2,})$/iu;  

      if (!emailRegex.test(value)) {
        callback(new Error('Invalid email format entered'))
      } else {
        callback()
      }
    
  }, 1000)
}

const checkPhone = (rule: any, value: any, callback: any) => {
  if (!value) {
    callback()
  }
  setTimeout(() => {
      const emailRegex = /^(\s*)?(\+)?([- _():=+]?\d[- _():=+]?){10,14}(\s*)?$/;
     
      if (!emailRegex.test(value)) {
        callback(new Error('Invalid phone number format entered'))
      } else {
        callback()
      }
    
  }, 1000)
}

const validatePass = (rule: any, value: any, callback: any) => {
  if (value === '') {
    callback()
    // new Error('Please input the password')
  } else {
    if (ruleForm.checkPass !== '') {
      if (!ruleFormRef.value) return
      ruleFormRef.value.validateField('checkPass', (): any => null )
    }
    callback()
  }
}
const validatePass2 = (rule: any, value: any, callback: any) => {
  if (value === '') {
    callback()
    // new Error('Please input the password again')
  } else if (value !== ruleForm.pass) {
    callback(new Error("Two inputs don't match!"))
  } else {
    callback()
  }
}

const ruleForm = reactive({
  firstName: '',
  secondName: '',
  email: '',
  phoneNumber: '',
  age: '',
  gender: '',
  pass: '',
  checkPass: '',
})

const rules = reactive<FormRules<typeof ruleForm>>({
  firstName: [
    // { required: true, message: 'Please input first name', trigger: 'blur' },
    // { min: 3, max: 10, message: 'Length should be 3 to 10', trigger: 'blur' },
  ],
  secondName: [
    { required: true, message: 'Please input second name', trigger: 'blur' },
    { min: 3, max: 15, message: 'Length should be 3 to 15', trigger: 'blur' },
  ],
  email: [
    // { required: true, message: 'Please input email', trigger: 'blur' },
    // { min: 3, max: 15, message: 'Length should be 3 to 15', trigger: 'blur' },
    { validator: checkEmail, trigger: 'blur' }
  ],
  phoneNumber: [
    { validator: checkPhone, trigger: 'blur' }
  ],
  age: [
    { validator: checkAge, trigger: 'blur' },
    { type: 'number', message: 'Age must be a number' },
  ],
  gender: [
    { message: 'Please choose gender', trigger: 'change' }
  ],
  pass: [{ validator: validatePass, trigger: 'blur' }],
  checkPass: [{ validator: validatePass2, trigger: 'blur' }],
})

const submitForm = (formEl: FormInstance | undefined) => {
  if (!formEl) return
  formEl.validate((valid): any => {
    if (valid) {
      dialogVisible.value = true

      axios.get('https://jsonplaceholder.typicode.com/todos/acomps')
    
      formEl.resetFields()
    } else {
      return false
    }
  })
}

// const resetForm = (formEl: FormInstance | undefined) => {
//   if (!formEl) return
//   formEl.resetFields()
// }
</script>

<style lang="css">
.dialog__img {
  width: 100%;
  margin: 30px 0;
}

.form-button__left {
  margin-left: auto;
}
</style>