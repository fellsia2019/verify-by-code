<template>
  <div class="verify-block">
    <div class="verify-block__caption text-base">
      Введите код из смс
    </div>
    <div class="verify-block__body" :style="`--colums: ${countFields}`">
      <input
        v-for="(field, i) in fields"
        :key="`verify-block__input-${field.id}`"
        v-model="field.value"
        maxlength="1"
        min="0"
        max="9"
        type="number"
        class="verify-block__input title-xl tt-uppercase"
        :class="{'verify-block__input--is-error': isRequiredErrorField(i)}"
        :ref="`field-${field.id}`"
        @input="(e) => onInput(e, field)"
        @keydown="(e) => onKeydown(e, field)"
        @beforeinput="onBeforeinput"
      >
    </div>
    <div class="verify-block__actions">
      <button
        class="verify-block__btn"
        @click="onSubmit"
      >
        Отправить
      </button>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import { validationMixin } from "vuelidate";
import { required } from "vuelidate/lib/validators";

export default {
  mixins: [validationMixin],
  props: {
    actionVerify: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      countFields: 6,
      fields: [],
      reg: /[^0-9]/g,
      isValidKeys: ['1','2','3','4','5','6','7','8','9','0'],
    }
  },
  validations: {
    fields: {
      $each: {
        value: { required }
      }
    }
  },
  computed: {
    getFormData() {
      const data = new FormData()

      const code = this.fields
        .map(field => field.value)
        .join('')

      data.append('code', code)

      return data
    },
  },
  methods: {
    initFields() {
      const fields = []

      for (let i = 1; i <= this.countFields; i++) {
        fields.push(
          {
            id: i,
            value: '',
          }
        )
      }

      this.fields = fields
    },
    getField(id, forNextStep = true) {
      const nextId = forNextStep ? id + 1 : id - 1
      const nextField = this.$refs[`field-${nextId}`]
      return nextField && nextField[0] ? nextField[0] : null
    },
    isValidInputValue(e) {
      const expectedValue = e.data ? e.target.value + e.data : e.target.value

      return !expectedValue.match(this.reg)
    },
    onBeforeinput(e) {
      if (!this.isValidInputValue(e)) {
        e.preventDefault()
      }
    },
    onKeydown(e, field) {
      const expectedValue = e.target.value + e.key
      if (
        (!this.isValidKeys.includes(e.key) || expectedValue.length > 1)
        && (e.code !== 'Backspace' && e.key !== 'Backspace')
      ) {
        e.preventDefault()
      }

      // Фокус на след инпут для ввода
      if (
        this.isValidKeys.includes(e.key)
        && this.isValidKeys.includes(e.target.value)
        && field.id !== this.fields.length
      ) {
        const input = this.getField(field.id)
        input.focus()
        if (!input.value) {
          this.fields[field.id].value = e.key
        }
      }

      // Фокус на пред инпут для ввода
      if (
        (e.key === 'Backspace' || e.code === 'Backspace')
        && !e.target.value
        && field.id > 1
      ) {
        e.preventDefault()
        const input = this.getField(field.id, false)
        input.focus()
      }

    },
    onInput(e, field) {
      if (!this.isValidInputValue(e)) {
        e.target.value = e.target.value.replace(this.reg, '')
      }

      // Фокус на след инпут для ввода
      if (e.target.value && field.id !== this.fields.length) {
        const input = this.getField(field.id)
        input.focus()
      }

      // Фокус на пред инпут для ввода
      if (e.inputType === 'deleteContentBackward' && field.id > 1) {
        const input = this.getField(field.id, false)
        input.focus()
      }
    },
    isRequiredErrorField(index) {
      return this.$v.fields.$each[index].$error && !this.$v.fields.$each[index].required
    },
    async onSubmit() {
      this.$v.$touch();

      if (this.$v.$invalid) {
        return
      }

      const data = this.getFormData

      const options = {
        url: this.actionVerify,
        method: 'POST',
        data
      }

      const response = await axios(options);

      if (response.data.status === 'success') {
        this.$emit('submit')
      } else {
        this.$emit('submitError')
      }
    }
  },
  created() {
    this.initFields()
  }
}
</script>

<style lang="scss">
@import '/frontend/scss/base/u-includes';

$b: '.verify-block';
$gap: 8px;
$size: calc((100% - ($gap * (var(--colums) - 1))) / var(--colums));

#{$b} {
  font-family: $font-family-inter;
  color: $black-true;

  // .verify-block__caption
  &__caption {
    margin-bottom: 16px;
  }

  // .verify-block__body
  &__body {
    display: grid;
    grid-template-columns: repeat(var(--colums), $size);
    align-items: center;
    gap: $gap;
  }

  // .verify-block__input
  &__input {
    height: 80px;
    font-family: $font-family-fugue;
    background-color: transparent;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    text-align: center;
    border: 1px solid $color-base-origin;
    border-radius: 16px;
    transition: $transtion-default;
    -moz-appearance: textfield;

    &::-webkit-outer-spin-button,
    &::-webkit-inner-spin-button {
      -webkit-appearance: none;
      margin: 0;
    }

    // .verify-block__input--is-error
    &--is-error {
      border-color: $color-error;;
    }

    @include mobile {
      height: auto;
      width: 100%;
      aspect-ratio: 1/1;
    }
  }

  // .verify-block__actions
  &__actions {
    margin-top: 32px;
  }

  // .verify-block__btn
  &__btn {
    @include mobile {
      width: 100%;
    }
  }
}
</style>
