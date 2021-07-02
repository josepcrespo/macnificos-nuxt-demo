<template>
  <v-row>
    <v-col
      class="text-center"
      cols="12"
    >
      Árbol resultado de las categorías y subcategorías obtenidas de la API.
    </v-col>
    <v-col cols="12">
      <v-treeview
        :items="categoriesTree"
        dense
      />
    </v-col>
  </v-row>
</template>

<script>
export default {
  name: 'TestMacnificos',
  data () {
    return {
      categoriesFromApi: {},
      categoriesTree: [],
      categoryObjects: [],
      getCategoryObjectsPromise: undefined,
      getCategoriesPromise: this.getCategories()
    }
  },
  mounted () {
    this.getCategoryObjectsPromise = this.getCategoryObjects()
    this.buildCategoriesTree()
  },
  methods: {
    async buildCategoriesTree () {
      await this.getCategoryObjectsPromise

      // https://stackoverflow.com/a/55362597/2332731
      const categoryArr1 = JSON.parse(JSON.stringify(this.categoryObjects))
      const categoryArr2 = categoryArr1

      categoryArr1.forEach((catFromArr1, index1) => {
        categoryArr2.forEach((catFromArr2, index2) => {
          if ('children' in catFromArr1) {
            const results = catFromArr1.children.map((child) => {
              const attachChild = (child.id === catFromArr2.id)
              if (attachChild) {
                categoryArr1[index2].rootCategory = false
              }
              return attachChild ? catFromArr2 : child
            })
            categoryArr1[index1].children = results
          }
        })
      })

      const onlyRootCategories = categoryArr1.filter((category, index, arr) => {
        return category.rootCategory !== false
      })

      this.categoriesTree = onlyRootCategories
    },
    /**
     * Returns a categories array of objects properly formated for displaying the
     * tree in the DOM with the Vuetify `v-treeview` component.
     */
    async getCategoryObjects () {
      await this.getCategoriesPromise
      /**
       * Making use here of the `async/await` syntax for JavaScript promises,
       * guarrants us we already have the categories, before we start to build
       * the final categories tree.
       */
      Object.entries(this.categoriesFromApi).forEach(([key, value]) => {
        /**
         * Example of category object for the categories tree:
         *
         * {
         *   id: 1,
         *   name: 'Applications',
         *   children: [
         *     { id: 2, name: 'Finder' },
         *     { id: 3, name: 'Safari' },
         *     { id: 4, name: 'Mail' }
         *   ]
         * }
         */
        const newCategoryObject = {
          id: window.parseInt(key),
          name: key
        }
        if (Array.isArray(value) && value.length) {
          newCategoryObject.children = this.buildChildrenArray(value)
        }
        this.categoryObjects.push(newCategoryObject)
      })

      return Promise.resolve(this.categoryObjects)
    },
    /**
     * Returns a children array of object properly formated for displaying the
     * tree in the DOM with the Vuetify `v-treeview` component.
     */
    buildChildrenArray (rawChildrenArr) {
      const childrenArray = []
      rawChildrenArr.forEach((item) => {
        childrenArray.push({
          id: item,
          name: item.toString()
        })
      })

      return childrenArray
    },
    /**
     * The API is suposed to return a one level array with the category IDs as
     * the property name. Each category ID has an array of IDs that are children
     * of this category.
     */
    getCategories () {
      // Using randomuser.me API for simulating an AJAX request.
      return this.$axios.get('https://randomuser.me/api/')
        .then((response) => {
          /**
           * If the response is successful, return our mock response.
           * Each object key is a "category" and, each one stores an array of
           * his "subcategories" (or an empty array if don't have any
           * "subcategory"). Each children can be, father of another one or
           * others childs, or from none.
           */
          this.categoriesFromApi = {
            /* eslint-disable indent */
               1: [11, 12],
               2: [21, 22, 23],
               3: [201],
               4: [],
              12: [101],
              21: [101],
              23: [201],
             101: [1001, 1002],
            1002: [10001, 10002, 10003]
            /* eslint-enable indent */
          }
        })
        .catch((error) => {
          /* eslint-disable-next-line no-console */
          console.error(error)
        })
    }
  }
}
</script>
