<template>

<div class="modal" :class="{'is-active': showModal}">
  <div class="modal-background" @click="closeModal"></div>
  <div class="modal-card" style="width:300px">
    <header class="modal-card-head">
      <p class="modal-card-title is-size-4 has-text-info has-text-weight-semibold">{{ $t("msg.lang.title") }}</p>
      <button class="delete" aria-label="close" @click="closeModal"></button>
    </header>
    <section class="modal-card-body" style="height:200px">
      
      
      <div class="field">
        <label class="label">{{ $t("msg.lang.lang") }}</label>

        <div class="control">
          <div class="select">
            <select v-model="localeSelected">
              <option disabled value="">{{current}}</option>
              <option v-for="(lang, locale) in langs" :value="locale" :key="lang.id">{{lang}}</option>
            </select>
          </div>
        </div>
      </div>
      
       <br/>
      <div class="field is-grouped">
        <div class="control">
          <button class="button is-info" @click="select">{{ $t("msg.lang.select") }}</button>
        </div>
        <div class="control">
          <button class="button is-text" @click="closeModal">{{ $t("msg.cancel") }}</button>
        </div>
      </div>
    </section>
  </div>
</div>

</template>
<script>
import { messageBus } from '@/messagebus'
import { locale, langs, setLocale } from '../../shared/config'

export default {
  name: "lang",
  props: {
    showModal: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      current: langs[locale],
      localeSelected: '',
      langs:langs
    }
  },
  methods: {
    select(){
      this.$i18n.locale = this.localeSelected
      setLocale(this.localeSelected)
      messageBus.$emit('selectLocale', this.localeSelected)
      this.closeModal()
    },
    closeModal() {
      messageBus.$emit('close', 'windowLang');
    }
  }
}
</script>
<style>
</style>
