```javascript
let Vue

class Store {
  constructor(options) {
    this._vm = new Vue({
      data: {
        $$state: options.state
      },
      computed: this.initGetters(options.getters)
    })
    this._mutations = options.mutations
    this._actions = options.actions
  }
  get state() {
    return this._vm.$data.$$state
  }
  get getters() {
    return this._vm
  }
  commit = (type, payload) => {
    this._mutations[type](this.state, payload)
  }
   dispatch = (type, payload) => {
    let context = {
      commit: this.commit,
      state: this.state,
    }
    this._actions[type](context, payload)
  }
}

Store.prototype.initGetters = function (getters) {
  let computed = {}
  Object.keys(getters).forEach(key => {
    computed[key] = () => getters[key](this.state)
  })
  return computed
}

function install(_Vue) {
  Vue = _Vue
  Vue.mixin({
    beforeCreate() {
      let options = this.$options
      if (options.store) {
        this.$store = options.store
      } else if (options.parent && options.parent.$store) {
        this.$store = options.parent.$store

      }
    }
  })
}


export default { 
    Store, 
    install
 };
```
