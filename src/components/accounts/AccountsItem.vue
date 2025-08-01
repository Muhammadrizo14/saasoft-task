<template>
  <TableRow class="border-b">
    <TableCell class="p-2">
      <Input
        v-model="labelsInput"
        @blur="handleLabelsBlur"
        placeholder="Метки через ;"
        maxlength="50"
        class="w-full"
      />
    </TableCell>
    <TableCell class="p-2">
      <Select :model-value="account.type" @update:model-value="handleTypeChange">
        <SelectTrigger class="w-[120px]">
          <SelectValue placeholder="Тип" />
        </SelectTrigger>
        <SelectContent>
          <SelectItem :value="AccountType.LDAP">LDAP</SelectItem>
          <SelectItem :value="AccountType.LOCAL">Локальная</SelectItem>
        </SelectContent>
      </Select>
    </TableCell>
    <TableCell class="p-2" :colspan="account.type === AccountType.LDAP ? 2 : 1">
      <Input
        v-model="account.login"
        @blur="handleLoginBlur"
        :class="{ 'border-red-500': account.errors.login }"
        maxlength="100"
        class="w-full"
      />
    </TableCell>
    <TableCell v-if="account.type === AccountType.LOCAL" class="p-2 relative">
      <div class="relative">
        <Input
          v-model="passwordValue"
          :type="showPassword ? 'text' : 'password'"
          @blur="handlePasswordBlur"
          :class="{ 'border-red-500': account.errors.password }"
          maxlength="100"
          class="w-full pr-10"
        />
        <Button
          variant="ghost"
          size="icon"
          class="absolute right-2 top-1/2 -translate-y-1/2"
          @click="togglePasswordVisibility"
        >
          <EyeIcon v-if="!showPassword" class="w-5 h-5" />
          <EyeOffIcon v-else class="w-5 h-5" />
        </Button>
      </div>
    </TableCell>
    <TableCell class="p-2">
      <Button variant="destructive" size="sm" @click="handleRemove">
        <svg  viewBox="0 0 1024 1024" xmlns="http://www.w3.org/2000/svg"><path fill="#FFF" d="M160 256H96a32 32 0 0 1 0-64h256V95.936a32 32 0 0 1 32-32h256a32 32 0 0 1 32 32V192h256a32 32 0 1 1 0 64h-64v672a32 32 0 0 1-32 32H192a32 32 0 0 1-32-32V256zm448-64v-64H416v64h192zM224 896h576V256H224v640zm192-128a32 32 0 0 1-32-32V416a32 32 0 0 1 64 0v320a32 32 0 0 1-32 32zm192 0a32 32 0 0 1-32-32V416a32 32 0 0 1 64 0v320a32 32 0 0 1-32 32z"/></svg>
      </Button>
    </TableCell>
  </TableRow>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { useAccountsStore } from '@/stores/accounts'
import type { Account } from '@/types'
import { AccountType } from '@/types'
import { EyeIcon, EyeOffIcon } from 'lucide-vue-next'

import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select'
import { TableCell, TableRow } from '@/components/ui/table'

const { account } = defineProps<{
  account: Account
}>()

const store = useAccountsStore()
const labelsInput = ref(account.labels.map(label => label.text).join('; '))
const showPassword = ref(false)

const passwordValue = computed({
  get: () => account.password === null ? '' : account.password,
  set: (value) => {
    account.password = value
  }
})

// Обработка изменения меток (onBlur)
const handleLabelsBlur = () => {
  if (labelsInput.value !== account.labels.map(label => label.text).join('; ')) {
    store.updateAccount(account.id, {
      labels: labelsInput.value
        .split(';')
        .map(text => text.trim())
        .filter(text => text)
        .map(text => ({ text }))
        .slice(0, 50)
    })
  }
}

// Обработка изменения типа (onBlur)
const handleTypeChange = (value: any) => {
  if (value && (value === AccountType.LDAP || value === AccountType.LOCAL) && value !== account.type) {
    const accountType = value as AccountType;
    store.updateAccount(account.id, {
      type: accountType,
      password: accountType === AccountType.LDAP ? null : account.password
    })
    store.validateAccount(account.id)
  }
}

// Обработка логина (onBlur)
const handleLoginBlur = () => {
  store.updateAccount(account.id, { login: account.login })
  store.validateAccount(account.id)
}

// Обработка пароля (onBlur)
const handlePasswordBlur = () => {
  store.updateAccount(account.id, { password: account.password })
  store.validateAccount(account.id)
}

// Удаление учетной записи
const handleRemove = () => {
  store.removeAccount(account.id)
}

// Переключение видимости пароля
const togglePasswordVisibility = () => {
  showPassword.value = !showPassword.value
}
</script>
