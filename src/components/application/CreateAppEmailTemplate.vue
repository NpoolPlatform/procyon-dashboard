<template>
  <q-card class='container'>
    <q-card-section>
      <q-item-label>{{ $t('MSG_CREATE_APP_EMAIL_TEMPLATE') }}</q-item-label>
    </q-card-section>
    <q-item-section>
      <q-btn-dropdown
        dense
        split
        icon='window'
        no-caps
        align='left'
        :label='selectedLangName'
      >
        <q-list>
          <q-item
            v-for='lang in languages'
            :key='lang.ID'
            v-close-popup
            clickable
            @click='onLangItemClick(lang)'
          >
            <q-item-section avatar>
              <q-avatar :icon='lang.Logo' />
            </q-item-section>
            <q-item-label>{{ lang.Name }}</q-item-label>
          </q-item>
        </q-list>
      </q-btn-dropdown>
      <q-input
        v-model='sender'
        :label='$t("MSG_SENDER")'
      >
        <template #prepend>
          <q-icon name='window' />
        </template>
      </q-input>
      <q-input
        v-model='defaultToUsername'
        :label='$t("MSG_DEFAULT_TO_USERNAME")'
      >
        <template #prepend>
          <q-icon name='window' />
        </template>
      </q-input>
      <q-input
        v-model='replyTos'
        :label='$t("MSG_REPLY_TO")'
      >
        <template #prepend>
          <q-icon name='window' />
        </template>
      </q-input>
      <q-input
        v-model='ccTos'
        :label='$t("MSG_CC_TO")'
      >
        <template #prepend>
          <q-icon name='window' />
        </template>
      </q-input>
      <q-input
        v-model='subject'
        :label='$t("MSG_SUBJECT")'
      >
        <template #prepend>
          <q-icon name='window' />
        </template>
      </q-input>
      <q-input
        v-model='body'
        :label='$t("MSG_BODY")'
        type='textarea'
      >
        <template #prepend>
          <q-icon name='window' />
        </template>
      </q-input>
      <q-input
        v-model='usedFor'
        :label='$t("MSG_USED_FOR")'
      >
        <template #prepend>
          <q-icon name='window' />
        </template>
      </q-input>
    </q-item-section>
    <q-item-section>
      <q-btn
        class='submit-btn'
        :label='$t("MSG_SUBMIT")'
        @click='onSubmit'
      />
    </q-item-section>
  </q-card>
</template>

<script setup lang='ts'>
import { defineProps, toRef, computed, defineEmits, watch, onMounted, ref } from 'vue'
import { AppEmailTemplate } from 'src/store/appemailtemplates/types'
import { useStore } from 'src/store'
import { Language } from 'src/store/languages/types'
import { ActionTypes } from 'src/store/languages/action-types'
import { ModuleKey, Type as NotificationType } from 'src/store/notifications/const'
import { useI18n } from 'vue-i18n'

const store = useStore()

// eslint-disable-next-line @typescript-eslint/unbound-method
const { t } = useI18n({ useScope: 'global' })

interface Props {
  editTemplate?: AppEmailTemplate
}

const props = defineProps<Props>()

const editTemplate = toRef(props, 'editTemplate')

const languages = computed(() => store.getters.getAppLangInfos)

const editTemplateLang = computed(() => {
  if (!editTemplate.value) {
    return {} as Language
  }
  for (let i = 0; i < languages.value.length; i++) {
    if ((editTemplate.value as unknown as AppEmailTemplate).LangID === languages.value[i].ID) {
      return languages.value[i]
    }
  }
  return {} as Language
})

const selectedLang = ref(editTemplateLang.value)
const selectedLangName = computed(() => selectedLang.value.Name)
const selectedLangID = computed(() => selectedLang.value.ID)

const editTemplateReplyTos = computed(() => {
  return editTemplate.value && editTemplate.value.ReplyTos ? editTemplate.value.ReplyTos.join(',') : ''
})
const replyTos = ref(editTemplateReplyTos.value)

const editTemplateCCTos = computed(() => {
  return editTemplate.value && editTemplate.value.CCTos ? editTemplate.value.CCTos.join(',') : ''
})
const ccTos = ref(editTemplateCCTos.value)

const editTemplateSender = computed(() => {
  return editTemplate.value ? editTemplate.value.Sender : ''
})
const sender = ref(editTemplateSender.value)

const editTemplateSubject = computed(() => {
  return editTemplate.value ? editTemplate.value.Subject : ''
})
const subject = ref(editTemplateSubject.value)

const editTemplateBody = computed(() => {
  return editTemplate.value ? editTemplate.value.Body : ''
})
const body = ref(editTemplateBody.value)

const editTemplateUsedFor = computed(() => {
  return editTemplate.value ? editTemplate.value.UsedFor : ''
})
const usedFor = ref(editTemplateUsedFor.value)

const editTemplateDefaultToUsername = computed(() => {
  return editTemplate.value ? editTemplate.value.DefaultToUsername : ''
})
const defaultToUsername = ref(editTemplateDefaultToUsername.value)

const editTemplateID = computed(() => {
  return editTemplate.value ? editTemplate.value.ID : undefined
})
const id = ref(editTemplateID.value)

const template = computed(() => {
  return {
    ID: id.value as string,
    LangID: selectedLangID.value,
    ReplyTos: replyTos.value.split(','),
    CCTos: ccTos.value.split(','),
    Sender: sender.value,
    Subject: subject.value,
    Body: body.value,
    UsedFor: usedFor.value,
    DefaultToUsername: defaultToUsername.value
  } as AppEmailTemplate
})

watch(template, () => {
  emit('update', template.value)
})

const emit = defineEmits<{(e: 'submit', info: AppEmailTemplate): void,
  (e: 'update', info: AppEmailTemplate): void
}>()

const onSubmit = () => {
  emit('submit', template.value)
}

const onLangItemClick = (lang: Language) => {
  selectedLang.value = lang
}

onMounted(() => {
  store.dispatch(ActionTypes.GetAppLangInfos, {
    Message: {
      ModuleKey: ModuleKey.ModuleApplication,
      Error: {
        Title: t('MSG_GET_APP_LANG_INFOS_FAIL'),
        Popup: true,
        Type: NotificationType.Error
      }
    }
  })
})

</script>

<style lang='sass' scoped>
.container
  padding: 24px
  min-width: 800px

.submit-btn
  margin-top: 24px
  background-color: $blue-6
  color: white
</style>
