<template>
  <b-tab :title="$t('content.label')">
    <b-form-group>
      <c-rich-text-input
        v-model="options.body"
        :labels="{
          urlPlaceholder: $t('content.urlPlaceholder'),
          ok: $t('content.ok'),
          openLinkInNewTab: $t('content.openLinkInNewTab'),
        }"
      />

      <i18next
        path="interpolationFootnote"
        tag="small"
        class="text-muted"
      >
        <code>${record.values.fieldName}</code>
        <code>${recordID}</code>
        <code>${ownerID}</code>
        <span><code>${userID}</code>, <code>${user.name}</code></span>
      </i18next>
    </b-form-group>
  </b-tab>
</template>
<script>
import base from './base'
import { components } from '@cortezaproject/corteza-vue'
const { CRichTextInput } = components

export default {
  i18nOptions: {
    namespaces: 'block',
  },

  // eslint-disable-next-line vue/no-reserved-component-names
  name: 'Content',

  components: {
    CRichTextInput,
  },

  extends: base,
}
</script>
