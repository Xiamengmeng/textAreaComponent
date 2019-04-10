<template>
  <el-cascader
    style="width: 100%"
    :filterable="filterable"
    :clearable="clearable"
    :value="value"
    :options="options"
    :placeholder="placeholder"
    :separator="separator"
    :size="size"
    @change="handleChange"
    @active-item-change="handleItemChange"
    :disabled="isDisable"
  ></el-cascader>
</template>

<script>
  /**
   * 选择所在地-基础组件
   * @author chen.haoran
   * @date 2019-01-17 12:22:11
   */
  import api from '../api/commonApi';
  export default {
    name: "SelectAddress",
    props: {
      value: {
        type: Array,
        default() {
          return [];
        }
      },
      limit: {
        type: Number,
        default: 3
      },
      nations: {
        type: Boolean,
        default: false
      },
      labelPattern: {
        type: Boolean,
        default: false
      },
      filterable: {
        type: Boolean,
        default: true
      },
      clearable: {
        type: Boolean,
        default: true
      },
      placeholder: {
        type: String,
        default: '请选择所在地'
      },
      separator: {
        type: String,
        default: '/'
      },
      size: {
        type: String,
        default: null
      },
      isDisable: {
        type: Boolean,
        default: false
      }
    },
    data() {
      return {
        options: []
      }
    },
    watch: {
      limit(newValue, oldValue) {
        if (newValue && newValue !== oldValue) {
          this.options = this.recursionFormatLimitList(this.options.slice(), newValue, this.nations ? 0 : 1, newValue < oldValue);
        }
      },
      value(newValue, oldValue) {
        let count = 0;
        const timer = setInterval(() => {
          if (this.options.length > 0 || count > 360) {
            clearInterval(timer);
            if (!this.nations && Array.isArray(newValue) && newValue.length >= this.limit && newValue !== oldValue) {
              this.handleItemChange(newValue.slice(0, 2), true);
            } else if (this.limit !== 1)  {
              if (this.labelPattern) {
                const nation = this.options.find(i => i.label === newValue[0]) || {};
                this.handleItemChange([nation.value]);
              } else {
                this.handleItemChange(newValue);
              }
            }
          }
          count++;
        }, 30);
      }
    },
    methods: {
      // 组件事件
      recursionFormatLimitList(options, limit, order, isSmaller) {
        if (!options) { return; }
        if (order > limit + 1) {
          return options;
        } else {
          return options.map(item => {
            if (item.children) {
              if (limit === order) {
                item.children = isSmaller ? null : [];
              } else {
                item.children = this.recursionFormatLimitList(item.children, limit, order + 1);
              }
            } else if (!isSmaller) {
              item.children = [];
            }
            return item;
          });
        }
      },
      recursionFormatList(options, codeList, order) {
        if (order >= codeList.length) {
          return options;
        }
        const value = codeList[order];
        options = options.map(item => {
          if (item.value === value) {
            if (!item.children || (item.children && item.children.length <= 0)) {
              if (item.level === 0) {
                this.getProvinceList((list) => {
                  item.children = this.formatAddressList(list, item.level + 1);
                  if (item.children && (item.level + 2 < this.limit)) {
                    this.handleItemChange(this.value.slice(0, item.level + 2));
                  }
                }, value);
              } else {
                this.getAddressList(value, (list) => {
                  item.children = item.level >= (this.nations ? this.limit - 1 : this.limit) ? null :
                    this.formatAddressList(list, item.level + 1);
                  if (item.children && (item.level + (this.nations ? 2 : 1) < this.limit)) {
                    this.handleItemChange(this.value.slice(0, item.level + (this.nations ? 2 : 1)));
                  }
                });
              }
            } else {
              item.children = this.recursionFormatList((item.children || []).slice(), codeList, order + 1);
            }
          }
          return item;
        });
        return options;
      },
      handleItemChange(codeList) {
        if (this.nations && codeList[0] !== 'CN') { return }
        const options = this.recursionFormatList(this.options.slice(), codeList, 0);
        if (options) {
          this.options = options;
        }
      },
      handleChange(value) {
        if (this.nations && this.limit === 1) {
          const nationVo = this.options.find(i => i.value === value[0]) || {};
          this.$emit('input', [nationVo.value]);
          this.$emit('change', [nationVo]);
        } else {
          const list = [];
          if (value[0]) {
            list[0] = this.options.find(i => i.value === value[0]);
          }
          if (value[1]) {
            list[1] = list[0].children.find(i => i.value === value[1]);
          }
          if (value[2]) {
            list[2] = list[1].children.find(i => i.value === value[2]);
          }
          this.$emit('input', value);
          this.$emit('change', list);
        }
      },
      // 公共事件
      formatNationList(list) {
        list = list.map(item => {
          item.value = item.nationCode;
          item.label = item.nationName;
          item.level = 0;
          item.children = this.limit === 1 ? null : (item.nationCode !== 'CN' ? null : []);
          return item;
        });
        return list;
      },
      formatAddressList(list, level) {
        list = list.map(item => {
          item.value = item.adressCode;
          item.label = item.adress;
          item.level = level;
          item.children = level >= (this.nations ? this.limit - 1 : this.limit) ? null : [];
          return item;
        });
        return list;
      },
      // getNationList() {
      //   api.getNationList().then(data => {
      //     const response = data ? data : {};
      //     if (response.code === 200 || response.ok) {
      //       this.options = this.formatNationList(response.data || []);
      //     }
      //   }).catch(error => { });
      // },
      getProvinceList(callback, nationCode = 'CN') {
        api.getRegionByCode(null, nationCode).then(data => {
          const response = data ? data : {};
          if (response.code === 200 || response.ok) {
            if (callback) {
              callback(response.data || []);
            } else {
              this.options = this.formatAddressList(response.data || [], 1);
            }
          }
        }).catch(error => { });
      },
      getAddressList(code, callback) {
        api.getRegionByCode(code).then(data => {
          const response = data ? data : {};
          if (response.code === 200 || response.ok) {
            if (callback) {
              callback(response.data || []);
            }
          }
        }).catch(error => {});
      }
    },
    mounted() {
    //   if (this.nations) {
    //     this.getNationList();
    //   } else {
        this.getProvinceList();
    //   }
    }
  }
</script>

<style scoped>

</style>
