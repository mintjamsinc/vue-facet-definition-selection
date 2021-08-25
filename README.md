<img src="https://www.mintjams.jp/img/cr.svg" alt ="" width="64">

# vue-facet-definition-selection
A reusable FacetDefinitionSelection component for Vue.js 2.x used by webtop applications.

## Installation

```sh
npm install --save-dev @mintjamsinc/vue-facet-definition-selection
```

## Usage

```vue
<FacetDefinitionSelection
  ref="propertySelection"
  :excludes="propertyKeys"
  class="bg-white"
  v-hook="{inserted: onPropertySelectionLoad}"/>
```

```js
import FacetDefinitionSelection from '@mintjamsinc/vue-facet-definition-selection';

export default {
  components: {
    FacetDefinitionSelection,
  },
  methods: {
    onPropertySelectionLoad() {
      // let ui = vm.$refs.propertySelection.ui;
      // ui.onChanged = function() {};
    },
  },
}
```

## License

[MIT](https://opensource.org/licenses/MIT)

Copyright (c) 2021 MintJams Inc.