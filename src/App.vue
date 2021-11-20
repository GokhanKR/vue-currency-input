<template>
  <div id="app" class="container">
    <h1>Currency Input</h1>
    <div class="mb-3">
      <label for="priceInput" class="form-label">Money Format</label>
      <input
        type="text"
        class="form-control"
        id="priceInput"
        placeholder="Price"
        ref="inputRef"
        :value="inputValue"
        @keydown="keyup($event)"
        @focus="setTarget(1)"
        @click="setTarget(1)"
      />
      <br />
      {{ input }}
    </div>
  </div>
</template>

<script>
const options = {
  decimalSeparator: ',',
  currency: 'tr-TR',
  tousandsDefault: '0',
  decimalDefault: '00',
  displayDecimal: 2,
};
export default {
  name: 'App',
  data: () => ({
    input: {
      tousands: options.tousandsDefault,
      decimal: options.decimalDefault,
      set value(value) {
        [this.tousands, this.decimal] = value.split(options.decimalSeparator);
      },
      get value() {
        return this.tousands + options.decimalSeparator + this.decimal;
      },
      target: 1,
    },
    targets: {
      1: (value) => {
        let val = value.replace(/\.|\,/g, '');
        return new Intl.NumberFormat(options.currency).format(val);
      },
      2: (value) => {
        let val =
          value.length <= options.decimalDefault.length
            ? value.padStart(options.displayDecimal, '0')
            : value;

        return val.length <= options.displayDecimal
          ? val
          : val.substr(
              val.length - options.decimalDefault.length,
              options.decimalDefault.length
            );
      },
    },
    inputValue: null,
  }),
  computed: {
    /**
     * @returns {Object}
     */
    inputRef() {
      return this.$refs.inputRef;
    },
    getValue() {
      return this.input.target == 1 ? this.input.tousands : this.input.decimal;
    },
  },
  methods: {
    keyup(e) {
      e.preventDefault();

      let key = e.key,
        keyCode = e.keyCode;

      if ([37, 39, 8, 188, 191].includes(keyCode)) {
        if (keyCode == 37) {
          this.setTarget(1);
        } else if (keyCode == 39 || [188, 191].includes(keyCode)) {
          this.setTarget(2);
        } else if (keyCode == 8) {
          this.setValue(false);
        }
        return;
      }

      let val = Number(key.match(/\d+/g, '')),
        currentVal = val ? val : '';

      if (!val && key != '0') {
        return;
      }

      if (key == '0') currentVal = '0';

      let activeVal = String(this.getValue),
        newVal =
          (activeVal == '0' || activeVal == '00' ? '' : activeVal) + currentVal;

      this.setValue(newVal);
    },
    setTarget(target) {
      let ref = this.inputRef,
        val;
      if (target === 1) {
        val = this.input.tousands.length;
        ref.selectionStart = 0;
        ref.selectionEnd = val;
      } else if (target === 2) {
        val = this.input.tousands.length + this.input.decimal.length + 1;
        ref.selectionStart = this.input.tousands.length + 1;
        ref.selectionEnd = val;
      }
      this.input.target = target;
    },
    setValue(value) {
      let target = this.input.target;

      if (target == 1) {
        this.input.tousands =
          value !== false
            ? this.targets[target](value)
            : options.tousandsDefault;
      } else if (target == 2) {
        this.input.decimal =
          value !== false
            ? this.targets[target](value)
            : options.decimalDefault;
      }

      this.inputValue = this.input.value;
    },
  },
  mounted() {
    this.inputValue = this.input.value;
  },
  watch: {
    inputValue(value) {
      this.$emit('input', value);
    },
  },
};
</script>

<style lang="scss" scoped>
@import 'bootstrap/scss/bootstrap';

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

input {
  caret-color: transparent;
  letter-spacing: 0.05rem;
}

input::selection {
  background-color: #ceebfd;
  color: #032e49;
}
</style>
